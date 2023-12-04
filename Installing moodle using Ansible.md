# Automatization of Computer Systems Management
For my Undergraduate thesis at FIDIT Rijeka I want to explore the topic of computer systems management automatization using Ansible.
All my thesis work will be placed here as I work.

* Projektni zadatak prikazuje korake koje je potrebno provesti u jednom virtualnom stroju kako bi se konfiguriralo posluživanje moodle platforme, ali i isto rješeno automatizacijom putem sustava Ansible.

* Virtualni stroj pokreće Archlinux operacijski sustav, koristi zrcaljenje pogona (eng. „mirror“) i automatsko sigurnosno kopiranje moodle direktorija, vatrozid koji blokira sav promet van vrata 80 (HTTP), 443 (SSL, HTTPS) i 22 (SSH), MariaDB sustav za upravljanje bazom podataka, Apache web poslužitelj i PHP7.4 parser.

# Virtualni stroj:
1. Preuzeti sliku sustava [archlinux](https://gitlab.archlinux.org/archlinux/arch-boxes/-/jobs/58259/artifacts/file/output/Arch-Linux-x86_64-cloudimg-20220528.58259.qcow2) za posluživanje u oblaku.
2. Napraviti mapu projekta, preuzetu sliku sustava prekopirati u mapu i preimenovati te stvoriti datoteke user-data i meta-data za sustav pokretanja virtualnih strojeva - cloud-init:
```shell
[lljubojevic@Ljubojevic-PC ~]$ mkdir projekt-urs
[lljubojevic@Ljubojevic-PC ~]$ cd projekt-urs/
[lljubojevic@Ljubojevic-PC projekt-urs]$ nano user-data
[lljubojevic@Ljubojevic-PC projekt-urs]$ touch meta-data
[lljubojevic@Ljubojevic-PC projekt-urs]$ ls
meta-data  user-data
```
* Datoteka user-data je sadržaja:
```shell
#cloud-config
users:
  - default

system_info:
   default_user:
     name: seminar
     plain_text_passwd: '1234'
     gecos: arch Cloud User
     groups: [wheel, adm]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
     lock_passwd: False
```
3. Generirati sliku, u formatu ISO, s konfiguracijskim datotekama za cloud-init:
```shell
[lljubojevic@Ljubojevic-PC projekt-urs]$ xorriso -as genisoimage -output cloud-init.iso -volid CIDATA -joliet -rock user-data meta-data
xorriso 1.5.4 : RockRidge filesystem manipulator, libburnia project.

Drive current: -outdev 'stdio:cloud-init.iso'
Media current: stdio file, overwriteable
Media status : is blank
Media summary: 0 sessions, 0 data blocks, 0 data,  272g free
Added to ISO image: file '/user-data'='/home/lljubojevic/projekt-urs/user-data'
xorriso : UPDATE :       1 files added in 1 seconds
Added to ISO image: file '/meta-data'='/home/lljubojevic/projekt-urs/meta-data'
xorriso : UPDATE :       2 files added in 1 seconds
ISO image produced: 184 sectors
Written to medium : 184 sectors at LBA 0
Writing to 'stdio:cloud-init.iso' completed successfully.
```
4. Povećati prostor za pohranu na preuzetoj slici sustava:
```shell
[lljubojevic@Ljubojevic-PC projekt-urs]$ qemu-img resize myaarch.qcow2 20G
Image resized.
```
5. Stvoriti virtualni stroj u alatu virt-manager s koracima:
* Import existing image - Browse local - odabrati sliku sustava
* Operacijski sustav: archlinux 
* Memorija: 2048 MB - Procesor: 2 jezgre 
* Odabrati "Customize before install", zatim "Add hardware" - Storage - izabrati ISO sliku za cloud-init - tip: cdrom
* Odabrati ponovno "Add hardware" - Storage - kreirati dva nova virtualna diska veličine 20GB - tip: VirtIO Disk
* Begin Installation

6. U sučelju za interakciju s virtualnim strojem QEMU/KVM izvršiti prijavu na sustav i saznati IP adresu virtualnog stroja:
![](https://i.imgur.com/tYg1Lhq.png)

7. Spojiti se s glavnog računala naredbom ssh i ažurirati sustav:
```shell
[lljubojevic@Ljubojevic-PC ~]$ ssh seminar@192.168.122.64
[seminar@archlinux ~]$ sudo pacman -Syyu
```shell
ip address -> 192.168.122.62
ssh seminar@192.168.122.62
sudo pacman -Syyu
```
8. Inicijalizirati diskove i particije za ZFS zrcaljenje pogona:
```shell
[seminar@archlinux ~]$ sudo fdisk /dev/vdb

Welcome to fdisk (util-linux 2.38).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x4f56e4a7.

Command (m for help): g
Created a new GPT disklabel (GUID: AB543153-05B1-0344-939E-0B2E121700D9).

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
```shell
[seminar@archlinux ~]$ sudo fdisk /dev/vdc

Welcome to fdisk (util-linux 2.38).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x0dca6cdc.

Command (m for help): g
Created a new GPT disklabel (GUID: 40E56ED1-5399-F14F-945A-D6F9E5FAC901).

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
9. Instalirati sav potreban softver:
```shell
[seminar@archlinux ~]$ sudo pacman -S make fakeroot patch autoconf linux-headers git automake dkms nano rsync nftables apache mariadb php7 php7-apache graphviz aspell ghostscript clamav php7-pspell  php7-gd php7-intl git
```
10. Dohvatiti ZFS i ZFS-utils izvorni kod iz Arclinux korisničkog repozitorija, ovjeriti ključeve paketa
```shell
[seminar@archlinux zfs]$ curl -o 0001-only-build-the-module-in-dkms.conf.patch "https://aur.archlinux.org/cgit/aur.git/plain/0001-only-build-the-module-in-dkms.conf.patch?h=zfs-dkms"

[seminar@archlinux zfs]$ curl -o PKGBUILD "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-dkms"

[seminar@archlinux zfs]$ curl -o zfs.initcpio.hook "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.hook?h=zfs-utils"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3620  100  3620    0     0  18612      0 --:--:-- --:--:-- --:--:-- 18564
[seminar@archlinux zfs]$ curl -o zfs.initcpio.install "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.install?h=zfs-utils"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2301  100  2301    0     0  10843      0 --:--:-- --:--:-- --:--:-- 10853

[seminar@archlinux zfs]$ curl -o PKGBUILD-utils "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-utils"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2779  100  2779    0     0  12819      0 --:--:-- --:--:-- --:--:-- 12865

[seminar@archlinux zfs]$ sudo gpg --receive-keys 6AD860EED4598027
gpg: keybox '/root/.gnupg/pubring.kbx' created
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 6AD860EED4598027: public key "Tony Hutter (GPG key for signing ZFS releases) <hutter2@llnl.gov>" imported
gpg: Total number processed: 1
gpg:               imported: 1


```
11. Generirati, iz izvornog koda, instalacijske pakete za ZFS i ZFS-utils i instalirati ih pomoću upravitelja paketa:
```shell
[seminar@archlinux zfs]$ makepkg -p PKGBUILD-utils
==> Making package: zfs-utils 2.1.4+65.r05147319b0-1 (Thu 26 May 2022 08:52:27 AM UTC)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
  -> Updating zfs git repo...
Fetching origin
  -> Found zfs.initcpio.install
  -> Found zfs.initcpio.hook
...
==> Creating package "zfs-utils"...
  -> Generating .PKGINFO file...
  -> Generating .BUILDINFO file...
  -> Generating .MTREE file...
  -> Compressing package...
==> Leaving fakeroot environment.
==> Finished making: zfs-utils 2.1.4+65.r05147319b0-1 (Thu 26 May 2022 09:02:19 AM UTC)

[seminar@archlinux zfs]$ makepkg -p PKGBUILD
==> Making package: zfs-dkms 2.1.4+65.r05147319b0-2 (Thu 26 May 2022 09:03:27 AM UTC)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
  -> Updating zfs git repo...
Fetching origin
  -> Found zfs.initcpio.install
  -> Found zfs.initcpio.hook
...
==> Creating package "zfs-dkms"...
  -> Generating .PKGINFO file...
  -> Generating .BUILDINFO file...
  -> Generating .MTREE file...
  -> Compressing package...
==> Leaving fakeroot environment.
==> Finished making: zfs-dkms 2.1.4+65.r05147319b0-2 (Thu 26 May 2022 09:10:19 AM UTC)

[seminar@archlinux zfs]$ sudo pacman -U zfs-dkms-2.1.4+65.r05147319b0-2-any.pkg.tar.zst zfs-utils-2.1.4+65.r05147319b0-1-x86_64.pkg.tar.zst 
```
12. Ponovno pokrenuti sustav
13. Učitati zfs modul jezgre:
```shell
[seminar@archlinux ~]$ sudo modprobe zfs
```
14. Kreirati zrcaljenje pogona pomoću upravitelja zpool:
```shell
[seminar@archlinux ~]$ sudo zpool create -m /mymirror mymirror mirror /dev/vdb1 /dev/vdc1
[seminar@archlinux ~]$ zpool status
  pool: mymirror
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        mymirror    ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            vdb1    ONLINE       0     0     0
            vdc1    ONLINE       0     0     0

errors: No known data errors
```
15. Kreirati systemd jedinicu, Bash skriptu (koja pokreće sigurnosnu kopiju) i brojač vremena za automatsko sigurnosno kopiranje:
```shell
[seminar@archlinux ~]$ cd /etc/systemd/system/
[seminar@archlinux system]$ sudo nano backup.timer
[seminar@archlinux system]$ sudo nano backup.service
```
* Datoteka backup.timer je sadržaja:
```shell
[Unit]
Description=test

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```
* Datoteka backup.service je sadržaja:
```shell
[Unit]
Description=backup

[Service]
Type=oneshot
ExecStart=/bin/bash /home/seminar/backup.sh
```
* Datoteka backup.sh je sadržaja:
```shell
#!/bin/sh
sudo rsync -a --delete --quiet /var/www/html /mymirror
```
16. Omogućiti izvršavanje skripte, brojača i jedinice:
```shell
[seminar@archlinux ~]$ sudo systemctl daemon-reload
[seminar@archlinux ~]$ sudo systemctl unmask backup.timer
[seminar@archlinux ~]$ sudo systemctl unmask backup.service
[seminar@archlinux ~]$ sudo systemctl enable backup.timer
[seminar@archlinux ~]$ sudo systemctl enable backup.service
[seminar@archlinux ~]$ sudo systemctl start backup.timer
[seminar@archlinux ~]$ sudo chmod u+x backup.sh
```
17. Podesiti vatrozid s tablicom "moja_pravila" i lancem "moj_lanac" dodati pravilo da se sav promet prema poslužitelju, osim na vratima 80 (HTTP), 22 (SSH) i 443 (HTTPS/SSL), blokira:
```shell
[seminar@archlinux ~]$ sudo nft add table inet my_rules
[seminar@archlinux ~]$ sudo nft add chain inet my_rules my_chain '{type filter hook input priority 0; policy drop;}'
[seminar@archlinux ~]$ sudo nft add rule inet my_rules my_chain  tcp dport {22, 80, 443} accept
```
18. Inicijalizirati MariaDB SUBP, Konfigurirati bazu podataka i korisnika „moodle“, korisniku omogućiti prava pristupa bazi podataka moodle, kreirati lozinku korijenskog korisnika
```shell
[seminar@archlinux ~]$ sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
[seminar@archlinux ~]$ sudo systemctl start mariadb
[seminar@archlinux ~]$ mysql -u root -p
MariaDB [(none)]> CREATE USER 'moodle'@'localhost' IDENTIFIED BY 'vrloTajniPass';
Query OK, 0 rows affected (0.012 sec)

MariaDB [(none)]> CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodle.* TO 'moodle'@'localhost';
Query OK, 0 rows affected (0.009 sec)

MariaDB [(none)]> ALTER USER 'root'@'localhost' IDENTIFIED BY 'mypass';
Query OK, 0 rows affected (0.009 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)
```
19. Promijeniti konfiguracijsku datoteku Apache poslužitelja tako da se uključe PHP moduli i postavi se direktorij posluživanja /var/www/html:
```shell
[seminar@archlinux conf]$ sudo nano httpd.conf 
DocumentRoot "/var/www/html"
SetHandler application/x-httpd-php

LoadModule php7_module modules/libphp7.so
AddHandler php-script .php

Include conf/extra/php7_module.conf
#LoadModule mpm_event_module modules/mod_mpm_event.so
```
20. Preuzeti moodle s Git repozitorija, premjestiti ga u /var/www/html
```shell
[seminar@archlinux ~]$ mkdir moodle
[seminar@archlinux ~]$ cd moodle
[seminar@archlinux moodle]$ sudo git clone git://git.moodle.org/moodle.git
[seminar@archlinux moodle]$ sudo cp -R moodle /var/www/html/moodle
[seminar@archlinux moodle]$ sudo mkdir /var/moodledata
[seminar@archlinux moodle]$ sudo chmod -R 777 /var/www/moodledata
[seminar@archlinux moodle]$ sudo chmod -R 0755 /var/www/html/moodle
[seminar@archlinux html]$ sudo chown -R http /var/moodledata
```
21. Promijeniti php konfiguracijsku datoteku na način da se uključe ekstenzije potrebne za moodle, uključiti keširanje:
```shell
[seminar@archlinux php]$ sudo nano /etc/php7/php.ini
#LoadModule mpm_event_module modules/mod_mpm_event.so
...
zend_extension=opcache
extension=curl
extension=gd
extension=sodium
extension=exif
extension=iconv
extension=intl
extension=ldap
extension=mysqli
extension=pspell
extension=soap
extension=xmlrpc
extension=zip
...
max_input_vars = 6000
..
opcache.enable=1
```
22. Pokrenuti Apache poslužitelj
```shell
[seminar@archlinux conf]$ sudo systemctl start httpd
```
24. Instalirati moodle putem web sučelja:
![](https://i.imgur.com/LtNTR6g.png)
![](https://i.imgur.com/z3RO2NO.png)
![](https://i.imgur.com/2AQarqD.png)
![](https://i.imgur.com/isjrU9l.png)
![](https://i.imgur.com/yWeuAcB.png)
![](https://i.imgur.com/vyz7llL.png)
![](https://i.imgur.com/o9PmVD3.png)
![](https://i.imgur.com/w0lkSMj.png)
![](https://i.imgur.com/h0hxFxH.png)
![](https://i.imgur.com/COx7RgL.png)
![](https://i.imgur.com/1gOee10.png)

25. Spremiti moodle konfiguraciju u /var/www/html/config.php

# Ansible
## Koraci:
1. Preuzeti sliku sustava [archlinux](https://gitlab.archlinux.org/archlinux/arch-boxes/-/jobs/58259/artifacts/file/output/Arch-Linux-x86_64-cloudimg-20220528.58259.qcow2) za posluživanje u oblaku.
2. Napraviti mapu projekta, preuzetu sliku sustava prekopirati u mapu i preimenovati te stvoriti datoteke user-data i meta-data za sustav pokretanja virtualnih strojeva - cloud-init:
```shell
[lljubojevic@Ljubojevic-PC ~]$ mkdir projekt-urs
[lljubojevic@Ljubojevic-PC ~]$ cd projekt-urs/
[lljubojevic@Ljubojevic-PC projekt-urs]$ nano user-data
[lljubojevic@Ljubojevic-PC projekt-urs]$ touch meta-data
[lljubojevic@Ljubojevic-PC projekt-urs]$ ls
meta-data  user-data
```
* Datoteka user-data je sadržaja:
```shell
#cloud-config
users:
  - default

system_info:
   default_user:
     name: seminar
     plain_text_passwd: '1234'
     gecos: arch Cloud User
     groups: [wheel, adm]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
     lock_passwd: False
```
3. Generirati sliku, u formatu ISO, s konfiguracijskim datotekama za cloud-init:
```shell
[lljubojevic@Ljubojevic-PC projekt-urs]$ xorriso -as genisoimage -output cloud-init.iso -volid CIDATA -joliet -rock user-data meta-data
xorriso 1.5.4 : RockRidge filesystem manipulator, libburnia project.

Drive current: -outdev 'stdio:cloud-init.iso'
Media current: stdio file, overwriteable
Media status : is blank
Media summary: 0 sessions, 0 data blocks, 0 data,  272g free
Added to ISO image: file '/user-data'='/home/lljubojevic/projekt-urs/user-data'
xorriso : UPDATE :       1 files added in 1 seconds
Added to ISO image: file '/meta-data'='/home/lljubojevic/projekt-urs/meta-data'
xorriso : UPDATE :       2 files added in 1 seconds
ISO image produced: 184 sectors
Written to medium : 184 sectors at LBA 0
Writing to 'stdio:cloud-init.iso' completed successfully.
```
4. Povećati prostor za pohranu na preuzetoj slici sustava:
```shell
[lljubojevic@Ljubojevic-PC projekt-urs]$ qemu-img resize myaarch.qcow2 20G
Image resized.
```
5. Stvoriti virtualni stroj u alatu virt-manager s koracima:
* Import existing image - Browse local - odabrati sliku sustava
* Operacijski sustav: archlinux 
* Memorija: 2048 MB - Procesor: 2 jezgre 
* Odabrati "Customize before install", zatim "Add hardware" - Storage - izabrati ISO sliku za cloud-init - tip: cdrom
* Odabrati ponovno "Add hardware" - Storage - kreirati dva nova virtualna diska veličine 20GB - tip: VirtIO Disk
* Begin Installation

6. U sučelju za interakciju s virtualnim strojem QEMU/KVM izvršiti prijavu na sustav i saznati IP adresu virtualnog stroja:
![](https://i.imgur.com/mgv0kbH.png)

7. Proslijediti na virtualni stroj skripte diskB i diskC, http i php konfiguraciju:
```shell
[lljubojevic@Ljubojevic-PC Desktop]$ scp diskB.sh diskC.sh httpd.conf php.ini seminar@192.168.122.102:/home/seminar
```
* Konfiguracije su zadane od softvera uz izmjene kao u sekciji iznad.

8. Instalirati na domaćinu Ansible, instalirati sshpass i zapisati IP adresu od virtualnog stroja u /etc/ansible/known-hosts
```shell
[lljubojevic@Ljubojevic-PC Desktop]$ sudo pacman -S ansible sshpass
[lljubojevic@Ljubojevic-PC Desktop]$ sudo nano /etc/ansible/hosts
[me]
192.168.0.17

[virtualka]
192.168.122.102 ansible_user=seminar

[seminar@archlinux ~]$ sudo -k ansible-playbook moodle.yml 
SSH password: 

PLAY [Instalacija svih potrebnih requirementa] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host 192.168.122.102 is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-core/2.13/reference_appendices/interpreter_discovery.html for more information.

ok: [192.168.122.102]

TASK [update packages] *********************************************************************************************************************************************************************************************************************
ok: [192.168.122.102]

TASK [Download all packages] ***************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [prepare zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [install zfs-utils] *******************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [install zfs] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [Reboot] ******************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [cekanje na reboot] *******************************************************************************************************************************************************************************************************************
ok: [192.168.122.102]

TASK [initialize zfs] **********************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [partition disk vdb] ******************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [partition disk vdc] ******************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [partition disk vdb] ******************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [partition disk vdc] ******************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [create a mirror] *********************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [make service and timer files] ********************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [make service and timer files] ********************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [make backup script] ******************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [reload systemd daemon] ***************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [unmask unit] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [unmask unit] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [enable unit] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [enable unit] *************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [start unit] **************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [mkdir /var/www/html] *****************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [start unit] **************************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [initialize firewall] *****************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [initialize firewall] *****************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [initialize firewall] *****************************************************************************************************************************************************************************************************************
changed: [192.168.122.102]

TASK [enable mariadb] **************************************************************************************************************************************
changed: [192.168.122.102]

TASK [configure mariadb] ***********************************************************************************************************************************
changed: [192.168.122.102]

TASK [enable mariadb] **************************************************************************************************************************************
changed: [192.168.122.102]

TASK [install pip] *****************************************************************************************************************************************
changed: [192.168.122.102]

TASK [install mysqlclient] *********************************************************************************************************************************
changed: [192.168.122.102]

TASK [Create a new database mojabaza] **********************************************************************************************************************
changed: [192.168.122.102]

TASK [Create user] *****************************************************************************************************************************************
changed: [192.168.122.102]

TASK [Set MariaDB root password for the first time (root@localhost)] ***************************************************************************************
changed: [192.168.122.102]

TASK [replace httpd conf] **********************************************************************************************************************************
changed: [192.168.122.102]

TASK [replace php ini] *************************************************************************************************************************************
changed: [192.168.122.102]

TASK [download moodle] *************************************************************************************************************************************
changed: [192.168.122.102]

TASK [move moodle to /var/www/html] ************************************************************************************************************************
changed: [192.168.122.102]

TASK [create moodledata] ***********************************************************************************************************************************
changed: [192.168.122.102]

TASK [restart httpd] ***************************************************************************************************************************************
changed: [192.168.122.102]
```

10. Konfigurirati moodle putem web sučelja:
![](https://i.imgur.com/LtNTR6g.png)
![](https://i.imgur.com/z3RO2NO.png)
![](https://i.imgur.com/2AQarqD.png)
![](https://i.imgur.com/isjrU9l.png)
![](https://i.imgur.com/yWeuAcB.png)
![](https://i.imgur.com/vyz7llL.png)
![](https://i.imgur.com/o9PmVD3.png)
![](https://i.imgur.com/w0lkSMj.png)
![](https://i.imgur.com/h0hxFxH.png)
![](https://i.imgur.com/COx7RgL.png)
![](https://i.imgur.com/1gOee10.png)

11. Spremiti ispisanu moodle konfiguraciju u /var/www/html/moodle/config.php




* Skripta DiskB je sadržaja:
```shell
#!bin/sh 
echo "g
n
p
1


t
157
w" | fdisk /dev/vdb
```

* Skripta DiskC je sadržaja:
```shell
#!bin/sh 
echo "g
n
p
1


t
157
w" | fdisk /dev/vdc
```
* Playbook moodle.yml je sadržaja:
```yaml
---
    - name: Instalacija svih potrebnih requirementa
      hosts: virtualka
      become: yes
      tasks:
      - name: update packages
        ansible.builtin.pacman:
          update_cache: yes
          upgrade: yes
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
            - rsync
            - nftables
            - apache
            - mariadb
            - php7
            - php7-apache
            - graphviz
            - aspell
            - ghostscript
            - clamav
            - php7-pspell
            - php7-gd
            - php7-intl
            - git
          state: latest
      - name: prepare zfs
        ansible.builtin.command:
          cmd: curl -o 0001-only-build-the-module-in-dkms.conf.patch "https://aur.archlinux.org/cgit/aur.git/plain/0001-only-build-the-module-in-dkms.conf.patch?h=zfs-dkms"
      - name: prepare zfs
        ansible.builtin.command:     
          cmd: curl -o PKGBUILD "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-dkms"
      - name: prepare zfs
        ansible.builtin.command:      
          cmd: curl -o PKGBUILD "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-dkms"   
      - name: prepare zfs
        ansible.builtin.command:      
          cmd: curl -o zfs.initcpio.install "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.install?h=zfs-utils"      
      - name: prepare zfs
        ansible.builtin.command:     
          cmd: curl -o zfs.initcpio.hook "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.hook?h=zfs-utils"    
      - name: prepare zfs
        ansible.builtin.command:      
          cmd: curl -o PKGBUILD-utils "https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=zfs-utils"     
      - name: prepare zfs
        ansible.builtin.command:
          cmd: gpg --receive-keys 6AD860EED4598027 
      - name: prepare zfs
        ansible.builtin.command:
          cmd: makepkg -p PKGBUILD-utils
        become_user: seminar
      - name: prepare zfs
        ansible.builtin.command:      
          cmd: makepkg   
        become_user: seminar
      - name: install zfs-utils
        ansible.builtin.pacman:
          name:
            - zfs-utils-2.1.4+65.r05147319b0-1-x86_64.pkg.tar.zst
          state: present
      - name: install zfs
        ansible.builtin.pacman:
          name:
            - zfs-dkms-2.1.4+65.r05147319b0-2-any.pkg.tar.zst
          state: present

      - name: Reboot
        shell: "sleep 5 && reboot"
        async: 1
        poll: 0
    
      - name: cekanje na reboot
        wait_for_connection:
          connect_timeout: 60
          sleep: 5
          delay: 5
          timeout: 300
    
      - name: initialize zfs
        ansible.builtin.command:
          cmd: modprobe zfs
      - name: partition disk vdb
        ansible.builtin.command:
          cmd: chmod u+x diskB.sh
      - name: partition disk vdc
        ansible.builtin.command:
          cmd: chmod u+x diskC.sh
      - name: partition disk vdb
        ansible.builtin.command:
          cmd: bash diskB.sh
      - name: partition disk vdc
        ansible.builtin.command:
          cmd: bash diskC.sh
          
      - name: create a mirror
        ansible.builtin.command:
          cmd: zpool create -m /mymirror mymirror mirror /dev/vdb1 /dev/vdc1
      - name: make service and timer files
        ansible.builtin.copy:
          dest: /etc/systemd/system/backup.timer
          content: |
            [Unit]
            Description=test
            
            [Timer]
            OnCalendar=daily
            Persistent=true
            
            [Install]
            WantedBy=timers.target
      - name: make service and timer files
        ansible.builtin.copy:
          dest: /etc/systemd/system/backup.service
          content: |
            [Unit]
            Description=backup
    
            [Service]
            Type=oneshot
            ExecStart=/bin/bash /home/seminar/backup.sh
      - name: make backup script
        ansible.builtin.copy:
          dest: /home/seminar/backup.sh
          content: |
            #!/bin/sh
            sudo rsync -a --delete --quiet /var/www/html /mymirror
      - name: reload systemd daemon
        ansible.builtin.command:
          cmd: systemctl daemon-reload
      - name: unmask unit       
        ansible.builtin.command:
           cmd: systemctl unmask backup.timer
      - name: unmask unit
        ansible.builtin.command:
           cmd: systemctl unmask backup.service
      - name: enable unit   
        ansible.builtin.command:
           cmd: systemctl enable backup.timer
      - name: enable unit   
        ansible.builtin.command:
           cmd: systemctl enable backup.service      
      - name: start unit 
        ansible.builtin.command:
           cmd: systemctl start backup.timer
      - name: mkdir /var/www/html
        ansible.builtin.file:
           path: /var/www/html
           state: directory
           mode: '0755'
      - name: start unit   
        ansible.builtin.command:
           cmd: systemctl start backup.service
           
      - name: initialize firewall
        ansible.builtin.command:
          cmd: nft add table inet my_rules
      - name: initialize firewall
        ansible.builtin.command:      
          cmd: nft add chain inet my_rules my_chain '{type filter hook input priority 0;}'       
      - name: initialize firewall
        ansible.builtin.command:      
          cmd: nft add rule inet my_rules my_chain  tcp dport {22, 80, 443} accept  
          
      - name: enable mariadb
        ansible.builtin.command:      
          cmd: systemctl enable mariadb
      - name: configure mariadb
        ansible.builtin.command:      
          cmd: mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
      - name: enable mariadb
        ansible.builtin.command:      
          cmd: systemctl start mariadb           
      - name: install pip
        ansible.builtin.pacman:
          name: python-pip
          state: latest
      - name: install mysqlclient
        ansible.builtin.command:
          cmd: pip install mysqlclient
      - name: Create a new database mojabaza
        community.mysql.mysql_db:
          name: moodle
          state: present
      - name: Create user
        community.mysql.mysql_user:
          name: moodle
          password: 'vrloTajniPass'
          priv: 'moodle.*:SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER'
          state: present
      - name: Set MariaDB root password for the first time (root@localhost)
        mysql_user:
          login_user: 'root'
          login_host: 'localhost'
          login_password: ''
          name: root
          password: 'passwd'
          state: present
      - name: replace httpd conf
        ansible.builtin.copy:
          src: httpd.conf
          dest: /etc/httpd/conf/httpd.conf
      - name: replace php ini
        ansible.builtin.copy:
          src: php.ini
          dest: /etc/php7/php.ini
      - name: download moodle
        ansible.builtin.command:
          cmd: git clone git://git.moodle.org/moodle.git
      - name: move moodle to /var/www/html
        ansible.builtin.copy:
          src: /home/seminar/moodle
          dest: /var/www/html/moodle
          mode: '0755'
          owner: http
          remote_src: yes
      - name: create moodledata
        ansible.builtin.file:
          path: /var/www/moodledata
          state: directory
          mode: '777'
      - name: restart httpd
        ansible.builtin.command:
          cmd: systemctl restart httpd     
```
