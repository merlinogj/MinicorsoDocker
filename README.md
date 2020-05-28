# Intro
## Minicorso Docker

https://docs.docker.com/get-started/

### installa docker nel sistema linux ubuntu 18.04
sudo apt update && sudo apt upgrade -y && sudo apt install docker docker.io

### Il mio utente è nel gruppo docker?
``$ id``      ###serve per capire di quali gruppi facciamo parte

``uid=1000(massimo) gid=1000(massimo) gruppi=1000(massimo),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),112(lpadmin),124(sambashare),125(vboxusers),148(ubridge),149(libvirt),1001(bumblebee),1003(docker)
``
### controllo se esiste il gruppo docker
``$ sudo cat /etc/group``
### se esiste aggiungo l'utente al gruppo
``$ sudo usermod -aG docker $USER``
### se non esiste, creo il gruppo ed aggiungo l'utente
``$ sudo groupadd docker``

``$ sudo usermod -aG docker $USER``

### verifichiamo che docker è installato e che lo possiamo lanciare
``$ docker info``

### creiamo il nostro account su https://hub.docker.com e lo usiamo per fare il docker login
``$ docker login``

##Esercizio 01
### creiamo il nostro primo container, creiamo un DB
``$ docker run --name=mysql1 -d mysql/mysql-server``
``$ docker container ls``
### vediamo i "logs" di installazione in cui compare la password creata automaticamente
``$ docker container logs mysql1 | grep GENERATED``
[Entrypoint] GENERATED ROOT PASSWORD: GawLYjBYBUx3vjANyfuvez-Ixop

docker exec -it mysql1 bash
###### resetto la password per l'utente root di mysql facendo il login con la password one-time generata del container
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
###########

 ### per vedere i container che non sono in running
 docker container ls -a

### proviamo ad installare diversi database anche di vesioni precedenti
 docker container run --name=mysql2 -d mysql/mysql-server
 docker container run --name=mysql5.7 -d mysql/mysql-server:5.7
 docker container ls
### mysql2 ha la stessa passwor generata di mysql1?
 docker conatainer logs mysql2  | grep GENERATED
 [Entrypoint] GENERATED ROOT PASSWORD: !AwfEBC0vYHIGExUc2ehKUbumef

### voglio sapere l'ip del container
 docker container inspect mysql2 | grep IPAddress
 > "IPAddress": "172.17.0.3",
 ping -c1 172.17.0.3
 ip a
####
 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:f7:2a:92 brd ff:ff:ff:ff:ff:ff
    inet 172.16.16.95/24 brd 172.16.16.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef7:2a92/64 scope link
       valid_lft forever preferred_lft forever
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:95:2c:54:86 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:95ff:fe2c:5486/64 scope link
       valid_lft forever preferred_lft forever
####

 ###voglio vedere i processi che girano dentro il container
 docker container top mysql1
 UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
27                  20372               20324               1                   09:57               ?                   00:00:13            mysqld --init-file=/var/lib/mysql-files/2AZpaJenaI
 docker container top mysql2
 UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
27                  21288               21263               1                   10:09               ?                   00:00:07            mysqld --init-file=/var/lib/mysql-files/Qmit2GAWht

### proviamo a lanciare container con contenuto differente, tipo un frontend
 docker container run --name=web1 -d nginx
 docker container run --name=web2 -d nginx

 ### i container possono essere stoppati o cancellati, se faccio stop posso fare lo start, se cancello con rm il container è perduto
 docker container stop web2
 docker container start web2
 docker  container inspect web2 | grep IPAddress
 "IPAddress": "172.17.0.4",
### sapendo quale è l'ip possiamo vedere se il servizio web serve funziona sulla porta 80 tramite il comando curl
 curl 172.17.0.4
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

### vediamo in questo tipo di container cosa gira
 docker container top web2
 UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                23605               23576               0                   10:26               ?                   00:00:00            nginx: master process nginx -g daemon off;
systemd+            23635               23605               0                   10:26               ?                   00:00:00            nginx: worker process
### con "docker container ls" vediamo anche le porte esposte dal container che differiscono per tipologia
docker container ls
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS                    PORTS                 NAMES
b512ffb058b7        nginx                "nginx -g 'daemon of…"   7 minutes ago       Up 7 minutes              80/tcp                web1
214689ef3c60        nginx                "nginx -g 'daemon of…"   8 minutes ago       Up 8 minutes              80/tcp                web2
70870e6db10f        mysql/mysql-server   "/entrypoint.sh mysq…"   25 minutes ago      Up 25 minutes (healthy)   3306/tcp, 33060/tcp   mysql2
9af523f1453d        mysql/mysql-server   "/entrypoint.sh mysq…"   38 minutes ago      Up 38 minutes (healthy)   3306/tcp, 33060/tcp   mysql1

 ###VOLUMES
 ### andiamo a vedere se i container MySQL hanno creato i propri volumi
 docker volume ls
DRIVER              VOLUME NAME
local               bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006
local               d72bde5c8d9d483956cdddb4026b912a7e57d5f832b15c5b00abca874de759ca

###vogliamo sapere dove docker mette i files dei container
 sudo find / 2> /dev/null | grep bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006
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
 ### per andare a vedere i files dobbiamo essere root ed il comando per diventare root è il seguente
 sudo -i

###per vedere se un volume fa parte di un container usiamo l'inspect come segue
docker container inspect mysql1 | grep bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006
                 "Name": "bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006",
                "Source": "/var/lib/docker/volumes/bc0fb476d54e21490b3b6bbcda6eb348768f708314d4ae3634b67a0f8f158006/_data",

 ### dopo aver creato gli oggetti in docker questi sono i comandi per vederne alcuni di interesse
 docker container ls -a
 docker volume ls
 docker image ls
 docker network ls
 ip a

###
 docker container inspect web2
 ping -c1 172.17.0.3
 sudo arp -a

### Skill up - passiamo una variabile d'ambiente (environment) al container per settare la password di root di MariaDB
 docker container run -d -e "MYSQL_ROOT_PASSWORD=password" --name db1 mariadb
 docker container ls
### voglio usare il servizio del container con un client mysql
 sudo apt install mysql-client
 docker container inspect db1 | grep IPAddress
             "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
 mysql -u root -p -h 172.17.0.2
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

mysql>

###  facciamo un po' di pulizia con il prune che libera ciò che non è utilizzato
 docker container prune
 docker volume prune
 docker image prune
 docker container ls -a

### ora vogliamo creare il FE che si aggancia al BE con l'IP, user e password di MariaDB
docker run --name frontend -e WORDPRESS_DB_HOST=172.17.0.2:3306 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password -d wordpress
### controllo che il frontend interagisca con il backend creando il DB wordpress
 mysql -u root -p -h 172.17.0.2
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

### vediamo la orte esposta dal FE
docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
870a1740ac88        wordpress           "docker-entrypoint.s…"   6 minutes ago       Up 6 minutes        80/tcp              frontend
ace04ec47218        mariadb             "docker-entrypoint.s…"   14 minutes ago      Up 14 minutes       3306/tcp            db1
b512ffb058b7        nginx               "nginx -g 'daemon of…"   57 minutes ago      Up 57 minutes       80/tcp              web1

### controlliamo cosa gira dentro il container frontend
docker container top frontend
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                29277               29254               0                   11:18               ?                   00:00:01            apache2 -DFOREGROUND
www-data            29498               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
www-data            29499               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
www-data            29500               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
www-data            29501               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND
www-data            29502               29277               0                   11:18               ?                   00:00:00            apache2 -DFOREGROUND

###INOLTRO delle PORT, si aggiunge "-p 80:80" nella creazione del container
### prima lo rimuoviamo e poi lo ricreiamo con la dichiarazione della porta
docker container rm -f frontend
docker run --name frontend -e WORDPRESS_DB_HOST=172.17.0.2:3306 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password -d -p 80:80 wordpress
### altro esempio di come si pubblica una porta 80 sulla porta 81 dell'host
docker container run --name web1 -p 81:80 -d nginx

###VOLUMES
### voglio creare un volume "persistente" di nome "massimo" condiviso da più container
docker volume create massimo
docker volume ls
docker run --name frontend -e WORDPRESS_DB_HOST=172.17.0.2:3306 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password -v massimo:/var/www/html -d -p 80:80 wordpress
docker container run -v massimo:/html -it --rm busybox

###voglio creare un volume condiviso per editare la pagina di default di nginx
docker volume create nginx
docker container rm -f web1
docker container run -d --name web1 -v nginx:/usr/share/nginx/html nginx
docker container run -v nginx:/html -it --rm busybox
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



## docker-compose mi aiuta a risparmiare tempo e mi introduce nell'"Infrastructure as a code"
vi wp.yaml
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


### abbiamo bisogno di installare docker compose
 sudo apt install docker-compose
 docker-compose -f wp.yaml up -d
docker container ls
docker network ls
docker volume ls


vi rocket.yaml
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
              _id: 'rs0',
              members: [ { _id: 0, host: 'localhost:27017' } ]})\" &&
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

docker-compose -f rocket.yaml up -d
docker-compose -f rocket.yaml logs

###NETWORK
### voglio vedere le network di docker la docker0 è la bridged, la host si riferisce all'ip dell'host
docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
171d7f63e4a0        bridge              bridge              local
626776a1047d        host                host                local
517888c34e3a        none                null                local

###voglio crere una network differente per isolare alcuni container
docker network create newnet1
docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
171d7f63e4a0        bridge              bridge              local
626776a1047d        host                host                local
76759826efad        newnet1             bridge              local
517888c34e3a        none                null                local


docker run -it  --network newnet1 -v nginx:/html --rm busybox
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

###proviamo a lanciare un container nella rete host
 docker run -it  --network host -v nginx:/html --rm busybox

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

###nginx da file yaml e busybox

### file nginx.yaml

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

###comandi per busybox

docker container run -it -v annaleda_nginx:/html --rm busybox
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
###--provare vi index.html
/html # echo 'ciao vincenzo giovanni ettore andre arturo'>>index.html
/html # exit


### Build and run your image
git clone https://github.com/dockersamples/node-bulletin-board
cd node-bulletin-board/bulletin-board-app
ls
cat Dockerfile
docker build --tag bulletinboar:1.0 .
ls
docker image ls
docker container run --network host -d bulletinboard:1.0
docker container ls
docker login
###rendo pubblica (condivido) la docker creata
docker tag bulletinboard:1.0 [accaunt]/bulletinboard:1-0
###trasferisco su dockerhub
docker push bulletinboard:1.0 [accaunt]/bulletinboard:1-0
###installo la docker caricata du dockerhub
docker pull merlinogj/bulletinboard:1.0

### creo una immagine semplice partendo da nginx e cambiando la pagina index.html
mkdir ah
vi Dockerfile
FROM nginx:latest
WORKDIR /usr/share/nginx/html
COPY index.html .
RUN apt update


vi index.html
<!DOCTYPE html> <html> <head> <title>Welcome to Massimo!</title> <style> body { width: 35em; margin: 0 auto; font-family: Tahoma, Verdana, Arial, sans-serif; } </style> </head> <body> <h1>Welcome to Massimo!</h1> <p>If you see this page, the nginx web server is successfully installed and working. Further configuration is required.</p> <p>For online documentation and support please refer to <a href="http://nginx.org/">nginx.org</a>.<br/> Commercial support is available at <a href="http://nginx.com/">nginx.com</a>.</p> <p><em>Thank you for using nginx.</em></p> </body> </html>

docker build --tag maxnignx:1.1 .
docker image ls
docker image ls
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
maxnignx                  1.1                 e1105be7f239        3 hours ago         144MB

docker run -d -p 81:80 maxnignx:1.1
docker tag maxnignx:1.1 merlinogj/maxnginx:1.1
docker push merlinogj/maxnginx:1.1
