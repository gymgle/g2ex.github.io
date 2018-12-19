title: 部署Candy - 搭建自己的Web聊天服务器
date: 2013-7-21
tags:
- Candy
- Jabber
- Openfire
- Apache
permalink: Deploy-Candy-Chat-Jabber-Server
---

![candy][candy01]

Candy是基于JavaScript的多用户通信Web客户端，可用于搭建互联网和局域网的聊天系统，它最大的特点是多用户、实时、简单，以下是官网对Candy的介绍，你还可以去[Candy](http://candy-chat.github.io/candy/)的主页体验一下它的Demo。

> 1. Focused on real-time multi-user chatting
> 2. Easy to configure, easy to run, easy to use
> 3. Highly customizable
> 4. 100% well-documented JavaScript source code
> 5. Built for Jabber (XMPP), using famous technologies
> 6. Used and approved in a productive environment with up to 400 concurrent users
> 7. Works with all major web browsers including IE7

Candy的部署需要用到两个服务：HTTP服务和Jabber（XMPP）服务。提供HTTP服务常用的软件有apache httpd、nginx、lighttpd、IIS、Node.js等。XMPP（Extensible Messaging and Presence Protocol，前称Jabber）是一种以XML为基础的开放式实时通信协议，能提供Jabber服务的软件有Openfire、ejabberd等。

说到部署自己的聊天服务，最简单的方法是在服务器上安装Openfire或ejabberd，就可以在其他电脑上使用Spark或者Pidgin进行聊天。Candy的好处是不需要Spark或Pidgin等客户端，在服务器端部署好HTTP服务，那么在其他计算机上只需要打开网页就可以聊天了，这也就是为什么Candy的部署需要Apache或者其他HTTP服务的支持了。

本文介绍如何在Windows上部署Candy，在开始之前，需要下载：XAMPP、Openfire和Candy。其中XAMPP包含了Apache和MySQL等。

目前，XAMPP最新版本是1.8.2；Openfire版本是3.8.2；Candy版本是1.0.9。

## 一、安装xampp

1. 在选择要安装组件步骤时，Apache和PHP是默认选中的，如果想在Candy中使用MySQL数据库而不是Candy的内嵌数据库，需要勾选上`MySQ`L和`phpMyAdmin`。

2. 在XAMPP控制面板，点击Apache的Config => httpd.conf，依次去掉下面三行前面的`#`来启用`Apache的mod_rewrite`、`mod_proxy`和`mod_proxy_http`模块。
    ```ini
    LoadModule rewrite_module modules/mod_rewrite.so
    LoadModule proxy_module modules/mod_proxy.so
    LoadModule proxy_http_module modules/mod_proxy_http.so
    ```
实际上，XAMPP已经启用了`mod_rewrite`和`mod_proxy`，我们只需要去掉`mod_proxy_http`之前的`#`就可以了。

## 二、配置MySQL

1. 如果没有安装MySQL或者不想使用MySQL，请跳过这一步。

2. 在XAMPP控制面板启动Apache和MySQL的服务，点击Apache的`Admin`，选择语言后点击左侧的`安全`，如下图所示，可以看到目前服务器的安全状态。

    ![XAMPP安全][xampp01]

    点击图中红框标记的链接，为MySQL的root账号添加密码（同时也可以为XAMPP添加具有访问权限的账号和密码）。

3. 在XAMPP控制面板点击MySQL的`Admin`，使用root账号和刚刚设置的root密码登陆，创建名为`candy`的数据库，当然，数据库的名字是自定义的，也可以用自己喜欢易记的名字。这个名字在Openfire初始化的时候会用到。

## 三、安装并初始化Openfire

1. 点击`Launch Admin`，选择语言（这里选择了中文），下一步是服务器设置，如下图所示，域默认的是计算机的主机名，可以保持默认，也可以修改成其他，这里改为`localhost`，可以把鼠标移到相应的`问号`图标上查看设置的说明。

	![Openfire服务器设置][openfire01]

2. 点击`继续`进入数据库设置，如下图所示，这里选择`标准数据库连接`。如果没有安装MySQL或其他数据库，那就只能选择`嵌入的数据库`了，设置更方便。

	![Openfire数据库设置][openfire02]

3. 点击`继续`，在`数据驱动选项`中选择`MySQL`，把`数据库URL`中的`[host-name]`改为`localhost`，把`[database-name]`改为`candy`。如下图所示：

	![Openfire的MySQL数据库设置][openfire03]

4. 点击`继续`，不用更改`特性设置`，点击`继续`，为管理员账户设置电子邮件和密码，点击`继续`安装完成。在Openfire管理框中先`Stop`，再`Start`重启Openfire。点击`Launch Admin`接下来就要使用`admin`账号和刚刚设置的密码登陆到管理控制台了。

## 四、设置Openfire

使用`admin`登陆Openfire管理控制台之后，需要做如下设置：

1. 服务器 => 服务器设置 => HTTP绑定，按下图设置，如果你更改了端口号，那么在后续的Candy设置中也要做相应的更改。

	![Openfire的HTTP绑定设置][openfire04]

2. 分组聊天 => 房间管理员 => 创建新房间，记住创建的房间标识，这里取名为`Default`，如下图所示：

	![Openfire创建新房间][openfire05]

3. 插件 => 有效的插件，安装Client Control插件。

4. 服务器 => 客户端管理Client Management => 分组聊天书签 => 增加分组对话书签，分组对话地址是`Default@conference.localhost`，其中`Default`是上面新建房间的名字，`localhost`是在`步骤0x02`中设置的域名字。

	勾选`所有用户`和`自动加入`，这样登陆聊天服务器的所有人都会自动加入到`Default`这个房间。

## 五、配置Candy

1. 解压candy，为方便起见，把解压得到的文件夹重命名为`candy`，并放到Apache web目录xampp/htdocs下，把htdocs/candy/example/目录下的htaccess和index.html复制到htdocs/candy/目录下。

2. 修改htacess，把最后一行中的`5280`改为`7070`。这是因为在上面我们设置Openfire的HTTP绑定端口为`7070`，如果你自己改成了其他端口号，这里就把`5280`改为你定义的端口号。
3. 把htaccess重命名为.htaccess。注意：在Windows中直接改名字不允许以`.`开头，可以在命令行下使用rename命令完成重命名，如下图所示：

	![重命名htaccess为.htaccess][cmd]

4. 修改index.html，把5处`..`都改为`.`（可以使用替换工具），然后把
    ```JavaScript
    $(document).ready(function() {
    	Candy.init('http-bind/', {
        core: { debug: false },
        view: { resources: './res/' }
    });
     
    Candy.Core.connect();
    });
    ```
修改为：
    ```JavaScript
    $(document).ready(function() {
    	Candy.init('http-bind/', {
    	core: { debug: false, autojoin: ['Default@conference.localhost'] },
    	view: { resources: './res/', language: 'cn' }
    });
    
    Candy.Core.connect('localhost');
    });
    ```
	上面代码中`Candy.Core.connect()`是Candy的登陆方法，有多种参数，可以参考Candy的`Login Methods`的介绍，修改后的代码实现了Candy Demo一样的效果，即不需要使用密码就可以登陆聊天。

## 六、修改Apache配置 

最后，修改Apache的`httpd.conf`配置文件，查找`htdocs`，在其后添加`/candy`，修改后的两行最终效果为：
```xml
DocumentRoot "D:/xampp/htdocs/candy"
<directory "D:/xampp/htdocs/candy">
```

OK！一切部署完毕！重启Apache，重启Openfire，在局域网中任何一台电脑的浏览器中输入服务器计算机的IP地址，就可以看到登陆界面了。

![Candy登陆界面][candy02]

另外需要说明的是，如果想让Candy支持SSL加密连接，需要修改Apache的「httpd-ssl.conf」配置文件：

把`DocumentRoot "D:/xampp/htdocs"`修改为`DocumentRoot "D:/xampp/htdocs/candy"`，
把`ServerName www.example.com:443`修改为`ServerName localhost:443`或`ServerName 『你的服务器IP地址』:443`。

## 参考内容

1. [http://candy-chat.github.io/candy/](http://candy-chat.github.io/candy/)
2. [https://github.com/candy-chat/candy/wiki/Installing-a-Jabber-server](https://github.com/candy-chat/candy/wiki/Installing-a-Jabber-server)
3. [https://zh.wikipedia.org/zh-cn/XMPP](https://zh.wikipedia.org/zh-cn/XMPP)
4. [http://www.yuexuan.org/?p=1570](http://www.yuexuan.org/?p=1570)


  [candy01]:https://i.imgur.com/4EvrCIf.png
  [xampp01]:https://i.imgur.com/qGRxqdB.png
  [openfire01]:https://i.imgur.com/bARhKBe.png
  [openfire02]:https://i.imgur.com/GIbcoZ9.png
  [openfire03]:https://i.imgur.com/8WfdZTt.png
  [openfire04]:https://i.imgur.com/JfZVREX.png
  [openfire05]:https://i.imgur.com/jlg1fXV.png
  [cmd]:https://i.imgur.com/83ENnjg.png
  [candy02]:http://i.imgur.com/bkrc69o.png
