# Crack the Hash

> Lorenzo De Simone | 21.05.2023

- [Crack the Hash](#crack-the-hash)
  - [Exploitation](#exploitation)
    - [Stage 1](#stage-1)
      - [Flags](#flags)
    - [Stage 2](#stage-2)
      - [Flags](#flags-1)


## Exploitation

We can use the command **hash-identifier** or the Website https://hashes.com/en/tools/hash_identifier to identify the hashes.

### Stage 1

1. 48bb6e862e54f2a795ffc4e541caed4d

Looks like a md5 Hash, could be crackable with hashcat.

```console
hashcat -m 0 md5_hash /usr/share/wordlists/rockyou.txt

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

48bb6e862e54f2a795ffc4e541caed4d:easy 
```
- **-m 0** Tells hashcat the mode to use, 0 is MD5
- **md5_hash** File containing the md5 hash
- Append the worldlist you want to use, in this case rockyou.txt


1. CBFDAC6008F9CAB4083784CBD1874F76618D2A97

Looks like a SHA1 Hash, could be also crackable with hashcat.

```console
hashcat -m 100 sha1_hash /usr/share/wordlists/rockyou.txt

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

cbfdac6008f9cab4083784cbd1874f76618d2a97:password123      

```
- **-m 100** Tells hashcat the mode to use, 100 is SHA1
- **sha1_hash** File containing SHA1 hash 
- Append the wordlist you want to use, in this case rockyou.txt


3. 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

Looks like a SHA256 Hash, let's try hashcat aswell. 

```console
hashcat -m 1400 sha256_hash /usr/share/wordlists/rockyou.txt

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

1c8bfe8f801d79745c4631d09fff36c82aa37fc4cce4fc946683d7b336b63032:letmein
```
- **-m 1400** Tells hashcat the mode to use, 1400 is SHA256
- **sha_256_hash** File containing the SHA256 hash
- Append wordlist, in this case rockyou.txt


4. $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

Possible algorithms: bcrypt $2*$, Blowfish (Unix)

```console
hashcat -m 3200 blow_fish_hash /usr/share/wordlists/rockyou.txt

```
This will take a long time, we can see that the flag is 4 characters long, maybe we can use a mask on hashcat

```console
hashcat -m 3200 -a 3  blow_fish_hash b?l?l?l
```
- **-m 3200** Specifies the hashcat mode, 3200 is bcrypt
- **-a 3** Specifies the attack mode, 3 is brutforce
- **blow_fish_hahs** File containing the bcrypt hash
- **b?l?l?l** Specifies a mask, String is 4 lower case characters long and starts with **b**

A faster way could also be to filter rockyou.txt to words that are only 4 characters long. 


5. 279412f945939ba78ce0758d3fd83daa

Looks like a MD4 hash, crackstation may also be able to crack this (rainbow tables). 

Crackstation got a result => Eternity22

#### Flags 

1. 48bb6e862e54f2a795ffc4e541caed4d
> easy

2. CBFDAC6008F9CAB4083784CBD1874F76618D2A97
> password123

3. 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
> letmein

4. $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
> bleh 

5. 279412f945939ba78ce0758d3fd83daa
> Eternity22

### Stage 2

1. F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

Looks like SHA256, Maybe hashcat again? 

```console
hashcat -m 1400 sha256_hash /usr/share/wordlists/rockyou.txt

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

f09edcb1fcefc6dfb23dc3505a882655ff77375ed8aa2d1c13f640fccc2d0c85:paule
```


2. 1DFECA0C002AE40B8619ECF94819CC1B

Looks like NTLM algorithm, does hashcat have something for that? 
> Mode 1000 seems to be ntlm

```console
hashcat -m 1000 ntlm_hash /usr/share/wordlists/rockyou.txt

1dfeca0c002ae40b8619ecf94819cc1b:n63umy8lkf4i
```


3. $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.

Looks like sha512crypt, salt is present in hash.
> Mode 1800

This could take some time. We know the password is 6 characters long. So let's only use the entries in rockyou.txt that are 6 chars long.

```console
awk 'length($0) ==6' rockyou.txt | tee rockyou_6.txt

hashcat -m 1800 sha512_hash rockyou_6.txt

$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.:waka99
```


4. e5d8870e5bdd26602cab8dbe07a942c8669e56d6

SHA1, but we also have salt in the challenge => **tryhackme**
> Mode 110 or 120, probably

After trying different SHA1 Modes, Mode 160 was able to crack the hash

```console
hashcat -m 160 sha1_hash /usr/share/wordlists/rockyou.txt

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme:481616481616
```
- The File **sha1_hash** contains the hash and the salt in the following format => **HASH:SALT**

```console
cat sha1_hash    
e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme
```

#### Flags

1. F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
>  paule

2. 1DFECA0C002AE40B8619ECF94819CC1B
> n63umy8lkf4i

3. $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
> waka99

4. e5d8870e5bdd26602cab8dbe07a942c8669e56d6
> 481616481616