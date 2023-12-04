# Systemd essentials
[Based on Scaleway systemd-essentials tutorial](https://www.scaleway.com/en/docs/tutorials/systemd-essentials/?facetFilters=%5B%22categories%3Ainstances%22%5D&page=1)
## systemd
>systemd is a suite of tools that provides a fast and flexible init model for managing an entire machine from boot onwards. It provides a system and service manager which runs as PID 1 and controls the start of the rest of the system. In recent years the majority of Linux distributions have adopted systemd as their default init system.
* [source](https://www.scaleway.com/en/docs/tutorials/systemd-essentials/?facetFilters=%5B%22categories%3Ainstances%22%5D&page=1)
>systemd is a suite of basic building blocks for a Linux system. It provides a system and service manager that runs as PID 1 and starts the rest of the system. systemd provides aggressive parallelization capabilities, uses socket and D-Bus activation for starting services, offers on-demand starting of daemons, keeps track of processes using Linux control groups, maintains mount and automount points, and implements an elaborate transactional dependency-based service control logic. systemd supports SysV and LSB init scripts and works as a replacement for sysvinit. Other parts include a logging daemon, utilities to control basic system configuration like the hostname, date, locale, maintain a list of logged-in users and running containers and virtual machines, system accounts, runtime directories and settings, and daemons to manage simple network configuration, network time synchronization, log forwarding, and name resolution.
* [source](https://www.freedesktop.org/wiki/Software/systemd/)
>Systemd is a system and service manager for Linux, compatible with SysV and LSB inyscript. Systemd provides outstanding ability to parallelize, use socket, and D-Bus activation to run services, enable on-demand daemons, monitor processes using Linux control groups, support recording and system restores, maintain gathering places, and automated build and assembly service, maintains a system environment and automated assembly services and implements a dependency management system based on logical control of services. »
Among the most important features we can highlight:
    Process parallelization (the ability to run 2 or more processes simultaneously) which translates into faster system startup.
    Optimize resource use with groups.
    Supports snapshots and system restore.
    Operate mounting points and bracket storage units.
* [source](https://hr.admininfo.info/qu-es-systemd)    

Useful link - CROATIAN: [carnet](https://sysportal.carnet.hr/node/1748)    
    
>Systemd is a set of daemons, libraries, and system tools designed to centrally manage and configure a Linux operating system. As you can see from the above, this is not just one daemon or service like init, but several, but the goal is the same - to load all the necessary components for OS operation and management.
* [source](https://pcchip.hr/softver/korisni/systemd-kontroverzni-projekt-u-svijetu-linuxa/)

## Learn systemd essentials
Display the current status of the system:
```shell
[root@scw-zavrsni-ispit ~]# systemctl status
● scw-zavrsni-ispit
State: running
Jobs: 0 queued
Failed: 0 units
Since: Sat 2022-03-05 11:01:52 UTC; 7min ago
CGroup: /
├─init.scope
│ └─1 /sbin/init
├─system.slice
│ ├─dbus.service
│ │ └─257 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --sy>
│ ├─sshd.service
│ │ └─487 "sshd: /usr/bin/sshd -D [listener] 0 of 10-100 startups"
│ ├─system-getty.slice
│ │ └─getty@tty1.service
│ │   └─440 /sbin/agetty -o "-p -- \\u" --noclear - linux
│ ├─system-serial\x2dgetty.slice
│ │ └─serial-getty@ttyS0.service
│ │   └─441 /sbin/agetty -o "-p -- \\u" --keep-baud 115200,57600,38400,9600 - vt220
│ ├─systemd-homed.service
│ │ └─260 /usr/lib/systemd/systemd-homed
│ ├─systemd-journald.service
│ │ └─214 /usr/lib/systemd/systemd-journald
│ ├─systemd-logind.service
│ │ └─261 /usr/lib/systemd/systemd-logind
│ ├─systemd-networkd.service
│ │ └─394 /usr/lib/systemd/systemd-networkd
│ ├─systemd-resolved.service
│ │ └─337 /usr/lib/systemd/systemd-resolved
│ ├─systemd-timesyncd.service
│ │ └─252 /usr/lib/systemd/systemd-timesyncd
│ ├─systemd-udevd.service
│ │ └─232 /usr/lib/systemd/systemd-udevd
│ └─systemd-userdbd.service
│   ├─227 /usr/lib/systemd/systemd-userdbd
│   ├─506 systemd-userwork
│   ├─507 systemd-userwork
│   └─508 systemd-userwork
└─user.slice
└─user-0.slice
├─session-1.scope
│ ├─492 "sshd: root@pts/0"
│ ├─502 -bash
│ ├─509 systemctl status
│ └─510 less
└─user@0.service
└─init.scope
├─495 /usr/lib/systemd/systemd --user
└─496 "(sd-pam)"
```
Displays all currently running services and their status.
Some of the systemd services:
* systemd-homed.service
    * systemd-homed(8) is a systemd service providing portable human-user accounts that are not dependent on current system configuration. 
    * used to enable home directory portability for a user and automatic home directory encryption when logging in
    * **homectl** - homectl is the main utility you will use for homed. With it, you can create, update, and inspect users; their home directories; and their ~/.identity files controlled by the systemd-homed service.
    * Adding and deleting a users: 
```shell
[root@scw-zavrsni-ispit ~]# homectl create user
🔐 Please enter new password for user user:
🔐 Please enter new password for user user (repeat):
[root@scw-zavrsni-ispit ~]# homectl remove user
```
Script that automatically creates users:
```
[root@scw-zavrsni-ispit ~]# nano /tmp/userlist
user1
user2
user3
[root@scw-zavrsni-ispit ~]# nano /usr/sbin/createuser.sh
#!/bin/bash
userfile=/tmp/userlist
username=$(cat /tmp/userlist | tr 'A-Z'  'a-z')
password=$username
for user in $username
do
useradd $user
echo $password | passwd $user
done
echo "$(wc -l /tmp/userlist) created user"
tail -n$(wc -l /tmp userlist) /etc/passwd

[root@scw-zavrsni-ispit ~]# /usr/sbin/createuser.sh
New password: Retype new password: Password change has been aborted.
passwd: Authentication token manipulation error
passwd: password unchanged
New password: Retype new password: Password change has been aborted.
passwd: Authentication token manipulation error
passwd: password unchanged
New password: Retype new password: Password change has been aborted.
passwd: Authentication token manipulation error
passwd: password unchanged
3 /tmp/userlist stvorio korisnike
wc: /tmp: Is a directory
wc: userlist: No such file or directory
```
View users:
```shell
[root@scw-zavrsni-ispit ~]# less /etc/passwd
user1:x:1001:1001::/home/user1:/bin/bash
user2:x:1002:1002::/home/user2:/bin/bash
user3:x:1003:1003::/home/user3:/bin/bash
```

Delete:
```shell
[root@scw-zavrsni-ispit ~]# userdel user1
[root@scw-zavrsni-ispit ~]# userdel user2
[root@scw-zavrsni-ispit ~]# userdel user3
```
[source](**https://wiki.archlinux.org/title/Systemd-homed**)
[source](https://www.geeksforgeeks.org/create-multiple-users-using-shell-script-in-linux/)

* systemd-networkd.service
    * systemd-networkd is a system daemon that manages network configurations. It detects and configures network devices as they appear; it can also create virtual network devices. This service can be especially useful to set up complex network configurations for a container managed by systemd-nspawn or for virtual machines. It also works fine on simple connections. 
```shell
[root@scw-zavrsni-ispit ~]# systemctl status systemd-networkd
● systemd-networkd.service - Network Configuration
Loaded: loaded (/usr/lib/systemd/system/systemd-networkd.service; enabled; vendor preset: enabled)
Active: active (running) since Sat 2022-03-05 11:02:03 UTC; 43min ago
TriggeredBy: ● systemd-networkd.socket
Docs: man:systemd-networkd.service(8)
Main PID: 394 (systemd-network)
Status: "Processing requests..."
Tasks: 1 (limit: 1141)
Memory: 1.4M
CPU: 52ms
CGroup: /system.slice/systemd-networkd.service
└─394 /usr/lib/systemd/systemd-networkd

Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: eth0: Link UP
Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: eth0: Gained carrier
Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: lo: Link UP
Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: lo: Gained carrier
Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: eth0: Gained IPv6LL
Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: Enumeration completed
Mar 05 11:02:03 scw-zavrsni-ispit systemd[1]: Started Network Configuration.
Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: eth0: DHCPv4 address 10.18.0.195/31, gateway 10.18.0.194 ac>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: Could not set hostname: Access denied
Mar 05 11:02:03 scw-zavrsni-ispit systemd-networkd[394]: Could not set hostname: Access denied
```
[source](https://wiki.archlinux.org/title/systemd-networkd)
* systemd-journald.service
    * systemd-journald is a system service that collects and stores logging data. It creates and maintains structured, indexed journals based on logging information that is received from a variety of sources:
        * Kernel log messages, via kmsg
        * Simple system log messages, via the libc syslog(3) call
        * Structured system log messages via the native Journal API, see sd_journal_print(3) and Native Journal Protocol
       * Standard output and standard error of service units. For further details see below.
       * Audit records, originating from the kernel audit subsystem
```shell
root@scw-zavrsni-ispit network]# systemctl status systemd-journald
● systemd-journald.service - Journal Service
Loaded: loaded (/usr/lib/systemd/system/systemd-journald.service; static)
Active: active (running) since Sat 2022-03-05 11:01:53 UTC; 47min ago
TriggeredBy: ● systemd-journald-audit.socket
● systemd-journald-dev-log.socket
● systemd-journald.socket
Docs: man:systemd-journald.service(8)
man:journald.conf(5)
Main PID: 214 (systemd-journal)
Status: "Processing requests..."
Tasks: 1 (limit: 1141)
Memory: 5.9M
CPU: 300ms
CGroup: /system.slice/systemd-journald.service
└─214 /usr/lib/systemd/systemd-journald

Mar 05 11:01:53 scw-zavrsni-ispit systemd-journald[214]: Journal started
Mar 05 11:01:53 scw-zavrsni-ispit systemd-journald[214]: Runtime Journal (/run/log/journal/6ae3079320e2465abecf21a39>
Mar 05 11:01:53 scw-zavrsni-ispit systemd-journald[214]: Time spent on flushing to /var/log/journal/6ae3079320e2465a>
Mar 05 11:01:53 scw-zavrsni-ispit systemd-journald[214]: System Journal (/var/log/journal/6ae3079320e2465abecf21a397>
Mar 05 11:01:53 scw-zavrsni-ispit systemd-journald[214]: Received client request to flush runtime journal.
Notice: journal has been rotated since unit was started, output may be incomplete.
```
Config file is located in: /etc/systemd/journald.conf
[source](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html)
* systemd-logind.service
    * systemd-logind is a system service that manages user logins. It is responsible for:
        * Keeping track of users and sessions, their processes and their idle state.
        * Generating and managing session IDs.
      *  Providing polkit-based access for users for operations such as system shutdown or sleep
       * Implementing a shutdown/sleep inhibition logic for applications
      *  Handling of power/sleep hardware keys
       * Multi-seat management
       * Session switch management
        * Device access management for users
        *  Automatic spawning of text logins (gettys) on virtual console activation and user runtime directory management
```
[root@scw-zavrsni-ispit network]# systemctl status systemd-logind
● systemd-logind.service - User Login Management
Loaded: loaded (/usr/lib/systemd/system/systemd-logind.service; static)
Active: active (running) since Sat 2022-03-05 11:01:54 UTC; 50min ago
Docs: man:sd-login(3)
man:systemd-logind.service(8)
man:logind.conf(5)
man:org.freedesktop.login1(5)
Main PID: 261 (systemd-logind)
Status: "Processing requests..."
Tasks: 1 (limit: 1141)
Memory: 1.5M
CPU: 80ms
CGroup: /system.slice/systemd-logind.service
└─261 /usr/lib/systemd/systemd-logind

Mar 05 11:01:54 scw-zavrsni-ispit systemd[1]: Starting User Login Management...
Mar 05 11:01:54 scw-zavrsni-ispit systemd-logind[261]: Watching system buttons on /dev/input/event0 (Power Button)
Mar 05 11:01:54 scw-zavrsni-ispit systemd-logind[261]: Watching system buttons on /dev/input/event1 (AT Translated S>
Mar 05 11:01:54 scw-zavrsni-ispit systemd-logind[261]: New seat seat0.
Mar 05 11:01:54 scw-zavrsni-ispit systemd[1]: Started User Login Management.
Mar 05 11:02:07 scw-zavrsni-ispit systemd-logind[261]: New session 1 of user root.
```
[source](https://www.freedesktop.org/software/systemd/man/systemd-logind.service.html)
* systemd-resolved.service
    * systemd-resolved is a system service that provides network name resolution to local applications
```
[root@scw-zavrsni-ispit network]# systemctl status systemd-resolved
● systemd-resolved.service - Network Name Resolution
Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: enabled)
Active: active (running) since Sat 2022-03-05 11:01:59 UTC; 52min ago
Docs: man:systemd-resolved.service(8)
man:org.freedesktop.resolve1(5)
https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
Main PID: 337 (systemd-resolve)
Status: "Processing requests..."
Tasks: 1 (limit: 1141)
Memory: 4.8M
CPU: 91ms
CGroup: /system.slice/systemd-resolved.service
└─337 /usr/lib/systemd/systemd-resolved

Mar 05 11:01:59 scw-zavrsni-ispit systemd[1]: Starting Network Name Resolution...
Mar 05 11:01:59 scw-zavrsni-ispit systemd-resolved[337]: Positive Trust Anchors:
Mar 05 11:01:59 scw-zavrsni-ispit systemd-resolved[337]: . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409>
Mar 05 11:01:59 scw-zavrsni-ispit systemd-resolved[337]: Negative trust anchors: home.arpa 10.in-addr.arpa 16.172.in>
Mar 05 11:01:59 scw-zavrsni-ispit systemd-resolved[337]: Using system hostname 'scw-zavrsni-ispit'.
Mar 05 11:01:59 scw-zavrsni-ispit systemd[1]: Started Network Name Resolution.
Mar 05 11:02:34 scw-zavrsni-ispit systemd-resolved[337]: Clock change detected. Flushing caches.
```
Displays and saves the hostname for applications running on a computer or localhost ... an example of this is a Oracle DB TNSListener when Oracle connects via the localhost hostname.
[source](https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html)
* systemd-timesyncd.service
    * Synchronizes time with a network time server - Synchronizing clock RTC with flags
    * systemd-timesyncd is a system service that may be used to synchronize the local system clock with a remote Network Time Protocol (NTP) server. It also saves the local time to disk every time the clock has been synchronized and uses this to possibly advance the system realtime clock on subsequent reboots to ensure it (roughly) monotonically advances even if the system lacks a battery-buffered RTC chip.
```shell
root@scw-zavrsni-ispit network]# systemctl status systemd-timesyncd
● systemd-timesyncd.service - Network Time Synchronization
Loaded: loaded (/usr/lib/systemd/system/systemd-timesyncd.service; enabled; vendor preset: enabled)
Active: active (running) since Sat 2022-03-05 11:01:54 UTC; 56min ago
Docs: man:systemd-timesyncd.service(8)
Main PID: 252 (systemd-timesyn)
Status: "Initial synchronization to time server 10.196.2.3:123 (10.196.2.3)."
Tasks: 2 (limit: 1141)
Memory: 2.1M
CPU: 88ms
CGroup: /system.slice/systemd-timesyncd.service
└─252 /usr/lib/systemd/systemd-timesyncd

Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:03 scw-zavrsni-ispit systemd-timesyncd[252]: Network configuration changed, trying to establish connect>
Mar 05 11:02:34 scw-zavrsni-ispit systemd-timesyncd[252]: Initial synchronization to time server 10.196.2.3:123 (10.>
```
[source](https://www.freedesktop.org/software/systemd/man/systemd-timesyncd.service.html)
* systemd-userdbd.service
    * systemd-userdbd is a system service that multiplexes user/group lookups to all local services that provide JSON user/group record definitions to the system. In addition it synthesizes JSON user/group records from classic UNIX/glibc NSS user/group records in order to provide full backwards compatibility.
    * The userdbctl command is used to list all users from the user database
```shell
[root@scw-zavrsni-ispit ~]# userdbctl
NAME                   DISPOSITION   UID   GID REALNAME                     HOME            SHELL
root                   intrinsic       0     0 -                            /root           /bin/bash
nobody                 intrinsic   65534 65534 Nobody                       /               /usr/bin/nologin
bin                    system          1     1 -                            /               /usr/bin/nologin
daemon                 system          2     2 -                            /               /usr/bin/nologin
mail                   system          8    12 -                            /var/spool/mail /usr/bin/nologin
ftp                    system         14    11 -                            /srv/ftp        /usr/bin/nologin
http                   system         33    33 -                            /srv/http       /usr/bin/nologin
uuidd                  system         68    68 -                            /               /usr/bin/nologin
dbus                   system         81    81 System Message Bus           /               /usr/bin/nologin
systemd-timesync       system        976   976 systemd Time Synchronization /               /usr/bin/nologin
systemd-resolve        system        977   977 systemd Resolver             /               /usr/bin/nologin
systemd-journal-remote system        978   978 systemd Journal Remote       /               /usr/bin/nologin
systemd-oom            system        979   979 systemd Userspace OOM Killer /               /usr/bin/nologin
systemd-network        system        980   980 systemd Network Management   /               /usr/bin/nologin
systemd-coredump       system        981   981 systemd Core Dumper          /               /usr/bin/nologin
arch                   regular      1000  1000 arch Cloud User              /home/arch      /bin/bash

16 users listed.
```
[source](https://www.freedesktop.org/software/systemd/man/systemd-userdbd.service.html)
* systemd-udevd.service
    * systemd-udevd listens to kernel uevents. For every event, systemd-udevd executes matching instructions specified in udev rules 
    * udevadm trigger - Request device events from the kernel. Usually used to replay events at system coldplug time. --verbose Print the list of devices which will be triggered. 
```shell
[root@scw-zavrsni-ispit ~]# udevadm trigger --verbose
/sys/devices/LNXSYSTM:00
/sys/devices/LNXSYSTM:00/LNXPWRBN:00
/sys/devices/LNXSYSTM:00/LNXPWRBN:00/input/input0
/sys/devices/LNXSYSTM:00/LNXPWRBN:00/input/input0/event0
/sys/devices/LNXSYSTM:00/LNXPWRBN:00/wakeup/wakeup7
/sys/devices/LNXSYSTM:00/LNXSYBUS:00
/sys/devices/LNXSYSTM:00/LNXSYBUS:00/ACPI0010:00
/sys/devices/LNXSYSTM:00/LNXSYBUS:00/ACPI0010:00/LNXCPU:00
/sys/devices/LNXSYSTM:00/LNXSYBUS:00/PNP0103:00
/sys/devices/LNXSYSTM:00/LNXSYBUS:00/PNP0A03:00
/sys/devices/LNXSYSTM:00/LNXSYBUS:00/PNP0A03:00/PNP0A06:00
..
/sys/devices/LNXSYSTM:00/LNXSYBUS:01
/sys/devices/breakpoint
/sys/devices/cpu
/sys/devices/kprobe
/sys/devices/msr
/sys/devices/pci0000:00/0000:00:00.0
/sys/devices/pci0000:00/0000:00:01.0
/sys/devices/pci0000:00/0000:00:01.1
/sys/devices/pci0000:00/0000:00:01.1/ata1/ata_port/ata1
/sys/devices/pci0000:00/0000:00:01.1/ata1/host0
/sys/devices/pci0000:00/0000:00:01.1/ata1/host0/scsi_host/host0
/sys/devices/pci0000:00/0000:00:01.1/ata1/link1/ata_link/link1
/sys/devices/pci0000:00/0000:00:01.1/ata1/link1/dev1.0/ata_device/dev1.0
/sys/devices/pci0000:00/0000:00:01.1/ata1/link1/dev1.1/ata_device/dev1.1
/sys/devices/pci0000:00/0000:00:01.1/ata2/ata_port/ata2
/sys/devices/pci0000:00/0000:00:01.1/ata2/host1
/sys/devices/pci0000:00/0000:00:01.1/ata2/host1/scsi_host/host1
/sys/devices/pci0000:00/0000:00:01.1/ata2/link2/ata_link/link2
/sys/devices/pci0000:00/0000:00:01.1/ata2/link2/dev2.0/ata_device/dev2.0
/sys/devices/pci0000:00/0000:00:01.1/ata2/link2/dev2.1/ata_device/dev2.1
/sys/devices/pci0000:00/0000:00:01.3
/sys/devices/pci0000:00/0000:00:01.3/i2c-0
/sys/devices/pci0000:00/0000:00:02.0
...
/sys/devices/pci0000:00/QEMU0002:00
/sys/devices/pci0000:00/pci_bus/0000:00
/sys/devices/platform/PNP0103:00
/sys/devices/platform/efivars.0
/sys/devices/platform/i8042
/sys/devices/platform/i8042/serio0
/sys/devices/platform/i8042/serio0/input/input1
/sys/devices/platform/i8042/serio0/input/input1/event1
/sys/devices/platform/i8042/serio0/input/input1/input1::capslock
/sys/devices/platform/i8042/serio0/input/input1/input1::numlock
/sys/devices/platform/i8042/serio0/input/input1/input1::scrolllock
/sys/devices/platform/i8042/serio1
/sys/devices/platform/i8042/serio1/input/input4
/sys/devices/platform/i8042/serio1/input/input4/event4
/sys/devices/platform/i8042/serio1/input/input4/mouse1
/sys/devices/platform/i8042/serio1/input/input5
/sys/devices/platform/i8042/serio1/input/input5/event3
/sys/devices/platform/i8042/serio1/input/input5/mouse0
/sys/devices/platform/intel_rapl_msr.0
/sys/devices/platform/pcspkr
/sys/devices/platform/pcspkr/input/input3
/sys/devices/platform/pcspkr/input/input3/event2
/sys/devices/platform/platform-framebuffer.0
/sys/devices/platform/reg-dummy
/sys/devices/platform/reg-dummy/regulator/regulator.0
/sys/devices/platform/regulatory.0
/sys/devices/platform/rtc-efi.0
/sys/devices/platform/serial8250
/sys/devices/platform/serial8250/tty/ttyS1
...
/sys/devices/pnp0/00:00
/sys/devices/pnp0/00:00/cmos_nvram0
/sys/devices/pnp0/00:00/rtc/rtc0
/sys/devices/pnp0/00:00/rtc/rtc0/alarmtimer.0.auto
/sys/devices/pnp0/00:00/rtc/rtc0/alarmtimer.0.auto/wakeup/wakeup9
/sys/devices/pnp0/00:00/wakeup/wakeup8
/sys/devices/pnp0/00:01
/sys/devices/pnp0/00:02
/sys/devices/pnp0/00:03
/sys/devices/pnp0/00:04
/sys/devices/pnp0/00:04/tty/ttyS0
/sys/devices/software
/sys/devices/system/clockevents/broadcast
/sys/devices/system/clockevents/clockevent0
/sys/devices/system/clocksource/clocksource0
/sys/devices/system/container/PNP0A06:00
/sys/devices/system/container/PNP0A06:01
/sys/devices/system/container/PNP0A06:02
/sys/devices/system/cpu/cpu0
/sys/devices/system/edac/mc
/sys/devices/system/machinecheck/machinecheck0
/sys/devices/system/memory/memory0
/sys/devices/system/memory/memory1
/sys/devices/system/memory/memory2
/sys/devices/system/memory/memory3
/sys/devices/system/memory/memory4
/sys/devices/system/memory/memory5
/sys/devices/system/memory/memory6
/sys/devices/system/memory/memory7
/sys/devices/system/node/node0
/sys/devices/tracepoint
/sys/devices/uprobe
/sys/devices/virtual/bdi/254:0
/sys/devices/virtual/bdi/btrfs-1
/sys/devices/virtual/cpuid/cpu0
...
/sys/devices/virtual/misc/btrfs-control
/sys/devices/virtual/misc/cpu_dma_latency
/sys/devices/virtual/misc/device-mapper
/sys/devices/virtual/misc/fuse
/sys/devices/virtual/misc/hpet
/sys/devices/virtual/misc/hw_random
/sys/devices/virtual/misc/kvm
/sys/devices/virtual/misc/loop-control
/sys/devices/virtual/misc/nvram
/sys/devices/virtual/misc/psaux
/sys/devices/virtual/misc/rfkill
/sys/devices/virtual/misc/snapshot
/sys/devices/virtual/misc/udmabuf
/sys/devices/virtual/misc/vga_arbiter
/sys/devices/virtual/msr/msr0
/sys/devices/virtual/net/lo
/sys/devices/virtual/powercap/dtpm
/sys/devices/virtual/thermal/cooling_device0
/sys/devices/virtual/tty/console
/sys/devices/virtual/tty/ptmx
/sys/devices/virtual/tty/tty
/sys/devices/virtual/tty/tty0
/sys/devices/virtual/tty/tty1
...
```
[source](https://linux.die.net/man/8/udevadm)
[source](https://www.freedesktop.org/software/systemd/man/systemd-udevd.service.html)
**Using units**
* The most common object that systemd manages and acts upon is a ”unit”. Units can be, for example, services (.service), devices (.device), mount points (.mount), or sockets (.socket).
* Unit - part of a computer system:
    * services, devices, mounting points or sockets    
    * Unit files are stored in the /usr/lib/systemd directory and its subdirectories
```shell
[root@scw-zavrsni-ispit ~]# ls -la /usr/lib/systemd
total 10080
drwxr-xr-x 1 root root    2672 Jan 24 15:48 .
drwxr-xr-x 1 root root   18492 Mar  5 11:19 ..
drwxr-xr-x 1 root root       6 Jan 23 19:46 boot
drwxr-xr-x 1 root root     456 Jan 24 15:48 catalog
-rw-r--r-- 1 root root    9551 Jan 18 12:27 import-pubring.gpg
-rwxr-xr-x 1 root root 3360192 Jan 18 12:27 libsystemd-shared-250.so
drwxr-xr-x 1 root root     404 Jan 24 15:48 network
drwxr-xr-x 1 root root      48 Jan 24 15:48 ntp-units.d
drwxr-xr-x 1 root root      14 Jan 23 19:46 portable
-rw-r--r-- 1 root root     710 Jan 18 12:27 resolv.conf
drwxr-xr-x 1 root root      28 Jan 23 19:46 scripts
drwxr-xr-x 1 root root   11360 Mar  5 11:19 system
-rwxr-xr-x 1 root root 1882368 Jan 18 12:27 systemd
-rwxr-xr-x 1 root root   14240 Jan 18 12:27 systemd-ac-power
-rwxr-xr-x 1 root root   30632 Jan 18 12:27 systemd-backlight
-rwxr-xr-x 1 root root   18424 Jan 18 12:27 systemd-binfmt
-rwxr-xr-x 1 root root   30816 Jan 18 12:27 systemd-bless-boot
-rwxr-xr-x 1 root root   14328 Jan 18 12:27 systemd-boot-check-no-failures
-rwxr-xr-x 1 root root   14248 Jan 18 12:27 systemd-cgroups-agent
-rwxr-xr-x 1 root root   55344 Jan 18 12:27 systemd-coredump
-rwxr-xr-x 1 root root   75960 Jan 18 12:27 systemd-cryptsetup
-rwxr-xr-x 1 root root   38904 Jan 18 12:27 systemd-export
-rwxr-xr-x 1 root root   22448 Jan 18 12:27 systemd-fsck
-rwxr-xr-x 1 root root   22520 Jan 18 12:27 systemd-growfs
-rwxr-xr-x 1 root root   14248 Jan 18 12:27 systemd-hibernate-resume
-rwxr-xr-x 1 root root  223256 Jan 18 12:27 systemd-homed
-rwxr-xr-x 1 root root  223144 Jan 18 12:27 systemd-homework
-rwxr-xr-x 1 root root   43024 Jan 18 12:27 systemd-hostnamed
-rwxr-xr-x 1 root root   51224 Jan 18 12:27 systemd-import
-rwxr-xr-x 1 root root   43040 Jan 18 12:27 systemd-importd
-rwxr-xr-x 1 root root   30728 Jan 18 12:27 systemd-import-fs
...
```
```shell
[root@scw-zavrsni-ispit ~]# systemctl list-units
UNIT                                                              LOAD   ACTIVE SUB       DESCRIPTION    
sys-subsystem-net-devices-eth0.device                             loaded active plugged   Virtio network de>
-.mount                                                           loaded active mounted   Root Mount
boot-efi.mount                                                    loaded active mounted   /boot/efi
dev-hugepages.mount                                               loaded active mounted   Huge Pages File S>
dev-mqueue.mount                                                  loaded active mounted   POSIX Message Que>
run-user-0.mount                                                  loaded active mounted   /run/user/0
sys-fs-fuse-connections.mount                                     loaded active mounted   FUSE Control File>
sys-kernel-config.mount                                           loaded active mounted   Kernel Configurat>
sys-kernel-debug.mount                                            loaded active mounted   Kernel Debug File>
sys-kernel-tracing.mount                                          loaded active mounted   Kernel Trace File>
tmp.mount                                                         loaded active mounted   Temporary Directo>
systemd-ask-password-console.path                                 loaded active waiting   Dispatch Password>
systemd-ask-password-wall.path                                    loaded active waiting   Forward Password >
init.scope                                                        loaded active running   System and Servic>
session-3.scope                                                   loaded active running   Session 3 of User>
cloud-config.service                                              loaded active exited    Apply the setting>
cloud-final.service                                               loaded active exited    Execute cloud use>
cloud-init-local.service                                          loaded active exited    Initial cloud-ini>
cloud-init.service                                                loaded active exited    Initial cloud-ini>
dbus.service                                                      loaded active running   D-Bus System Mess>
getty@tty1.service                                                loaded active running   Getty on tty1
kmod-static-nodes.service                                         loaded active exited    Create List of St>
serial-getty@ttyS0.service                                        loaded active running   Serial Getty on t>
sshd.service                                                      loaded active running   OpenSSH Daemon
systemd-boot-update.service                                       loaded active exited    Automatic Boot Lo>
systemd-homed-activate.service                                    loaded active exited    Home Area Activat>
...     
```
Working with units:
```shell
[root@scw-zavrsni-ispit ~]# systemctl stop systemd-journald.service  
Warning: Stopping systemd-journald.service, but it can still be activated by:
systemd-journald-audit.socket
systemd-journald-dev-log.socket
systemd-journald.socket
[root@scw-zavrsni-ispit ~]# systemctl start systemd-journald.service
[root@scw-zavrsni-ispit ~]# systemctl restart systemd-journald.service
[root@scw-zavrsni-ispit ~]# systemctl reload systemd-journald.service
Failed to reload systemd-journald.service: Job type reload is not applicable for unit systemd-journald.service.
[root@scw-zavrsni-ispit ~]# systemctl status systemd-journald.service
● systemd-journald.service - Journal Service
Loaded: loaded (/usr/lib/systemd/system/systemd-journald.service; static)
Active: active (running) since Wed 2022-03-09 12:13:59 UTC; 15s ago
TriggeredBy: ● systemd-journald-audit.socket
● systemd-journald-dev-log.socket
● systemd-journald.socket
Docs: man:systemd-journald.service(8)
man:journald.conf(5)
Main PID: 18826 (systemd-journal)
Status: "Processing requests..."
Tasks: 1 (limit: 1141)
Memory: 1.1M
CPU: 37ms
CGroup: /system.slice/systemd-journald.service
└─18826 /usr/lib/systemd/systemd-journald

Mar 09 12:13:59 scw-zavrsni-ispit systemd-journald[18826]: Journal started
Mar 09 12:13:59 scw-zavrsni-ispit systemd-journald[18826]: System Journal (/var/log/journal/6ae3079320e2465ab>
lines 1-18/18 (END)
```
* Every service unit that is known to systemd can be started manually – even if it is disabled. To explicitly avoid that a service runs, use the mask command

**Shutdown and reboot**
```
sudo systemctl reboot now
sudo systemctl suspend now
sudo systemctl shutdown now
sudo systemctl poweroff now
```
now applies command instantly, default waits a minute.

# TODO:
* **System logs and Unit states**
* **Viewing and editing Units**
* **System logs**
* **Targets**

