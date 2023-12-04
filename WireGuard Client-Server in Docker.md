# Creating a Docker-based VPN using WireGuard
Created for doc. dr. sc. [Vedran Miletić](https://vedran.miletic.net/) to create teaching materials for course [Security of Information and Communication Systems](https://group.miletic.net/hr/nastava/materijali/wireguard-virtualna-privatna-mreza/)

A [virtual private network](https://en.wikipedia.org/wiki/Virtual_private_network) (VPN) provides private network services using a public network such as the Internet. With a VPN, we can send and receive data over a public network while using a private LAN configuration. VPN uses various security mechanisms to ensure the confidentiality and authenticity of the data we send. A virtual private network can be created with a number of protocols, such as [L2TP](https://en.wikipedia.org/wiki/Layer_2_Tunneling_Protocol) on the link layer (which implements [xl2tpd](https://github.com/xelerance/xl2tpd)), [IPsec](https://en.wikipedia.org/wiki/IPsec) on the network layer (which implements [strongSwan](https://www.strongswan.org/)) and [SSTP](https://en.wikipedia.org/wiki/Secure_Socket_Tunneling_Protocol)  on the transport layer (which implements  [SoftEther VPN](https://www.softether.org/)).

WireGuard is a very simple but modern VPN server and client that uses the latest cryptographic technologies. It aims to be faster, simpler, more reliable and more useful than IPsec. It is also a serious competition to OpenVPN.

It was initially released for Linux kernel-based systems, but has now become a multi-platform solution for establishing a VPN network. The development of WireGuard is still ongoing, although it is already considered the safest and easiest to use VPN solution in the industry.

WireGuard uses state-of-the-art cryptographic technologies such as:
* [Noise protocol framework](http://www.noiseprotocol.org/), 
* [Curve25519](https://en.wikipedia.org/wiki/Curve25519), 
* [ChaCha20](https://www.cryptopp.com/wiki/ChaCha20), 
* [Poly1305](https://en.wikipedia.org/wiki/Poly1305), 
* [BLAKE2](https://www.blake2.net/), 
* [SipHash24](https://en.wikipedia.org/wiki/SipHash), 
* [HKDF](https://en.wikipedia.org/wiki/HKDF)

With the release of the Linux kernel 5.6, the version of WireGuard for the Linux platform is moving to a stable production release and is being built into the Linux kernel itself. Because the Linux kernel, and its components, are open source (GNU General Public License (GPL) version 2), by merging WireGuard with the Linux kernel, the source code becomes available to the general public.

After connecting the WireGuard client to the WireGuard server (described below), a tunnel is created through the Internet through which data is then sent. The data passing through the tunnel is encrypted so that third parties on the Internet cannot read it, and various types of compression can be used to make the data smaller.

# Setting up WireGuard VPN using Docker

## Installing Docker
### If you are not a sudo user:
Ask your administrator to install Docker with the command:
* Ubuntu-based distros:
```shell
sudo apt-get install docker docker-compose
```
* Arch-based distros:
```shell
sudo pacman -S docker docker-compose
```
Then install all Docker dependencies:
* Ubuntu-based distros:
```shell
sudo apt-get install -y dbus-user-session
sudo apt-get install -y uidmap
```
* Arch-based distros:
```shell
sudo pacman -S shadow
sudo pacman -S fuse-overlayfs
```
Zatim ugasite Docker servis naredbom:
```shell
sudo systemctl disable --now docker.service docker.socket
```
Before installing Docker alata Docker you must generate subuid and subgid.
[Subuid](https://man7.org/linux/man-pages/man5/subuid.5.html) authorizes the user ID to map the range of user IDs from its namespace to child namespaces.
[Subgid](https://man7.org/linux/man-pages/man5/subgid.5.html) authorizes the user group ID to map the range of group IDs from its namespaces to child namespaces.

To generate subuids and subgids, you need to write a Python script with the following code:
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
Then we execute the Python script and install Docker from the repository "[rootless](https://docs.docker.com/engine/security/rootless/)":
```shell
sudo python3 uid.py
curl -fsSL https://get.docker.com/rootless | sh
```

After installation, if you use systems based on Ubuntu distribution, mark variables and functions that are passed to subordinate processes ("export"):
```shell
export PATH=/home/lljubojevic/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
```
Set the appropriate access permissions for your Docker user with the command:
```shell
sudo chmod 666 /var/run/docker.sock
```
Finally, to configure the WireGuard client server, execute port traffic acceptance commands that the WireGuard server will use:
```shell
sudo ufw allow 51820
sudo iptables -I INPUT -p tcp -m tcp --dport 51820 -j ACCEPT
```
Then, start Docker as a rootless user:
```shell
systemctl --user enable docker
systemctl --user start docker
```
### If you are a sudo user:
Install Docker this way:
* Ubuntu-based distros:
```shell
sudo apt-get install docker docker-compose
```
* Arch-based distros:
```shell
sudo pacman -S docker docker-compose
```
Start Docker service using:
```shell
sudo systemctl enable --now docker.service docker.socket
```
Additionally, if you want to start Docker at startup, run the command:
* Ubuntu-based distros:
```shell
sudo loginctl enable-linger $(whoami)
```
* Arch-based distros:
```shell
sudo loginctl enable-linger (whoami)
```
## WireGuard VPN Server Setup
Assuming you have administrator privileges, to configure the WireGuard client server, execute port traffic acceptance commands that the WireGuard server will use:
```shell
sudo ufw allow 51820
sudo iptables -I INPUT -p tcp -m tcp --dport 51820 -j ACCEPT
```
Make a directory:
```shell
mkdir wireguard-server
```
In that directory create a "docker-compose.yaml" file:
```shell
nano docker-compose.yaml
```
**docker-compose.yaml for server**:
```shell
version: "2.1"
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard-server
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Zagreb
      - SERVERURL=10.13.13.1
      - SERVERPORT=51820
      - PEERS=1
      - PEERDNS=auto
    volumes:
      - /home/lljubojevic/wireguard-server/config:/config
      - /home/lljubojevic/wireguard-server/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv6.conf.all.disable_ipv6=0
    restart: unless-stopped
```
Then execute the creation of a Docker WireGuard VPN server container:
```
docker-compose up -d
```
**Does it work?:**
If the command:
```shell
docker logs wireguard-server
```
shows output as:
```shell
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-envfile: executing... 
[cont-init.d] 01-envfile: exited 0.
[cont-init.d] 01-migrations: executing... 
[migrations] started
[migrations] no migrations found
[cont-init.d] 01-migrations: exited 0.
[cont-init.d] 02-tamper-check: executing... 
[cont-init.d] 02-tamper-check: exited 0.
[cont-init.d] 10-adduser: executing... 

-------------------------------------
_         ()
| |  ___   _    __
| | / __| | |  /  \ 
| | \__ \ | | | () |
|_| |___/ |_|  \__/


Brought to you by linuxserver.io
-------------------------------------

To support the app dev(s) visit:
WireGuard: https://www.wireguard.com/donations/

To support LSIO projects visit:
https://www.linuxserver.io/donate/
-------------------------------------
GID/UID
-------------------------------------

User uid:    1000
User gid:    1000
-------------------------------------

[cont-init.d] 10-adduser: exited 0.
[cont-init.d] 30-module: executing... 
Uname info: Linux 12320232623e 5.16.10-zen1-1-zen #1 ZEN SMP PREEMPT Wed, 16 Feb 2022 19:35:21 +0000 x86_64 x86_64 x86_64 GNU/Linux
**** It seems the wireguard module is already active. Skipping kernel header install and module compilation. ****
[cont-init.d] 30-module: exited 0.
[cont-init.d] 40-confs: executing... 
**** Server mode is selected ****
**** External server address is set to 10.13.13.1 ****
**** External server port is set to 51820. Make sure that port is properly forwarded to port 51820 inside this container ****
**** Internal subnet is set to 10.13.13.0 ****
**** AllowedIPs for peers 0.0.0.0/0, ::/0 ****
**** PEERDNS var is either not set or is set to "auto", setting peer DNS to 10.13.13.1 to use wireguard docker host's DNS. ****
**** Server mode is selected ****
**** Server related environment variables changed, regenerating 1 server and 1 peer/client confs ****
PEER 1 QR code:
█████████████████████████████████████████████████████████████████
█████████████████████████████████████████████████████████████████
████ ▄▄▄▄▄ █▀▀█▄█ ▄▄ ▄██ ▄▀▀▄▄▄ ▄█ ▄▄▄▄▄▀▄ █▄ ▄█▄▀▄ ██ ▄▄▄▄▄ ████
████ █   █ █▄█ ▀▀▄███ ▄ █▄▄█▄▀▄█▄▀ ▄▄█ █▄▀▀▄▄▀▀▄▄▀▄ ██ █   █ ████
████ █▄▄▄█ █ █▄▀▄▄████ ▀██▀▀▄  ▄▄▄ ▀ ▄██▄▀▄ █  ▀▀▄▀▄██ █▄▄▄█ ████
████▄▄▄▄▄▄▄█ ▀ █ █ ▀ ▀ ▀ ▀ ▀▄█ █▄█ ▀▄▀▄█▄█ ▀▄█▄▀ ▀ ▀▄█▄▄▄▄▄▄▄████
████  ▀▄█▀▄ ▀▄█  ▄▀██▀▄▀█  ▄▄▀▄ ▄▄▄▀█▀▄▄ ▀█▀█ █▀▄█▀▄▄█▄  ▀  ▀████
████▀████▀▄  █ ▄█▀█ ▀▀▄█▄  ▀▀   ▀▀█▄█ █  ▀▀▄▀▄█   ▀▄▄ ▄ ▀ █  ████
██████▀   ▄█▄▀▀ ▄▄██▀█▄▄▄ ▀   ▀█ ▀█ ▄ ▀ ▄█▄ ▄ █▄▄█▀▀▄ ▄ ▄█▄█▄████
████ ▄    ▄ █▄▀▀█ █▀▄█▄██▄ ███▄███▄█ ▀▄▀▀  ▀▄ ██ █ █ █ ▀ ▄███████
████  ▄▄▀▀▄▀ ▄ ▀▄ ▄██▄▄▄▀▄▀ ██ █ ▀▄▄ █▄▄▄█  ▀ █  ▄▄██▄  ▄█▀██████
████▄▀▀▀█▀▄ ▄ ▄▀▀▀▄█ ▀▄███ ▄███▄ ▀▀▄ ▄▀▀▄ █▀▀ ▄▀ ███▀▀█▀▄███▄████
█████ █▀█ ▄██▀█ ████ ▀▄ █▄▄▄ ▄▄   ▄ ▀▄█ ▄ █▄▄██▄█ ▄ ██▄ ▄▀▄██████
████▀ ▄ ▀▀▄ █▀ ▄▄  █  ▄▀ █▄▀▀▀ ▀▄ ▀█ ▄█ ▄▀█ ▀▀▄▀▄█▄▄ ▄█  ▄▀█ ████
████▀▄ ▄██▄ ███▀▀▄██▀  ▄█▄ █▄▀  ▄ █▀▀ ▀▀▄▄ ▀▄▀█▀▄▀ ██▀█▄   ▄ ████
████ ▄▄█ ▄▄▄ ▄▄ ▀▄ ▄▀▀▀ ██▄  █ ▄▄▄ ▄▀   █ █▀ ▀█▄▀▀▄  ▄▄▄ █▄██████
████▄ ▄▄ █▄█ ▄ ▀ ▀█▄█▄█▄▄██▄▄  █▄█ ▄ █▄▄ ▄▄▄▀█▄█▄▄█▄ █▄█ ▀▀█▀████
████▄▀ ▄▄▄▄▄ ▄ ▀▀██ ██████ ▀▄▄▄ ▄▄ ▀▀▄ ████▀▀▄█  █▀   ▄▄ ▄█▄ ████
████ ▀██▄ ▄ ▀▄▄▀▀▄ ▄█▀  ▄▀ █▄ ▀▄█ ▄▀█▀▄▀ ▄▄█▄▄▄ ▀▀█ ▀█▀▄█▀██▀████
████ ▀█▄ █▄▄▄ ▀█▀███▀▄██▀▀█▀▀▀ ▄█▀▄▄▄▀▄▄▀█▀▄▄▄▄ █ ▄█▄▀ ▄█▀▄▄ ████
████▀ ▄██ ▄█  ▄ ▀▄▄  ▀▄▀ █▄ ▄▄▀  ▄▄ ▄▄ ▀▄▀▄████▄ █   █  ▀▄█▀▄████
█████▀▄▀▄▄▄▄▀▄█▀▀█  ▄▀▀█  ▄▀▄▀▄▄ ▀▄█▄ ▀█▄ ▄▄ ██▀█▀██▀ ▄▀▄█▄▀█████
████▀▄▄█ █▄██▄ ▄ ▄ █▄▄ █  ▀ ▄  ███   ▀██▄  ▀ ▄▀▄▄▀▄█▀▄ █▄▀ ▄▄████
████ ▄▀███▄▀  ▄█▀ █▀ █▄█  █▀ █▄▀▀█ ▀█  ▀   ▀▄▄▀ ██ ██▄ ▀▀███▄████
████▀█▄▀▀▀▄▄ ▄▀ █  ▄▄▄▄█▄ █ █▀█▀▄ █ ██▄▀ ███ ▀▄ ▀█▄ █▀ █▄▀██▀████
████ ▀ ▀▀▄▄▀  ▀▀▀▀▄▀ ▄▄▄▄█   ▀▄▀█▀  █ ▄█▀▄▀█▀█▀ ▄▀▄  ▀ ▄█ ██ ████
██████████▄█ ▀▄█   █ ▄▄▀█▄ █▄█ ▄▄▄ ▀▄▀ ▄▄▀▄█ █▄▄ ▄▀█ ▄▄▄ ▄█ ▄████
████ ▄▄▄▄▄ █▄   ▄█▀▄▄ ▀█▄▀▄  ▀ █▄█    █▀▄▀▄██▀ ▄▀█▀▀ █▄█ █▄ ▀████
████ █   █ █▀▄█▀ ▀▄█▀█▄▀   ▀ ▄▄    ▀▄▀█▄▀█ ▀█▀█▀ █▄█  ▄  ██ ▀████
████ █▄▄▄█ █▀█▀▄▀▄███ ▄▄▄█ █▀▄▀▄▄ ▄▀▄▄▀█▄ ▀█▄▀ ▄▄  ▀▄ █  ▄▄█▄████
████▄▄▄▄▄▄▄█▄█▄▄▄▄▄▄██▄▄▄▄▄███████▄██▄█▄██▄███████▄█▄█▄▄▄█▄▄█████
█████████████████████████████████████████████████████████████████
█████████████████████████████████████████████████████████████████
[cont-init.d] 40-confs: exited 0.
[cont-init.d] 90-custom-folders: executing...
[cont-init.d] 90-custom-folders: exited 0.
[cont-init.d] 99-custom-scripts: executing...
[custom-init] no custom files found exiting...
[cont-init.d] 99-custom-scripts: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.13.13.1 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] ip -4 route add 10.13.13.2/32 dev wg0
[#] iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
.:53
CoreDNS-1.9.0
linux/amd64, go1.17.6, ace3dcb
```
It works!

## WireGuard VPN Client Setup
Make a directory:
```shell
mkdir wireguard-klijent
```
In that directory create the "docker-compose.yaml" file:
```shell
nano docker-compose.yaml
```
**docker-compose.yaml for client**:
```shell
version: '3.7'
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard-klijent
    restart: unless-stopped
    networks:
      - backbone
    volumes:
      - './wireguard:/config'
      - '/lib/modules:/lib/modules:ro'
    environment:
      - PUID=1000
      - PGID=1000
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv6.conf.all.disable_ipv6=0
       
networks:
  backbone:
    driver: bridge
```
It is necessary to generate a docker container for the client, which is not currently functional, in order for a creation of the configuration directory:
```shell
docker-compose up -d
```
Then, after launching the container, it is necessary to create a WireGuard network interface through which the client communicates with the server:
```shell
docker exec -it wireguard-klijent ip link add dev wg0 type wireguard
```
When the server is generated, a "peer" configuration file called "peerX.conf" will also be created. The contents of this file need to be overwritten in the wg0.conf file, which will be located in the wireguard-client directory.
Let's do that:
```shell
cp ./wireguard-server/config/peer/peer1.conf ./wireguard-klijent/wireguard/wg0.conf
```
Change the wg0.conf file as following:
```shell
[Interface]
Address = 10.13.13.2
PrivateKey = 8NWt7q+Vg2f5IdULtIZQ06osuEfNZI4aOf3fjp0CTXY=
ListenPort = 51820
DNS = 10.13.13.1

[Peer]
PublicKey = nRWxNsBzC+bre4qzjLsWQp8Y9vo82JYF7N8XNLiTaxQ=
Endpoint = 10.13.13.1:51820
AllowedIPs = 0.0.0.0/0, ::/0
```

Notes:
* If you set a parameter from the server configuration to a number greater than one (PEERS = n) n directories for nodes and configurations for them will be created, the instructions assume that you have only one client node.
* The wg0.conf file may vary.
* 
## Testing the Client-Server VPN architecture:
```shell
docker exec -it wireguard-server curl -w "\n" ifconfig.me
46.188.154.181

docker exec -it wireguard-klijent curl -w "\n" ifconfig.me
46.188.154.181
```
If the IP addresses are the same, the configuration is valid.
The IP address displayed is your current (or permanent) IP address by which you access the Internet.

# Explanation of configuration parameters in docker-compose.yaml file
Server side must contain parameters:
* version: "2.1" - the required version of the docker-compose tool
* services: - services started by the container
* wireguard: - service name
* image: - the requested image that the Docker tool retrieves from its repository, and container_name: - arbitrary container name.
* cap_add: - gives the container elevated permissions on the host computer and allows it to interact with the host's network interfaces.
* environment: - environment settings inside the Docker container
    * PUID and PGID - variables that define the user and group
    * TZ = Europe / Zagreb - container time zone, needs to be defined
    * SERVERURL - server domain name (can be empty, defined by the public IP address of the server or set to automatic configuration)
    * SERVERPORT - the door on which the server opens
    * PEERS - number of nodes in the VPN network (can be number or type name: laptop, tablet, phone)
    * PEERDNS - when set to auto, the container adjusts the DNS settings itself
* volumes: - directories on the host that the container may use
* ports: - the door that the container may use
* sysctls: - parameters that allow the container to communicate with networks externally
* restart: - defines in which cases the container may be restarted.

# Sources
[Pedrolamas](https://www.pedrolamas.com/2020/11/20/how-to-connect-to-a-wireguard-vpn-server-from-a-docker-container/)
[Markontech](https://markontech.com/linux/install-wireguard-vpn-server-with-docker/)
[Spad.uk](https://spad.uk/wireguard-as-a-vpn-client-in-docker-using-pia/)
