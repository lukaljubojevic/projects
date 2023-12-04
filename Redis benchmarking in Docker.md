# Mjerenje performansi Redis baze podataka

## Način rada s jednom Redis instancom

Pokrećemo Redis Docker kontejner:

```shell
docker pull docker pull bitnami/redis:latest
```

Kreiramo docker-compose.yml datoteku:

```shell
vim docker-compose.yml
```

i punimo ju sadržajem oblika:

```yaml
version: '3'
services:
  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
volumes:
  redis-data:
```

> Datoteka docker-compose.yml pokreće redis na zadanom portu 6379 i montira na sebe /data Docker volumen.

Pokretanje Redis kontejnera vršimo koristeći Docker Compose:

```shell
docker compose up -d
```

Izlaz:

```shell
[luka@test-vm ~]$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                                       NAMES
0cbec8f7fb82   redis:latest   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   documents-redis-1
```

Za spajanje na Redis, odnosno interakciju s redis-cli koristimo redis-cli:

```shell
sudo dnf install redis -y # paket koji omogućuje korištenje redis-cli
redis-cli -h localhost -p 6379
```

Izlaz:

```shell
[luka@test-vm ~]$ redis-cli -h localhost -p 6379
localhost:6379> DBSIZE
(integer) 0
```

Naredba za stopiranje redisa:

```shell
docker compose down
```

### Opterećivanje baze podataka podacima

#### Hardverske spesifikacije računala

```shell
[luka@test-vm Documents]$ neofetch
 luka@test-vm.lab
  ------------------- 
OS: Rocky Linux 9.1 (Blue Onyx) x86_64 
Host: Virtual Machine Hyper-V UEFI Release v4.1 
Kernel: 5.14.0-162.6.1.el9_1.0.1.x86_64 
Uptime: 1 hour, 46 mins 
Packages: 1898 (rpm) 
Shell: bash 5.1.8 
Resolution: 1920x1080 
DE: GNOME 40.10 
WM: Mutter 
WM Theme: Adwaita 
Theme: Adwaita [GTK2/3] 
Icons: Adwaita [GTK2/3] 
Terminal: gnome-terminal 
CPU: AMD EPYC 7763 (2) @ 3.216GHz 
Memory: 2371MiB / 7680MiB 
```

### Stvaranje zapisa u Redis bazi

Potrebno ući u shell pokrenutog redis kontejnera te pomoću skripte stvariti 50000 zapisa u bazi podataka.

Skripta:

```shell
eval "local a={} for i=1,1000 do a[i] = '1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111' end for i=1, 50000 do redis.call('RPUSH', i, unpack(a)) end" 0
```

Izlaz:

```shell
[luka@test-vm ~]$ redis-cli -h localhost -p 6379
localhost:6379> eval "local a={}  for i=1,1000 do  a[i] = '1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111' end for i=1, 5000 do redis.call('RPUSH', i, unpack(a)) end" 0
(nil)
(0.76s)
```

Provjera veličine baze podataka:

```shell
localhost:6379> DBSIZE
(integer) 50000
```

### Redis-benchmark

Pokretanje mjerenja performansi na redis kontejneru. Koristiti će se ugrađeni redis-benchmark.

```shell
redis-benchmark -t set,get -d 1000000 -n 100000 -q
```

> parametrom -d postavljamo broj bajtova po ključu
> parametrom -n postavljamo broj zahtjeva
> parametrom -q postavljamo izvođenje redis-benchmark alata da se izvršava u pozadini

Izlaz:

```shell
[luka@test-vm Documents]$ sudo docker exec -i 0cbec8f7fb82 bash -c "redis-benchmark -t set,get -d 1000000 -n 100000 -q"
SET: 3966.99 requests per second, p50=6.055 msec                    
GET: 2526.27 requests per second, p50=15.671 msec
```

> Stvoreni Redis kontejner obradi 3966 SET zahtjeva po sekundi i 2526 GET zahtjeva po sekundi.

Zatim pomoću bash skripte se spajamo na Redis Docker kontejner (putem PID-a), izvršavamo redis-benchmark s 100 000 GET i SET zahtjeva te dužinom ključa od 1000000 bajtova. Potom se putem tri FOR petlje mjeri opterećenje procesora, korištenje tvrdog diska domaćinskog računala te zauzeće RAM memorije. Mjerenje se izvršava u 10000 koraka i svaki korak mjerenja zapisuje se u .txt datoteku.

Skripta:

```shell
#!/bin/bash

CONTAINER_ID=$(docker ps -q | awk 'NR==1{print $1}')
PID=$(pidof redis-server)

# Start a Redis benchmark in a Docker container
sudo docker exec -i $CONTAINER_ID bash -c "redis-benchmark -t set,get -d 1000000 -n 100000 -q" &

# Write the load average to cpuload.txt
for i in {1..10000}; do
  TEST=$(cat /proc/loadavg | awk '{print $1}') && echo "$1 $TEST" >> loadcpu.txt
done &

# Write the disk usage of /var/lib/docker/overlay2 to diskus.txt
for i in {1..10000}; do
  cat /proc/$PID$/io | awk '$1 !~ /cancelled_write_bytes/ && ($1 ~ /read_bytes/ || $1 ~ /write_bytes/){printf("%s ",($2/(1024 * 1024)))} END {print ""}' >> disk.txt
done &

for i in {1..10000}; do
  mem_usage=$(cat /proc/$PID/smaps | grep -i pss | awk '{Total+=$2} END {print Total/1024""}')
  echo "$i $mem_usage" >> ram.txt
done &

# Wait for all background jobs to finish
wait

# Set the output file for the first plot
gnuplot -e "set terminal png" -e "set output \"ram.png\"" -e "set xlabel \"Timeline\"" -e "set ylabel \"MB\"" -e "plot \"ram.txt\" w lp"

# Set the output file for the second plot
gnuplot -e "set terminal png" -e "set xlabel \"write MB\"" -e "set ylabel \"read MB\"" -e "set output \"disk.png\"" -e "plot \"disk.txt\" w lp"

# Set the output file for the third plot
gnuplot -e "set terminal png" -e "set xlabel \"Timeline\"" -e "set ylabel \"jobs AVG\"" -e "set output \"loadcpu.png\"" -e "plot \"loadcpu.txt\" w lp"
```

> Na kraju kad su svi podaci zapisani u .txt datoteke, skripta generira grafove koji prikazuju stanje opterećenja.

#### Dobiveni rezultat - grafovi opterećenja

Opterećenje procesora:

![cpu](Images/loadcpu.png)

Opterećenje RAM memorije:

![ram](Images/ram.png)

Opterećenje diska:

![disk](Images/disk.png)

## Skaliranje na više Redis instanci

Što je AOF?
>How Redis writes data to disk
>
>Persistence refers to the writing of data to durable storage, such as a solid-state disk (SSD). Redis provides a range of persistence options.
>
>These include:
>
>RDB (Redis Database): RDB persistence performs point-in-time snapshots of your dataset at specified intervals.
>
>AOF (Append Only File): AOF persistence logs every write operation received by the server. These operations can then be replayed again at server startup, reconstructing the original dataset. Commands are logged using the same format as the Redis protocol itself.
>
>No persistence: You can disable persistence completely. This is sometimes used when caching.
>
>RDB + AOF: You can also combine both AOF and RDB in the same instance.

Koristiti ćemo RDB + AOF.

Kako se vrši repliciranje u redisu?
>At the base of Redis replication (excluding the high availability features provided as an additional layer by Redis Cluster or Redis Sentinel) there is a leader follower (master-replica) replication that is simple to use and configure. It allows replica Redis instances to be exact copies of master instances. The replica will automatically reconnect to the master every time the link breaks, and will attempt to be an exact copy of it regardless of what happens to the master.
>
>This system works using three main mechanisms:
>
> When a master and a replica instances are well-connected, the master keeps the replica updated by sending a stream of commands to the replica to replicate the effects on the dataset happening in the master side due to: client writes, keys expired or evicted, any other action changing the master dataset.
>
> When the link between the master and the replica breaks, for network  issues or because a timeout is sensed in the master or the replica, the replica reconnects and attempts to proceed with a partial resynchronization: it means that it will try to just obtain the part of the stream of commands it missed during the disconnection.
>
>When a partial resynchronization is not possible, the replica will ask for a full resynchronization. This will involve a more complex process in which the master needs to create a snapshot of all its data, send it to the replica, and then continue sending the stream of commands as the dataset changes.

```shell
docker pull docker pull bitnami/redis:latest
```

### Konfiguracijska datoteka docker-compose koja podržava replikaciju redis baze podataka

```yaml
version: '2'

services:
    redis-master:
      image: 'bitnami/redis:latest'
      ports:
        - '6379'
      environment:
        - REDIS_REPLICATION_MODE=master
        - REDIS_PASSWORD=jakotajnipass

    redis-secondary:
      image: 'bitnami/redis:latest'
      environment:
        - REDIS_DISABLE_COMMANDS=FLUSHDB,FLUSHALL,CONFIG
        - REDIS_REPLICATION_MODE=slave
        - REDIS_MASTER_HOST=redis-master
        - REDIS_MASTER_PORT_NUMBER=6379
        - REDIS_MASTER_PASSWORD=jakotajnipass
        - REDIS_PASSWORD=mojpasswd
      ports:
        - '6379'
```

### Pojašnjenje konfiguracijske datoteke

Konfiguracija se sastoji od definicije dva servisa: master-a i slave-a. Master je glavni čvor koji obrađuje sve podatke, a slave čvorovi dohvaćaju asinkrono podatke od mastera i sadrže iste podatke.

Koristimo sliku 'bitnami/redis:latest', pokrećemo redis server na vratima 6379 te koristimo varijable okruženja za konfiguraciju načina rada.

Master:
Definira način replikacije master (definira master čvor način rada) te lozinku za master čvor.

Slave:
Definira nedozvoljene komande (brisanje i konfiguraciju baze podataka) - s obzirom da je replika, nije nam u interesu da se mijenja njezina konfiguracija niti sadržaj baze. Definiramo način rada slave, ime master hosta, vrata na kojima je pokrenut master, njegovu lozinku te lozinku svakog slave čvora

Pomoću ove naredbe inicijaliziramo master čvor:

```shell
lljubojevic@LjubojevicPC:~/Desktop$ sudo docker run --name master-cache -e REDIS_REPLICATION_MODE=master  -e REDIS_PASSWORD=jakotajnipass  bitnami/redis:latest
```

Pomoću ove naredbe pokrećemo docker kontejnere na način da se pokrene jedan master čvor te tri pomoćna slave čvora.

lljubojevic@LjubojevicPC:~/Desktop$ sudo docker-compose up --scale redis-master=1 --scale redis-secondary=3

Vidimo da su nastala tri kontejnera (tri procesa):

```shell
lljubojevic@IdeaPad-5-Pro:~$ sudo docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                         NAMES
6f2926fcc8c9   bitnami/redis:latest   "/opt/bitnami/script…"   48 seconds ago   Up 47 seconds   0.0.0.0:49154->6379/tcp, :::49154->6379/tcp   pz_redis-secondary_3
c915199a3522   bitnami/redis:latest   "/opt/bitnami/script…"   48 seconds ago   Up 47 seconds   0.0.0.0:49153->6379/tcp, :::49153->6379/tcp   pz_redis-secondary_1
4810ac537405   bitnami/redis:latest   "/opt/bitnami/script…"   48 seconds ago   Up 47 seconds   0.0.0.0:49156->6379/tcp, :::49156->6379/tcp   pz_redis-master_1
c13f0ee4b01d   bitnami/redis:latest   "/opt/bitnami/script…"   48 seconds ago   Up 47 seconds   0.0.0.0:49155->6379/tcp, :::49155->6379/tcp   pz_redis-secondary_2
```

### Opterećivanje baze podataka podacima i testiranje replikacije

Ulazimo u shell master čvora, pokrećemo redis naredbenu ljusku, vršimo autentifikaciju te pomoću [LUA](https://www.lua.org/about.html) skripte stvaramo 5000 zapisa u bazi podataka.

```shell
lljubojevic@IdeaPad-5-Pro:~/PZ$ sudo docker exec -it 4810ac537405 bash

I have no name!@4810ac537405:/$ redis-cli 

127.0.0.1:6379> AUTH jakotajnipass
OK

127.0.0.1:6379> 127.0.0.1:6379> eval "local a={}  for i=1,1000 do  a[i] = '1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111' end for i=1, 5000 do redis.call('RPUSH', i, unpack(a)) end" 0
```

Provjerimo veličinu baze podataka:

```shell
127.0.0.1:6379> dbsize
(integer) 50002

I have no name!@4810ac537405:/bitnami/redis/data$ ls -lah
total 108M
drwxrwxr-x 1 root root 4.0K Jan  6 15:50 .
drwxrwxr-x 1 root root 4.0K Jan  3 20:49 ..
drwxr-xr-x 2 1001 root 4.0K Jan  6 15:50 appendonlydir
-rw-r--r-- 1 1001 root 126M Jan  6 15:50 dump.rdb
```

Provjerimo veličinu datoteke redis baze podataka na jednom od slave čvorova (secondary_3):

```shell
I have no name!@6f2926fcc8c9:/bitnami/redis/data$ ls -lah
total 125M
drwxrwxr-x 1 root root 4.0K Jan  6 15:55 .
drwxrwxr-x 1 root root 4.0K Jan  3 20:49 ..
drwxr-xr-x 2 1001 root 4.0K Jan  6 15:55 appendonlydir
-rw-r--r-- 1 1001 root 126M Jan  6 15:55 dump.rdb
```

Podaci su se replicirali.

### Mjerenje performansi

Provjeravamo naredbom htop PID od redis-server servisa koji se izvodi u master Docker kontejneru:

![Redis server](Images/reds.png)

Provjerimo odmah i postoji li servis za repliciranje podataka:

![Redis replication](Images/rdb.png)

Postoji.

#### redis-benchmark

Pokrenimo mjerenje performansi na master čvoru. Koristimo ugrađeni alat redis-benchmark koji izvodi zahtjeve tipa GET i SET s parametrom -d (broj bytova po ključu) 1000000, parametrom -n kao brojem zahtjeva 1000 i -q kao parametar za izvođenje u pozadini (output samo trenutne brzine izvođenja):

```shell
lljubojevic@IdeaPad-5-Pro:~/PZ$ sudo docker exec -it 4810ac537405 bash

I have no name!@4810ac537405:/$ redis-benchmark -a jakotajnipass -t set,get -d 1000000 -n 1000 -q
SET: 692.04 requests per second, p50=31.535 msec                   
GET: 1310.62 requests per second, p50=0.495 msec   
```

Vidimo da su performanse redis-servera na ovom okruženju:

```shell
lljubojevic@IdeaPad-5-Pro:~$ screenfetch
                                       lljubojevic@IdeaPad-5-Pro
 MMMMMMMMMMMMMMMMMMMMMMMMMmds+.        OS: Linuxmint 21 vanessa
 MMm----::-://////////////oymNMd+`     Kernel: x86_64 Linux 5.15.0-57-generic
 MMd      /++                -sNMd:    Uptime: 2h 45m
 MMNso/`  dMM    `.::-. .-::.` .hMN:   Packages: 3019
 ddddMMh  dMM   :hNMNMNhNMNMNh: `NMm   Shell: bash 5.1.16
     NMm  dMM  .NMN/-+MMM+-/NMN` dMM   Resolution: 6400x1600
     NMm  dMM  -MMm  `MMM   dMM. dMM   DE: KDE 5.98.0 / Plasma 5.24.7
     NMm  dMM  -MMm  `MMM   dMM. dMM   WM: KWin
     NMm  dMM  .mmd  `mmm   yMM. dMM   GTK Theme: Breeze [GTK2/3]
     NMm  dMM`  ..`   ...   ydm. dMM   Icon Theme: breeze-dark
     hMM- +MMd/-------...-:sdds  dMM   Disk: 212G / 667G (33%)
     -NMm- :hNMNNNmdddddddddy/`  dMM   CPU: AMD Ryzen 9 5900HX with Radeon Graphics @ 16x 4.68GHz
      -dMNs-``-::::-------.``    dMM   GPU: AMD RENOIR (LLVM 13.0.1, DRM 3.42, 5.15.0-57-generic)
       `/dMNmy+/:-------------:/yMMM   RAM: 6880MiB / 30945MiB
          ./ydNMMMMMMMMMMMMMMMMMMMMM  
             \.MMMMMMMMMMMMMMMMMMM    
```

takve da se odradi 692 SET zahtjeva po sekundi i 1310 GET zahtjeva po sekundi - što je i očekivano jer pisanje je skuplja operacija od čitanja.

Tokom benchmarka na domaćinu su pokrenute shell naredbe koje prate opterećenje:

```shell
lljubojevic@IdeaPad-5-Pro:~$ for i in {1..10000}; do sudo cat /proc/loadavg | awk '{print $1","$2","$3}';done > cpuload.txt

lljubojevic@IdeaPad-5-Pro:~$ for i in {1..10000}; do sudo du -shc /var/lib/docker/overlay2 | grep total ; done > diskus.txt

lljubojevic@IdeaPad-5-Pro:~$ for i in {1..10000}; do sudo cat /proc/106225/smaps | grep -i pss |  awk '{Total+=$2} END {print Total/1024" MB"}' > cpu.txt;done     
```

Prva naredba iterira 10000 puta naredbu ispisa prosječnog opterećenja sustava /proc/loadavg, uzima samo prva tri parametra te ih zapisuje u datoteku.
Parametri su:

```shell
lljubojevic@IdeaPad-5-Pro:~$ sudo man proc | sed -n '/loadavg/,/^$/ p'
[sudo] password for lljubojevic:         
       /proc/loadavg
              The first three fields in this file are load average figures giving the number of jobs in the run queue (state R) or waiting for disk I/O (state D) averaged
              over 1, 5, and 15 minutes.  They are the same as the load average numbers given by uptime(1) and other programs.  The fourth field consists of  two  numbers
              separated  by  a  slash  (/).   The first of these is the number of currently runnable kernel scheduling entities (processes, threads).  The value after the
              slash is the number of kernel scheduling entities that currently exist on the system.  The fifth field is the PID of the process that was most recently cre‐
              ated on the system.
```

Druga naredba iterira 10000 puta ispis prostora na tvrdom disku kojeg zauzima direktorji u kojemu se nalaze podaci iz Docker kontejnera /var/lib/docker/overlay2, uzima samo ukupno zauzeće i zapisuje u datoteku.

Treća naredba iterira 10000 ispis zauzeća memorije redis-server procesa na master čvoru. Procesi i njihovi resursi se na GNU/Linux i UNIX sustavima spremaju kao direktorij /proc/PID/smaps. Iz toga uzimamo PSS:

> The "proportional set size" (PSS) of a process is the count of pages it has in memory

te ga alatom awk pretvaramo u MB i zapisujemo sve u datoteku.

Pokrećemo naredbu docker stats kako bi pratili opterećenje koje stvaraju kontejneri. Vidim da master čvor koristi najviše resursa - što je i očekivano s obzirom na tip replikacije koji redis koristi (asinkrona). Svi zahtjevi stižu na master čvor te se, u intervalima, asinkrono, repliciraju dalje:

```shell
sudo docker stats
CONTAINER ID   NAME                   CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O        PIDS
6f2926fcc8c9   pz_redis-secondary_3   0.19%     587.7MiB / 30.22GiB   1.90%     1.39GB / 4.48MB   8.19kB / 2.1GB   7
c915199a3522   pz_redis-secondary_1   0.18%     587.5MiB / 30.22GiB   1.90%     1.32GB / 4.36MB   69.6kB / 2.1GB   7
4810ac537405   pz_redis-master_1      248.23%   11.99GiB / 30.22GiB   39.67%    13.3MB / 4.02GB   111kB / 201GB    10
c13f0ee4b01d   pz_redis-secondary_2   0.19%     587.5MiB / 30.22GiB   1.90%     1.31GB / 4.48MB   4.1kB / 2.06GB   7
```

Kako bi se uvjerili da se repliciranje se događa u pozadini, promatrali smo izlaz (log) naredbe docker-compose:

```shell
redis-secondary_3  | 1:S 08 Jan 2023 12:22:16.624 * Trying a partial resynchronization (request 5227d7c9d2e9ea4f1f8a12ced74bbbb3f4735632:195718029565).
redis-secondary_1  | 1:S 08 Jan 2023 12:22:16.624 * Trying a partial resynchronization (request 5227d7c9d2e9ea4f1f8a12ced74bbbb3f4735632:195718029565).
redis-secondary_2  | 1:S 08 Jan 2023 12:22:16.865 * Trying a partial resynchronization (request 5227d7c9d2e9ea4f1f8a12ced74bbbb3f4735632:195718029565)
...
redis-secondary_2  | 1:S 08 Jan 2023 12:22:21.802 * Full resync from master: 5227d7c9d2e9ea4f1f8a12ced74bbbb3f4735632:226418045149
redis-secondary_1  | 1:S 08 Jan 2023 12:22:21.802 * Full resync from master: 5227d7c9d2e9ea4f1f8a12ced74bbbb3f4735632:226418045149
redis-secondary_3  | 1:S 08 Jan 2023 12:22:21.802 * Full resync from master: 5227d7c9d2e9ea4f1f8a12ced74bbbb3f4735632:226418045149
```

Repliciranje se događa u intervalima.

### Grafovi opterećenja

Dodatno smo za seminarski rad odlučili nacrtati grafove korištenja sustavskih resursa. Iz dobivenih datoteka bash for petlji dobili smo podatke oblika:

**Datoteka ram.txt**:

```text
2551 MB
2635.39 MB
2646.76 MB
2669.3 MB
2701.28 MB
2655.84 MB
2867.28 MB
2740.61 MB
...
```

**Datoteka diskus.txt**:

```text
5,2G    total
5,2G    total
5,2G    total
...
```

**Datoteka cpuload.txt**:

```text
7.50,5.85,3.83
7.50,5.85,3.83
7.46,5.87,3.85
7.46,5.87,3.85
7.50,5.91,3.87
...
```

Pomoću traži i zamijeni mogućnosti tekstualnog uređivača "Kate" mičemo "MB" i crtamo graf korištenja memorije za master redis-server proces:

```shell
gnuplot> set xlabel "master"
gnuplot> set ylabel "MB"
gnuplot> p "ram.txt" w lp
```

Zatim na isti način mičemo "G total" iz datoteke zauzeća diska te crtamo graf zauteća diska:

```shell
gnuplot> set ylabel "GB"
gnuplot> p "diskus.txt" w lp
```

Konačno, za graf prosječno opterećenja procesora procesima koji se trenutno izvode koristimo regularni izraz: "\,[1-9]\.[1-9]*" kako bi maknuli drugu i treću vrijednost te crtamo graf:

```shell
gnuplot> set ylabel "Jobs avg"
gnuplot> p "cpuload.txt" w lp
```

#### Grafovi opterećenja sustava

##### Graf opterećenja procesora

![RAM](Images/CPU-LOAD.png)

##### Graf opterećenja memorija

![RAM](Images/RAM-USAGE.png)

##### Graf opterećenja pohrane

![Disk](Images/DISK-USAGE.png)

## Opažanja o opterećenju sustava

### Jedna instanca redis-servera

#### Zauzeće procesora

Možemo primijetiti da napretkom benchmarka opterećenje procesora raste, što je i očekivano kako pristižu zahtjevi. Kada se alocira dovoljna količina procesorske moći otperećenje postaje relativno konstantno.

#### Zauzeće diska

Možemo primijetiti da količine koje se zapisuju i dohvaćaju s diska ostaju prilično konstantne prilikom izvođenja mjerenja.

#### Zauzeće memorije

Možemo primijetiti da se prilikom pokretanja benchmarka alocira 5250MB memorije, te da taj iznos memorije ostaje konstantan tokom cijelog izvođenja mjerenja.

### Redis sustav koji koristi replikaciju

#### Disk

Vidimo da zauzeće diska raste i pada, to je upravo zato što se AOF datoteka puni log zapisima operacija, ona je konfigurana tako da se - kada se zapuni - automatski reže.

To se može izvesti i ručno:

```shell
redis-check-aof --fix /PUTANJA/DO/.aof
```

ili petljom, ukoliko postoji više instanci:

```shell
for i in $(ls -1 /PUTANJA/DO/*.aof); do redis-check-aof --fix $i; done  
```

A zašto nemamo konstantan rast kako se puni baza podataka?

Redis server s vremena na vrijeme sprema sve svoje podatke na disk, no to se ne događa redovno.

Podaci se spremju u jednom od sljedećih slučajeva:

* automatski s vremena na vrijeme
* pozivom naredbe BGSAVE
* gašenjem redis server procesa

Redis uglavom radi u memoriji računala, poziva se na disk kada nema dovoljno memorije za obradu svih zahtjeva (AOF i RDB datoteke). Zauzeće diska smo pratili samo kako bi pokazali zahtjeve koji dolaze i da se njihove radnje spremaju u AOF datoteku.

##### Procesor

Vidimo da se dogodi nagli porast opterećenja na samom početku mjerenja performasni - očekivano - jer se skaliranje vrši u vremenskim intervalima - u tom trenutku dogodilo se repliciranje pa je procesor bio dodatno opterećen. Također možemo vidjeti da se opterećenje bolje raspoređuje nego kod jedne instance Redisa - odnosno vidimo da je na slabijem sklopovlju manje opterećenje.

##### Memorija

S obzirom da je odabran način rada - skaliranje - vidimo da se RAM memorija, po konstantnim uzorcima, dinamički alocira i dealocira kako se puni/prazni Redis baza podataka (jasno je da je baza podataka u RAM memoriji), a podaci prebacuju u AOF datoteku. Opterećenje RAM memorije nije konstatno te je manje nego u načinu rada s jednom instancom.

## Literatura

* [1] <https://www.lua.org/about.html/>
* [2] <https://www.digitalocean.com/community/tutorials/how-to-perform-redis-benchmark-tests/>
* [3] <https://redis.io/docs/management/replication/>
* [4] <https://redis.io/docs/management/optimization/benchmarks/>
