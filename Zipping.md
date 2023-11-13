# Zipping

Hola, hoy tengo una guia de una maquina de HackTheBox, en las cuales estaremos viendo explotación a partir de subida de archivos y secuestro de librerias para escalada de privilegios. Esta maquina ya la habia resuelto con el metodo del null byte en un archivo zip, pero esto fue parcheado hace unos meses así que mostraré un metodo alterno, por lo que me saltaré la parte de los escaneos

- Plataforma: HackTheBox
- Dificultad: Medium
- OS: Linux
- IP: 10.10.11.229

- Tenemos dos puertos abiertos, el 80 y 22, la pagina web tiene un apartado para subir archivos zip, nosotros crearemos un link simbolico a /etc/hosts de esta forma:
~~~ bash 
ln -s ../../../../../../etc/hosts document.pdf
zip --symlinks zip.zip document.pdf
~~~
- Ahora al subirlo vemos que hay conectividad y una respuesta por parte del servidor, por lo que podemos concluir que es vulnerable a LFI, devolviéndome una cadena de texto en base64:
![[Pasted image 20231103165506.png]]
![[Pasted image 20231103165521.png]]
- También podemos comprobarlo bajando el archivo directamente:
  ![[Pasted image 20231103165714.png]]![[Pasted image 20231103165726.png]]
  - A este punto me quedé sin ideas, hasta que se me ocurrió bajar algunos de los archivos que nos muestra como trabaja la aplicación por detrás, en cart.php encontré algo interesante:

![[Pasted image 20231103171141.png]]
- Según HackTricks podemos ofuscar un payload a través de un bit nulo %0A, si tenemos exito deberiamos ver un codigo de redirección 302:
- Al parecer el servicio de mysql se ejecuta como root, por lo que podremos intentar utilizarlo para que ejecute nuestro payload:
  ![[Pasted image 20231103171952.png]]
  - Y podemos ver que nos responde correctamente, por lo que intentaremos inyectar el payload a través de este metodo:
  - Primero, creamos nuestro payload, encondeando 2 veces en base64:
![[Pasted image 20231103174330.png]]
- Después, creamos un archivo en php con la siguiente cadena:
  ![[Pasted image 20231103174410.png]]
  - Y encodeamos este payload en base64 para así poder subirlo de la siguiente forma:
![[Pasted image 20231103174444.png]]
![[Pasted image 20231103175237.png]]
- Perfecto, el archivo está subido y lo único que debemos hacer es tener un listener en nuestro puerto 4444 y solicitar en payload con GET:
![[Pasted image 20231103175302.png]]
![[Pasted image 20231103175317.png]]


***
<h3>PrivEsc</h3>

- Podemos encontrar un vector de ataque bastante rápido, al observar que podemos ejecutar un binario como sudo:
  ![[Pasted image 20231103175500.png]]
  - Ejecutamos el binario y observamos que solicita una contraseña:

![[Pasted image 20231103175554.png]]
- Llegó la hora de examinar el binario, primero que nada utilizaremos strings, podemos ver que es un binario compilado en C, además de lo que parece ser una contraseña:
  ![[Pasted image 20231103175721.png]]
  Al ingresar la contraseña, el binario simplemente deja de funcionar, al parecer se congela:
  ![[Pasted image 20231103183841.png]]
  - Observamos que intenta llamar a una librería que no existe, por lo cual podemos reemplazar esta con una bash ejecutada desde un archivo C, que crearemos desde /tmp y compilaremos con gcc en la dirección donde debería estar
~~~ c 
#include <stdio.h>
#include <stdlib.h>
void false_lib() { system("/bin/bash"); } 
__attribute__((constructor)) void setup() { false_lib(); }
~~~
 - Este archivo ejecuta una bash a través de la función `setup` que se ejecutará automáticamente antes de la función `main` gracias al atributo `__attribute__((constructor))`, aprovechándonos de que se ejecutará como root, ganaremos privilegios instantáneamente.

   ~~~ bash 
   gcc -shared -o /home/rektsu/.config/libcounter.so -fPIC lib.c
   ~~~
   ![[Pasted image 20231103193618.png]]
 - Y finalmente somos root :D
