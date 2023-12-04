# DIO POTREBAN ZA POKRETANJE DOCKER-A "ROOTLESS":
[izvor](https://docs.docker.com/engine/security/rootless/)

## Dependencies:
**Instaliran Docker putem sudo apt-get install docker docker-compose ili pacman -S docker docker-compose**

**ZA ARCHLINUX:**
```shell
sudo pacman -S shadow
sudo pacman -S fuse-overlayfs
```

**ZA UBUNTU**
```shell
sudo apt-get install -y dbus-user-session
sudo apt-get install -y uidmap
```

## Postupak:
Prvo ugasiti Docker servis:
```shell
sudo systemctl disable --now docker.service docker.socket
```

Potreno je napraviti py skriptu za generiranje subuid i subgid-a:
```python
f = open("/etc/subuid", "w")
for uid in range(1000, 65536):
    f.write("%d:%d:65536\n" %(uid,uid*65536))
f.close()

f = open("/etc/subgid", "w")
for uid in range(1000, 65536):
    f.write("%d:%d:65536\n" %(uid,uid*65536))
f.close()
```

Onda:
```
sudo python3 uid.py
curl -fsSL https://get.docker.com/rootless | sh
```

**Ovo će raditi samo na Ubuntuu**
```
export PATH=/home/lljubojevic/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
```
Napomena, radilo je i bez export-a na archlinux ali stoji u uputama.

Pokretanje Dockera:
```shell
sudo systemctl enable --now docker.service docker.socket
sudo chmod 666 /var/run/docker.sock

ljubojevic@Omen17 in ~ via  v3.10.2 took 4ms
λ systemctl --user start docker

lljubojevic@Omen17 in ~ via  v3.10.2 took 5ms
λ systemctl --user enable docker

sudo loginctl enable-linger $(whoami) //za ubuntu

lljubojevic@Omen17 in ~ via  v3.10.2 took 98ms
λ sudo loginctl enable-linger (whoami) // za arch
```

## TEST WIREGURAD IMAGE:
```
mkdir wireguard-server1
cd wireguard-server1
nano docker-compose.yaml
```

Sadržaj docker-compose.yaml:
```
version: "2.1"
services:
wireguard:
image: linuxserver/wireguard
container_name: wireguard1
cap_add:
- NET_ADMIN
- SYS_MODULE
environment:
- PUID=1000
- PGID=1000
- TZ=Europe/Zagreb
- SERVERPORT=51820
volumes:
- /home/lljubojevic/wireguard-server1/config:/config
- /lib/modules:/lib/modules
ports:
- 51820:51820/udp
sysctls:
- net.ipv4.conf.all.src_valid_mark=1
restart: unless-stopped
```

Kako testirati radi li?
>The first command will retrieve your real Public IP, matching the one your ISP has provided you with.
The second command will do the same but from inside the Wireguard Docker container, and it should match the connected WireGuard VPN Server IP."

```shell
lljubojevic@Omen17 in ~/wireguard-server2🔒 as 🧙 took 62ms
λ curl -w "\n" ifconfig.me
46.188.154.181


lljubojevic@Omen17 in ~/wireguard-server2🔒 as 🧙 took 26ms
[🔴] × docker exec -it wireguard1 curl -w "\n" ifconfig.me
46.188.154.181
```
Ako je isti IP, radi.