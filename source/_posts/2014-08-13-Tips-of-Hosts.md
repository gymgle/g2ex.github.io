title: 科学利用HOSTS文件
date: 2014-08-13 21:45
tags:
- Hosts
permalink: Tips-of-Hosts
---

> hosts文件是一个用于储存计算机网络中各节点信息的计算机文件。这个文件负责将主机名映射到相应的IP地址。hosts文件通常用于补充或取代网络中DNS的功能。和DNS不同的是，计算机的用户可以直接对hosts文件进行控制。——[维基百科][6]

我们在浏览器中输入域名访问网站时，DNS（Domain Name System）会把域名“解释”为网站的IP地址，这个解析的过程如下图所示：

![DNS解析过程][1]

在图中可以看到，DNS解析时首先会使用本地DNS缓存，这时会查询本地hosts文件。如果本地没有找到DNS记录，则要去DNS服务器去查询。

如果我们查询的DNS服务器返回给我们的IP地址是其恶意构造出来的，把域名指向了不正确的IP地址，这就是`DNS缓存污染`（又称为`DNS缓存投毒`）。

对于DNS缓存污染，我们可以在计算机中指定可信的DNS服务器，比如Google的`8.8.8.8`和OpenerDNS的`42.120.21.30`，不过这两个在大陆都已经中枪牺牲了，曾经他们可以解析出Google、Drobox、Twitter等IP。
对付DNS缓存污染的另外一个办法就是修改本地hosts，在hosts中加入被污染的域名和可用的IP地址，因为在上文得知，本地hosts先于DNS服务器解析。

hosts文件中每一行作为一条记录，以`#`开头的行是注释行。hosts记录的格式为：`IP地址 域名`，例如`127.0.0.1 localhost`的意思是把`localhost`的域名解析为`127.0.0.1`的IP地址。

XP/7/8/8.1的hosts位于系统盘的`Windows\System32\drivers\etc`目录下，Mac/Linux/Android的hosts位于`/etc/`目录下。Windows XP/7中修改方法见《Windows中修改hosts的方法》。

## 在哪里可以获取现成的hosts文件？

1. [huhamhire-hosts][2]——可以去广告、屏蔽恶意网站、科学上网
2. [smarthosts][3]——主要用于科学上网
3. ~~[projecth.us-hosts][4]~~
4. ...

>PS：
最近Dropbox又中枪了，在之后的文章中会介绍如何手动获取Dropbox可用hosts中的IP。

## 参考内容

1. [http://msdn.microsoft.com/zh-cn/library/cc775637(v=ws.10).aspx][5]
2. [http://zh.wikipedia.org/wiki/Hosts文件][6]
3. [http://zh.wikipedia.org/wiki/域名劫持][7]


  [1]: http://i.msdn.microsoft.com/dynimg/IC195944.gif "DNS解析过程"
  [2]: https://code.google.com/p/huhamhire-hosts/ "huhamhire-hosts"
  [3]: https://code.google.com/p/smarthosts/ "smarthosts"
  [4]: https://www.projecth.us/sources
  [5]: http://msdn.microsoft.com/zh-cn/library/cc775637%28v=ws.10%29.aspx
  [6]: http://zh.wikipedia.org/wiki/Hosts%E6%96%87%E4%BB%B6
  [7]: http://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E5%8A%AB%E6%8C%81