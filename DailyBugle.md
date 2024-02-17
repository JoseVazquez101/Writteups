# Daily Bugle

![image](https://github.com/JoseVazquez101/Writteups/assets/111292579/682e0484-03da-44d6-85ec-26971c4c2168)

***
- Source: https://tryhackme.com/room/dailybugle
- OS: Linux
- Dificultad: Hard
- IP: No estática
- Temas: ``CMS``, ``Joomla``, `Sqli`, `Linux PrivEsc`.
***

~~~bash
┌──(kali💀Dedsec)-[~/Maquinas/Linux/Dayly_Bugle]
└─$ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -sV 10.10.19.253
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-16 19:34 EST
NSE: Loaded 46 scripts for scanning.
Initiating SYN Stealth Scan at 19:34
Scanning 10.10.19.253 [65535 ports]
Discovered open port 3306/tcp on 10.10.19.253
Discovered open port 22/tcp on 10.10.19.253
Discovered open port 80/tcp on 10.10.19.253
Completed SYN Stealth Scan at 19:35, 24.95s elapsed (65535 total ports)
Initiating Service scan at 19:35
Scanning 3 services on 10.10.19.253
Completed Service scan at 19:35, 11.83s elapsed (3 services on 1 host)
NSE: Script scanning 10.10.19.253.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 19:35
Completed NSE at 19:35, 1.03s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 19:35
Completed NSE at 19:35, 1.18s elapsed
Nmap scan report for 10.10.19.253
Host is up, received user-set (0.51s latency).
Scanned at 2024-02-16 19:35:00 EST for 39s
Not shown: 39102 filtered tcp ports (no-response), 26430 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 61 OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    syn-ack ttl 61 Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
3306/tcp open  mysql   syn-ack ttl 61 MariaDB (unauthorized)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.37 seconds
           Raw packets sent: 121003 (5.324MB) | Rcvd: 27783 (1.111MB)

~~~
