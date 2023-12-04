# Virtual Machines
Luka Ljubojević
# Introduction
For the topic of the 1st seminar paper from the course Network and Mobile Operating Systems, I created a web server configuration for managing the status and location of items in the warehouse.

Shema consists of: 

 - Two web servers,
 - MariaDB database,
 - Load balancer,
 - Three clients(Android, Windows, GNU Linux - Ubuntu),

Visually:
![](https://i.imgur.com/39rgXxc.png)

Schema elements are configured as virtual machines within the "Virtual Machine Manager" tool.

The seminar paper was developed in GNU Linux - Mint, with packages (on a server computer - running virtual machines):

 - virt-manager, 
 - cloud-localds, 
 - qemu, 
 - mariadb-client-core-10.3, 
 - python3 pip, 
 - mysqlclient

# Downloading ubuntu-server system image

For the purposes of the seminar paper, we download the "Ubuntu Server 20.04 LTS (Focal Fossa) daily builds" system image,
* Available at: https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
 
We copy the system image four times and rename it according to the name of the virtual machines and arrange them in folders according to the following hierarchy:
* PZ MMOS
    * LB
    * MySQL
    * WS1
    * WS2

Then we create a user-data file using the cloud-init service that contains:
* Password
* Attributes that the password does not expire and can be used for ssh authentication

User-data:
```
#cloud-config
password: lozinka1
chpasswd: {expire: False}
ssh_pwauth: True
```
Using the user-data file, we are creating another disk for the virtual machine that will enable login to the system (user-data.img) ...
Commands used:
```
lljubojevic@Omen17:~$ cloud-localds user-data.img user-data
lljubojevic@Omen17:~$ mv user-data.img "/home/lljubojevic/PZ MMOS/MySQL"
lljubojevic@Omen17:~$ cloud-localds user-data.img user-data
lljubojevic@Omen17:~$ mv user-data.img "/home/lljubojevic/PZ MMOS/LB"
lljubojevic@Omen17:~$ cloud-localds user-data.img user-data
lljubojevic@Omen17:~$ mv user-data.img "/home/lljubojevic/PZ MMOS/WS1"
lljubojevic@Omen17:~$ cloud-localds user-data.img user-data
lljubojevic@Omen17:~$ mv user-data.img "/home/lljubojevic/PZ MMOS/WS2"
```
After sorting the images into folders, we increase the capacity (storage space) on each image with the following commands:
```
lljubojevic@Omen17:~$ cd PZ\ MMOS/
lljubojevic@Omen17:~/PZ MMOS$ ls
LB  MySQL  WS1  WS2

lljubojevic@Omen17:~/PZ MMOS$ cd LB
lljubojevic@Omen17:~/PZ MMOS/LB$ qemu-img resize lb.img 30G
Image resized.

lljubojevic@Omen17:~/PZ MMOS/LB$ cd ..
lljubojevic@Omen17:~/PZ MMOS$ cd MySQL/
lljubojevic@Omen17:~/PZ MMOS/MySQL$ qemu-img resize mysql.img 30G
Image resized.

lljubojevic@Omen17:~/PZ MMOS/MySQL$ cd ..
lljubojevic@Omen17:~/PZ MMOS$ cd WS1
lljubojevic@Omen17:~/PZ MMOS/WS1$ qemu-img resize ws1.img 30G
Image resized.

lljubojevic@Omen17:~/PZ MMOS/WS1$ cd ..
lljubojevic@Omen17:~/PZ MMOS$ cd WS2
lljubojevic@Omen17:~/PZ MMOS/WS2$ qemu-img resize ws2.img 30G
Image resized.

```
# Configuring virtual machines
## MySQL VM
The virtual machine is configured as follows:
* Import existing image -> Browse Local -> mysql.img
* Operating system: Ubuntu 20.04
* Memory: 4096, CPUS: 2
* Name MySQL
* Customize configuration before install -> Add Hardware -> Storage -> Select storage -> Local -> user-data.img
![](https://i.imgur.com/ug1QA36.png)

## Load balancer VM 
The virtual machine is configured as follows:
* Import existing image -> Browse Local -> lb.img
* Operating system: Ubuntu 20.04
* Memory: 4096, CPUS: 2
* Name LB
* Customize configuration before install -> Add Hardware -> Storage -> Select storage -> Local -> user-data.img
![](https://i.imgur.com/fDCoY0e.png)


## Web Server 1 VM
The virtual machine is configured as follows:
* Import existing image -> Browse Local -> ws1.img
* Operating system: Ubuntu 20.04
* Memory: 4096, CPUS: 2
* Name WS1
* Customize configuration before install -> Add Hardware -> Storage -> Select storage -> Local -> user-data.img
![](https://i.imgur.com/dGaSvhZ.png)


## Web Server 2 VM
The virtual machine is configured as follows:
* Import existing image -> Browse Local -> ws2.img
* Operating system: Ubuntu 20.04
* Memory: 4096, CPUS: 2
* Name WS2
* Customize configuration before install -> Add Hardware -> Storage -> Select storage -> Local -> user-data.img
![](https://i.imgur.com/FPUHujU.png)

## Windows client VM
The virtual machine is configured as follows:
* Local install media -> Browse Local -> Windows 7 Install DVD.iso
* Operating system: Windows Server 2016
* Memory: 4096, CPUS: 2
* Enable storage, disk image: 40GiB
* Name WinCl
![](https://i.imgur.com/UcJSakm.png)


## Ubuntu client VM
The virtual machine is configured as follows:
* Local install media -> Browse Local -> Ubuntu 20.04 Install DVD.iso
* Operating system: Ubuntu 20.04
* Memory: 2048, CPUS: 2
* Enable storage, disk image: 20GiB
* Name UbuntuCl
![](https://i.imgur.com/qKTOpCC.png)

## Andrx86 client VM
The virtual machine is configured as follows:
* Local install media -> Browse Local -> Android x86 Install DVD
* Operating system: Android-x86 9.0
* Memory: 2048, CPUS: 2
* Enable storage, disk image: 10GiB
* Name Andrx86Cl
![](https://i.imgur.com/kY2JgOe.png)

# Virtualn machine MySQL (database)
After a successfull log in from the virt-manager tool, the database server configuration begins.

First we find the IP address of the VM:
```
ubuntu@ubuntu:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:5a:31:f3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.77/24 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 2003sec preferred_lft 2003sec
    inet6 fe80::5054:ff:fe5a:31f3/64 scope link 
       valid_lft forever preferred_lft forever
```

For simplicity, we connect to the virtual machine from the host using the ssh command:
```
lljubojevic@Omen17:~$ ssh ubuntu@192.168.122.77
The authenticity of host '192.168.122.77 (192.168.122.77)' can't be established.
ECDSA key fingerprint is SHA256:91CCjASJznu9k2edurBZjRXqU4CFqa9piL2tFY3VY7Q.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.122.77' (ECDSA) to the list of known hosts.
ubuntu@192.168.122.77's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jan  9 16:24:25 UTC 2022

  System load:  0.21              Processes:               137
  Usage of /:   6.0% of 28.90GB   Users logged in:         1
  Memory usage: 7%                IPv4 address for enp1s0: 192.168.122.77
  Swap usage:   0%


31 updates can be applied immediately.
14 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Sun Jan  9 16:20:28 2022
```
After setting up the ssh connection, we update all repositories and install the MariaDB server:
```
ubuntu@ubuntu:~$ sudo apt update
ubuntu@ubuntu:~$ sudo apt install mariadb-server
ubuntu@ubuntu:~$ sudo mariadb
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 50
Server version: 10.3.32-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
With command:
```
ubuntu@ubuntu:~$ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
In the section [mysqld] we change the listener address to 0.0.0.0:
    bind-address            = 0.0.0.0
    
Address 0.0.0.0 represents listening (binding) to each available network - that is, it accepts requests from all parts of the network.

After changing the listening address, we restart the MariaDB service:
```
ubuntu@ubuntu:~$ sudo systemctl restart mysql

ubuntu@ubuntu:~$ sudo systemctl status mariadb.service
● mariadb.service - MariaDB 10.3.32 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-01-09 16:57:15 UTC; 18s ago
    ...
   Main PID: 3143 (mysqld)
     Status: "Taking your SQL requests now..."
      Tasks: 31 (limit: 4682)
     Memory: 63.8M
     CGroup: /system.slice/mariadb.service
             └─3143 /usr/sbin/mysqld
```
After setting up the MariaDB service, we create a database for the needs of the web server (web application).
In addition to the database, it is necessary to create a user who will have the right to access the database.
```
MariaDB [(none)]> CREATE DATABASE Skladiste;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> USE Skladiste;
Database changed

MariaDB [Skladiste]> CREATE USER webserver IDENTIFIED BY 'lozinka1';
Query OK, 0 rows affected (0.002 sec)

MariaDB [Skladiste]> GRANT ALL PRIVILEGES ON Skladiste TO webserver;
Query OK, 0 rows affected (0.001 sec)
```
The database is ready to be used.
Let's see if we can connect to the database from another computer:
```
lljubojevic@Omen17:~$ mysql -h 192.168.122.77 -u webserver -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 39
Server version: 10.3.32-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```
# Creating a simple Django web app
We will create a web application on the host computer in VS Code.
Before creating a web app, you need to install packages to connect to the mysql database:
```
lljubojevic@Omen17:~$ sudo apt install mariadb-client-core-10.3
lljubojevic@Omen17:~/PZ MMOS/Skladiste/skladiste$ pip install mysqlclient
```
Then we create a folder which will contain the web app
```
lljubojevic@Omen17:~/PZ MMOS$ mkdir Skladiste
lljubojevic@Omen17:~/PZ MMOS$ cd Skladiste
lljubojevic@Omen17:~/PZ MMOS/Skladiste$ django-admin startproject skladiste
lljubojevic@Omen17:~/PZ MMOS/Skladiste$ ls
skladiste
```
Before you start creating a web application, you need to change the database from SQLite to MySQL.
(By default in Django uses SQLite database).

In the settings.py file, change DATABASES as follows:
```
lljubojevic@Omen17:~/PZ MMOS/Skladiste$ cd skladiste/skladiste
lljubojevic@Omen17:~/PZ MMOS/Skladiste/skladiste/skladiste$ nano settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'Skladiste',
        'HOST': '192.168.122.77',
        'PORT': '3306',
        'USER': 'webserver',
        'PASSWORD': 'lozinka1',
    }
}
```
Where:
* 'NAME' is the name of the database on the MySQL VM, 
* 'HOST' IP address of VM MySQL, 
* 'PORT' default MySQL - Mariadb port, 
* 'USER' i 'PASSWORD' user that has rights to edit the database

We start creating a web application by creating a main app:
```
lljubojevic@Omen17:~/PZ MMOS/Skladiste/skladiste$ django-admin startapp main
```
The application will display the status of the goods in the warehouse.

Within the structure of the application we add the following code in the following files:

**main/urls.py:**
```
from django.urls import path
from . import views

app_name = 'main'

urlpatterns = [
    path('homepage', views.homepage, name='homepage'),
]

```
**main/views.py:**
```
from django.shortcuts import render
from main.models import Artikl

def homepage(request):
    artikls = Artikl.objects.all()
    context = {'artikls': artikls}
    return render(request, 'artikli.html', context=context)
```
**main/models.py:**
```
from django.db import models

# Create your models here.
class Artikl(models.Model):
    SifraArt=models.IntegerField(primary_key=True)
    NazivArt=models.CharField(max_length=350)
    KolNaStanju=models.IntegerField()
    Lokacija=models.CharField(max_length=400)
    Dostupan=models.BooleanField(default=True)

class Meta:
        db_table = 'Skladiste'
```
**main/admin.py:**
```
from django.contrib import admin
from django.db import models
from main.models import Artikl

# Register your models here.
admin.site.register(Artikl)
```
**main/templates/artikli.html:**
```
<ul>
    {% for a in artikls %}
        <li>
            Sifra Artikla: {{ a.SifraArt }}<br>
            Naziv Artikla: {{ a.NazivArt }}<br>
            Kolicina na stanju:{{ a.KolNaStanju }}<br>
            Lokacija:{{ a.Lokacija }} <br>
            Dostupan:{{ a.Dostupan }} 
        </li><br>
    {% endfor %}
</ul>
```
**skladiste/urls.py:**
```
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('main/', include('main.urls')),
]

```
**skladiste/settings.py:**
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'main.apps.MainConfig'
]
```
We get a web app that looks something like this:
![](https://i.imgur.com/AQD1i2C.png)

# Virtual machines WS1 and WS2 (web servers 1 and 2)
After a successfull log in into the systems from the virt-manager tool, the configuration of both servers serving the same web application begins.

We first check the IP addresses of the virtual machines:

WS1:
```
ubuntu@ubuntu:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:d7:39:a7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.133/24 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 3524sec preferred_lft 3524sec
    inet6 fe80::5054:ff:fed7:39a7/64 scope link 
       valid_lft forever preferred_lft forever

```

WS2:
```
ubuntu@ubuntu:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:fe:8c:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.115/24 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 3494sec preferred_lft 3494sec
    inet6 fe80::5054:ff:fefe:8c56/64 scope link 
       valid_lft forever preferred_lft forever

```
Also, for simplicity, we will connect to virtual machines from the host via the ssh command.

We install Python3 interpreter and Django framework on both web servers:
```
ubuntu@ubuntu:~$ sudo apt update
ubuntu@ubuntu:~$ sudo apt install python3 python3-pip python-is-python3 pylint
ubuntu@ubuntu:~$ sudo pip3 install Django
Collecting Django
  Downloading Django-4.0.1-py3-none-any.whl (8.0 MB)
     |████████████████████████████████| 8.0 MB 2.8 MB/s 
Collecting backports.zoneinfo; python_version < "3.9"
  Downloading backports.zoneinfo-0.2.1-cp38-cp38-manylinux1_x86_64.whl (74 kB)
     |████████████████████████████████| 74 kB 176 kB/s 
Collecting sqlparse>=0.2.2
  Downloading sqlparse-0.4.2-py3-none-any.whl (42 kB)
     |████████████████████████████████| 42 kB 747 kB/s 
Collecting asgiref<4,>=3.4.1
  Downloading asgiref-3.4.1-py3-none-any.whl (25 kB)
Installing collected packages: backports.zoneinfo, sqlparse, asgiref, Django
Successfully installed Django-4.0.1 asgiref-3.4.1 backports.zoneinfo-0.2.1 sqlparse-0.4.2

```
We set up the root directory of the web application on BOTH web servers:

We create a virtual CD image that contains all the files of our web app:
```
lljubojevic@Omen17:~/PZ MMOS/Skladiste$ mkisofs -o webapp.iso ./skladiste
I: -input-charset not specified, using utf-8 (detected in locale settings)
Total translation table size: 0
Total rockridge attributes bytes: 0
Total directory bytes: 14528
Path table size(bytes): 118
Max brk space used 22000
208 extents written (0 MB)

```
Within the virt-manager tool, we add the CD-ROM image to virtual machines:
![](https://i.imgur.com/48dbmdh.png)

We mount the CD-ROM to the system with commands:
```
ubuntu@ubuntu:/dev$ cd /media
ubuntu@ubuntu:/media$ sudo mkdir cd
ubuntu@ubuntu:/media$ sudo mount /dev/sr0 /media/cd
mount: /media/cd: WARNING: device write-protected, mounted read-only.

```
All items from the CD-ROM image we copy to te home directory of the server:
```
ubuntu@ubuntu:~$ cd /media/cd
ubuntu@ubuntu:/media/cd$ cp -r . /home/ubuntu/webapp
ubuntu@ubuntu:/media/cd$ cd /home/ubuntu/webapp
ubuntu@ubuntu:~/webapp$ ls
main  manage.py  skladiste

```
Yes, I now know that you can use [sftp](https://linuxize.com/post/how-to-use-linux-sftp-command-to-transfer-files/).. and that it is much easier. :/

On the servers we install MySQL clients (for fetching data):
```
ubuntu@ubuntu:~/webapp$ sudo apt install mariadb-client-core-10.3
ubuntu@ubuntu:~/webapp$ sudo apt-get install python3-dev default-libmysqlclient-dev build-essential
ubuntu@ubuntu:~/webapp$ sudo pip3 install mysqlclient
Collecting mysqlclient
  Using cached mysqlclient-2.1.0.tar.gz (87 kB)
Building wheels for collected packages: mysqlclient
  Building wheel for mysqlclient (setup.py) ... done
  Created wheel for mysqlclient: filename=mysqlclient-2.1.0-cp38-cp38-linux_x86_64.whl size=109128 sha256=7d74ec4c97da034715dcdb88d94a14c6d3ec0fbbc505e94c3ace8d659a388b4a
  Stored in directory: /root/.cache/pip/wheels/61/e7/42/9d56347e42d7ce19397c0ca050c6bef56640e18be7021ac189
Successfully built mysqlclient
Installing collected packages: mysqlclient
Successfully installed mysqlclient-2.1.0

```
We add our servers and the load balancer to allowed hosts in the settings.py file:
```
#WS1
ubuntu@ubuntu:~/webapp/skladiste$ sudo nano settings.py
ALLOWED_HOSTS = ['192.168.122.133', '192.168.122.132']

#WS2
ubuntu@ubuntu:~/webapp/skladiste$ sudo nano settings.py
ALLOWED_HOSTS = ['192.168.122.115','192.168.122.132']

```
We start to host the web app on both servers:
```
#WS1:
ubuntu@ubuntu:~/webapp$ ./manage.py runserver 192.168.122.133:8000

#WS2:
ubuntu@ubuntu:~/webapp$ ./manage.py runserver 192.168.122.115:8000
```

We remove the CD image from both servers in the virt-manager tool and test the servers.

WS1:
![](https://i.imgur.com/PFaHHYh.png)


WS2:
![](https://i.imgur.com/eRvCnNp.png)


# Virtual machine LB (Load balancer)
After a successfull log in into the system from the virt-manager tool, the load balancer configuration begins.

First, We check the IP address of the virtual machine:

```
ubuntu@ubuntu:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:fd:36:0d brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.132/24 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 3503sec preferred_lft 3503sec
    inet6 fe80::5054:ff:fefd:360d/64 scope link 
       valid_lft forever preferred_lft forever

```

We will establish a ssh connection with the VM:
```
ubuntu@ubuntu:~$ sudo apt update.
ubuntu@ubuntu:~$ sudo apt install haproxy

```

In the file /etc/haproxy/haproxy.cfg we add the servers configuration:
```
frontend skladiste
        bind :8000
        default_backend sklserveri
backend sklserveri
        server ws1 192.168.122.133:8000
        server ws2 192.168.122.115:8000

```
And restart the HAProxy service:
```
ubuntu@ubuntu:~$ sudo service haproxy restart
```
We can now access web servers via the load balancer:
![](https://i.imgur.com/2V5ONnO.png)

# Virtualn Machine Windows client
As an example of a architecture database - servers - load balancer - clients, we will use multiple clients on different operating systems.

We installed on one virtual machine Windows 7 OS, install options are all defaults.

Web application from a client running the Windows 7 operating system:
![](https://i.imgur.com/6Gj4f1A.png)


# Virtual Machine GNU Linux - Ubutnu client
As an example of a architecture database - servers - load balancer - clients, we will use multiple clients on different operating systems.

We installed on one virtual machine Ubuntu 20.04 OS, install options are all defaults.

Web application from a client running the Ubuntu operating system:
![](https://i.imgur.com/TP5e2CR.png)


# Virtual Machine Androidx86 Client
As an example of a architecture database - servers - load balancer - clients, we will use multiple clients on different operating systems.

We installed on one client Android x86
(Android for x86 processors - processors in typical computers).
Installation options are trivial, all defaults.

Web application from a client running the Android operating system:
![](https://i.imgur.com/WYYrEM7.png)
