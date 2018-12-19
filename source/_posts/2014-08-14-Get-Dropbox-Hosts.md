title: 如何获取Dropbox的可用HOSTS IP
date: 2014-08-14 23:20
tags:
- Hosts
- Drobox
permalink: Get-Dropbox-Hosts
---

## 一、命令介绍

命令：`nslookup -vc www.dropbox.com 8.8.8.8`

解释：`nslookup`(name server lookup)域名查询，用于查询DNS的记录。这里指定使用Google的`8.8.8.8`DNS服务器查询`www.dropbox.com`的IP地址。参数`vc`代表`Always use a virtual circuit when sending requests to the server.`，意思是向DNS发送的查询请求始终使用虚电路模式，也就是使用TCP协议查询域名的IP地址。

**为什么要是使用`vc`参数？**

`nslookup`命令默认使用UDP协议查询，因为UDP协议的数据包是无序到达的，被GFW（Great Firewall of China，防火长城）构造的无效IP地址会先到达我们的电脑，我们会被[DNS缓存污染][1]导致无法访问该域名。使用TCP协议可以保证在指定的DNS可用情况下得到正确的IP地址。

下图中没有使用`vc`参数得到的两条IP地址是被GFW污染的虚假IP，无法`ping`通。

![nslookup命令图示][2]

在浏览器里输入使用`vc`参数得到的`108.160.166.13`，则会跳转到Dropbox的主页。

要想正常是用Dropbox网页和客户端，不仅要知道正确的`www.dropbox.com`IP地址，还需要知道以下域名对应的IP地址。

```ini
dropbox.com
forums.dropbox.com
dl.dropboxusercontent.com
d.dropbox.com
client-lb.dropbox.com
dl-client(1,2...999).dropbox.com
dl-debug(1,2...40).dropbox.com
client(1,2...99).dropbox.com
notify(1,2...10).drobox.com
```

手动获取这些域名的IP地址太费时，那就需要用到下面的脚本了。感谢 [Yannis Xu][3] 的基本脚本 和 [doc001][4] 对脚本的完善。

## 二、获取Dropbox hosts ip的Python脚本

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-  

def GetLists(subdomain,start,end):
    ret = []
    for i in xrange(int(start),int(end)+1):
        ret.append(subdomain+str(i)+'.dropbox.com')
    return ret

def GetDlClientLists():
    return GetLists('dl-client',1,999)

def GetDlDebugLists():
    return GetLists('dl-debug',1,40)

def GetClientLists():
    return GetLists('client',1,99)

def GetNotifyLists():
    return GetLists('notify',1,10)

hosts = []
hosts.extend([
        'dropbox.com',
        'www.dropbox.com',
        'forums.dropbox.com',
        'dl.dropboxusercontent.com',
        'd.dropbox.com',
        'client-lb.dropbox.com'
        ])
hosts.extend(GetDlClientLists())
hosts.extend(GetDlDebugLists())
hosts.extend(GetClientLists())
hosts.extend(GetNotifyLists())

import subprocess
for h in hosts:
    cmd = 'nslookup -vc ' + h + ' 8.8.8.8'
    p = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)

    valid = False
    for line in p.stdout.readlines():
        if line.startswith('Non-authoritative answer:'):
            valid = True
        elif valid and line.startswith('Address:'):
            ip = line.replace('Address: ','').replace('\n','')
            print ip + ' ' + h
```

把脚本保存为`GetDropboxIP.py`，在命令行窗口或终端中执行：

````python
python GetDropboxIP.py > hosts_dropbox.txt
```

最后把`hosts_dropbox.txt`中的记录放到系统hosts文件中。终于可以正常使用Dropbox了。

## 三、另一个问题

毕竟我们在大陆，Google的`8.8.8.8`DNS服务器在国外，GFW位于两者中间，使用`nslookup -vc HOSTNAME 8.8.8.8`命令经常会出现丢包现象，更不要说连续查询上千条记录。所以，我们可以**使用国外的计算机查询Dropbox的IP地址**。

拥有国外VPS的同学就非常方便了。没有国外VPS的同学呢？让你的国外小伙伴来跑脚本把结果给你？还是自己动手吧。

[Cloud 9][5]、[Koding][6]、[Codebox][7]、[Codenvy][8]、[Nitrous.IO][9] 等网站都是国外优秀的虚拟主机编程IDE，可以在这些免费主机中运行上面的脚本，这里以`Cloud 9`为例。

1. 首先需要注册一个免费账号，然后，在`DASHBOARD`里通过点击`CREATE NEW WORKSPACE`来创建新的工程，这里直接使用它默认创建的示例工程，选择`demo-project`，然后点击`START EDITING`。
![创建工程并编辑][10]

2. 在Python目录下新建`GetDropboxIP.py`文件，并把脚本内容粘贴到文件中。
![新建文件][11]

3. 在IDE下方的`bash`里执行:`python GetDropboxIP.py > hosts_dropbox.txt`
![执行脚本][12]

4. 等待几十秒，脚本执行完成后，在IDE左侧文件列表中右键点击生成的`hosts_dropbox.txt`，选择`Download`下载到本地。
![下载hosts][13]

5. Enjoy.

## 四、参考内容

1. [http://yannisxu.me/post/reconnect-dropbox][14]
2. [http://www.doc001.com/post/2014-06-22][15]


  [1]: http://zh.wikipedia.org/zh-cn/域名服务器缓存污染 "维基百科"
  [2]: https://i.imgur.com/wwyxB0s.png "nslookup命令图示"
  [3]: http://yannisxu.me/post/reconnect-dropbox
  [4]: http://www.doc001.com/post/2014-06-22
  [5]: https://c9.io
  [6]: https://koding.com/
  [7]: https://www.codebox.io/
  [8]: https://codenvy.com/
  [9]: https://www.nitrous.io/
  [10]: https://i.imgur.com/SCuQ94L.png "创建工程并编辑"
  [11]: https://i.imgur.com/DshvVpI.png "新建文件"
  [12]: https://i.imgur.com/gObHuje.png "执行脚本"
  [13]: https://i.imgur.com/6nNtCRO.png "下载hosts"
  [14]: http://yannisxu.me/post/reconnect-dropbox
  [15]: http://www.doc001.com/post/2014-06-22