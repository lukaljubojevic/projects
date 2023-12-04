# Setting up redis replication (master-slave) in k8s cluster

Let's say we want to create a HA redis k8s cluster. To keep things simple we will assume that the cluster will be used for internal use only. Since this is a simulation task we will not create any firewall rules for inbound/outbound connection, ssh limits for users etc which we would do in a production environment.

To setup a ecosystem like that - simplest as possible - we will need:

* One master server (dev-master) from which we can access the k8s cluster (in this case a VM running kubernetes)
* One k8s server running our cluster with a port forwared
* One kibana server for visualization

## Preparing our environment

### Dev-master

First we will need to setup our dev-master.
We will use a Ubuntu server iso install image and we will setup a Ansible playbook that will prepare that server for us (post-install).

After basic installation we will generate a SSH key pair and add public key to known hosts:

```shell
lljubojevic@IdeaPad-5-Pro:~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/lljubojevic/.ssh/id_rsa): /home/lljubojevic/dev-id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/lljubojevic/dev-id_rsa
Your public key has been saved in /home/lljubojevic/dev-id_rsa.pub
The key fingerprint is:
SHA256:QxCAkO0qZMANNwWEi+uA3C9DpHtEorxSU0XGGArpFtg lljubojevic@IdeaPad-5-Pro
The key's randomart image is:
+---[RSA 3072]----+
|=B=*=B=.         |
|==Eoo.o.         |
|+.+  .  .        |
|.B.o.  .         |
|Oo*.    S        |
|B+o+     .       |
|+.=..            |
|.+ + .           |
|. . o            |
+----[SHA256]-----+
```

And then we will input the public key on our master:

```shell
dev@dev-master:~$ vim .ssh/authorized_keys
```

We will repeat the same procedure for the root user.

Let's begin with the Ansible playbook for setting up our dev-master.

The playbook will install openssh server, git, ansible, nano, vim, rsync and iptables and make the directory for our kubernetes scripts:

```yaml
---
    - name: dev-master setup
      hosts: dev-master
      become: yes
      tasks:
      - name: update packages
        ansible.builtin.apt:
          update_cache: yes
      - name: Download all packages
        ansible.builtin.apt:
          name:
            - ansible
            - git
            - nano
            - vim
            - rsync
            - iptables
            - git
          state: latest
      - name: create k8s scripts dir
        ansible.builtin.file:
          path: /home/dev/k8s-scripts
          state: directory
          mode: 0777
          owner: dev
```

Now we need to edit /etc/ansible/hosts and add the IP of our dev master:

```text
[dev-master]
root@192.168.1.175
```

Let's run the playbook for dev-master:

```shell
lljubojevic@IdeaPad-5-Pro:~$ ansible-playbook --private-key /home/lljubojevic/dev-id_rsa server-dev.yaml
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [dev-master setup] ***************************************************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************************************************************************************************************************************************************
ok: [root@192.168.1.175]

TASK [update packages] ****************************************************************************************************************************************************************************************************************************************************************************************************
changed: [root@192.168.1.175]

TASK [Download all packages] **********************************************************************************************************************************************************************************************************************************************************************************************
ok: [root@192.168.1.175]

TASK [create k8s scripts dir] *********************************************************************************************************************************************************************************************************************************************************************************************
ok: [root@192.168.1.175]

PLAY RECAP ****************************************************************************************************************************************************************************************************************************************************************************************************************
root@192.168.1.175         : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### k8s-cluster

Now we need our kubernetes cluster server. We will use the same image and an Ansible playbook that will prepare the server for us.

First we need to generate generate ssh key on dev-master and write that pubkey on our k8s server:

```shell
dev@dev-master:~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/dev/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/dev/.ssh/id_rsa
Your public key has been saved in /home/dev/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:Vh0Nsh0bq/oEfMudCp3CtcPzZT0PkipYyruN/SIELX0 dev@dev-master
The key's randomart image is:
+---[RSA 3072]----+
|          . =o   |
|           = *.  |
|     o    o =    |
|    o o.E. .     |
|     o .S +      |
|      .o.O = o . |
|     o += @ = +..|
|      =+o= * + .o|
|      ++o+= .   .|
+----[SHA256]-----+

kube@k8s-cluster:~$ sudo cat /root/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCee5rqsQD7QqAPzyP/T8x+/zGWdjGM4/ycPe2+yEO06OClIwk10Y39lp//un5hzJfz1g1VfotQG/t/7AZApmeUOBMR7j40EgVFXAPchUibkeN2FxC7oB7JfgP5hfQy3VKUCswiYR5DiXzh6KWvWdU3Qb8iNJ0wUCacqZQaB5gN/8sZGlYruceukxM3DFVlM7bN0AgAOPmDUQ9UXFx5lq+qJezSfUxTiQ/0/TN3cJktwtsKuSo5lCntceLHGMp2n2vGE/oXWcibnVFtdYQ20eOgFgmCXTcaZ3aJrhmc5yssQyKRXbRkeXhJnKAmUCDOjUShQm1efKWELXb43FoiOhwyFWe97P+g1uqX9wNgQntdMD2EOFQiZ+rahoseTPE5sYXYB+e53AwzmYfVRf5J+fC+dLS7nokb8nLbSMkT9ZvcdhL9UnrdXl6S8d9TEzb+PFt2J5tn6LuNeDuUAfTzgVgXnvLV4Mn+VUZ0cJ8duioy0pcn+AGpIUgW8lzV/lFiVAM= dev@dev-master
```

And now we will set it up using Ansible:

```shell
dev@dev-master:~/ansible-scripts$ cat setup-k8s.yml
---
    - name: k8s setup
      hosts: k8s
      become: yes
      tasks:
      - name: update packages
        ansible.builtin.apt:
          update_cache: yes
          upgrade: yes
      - name: Download all packages
        ansible.builtin.apt:
          name:
            - kubectl
            - iptables
            - rsync
            - virtualbox
          state: latest
      - name: download minikube installer
        ansible.builtin.command:
          cmd: curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
      - name: install minikube
        ansible.builtin.command:
          cmd: sudo dpkg -i minikube_latest_amd64.deb

dev@dev-master:~/ansible-scripts$ ansible-playbook setup-k8s.yml

PLAY [k8s setup] **********************************************************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************************************************************************************************************************************************************
ok: [root@192.168.1.128]

TASK [update packages] ****************************************************************************************************************************************************************************************************************************************************************************************************
[WARNING]: The value "True" (type bool) was converted to "'True'" (type string). If this does not look like what you expect, quote the entire value to ensure it does not change.
ok: [root@192.168.1.128]

TASK [Download all packages] **********************************************************************************************************************************************************************************************************************************************************************************************
ok: [root@192.168.1.128]

TASK [download minikube installer] ****************************************************************************************************************************************************************************************************************************************************************************************
[WARNING]: Consider using the get_url or uri module rather than running 'curl'.  If you need to use command because get_url or uri is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.
changed: [root@192.168.1.128]

TASK [install minikube] ***************************************************************************************************************************************************************************************************************************************************************************************************
[WARNING]: Consider using 'become', 'become_method', and 'become_user' rather than running sudo
changed: [root@192.168.1.128]

PLAY RECAP ****************************************************************************************************************************************************************************************************************************************************************************************************************
root@192.168.1.128         : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Now we need a script that will "push" our kubernetes script on our k8s cluster. We will create an Anisble playbook for that:

```shell
dev@dev-master:~/ansible-scripts$ cat conf-push.yaml
---
- name: Copy and chmod +x files
  hosts: k8s
  become: yes

  vars:
    source_directory: "/home/dev/k8s-scripts"
    destination_directory: "/home/kube/"

  tasks:
    - name: copy files
      ansible.builtin.copy:
        src: "{{ source_directory }}"
        dest: "{{ destination_directory }}"
        owner: "kube"
        mode: "0777"
      register: copy_result

    - name: execute permission
      ansible.builtin.find:
        paths: "{{ destination_directory }}"
        recurse: true
        patterns: "*"
      register: files_to_chmod

    - name: make exec
      ansible.builtin.file:
        path: "{{ item.path }}"
        mode: "0755"
      with_items: "{{ files_to_chmod.files }}"

dev@dev-master:~/ansible-scripts$ ansible-playbook conf-push.yaml

PLAY [Copy and chmod +x files] ********************************************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************************************************************************************************************************************************************
ok: [root@192.168.1.128]

TASK [copy files] *********************************************************************************************************************************************************************************************************************************************************************************************************
changed: [root@192.168.1.128]

TASK [execute permission] *************************************************************************************************************************************************************************************************************************************************************************************************
ok: [root@192.168.1.128]

TASK [make exec] **********************************************************************************************************************************************************************************************************************************************************************************************************
ok: [root@192.168.1.128] => (item={'path': '/home/kube/.cache/motd.legal-displayed', 'mode': '0755', 'isdir': False, 'ischr': False, 'isblk': False, 'isreg': True, 'isfifo': False, 'islnk': False, 'issock': False, 'uid': 1000, 'gid': 1000, 'size': 0, 'inode': 3281033, 'dev': 64768, 'nlink': 1, 'atime': 1699632043.4759989, 'mtime': 1699632043.4759989, 'ctime': 1699633254.4812367, 'gr_name': 'kube', 'pw_name': 'kube', 'wusr': True, 'rusr': True, 'xusr': True, 'wgrp': False, 'rgrp': True, 'xgrp': True, 'woth': False, 'roth': True, 'xoth': True, 'isuid': False, 'isgid': False})
ok: [root@192.168.1.128] => (item={'path': '/home/kube/.ssh/authorized_keys', 'mode': '0755', 'isdir': False, 'ischr': False, 'isblk': False, 'isreg': True, 'isfifo': False, 'islnk': False, 'issock': False, 'uid': 1000, 'gid': 1000, 'size': 569, 'inode': 3281025, 'dev': 64768, 'nlink': 1, 'atime': 1699632183.1219516, 'mtime': 1699632143.8645444, 'ctime': 1699633254.6612349, 'gr_name': 'kube', 'pw_name': 'kube', 'wusr': True, 'rusr': True, 'xusr': True, 'wgrp': False, 'rgrp': True, 'xgrp': True, 'woth': False, 'roth': True, 'xoth': True, 'isuid': False, 'isgid': False})
changed: [root@192.168.1.128] => (item={'path': '/home/kube/k8s-scripts/_init_variables.sh', 'mode': '0777', 'isdir': False, 'ischr': False, 'isblk': False, 'isreg': True, 'isfifo': False, 'islnk': False, 'issock': False, 'uid': 1000, 'gid': 0, 'size': 34, 'inode': 2097172, 'dev': 64768, 'nlink': 1, 'atime': 1699633363.0481238, 'mtime': 1699633362.8001263, 'ctime': 1699633363.0481238, 'gr_name': 'root', 'pw_name': 'kube', 'wusr': True, 'rusr': True, 'xusr': True, 'wgrp': True, 'rgrp': True, 'xgrp': True, 'woth': True, 'roth': True, 'xoth': True, 'isuid': False, 'isgid': False})
changed: [root@192.168.1.128] => (item={'path': '/home/kube/k8s-scripts/00-minikube-up.sh', 'mode': '0777', 'isdir': False, 'ischr': False, 'isblk': False, 'isreg': True, 'isfifo': False, 'islnk': False, 'issock': False, 'uid': 1000, 'gid': 0, 'size': 735, 'inode': 2097171, 'dev': 64768, 'nlink': 1, 'atime': 1699633283.168927, 'mtime': 1699633253.5172474, 'ctime': 1699633363.4121203, 'gr_name': 'root', 'pw_name': 'kube', 'wusr': True, 'rusr': True, 'xusr': True, 'wgrp': True, 'rgrp': True, 'xgrp': True, 'woth': True, 'roth': True, 'xoth': True, 'isuid': False, 'isgid': False})

PLAY RECAP ****************************************************************************************************************************************************************************************************************************************************************************************************************
root@192.168.1.128         : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Test:

```shell
kube@k8s-cluster:~/k8s-scripts$ ls
00-minikube-up.sh  _init_variables.sh
```

Now lets create our kubernetes environment:

```shell
kube@k8s-cluster:~/k8s-scripts$ ./00-minikube-up.sh
/usr/bin/minikube
/usr/bin/kubectl
❗  These changes will take effect upon a minikube delete and then a minikube start
🤷  Profile "minikube" not found. Run "minikube profile list" to view all profiles.
👉  To start a cluster, run: "minikube start"
😄  minikube v1.32.0 on Ubuntu 22.04
✨  Using the virtualbox driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🔥  Creating virtualbox VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.21.2 on Docker 24.0.7 ...
❌  Unable to load cached images: loading cached images: stat /home/kube/.minikube/cache/images/amd64/registry.k8s.io/pause_3.4.1: no such file or directory
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node minikube-m02 in cluster minikube
🔥  Creating virtualbox VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.59.103
🐳  Preparing Kubernetes v1.21.2 on Docker 24.0.7 ...
    ▪ env NO_PROXY=192.168.59.103
🔎  Verifying Kubernetes components...

❗  /usr/bin/kubectl is version 1.28.3, which may have incompatibilities with Kubernetes 1.21.2.
    ▪ Want kubectl v1.21.2? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
💡  metrics-server is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/metrics-server/metrics-server:v0.6.4
🌟  The 'metrics-server' addon is enabled

Cool! MiniKube is up on 192.168.59.103 address.
```

We are ready for the tasks.

## Redis master-slave cluster

Let's create a redis master-slave replication initialization script:

```shell
dev@dev-master:~/k8s-scripts$ cat 01-redis-up.sh
#!/usr/bin/env bash

export NS_DEF=namespace-def.yaml
export NS_NAME=redis
export CONTEXT=ha-redis
echo
echo "Current namespaces"
kubectl get namespaces
echo
echo "Creating namespace $NS_DEF"
kubectl create -f $NS_DEF
echo "\nIs it created?"
kubectl get namespaces --show-labels | grep $NS_NAME
echo 
echo "Creting context $CONTEXT"
kubectl config set-context $CONTEXT --namespace=redis --cluster=minikube --user=minikube
echo 
echo "Switching to context $CONTEXT"
kubectl config use-context $CONTEXT
echo 
echo "Whats my current context?"
kubectl config current-context
echo 
echo "Applying redis-master and redis-slave deployments"
kubectl apply -f redis-master.yaml
kubectl apply -f redis-slave.yaml
echo 
echo "Fetching deployment/pod data"
kubectl get deployments.v1.apps
echo 
kubectl get pods
echo 
kubectl get deployment
echo 
echo "Done!"

```

Where redis-master.yaml contains:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  namespace: redis
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      nodeName: minikube
      subdomain: redis-master
      containers:
      - name: redis-master
        image: redis:latest
        ports:
          - containerPort: 6379
        command:
          - redis-server
        args:
          - "--protected-mode"
          - "no"
        volumeMounts:
        - mountPath: /data
          name: redis-master-data
      volumes:
      - name: redis-master-data
        hostPath:
          path: /redis-data
---
apiVersion: v1
kind: Service
metadata:
  name: redis-master
spec:
  selector:
    name: redis-master
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
    name: redis
  type: NodePort
```

and redis-slave.yaml contains:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  namespace: redis
  labels:
    name: redis-slave
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis-slave
  template:
    metadata:
      labels:
        name: redis-slave
    spec:
      nodeName: minikube-m02
      subdomain: redis-slave
      containers:
      - name: redis-slave
        image: redis:latest
        ports:
        - containerPort: 6379
        command:
          - redis-server
        args:
          - "--replicaof"
          - "redis-master.redis.svc.cluster.local"
          - "6379"
          - "--protected-mode"
          - "no"
      volumes:
      - emptyDir: {}
        name: redis-slave-data
---
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  namespace: redis
spec:
  selector:
    name: redis-slave
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
    name: redis
  type: NodePort
```

Let's run that script:

```shell
kube@k8s-cluster:~/k8s-scripts$ ./01-redis-up.sh

Current namespaces
NAME              STATUS   AGE
default           Active   36m
kube-node-lease   Active   36m
kube-public       Active   36m
kube-system       Active   36m

Creating namespace namespace-def.yaml
namespace/redis created
\nIs it created?
redis             Active   0s    kubernetes.io/metadata.name=redis,name=redis-HA-cluster

Creting context ha-redis
Context "ha-redis" created.

Switching to context ha-redis
Switched to context "ha-redis".

Whats my current context?
ha-redis

Applying redis-master and redis-slave deployments
deployment.apps/redis-master created
service/redis-master created
deployment.apps/redis-slave created
service/redis-slave created

Fetching deployment/pod data
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   0/1     1            0           1s
redis-slave    0/1     1            0           0s

NAME                            READY   STATUS              RESTARTS   AGE
redis-master-66ff4959c6-nfqm9   0/1     ContainerCreating   0          1s
redis-slave-964cc85b6-9wfbj     0/1     ContainerCreating   0          0s

NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   0/1     1            0           1s
redis-slave    0/1     1            0           0s

Done!
```

Checking if all went right:

```shell
kube@k8s-cluster:~/k8s-scripts$ kubectl get svc
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
redis-master   NodePort   10.101.28.88    <none>        6379:30126/TCP   14s
redis-slave    NodePort   10.105.81.207   <none>        6379:30225/TCP   13s
```

Pods have started on different nodes:

```shell
kube@k8s-cluster:~/k8s-scripts$ kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
redis-master-66ff4959c6-nfqm9   1/1     Running   0          42s   10.244.0.8   minikube       <none>           <none>
redis-slave-964cc85b6-9wfbj     1/1     Running   0          41s   10.244.1.8   minikube-m02   <none>           <none>
```

Can we access redis from the host?

```shell
kube@k8s-cluster:~/k8s-scripts$ minikube ip
192.168.59.103

kube@k8s-cluster:~/k8s-scripts$ redis-cli -h 192.168.59.103 -p 30126 set luka 1
OK
kube@k8s-cluster:~/k8s-scripts$ redis-cli -h 192.168.59.103 -p 30126 set pero 2
OK
```

Yes, we can.

Does replication work?

```shell
kube@k8s-cluster:~/k8s-scripts$ redis-cli -h 192.168.59.103 -p 30225 keys \*
1) "pero"
2) "luka"
Replication works!
```

Does our redis data survive pod restarts
<br>Master:

```shell
kube@k8s-cluster:~/k8s-scripts$ kubectl delete pod redis-master-66ff4959c6-nfqm9
pod "redis-master-66ff4959c6-nfqm9" deleted
kube@k8s-cluster:~/k8s-scripts$ kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
redis-master-66ff4959c6-xnpds   1/1     Running   0          10s   10.244.0.9   minikube       <none>           <none>
redis-slave-964cc85b6-2n47t     1/1     Running   0          84s   10.244.1.9   minikube-m02   <none>           <none>
kube@k8s-cluster:~/k8s-scripts$ kubectl exec redis-master-66ff4959c6-xnpds -- redis-cli keys \*
luka
pero
```

Slave:

```shell
kube@k8s-cluster:~/k8s-scripts$ kubectl delete pod redis-slave-964cc85b6-9wfbj
pod "redis-slave-964cc85b6-9wfbj" deleted
kube@k8s-cluster:~/k8s-scripts$ kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
redis-master-66ff4959c6-nfqm9   1/1     Running   0          3m41s   10.244.0.8   minikube       <none>           <none>
redis-slave-964cc85b6-2n47t     1/1     Running   0          11s     10.244.1.9   minikube-m02   <none>           <none>
kube@k8s-cluster:~/k8s-scripts$ kubectl exec redis-slave-964cc85b6-2n47t -- redis-cli keys \*
luka
pero
```

Data is persistent.

Now we need a inverse script 98-redis-down.sh:

```shell
export NS_DEF=namespace-def.yaml
export NS_NAME=redis
export CONTEXT=ha-redis
echo "Current namespace state"
kubectl get namespaces --show-labels | grep $NS_NAME
echo 
echo "Switching to default context"
kubectl config use-context minikube
echo
echo "Removing context $CONTEXT"
kubectl config delete-context ha-redis --namespace=redis --cluster=minikube --user=minikube
echo
echo "Removing namespace $NS_NAME"
kubectl delete namespace $NS_NAME
echo
echo "Current context and namespace"
kubectl config current-context

kubectl get namespaces --show-labels | grep $NS_NAME
echo 
echo "Removed redis cluster"
```

## Redis (master) data import / clean script

Let's create a script for importing data:

```shell
kube@k8s-cluster:~/k8s-scripts$ cat redisImport.sh
#!/usr/bin/env bash
echo "Fetching minikube ip"
MINIKUBE=$(/usr/bin/minikube ip)
echo
echo "Fetching redis-master port"
P=$(/usr/bin/kubectl get svc redis-master |grep -v NAME|cut -f18 -d' '|cut -f2 -d':'|cut -f1 -d'/')
echo
echo "Inputing data"
cat ~/k8s-scripts/redis-data/data.txt | redis-cli -h  $MINIKUBE -p  $P
echo
echo "Data inputed"
```

Now we will use the script to input data:

```shell
kube@k8s-cluster:~/k8s-scripts$ ./redisImport.sh
Fetching minikube ip

Fetching redis-master port

Inputing data
OK
(integer) 1
(integer) 2
(integer) 42
OK
OK

Data inputed
```

Testing if data was inputed correctly:

```shell
kube@k8s-cluster:~/k8s-scripts$ redis-cli -h 192.168.59.103 -p 30126 keys \*
1) "Test"
2) "Hello"
3) "luka"
4) "pero"
5) "Answer"
6) "Dev"
```

Yes, and synced (test from slave).

Now we need an inverse script to purge all data form database:

redisClean.sh:

```shell
echo "Fetching minikube ip"
MINIKUBE=$(/usr/bin/minikube ip)
echo "Fetching redis-master port"
P_M=$(/usr/bin/kubectl get svc redis-master |grep -v NAME|cut -f18 -d' '|cut -f2 -d':'|cut -f1 -d'/')
echo "Flushing db"
redis-cli -h $MINIKUBE -p $P_M FLUSHALL
```

Test:

```shell
kube@k8s-cluster:~/k8s-scripts$ ./redisClean.sh
Fetching minikube ip
Fetching redis-master port
Flushing db
OK

kube@k8s-cluster:~/k8s-scripts$ redis-cli -h 192.168.59.103 -p 30609 keys \*
(empty array)
kube@k8s-cluster:~/k8s-scripts$ redis-cli -h 192.168.59.103 -p 32015 keys \*
(empty array)
```

Data purged and sync was sucessfull.

## Recurring tasks

To create a recurring tasks we use cron. Kubernetes has it's own cronjob pods.
Let's write a kubernetes cronjob for checking info replication output:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: info-replica
  namespace: redis
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: redis-info-cron
            image: redis
            imagePullPolicy: IfNotPresent
            command:
            - redis-cli
            - -h
            - redis-master.redis.svc.cluster.local
            - -p
            - "6379"
            - info
            - replication
          restartPolicy: OnFailure
```

And let's create a script that will deploy it:

```shell
kube@k8s-cluster:~/k8s-scripts$ cat 02-redis-info.sh
echo "Whats my current context?"
kubectl config current-context
echo
echo "Applying info cronjob"
kubectl apply -f redis-info.yaml
echo
echo "Cronjob created"
```

Now we can start the cronjob:

```shell
kube@k8s-cluster:~/k8s-scripts$ ./02-redis-info.sh
Whats my current context?
ha-redis

Applying info cronjob
cronjob.batch/info-replica created

Cronjob created
```

Testing cronjob output:

```shell
kube@k8s-cluster:~/k8s-scripts$ kubectl logs info-replica-28327752-xbv6g
# Replication
role:master
connected_slaves:1
slave0:ip=10.244.1.18,port=6379,state=online,offset=6273,lag=0
master_failover_state:no-failover
master_replid:22e674e662d47acd7e1b6f9ae48d96f6ca76268b
master_replid2:c0840af9cba71be3362a9bcb68d095b8669e6e41
master_repl_offset:6273
second_repl_offset:5419
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:5419
repl_backlog_histlen:855
```

Now we need a script to delete the cronjob as well:

```shell
kube@k8s-cluster:~/k8s-scripts$ cat 97-redis-info-downq.sh
echo "Current context"
kubectl config current-context
echo ""
echo "Deleting cronjob"
kubectl delete cronjob info-replica
echo
echo "Cronjobs left:"
kubectl get cronjob
echo ""
echo "Cronjob deleted"
```

We can now delete the cronjob:

```shell
kube@k8s-cluster:~/k8s-scripts$ ./97-redis-info-downq.sh
Current context
ha-redis

Deleting cronjob
cronjob.batch "info-replica" deleted

Cronjobs left:
No resources found in redis namespace.

Cronjob deleted
```

## Final polish

To make final optimizations we will add Redis master and slave ip's and ports in our env. For that we will use a script called _dev_vars.sh

```shell
kube@k8s-cluster:~/k8s-scripts$ cat _dev_vars.sh
#!/usr/bin/env bash
echo "Fetching minikube ip"
MINIKUBE=$(/usr/bin/minikube ip)
echo "Fetching redis-master port"
P_M=$(/usr/bin/kubectl get svc redis-master |grep -v NAME|cut -f18 -d' '|cut -f2 -d':'|cut -f1 -d'/')
P_S=$(/usr/bin/kubectl get svc redis-slave |grep -v NAME|cut -f18 -d' '|cut -f2 -d':'|cut -f1 -d'/')
echo "Exporting values to env"
export REDIS_MASTER_HOST_PORT=$MINIKUBE:$P_M
export REDIS_SLAVE_HOST_PORT=$MINIKUBE:$P_S
echo "Values exported"
```

Now we can load the _dev_vars.sh to load the script and it's variables (and make them persistent) in the current shell session.

```shell
kube@k8s-cluster:~/k8s-scripts$ . _dev_vars.sh
Fetching minikube ip
Fetching redis-master port
Exporting values to env
Values exported
```

Test:

```shell
kube@k8s-cluster:~/k8s-scripts$ printenv|grep REDIS_
REDIS_SLAVE_HOST_PORT=192.168.59.103:30225
REDIS_MASTER_HOST_PORT=192.168.59.103:30126
```

Variables loaded.

## Data visualizaton

We will now create the third VM on which we will setup elasticsearch, kibana, [redli](https://github.com/IBM-Cloud/redli) and HAProxy for auth.

First we install all the software:

```shell
elk@elk-monitoring:/tmp$ wget https://github.com/IBM-Cloud/redli/releases/download/v0.5.2/redli_0.5.2_linux_amd64.tar.gz
elk@elk-monitoring:/tmp$ sudo mv redli /usr/local/bin/
elk@elk-monitoring:~$ curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
[sudo] password for elk:
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK
elk@elk-monitoring:~$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
deb https://artifacts.elastic.co/packages/7.x/apt stable main
elk@elk-monitoring:~$ sudo apt update
elk@elk-monitoring:~$ sudo apt install kibana logstash
elk@elk-monitoring:~$ sudo apt install haproxy
elk@elk-monitoring:~$ sudo systemctl enable logstash
elk@elk-monitoring:~$ sudo systemctl enable kibana
elk@elk-monitoring:~$ sudo systemctl enable haproxy
elk@elk-monitoring:~$ sudo systemctl start kibana
elk@elk-monitoring:~$ sudo systemctl start elasticsearch.service 
elk@elk-monitoring:~$ curl -X GET "localhost:9200/"
{
  "name" : "elk-monitoring",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "I6oodhweQmKWz_CEikgBHw",
  "version" : {
    "number" : "7.17.14",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "774e3bfa4d52e2834e4d9d8d669d77e4e5c1017f",
    "build_date" : "2023-10-05T22:17:33.780167078Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
elk@elk-monitoring:~$ sudo vim /etc/elasticsearch/elasticsearch.yml
network.host: 0.0.0.0
discovery.type: single-node
elk@elk-monitoring:~$ sudo service elasticsearch restart
```

Then we need to setup port forwarding on our k8s cluster VM host:

```shell
kube@k8s-cluster:~/k8s-scripts$ sudo sysctl -w net.ipv4.ip_forward=1
[sudo] password for kube: 
net.ipv4.ip_forward = 1
kube@k8s-cluster:~/k8s-scripts$ sudo iptables -t nat -A PREROUTING -p tcp --dport 32015 -j DNAT --to-destination 192.168.59.103:32015
kube@k8s-cluster:~/k8s-scripts$ sudo iptables -t nat -A POSTROUTING -p tcp --dport 32015 -j MASQUERADE
```

This was made into a script and was saved as open-firewall.sh on k8s-cluster VM. ^

Now we can access Redis from anywhere using the k8s-cluster IP:

```shell
lljubojevic@IdeaPad-5-Pro:~/redis-ms$ redis-cli -h 192.168.1.128 -p 32015 monitor
OK
1699708045.073241 [0 10.99.87.207:6379] "ping"
```

We will now setup logstash and display Redis master metrics in Kibana.

```shell
elk@elk-monitoring:~$ sudo vim /etc/logstash/conf.d/redis.conf
elk@elk-monitoring:~$ cat /etc/logstash/conf.d/redis.conf
input {
        exec {
                command => "/usr/local/bin/redli -h 192.168.1.128 -p 32015 info"
                interval => 10
                type => "redis_info"
        }
}

filter {
        kv {
                value_split => ":"
                field_split => "\r\n"
                remove_field => [ "command", "message" ]
        }

        ruby {
                code =>
                "
                event.to_hash.keys.each { |k|
                        if event.get(k).to_i.to_s == event.get(k) # is integer?
                                event.set(k, event.get(k).to_i) # convert to integer
                        end
                        if event.get(k).to_f.to_s == event.get(k) # is float?
                                event.set(k, event.get(k).to_f) # convert to float
                        end
                }
                puts 'Ruby filter finished'
                "
        }
}

output {
    elasticsearch {
        hosts => "http://localhost:9200"
        index => "%{type}"
    }
}
```

Let's test our configuration:

```shell
elk@elk-monitoring:~$ sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/redis.conf
Using bundled JDK: /usr/share/logstash/jdk
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
WARNING: Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. You can specify the path using --path.settings. Continuing using the defaults
Could not find log4j2 configuration at path /usr/share/logstash/config/log4j2.properties. Using default config which logs errors to the console
[INFO ] 2023-11-12 11:00:09.106 [main] runner - Starting Logstash {"logstash.version"=>"7.17.14", "jruby.version"=>"jruby 9.2.20.1 (2.5.8) 2021-11-30 2a2962fbd1 OpenJDK 64-Bit Server VM 11.0.20+8 on 11.0.20+8 +indy +jit [linux-x86_64]"}
...
Ruby filter finished
Ruby filter finished
Ruby filter finished
Ruby filter finished
Ruby filter finished
```

Configuration works, we can now start logstash service and test elasticsearch output:

```shell
elk@elk-monitoring:~$ sudo systemctl start logstash

elk@elk-monitoring:~$ curl -ILX GET "http://localhost:9200/redis_info"
HTTP/1.1 200 OK
X-elastic-product: Elasticsearch
Warning: 299 Elasticsearch-7.17.14-774e3bfa4d52e2834e4d9d8d669d77e4e5c1017f "Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html to enable security."
content-type: application/json; charset=UTF-8
content-length: 10803

elk@elk-monitoring:~$ curl "http://localhost:9200/redis_info"
{"redis_info":{"aliases":{},"mappings":{"properties":{"@timestamp":{"type":"date"},"@version":{"type":"long"},"acl_access_denied_auth":{"type":"long"},"acl_access_denied_channel":{"type":"long"},"acl_access_denied_cmd":{"type":"long"},"acl_access_denied_key":{"type":"long"},"active_defrag_hits":{"type":"long"},"active_defrag_key_hits":{"type":"long"},"active_defrag_key_misses":{"type":"long"},"active_defrag_misses":{"type":"long"},"active_defrag_running":{"type":"long"},"allocator_active":{"type":"long"},"allocator_allocated":{"type":"long"},"allocator_frag_bytes":{"type":"long"},"allocator_frag_ratio":{"type":"float"},"allocator_resident":{"type":"long"},"allocator_rss_bytes":{"type":"long"},"allocator_rss_ratio":{"type":"float"},"aof_current_rewrite_time_sec":{"type":"long"},"aof_enabled":{"type":"long"},"aof_last_bgrewrite_status":{"type":"text","fields":{"keyword":{"type":"keyword","ignore_above":256}}},"aof_last_cow_size":{"type":"long"},"aof_last_rewrite_time_sec":{"type":"long"},"aof_last_write_status":{"type":"text","fields":{"keyword":{"type":"keyword","ignore_above":256}}},"aof_rewrite_in_progress":{"type":"long"},"aof_rewrite_scheduled":{"type":"long"},"aof_rewrites":{"type":"long"},"aof_rewrites_consecutive_failures":{"type":"long"},"arch_bits":{"type":"long"},
```

When that's setup, we can proceed with conifguring Kibana.

```shell
elk@elk-monitoring:~$ sudo vim /etc/kibana/kibana.yml
server.port: 5601
elasticsearch.hosts: ["http://localhost:9200"]
server.host: 0.0.0.0
elk@elk-monitoring:~$ service kibana restart
```

Now we connect to web UI and finish setting up:
![1](images/1.png)
![2](images/2.png)
![3](images/3.png)
![4](images/4.png)
![5](images/5.png)
![6](images/6.png)

### Auth using HAProxy

Lastly, let's setup auth using HAProxy

```shell
elk@elk-monitoring:~$ sudo vim /etc/haproxy/haproxy.cfg
elk@elk-monitoring:~$ sudo haproxy -c -f /etc/haproxy/haproxy.cfg
Configuration file is valid
elk@elk-monitoring:~$ cat /etc/haproxy/haproxy.cfg
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend kibana
        bind *:80
        mode http
        default_backend kibana_servers
        http-request auth unless { http_auth(login) }

backend kibana_servers
        mode http
        server kibana1 localhost:5601

userlist login
        user kibana insecure-password kibana
elk@elk-monitoring:~$ sudo systemctl start haproxy
elk@elk-monitoring:~$ sudo systemctl restart haproxy
elk@elk-monitoring:~$ curl -ILX GET "http://localhost:80"
HTTP/1.1 401 Unauthorized
content-length: 112
cache-control: no-cache
content-type: text/html
www-authenticate: Basic realm="kibana"
```

Test:

![haproxy login](images/7.png)
![login success](images/8.png)

Now we need to change back elasticsearch and kibana bind back to localhost only.

```shell
elk@elk-monitoring:~$ sudo vim /etc/kibana/kibana.yml
#server.host: 0.0.0.0
elk@elk-monitoring:~$ service kibana restart

elk@elk-monitoring:~$ sudo vim /etc/elasticsearch/elasticsearch.yml
# network.host: 0.0.0.0
#discovery.type: single-node
elk@elk-monitoring:~$ sudo service elasticsearch restart
```

Testing the configuration:

```shell
elk@elk-monitoring:~$ curl -ILX GET "http://localhost:9200/redis_info"
HTTP/1.1 200 OK
X-elastic-product: Elasticsearch
Warning: 299 Elasticsearch-7.17.14-774e3bfa4d52e2834e4d9d8d669d77e4e5c1017f "Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html to enable security."
content-type: application/json; charset=UTF-8
content-length: 10803

elk@elk-monitoring:~$ curl -ILX GET "http://localhost:5601/"
HTTP/1.1 302 Found
location: /spaces/enter
x-content-type-options: nosniff
referrer-policy: no-referrer-when-downgrade
content-security-policy: script-src 'unsafe-eval' 'self'; worker-src blob: 'self'; style-src 'unsafe-inline' 'self'
kbn-name: elk-monitoring
kbn-license-sig: ec470d9ae4dcb0a95cfbaf1dab1956763d7d9f4d6b2886015bf297eb5673197d
cache-control: private, no-cache, no-store, must-revalidate
content-length: 0
Date: Sun, 12 Nov 2023 11:59:32 GMT
Connection: keep-alive
Keep-Alive: timeout=120

HTTP/1.1 302 Found
location: /app/home
x-content-type-options: nosniff
referrer-policy: no-referrer-when-downgrade
content-security-policy: script-src 'unsafe-eval' 'self'; worker-src blob: 'self'; style-src 'unsafe-inline' 'self'
kbn-name: elk-monitoring
kbn-license-sig: ec470d9ae4dcb0a95cfbaf1dab1956763d7d9f4d6b2886015bf297eb5673197d
cache-control: private, no-cache, no-store, must-revalidate
content-length: 0
Date: Sun, 12 Nov 2023 11:59:32 GMT
Connection: keep-alive
Keep-Alive: timeout=120

HTTP/1.1 200 OK
x-content-type-options: nosniff
referrer-policy: no-referrer-when-downgrade
content-security-policy: script-src 'unsafe-eval' 'self'; worker-src blob: 'self'; style-src 'unsafe-inline' 'self'
kbn-name: elk-monitoring
kbn-license-sig: ec470d9ae4dcb0a95cfbaf1dab1956763d7d9f4d6b2886015bf297eb5673197d
content-type: text/html; charset=utf-8
cache-control: private, no-cache, no-store, must-revalidate
content-length: 145058
vary: accept-encoding
accept-ranges: bytes
Date: Sun, 12 Nov 2023 11:59:32 GMT
Connection: keep-alive
Keep-Alive: timeout=120

lljubojevic@IdeaPad-5-Pro:~/redis-ms$ curl -ILX GET "http://192.168.1.181:9200"
curl: (7) Failed to connect to 192.168.1.181 port 9200 after 112 ms: Connection refused
lljubojevic@IdeaPad-5-Pro:~/redis-ms$ curl -ILX GET "http://192.168.1.181:5601"
curl: (7) Failed to connect to 192.168.1.181 port 5601 after 74 ms: Connection refused

lljubojevic@IdeaPad-5-Pro:~/redis-ms$ curl -ILX GET "http://192.168.1.181"
HTTP/1.1 401 Unauthorized
content-length: 112
cache-control: no-cache
content-type: text/html
www-authenticate: Basic realm="kibana"
```
