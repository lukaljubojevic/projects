# Instalacija Matomo aplikacije za web analitiku

Matomo - wiki:
```
Matomo is the most common free and open source web analytics application to track online visits to one or more websites and display reports on these visits for analysis.

Features
Matomo is developed by a team of international developers and runs on a PHP/MySQL webserver. It has been translated into 54 languages.
```

1. Preuzeo sam archlinux cloud VM image s linka: https://gitlab.archlinux.org/archlinux/arch-boxes/-/jobs/78332/artifacts/browse/output ; stavio ga u novu mapu i preimenovao u mojarch.qcow2.
Za taj arch cloud img izvršio naredbu:
```shell
qemu-img resize mojarch.qcow2 20G
```
2. Stvorio sam dvije prazne datoteke: user-data i meta-data. Datoteka meta-data ostaje prazna,a user-data sam ispunio sadrzajem:

touch user-data
touch meta-data
nano user-data
```
user-data:
#cloud-config
users:
  - default

system_info:
   default_user:
     name: luka
     plain_text_passwd: 'passw'
     gecos: arch Cloud User
     groups: [wheel, adm]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
     lock_passwd: False
```
Nakon toga sam kreirao cloud init sliku naredbom:
```
xorriso -as genisoimage -output cloud-init.iso -volid CIDATA -joliet -rock user-data meta-data
```
3. Stvaranje Virtual Machine-a
- new VM - manual install - os: archlinux - ram 2048mb cpus 2 - select or create custom storage - browse local - mojarch.qcow2 - customize configuration before install - finish - add hardware - storage - device type: CDROM - select or create - browse local - cloud-init.iso - finish
- add hardware - storage - 20gb type: disk (DEFAULT)
- add hardware - storage - 20gb type: disk (DEFAULT) 

Nakon svega BEGIN INSTALLATION

- Prije nastavka sam u terminalu pomoću "ip addr" saznao ip adresu (eth0) koja mi je potrebna da ju upisem u nginx.conf 

4. Virtual Machine, stvaranje ZFS particije 
```
ssh luka@"ip adresa"
passw

sudo pacman -Syyu 
sudo reboot

ssh luka@ip
passw

sudo pacman -S ansible
sudo fdisk -l
OUTPUT:
[luka@archlinux ~]$ sudo fdisk -l
Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 45D1DB22-0E35-4E59-A903-177733C73071

Device     Start      End  Sectors Size Type
/dev/vda1   2048     4095     2048   1M BIOS boot
/dev/vda2   4096 41943006 41938911  20G Linux filesystem

Disk /dev/vdb: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/vdc: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes




[luka@archlinux ~]$ sudo fdisk /dev/vdb

OUTPUT:
Welcome to fdisk (util-linux 2.38.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x99d30929.

Command (m for help): g
Created a new GPT disklabel (GUID: 74ACF689-119A-E24F-AB04-03690340520E).

Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-41943006, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943006, default 41940991): 

Created a new partition 1 of type 'Linux filesystem' and of size 20 GiB.

Command (m for help): t
Selected partition 1
Partition type or alias (type L to list all): 157
Changed type of partition 'Linux filesystem' to 'Solaris /usr & Apple ZFS'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.




sudo fdisk /dev/vdc
[luka@archlinux ~]$ sudo fdisk /dev/vdc

OUTPUT:
Welcome to fdisk (util-linux 2.38.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x8f00618d.

Command (m for help): g
Created a new GPT disklabel (GUID: CCBAFB6F-75FF-014F-B7CD-669A87ABF545).

Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-41943006, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943006, default 41940991): 

Created a new partition 1 of type 'Linux filesystem' and of size 20 GiB.

Command (m for help): t
Selected partition 1
Partition type or alias (type L to list all): 157
Changed type of partition 'Linux filesystem' to 'Solaris /usr & Apple ZFS'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Nakon toga sam izišao iz virtualke radi prosljedivanja datoteka.


5. Prosljedivanje datoteka, yml-ova i config-ova u Virtual Machine i pokretanje playbook-a za ZFS
```
scp SKvatrozidMDB.yml ZFS.yml nftables.conf sigurnosnokopiranje.service sigurnosnokopiranje.timer sigurnosnokopiranje.sh nginx.conf php.ini Matomo.yml luka@192.168.122.XX:/home/luka

luka@192.168.122.253's password:
SKvatrozidMDB.yml                                                                100% 3148     7.4MB/s   00:00    
ZFS.yml                                                                          100% 2378     8.0MB/s   00:00    
nftables.conf                                                                    100% 1054     4.6MB/s   00:00    
sigurnosnokopiranje.service                                                      100%  123     1.0MB/s   00:00    
sigurnosnokopiranje.timer                                                        100%  149     1.1MB/s   00:00    
sigurnosnokopiranje.sh                                                           100%   70   549.2KB/s   00:00    
nginx.conf                                                                       100%  739     5.1MB/s   00:00        
php.ini                                                                          100%   71KB  66.3MB/s   00:00    
Matomo.yml                                                                       100% 1425     9.5MB/s   00:00 
```

Nakon toga ulazimo u VM i provjeravamo jesu li sve datoteke unutra
```
ssh luka@ip
passw
ls
```
Ako su sve datoteke proslijedene mozemo nastaviti dalje:


```
[luka@archlinux ~]$ ansible-playbook ZFS.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'
[WARNING]: packaging Python module unavailable; unable to validate collection Ansible
version requirements

PLAY [Instalacija ZFS-a] ********************************************************************

TASK [Gathering Facts] **********************************************************************
ok: [localhost]

TASK [Download all packages] ****************************************************************
changed: [localhost]

TASK [preuzimanje zfs zakrpe s AUR repozitorija] ********************************************
changed: [localhost]

TASK [preuzimanje zfs paketa s AUR repozitorija] ********************************************
changed: [localhost]

TASK [preuzimanje zfs-utils zakrpi s AUR repozitorija] **************************************
changed: [localhost]

TASK [preuzimanje zfs-utils zakrpi s AUR repozitorija] **************************************
changed: [localhost]

TASK [preuzimanje zfs-utils paketa s AUR repozitorija] **************************************
changed: [localhost]

TASK [dodavanje pacman zfs kljuca sudo] *****************************************************
changed: [localhost]

TASK [dodavanje pacman zfs kljuca user] *****************************************************
changed: [localhost]

TASK [razvijanje zfs-utils] *****************************************************************
changed: [localhost]

TASK [razvijanje zfs paketa] ****************************************************************
changed: [localhost]

TASK [instalacija zfs-utilsa] ***************************************************************
changed: [localhost]

TASK [instalacija zfsa] *********************************************************************
changed: [localhost]

PLAY RECAP **********************************************************************************
localhost                  : ok=13   changed=11   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
ZFS.yml:
```
---
    - name: Instalacija ZFS-a
      hosts: localhost
      become: yes
      tasks:
      - name: Download all packages
        ansible.builtin.pacman:
          name: 
            - make
            - fakeroot
            - patch
            - autoconf 
            - linux-headers
            - git
            - automake
            - dkms 
            - nano
          state: latest
      - name: preuzimanje zfs zakrpe s AUR repozitorija
        ansible.builtin.command:
          cmd: curl -o 0001-only-build-the-module-in-dkms.conf.patch "https://aur.archlinux.org/cgit/aur.git/plain/0001-only-build-the-module-in-dkms.conf.patch?h=zfs-dkms"
      - name: preuzimanje zfs paketa s AUR repozitorija
        ansible.builtin.command:     
          cmd: curl -o PKGBUILD "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-dkms"
      - name: preuzimanje zfs-utils zakrpi s AUR repozitorija
        ansible.builtin.command:      
          cmd: curl -o zfs.initcpio.install "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.install?h=zfs-utils"      
      - name: preuzimanje zfs-utils zakrpi s AUR repozitorija
        ansible.builtin.command:     
          cmd: curl -o zfs.initcpio.hook "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.hook?h=zfs-utils"    
      - name: preuzimanje zfs-utils paketa s AUR repozitorija
        ansible.builtin.command:      
          cmd: curl -o PKGBUILD-utils "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-utils"     
      - name: dodavanje pacman zfs kljuca sudo
        ansible.builtin.command:
          cmd: gpg --receive-keys 6AD860EED4598027 
      - name: dodavanje pacman zfs kljuca user
        ansible.builtin.command:
          cmd: gpg --receive-keys 6AD860EED4598027 
        become_user: luka
      - name: razvijanje zfs-utils
        ansible.builtin.command:
          cmd: makepkg -p PKGBUILD-utils
        become_user: luka
      - name: razvijanje zfs paketa
        ansible.builtin.command:      
          cmd: makepkg   
        become_user: luka
      - name: instalacija zfs-utilsa
        ansible.builtin.pacman:
          name:
            - zfs-utils-2.1.5-1-x86_64.pkg.tar.zst
          state: present
      - name: instalacija zfsa
        ansible.builtin.pacman:
          name:
            - zfs-dkms-2.1.5-1-any.pkg.tar.zst
          state: present

```
Nakon ovoga potrebno je resetirati VM "sudo reboot" prije stvaranje ZFS polja.

6. Stvaranje ZFS polja.

ssh luka@ip ; passw
```
[luka@archlinux ~]$ sudo modprobe zfs
[luka@archlinux ~]$ sudo zpool create -m /mojezrcalo mojezrcalo /dev/vdb1 /dev/vdc1
[luka@archlinux ~]$ ss

"sudo zpool status" !!!!!
"""

  pool: mojezrcalo
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        mojezrcalo  ONLINE       0     0     0
          vdb1      ONLINE       0     0     0
          vdc1      ONLINE       0     0     0

errors: No known data errors

```

7. Pokretanje ansible-a za firewall, database i rsync.
```
[luka@archlinux ~]$ ansible-playbook SKvatrozidMDB.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not
match 'all'
[WARNING]: packaging Python module unavailable; unable to validate collection Ansible version requirements

PLAY [Sigurnosno kopiranje, baza podataka i vatrozid] *************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [localhost]

TASK [Download all packages] **************************************************************************************
ok: [localhost]

TASK [kopiranje datoteka servisa] *********************************************************************************
changed: [localhost]

TASK [kopiranje datoteka servisa] *********************************************************************************
changed: [localhost]

TASK [koprianje skripte za backup] ********************************************************************************
ok: [localhost]

TASK [ponovno ucitavanje servisa] *********************************************************************************
changed: [localhost]

TASK [omogucavanje brojaca] ***************************************************************************************
changed: [localhost]

TASK [omogucavanje servisa] ***************************************************************************************
changed: [localhost]

TASK [omogucavanje brojaca] ***************************************************************************************
changed: [localhost]

TASK [omogucavanje servisa] ***************************************************************************************
changed: [localhost]

TASK [pokretanje brojaca] *****************************************************************************************
changed: [localhost]

TASK [mkdir /var/www/html] ****************************************************************************************
ok: [localhost]

TASK [start unit] *************************************************************************************************
changed: [localhost]

TASK [konfiguracija vatrozida] ************************************************************************************
ok: [localhost]

TASK [instalacija mysqlclienta] ***********************************************************************************
changed: [localhost]

TASK [instalacija mariadb dbmsa] **********************************************************************************
changed: [localhost]

TASK [omogucavanje mariadb servisa] *******************************************************************************
changed: [localhost]

TASK [omogucavanje mariadb servisa] *******************************************************************************
changed: [localhost]

TASK [Stvaranje matomo bp] ****************************************************************************************
changed: [localhost]

TASK [Stvaranje korisnika] ****************************************************************************************
changed: [localhost]

TASK [Postavljanje mariadb root lozinke] **************************************************************************
changed: [localhost]

PLAY RECAP ********************************************************************************************************
localhost                  : ok=21   changed=14   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

9. Pokretanje ansible-a koji build-a Matomo i instalira sav SW. Nakon toga ansible prebacuje gotove config-ove na direktorije.
```
sudo pacman -S php-fpm

[luka@archlinux ~]$ ansible-playbook Matomo.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Instalacija matomo web aplikacije] **********************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [localhost]

TASK [Preuzimanje paketa] *************************************************************************************************************
ok: [localhost]

TASK [konfiguracija posluzitelja nginx] ***********************************************************************************************
ok: [localhost]

TASK [kopiranje php konfiguracijske datoteke] *****************************************************************************************
ok: [localhost]

TASK [preuzimanje matomo repozitorija] ************************************************************************************************
changed: [localhost]

TASK [premjestanje matomo web aplikacije u nginx posluziteljski direktorij] ***********************************************************
changed: [localhost]

TASK [omogucavanje nginxa] ************************************************************************************************************
changed: [localhost]

TASK [pokretanje nginxa] **************************************************************************************************************
changed: [localhost]

```

Ovaj dio sam pokusao napraviti putem ansible-a, ali composer package manager mi nije radio pa sam napravio rucno (treba ga instalirati u specifičan direktorij, a ansible koristi userov home direktorij).

11. Instalacija php composer package manager-a
```
"cd /usr/share/webapps/matomo/matomo"
[luka@archlinux matomo]$ "sudo curl -sS https://getcomposer.org/installer | sudo php"

All settings correct for using Composer
Downloading...

Composer (version 2.4.1) successfully installed to: /usr/share/webapps/matomo/matomo/composer.phar
Use it: php composer.phar

"sudo php composer.phar install --ignore-platform-req=ext-gd"

Generating autoload files
32 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```
Dozvole direktorija:
```
sudo chown -R http:http /usr/share/webapps/matomo/matomo
sudo find /usr/share/webapps/matomo/matomo -type f -exec chmod 644 {} \;
sudo find /usr/share/webapps/matomo/matomo -type d -exec chmod 755 {} \;
sudo chown -R http:http /usr/share/webapps/matomo/matomo/tmp/*
```
Ponovno pokretanje nginx-a i php-fpm-a.
```
ps -aux | grep fpm
sudo killall php-fpm
sudo systemctl restart nginx
sudo php-fpm
```
10. Pristupanje na 192.168.122.XX/matomo/index.php kroz browser. 

Cesto tokom instalacije je dolazilo do greske koju sam rijesio na sljedeci nacin
```
sudo nano /etc/php/php.ini
```
U php.ini sam pod "extensions" odkomentirao mysqli.

Nakon toga sam napravio ponovni reset:
```
sudo killall php-fpm
sudo systemctl restart nginx
sudo php-fpm
```

Ponovnim pristupom na 192.168.122.XX/matomo/index.php

```
SETUP 
  Database setup
    Login: matomo
    password: passw
    database name: matomo

SUPERUSER
    login: matomo
    password: password
    password repeat: password
    b@b.hr
```


![](https://i.imgur.com/5DGCEzw.png)
![](https://i.imgur.com/0ZST5YP.png)
![](https://i.imgur.com/sklgS59.png)
![](https://i.imgur.com/XmnkWNo.png)



Konfiguracijska datoteka php.ini
```
[PHP]
...
;;;;;;;;;;;;;;;;;;;;;;
; Dynamic Extensions ;
;;;;;;;;;;;;;;;;;;;;;;

; If you wish to have an extension loaded automatically, use the following
; syntax:
;
;   extension=modulename
;
; For example:
;
;   extension=mysqli
;
; When the extension library to load is not located in the default extension
; directory, You may specify an absolute path to the library file:
;
;   extension=/path/to/extension/mysqli.so
;
; Note : The syntax used in previous PHP versions ('extension=<ext>.so' and
; 'extension='php_<ext>.dll') is supported for legacy reasons and may be
; deprecated in a future PHP major version. So, when it is possible, please
; move to the new ('extension=<ext>) syntax.
;
; Notes for Windows environments :
;
; - Many DLL files are located in the extensions/ (PHP 4) or ext/ (PHP 5+)
;   extension folders as well as the separate PECL DLL download (PHP 5+).
;   Be sure to appropriately set the extension_dir directive.
;
;extension=bz2
;extension=curl
;extension=ffi
;extension=ftp
;extension=fileinfo
extension=iconv
extension=gd
extension=gd2
;extension=gettext
;extension=gmp
;extension=intl
;extension=imap
;extension=ldap
;extension=mbstring
;extension=exif      ; Must be after mbstring as it depends on it
extension=mysqli
;extension=oci8_12c  ; Use with Oracle Database 12c Instant Client
;extension=odbc
;extension=openssl
;extension=pdo_firebird
extension=pdo_mysql
;extension=pdo_oci
;extension=pdo_odbc
;extension=pdo_pgsql
;extension=pdo_sqlite
;extension=pgsql
;extension=shmop

; The MIBS data available in the PHP distribution must be installed.
; See http://www.php.net/manual/en/snmp.installation.php
;extension=snmp

;extension=soap
;extension=sockets
;extension=sodium
;extension=sqlite3
;extension=tidy
;extension=xmlrpc
;extension=xsl
...
```
Konfiguracijska datoteka nginx.conf
```
user       http http;  ## Default: nobody
worker_processes  5;  ## Default: 1
worker_rlimit_nofile 8192;

events {
  worker_connections  4096;  ## Default: 1024
}

http{
    include /etc/nginx/mime.types;

    server
    {
        listen      80;
        listen      [::]:80;
        server_name 192.168.122.XXX;
        root        /usr/share/webapps/matomo/;
        index       index.php;

        location ~ ^/(\.git/|config/|core/|lang/|tmp/)
        {
            return  403;
        }

        location ~ \.php$
        {
            try_files   $uri =404;

            # FastCGI
            include         fastcgi.conf;
            fastcgi_pass    unix:/run/php-fpm/php-fpm.sock;
            fastcgi_index   index.php;
        }

        location ~ \.(avi|css|eot|gif|htm|html|ico|jpg|js|json|mp3|mp4|ogg|png|svg|ttf|wav|woff|woff2)$
        {
            try_files   $uri =404;
        }

        location ~ ^/(libs/|misc/|node_modules/|plugins/|vendor/)
        {
            return  403;
        }
    }
}

```
Konfiguracijska datoteka nftables.conf
```
#!/usr/bin/nft -f
# vim:set ts=2 sw=2 et:
# IPv4/IPv6 Simple & Safe firewall ruleset.
# More examples in /usr/share/nftables/ and /usr/share/doc/nftables/examples/.
table inet filter
delete table inet filter
table inet filter {
    chain input {
        type filter hook input priority filter
        policy drop
        ct state invalid drop comment "early drop of invalid connections"
        ct state {established, related} accept comment "allow tracked connections"
        iifname lo accept comment "allow from loopback"
        ip protocol icmp accept comment "allow icmp"
        meta l4proto ipv6-icmp accept comment "allow icmp v6"
        tcp dport ssh accept comment "allow sshd"
        pkttype host limit rate 5/second counter reject with icmpx type admin-prohibited
        counter
    }
    chain forward {
        type filter hook forward priority filter
        policy drop
    }
}
table inet moja_tablica {
    chain moj_lanac{
        type filter hook input priority 0
        policy drop
        tcp dport {22, 80, 443} accept
    }
}
```

User-data
```
#cloud-config
users:
  - default

system_info:
   default_user:
     name: luka
     plain_text_passwd: 'passw'
     gecos: arch Cloud User
     groups: [wheel, adm]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
     lock_passwd: False
```

Brojač
```
[Unit]
Description=timer za sigurnosno kopiranje
            
[Timer]
OnCalendar=weekly
Persistent=true
            
[Install]
WantedBy=timers.target
```

Servis
```
[Unit]
Description=sigurnosno kopiranje
    
[Service]
Type=oneshot
ExecStart=/bin/bash /home/luka/sigurnosnokopiranje.sh

```

Skripta
```
#!/bin/sh
sudo rsync -a --delete --quiet /usr/share/webapps/matomo /mojezrcalo  
```
ZFS.yml
```
---
    - name: Instalacija ZFS-a
      hosts: localhost
      become: yes
      tasks:
      - name: Download all packages
        ansible.builtin.pacman:
          name: 
            - make
            - fakeroot
            - patch
            - autoconf 
            - linux-headers
            - git
            - automake
            - dkms 
            - nano
          state: latest
      - name: preuzimanje zfs zakrpe s AUR repozitorija
        ansible.builtin.command:
          cmd: curl -o 0001-only-build-the-module-in-dkms.conf.patch "https://aur.archlinux.org/cgit/aur.git/plain/0001-only-build-the-module-in-dkms.conf.patch?h=zfs-dkms"
      - name: preuzimanje zfs paketa s AUR repozitorija
        ansible.builtin.command:     
          cmd: curl -o PKGBUILD "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-dkms"
      - name: preuzimanje zfs-utils zakrpi s AUR repozitorija
        ansible.builtin.command:      
          cmd: curl -o zfs.initcpio.install "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.install?h=zfs-utils"      
      - name: preuzimanje zfs-utils zakrpi s AUR repozitorija
        ansible.builtin.command:     
          cmd: curl -o zfs.initcpio.hook "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.hook?h=zfs-utils"    
      - name: preuzimanje zfs-utils paketa s AUR repozitorija
        ansible.builtin.command:      
          cmd: curl -o PKGBUILD-utils "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-utils"     
      - name: dodavanje pacman zfs kljuca sudo
        ansible.builtin.command:
          cmd: gpg --receive-keys 6AD860EED4598027 
      - name: dodavanje pacman zfs kljuca user
        ansible.builtin.command:
          cmd: gpg --receive-keys 6AD860EED4598027 
        become_user: luka
      - name: razvijanje zfs-utils
        ansible.builtin.command:
          cmd: makepkg -p PKGBUILD-utils
        become_user: luka
      - name: razvijanje zfs paketa
        ansible.builtin.command:      
          cmd: makepkg   
        become_user: luka
      - name: instalacija zfs-utilsa
        ansible.builtin.pacman:
          name:
            - zfs-utils-2.1.5-1-x86_64.pkg.tar.zst
          state: present
      - name: instalacija zfsa
        ansible.builtin.pacman:
          name:
            - zfs-dkms-2.1.5-1-any.pkg.tar.zst
          state: present

```

Podesavanje sustava.yml
```
---
    - name: Sigurnosno kopiranje, baza podataka i vatrozid
      hosts: localhost
      become: yes
      tasks:
      - name: Download all packages
        ansible.builtin.pacman:
          name: 
            - rsync
            - nftables
            - mariadb
            - python-pip
          state: latest
      - name: kopiranje datoteka servisa
        ansible.builtin.copy:
          src: sigurnosnokopiranje.timer
          dest: /etc/systemd/system/sigurnosnokopiranje.timer
      - name: kopiranje datoteka servisa
        ansible.builtin.copy:
          src: sigurnosnokopiranje.service
          dest: /etc/systemd/system/sigurnosnokopiranje.service
      - name: koprianje skripte za backup
        ansible.builtin.copy:
          src: sigurnosnokopiranje.sh
          dest: /home/luka/sigurnosnokopiranje.sh
      - name: ponovno ucitavanje servisa
        ansible.builtin.command:
          cmd: systemctl daemon-reload
      - name: omogucavanje brojaca       
        ansible.builtin.command:
           cmd: systemctl unmask sigurnosnokopiranje.timer
      - name: omogucavanje servisa
        ansible.builtin.command:
           cmd: systemctl unmask sigurnosnokopiranje.service
      - name: omogucavanje brojaca  
        ansible.builtin.command:
           cmd: systemctl enable sigurnosnokopiranje.timer
      - name: omogucavanje servisa 
        ansible.builtin.command:
           cmd: systemctl enable sigurnosnokopiranje.service      
      - name: pokretanje brojaca
        ansible.builtin.command:
           cmd: systemctl start sigurnosnokopiranje.timer
      - name: Stvaranje matomo direktorija
        ansible.builtin.file:
           path: /usr/share/webapps/matomo
           state: directory
           mode: '0755'
      - name: start unit   
        ansible.builtin.command:
           cmd: systemctl start sigurnosnokopiranje.service
      - name: konfiguracija vatrozida
        ansible.builtin.copy:
          src: nftables.conf
          dest: /etc/nftables.conf
      - name: instalacija mysqlclienta
        ansible.builtin.command:
          cmd: pip install mysqlclient 
      - name: instalacija mariadb dbmsa
        ansible.builtin.command:      
          cmd: mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql 
      - name: omogucavanje mariadb servisa
        ansible.builtin.command:      
          cmd: systemctl enable mariadb 
      - name: omogucavanje mariadb servisa
        ansible.builtin.command:      
          cmd: systemctl start mariadb           
      - name: Stvaranje matomo bp
        community.mysql.mysql_db:
          name: matomo
          check_implicit_admin: true
          login_unix_socket: /var/run/mysqld/mysqld.sock          
          state: present    
      - name: Stvaranje korisnika
        community.mysql.mysql_user:
          name: matomo
          password: 'passw'
          priv: 'matomo.*:SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER'
          check_implicit_admin: true
          login_unix_socket: /var/run/mysqld/mysqld.sock            
          state: present              
      - name: Postavljanje mariadb root lozinke
        mysql_user:
          login_user: 'root'
          login_host: 'localhost'
          login_password: ''
          check_implicit_admin: true
          login_unix_socket: /var/run/mysqld/mysqld.sock           
          name: root
          password: 'passw'
          state: present 

```

Matomo.yml
```
---
    - name: Instalacija matomo web aplikacije
      hosts: localhost
      become: yes
      tasks:
      - name: Preuzimanje paketa
        ansible.builtin.pacman:
          name: 
            - nginx
            - php
            - php-gd
            - php-fpm
            - composer
            - git
            - git-lfs
          state: latest
      - name: omogucavanje php-fpm servisa
        ansible.builtin.command:
          cmd: systemctl enable php-fpm
      - name: pokretanje php-fpm servisa
        ansible.builtin.command:
          cmd: systemctl start php-fpm
      - name: konfiguracija posluzitelja nginx
        ansible.builtin.copy:
          src: nginx.conf
          dest: /etc/nginx/nginx.conf
      - name: kopiranje php konfiguracijske datoteke
        ansible.builtin.copy:
          src: php.ini
          dest: /etc/php/php.ini
      - name: preuzimanje matomo repozitorija
        ansible.builtin.command:
          cmd: git clone https://github.com/matomo-org/matomo.git
      - name: premjestanje matomo web aplikacije u nginx posluziteljski direktorij
        ansible.builtin.copy:
          src: /home/luka/matomo
          dest: /usr/share/webapps/matomo
          mode: '0755'
          owner: http
          remote_src: yes
      - name: omogucavanje nginxa
        ansible.builtin.command:
          cmd: systemctl enable nginx     
      - name: pokretanje nginxa
        ansible.builtin.command:
          cmd: systemctl start nginx

```
