# Managing Computer Systems
## Literature
* Essential System administration O'Reilly,2002 - Frisch, Ae.
* Unix and Linux system administration handbook - Nemeth, E. Addison 2017
* Doleželová, M., Muehlfeld, M., Svistunov, M., Wadeley, S., Čapek, T., Hradílek, J., Silas, D., Heves, J., Kovář, P., Ondrejka, P., Bokoč, P., Prpič, M., Slobodová, E., Kopalová, E., Svoboda, M., O'Brien, D., Hideo, M., Domingo, D. & Ha, J. System administrator's guide. (Red Hat, 2018)
* Aoki, O. Debian reference. (Debian, 2018).
* The FreeBSD documentation project. FreeBSD handbook. (FreeBSD, 2018.)
* How Linux Works, 3rd Edition: What Every Superuser Should Know 3rd Edition, Brian Ward
* [How to properly document software?](https://www.doxygen.nl/manual/index.htm)
* [How to properly document systems?](https://group.miletic.net/hr/nastava/materijali/upravljanje-dokumentacijom-racunalnih-sustava-i-mreza/)
    * MkDocs - Markdown or VS Code with markdownlint extension    

**.markdownlint.json example:**
```json
{
 "MD007" : { "indent": 4 },
 "MD014" : false
}
```

## MD document example:
e-Mail server is configured on IP: 10.27.45.11.
```python
print("hello")
```

URL's and lists:
+ [www.miletic.net](https://www.miletic.net/) ili <https://www.miletic.net>
+ a
+ b
    + a
    + b
        1. third
        1. one more

Code:
```shell
$ ls
```
Graph (Graphviz):
```graphviz
a -> b;
```
Quoting:
> Registering and Subscribing Your System
    Register your system:
    ~]# `subscription-manager register`
    The command will prompt you to enter your Red Hat Customer Portal user name
    and password.
    Determine the pool ID of a subscription that you require:
    ~]# `subscription-manager list --available`
    This command displays all available subscriptions for your Red Hat account.
    For every subscription, various characteristics are displayed, including the
    pool ID.
    Attach the appropriate subscription to your system by replacing pool_id with
    the pool ID determined in the previous step:
    ~]# `subscription-manager attach --pool=pool_id`
For more information on registration of your system and attachment of the Red
Hat Content Delivery Network subscriptions, see Chapter 7, Registering the
System and Managing Subscriptions.

+ [source](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-getting_started#sec-Networking-Cockpit)


Single quotes about everything that appears on the screen!
MD can be versioned on GitHub!

Good practice for System Administrators is to use Password Managers e.g. KeepassXC while ensuring that the password length is 12 or more different characters (or let the Password Manager auto generate the password).

## Graph
VS Code extension: Graphviz Markdown Preview v0.0.8 or Mermaid

# Cloud-config
[ArchLinux VM Img Cloud-init ready](https://gitlab.archlinux.org/archlinux/arch-boxes/-/jobs/50048/artifacts/file/output/Arch-Linux-x86_64-cloudimg-20220310.50048.qcow2)
[SSH](https://www.ssh.com/academy/ssh/keygen)
[QEMU](https://wiki.archlinux.org/title/QEMU#Network)
[Cloud-Init](https://wiki.archlinux.org/title/Cloud-init)

## Cloud-config for a virtual machine:
## What is a virtual machine?
According to vmware documentation:
>A Virtual Machine (VM) is a compute resource that uses software instead of a physical computer to run programs and deploy apps. One or more virtual “guest” machines run on a physical “host” machine.

According to Wikipedia:
>Virtual machine (VM) is the virtualization/emulation of a computer system. Virtual machines are based on computer architectures and provide functionality of a physical computer. Their implementations may involve specialized hardware, software, or a combination. 
## How to create and setup a virtual machine?
**1. Generate SSH key:**
```shell
ssh-keygen -t rsa -b 4096 ssh-keygen -t dsa 
ssh-keygen -t ecdsa -b 521
```
**2. Create user-data and meta-data files:**

**Basic user-data with SSH key authentication:**
```shell
#cloud-config
users:
  - name: fidit
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZWfY7BinoB43gEXhlyvd97Yo9tKHpmx/+J0dkdyVSgX3fFlmMBpfulc7WsZ7tw3BOCgFPVHlX7awGylDmff3HE2+CntQuV0vwhqJ28yRu3LvxPm6Q4wbk+lNBO7qOg8wDroE0z0ikNg+DgqqJLXhPxX0R1lawZ0Vf+cL+M56roCMZl0CZy7TuZFcBCpEb/F5ipGQehVQVuixkA7G45+R5i7x8rZbi7wztS7oYzsgNKO/z66w4xVujLf2eMaIYEL2p7/CGc3U+nNULEUTwdJ6NsBmb2lv5HfRPrmmvIxJTSNMBEdVd2fhikFkfpzGyuKwy7p8iNRkQICURC+qEOEi4HYZvYtLsSeee0GQ5P0jTMp2Ya5vVhrz0fVn4byW8esOob7eiUX3E75iwtzUUmNLN8cuCZOhSTUlcqqJMYA701CJegWNDQw0f7lKvtvwFcuscK0+g80x8rXTngiLNTCXwVAFgSoRtI7RlQJcMOo9Z91P/NT3adC6I2G4Btmgoe+0= korisnik@ODJ-O365-117

```

**Basic user-data with SSH key authentication for root and specific user:**
```shell
#cloud-config
users:
  - name: myuser
    ssh_authorized_keys:
    - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZWfY7BinoB43gEXhlyvd97Yo9tKHpmx/+J0dkdyVSgX3fFlmMBpfulc7WsZ7tw3BOCgFPVHlX7awGylDmff3HE2+CntQuV0vwhqJ28yRu3LvxPm6Q4wbk+lNBO7qOg8wDroE0z0ikNg+DgqqJLXhPxX0R1lawZ0Vf+cL+M56roCMZl0CZy7TuZFcBCpEb/F5ipGQehVQVuixkA7G45+R5i7x8rZbi7wztS7oYzsgNKO/z66w4xVujLf2eMaIYEL2p7/CGc3U+nNULEUTwdJ6NsBmb2lv5HfRPrmmvIxJTSNMBEdVd2fhikFkfpzGyuKwy7p8iNRkQICURC+qEOEi4HYZvYtLsSeee0GQ5P0jTMp2Ya5vVhrz0fVn4byW8esOob7eiUX3E75iwtzUUmNLN8cuCZOhSTUlcqqJMYA701CJegWNDQw0f7lKvtvwFcuscK0+g80x8rXTngiLNTCXwVAFgSoRtI7RlQJcMOo9Z91P/NT3adC6I2G4Btmgoe+0= korisnik@ODJ-O365-117
  - name: root
    ssh_authorized_keys:
    - ecdsa-sha2-nistp521 AAAAE2VjZHNhLXNoYTItbmlzdHA1MjEAAAAIbmlzdHA1MjEAAACFBAA5J/SRFcYnps9vCQQwnsXquEeKt9R7/7Ab6YovEoU5nE1vUSFHXoBNPhIXKOJU8G2pGi9nMFS27z9B7+oyLPTQhwFFbUOoG+/jTrc4CHy+b2rdx7x1CbR4fQSsgC/t0g+z05o9jar/t0Lea6/6hIAM2uGh+KGiKzGT/aug2VBU3SvcbQ== korisnik@ODJ-O365-117

```
[What else can you do with cloud-config?](https://cloudinit.readthedocs.io/en/latest/topics/examples.html#)
**3. Generate cloud-init.iso:**
```shell
xorriso -as genisoimage -output cloud-init.iso -volid CIDATA -joliet -rock user-data meta-data
```
**4. Create a virtual machine:**
4.1. Expand ArchLinux disk image if you need more space:
```shell
qemu-img resize aarch.qcow2 +10G
```
4.2. Create a virtual machine
```shell
qemu-system-x86_64 -m 512M aarch.qcow2 -cdrom cloud-init.iso -nic user,hostfwd=tcp::60022-:22
```
4.3. Add port for acessing the virtual machine from host PC:
```shell
ssh-keygen -f "/home/korisnik/.ssh/known_hosts" -R "[localhost]:60022"
```
**5. Connect to the virtual machine:**
```shell
ssh -v -p 60022 fidit@localhost
ssh -v -p 60022 root@localhost
```
## Some basic networking tasks after setting up a virtual machine or server enviroment:
**Check network connection**
1. Find your network interface name
```shell
[test@test-PC ~]$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether e0:d5:5e:44:69:77 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.17/24 brd 192.168.0.255 scope global dynamic enp7s0
       valid_lft 2613sec preferred_lft 2613sec
    inet6 2a05:4f44:111c:8900::18/128 scope global dynamic noprefixroute 
       valid_lft 410132sec preferred_lft 341012sec
    inet6 2a05:4f44:111c:8900:50bd:bfcb:7176:40fb/64 scope global dynamic noprefixroute 
       valid_lft 414713sec preferred_lft 345593sec
    inet6 2a05:4f44:111c:8900:e2d5:5eff:fe44:6977/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 414711sec preferred_lft 345591sec
    inet6 fe80::645f:28c4:cfb8:75e3/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
Network interfaces are listed by numbers 1:, 2:, 3:, etc. 
* In a virtual machine it's named mostly eth0
* In a PC it's named mostly enpXsY
2. View network interface status and data about the network connection
```shell
[lljubojevic@Ljubojevic-PC ~]$ networkctl status enp7s0
● 2: enp7s0                                                                                                 
                     Link File: /usr/lib/systemd/network/99-default.link
                  Network File: /etc/systemd/network/20-ethernet.network
                          Type: ether
                         State: routable (configured)
                  Online state: online                                                                      
                          Path: pci-0000:07:00.0
                        Driver: r8169
                        Vendor: Realtek Semiconductor Co., Ltd.
                         Model: RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (Onboard Ethernet)
              Hardware Address: XYZ
                           MTU: 1500 (min: 68, max: 9194)
                         QDisc: fq_codel
  IPv6 Address Generation Mode: none
          Queue Length (Tx/Rx): 1/1
              Auto negotiation: yes
                         Speed: 1Gbps
                        Duplex: full
                          Port: tp
                       Address: XYZ
                       Gateway: XYZ
                           DNS: XYZ
...
```

**Hostnamectl**
According to [Linuxhint](https://linuxhint.com/set-hostname-using-hostnamectl-command/):
>The hostname is an identity of the system and is used by the networks to search the system.
>The “hostnamectl” is a Linux command that is used to set the hostname in the terminal without even opening and editing in the etc/hostname file of a system.
>Using the “hostnamectl” command, the user can edit the static, pretty, and transient hostname as well.

Output of hostnamectl:
```shell
[test@test-PC ~]$ hostnamectl
 Static hostname: test-PC
       Icon name: computer-desktop
         Chassis: desktop 🖥
      Machine ID: 801cb2df0c5647d1835be225e0073015
         Boot ID: 4fe8134f95d041af8dc53ab247179c28
Operating System: Arch Linux                      
          Kernel: Linux 5.17.4-arch1-1
    Architecture: x86-64
 Hardware Vendor: Gigabyte Technology Co., Ltd.
  Hardware Model: Z370P D3
```

Hostnamectl displays:
* Computer name (hostname)
* Icon name is the machine identifying name according to XDG Icon Naming Specification.
* What type of system it is? (Desktop, laptop, workstation etc.)
* Machine and Boot ID, OS and Kernel version
* CPU Architecture 
* Motherboard vendor and model
* Deployment is the enviroment in which is the system used e.g.  "development", "integration", "staging", "production"...
* Location - useful when managing multiple servers/VM's

It can be used for setting up:

* Hostname:
```shell
[root@archlinux ~]# hostnamectl set-hostname myServer1
[root@archlinux ~]# hostnamectl hostname 
myServer1
```
* Icon-name
```shell
[root@archlinux ~]# hostnamectl set-icon-name computer
[root@archlinux ~]# hostnamectl icon-name
computer
[root@archlinux ~]# hostnamectl set-icon-name workstation
[root@archlinux ~]# hostnamectl icon-name
workstation
[root@archlinux ~]# hostnamectl set-icon-name chassis-laptop
[root@archlinux ~]# hostnamectl icon-name
chassis-laptop
[root@archlinux ~]# hostnamectl set-icon-name chassis-watch
[root@archlinux ~]# hostnamectl icon-name
chassis-watch
[root@archlinux ~]# hostnamectl set-icon-name embedded
[root@archlinux ~]# hostnamectl icon-name
embedded
```
* Deployment
```shell
[root@archlinux ~]# hostnamectl deployment staging
[root@archlinux ~]# hostnamectl deployment
staging
```
* Location
```shell
[root@archlinux ~]# hostnamectl location "Berlin, room 7, rack 6"
```
End result?
```shell
[root@archlinux ~]# hostnamectl
 Static hostname: myServer1
       Icon name: embedded
         Chassis: vm 🖴
      Deployment: staging
        Location: Berlin, room 7, rack 6
      Machine ID: 168fc71dc2b143fab51a45ad7b066d1b
         Boot ID: 5dd62398ce6a4a859179e8f2b25911bf
  Virtualization: qemu
Operating System: Arch Linux                       
          Kernel: Linux 5.16.13-arch1-1
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
```
[more at](https://www.freedesktop.org/software/systemd/man/hostnamectl.html)

**DNS Settings?**
>resolvectl may be used to resolve domain names, IPv4 and IPv6 addresses, DNS resource records and services with the systemd-resolved.service(8) resolver service. By default, the specified list of parameters will be resolved as hostnames, retrieving their IPv4 and IPv6 addresses. If the parameters specified are formatted as IPv4 or IPv6 operation the reverse operation is done, and a hostname is retrieved for the specified addresses.

Use resolvectl command to edit those settings, e.g.:
```shell
[test@test-PC ~]$ resolvectl help
resolvectl [OPTIONS...] COMMAND ...

Send control commands to the network name resolution manager, or
resolve domain names, IPv4 and IPv6 addresses, DNS records, and services.

Commands:
  query HOSTNAME|ADDRESS...    Resolve domain names, IPv4 and IPv6 addresses
  service [[NAME] TYPE] DOMAIN Resolve service (SRV)
  openpgp EMAIL@DOMAIN...      Query OpenPGP public key
  tlsa DOMAIN[:PORT]...        Query TLS public key
  status [LINK...]             Show link and server status
  statistics                   Show resolver statistics
  reset-statistics             Reset resolver statistics
  flush-caches                 Flush all local DNS caches
  reset-server-features        Forget learnt DNS server feature levels
  dns [LINK [SERVER...]]       Get/set per-interface DNS server address
  domain [LINK [DOMAIN...]]    Get/set per-interface search domain
  default-route [LINK [BOOL]]  Get/set per-interface default route flag
  llmnr [LINK [MODE]]          Get/set per-interface LLMNR mode
  mdns [LINK [MODE]]           Get/set per-interface MulticastDNS mode
  dnsovertls [LINK [MODE]]     Get/set per-interface DNS-over-TLS mode
  dnssec [LINK [MODE]]         Get/set per-interface DNSSEC mode
  nta [LINK [DOMAIN...]]       Get/set per-interface DNSSEC NTA
  revert LINK                  Revert per-interface configuration
  log-level [LEVEL]            Get/set logging threshold for systemd-resolved
...
```

## Regional settings
Some services that run on servers depend on time zone settings to work correctly. 
It's important, when planing the virtual or server environment to set it up properly.

**localectl**
>localectl - Control the system locale and keyboard layout settings

[Full list of commands](https://man.archlinux.org/man/core/systemd/localectl.1.en#COMMANDS)

**How to set it up?**
1. Check current regional settings
```shell
[test@test-PC ~]$ localectl status
   System Locale: LANG=hr_HR.UTF-8
       VC Keymap: croat
      X11 Layout: hr
```
2. List avaliable locales and keymaps - keyboard layouts      
```shell
[test@test-PC ~]$ localectl list-locales
en_US.UTF-8
hr_HR.UTF-8

[test@test-PC ~]$ localectl list-keymaps
ANSI-dvorak
adnw
amiga-de
amiga-us
apple-a1048-sv
apple-a1243-sv
apple-a1243-sv-fn-reverse
apple-internal-0x0253-sv
apple-internal-0x0253-sv-fn-reverse
applkey
atari-de
atari-se
...
```
3. Set the needed locale configuration
```shell
[test@test-PC ~]$ localectl set-locale hr_HR.UTF-8
[test@test-PC ~]$ localectl set-x11-keymap hr
[test@test-PC ~]$ setxkbmap hr
```
4. Generate the locale config file
```shell
[test@test-PC ~]$ locale-gen
```
**What about timezone?**
Use timedatectl command to set it:
```shell
timedatectl set-timezone Europe/Zagreb
```

Also note, Regional setting can be defined in cloud-init configuration files.

## Managing users and permissions
Let's try managing users in a VM created using GUI tool -> virt-manager.

Requirements:
* Virt-manager and ArchLinux VM
* Cloud-init:

**user data:**
```shell
#cloud-config
users:
  - default

system_info:
   default_user:
     name: myuser
     plain_text_passwd: '1234'
     gecos: arch Cloud User
     groups: [wheel, adm]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
     lock_passwd: False
     
```
**Note: Cloud-init is sensitive with tab spaces and requires a empty line at the end of file**

Make cloud-init.iso file:
```shell
xorriso -as genisoimage -output cloud-init.iso -volid CIDATA -joliet -rock user-data meta-data
```
Copy downloaded Arch system image (qcow2) in home directory and rename it to aarch.qcow2

Add more space to image:
```shell
qemu-img resize aarch.qcow2 +10G
```
Then in virt-manager:

* Use existing image -> aarch.qcow2 -> archlinux -> 1GB/2 cores -> Customize configuration before install -> Add cloud-init.iso and make it a CDROM type

Then start ArchLinx VM and login with SSH from terminal on host to it:
```
ssh myuser@IPadressOfVM
```
Be careful when configuring users -> detect which user has permissions and what permissions does he have.
* ! in /etc/shadow file mean that the user is forbiden to login into the system
* User has permission to access ports from 1024, system uses ports to 1024.
* kvm group in etc/groups exists for using nested virtualization
* /dev/vd* - Linux way of mapping path to hard drives -> user dosen't have acess to disk, but has to filesystem!

Working with users:
>In Unix-like operating systems, the chmod command sets the permissions of files or directories.

Syntax:
```shell
chmod ugo -rwx
```
Where:
* ugo specifies a user, group of users or others
* -rwx specifies allowed operations: read, write, execute
>On Unix-like operating systems, the chown command changes ownership of files and directories in a filesystem.

Syntax:
```shell
chown parameters user
```
Removing ownership:
```shell
[myuser@archlinux ~]$ chmod -R go-r ftp ftp/ 
[myuser@archlinux ~]$ chmod -R go-- ftp ftp/ 
```
Where:
* go-r removes reading permissions
* go-- removes all permisions

>On Unix-like operating systems, the usermod command modifies a user account.

Syntax:
```shell
[myuser@archlinux ~]$ sudo usermod -aG video myuser
```
* Adds user "myuser" to group "video"
```shell
[myuser@archlinux ~]$ grep video /etc/group
video:x:985:myuser
```
>On Unix-like operating systems, the useradd command creates a new user or sets the default information for new users.

Syntax:
```shell
useradd user -c "Basic User" -s /bin/bash -m
```
* Where "Basic User" is description of the user, -s which shell will he use, -m creates a home directory for him.
To add password for user "user" use:
```shell
passwd user
```
To block user loging into the system use:
```shell
usermod -s /bin/false user
```
>/bin/false does nothing and it just exits with a status code indicating failure when a user attempts to login to the machine.

To view all devices attached to system use:
```shell
ls -la /dev
```
To view all users in the system use:
```shell
[test@test-PC ~]$ userdbctl
NAME                   DISPOSITION   UID   GID REALNAME                       HOME              SHELL             
root                   intrinsic       0     0 -                              /root             /usr/bin/zsh
nobody                 intrinsic   65534 65534 Nobody                         /                 /usr/bin/nologin
bin                    system          1     1 -                              /                 /usr/bin/nologin
daemon                 system          2     2 -                              /                 /usr/bin/nologin
mail                   system          8    12 -                              /var/spool/mail   /usr/bin/nologin
ftp                    system         14    11 -                              /srv/ftp          /usr/bin/nologin
rpc                    system         32    32 Rpcbind Daemon                 /var/lib/rpcbind  /usr/bin/nologin
http                   system         33    33 -                              /srv/http         /usr/bin/nologin
named                  system         40    40 BIND DNS Server                /                 /usr/bin/nologin
uuidd                  system         68    68 -                              /                 /usr/bin/nologin
dbus                   system         81    81 System Message Bus             /                 /usr/bin/nologin
polkitd                system        102   102 PolicyKit daemon               /                 /usr/bin/nologin
partimag               system        110   110 Partimage user                 /                 /usr/bin/nologin
rtkit                  system        133   133 RealtimeKit                    /proc             /usr/bin/nologin
usbmux                 system        140   140 usbmux user                    /                 /usr/bin/nologin
nvidia-persistenced    system        143   143 NVIDIA Persistence Daemon      /                 /usr/bin/nologin
cups                   system        209   209 cups helper user               /                 /usr/bin/nologin
saned                  system        962   962 SANE daemon user               /                 /usr/bin/nologin
libvirt-qemu           system        964   964 Libvirt QEMU user              /                 /usr/bin/nologin
sddm                   system        968   968 Simple Desktop Display Manager /var/lib/sddm     /usr/bin/nologin
openvpn                system        969   969 OpenVPN                        /                 /usr/bin/nologin
nbd                    system        970   970 Network Block Device           /var/empty        /usr/bin/nologin
git                    system        971   971 git daemon user                /                 /usr/bin/git-shell
dnsmasq                system        972   972 dnsmasq daemon                 /                 /usr/bin/nologin
dhcpcd                 system        973   973 dhcpcd privilege separation    /                 /usr/bin/nologin
avahi                  system        974   974 Avahi mDNS/DNS-SD daemon       /                 /usr/bin/nologin
tss                    system        975   975 tss user for tpm2              /                 /usr/bin/nologin
systemd-timesync       system        976   976 systemd Time Synchronization   /                 /usr/bin/nologin
systemd-resolve        system        977   977 systemd Resolver               /                 /usr/bin/nologin
systemd-journal-remote system        978   978 systemd Journal Remote         /                 /usr/bin/nologin
systemd-oom            system        979   979 systemd Userspace OOM Killer   /                 /usr/bin/nologin
systemd-network        system        980   980 systemd Network Management     /                 /usr/bin/nologin
systemd-coredump       system        981   981 systemd Core Dumper            /                 /usr/bin/nologin
test                   regular      1000  1000 Test                           /home/test        /bin/bash

```
To kill all processes for a user session and deallocate the resources use:
```shell
loginctl kill-user user
```
* For those who want to know more:
Users interact with the system using GUI or by terminal interface.
[Here](https://www.golinuxcloud.com/difference-between-pty-vs-tty-vs-pts-linux/) you can find out more about terminal devices avaliable to users.
More simultaneous users with keyboards and screens on one PC? No problem! [Multi-seat](https://www.freedesktop.org/wiki/Software/systemd/multiseat/).
# Systemd
Quoting materials given for homework from doc. dr. Vedran Miletić @ UniRi - Faculty of Informatics and Digital Technologies
>modern operating systems based on Linux kernel use systemd for managing processes, specifically starting services on system startup.

Systemd ecosystem
>A set of programs called systemd actually contains a number of interconnected and somewhat independent tools, the most important of which are:
> * systemd init process, it is the first process to run the Linux kernel after loading and it is in charge of starting and managing the life of other processes
> * systemd services, which are files for defining requests, sequences and instructions for running other processes required for system operation
> * systemctl tool, for viewing and managing defined service states
> *journalctl tool, to view system events and service-related events

Service:
> A unit configuration file whose name ends in .service encodes information about a process controlled and supervised by systemd.
Target:
> A unit configuration file whose name ends in ".target" encodes information about a target unit of systemd, which is used for grouping units and as well-known synchronization points during start-up.
> 
These both refer to unit configuration files, which are:
>A unit configuration file encodes information about a service, a socket, a device, a mount point, an automount point, a swap file or partition, a start-up target, a watched file system path, a timer controlled and supervised by systemd(1), a resource management slice or a group of externally created processes.


Systemd unit is a configuration file used to implement startup services and goals:
> * services that run daemon processes are defined by * .service files,
> * startup targets are defined by * .target files.
## How services work?
### Pipeline
> A pipe is a form of redirection (transfer of standard output to some other destination) that is used in Linux and other Unix-like operating systems to send the output of one command/program/process to another command/program/process for further processing. 
> Pipe is used to combine two or more commands, and in this, the output of one command acts as input to another command, and this command’s output may act as input to the next command and so on. It can also be visualized as a temporary connection between two or more commands/ programs/ processes. The command line programs that do the further processing are referred to as filters. 
> This direct connection between commands/ programs/ processes allows them to operate simultaneously and permits data to be transferred between them continuously rather than having to pass it through temporary text files or through the display screen. 

Every service, process, program is connected to a service bus.
User service bus:
```shell
[root@archlinux ~]# stat /run/user/0/bus
  File: /run/user/0/bus
  Size: 0         	Blocks: 0          IO Block: 4096   socket
Device: 0,48	Inode: 16          Links: 1
Access: (0666/srw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-03-11 14:13:08.986285110 +0000
Modify: 2022-03-11 14:13:08.986285110 +0000
Change: 2022-03-11 14:13:08.986285110 +0000
 Birth: -
```
To list all bus connections viewed by process and the user who started it use:
```
[test@test-PC ~]$ busctl
NAME                                      PID PROCESS         USER            CONNECTION    UNIT             >
:1.0                                      481 systemd-resolve systemd-resolve :1.0          systemd-resolved.>
:1.1                                      472 systemd-network systemd-network :1.1          systemd-networkd.>
:1.11                                     635 sddm            root            :1.11         sddm.service     >
:1.118                                   3444 sudo            root            :1.118        user@1000.service>
:1.119                                   3448 busctl          root            :1.119        user@1000.service>
:1.12                                     647 Xorg            root            :1.12         sddm.service     >
:1.16                                     803 systemd         test            :1.16         user@1000.service>
:1.18                                     855 kded5           test            :1.18         session-1.scope  >
:1.19                                     858 kwin_x11        test            :1.19         session-1.scope  >
:1.2                                        1 systemd         root            :1.2          init.scope       >
:1.20                                     896 polkitd         polkitd         :1.20         polkit.service   >
:1.21                                     890 ksmserver       test            :1.21         session-1.scope  >
...
```
* Busctl - active mutual communication of processes locally, where they are connected, etc.
* [Any]ctl - bus in which a SERVICE communicate with a SERVICEd (daemon)
* machinectl is "systemd's docker"
* coredumpctl - coredumpctl is a tool that can be used to retrieve and process core dumps and metadata which were saved by systemd-coredump

Socket Statistics ss -l:
>Ss is an abbreviation for Socket Statistics. As the name suggests, the ss command can be used to get socket statistics, which can display something similar to netstat. The advantage of ss is that it can display more detailed information about TCP and connection status, and is faster and more efficient than netstat.

>When the number of socket connections on the server becomes very large, either using the netstat command or directly cat /proc/net/tcp, the execution speed will be slow.

## Some systemd units:
```shell
[root@archlinux ~]# ls /usr/bin/*ctl
/usr/bin/auditctl     /usr/bin/journalctl  /usr/bin/oomctl	 /usr/bin/userdbctl
/usr/bin/bootctl      /usr/bin/keyctl	   /usr/bin/portablectl  /usr/bin/wdctl
/usr/bin/busctl       /usr/bin/localectl   /usr/bin/resolvectl	 /usr/bin/zramctl
/usr/bin/coredumpctl  /usr/bin/loginctl    /usr/bin/sysctl
/usr/bin/homectl      /usr/bin/machinectl  /usr/bin/systemctl
/usr/bin/hostnamectl  /usr/bin/networkctl  /usr/bin/timedatectl
```
>oomctl may be used to get information about the various contexts read in by the systemd(1) userspace out-of-memory (OOM) killer, systemd-oomd(8). 
* e.g. when service mariadb crashes, view why it crashes.
* oomctl - "OOM KILLER" - deletes the service that uses a lot of system memory
## Systemd startup target unit

Systemd boots the system to a predefined target consisting of specific running services.

The predefined launch targets are:
* poweroff.target (SysV runlevel0')
* reboot.target (SysV runlevel6)
* multi-user.target (SysV runlevel[234])
* graphical.target (SysV runlevel5)
* rescure.target (SysV runlevel1)
* emergency.target

In the general case, systemd boots the system to the default.target state, which typically points to graphical.target. The system will boot to the graphical interface and allow you to log in only if one of the display managers, such as Simple Desktop Display Manager (SDDM) and GNOME Display Manager (GDM).
* systemctl isolate TARGET command can be used to move the system to the desired boot destination.

Services are implemented as service units, e.g. acpid.service, sshd.service.
A detailed specification can be found in the systemd.service man page.

## Syslog
```shell
command rsyslog
configuration file /etc/rsyslog.conf
  $ -- global commands
  $ModLoad
```

## Rotate log files
* command logrotate
* config file /etc/logrotate.conf
    * directives that determine how often rotation occurs: daily, weekly, monthly, yearly
    * directives that determine compression: compress, nocompress, compresscmd, uncompresscmd, compressext, compressoptions, delaycompress
    * directive rotate NUMBER makes sure that the NUMBER of rotated files is saved, ie that the log file passes the NUMBER of rotation before deleting
    * directive mail ADRESS allows you to send a log file by email to the ADDRESS address when rotating just before deleting

## Systemd timer
>to automate the execution of individual services in accordance with a certain time schedule, we can use systemd units of the type timer 
> * all existing timers on the system and their states can be viewed with the systemctl list-timers command
> * every times consists of two files located in directory /etc/systemd/system
>     * SERVICE.timer - defines the time dependency or start time of our service
>     * SERVICE.service - defines the service itself or the commands to run

Through all the examples we use SERVICE as the name of our pair of timers and systemd services:
* enabling and starting the timer is done with the help of systemctl tools as for all other types of units:
```shell
systemctl enable SERVICE.timer
systemctl disable SERVICE.timer
systemctl start SERVICE.timer
systemctl stop SERVICE.timer
```
Structure of systemd.timer files:
* the SERVICE.timer file must be located in the /etc/systemd/system directory and be of the following format:
```shell
[Unit]
Description=My time dependent service

[Timer]
# ...

[Install]
WantedBy=timers.target
```
Structure of SERVICE.service files:
>The difference between the SERVICE.service file for using a timer from a typical systemd service file is in:
>* SERVICE.service for timer file does not contain a section [Install] timer service
>* It does not need to be enabled or run with systemctl, we enable and run the SERVICE.timer unit instead

Example format:
```shell
[Unit]
# ...

[Service]
# ...
```

There are two types of timers:
* relative for a event
* defined for a specific time

Parameters for a timer defined relative to an event:
* OnActiveSec - relative to the start time of the timer itself 
* OnBootSec - relative to the start time of OS boot
* OnStartupSec - relative to starting systemd process
* OnUnitActiveSec - relative to the time of last unit start
* OnUnitInactiveSec - relative to the time of last unit stop
* OnCalendar - defined for a specific date-time
* Relative times are defined by combinig time measurement units: 1h 20m or 5 s
* Specific times can be defined generically or specifically by valuess: hourly, daily, weekly, monthly, 2003-03-05 05:40


## Systemd Journal
> Journal (daemon systemd-journald) is a component of systemd in charge of viewing and managing log files
> a replacement for Syslog, although systems can use both
> The systemd-journald service collects and stores log data and associated metadata in binary form
> * The journalctl tool is used to view log files
> * Output structure similar to / var / log / messages
> * High-priority messages are highlighted visually (red, bold)
> * Times are translated to the local time zone
> * All data is displayed, including rotated log files
* journalctl -n NUMBER displays only the last NUMBER of messages
* journalctl -o FORM shows the output in the form of FORM, where FORM can be verbose, export, json, ...
* journalctl -f tracks messages as they are created

## Rsync backup service
> rsync is an open source tool for efficient incremental data transfer. It is very useful as a data backup tool. When used for backup, a very important parameter is --archive, or -a which stores metadata about files
Usage:
```
rsync -a /myDirIwantToBackup /backupDir
```

## Starting units on boot 
### Example 1: Reflector
>Reflector is a Python script which can retrieve the latest mirror list from the Arch Linux Mirror Status page, filter the most up-to-date mirrors, sort them by speed and overwrite the file /etc/pacman.d/mirrorlist. 

```
[fidit@archlinux ~]$ systemctl status reflector-init.service
● reflector-init.service - Initializes mirrors for the VM
     Loaded: loaded (/etc/systemd/system/reflector-init.service; enabled; vendor preset: disabled)
     Active: active (exited) since Fri 2022-04-22 12:15:47 UTC; 8min ago
    Process: 692 ExecStart=reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/m>
   Main PID: 692 (code=exited, status=0/SUCCESS)
        CPU: 1.176s

[fidit@archlinux ~]$ cat /etc/systemd/system/reflector-init.service
[Unit]
Description=Initializes mirrors for the VM
After=network-online.target
Wants=network-online.target
Before=sshd.service cloud-final.service
ConditionFirstBoot=yes

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist

[Install]
WantedBy=multi-user.target
[fidit@archlinux ~]$ cat /etc/systemd/system/pacman-init.service
[Unit]
Description=Initializes Pacman keyring
Before=sshd.service cloud-final.service
ConditionFirstBoot=yes

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/pacman-key --init
ExecStart=/usr/bin/pacman-key --populate archlinux

[Install]
WantedBy=multi-user.target
```
* Services can be executed paralelly on boot process.
* Services know what they depend on, it is possible to change the order.
* After=network-online.target cloud-config.service rc-local.service - condition what needs to be started before this service.
* Before=sshd.service cloud-final.service - condition before which services must this service start
* ConditionFirstBoot=yes - runs only on first OS boot

### Example 2: cloud-final
>This stage runs as late in boot as possible. Any scripts that a user is accustomed to running after logging into a system should run correctly here. Things that run here include:
> *   package installations
> * configuration management plugins (puppet, chef, salt-minion)
> * user-defined scripts (e.g. shell scripts passed as user-data)

```shell
[fidit@archlinux ~]$ cat /usr/lib/systemd/system/cloud-final.service
[Unit]
Description=Execute cloud user/final scripts
After=network-online.target cloud-config.service rc-local.service
Wants=network-online.target cloud-config.service


[Service]
Type=oneshot
ExecStart=/usr/bin/cloud-init modules --mode=final
RemainAfterExit=yes
TimeoutSec=0
KillMode=process
TasksMax=infinity

# Output needs to appear in instance console output
StandardOutput=journal+console

[Install]
WantedBy=cloud-init.target

[fidit@archlinux ~]$ cat /usr/lib/systemd/system/cloud-config.service
[Unit]
Description=Apply the settings specified in cloud-config
After=network-online.target cloud-config.target
After=snapd.seeded.service
Wants=network-online.target cloud-config.target

[Service]
Type=oneshot
ExecStart=/usr/bin/cloud-init modules --mode=config
RemainAfterExit=yes
TimeoutSec=0

# Output needs to appear in instance console output
StandardOutput=journal+console

[Install]
WantedBy=cloud-init.target
```
List dependencies of a service:
```shell
[fidit@archlinux ~]$ systemctl list-dependencies cloud-init.target
cloud-init.target
● ├─cloud-config.service
● ├─cloud-final.service
● ├─cloud-init-local.service
● └─cloud-init.service
```

# Disks and ZFS
Let's add some storage to our VM.
We will add 5 disks using virt-manager

Linux stores all connected disks into the directory /dev/disk. 
You can view the disks by id, label, partition UUID or disk UUID.

In a virtual machine disks are stored in /dev/vd* where * is the number of a drive

Let's view all disks and their info in the system:
```shell
[test@test-PC ~]$ sudo fdisk -l
Disk /dev/vda: 12 GiB, 12884901888 bytes, 25165824 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 902FD2FE-154A-4635-9BF8-3FAB93BA67FC

Device     Start      End  Sectors Size Type
/dev/vda1   2048     4095     2048   1M BIOS boot
/dev/vda2   4096 25165790 25161695  12G Linux filesystem


Disk /dev/vdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vdc: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vdd: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vde: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

We will now create a RAID disk pool using ZFS.
ZFS:
* ZFS - [ArchWiki](https://wiki.archlinux.org/title/ZFS) - pay atention to the Linux kernel version and edition
    * ZFS needs to be compiled on the system and binds itself to the system hardware. 
    * Linux kernel versions: linux, lts, zen (multimedia), dkms - more on that later
    * Why ZFS? No technical filesystem limits (as of today), portable, fast etc.
    * Has built-in Volume manager

What's my Linux Kernel?: 
```shell
[test@test-PC ~]$uname -a
Linux archlinux 5.17.1-arch1-1 #1 SMP PREEMPT Mon, 28 Mar 2022 20:55:33 +0000 x86_64 GNU/Linux
```

**Install ZFS**
1. Download [PKGBUILD](https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-dkms)
2. Then prerequisites:
```shell
pacman -S make fakeroot patch autoconf automake dkms nano
``` 
Download 0001-only-build-the-module-in-dkms patch.
```shell
curl -o 0001-only-build-the-module-in-dkms.conf.patch "https://aur.archlinux.org/cgit/aur.git/plain/0001-only-build-the-module-in-dkms.conf.patch?h=zfs-dkms"
```
4. Problems with [GPG package](https://wiki.archlinux.org/title/Pacman/Package_signing ):
    * For signing use: [link](https://wiki.archlinux.org/title/GnuPG)
```
pacman-key --recv-keys keyid
or
gpg --recv-keys keyid
```
The key on 27.04.2022. was: 6AD860EED4598027
5. Install ZFS Utils
```
curl -o zfs.initcpio.hook "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.hook?h=zfs-utils"
curl -o zfs.initcpio.install "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.install?h=zfs-utils"
nano PKGBUILD-utils -> https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-utils
makepkg -p PKGBUILD-utils

```

6. Then build and install ZFS
```shell
makepkg
sudo pacman -U zfs-dkms-2.1.4-1-any.pkg.tar.zst zfs-utils-2.1.4-1-x86_64.pkg.tar.zst
```
7. Test does ZFS even work?
```shell
sudo modprobe zfs
```
No? That's because of [Kernel Headers](https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support) install them with:
```shell
pacman -S linux-headers
```
Kernel Headers are used when various modules are using (connecting) to the Linux kernel - let's say they are the same as libraries in programming languages.

**RAID pool*
Let's create a zpool.

ZFS is dividen in two layers:
* ZFS - High layer
* Zpool - low level

*Zpool is..?*
>ZFS filesystems are built on top of virtual storage pools called zpools. A zpool is constructed of virtual devices (vdevs), which are themselves constructed of block devices: files, hard drive partitions, or entire drives, with the last being the recommended usage.

RAID-Z - software RAID inside ZFS. Best for data recovery because it dosen't depend on a physical RAID controller.
It has features like:
* Parity disks
* Data deduplication - if finds multiple copies of the same file, leaves only one, deletes others and creates symbolic links to the one kept in RAID pool.
* Auto-compression (lz4 and zstd)

Setup all disks for ZFS and create a pool on disk vdb:
```
fdisk /dev/vd[bcde]
- Typed in: g, n, 1, enter, enter, t, L, 157 "Solaris /usr and Apple ZFS", w
zpool create mypool /dev/vdb
```
What have we done?
```shell
[user@archlinux ~]$ fdisk -l /dev/vd[bcde] | grep ZFS
/dev/vdb1   2048 2095103 2093056 1022M Solaris /usr & Apple ZFS
/dev/vdc1   2048 2095103 2093056 1022M Solaris /usr & Apple ZFS
/dev/vdd1   2048 2095103 2093056 1022M Solaris /usr & Apple ZFS
/dev/vde1   2048 2095103 2093056 1022M Solaris /usr & Apple ZFS
[user@archlinux ~]$ zpool create mojbazen /dev/vdb
[user@archlinux ~]$ zpool list
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
mypool     960M   120K   960M        -         -     0%     0%  1.00x    ONLINE  -
```

Delete a pool:
```shell
zpool destroy mypool
```

Let's create a RAID mirrored pool and mount it to the point /mypool: 
```shell
zpool create -m /mypool mypool mirror /dev/vdb1 /dev/vdc1
mount | grep poo
mypool on /mypool type zfs (rw,xattr,noacl)
zpool list
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
mypool     960M   120K   960M        -         -     0%     0%  1.00x    ONLINE  -
```
Let's create one directory and one file and give ourselfs ownership over the file in that mounted directory:
```shell
[fidit@archlinux ~]$ cd /mypool
[fidit@archlinux mypool]$ ls
[fidit@archlinux mypool]$ sudo mkdir mydir
[fidit@archlinux mypool]$ sudo touch myfile
[fidit@archlinux mypool]$ ls
myfile mydir
[fidit@archlinux mypool]$ ls -la
total 18
drwxr-xr-x 3 root root   4 Apr  1 14:17 .
drwxr-xr-x 1 root root 146 Apr  1 14:15 ..
-rw-r--r-- 1 root root   0 Apr  1 14:17 myfile
drwxr-xr-x 2 root root   2 Apr  1 14:16 mydir
[fidit@archlinux mypool]$ chown fidit:fidit myfile
```
We will now list our pool:
```shell
[fidit@archlinux mypool]$ zpool list
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
mypool     960M   153K   960M        -         -     0%     0%  1.00x    ONLINE  -
```
How to attach detach more disks to the pool:
```shell
[fidit@archlinux mypool]$ zpool attach mypool /dev/vdd1
[fidit@archlinux mypool]$ sudo zpool detach mypool vdb1
[fidit@archlinux mypool]$ sudo zpool add mypool vdb1
[fidit@archlinux mypool]$ sudo zpool add mypool vde1
```
We created a RAID-0 stripe:
```shell
[fidit@archlinux mypool]$ zpool status
  pool: mypool
 state: ONLINE
config:

	NAME        STATE     READ WRITE CKSUM
	mypool      ONLINE       0     0     0
	  vdc1      ONLINE       0     0     0
	  vdb1      ONLINE       0     0     0
	  vde1      ONLINE       0     0     0
```

Extending mirrors:
```shell
[fidit@archlinux ~]$ sudo zpool destroy mypool
[fidit@archlinux ~]$ sudo zpool create -m /mypool mypool mirror /dev/vdb1 /dev/vdc1 
[fidit@archlinux ~]$ sudo zpool attach mypool /dev/vdc1 /dev/vdd1
[fidit@archlinux ~]$ zpool status
  pool: mypool
 state: ONLINE
  scan: resilvered 232K in 00:00:00 with 0 errors on Fri Apr  1 14:46:44 2022
config:

	NAME        STATE     READ WRITE CKSUM
	mypool    ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vdb1    ONLINE       0     0     0
	    vdc1    ONLINE       0     0     0
	    vdd1    ONLINE       0     0     0
```

**RAID-Z**
>This is a non-standard RAID that uses the ZFS file system; no other file system can be used for this array. Note that there is not a single hardware controller that implements RAID-Z.
>The ZFS file system uses an additional level of checksums to search for data corruption without displaying the appropriate messages. ZFS uses checksums with any level of redundancy, including single disk pools. The distribution mechanism is similar to RAID 5, but it uses dynamic bandwidth. Any block, regardless of its size, has its own RAID bandwidth, which means that each RAID-Z record is a full-band record.
[source](https://www.diskinternals.com/raid-recovery/what-is-raidz/)

How to create RAID-Z pool:      
```shell
sudo zpool create -m /mypool mypool raidz /dev/vdb1 /dev/vdc1 /dev/vdd1 /dev/vde1
```

So what about parity?
* If you have up to 5 disks only one parity drive is needed
* If you have 6 - 8 disks -> 2 parity drives
* 8+ disks -> 3 parity and more..
    * Those parity disks are usually the same size

RAID-Z variants:
*RAID-Z
*RAID-Z2
*RAID-Z3

>In RAID-Z, files are never divided exactly in half, but the data is treated as blocks of a fixed length. The minimum number of disks you can use is three. Three drives are usually combined into a virtual device (vdev).

>RAID-Z (sometimes called RAID-Z1) will provide a record of each unique data block so that it can recover from the failure of any single disk on vdev. In this case, the data is automatically distributed across the disk in the most optimal way. RAID-Z1 is practically an analogue of RAID 5, as it uses single parity. It is not difficult to guess that fault tolerance will be the same; i.e., there may be a possible failure of a drive.

>RAID-Z2 is more fault-tolerant, as it uses two parity blocks and two data blocks from one piece of information. This is an analogue of RAID 6 and can also withstand the collapse of as many as two disks. In RAID-Z2, the maximum number of disks is at least four.

>You can go further and try RAID-Z3, which has a maximum of at least five disks and the ability to survive damage to up to three disks. RAID-Z3 is rarely used due to its size. It is also not as space-efficient as other RAID-Z options.
[source](https://www.diskinternals.com/raid-recovery/what-is-raidz/)
How to create RAID-Z2:
```shell
[fidit@archlinux ~]$ sudo zpool create -m /mypool mypool raidz2 /dev/vdb1 /dev/vdc1 /dev/vdd1 /dev/vde1
[fidit@archlinux ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
dev             478M     0  478M   0% /dev
run             485M  796K  485M   1% /run
/dev/vda2        12G  2.4G  9.3G  21% /
tmpfs           485M     0  485M   0% /dev/shm
tmpfs           485M     0  485M   0% /tmp
tmpfs            97M     0   97M   0% /run/user/1000
mypool          1.8G  128K  1.8G   1% /mypool
[fidit@archlinux ~]$ sudo zpool status
  pool: mypool
 state: ONLINE
config:

	NAME        STATE     READ WRITE CKSUM
	mypool      ONLINE       0     0     0
	  raidz2-0  ONLINE       0     0     0
	    vdb1    ONLINE       0     0     0
	    vdc1    ONLINE       0     0     0
	    vdd1    ONLINE       0     0     0
	    vde1    ONLINE       0     0     0
```
RAID-Z3:
```shell
[fidit@archlinux sudo zpool create -m /mypool raidz3 /dev/vdb1 /dev/vdc1 /dev/vdd1 /dev/vde1
[fidit@archlinux ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
dev             478M     0  478M   0% /dev
run             485M  796K  485M   1% /run
/dev/vda2        12G  2.4G  9.3G  21% /
tmpfs           485M     0  485M   0% /dev/shm
tmpfs           485M     0  485M   0% /tmp
tmpfs            97M     0   97M   0% /run/user/1000
mypool          832M  128K  832M   1% /mypool
```

Set compression method on a ZFS pool: 
```
[fidit@archlinux ~]$ sudo zfs set compression=zstd mypool
```

Security mechanisms
* Resilvering - zfs reads data from all devices and makes sure that all of it can be read correctly (no errors, proper checksums etc.)
* Scrubbing and resilvering are very similar operations. The difference is that resilvering only examines data that ZFS knows to be out of date (for example, when attaching a new device to a mirror or replacing an existing device), whereas scrubbing examines all data to discover silent errors due to hardware faults or disk failure.
* Because scrubbing and resilvering are I/O-intensive operations, ZFS only allows one at a time.
* A scrub is split into two parts: metadata scanning and block scrubbing. The metadata scanning sorts blocks into large sequential ranges which can then be read much more efficiently from disk when issuing the scrub I/O. 

[source](https://openzfs.github.io/openzfs-docs/man/8/zpool-scrub.8.html)
How to?
```shell
sudo zpool resilver mypool
sudo zpool scrub mypool
```

# Optimizing Kernel
Kernel? [Linux Kernel web](www.kernel.org)
ArchLinux always uses the latest kernel.
Officially supported kernels
* Stable — Vanilla Linux kernel and modules, with a few patches applied.
* Hardened — A security-focused Linux kernel applying a set of hardening patches to mitigate kernel and userspace exploits. It also enables more upstream kernel hardening features than linux.
* Longterm — Long-term support (LTS) Linux kernel and modules.
* Zen Kernel    

For every server that is used for providing some kind of service, it is recomended that it runs on a Long-term supported Linux kernel.  
    
**How to install long term support linux kernel?**
First install LTS kernel:
```shell
sudo pacman -S linux-lts
```
Then edit /boot/grub/grub.cfg:
```shell
linux   /boot/vmlinuz-linux-lts root=UUID=ef9af0ad-7937-442e-af1e-0f71094122b2 rw net.ifname>
initrd  /boot/initramfs-linux-lts.img
```
Linux will now boot with the LTS kernel, previously used kernel (linux linux - latest) is avaliable in GNU GRUB startup menu. 
    
GNU GRUB is:
> GNU GRUB is a Multiboot boot loader. It was derived from GRUB, the GRand Unified Bootloader, which was originally designed and implemented by Erich Stefan Boleyn.

Briefly, a boot loader is the first software program that runs when a computer starts. It is responsible for loading and transferring control to the operating system kernel software.
    
It can be run in:
* BIOS mode
* UEFI mode

Gummiboot instead of GRUB? (systemd's Gummiboot) -> GRUB and systemd bootloaders are not the same, Arch uses it's own bootloader.

In the boot process you can notice initramfs and vmliuz.. what is that?
* initramfs linux fallback - kernel modules that are required by your PC's (server, VM etc.) hardware
* vmlinuz - Linux Kernel - same on every computer running that kernel version

## Analyzing boot times
To view detail specification about loading times for all system and user services use:
```
systemd-analyze blame
```
If you want a graph of the system booting process:
```
systemd-analyze dot
systemd-analyze plot > graph.svg
``` 
You can see some red coloured items. They are part of the Critical-chain.
What is the Critical-chain? [Basically this.](https://trstringer.com/systemd-critical-chain/)
* "+" shows how much seconds has been added to the boot time and @ shows when did a service started compared to booting.

## Analyzing service security
To view the security state of all services use:
```shell
[fidit@archlinux ~]$ systemd-analyze security
UNIT                                 EXPOSURE PREDICATE HAPPY
auditd.service                            8.7 EXPOSED   🙁
dbus.service                              9.6 UNSAFE    😨
dm-event.service                          9.5 UNSAFE    😨
emergency.service                         9.5 UNSAFE    😨
getty@tty1.service                        9.6 UNSAFE    😨
rescue.service                            9.5 UNSAFE    😨
shadow.service                            9.6 UNSAFE    😨
sshd.service                              9.6 UNSAFE    😨
systemd-ask-password-console.service      9.4 UNSAFE    😨
systemd-ask-password-wall.service         9.4 UNSAFE    😨
systemd-homed.service                     5.8 MEDIUM    😐
systemd-journald.service                  4.3 OK        🙂
systemd-logind.service                    2.8 OK        🙂
systemd-networkd.service                  2.6 OK        🙂
systemd-resolved.service                  2.1 OK        🙂
systemd-rfkill.service                    9.4 UNSAFE    😨
systemd-timesyncd.service                 2.1 OK        🙂
systemd-udevd.service                     6.7 MEDIUM    😐
systemd-userdbd.service                   2.2 OK        🙂
user@1000.service                         9.4 UNSAFE    😨
```
To view the security state and parameters of a specific service use:
```shell
[fidit@archlinux ~]$ systemd-analyze security sshd.service
  NAME                                                        DESCRIPTION                           >
✗ RootDirectory=/RootImage=                                   Service runs within the host's root di>
  SupplementaryGroups=                                        Service runs as root, option does not >
  RemoveIPC=                                                  Service runs as root, option does not >
✗ User=/DynamicUser=                                          Service runs as root user             >
✗ CapabilityBoundingSet=~CAP_SYS_TIME                         Service processes may change the syste>
✗ NoNewPrivileges=                                            Service processes may acquire new priv>
✓ AmbientCapabilities=                                        Service process does not receive ambie>
✗ PrivateDevices=                                             Service potentially has access to hard>
✗ ProtectClock=                                               Service may write to the hardware cloc>
✗ CapabilityBoundingSet=~CAP_SYS_PACCT                        Service may use acct()                >
✗ CapabilityBoundingSet=~CAP_KILL                             Service may send UNIX signals to arbit>
✗ ProtectKernelLogs=                                          Service may read from or write to the >
✗ CapabilityBoundingSet=~CAP_WAKE_ALARM                       Service may program timers that wake u>
✗ CapabilityBoundingSet=~CAP_(DAC_*|FOWNER|IPC_OWNER)         Service may override UNIX file/IPC per>
✗ ProtectControlGroups=                                       Service may modify the control group f>
✗ CapabilityBoundingSet=~CAP_LINUX_IMMUTABLE                  Service may mark files immutable      >
✗ CapabilityBoundingSet=~CAP_IPC_LOCK                         Service may lock memory into RAM      >
✗ ProtectKernelModules=                                       Service may load or read kernel module>
✗ CapabilityBoundingSet=~CAP_SYS_MODULE                       Service may load kernel modules       >
✗ CapabilityBoundingSet=~CAP_SYS_TTY_CONFIG                   Service may issue vhangup()           >
✗ CapabilityBoundingSet=~CAP_SYS_BOOT                         Service may issue reboot()            >
✗ CapabilityBoundingSet=~CAP_SYS_CHROOT                       Service may issue chroot()            >
✗ PrivateMounts=                                              Service may install system mounts     >
✗ SystemCallArchitectures=                                    Service may execute system calls with >
✗ CapabilityBoundingSet=~CAP_BLOCK_SUSPEND                    Service may establish wake locks      >
✗ MemoryDenyWriteExecute=                                     Service may create writable executable>
✗ RestrictNamespaces=~user                                    Service may create user namespaces    >
✗ RestrictNamespaces=~pid                                     Service may create process namespaces >
✗ RestrictNamespaces=~net                                     Service may create network namespaces >
✗ RestrictNamespaces=~uts                                     Service may create hostname namespaces>
✗ RestrictNamespaces=~mnt                                     Service may create file system namespa>
✗ CapabilityBoundingSet=~CAP_LEASE                            Service may create file leases        >
✗ CapabilityBoundingSet=~CAP_MKNOD                            Service may create device nodes       >
✗ RestrictNamespaces=~cgroup                                  Service may create cgroup namespaces  >
✗ RestrictSUIDSGID=                                           Service may create SUID/SGID files    >
✗ RestrictNamespaces=~ipc                                     Service may create IPC namespaces     >
✗ ProtectHostname=                                            Service may change system host/domainn>
✗ CapabilityBoundingSet=~CAP_(CHOWN|FSETID|SETFCAP)           Service may change file ownership/acce>
✗ CapabilityBoundingSet=~CAP_SET(UID|GID|PCAP)                Service may change UID/GID identities/>
✗ LockPersonality=                                            Service may change ABI personality    >
✗ ProtectKernelTunables=                                      Service may alter kernel tunables     >
✗ RestrictAddressFamilies=~AF_PACKET                          Service may allocate packet sockets   >
✗ RestrictAddressFamilies=~AF_NETLINK                         Service may allocate netlink sockets  >
✗ RestrictAddressFamilies=~AF_UNIX                            Service may allocate local sockets    >
✗ RestrictAddressFamilies=~…                                  Service may allocate exotic sockets   >
✗ RestrictAddressFamilies=~AF_(INET|INET6)                    Service may allocate Internet sockets >
✗ CapabilityBoundingSet=~CAP_MAC_*                            Service may adjust SMACK MAC          >
✗ RestrictRealtime=                                           Service may acquire realtime schedulin>
```
* SSHD is executed as root user at the port 22
* To ensure that some services don't go crazy while working use capatibilities in security 
* It's only restricted on that specific local machine

## Editing boot options
Editing GRUB boot options /boot/grub/grub.cfg
Just in case, if you break something, edit the 1st of Advanced options in GRUB config menuentry - Submenu.
For pracice we will edit the following in the line that begins with linux:
* Read-write to read-only - ro - simulate disk error (when it occurs probably disks in the server are bad.)
* Parameter 1 - rescue mode
* Parameter 3 - no GUI option
* net.ifnames=1 - net.ifnames tells systemd to create predictable NIC names
```shell
linux /boot/vmlinuz-linux root=UUID=dcf21c7c-7209-49d1-b9a6-70e4689c16a6 ro 1 3 net.ifnames=1 quiet resume=UUID=91f4>
```
Let's see can we boot to GUI?    
```shell
[fidit@archlinux ~]$ systemctl get-default
graphical.target
Note: found "3" on the kernel commandline, which overrides the default unit.
```
List all modules:
```shell
lsmod
```
View info about a specific module:
```shell
modinfo name
[lljubojevic@Ljubojevic-PC ~]$ modinfo kvm
filename:       /lib/modules/5.17.4-arch1-1/kernel/arch/x86/kvm/kvm.ko.zst
license:        GPL
author:         Qumranet
srcversion:     6121525AFDBD6FA12A6EA40
depends:        irqbypass
retpoline:      Y
intree:         Y
name:           kvm
vermagic:       5.17.4-arch1-1 SMP preempt mod_unload 
sig_id:         PKCS#7
signer:         Build time autogenerated kernel key
sig_key:        36:76:4D:01:56:28:F8:D4:45:6E:7C:84:4E:B4:8A:6F:98:D8:6D:FD
sig_hashalgo:   sha512
signature:      30:64:02:30:4F:1A:2B:11:FE:3F:49:47:A2:26:BE:8A:87:F5:62:60:
                F9:15:74:57:BA:47:F2:B9:95:3E:25:78:4E:7D:1E:41:CF:C4:5B:FB:
                BC:91:BD:2F:CE:BF:1E:70:B5:93:A4:E5:02:30:41:5D:E7:62:8B:60:
                B3:15:09:19:3E:6D:13:A2:54:26:66:2F:3E:FF:12:74:47:10:6F:71:
                97:BC:24:EF:62:A4:58:15:7A:0B:27:5F:FC:F2:B0:2C:9E:A0:1D:D9:
                9E:73
parm:           tdp_mmu:bool
parm:           mmio_caching:bool
parm:           nx_huge_pages:bool
parm:           nx_huge_pages_recovery_ratio:uint
parm:           nx_huge_pages_recovery_period_ms:uint    
```
* parm: are parameters of a module

For example, let's optimize our boot proces by excluding sound module (servers in most cases don't need sound):
* snd_intel_hda - sound module
Shut it down with GRUB parameter in line linux:
```shell
linux /boot/vmlinuz-linux root=UUID=dcf21c7c-7209-49d1-b9a6-70e4689c16a6 snd-hda-intel.enable=0 quiet resume=UUID=91f4 
```
Test:
```shell
dmesg | grep snd
```
    
**Blacklisting modules**
>Blacklisting, in the context of kernel modules, is a mechanism to prevent the kernel module from loading. This could be useful if, for example, the associated hardware is not needed, or if loading that module causes problems: for instance there may be two kernel modules that try to control the same piece of hardware, and loading them together would result in a conflict. 

Example:
```shell
module_blacklist=kvm,btrfs,cdrom
```
* Dosen't boot? Btrfs is a filesystem and the core can't be loaded without it. 

Also a heads up, never delete kernels that WORK! :/
You may notice that on first boot when cloud-init is configuring the system, boot time is higher than later on. You can check if cloud-init has started on current boot with:
```shell
journalctl --unit cloud-init -b
```
That can be useful when you change cloud-init config files.

# Firewalls
Layer 3 and layer 2 firewalls always analyze packet headers
On Linux, software firewalls are:
* iptables - will be replaced with nftables
* nftables
    * > nftables is the modern Linux kernel packet classification framework. New code should use it instead of the legacy {ip,ip6,arp,eb}_tables (xtables) infrastructure. For existing codebases that have not yet converted, the legacy xtables infrastructure is still maintained as of 2021
* netlink socket 
    * > Netlink is used to transfer information between the kernel and
       user-space processes.  It consists of a standard sockets-based
       interface for user space processes and an internal kernel API for
       kernel modules.
       
How to use nftables firewall:       
```shell
nft
mnl.c:60: Unable to initialize Netlink socket: Protocol not supported
```
Fix: Reboot after system upgrade
```
[fidit@archlinux ~]$ find /lib/modules/5.15.36-1-lts/ -name \*tables\*
/lib/modules/5.15.36-1-lts/kernel/net/bridge/netfilter/ebtables.ko.zst
/lib/modules/5.15.36-1-lts/kernel/net/ipv4/netfilter/arp_tables.ko.zst
/lib/modules/5.15.36-1-lts/kernel/net/ipv4/netfilter/ip_tables.ko.zst
/lib/modules/5.15.36-1-lts/kernel/net/ipv6/netfilter/ip6_tables.ko.zst
/lib/modules/5.15.36-1-lts/kernel/net/netfilter/nf_tables.ko.zst
/lib/modules/5.15.36-1-lts/kernel/net/netfilter/x_tables.ko.zst
```
What can you do with a firewall?

1. **Create a table that supports IPv4 and IPv6:**
```shell
[fidit@archlinux ~]$ sudo nft add table inet my_table
[fidit@archlinux ~]$ sudo nft list ruleset
table inet my_table {
}

```
2. **Create a chain:**
```
sudo nft add chain inet  my_table my_chain
[fidit@archlinux ~]$ sudo nft add chain inet my_table my_chain2 '{type filter hook input priority 0;}'
[fidit@archlinux ~]$ sudo nft list ruleset
table inet my_table {
	chain my_chain {
	}

	chain my_chain2 {
		type filter hook input priority filter; policy accept;
	}
}

[fidit@archlinux ~]$ sudo nft chain inet my_table my_chain2 '{policy drop;}'
DONT DO THIS! YOU WILL DROP ALL TRAFFIC IN VM AND LOCK YOURSELF OUT:(!
```

* Chain actions:
    * Accept 
    * Drop 
    * Reject

* Chains in iptables:
    * forward, input, output -> FILTER
    * pre/postroute -> NAT

* Chains in nftables:
    * ipv4, inet, ip6

3. **Add rules to the chain**:
**Example 1**
Let's block (drop) all ipv4 traffic from 192.168.122.1:80
Hint: http uses port 80 on layer 5, and uses tcp protocol on l4
```shell
[fidit@archlinux ~]$ sudo nft add rule inet my_table my_chain2 ip saddr 192.168.122.1 tcp dport 80 drop

pacman -S nginx

[fidit@archlinux ~]$ sudo systemctl start nginx

[fidit@archlinux ~]$ systemctl status nginx

```
**Example 2**
Explicitly Enable access to TCP port 80 from a local address
```shell
sudo nft add rule inet my_table my_chain2 ip saddr 127.0.0.1 tcp dport 80 accept
```

**Example 3**
Add rule that discards all udp traffic
```shell
[fidit@archlinux ~]$ sudo nft add rule inet my_table my_chain2 ip protocol udp drop
```
**Example 4**
Disable ssh from IP address 13.198.209.200
```shell
sudo nft add rule inet my_table my_chain2 ip saddr 193.198.209.200 tcp dport 22 reject
```
**Example 5**
Disable connection from local to address 51.158.166.78 port 80
```shell
sudo nft add chain inet my_table my_chain3 '{type filter hook output priority 0;}'

sudo nft add rule inet my_table my_chain3 ip daddr 51.158.166.73 tcp dport 80 drop
```
**Example 6**
Block access to the address 120.240.11.25 from address 25.41.36.127.
```shell
sudo nft add chain inet my_table my_chain4 '{type filter hook forward priority 0;}'

sudo nft add rule inet my_table my_chain4 ip saddr 25.41.36.127 ip daddr 121.240.11.25 drop
```

# **[Infrastructure automation using Ansible](https://www.ansible.com)**
Tired of doing all the work manually? No problem, use infrastructure automation tools as: Terraform, Ansible, Puppet, Chef, SaltStack.. whatever you prefer.

We will try Red-Hat Ansible.
Let's install it:
```
pacman -S ansible ansible-lint code
```
We write ansible playbooks in VS Code with extension -> Remote Development v0.21.0

Process goes as following:
* Host (VS Code on host) connects remotely with extensions to Virtual Machine 1 that has Ansible installed.
* Virtual Machine 1 manages Virtual Machine 2 using Ansible

How to start?
1. In VS Code do ssh to your VM1.
2. Then edit /etc/ansible/hosts file as following:
```
[me]
192.168.122.225 -- your IP

[others]
192.168.122.13 -- IP of VM2
```
3. You can't ping VM's by authenticating with password because it only works with SSH keys! That's why you need to generate SSH keys on both VM's and add the public keys in each other to .ssh/allowed_hosts on VM1 and VM2

4. Create a playbook.yaml file in VS Code:
    * This playbook will perform a full system upgrade on both VM's
```
---
- name: All hosts up-to-date
  hosts: me others
  become: yes
  
  tasks:
    - name: full system upgrade
      pacman:
        update_cache: yes
        upgrade: yes
```
[More on Ansible playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)

Now, let's install Apache HTTP server on VM2 and a postgresql DB on VM1:
**playbook1.yaml:**
```yaml
---
- name: Install apache
  hosts: others
  become: yes
  
  tasks:
    - name: install apache
      pacman:
        name: apache
        state: latest
    - name: goooo apachee
      ansible.builtin.service:
        name: httpd
        state: started
```
**playbook2.yaml**
```yaml
- name: install postgresql
  hosts: me
  become: yes

  tasks:
  - name: install postgresql
    pacman:
      name: postgresql
      state: latest

  - name: gooo postgresql
    ansible.builtin.service:
      name: postgresql
      state: started
```

**Playbook to install Wordpress**
```yaml
---
- name: Update web servers
  hosts: lokalna
  become: yes
  vars:
    ime_baze: mojabaza
    ime_korisnika: mojkorisnik
    passwd: 1234
  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.pacman:
      name: apache
      state: latest
  - name: Ensure that apache is started
    ansible.builtin.service:
      name: httpd
      state: started
#  - name: Download index.html from group.miletic.net
 #   ansible.builtin.command:
  #    cmd: curl -o /srv/http/index.html https://group.miletic.net/index.html
  - name: 
    ansible.builtin.command:
      cmd: curl -o /srv/http/index.php https://www.miletic.net/index.txt
  - name: Ensure php-fpm is at the latest version
    ansible.builtin.pacman:
      name: php-fpm
      state: latest
  - name: Ensure that php-fpm is started
    ansible.builtin.service:
      name: php-fpm
      state: started
  - name: Copy httpd.conf
    ansible.builtin.copy:
      src: ./httpd.conf
      dest: /etc/httpd/conf/httpd.conf
  - name: Copy php-fpm.conf
    ansible.builtin.copy:
      src: ./php-fpm.conf
      dest: /etc/httpd/conf/extra/php-fpm.conf
  - name: Restart apache
    ansible.builtin.service:
      name: httpd
      state: restarted
  - name: pymysql
    ansible.builtin.pacman:
      name: python-pymysql
      state: latest
  - name: Ensure mariadb is at the latest version
    ansible.builtin.pacman:
      name: mariadb
      state: latest
  - name: Configure mariadb
    ansible.builtin.command:
      cmd: mariadb-install-db --user=mysql --auth-root-authentication-method=normal --basedir=/usr --datadir=/var/lib/mysql
  - name: Ensure that mariadb is started
    ansible.builtin.service:
      name: mariadb
      state: started
  - name: Create database user with name 'bob' and password '12345' with all database privileges
    community.mysql.mysql_user:
      name: "{{ime_korisnika}}"
      password: "{{passwd}}"     
      priv: '*.*:ALL'
      state: present
  - name: Create a new database mojabaza
    community.mysql.mysql_db:
      check_implicit_admin: yes
      login_user: "{{ime_korisnika}}"
      login_password: "{{passwd}}"      
      name: "{{ime_baze}}"
      state: present
  - name: install wordpress
    ansible.builtin.command:
      cmd: curl -o /tmp/wordpress.tar.gz  https://hr.wordpress.org/latest-hr.tar.gz      
  - name: wp
    ansible.builtin.command:
      cmd: tar xvf /tmp/wordpress.tar.gz -C /tmp
  - name: wp       
    ansible.builtin.command:      
      cmd: rm -r /srv/http
  - name: a
    ansible.builtin.command:
      cmd: mv /tmp/wordpress /srv/http
  - name: aaa
    ansible.builtin.template:
      src: ./wp-config.php.j2
      dest: /srv/http/wp-config.php
```

**Playbook to install mediawiki**
```yaml
---
- name: Update web servers
  hosts: lokalna
  become: yes
  vars:
    ime_baze: mojabaza
    ime_korisnika: mojkorisnik
    passwd: 1234
  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.pacman:
      name: apache
      state: latest
  - name: Ensure that apache is started
    ansible.builtin.service:
      name: httpd
      state: started
  - name: Ensure php-fpm is at the latest version
    ansible.builtin.pacman:
      name: php-fpm
      state: latest
  - name: Ensure that php-fpm is started
    ansible.builtin.service:
      name: php-fpm
      state: started
  - name: Copy httpd.conf
    ansible.builtin.copy:
      src: ./httpd.conf
      dest: /etc/httpd/conf/httpd.conf
  - name: Copy php-fpm.conf
    ansible.builtin.copy:
      src: ./php-fpm.conf
      dest: /etc/httpd/conf/extra/php-fpm.conf
  - name: Restart apache
    ansible.builtin.service:
      name: httpd
      state: restarted

  - name: Install postgresql
    ansible.builtin.pacman:
      name: postgresql
      state: latest

  - name: Install postgresql
    ansible.builtin.pacman:
      name: python-psycopg2

  #- name: Configure postgresql
  #  become_user: postgres
   # ansible.builtin.command:
    #  cmd: initdb -D /var/lib/postgres/data

  - name: Ensure that postgres is started
    ansible.builtin.service:
      name: postgresql
      state: started

  - name: Create database
    become: yes
    become_user: postgres
    postgresql_db:
      name: "{{ime_baze}}"


  - name: create db user
    become: yes
    become_user: postgres       
    postgresql_user:
      state: present
      name: "{{ime_korisnika}}"
      password: "{{passwd}}"
  

  - name: Create a new database mojabaza
    become: yes
    become_user: postgres    
    postgresql_privs:
      type: database
      database: "{{ ime_baze }}"
      roles: "{{ ime_korisnika }}"
      grant_option: no
      privs: all

  - name: install mediawiki
    ansible.builtin.command:
      cmd: curl -o /tmp/mediawiki.tar.gz  https://releases.wikimedia.org/mediawiki/1.37/mediawiki-1.37.2.tar.gz      
  - name: mwiki
    ansible.builtin.command:
      cmd: tar xvf /tmp/mediawiki.tar.gz -C /tmp

  #- name: a
   # ansible.builtin.command:
    #  cmd: mv /tmp/mediawiki-1.37.2 /srv/http

  - name: caarl
    ansible.builtin.command:
      cmd: curl -o myconf.php.j2 https://gist.github.com/5154384.git

  - name: aaa
    ansible.builtin.template:
      src: ./myconf.php.j2
      dest: /srv/http/LocalSettings.php

  - name: Install PHP packages
    vars: 
      ekstenzija: iconv
    ansible.builtin.template:
      src: ./phpextconf.conf.j2
      dest: /etc/php/php-fpm.d/{{ ekstenzija }}.conf
```

* How to add php extensions?
  * php-fpm: /etc/php/php-fpm.d/gd.conf or /etc/php/php.ini  

Ansible can do many more things, check out [Ansible extensions](https://docs.ansible.com/ansible/latest/collections/) 

## View ip info:

```shell
cat /usr/local/bin/ipinfo
#!/bin/bash
curl -s https://ipinfo.io/$1 | grep -v "{" | grep -v "}"
```
