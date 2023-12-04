# ML za detekciju DoS napada na web serveru

## Uvod

Na prvom virtualnom stroju biti će instaliran nginx web server s access logom, taj log će se pokretanjem Python skripti transformirati u JSON.

Drugi virtualni stroj će imati clickhouse DB na sebi koji će povlačiti podatke iz JSONA u tablicu.

Treći virtualni stroj će pokretati ML skriptu (Naive Bayes klasifikator) koji će detektirati DoS napade na temelju web access log-a.

## Infrastruktura

Tri virtualna stroja:

1. nginx web server - archlinux VM
2. clickhouse baza podataka - archlinux VM
3. archlinux VM s Python-om 3 za ML

### Podešavanje Infrastrukture

1. Preuzeta je archlinux cloud slika za VM na [linku](https://gitlab.archlinux.org/archlinux/arch-boxes/-/jobs/154961/artifacts/browse/output)
2. Kreirane su tri virtualke koristeći virt-manageru
3. Stvorena je cloud-init skripta za inicijalizaciju osnovnih postavki virutalnog stroja:

```shell
#cloud-config
users:
  - default

system_info:
   default_user:
     name: user
     plain_text_passwd: '1111'
     gecos: arch Cloud User
     groups: [wheel, adm]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
     lock_passwd: False
```

4. Iz cloud-init skripte kreirana je CD-ROM slika za podizanje sustava (inicijalizaciju postavki preuzete slike archlinuxa):

```shell
xorriso -as genisoimage -output cloud-init-1.iso -volid CIDATA -joliet -rock user-data meta-data
xorriso -as genisoimage -output cloud-init-2.iso -volid CIDATA -joliet -rock user-data meta-data
xorriso -as genisoimage -output cloud-init-3.iso -volid CIDATA -joliet -rock user-data meta-data
```

5. Pokrenute su virtualke

#### Virtualka nginx

Prvo smo saznali IP adresu od virtualke zatim smo se spojili na nju:

```shell
lljubojevic@IdeaPad-5-Pro:~$ ssh user@192.168.122.224
```

Preimenovali smo hostname da znamo o kojoj je riječ:

```shell
[user@archlinux ~]$ sudo hostnamectl hostname nginx
```

Instaliramo nginx:

```shell
[user@archlinux ~]$ sudo pacman -S nginx
resolving dependencies...
looking for conflicting packages...

Packages (4) geoip-1.6.12-2  geoip-database-20230519-1  mailcap-2.1.53-1  nginx-1.24.0-1

Total Download Size:   3.84 MiB
Total Installed Size:  9.76 MiB

:: Proceed with installation? [Y/n] y
...
:: Running post-transaction hooks...
(1/2) Reloading system manager configuration...

[user@nginx ~]$ sudo systemctl enable nginx
[user@nginx ~]$ sudo systemctl start nginx

[user@archlinux ~]$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2023-05-26 12:55:36 UTC; 1s ago
    Process: 831 ExecStart=/usr/bin/nginx -g pid /run/nginx.pid; error_log stderr; (code=exited, status=0/SUCCESS)
   Main PID: 832 (nginx)
      Tasks: 2 (limit: 2306)
     Memory: 2.2M
        CPU: 18ms
     CGroup: /system.slice/nginx.service
             ├─832 "nginx: master process /usr/bin/nginx -g pid /run/nginx.pid; error_log stderr;"
             └─833 "nginx: worker process"
```

Provjerimo radi li nginx:

```shell
lljubojevic@IdeaPad-5-Pro:~$ curl -ILX GET "http://192.168.122.224:80/"
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Fri, 26 May 2023 12:56:55 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 22 May 2023 23:05:34 GMT
Connection: keep-alive
ETag: "646bf53e-267"
Accept-Ranges: bytes
```

nginx radi.

Idući je korak prebaciti access log koji ćemo koristiti u odgovarajući direktorij:

```shell
lljubojevic@IdeaPad-5-Pro:~/Documents/lukaljubojevic/llj/alll$ scp access.log user@192.168.122.224:/home/user/access-2.log
user@192.168.122.224's password:
access.log

[user@archlinux ~]$ ls
access-2.log

[user@archlinux ~]$ sudo mv access-2.log /var/log/nginx/access-2.log

[user@archlinux ~]$ ls /var/log/nginx/
access-2.log  access.log
```

Napomena postoje dva access.log-a, access-2.log je log napunjen podacima koji koristimo, a access.log je defaultni kojeg je nginx stvorio.

Zatim pomoću skripte za transformaciju i randomizaciju loga u json (genjson.py) transformiramo log u pseudo-json format:

```python
import re
import sys
from typing import Dict, List

def parse_log_line(line: str) -> Dict[str, str]:
    pattern = r'(?P<ip>(?:\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})|(?:[A-Fa-f0-9]{0,4}:){2,7}[A-Fa-f0-9]{0,4}) - (?P<identd>-|\w*) \[(?P<timestamp>.*?)\] "(?P<request>.*?)" (?P<status>\d{3}) (?P<size>\d+|-) "(?P<referrer>.*?)" "(?P<user_agent>.*?)"'
    match = re.search(pattern, line)

    if match:
        return match.groupdict()
    else:
        return None
seen = {}
def parse_log_file(file_path: str) -> List[Dict[str, str]]:
    global generated_ips
    global seen
    parsed_data = []
    failed =0

    with open(file_path, 'r') as log_file:
        for line in log_file:
            parsed_line = parse_log_line(line)
            if parsed_line:

                if seen.get( parsed_line['ip'] ):
                  parsed_line['ip'] = seen[parsed_line['ip']]
                else:
                  seen[parsed_line['ip']] = generated_ips.pop()
                  parsed_line['ip'] = seen[parsed_line['ip']]

                pat = r'(?<=/~)(\w+)'
                repl = re.sub(pat, replace_with_lookup, parsed_line['request'])
                parsed_line['request'] = repl
                repl = re.sub(pat, replace_with_lookup, parsed_line['referrer'])
                parsed_line['referrer'] = repl

                pat = r'(?<=/%7E)(\w+)'
                repl = re.sub(pat, replace_with_lookup, parsed_line['request'])
                parsed_line['request'] = repl
                repl = re.sub(pat, replace_with_lookup, parsed_line['referrer'])
                parsed_line['referrer'] = repl

                dompat = r'(?:https?:\/\/)?(?:www\.)?[\w-]+(\.[\w-]+)*\.(?:com|org|net|edu|gov|io)(\/\S*)?'
                repl = re.sub(dompat, replace_with_lookup, parsed_line['request'])
                parsed_line['request'] = repl
                repl = re.sub(dompat, replace_with_lookup, parsed_line['referrer'])
                parsed_line['referrer'] = repl

                parsed_data.append(parsed_line)
            else:
                failed += 1
    print("FAILED:", failed)
    return parsed_data

import socket
import struct

def increment_ip_address(ip: str) -> str:
    packed_ip = socket.inet_aton(ip)
    ip_integer = struct.unpack("!I", packed_ip)[0]
    ip_integer += 1
    if ip_integer > 0xFFFFFFFF:
        raise ValueError("IP address exceeded the valid range.")

    incremented_ip = socket.inet_ntoa(struct.pack("!I", ip_integer))
    return incremented_ip

def generate_ip_addresses(start_ip: str, n: int):
    ip_addresses = [start_ip]

    for _ in range(n - 1):
        next_ip = increment_ip_address(ip_addresses[-1])
        ip_addresses.append(next_ip)

    return ip_addresses

def replace_with_lookup(match):
    global generated_ips
    global seen
    val =  seen.get(match.group(0))
    if not val:
      seen[match.group(0)] = generated_ips.pop()
      val =  seen[match.group(0)]
    return val

start_ip = "192.0.0.0"
num_ips_to_generate = 50000
generated_ips = generate_ip_addresses(start_ip, num_ips_to_generate)
log_file_path = "access.log"
parsed_log_data = parse_log_file(log_file_path)

for entry in parsed_log_data:
    print(entry)
```

Nakon toga koristimo novi python skriptu (syntax.py) koja će pročešljati stvorene jsone i ispraviti ih u skladu s json sintaksom te odbaciti one objekte koji nisu pravilno generirani:

```python
import json
import re
def load_json_lines(file_path: str):
    json_data = []

    with open(file_path, 'r') as file:
        for line in file:
            try:
                json_object = json.loads(line)
                json_data.append(json_object)
            except json.JSONDecodeError:
                pass

    return json_data
file_path = "access.jsons"
loaded_json = load_json_lines(file_path)

for entry in loaded_json:
    print(entry)
    #pass
```

I za kraj ponovo koristimo skriptu (timestamp.py) koja pretvara polje u json objektima timestamp u format kojeg clickhouse može razumjeti, clickhouse ne podržava apache timestamp format:

```python
import json
import datetime

def convert_timestamp(timestamp):
    parsed_timestamp = datetime.datetime.strptime(timestamp, "%d/%b/%Y:%H:%M:%S")
    converted_timestamp = parsed_timestamp.strftime("%d%m%y%H%M%S")
    return converted_timestamp

json_file = "access.jsons"

data = []
with open(json_file, "r") as file:
    for line in file:
        try:
            entry = json.loads(line)
            if "timestamp" in entry:
                entry["timestamp"] = convert_timestamp(entry["timestamp"])
            data.append(entry)
        except json.JSONDecodeError:
            print(f"Ignoring invalid JSON line: {line}")

with open(json_file, "w") as file:
    for entry in data:
        file.write(json.dumps(entry) + "\n")
```

Nakon korištenja svih skripti dobivamo datoteku access2.jsons koju ćemo prebaciti u clickhouse bazu podataka.
JSON-izirani log nam je ovog oblika:

```json
...
{"ip": "192.0.194.157", "identd": "", "timestamp": "140321194504", "request": "GET / HTTP/1.1", "status": "200", "size": "367", "referrer": "", "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36"}
{"ip": "192.0.194.156", "identd": "", "timestamp": "100321022141", "request": "GET / HTTP/1.1", "status": "200", "size": "367", "referrer": "", "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36"}
{"ip": "192.0.194.155", "identd": "", "timestamp": "240421221417", "request": "GET / HTTP/1.1", "status": "200", "size": "367", "referrer": "", "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36"}
{"ip": "192.0.194.154", "identd": "", "timestamp": "230421002823", "request": "GET / HTTP/1.1", "status": "200", "size": "367", "referrer": "", "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36"}
...
```

#### Virutalka Clickhouse

Prvo mijenjamo hostname virtualke radi lakšeg prepoznavanja:

```shell
[user@archlinux ~]$ hostnamectl hostname clickhouse
```

Preuzimamo i instaliramo clickhouse DB:

```shell
[user@clickhouse ~]$ curl https://clickhouse.com/ | sh
[user@clickhouse ~]$ sudo ./clickhouse install

Copying ClickHouse binary to /usr/bin/clickhouse.new
Renaming /usr/bin/clickhouse.new to /usr/bin/clickhouse.
...

ClickHouse has been successfully installed.

Start clickhouse-server with:
 sudo clickhouse start

Start clickhouse-client with:
 clickhouse-client --password
```

Pokrećemo clickhouse DB server:

```shell
[user@clickhouse ~]$  sudo clickhouse start
 chown -R clickhouse: '/var/run/clickhouse-server/'
Will run sudo -u 'clickhouse' /usr/bin/clickhouse-server --config-file /etc/clickhouse-server/config.xml --pid-file /var/run/clickhouse-server/clickhouse-server.pid --daemon
Waiting for server to start
Waiting for server to start
Server started
```

Stvaramo tablicu za prebacivanje log zapisa:

```shell
clickhouse :) CREATE TABLE IF NOT EXISTS HTTPLOG
(
    ip String,
    identd String,
    timestamp DateTime,
    request String,
    status String,
    size UInt32,
    referrer String,
    user_agent String
)
ENGINE = MergeTree()
ORDER BY timestamp;

CREATE TABLE IF NOT EXISTS HTTPLOG
(
    `ip` String,
    `identd` String,
    `timestamp` DateTime,
    `request` String,
    `status` String,
    `size` UInt32,
    `referrer` String,
    `user_agent` String
)
ENGINE = MergeTree
ORDER BY timestamp

Query id: 845c685a-82de-4cd8-9cc9-3bd165da044f

Ok.

0 rows in set. Elapsed: 0.025 sec.
```

Instaliramo JSON linter JQ:

```shell
[user@nginx ~]$ sudo pacman -S jq
```

Učitavamo JSON-izirani log u clickhouse:

Iz clickhouse dokumentacije:

```text
Examples
Using HTTP interface:

$ echo '{"foo":"bar"}' | curl 'http://localhost:8123/?query=INSERT%20INTO%20test%20FORMAT%20JSONEachRow' --data-binary @-
```

Pretvaramo u nama korisniju "one-liner" naredbu:

```shell
[user@nginx ~]$ cat access2.jsons |  while read -r line; do echo "$line" | curl -u default:1111 'http://192.168.122.204:8123/?query=INSERT%20INTO%20HTTPLOG%20FORMAT%20JSONEachRow' --data-binary @-; done

Code: 241. DB::Exception: Memory limit (total) exceeded: would use 1.77 GiB (attempt to allocate chunk of 4465068 bytes), maximum: 1.73 GiB. OvercommitTracker decision: Query was selected to stop by OvercommitTracker. (MEMORY_LIMIT_EXCEEDED) (version 23.5.1.2209 (official build))
```

* Ova naredba ispisuje cijeli access2.jsons i čita ga liniju po liniju, svaku liniju zatim pipe-a u API poziv (po query stringu vidimo da je API poziv SQL sintaksa INSERT INTO) koji sprema svaku liniju u bazu podataka.
* Output "Code: 241" javlja se iz razloga što VM ima dedicirano samo 2GB memorije, međutim to ne utječe na ispravnost rezultata već samo na brzinu popunjavanja podataka

Provjerimo jesu li svi zapisi na broju:

```shell
clickhouse :) SELECT * FROM HTTPLOG
...
│ 192.0.144.1   │        │ 1971-12-18 14:56:06 │ GET /.git/objects/4a/check_mk/login.py HTTP/1.1                                    │ 404    │  256 │          │ cyberscan.io
...

354654 rows in set. Elapsed: 0.160 sec. Processed 354.65 thousand rows, 52.86 MB (2.22 million rows/s., 330.97 MB/s.)
```

Jesu.

#### Virtualka ML

Hostname radi lakšeg raspoznavanja:

```shell
[user@archlinux ~]$ sudo hostnamectl hostname ML
```

Instalacija potrebnih paketa za čitanje iz clickhouse baze i scikit-learn-a:

```shell
[user@ML ~]$ pip install clickhouse-driver
[user@ML ~]$ pip install scikit-learn
```

Korisimo slijedeći kod za detekciju DoS napada:

```python
import clickhouse_driver
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

connection = clickhouse_driver.connect(
    host='192.168.122.204',
    port=9000,
    user='default',
    password='1111',
    database='default'
)

query = "SELECT * FROM HTTPLOG ORDER BY timestamp ASC"

cursor = connection.cursor()
cursor.execute(query)
data = cursor.fetchall()

X = []
y = []

ip_status = {}

for row in data:
    ip_address = row[0]
    response_code = row[4]
    log_entry = row[3]

    if ip_address not in ip_status:
        ip_status[ip_address] = response_code
    else:
        if response_code == '503' and ip_status[ip_address] == '200':
            X.append(log_entry)
            y.append(1)
        else:
            X.append(log_entry)
            y.append(0)

        ip_status[ip_address] = response_code


vectorizer = CountVectorizer()
X_transformed = vectorizer.fit_transform(X)
classifier = MultinomialNB()
classifier.fit(X_transformed, y)

total_count = len(y)
dos_count = sum(y)
normal_count = total_count - dos_count

print("Total requests:", total_count)
print("DoS attack requests:", dos_count)
print("Normal requests:", normal_count)
```

* ip_status = {}  # Dictionary za praćenje statusa requesta (200 or 503)
* ip_address = row[0]  # IP je u prvom stupcu tablice
* response_code = row[4]  # Response kod je u petom
* if response_code == '503' and ip_status[ip_address] == '200' # Ako se otkrije da je za istu IP adresu, prethodni odgovor bio 200 a je idući 503 obilježi kao DoS napad
* classifier = MultinomialNB() # Treniranje Naive Bayes klasifikatora

Pokrenimo sustav detekcije DoS napada:

```shell
[user@ML ~]$ python3 ml.py
Normal access log entry.
Total requests: 338655
DoS attack requests: 1
Normal requests: 338654
```

I stvarno ako pretražimo cijeli log (grep), vidimo da je samo jedan slučaj gdje za istu IP adresu imamo prvo odgovore 200 pa zatim 503:

```json
...
{"ip": "192.0.195.0", "identd": "", "timestamp": "010521065723", "request": "OPTIONS * HTTP/1.0", "status": "200", "size": "", "referrer": "", "user_agent": "Apache/2.4.38 (Debian) mod_auth_kerb/5.4 mod_fcgid/2.3.9 OpenSSL/1.1.1n (internal dummy connection)"}
{"ip": "192.0.195.0", "identd": "", "timestamp": "010521065724", "request": "OPTIONS * HTTP/1.0", "status": "200", "size": "", "referrer": "", "user_agent": "Apache/2.4.38 (Debian) mod_auth_kerb/5.4 mod_fcgid/2.3.9 OpenSSL/1.1.1n (internal dummy connection)"}
{"ip": "192.0.195.0", "identd": "", "timestamp": "010521065725", "request": "OPTIONS * HTTP/1.0", "status": "503", "size": "", "referrer": "", "user_agent": "Apache/2.4.38 (Debian) mod_auth_kerb/5.4 mod_fcgid/2.3.9 OpenSSL/1.1.1n (internal dummy connection)"}
{"ip": "192.0.195.0", "identd": "", "timestamp": "010521065727", "request": "OPTIONS * HTTP/1.0", "status": "503", "size": "", "referrer": "", "user_agent": "Apache/2.4.38 (Debian) mod_auth_kerb/5.4 mod_fcgid/2.3.9 OpenSSL/1.1.1n (internal dummy connection)"}
{"ip": "192.0.195.0", "identd": "", "timestamp": "010521065730", "request": "OPTIONS * HTTP/1.0", "status": "503", "size": "", "referrer": "", "user_agent": "Apache/2.4.38 (Debian) mod_auth_kerb/5.4 mod_fcgid/2.3.9 OpenSSL/1.1.1n (internal dummy connection)"}
...
```
