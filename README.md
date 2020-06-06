# Intro
>Questo corso è pienamente funzionale su una macchina virtuale (VirtualBox, VMware o KVM) con un sistema operativo Ubuntu (Desktop o Server) LTS 18.04 o superiore
## Minicorso Docker

https://docs.docker.com/get-started/

### installa docker nel sistema linux ubuntu 18.04
``$ sudo apt update && sudo apt upgrade -y && sudo apt install docker docker.io``

>opzionalmente potete creare l'utente docker con il comando 'addsuser docker' che si occuperà di creare anche il gruppo docker di cui l'utente farà parte

### Il mio utente è nel gruppo docker?

> *serve per capire di quali gruppi facciamo parte, in particolare vogliamo sapere se il nostro utente fa parte del gruppo docker*, di default gli utenti che fanno parte del gruppo **docker** possono usare i comandi docker senza i privilegi di root.
```
$ id
uid=1000(massimo) gid=1000(massimo) gruppi=1000(massimo),4(adm),24(cdrom),27(sudo),**1003(docker)**
```
### Controlliamo se esiste il gruppo docker
>Non è detto che l'installazione del pacchetto docker crei automaticamente il gruppo

``$ sudo cat /etc/group``

* Se esiste aggiungo l'utente al gruppo

  ``$ sudo usermod -aG docker $USER``

* Altrimenti creo anche il gruppo ed aggiungo l'utente
```
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

### Verifichiamo che docker è installato e che lo possiamo lanciare
``$ docker info``

### Creiamo il nostro account su https://hub.docker.com (ci servirà anche quando creeremo le nostri immagini docker personalizzate) e lo usiamo per fare il login come segue:
``$ docker login``

## Esercizio 01
### Creiamo il nostro primo container, vogliamo usare un DB
```
$ docker run --name=mysql1 -d mysql/mysql-server
Unable to find image 'mysql/mysql-server:latest' locally
latest: Pulling from mysql/mysql-server
0e690826fc6e: Pull complete
0e6c49086d52: Pull complete
862ba7a26325: Pull complete
7731c802ed08: Pull complete
Digest: sha256:a82ff720911b2fd40a425fd7141f75d7c68fb9815ec3e5a5a881a8eb49677087
Status: Downloaded newer image for mysql/mysql-server:latest
3cb5148b27b33874ee1be3c32b6dbf7d3b5d8383333cbafe1344c202ec70bb1e

$ docker container ls
CONTAINER ID        IMAGE                                COMMAND                  CREATED              STATUS                          PORTS                                 NAMES
3cb5148b27b3        mysql/mysql-server                   "/entrypoint.sh mysq…"   About a minute ago   Up About a minute (healthy)     3306/tcp, 33060/tcp                   mysql1

```
### Vogliamo vedere i "logs" di installazione in cui compare, per il container MySQL, la password creata automaticamente:
```
$ docker container logs mysql1 | grep GENERATED
  [Entrypoint] GENERATED ROOT PASSWORD: GawLYjBYBUx3vjANyfuvez-Ixop
```

> La password generata automaticamente è una one-time password, per poter utilizzare il container come BackEend bisogna cambiarla al primo login in MySQL.

#### Procediamo con il cambio della la password per l'utente root di mysql facendo il login con la password one-time generata del container
```
$ docker exec -it mysql1 bash
bash-4.2# mysql -u root -pGawLYjBYBUx3vjANyfuvez-Ixop
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 17

Server version: 8.0.20

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> alter user 'root'@'localhost' identified by 'password';
Query OK, 0 rows affected (0.02 sec)

mysql>
```

> In questo caso ho utilizzato una weak password, consiglio di utilizzare in produzione password più sicure e forti.

### Con il seguente comando si vedono i container che non sono in running

>Può succedere che un container vada in errore e va in status di EXIT o ERROR

``$ docker container ls -a``

### Installiamo diversi database anche di versioni diverse o precedenti

>La caratteristica più importante dei container è che ci permette di far coesistere interno allo stesso Sistema Operativo più processi di natura similare che non confliggono

```
$ docker container run --name=mysql2 -d mysql/mysql-server
$ docker container run --name=mysql5.7 -d mysql/mysql-server:5.7
$ docker container ls
```
### Controlliamo se mysql2 ha la stessa password generata di mysql1
``$ docker conatainer logs mysql2  | grep GENERATED``
```
[Entrypoint] GENERATED ROOT PASSWORD: !AwfEBC0vYHIGExUc2ehKUbumef
```

### ricavo l'ip del container
```
$ docker container inspect mysql2 | grep IPAddress
  "IPAddress": "172.17.0.3",

$ ping -c1 172.17.0.3
````
#### ricavo l'ip dell'host
```
$ ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
      valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:f7:2a:92 brd ff:ff:ff:ff:ff:ff
    inet **172.16.16.95/24** brd 172.16.16.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef7:2a92/64 scope link
       valid_lft forever preferred_lft forever

4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:95:2c:54:86 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:95ff:fe2c:5486/64 scope link
       valid_lft forever preferred_lft forever
```
### Voglio vedere i processi che girano dentro il container

>E' interessante capire cosa gira nel container in termini di processi, in effetti qui potete vedere che gira solo il processo di MySQL

```
$ docker container top mysql1

UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
27                  20372               20324               1                   09:57               ?                   00:00:13            mysqld --init-file=/var/lib/mysql-files/2AZpaJenaI

$ docker container top mysql2

UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
27                  21288               21263               1                   10:09               ?                   00:00:07            mysqld --init-file=/var/lib/mysql-files/Qmit2GAWht
```
### Proviamo a lanciare container con applicativi differenti, due web Server nginx
```
$ docker container run --name=web1 -d nginx
$ docker container run --name=web2 -d nginx
```

>NB. il secondo container viene eseguito più velocemente perché l'immaggine di nginx viene scaricata la prima volta che si lancia il comando

### I container possono essere messi in stop o cancellati, se faccio lo stop posso fare lo start, se cancello con rm il container è perduto
```
$ docker container stop web2
$ docker container start web2
$ docker  container inspect web2 | grep IPAddress
 "IPAddress": "172.17.0.4",
```
### Sapendo qual'è l'IP possiamo vedere se il servizio web serve funziona sulla porta 80 tramite il comando curl
```
$ curl 172.17.0.4

<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### vediamo in questo tipo di container cosa gira
```
$ docker container top web2
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                23605               23576               0                   10:26               ?                   00:00:00            nginx: master process nginx -g daemon off;
systemd+            23635               23605               0                   10:26               ?                   00:00:00            nginx: worker process
```
### con "docker container ls" vediamo anche le porte esposte dal container che differiscono per tipologia
```
$ docker container ls
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS                    PORTS                 NAMES
b512ffb058b7        nginx                "nginx -g 'daemon of…"   7 minutes ago       Up 7 minutes              80/tcp                web1
214689ef3c60        nginx                "nginx -g 'daemon of…"   8 minutes ago       Up 8 minutes              80/tcp                web2
70870e6db10f        mysql/mysql-server   "/entrypoint.sh mysq…"   25 minutes ago      Up 25 minutes (healthy)   3306/tcp, 33060/tcp   mysql2
9af523f1453d        mysql/mysql-server   "/entrypoint.sh mysq…"   38 minutes ago      Up 38 minutes (healthy)   3306/tcp, 33060/tcp   mysql1
```
## VOLUMES
### andiamo a vedere se i container MySQL hanno creato i propri volumi
```
$ docker volume ls
DRIVER              VOLUME NAME
local               bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006
local               d72bde5c8d9d483956cdddb4026b912a7e57d5f832b15c5b00abca874de759ca
```
### vogliamo sapere dove docker mette i files dei container
```
$ sudo find / 2> /dev/null | grep bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006

/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006
/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data
/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data/mysql
/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data/mysql/general_log.CSM
/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data/mysql/slow_log_203.sdi
/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data/mysql/slow_log.CSV
/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data/mysql/general_log_202.sdi
/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data/mysql/slow_log.CSM
/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data/mysql/general_log.CSV
[output omitted...]
```
### per andare a vedere i files dobbiamo essere root ed il comando per diventare root è il seguente
``$ sudo -i``

### per vedere se un volume fa parte di un container usiamo l'inspect come segue
```
$ docker container inspect mysql1 | grep bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006
                 "Name": "bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006",

                "Source": "/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data",
```
### dopo aver creato gli oggetti in docker questi sono i comandi per vederne alcuni di interesse
```
$ docker container ls -a
$ docker volume ls
$ docker image ls
$ docker network ls
$ ip a``
```

### con il seguente comando vediamo tutta la configurazione del container web2

``$ docker container inspect web2``

``$ ping -c1 172.17.0.3``

``$ sudo arp -a`` **con questo comando vediamo mac address table**

## Skill up
### passiamo una variabile d'ambiente (environment) al container per settare la password di root di MariaDB
``$ docker container run -d -e "MYSQL_ROOT_PASSWORD=password" --name db1 mariadb``

``$ docker container ls``

### voglio usare il servizio del container con un client mysql
```
$ sudo apt install mysql-client
$ docker container inspect db1 | grep IPAddress

"IPAddress": "172.17.0.2",

$ mysql -u root -p -h 172.17.0.2

Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.5.5-10.4.13-MariaDB-1:10.4.13+maria~bionic mariadb.org binary distribution

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)
```
###  facciamo un po' di pulizia con il **prune** che libera ciò che non è utilizzato
``$ docker container prune``

``$ docker volume prune``

``$ docker image prune``

``$ docker container ls -a``

### ora vogliamo creare il **FrontEnd** che si aggancia al BackEend con l'IP, user e password di MariaDB
``$ docker run --name frontend -e WORDPRESS_DB_HOST=172.17.0.2:3306 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password -d wordpress``
### controllo che il frontend interagisca con il backend creando il DB wordpress
```
$ mysql -u root -p -h 172.17.0.2

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wordpress          |
+--------------------+
4 rows in set (0.00 sec)
```
### vediamo la orte esposta dal FE
```
$ docker container ls -a

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
870a1740ac88        wordpress           "docker-entrypoint.s…"   6 minutes ago       Up 6 minutes        80/tcp              frontend
ace04ec47218        mariadb             "docker-entrypoint.s…"   14 minutes ago      Up 14 minutes       3306/tcp            db1
b512ffb058b7        nginx               "nginx -g 'daemon of…"   57 minutes ago      Up 57 minutes       80/tcp              web1
```
### controlliamo cosa gira dentro il container frontend
```
$ docker container top frontend

UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                29277               29254               0                   11:18               ?                   00:00:01            apache2 -DFOREGROUND
www-data            29498               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
www-data            29499               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
www-data            29500               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
www-data            29501               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
www-data            29502               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
```
## Per l'inoltro delle PORT, si aggiunge "-p 80:80" nella creazione del container
### prima lo rimuoviamo e poi lo ricreiamo con la dichiarazione della porta
``$ docker container rm -f frontend``

``$ docker run --name frontend -e WORDPRESS_DB_HOST=172.17.0.2:3306 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password -d -p 80:80 wordpress``
### altro esempio di come si pubblica una porta 80 sulla porta 81 dell'host
``$ docker container run --name web1 -p 81:80 -d nginx``

## VOLUMES
### voglio creare un volume "persistente" di nome "massimo" condiviso da più container
``$ docker volume create massimo``

``$ docker volume ls``

``$ docker run --name frontend -e WORDPRESS_DB_HOST=172.17.0.2:3306 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password -v massimo:/var/www/html -d -p 80:80 wordpress``

``$ docker container run -v massimo:/html -it --rm busybox``

### Creo un volume condiviso per editare la pagina di default di nginx
```
$ docker volume create nginx
$ docker container rm -f web1
$ docker container run -d --name web1 -v nginx:/usr/share/nginx/html nginx
$ docker container run -v nginx:/html -it --rm busybox
/ # cd html/
/html # ls
50x.html    index.html
/html # cat index.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to Massimo!</title>
<style>
    body {
      width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/html #
vi index.html
```

##Esercizio Docker compose
```
$ apt install -y docker-compose
```
### docker-compose mi aiuta a risparmiare tempo e mi introduce nell'"Infrastructure as a code"
``$ vi wp.yaml``
```
version: '2.0'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```
### abbiamo bisogno di installare docker compose
``$ sudo apt install -y docker-compose``

``$ docker-compose -f wp.yaml up -d``

``$ docker container ls``

``$ docker network ls``

``$ docker volume ls``

``$ vi rocket.yaml``
```
version: '2'

services:
  rocketchat:
    image: rocketchat/rocket.chat:latest
    command: >
      bash -c
        "for i in `seq 1 30`; do
          node main.js &&
          s=$$? && break || s=$$?;
          echo \"Tried $$i times. Waiting 5 secs...\";
          sleep 5;
        done; (exit $$s)"
    restart: unless-stopped
    volumes:
      - ./uploads:/app/uploads
    environment:
      - PORT=3000
      - ROOT_URL=http://localhost:3000
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - MONGO_OPLOG_URL=mongodb://mongo:27017/local
      - MAIL_URL=smtp://smtp.email
#       - HTTP_PROXY=http://proxy.domain.com
#       - HTTPS_PROXY=http://proxy.domain.com
    depends_on:
      - mongo
    ports:
      - 3000:3000
    labels:
      - "traefik.backend=rocketchat"
      - "traefik.frontend.rule=Host: your.domain.tld"

  mongo:
    image: mongo:4.0
    restart: unless-stopped
    volumes:
     - ./data/db:/data/db
     #- ./data/dump:/dump
    command: mongod --smallfiles --oplogSize 128 --replSet rs0 --storageEngine=mmapv1
    labels:
      - "traefik.enable=false"

  # this container's job is just run the command to initialize the replica set.
  # it will run the command and remove himself (it will not stay running)
  mongo-init-replica:
    image: mongo:4.0
    command: >
      bash -c
        "for i in `seq 1 30`; do
          mongo mongo/rocketchat --eval \"
            rs.initiate({
              \_id: 'rs0',
              members: [ { \_id: 0, host: 'localhost:27017' } ]})\" &&
          s=$$? && break || s=$$?;
          echo \"Tried $$i times. Waiting 5 secs...\";
          sleep 5;
        done; (exit $$s)"
    depends_on:
      - mongo

  # hubot, the popular chatbot (add the bot user first and change the password before starting this image)
  hubot:
    image: rocketchat/hubot-rocketchat:latest
    restart: unless-stopped
    environment:
      - ROCKETCHAT_URL=rocketchat:3000
      - ROCKETCHAT_ROOM=GENERAL
      - ROCKETCHAT_USER=bot
      - ROCKETCHAT_PASSWORD=botpassword
      - BOT_NAME=bot
  # you can add more scripts as you'd like here, they need to be installable by npm
      - EXTERNAL_SCRIPTS=hubot-help,hubot-seen,hubot-links,hubot-diagnostics
    depends_on:
      - rocketchat
    labels:
      - "traefik.enable=false"
    volumes:
      - ./scripts:/home/hubot/scripts
  # this is used to expose the hubot port for notifications on the host on port 3001, e.g. for hubot-jenkins-notifier
    ports:
      - 3001:8080

  #traefik:
  #  image: traefik:latest
  #  restart: unless-stopped
  #  command: >
  #    traefik
  #     --docker
  #     --acme=true
  #     --acme.domains='your.domain.tld'
  #     --acme.email='your@email.tld'
  #     --acme.entrypoint=https
  #     --acme.storagefile=acme.json
  #     --defaultentrypoints=http
  #     --defaultentrypoints=https
  #     --entryPoints='Name:http Address::80 Redirect.EntryPoint:https'
  #     --entryPoints='Name:https Address::443 TLS.Certificates:'
  #  ports:
  #    - 80:80
  #    - 443:443
  #  volumes:
  #    - /var/run/docker.sock:/var/run/docker.sock
```
``$ docker-compose -f rocket.yaml up -d``

``$ docker-compose -f rocket.yaml logs``

## NETWORK
### voglio vedere le network di docker la docker0 è la bridged, la host si riferisce all'ip dell'host
``$ docker network ls``

```
NETWORK ID          NAME                DRIVER              SCOPE
171d7f63e4a0        bridge              bridge              local
626776a1047d        host                host                local
517888c34e3a        none                null                local
```
### voglio crere una network differente per isolare alcuni container
``$ docker network create newnet1``

``$ docker network ls``
```
NETWORK ID          NAME                DRIVER              SCOPE
171d7f63e4a0        bridge              bridge              local
626776a1047d        host                host                local
76759826efad        newnet1             bridge              local
517888c34e3a        none                null                local
```
``$ docker run -it  --network newnet1 -v nginx:/html --rm busybox``
```
/ # ifconfig

eth0      Link encap:Ethernet  HWaddr 02:42:AC:15:00:02  
          inet addr:172.21.0.2  Bcast:172.21.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:14 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1156 (1.1 KiB)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # netstat -rn

Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         172.21.0.1      0.0.0.0         UG        0 0          0 eth0
172.21.0.0      0.0.0.0         255.255.0.0     U         0 0          0 eth0

/ # ping -c1 8.8.8.8

PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=127 time=21.327 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 21.327/21.327/21.327 ms

/ # ping -c1 www.google.it

PING www.google.it (216.58.206.35): 56 data bytes
64 bytes from 216.58.206.35: seq=0 ttl=127 time=13.114 ms

--- www.google.it ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 13.114/13.114/13.114 ms

/ # hostname
dc747114239f
```
### proviamo a lanciare un container nella rete host
``$ docker run -it  --network host -v nginx:/html --rm busybox``
```
/ # ifconfig

br-76759826efad Link encap:Ethernet  HWaddr 02:42:08:F7:90:D9  
          inet addr:172.21.0.1  Bcast:172.21.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:8ff:fef7:90d9/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:14 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:342 (342.0 B)  TX bytes:1118 (1.0 KiB)

docker0   Link encap:Ethernet  HWaddr 02:42:95:2C:54:86  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:95ff:fe2c:5486/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:2726 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3078 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:3201045 (3.0 MiB)  TX bytes:16402091 (15.6 MiB)

enp0s3    Link encap:Ethernet  HWaddr 08:00:27:F7:2A:92  
          inet addr:172.16.16.95  Bcast:172.16.16.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fef7:2a92/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1845604 errors:0 dropped:0 overruns:0 frame:0
          TX packets:209457 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2335248803 (2.1 GiB)  TX bytes:33265302 (31.7 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:172 errors:0 dropped:0 overruns:0 frame:0
          TX packets:172 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:13404 (13.0 KiB)  TX bytes:13404 (13.0 KiB)

virbr0    Link encap:Ethernet  HWaddr 52:54:00:6B:52:2A  
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # netstat -rn

Kernel IP routing table

Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         172.16.16.2     0.0.0.0         UG        0 0          0 enp0s3
172.16.16.0     0.0.0.0         255.255.255.0   U         0 0          0 enp0s3
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
172.21.0.0      0.0.0.0         255.255.0.0     U         0 0          0 br-76759826efad
192.168.122.0   0.0.0.0         255.255.255.0   U         0 0          0 virbr0

/ # hostname
NFServer
```
### nginx da file yaml e busybox

``$ vi nginx.yaml``
```
version: '2.0'

services:

  nginx:
    image: nginx
    restart: always
    ports:
      - 80:80
    environment:
      NGINX_HOST: nginx
      NGINX_USER: exampleuser
      NGINX_PASSWORD: examplepass
      NGINX_NAME: examplenginx
    volumes:
      - nginx:/usr/share/nginx/html

volumes:
  nginx:
```
### comandi per busybox

``$ docker container run -it -v annaleda_nginx:/html --rm busybox``
```
/ # cd html
/html # ls
50x.html    index.html
/html # cat index.html

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
### provare vi index.html
```
/html # echo 'ciao vincenzo giovanni ettore andre arturo'>>index.html
/html # exit
```
##Docker Image
### Build and run your image
```
$ git clone https://github.com/dockersamples/node-bulletin-board
$ cd node-bulletin-board/bulletin-board-app
$ ls

$ cat Dockerfile

$ docker build --tag bulletinboard:1.0 .
$ ls
$ docker image ls
$ docker container run --network host -d bulletinboard:1.0
$ docker container ls
$ docker login
```
>NB. se da problemi $ sudo apt install gnupg2 pass
### rendo pubblica (condivido) la docker creata

``$ docker tag bulletinboard:1.0 [account]/bulletinboard:1.0``

### trasferisco su dockerhub

``$ docker push [account]/bulletinboard:1.0``

### installo la docker caricata su dockerhub

``$ docker pull merlinogj/bulletinboard:1.0``

### creo una immagine semplice partendo da nginx e cambiando la pagina index.html
```
$ mkdir ah
$ vi Dockerfile

FROM nginx:latest
WORKDIR /usr/share/nginx/html
COPY index.html .
RUN apt update

$ vi index.html

<!DOCTYPE html>
<html>
<head>
<title>Welcome to New Image of nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to New Image of nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

$ docker build --tag maxnignx:1.1 .

$ docker image ls

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
maxnignx                  1.1                 e1105be7f239        3 hours ago         144MB

$ docker run -d -p 81:80 maxnignx:1.1
$ docker tag maxnignx:1.1 merlinogj/maxnginx:1.1
$ docker push merlinogj/maxnginx:1.1
```

### Cluster swarm
```
$ docker swarm init
Swarm initialized: current node (c6698xxyseo8nnb9rknz800k2) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-69832mntsgml7zpocn50wywj6cofybyjy1umtajia2tqfakrl5-6sdmelyeve2m2gk2ye32svlb8 93.186.253.253:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
> andiamosu un altro nodo e lanciamo il comando
```
docker@Will1:~$ docker swarm join --token SWMTKN-1-69832mntsgml7zpocn50wywj6cofybyjy1umtajia2tqfakrl5-6sdmelyeve2m2gk2ye32svlb8 93.186.253.253:2377
This node joined a swarm as a worker.
docker@Will1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:50:56:aa:06:81 brd ff:ff:ff:ff:ff:ff
    inet 80.211.239.62/24 brd 80.211.239.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:feaa:681/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 00:50:56:aa:f3:ee brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 00:50:56:aa:31:3b brd ff:ff:ff:ff:ff:ff
5: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:36:aa:84:62 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:36ff:feaa:8462/64 scope link
       valid_lft forever preferred_lft forever
22: docker_gwbridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:f4:f1:30:c6 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker_gwbridge
       valid_lft forever preferred_lft forever
    inet6 fe80::42:f4ff:fef1:30c6/64 scope link
       valid_lft forever preferred_lft forever
24: veth8a1989d@if23: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker_gwbridge state UP group default
    link/ether b6:5d:a9:33:8a:8b brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::b45d:a9ff:fe33:8a8b/64 scope link
       valid_lft forever preferred_lft forever
docker@Will1:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
2a499cbf6ef3        bridge              bridge              local
28d642a48e9a        docker_gwbridge     bridge              local
dcda668f88b5        host                host                local
y8nn5vw92ugx        ingress             overlay             swarm
276fa2c75f2a        none                null                local
```
>Da notare che sui rispettivi nodi si sono attivate altre network di tipo bridge e overlay

### Servizio semplice e scaling
```
$ docker service create --replicas 1 --name web nginx
5fngakweds4wunkzk47joh0jz
overall progress: 1 out of 1 tasks
1/1: running   
verify: Service converged
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                             PORTS
5fngakweds4w        web                 replicated          1/1                 nginx:latest                      
```

```
$ mkdir stackdemo
$ cd stackdemo
$ vi app.py
$ vi requirements.txt
$ vi Dockerfile
$ vi docker-compose.yml
$ ls
Dockerfile  app.py  docker-compose.yml  requirements.txt
$ docker-compose up -d
WARNING: The Docker Engine you're using is running in swarm mode.

Compose does not use swarm mode to deploy services to multiple nodes in a swarm. All containers will be scheduled on the current node.

To deploy your application across the swarm, use `docker stack deploy`.

Creating network "stackdemo_default" with the default driver
Building web
Step 1/5 : FROM python:3.4-alpine
3.4-alpine: Pulling from library/python
8e402f1a9c57: Pull complete
cda9ba2397ef: Pull complete
aafecf9bbbfd: Pull complete
bc2e7e266629: Pull complete
e1977129b756: Pull complete
Digest: sha256:c210b660e2ea553a7afa23b41a6ed112f85dbce25cbcb567c75dfe05342a4c4b
Status: Downloaded newer image for python:3.4-alpine
 ---> c06adcf62f6e
Step 2/5 : ADD . /code
 ---> 9d3b776cf32f
Step 3/5 : WORKDIR /code
 ---> Running in 0f024a381d40
Removing intermediate container 0f024a381d40
 ---> d3205a2e6fac
Step 4/5 : RUN pip install -r requirements.txt
 ---> Running in c3ecf8e6847f
DEPRECATION: Python 3.4 support has been deprecated. pip 19.1 will be the last one supporting it. Please upgrade your Python as Python 3.4 won't be maintained after March 2019 (cf PEP 429).
Collecting flask (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/d8/94/7350820ae209ccdba073f83220cea1c376f2621254d1e0e82609c9a65e58/Flask-1.0.4-py2.py3-none-any.whl (92kB)
Collecting redis (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/32/ae/28613a62eea0d53d3db3147f8715f90da07667e99baeedf1010eb400f8c0/redis-3.3.11-py2.py3-none-any.whl (66kB)
Collecting Werkzeug>=0.14 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/c2/e4/a859d2fe516f466642fa5c6054fd9646271f9da26b0cac0d2f37fc858c8f/Werkzeug-0.16.1-py2.py3-none-any.whl (327kB)
Collecting Jinja2>=2.10 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/65/e0/eb35e762802015cab1ccee04e8a277b03f1d8e53da3ec3106882ec42558b/Jinja2-2.10.3-py2.py3-none-any.whl (125kB)
Collecting itsdangerous>=0.24 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting click>=5.1 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl (81kB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.10->flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/b9/2e/64db92e53b86efccfaea71321f597fa2e1b2bd3853d8ce658568f7a13094/MarkupSafe-1.1.1.tar.gz
Building wheels for collected packages: MarkupSafe
  Building wheel for MarkupSafe (setup.py): started
  Building wheel for MarkupSafe (setup.py): finished with status 'done'
  Stored in directory: /root/.cache/pip/wheels/f2/aa/04/0edf07a1b8a5f5f1aed7580fffb69ce8972edc16a505916a77
Successfully built MarkupSafe
Installing collected packages: Werkzeug, MarkupSafe, Jinja2, itsdangerous, click, flask, redis
Successfully installed Jinja2-2.10.3 MarkupSafe-1.1.1 Werkzeug-0.16.1 click-7.0 flask-1.0.4 itsdangerous-1.1.0 redis-3.3.11
You are using pip version 19.0.3, however version 19.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Removing intermediate container c3ecf8e6847f
 ---> f2090fd0525c
Step 5/5 : CMD ["python", "app.py"]
 ---> Running in 55b3597809a6
Removing intermediate container 55b3597809a6
 ---> 2eb130ab162a
Successfully built 2eb130ab162a
Successfully tagged 127.0.0.1:5000/stackdemo:latest
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Pulling redis (redis:alpine)...
alpine: Pulling from library/redis
df20fa9351a1: Already exists
9b8c029ceab5: Pull complete
e983a1eb737a: Pull complete
2e09334c25b1: Pull complete
cd5f0c28bbb7: Pull complete
ff366bbd9e6e: Pull complete
Digest: sha256:50ce670996835d83e070a6b26ef168de774333cf6317cd1bad45f84da1421e24
Status: Downloaded newer image for redis:alpine
Creating stackdemo_redis_1 ...
Creating stackdemo_web_1 ...
Creating stackdemo_web_1
Creating stackdemo_web_1 ... done
```
### Servizio composto o stack
```
$ docker stack deploy --compose-file docker-compose.yml stackdemo
Ignoring unsupported options: build

Creating network stackdemo_default
Creating service stackdemo_redis
Creating service stackdemo_web

$ docker stack services stackdemo
ID                  NAME                MODE                REPLICAS            IMAGE                             PORTS
hn31w6xsn8j7        stackdemo_redis     replicated          1/1                 redis:alpine                      
lpvbhymyq2j0        stackdemo_web       replicated          1/1                 127.0.0.1:5000/stackdemo:latest   *:8000->8000/tcp

$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
bf1e8657ead6        bridge              bridge              local
8e1e247497ef        docker_default      bridge              local
54a6c86c3e19        docker_gwbridge     bridge              local
dbfef2fb149c        host                host                local
y8nn5vw92ugx        ingress             overlay             swarm
79d3632aa486        newnet1             bridge              local
99913a05d52f        none                null                local
21038e0e91db        stackdemo_default   bridge              local
quc50l7lcnca        stackdemo_default   overlay             swarm

$ docker stack ls
NAME                SERVICES            ORCHESTRATOR
stackdemo           2                   Swarm


```

### Monitoring con Prometheus
https://docs.docker.com/config/daemon/prometheus/
