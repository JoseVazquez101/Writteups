# The Great Escape

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/59726e92-d5ea-4a31-9070-8c56abdd89d0)

***
 Source: https://tryhackme.com/room/thegreatescape
- OS: Linux
- Dificultad: Medium
- IP: No estática
- Temas: `Docker Breakout`, `SSRF`.
***
- Realizamos un escaneo de puertos para saber a que nos enfrentamos.

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Great-Escape]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV escape.thm
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-09 10:43 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 10:43
Scanning 10.10.231.116 [65535 ports]
Discovered open port 80/tcp on 10.10.231.116
Discovered open port 22/tcp on 10.10.231.116
Completed SYN Stealth Scan at 10:43, 24.01s elapsed (65535 total ports)
Initiating Service scan at 10:43
Scanning 2 services on 10.10.231.116
Completed Service scan at 10:46, 163.78s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.231.116.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 10:46
Completed NSE at 10:46, 1.80s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 10:46
Completed NSE at 10:46, 1.33s elapsed
Nmap scan report for 10.10.231.116
Host is up, received user-set (0.23s latency).
Scanned at 2024-02-09 10:43:33 EST for 191s
Not shown: 34256 closed tcp ports (reset), 31277 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh?    syn-ack ttl 60
80/tcp open  http    syn-ack ttl 60 nginx 1.19.6
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port22-TCP:V=7.94%I=7%D=2/9%Time=65C64848%P=x86_64-pc-linux-gnu%r(Gener
SF:icLines,F,"\.kp=W>4a;=\x20c3\r\n");

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 191.28 seconds
           Raw packets sent: 118622 (5.219MB) | Rcvd: 39471 (1.579MB)
~~~

- Vemos solo dos servicios, en este caso creo que será más optimo enumerar el servidor http que está hosteado en el puerto 80.
