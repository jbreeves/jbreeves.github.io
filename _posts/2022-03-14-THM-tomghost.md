---
layout: post
title: TryHackMe - tomghost
tags: THM CTF Tomcat
---

*Free Room*

> AcousticGirl 
> March 14th, 2022

--------

IP = 10.10.118.36

1. Enumeration
- Rustscan w/ nmap
	```
	22/tcp   open  ssh        syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
	| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQvC8xe2qKLoPG3vaJagEW2eW4juBu9nJvn53nRjyw7y/0GEWIxE1KqcPXZiL+RKfkKA7RJNTXN2W9kCG8i6JdVWs2x9wD28UtwYxcyo6M9dQ7i2mXlJpTHtSncOoufSA45eqWT4GY+iEaBekWhnxWM+TrFOMNS5bpmUXrjuBR2JtN9a9cqHQ2zGdSlN+jLYi2Z5C7IVqxYb9yw5RBV5+bX7J4dvHNIs3otGDeGJ8oXVhd+aELUN8/C2p5bVqpGk04KI2gGEyU611v3eOzoP6obem9vsk7Kkgsw7eRNt1+CBrwWldPr8hy6nhA6Oi5qmJgK1x+fCmsfLSH3sz1z4Ln
	|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
	| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOscw5angd6i9vsr7MfCAugRPvtx/aLjNzjAvoFEkwKeO53N01Dn17eJxrbIWEj33sp8nzx1Lillg/XM+Lk69CQ=
	|   256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
	|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGqgzoXzgz5QIhEWm3+Mysrwk89YW2cd2Nmad+PrE4jw
	53/tcp   open  tcpwrapped syn-ack
	8009/tcp open  ajp13      syn-ack Apache Jserv (Protocol v1.3)
	| ajp-methods: 
	|_  Supported methods: GET HEAD POST OPTIONS
	8080/tcp open  http       syn-ack Apache Tomcat 9.0.30
	|_http-title: Apache Tomcat/9.0.30
	| http-methods: 
	|_  Supported Methods: GET HEAD POST OPTIONS
	|_http-favicon: Apache Tomcat
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
	```
Services of Note:
SSH
DNS?
Apache Jserv
Apache Tomcat 9.0.30

2. Web Site
- Standard Tomcat installation page, no robots.txt or subdirectories.

3. Metasploit
- searched for jserv and it returned 1 module. Ran the module and obtained credentials: `skyfuck:8###################s`

4. SSH
- SSH'd in using credentials found in Step 3.
- Obtained User flag in /home/merlin/user.txt - `THM{G#################y}`
- Exfiltrated credential.pgp and tryhackme.asc via bash TCP connection to use

5. PGP Key (part 1)
- Tried to import tryhackme.asc but it required a password.
- Previously discovered credentials did not work.

6. John The Ripper
- Updated John
- Used gpg2john on tryhackme.asc for password
	- password is `a#######u`

7. PGP Key (part 2)
- Imported PGP key with password found in step 6
- decoded credential.pgp with key/password from step 5-6
	- obtained merlin's credentials
	`merlin:a########################################################j`

8. Priv Esc / SSH (part 2)
- Logged in as `merlin` user with password recovered in step 7
- Ran `sudo -l` to ascertain if merlin has sudo privs and he does!
	```
	merlin@ubuntu:~$ sudo -l
	Matching Defaults entries for merlin on ubuntu:
	    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

	User merlin may run the following commands on ubuntu:
	    (root : root) NOPASSWD: /usr/bin/zip
	```
- Merlin can sudo the zip program with no password
- Looked at Zip in GTFOBins and see Sudo listed
- Tried Zip Sudo exploit and got root

9. Root Flag
- obtained root flag in /root/root.txt
	`THM{Z#########E}` 
