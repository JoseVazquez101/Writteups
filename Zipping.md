# Zipping

Hola, hoy tengo una guia de una maquina de HackTheBox, en las cuales estaremos viendo explotación a partir de subida de archivos y secuestro de librerias para escalada de privilegios. Esta maquina ya la habia resuelto con el metodo del null byte en un archivo zip, pero esto fue parcheado hace unos meses así que mostraré un metodo alterno, por lo que me saltaré la parte de los escaneos

- Plataforma: [HackTheBox](https://app.hackthebox.com/home)
- Dificultad: Medium
- OS: Linux
- IP: 10.10.11.229

- Tenemos dos puertos abiertos, el 80 y 22, la pagina web tiene un apartado para subir archivos zip, nosotros crearemos un link simbolico a /etc/hosts de esta forma:
~~~ bash 
ln -s ../../../../../../etc/hosts document.pdf
zip --symlinks zip.zip document.pdf
~~~
- Ahora al subirlo vemos que hay conectividad y una respuesta por parte del servidor, por lo que podemos concluir que es vulnerable a LFI, devolviéndome una cadena de texto en base64:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/8fdb597e-5693-4c3b-bb56-ef9926e711af)
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/055645c8-d1fe-42f9-a836-5bcdf9d735c4)

- También podemos comprobarlo bajando el archivo directamente:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/5cb00c9b-e1cf-4ecc-8f45-942bfefdebce)

- Para este punto me quedé sin ideas, hasta que se me ocurrió bajar algunos de los archivos que nos muestra como trabaja la aplicación por detrás, en cart.php encontré algo interesante, el valor de `product_id` parece estar intentando bloquear solicitudes con ciertos caracteres, solo aceptando algunos:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/f9e35c5a-6cc5-4143-8f05-51f50614b27d)

- Según HackTricks podemos ofuscar un payload a través de un bit nulo `%0A`, si tenemos exito deberiamos ver un codigo de redirección 302:
- Al parecer el servicio de mysql se ejecuta como root, por lo que podremos intentar utilizarlo para que ejecute nuestro payload:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/8b23d68c-dc25-41f7-8fa6-272d1f8809f9)
  - Y podemos ver que nos responde correctamente, por lo que intentaremos inyectar el payload a través de este metodo:
  - Primero, creamos nuestro payload, encondeando 2 veces en base64:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/5c2e9f4d-2ffa-4886-9a36-03d2d0b093ab)

- Después, creamos un archivo en php con la siguiente cadena:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/81fa0f62-3e77-46dc-95c3-4a487bc344ca)

- Y encodeamos este payload en base64 para así poder subirlo de la siguiente forma:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/aa944109-da87-4825-9642-71b3afe8c763)
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/31001874-7fdf-4f8e-b107-d37ae16f9782)


- Perfecto, el archivo está subido y lo único que debemos hacer es tener un listener en nuestro puerto 4444 y solicitar en payload con GET:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/d233ffd4-c449-4d62-bf78-93555cd234a4)


***
<h3>PrivEsc</h3>

- Podemos encontrar un vector de ataque bastante rápido, al observar que podemos ejecutar un binario como sudo:
  ![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/ad91c15f-388d-44aa-9807-f3343ca553a8)

  - Ejecutamos el binario y observamos que solicita una contraseña:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/691383d6-b937-4f41-9d11-10eaf5a25529)

- Llegó la hora de examinar el binario, primero que nada utilizaremos `strings`, podemos ver que es un binario compilado en C, además de lo que parece ser una contraseña:
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/82c39cb2-2605-47dd-9b7f-518c4620feb2)

- Al ingresar la contraseña, el binario simplemente deja de funcionar, al parecer se congela. Intento analizarlo con `strace` durante su ejecución y podemos ver algo mas:
~~~bash
strace sudo /usr/bin/stock
~~~
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/b1da549f-88b7-4c60-aff9-efa2adfaa02a)
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
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/d551ea10-aabf-456a-be4f-622465e13f97)

 - Y finalmente somos root :D. Podemos dar por concluida la maquina.
