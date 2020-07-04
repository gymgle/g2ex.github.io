title: "[译]如何在Ubuntu 14.04中为Flask应用部署Nginx和uWSGI服务"
date: 2015-10-30 20:00:00
tag:
- Nginx
- uWSGI
- Flask
- Python
- Ubuntu
permalink: How-to-Serve-Flask-Applications-with-uWSGI-and-Nginx-on-Ubuntu-14-04
---

英文链接：https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-14-04

中文由G2EX.ME作者翻译，转载请注明本文链接。

## 一、介绍

本指南中，我们在Ubuntu 14.04中使用微框架Flask搭建一个简单的Python应用。文中的大部分内容是介绍如何部署uWSGI服务来启动应用，以及如何部署Nginx作为前端到后端的反向代理。

## 二、预备条件

本指南开始之前，在服务器中你应该会有一个非root用户。为了能执行管理员的功能，这个用户需要`sudo`权限。了解如何设置用户和管理员权限，请阅读《[initial server setup guide][1]》。

想更多了解应用服务uWSGI和WSGI规范，你可以阅读[这个指南中的章节][2]。理解这些概念后能够让你更容易地阅读这个指南。

当你准备好后，开始阅读下文。

## 三、从Ubuntu仓库中安装组件

我们第一步是要从仓库中安装所需的组件。包括Python包管理器`pip`，它可以安装和管理Python组件。我们也会安装构建uWSGI所需的Python开发文件，以及安装Nginx。

更新你本地软件包的索引，然后输入以下命令安装软件包：

```bash
sudo apt-get update
sudo apt-get install python-pip python-dev nginx
```

## 四、创建Python虚拟环境

接下来，为了把Flask应用与系统上其他Python应用隔离开来，我们要创建一个虚拟环境。

使用`pip`安装`virtualenv`软件包：

```bash
sudo pip install virtualenv
```

现在，我们为Flask项目创建一个父目录。创建后进入该目录中：

```bash
mkdir ~/myproject
cd ~/myproject
```

输入下面的命令，我们可以创建一个虚拟环境，用以存储Flask项目所需的Python条件：

```bash
virtualenv myprojectenv
```

这会在你的项目目录中创建一个`myprojectenv`文件夹，该文件夹中安装了一份Python和`pip`的本地拷贝。

在这个虚拟环境中安装应用之前，我们需要激活它。输入下面的命令来激活这个虚拟环境：

```bash
source myprojectenv/bin/activate
```

你的终端中会显示你现在已经处于这个虚拟环境中了，类似`(myprojectenv)user@host:~/myproject$`。

## 五、创建Flask应用

现在你已经处于虚拟环境中了，我们可以安装Flask和uWSGI，并开始设计我们的应用：

### 1) 安装Flask和uWSGI

我们可以使用`pip`的本地实例来安装Flask和uWSGI。输入下面的命令安装这两个组件：

```bash
pip install uwsgi flask
```

### 2) 创建示例App

现在我们已经安装了Flask，可以创一个简单的应用了。Flask是一个微框架。它可能没有一些其他完整特性框架的工具，并主要以模块的形式存在，你可以把它导入到你的项目中帮助你初始化一个Web应用。

当应用比较复杂的时候，我们可以在一个单文件中创建Flask App，我们把这个文件命名为`myproject.py`：

```bash
nano ~/myproject/myproject.py
```

我们在这个文件中编写应用的代码。主要地，我们需要导入flask并实例化一个Flask对象。我们使用这个对象来定义功能，当特定的路由被请求时执行这些功能。在代码中，我们把Flask应用称为`application`，这与WSGI规范的示例一样：

```python
from flask import Flask
application = Flask(__name__)

@application.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    application.run(host='0.0.0.0')
```

这主要定义了当访问根域名时要显示哪些内容。编辑完成后，保存并关闭这个文件。

输入下面的命令，可以测试你的Flask应用：

```bash
python myproject.py
```

在浏览器中输入你的服务器域名或者IP地址并附加上端口号，端口号可以在终端的输出中看到（大部分显示`:5000`）。你应该看到这样的内容：

![显示内容][3]

执行完毕后，在终端中使用`CTRL-C`组合键停止Flask服务。

### 3) 创建WSGI入口点

接下来，我们创建一个文件，作为我们应用的入口点。该文件告诉uWSGI服务器如何与应用交互。

我们把这个文件命名为`wsgi.py`：

```bash
nano ~/myproject/wsgi.py
```

该文件非常简单，我们可以从应用中简单地导入Flask实例并运行它：

```python
from myproject import application

if __name__ == "__main__":
    application.run()
```

编辑完成后，保存并关闭该文件。

## 六、配置uWSGI

我们的应用已经完成并且建立了入口点。现在，让我们把目光转移到uWSGI。

### 1) 测试uWSGI服务

首先我们要测试一下，保证uSWGI能够为我们的应用提供服务。

我们可以通过简单地传递入口点的名称来进行测试。同时，也要指定socket，这样它可以用公开可用的接口和协议来启动，启动时使用HTTP协议而不是`uwsgi`二进制协议：

```bash
uwsgi --socket 0.0.0.0:8000 --protocol=http -w wsgi
```

如果使用浏览器访问你的服务器域名或者IP地址并附加上端口号`:8000`，你可以在浏览器中看到下面的内容：

![显示内容][3]

确认它能正确运行后，在终端中按`CTRL-C`组合键退出uWSGI服务。

现在我们已经用完了虚拟环境，可以关闭它了：

```bash
deactivate
```

所有有关系统的Python环境的操作至此已经完成了。

### 2) 创建uWSGI配置文件

我们已经测试了uWSGI能够为应用提供服务，但我们想要的是能够更健壮地长期使用它。我们可以创建一个uWSGI的配置文件并为其配置我们需要的选项。

把这个文件放到项目目录下，命名为`myproject.ini`：

```bash
nano ~/myproject/myproject.ini
```

在文件内，我们以`[uwsgi]`头开始，这样uWSGI知道如何应用设置。我们要指定`wsgi.py`中提及的模块，去掉扩展名：

```ini
[uwsgi]
module = wsgi
```

接下来，我们要告诉uWSGI以master模式启动并产生五个工作进程用于处理实际请求。

```ini
[uwsgi]
module = wsgi

master = true
processes = 5
```

在测试中，我们把uWSGI放置到了一个网络端口上。然而，我们将要使用Nginx来处理实际的客户端连接，并把这些请求转发给uWSGI。因为这些组件都是在同一台电脑上操作的，建议使用Unix的socket，这样不仅安全而且速度更快。我们把这个socket称为`myproject.sock`，并放到当前目录中。

我们也要更改socket上的权限。后续因为我们要赋予uWSGI进程Nginx组权限，所以需要确保socket的组所有者能从中读写信息。我们还要增加"vacuum"选项，当进程停止时进行socket的清理工作：

```ini
[uwsgi]
module = wsgi

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true
```

最后，我们需要设置`die-on-term`选项。因为Upstart初始化系统和uWSGI对不同进程信号有不同的处理。设置该选项可以让这两个系统组件一致，实现预期的行为：
（译者注：Upstart目前是Linux的一种启动方式，Upstart与uWSGI对于SIGTERM信号有不同的处理方式。为了解决这个差异让Upstart按预期运行，设置`die-on-term`选项，uWSGI会杀掉进程而不是重新加载。）

```ini
[uwsgi]
module = wsgi

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true

die-on-term = true
```

你可能会注意到我们没有像在命令行中那样指定一种协议。这是因为默认情况下uWSGI使用`uwsgi`协议，这是一种设计用来与其他服务器通信的二进制协议。Nginx天生就使用这种协议，所以使用`uwsgi`要比强制使用HTTP通信更好。

`myproject.ini`编辑完成后，保存并关闭该文件。

## 七、创建Upstart脚本

接下来我们要编写Upstart脚本。创建Upstart脚本能够让Ubuntu的初始化系统自动启动uWSGI，不论服务器什么时候启动，都能为我们的Flask应用提供服务。

在`/etc/init`目录下创建一个以`.conf`为扩展名的脚本文件：

```bash
sudo nano /etc/init/myproject.conf
```

最开始我们要添加一个简单的描述，说明该该脚本的用途。然后定义这个脚本启动和停止的条件。正常系统的runtime数量为2，3，4和5，所以我们要告诉系统当达到这其中的runlevel时启动我们的脚本。当处于其他runlevel时停止脚本（例如服务器重启，关闭或者单用户模式时）：

```ini
description "uWSGI server instance configured to serve myproject"

start on runlevel [2345]
stop on runlevel [!2345]
```

接下来，我们需要定义应作为uWSGI运行的用户和组。我们的项目是由自己的账号拥有的，所以我们应该让自己作为运行uWSGI的用户。Nginx服务器运行在`www-data`组下。要让Nginx能够读写socket文件，我们要赋予这个进程的组权限。

```ini
description "uWSGI server instance configured to serve myproject"

start on runlevel [2345]
stop on runlevel [!2345]

setuid user    # 注意替换成自己的用户名
setgid www-data
```

接下来，我们要设置进程让它能够正确地找到我们的文件并进行处理。我们已经在虚拟环境中安装了所有Python组件，所以我们需要设置一个环境变量作为虚拟环境的路径。我们还需要改变目录进入项目目录。然后，简单地调用uWSGI并把写好的配置文件作为参数传递给它：

```ini
description "uWSGI server instance configured to serve myproject"

start on runlevel [2345]
stop on runlevel [!2345]

setuid user    # 注意把user替换成自己的用户名
setgid www-data

env PATH=/home/user/myproject/myprojectenv/bin    # 注意把user替换成自己的用户名
chdir /home/user/myproject                        # 注意把user替换成自己的用户名
exec uwsgi --ini myproject.ini
```

编辑完成后，保存并关闭文件。

输入以下命令可以快速启动uWSGI进程：

```bash
sudo start myproject
```

## 八、配置Nginx代理

我们的uWSGI服务现在应该启动并运行了，等待着项目目录下socket文件中的请求。我们需要配置Nginx使用`uwsgi`协议向socket转发Web请求。

首先在Nginx的`sites-available`目录下创建一个新的服务块配置文件。为了与该指南其他部分一致，我们把该配置文件命名为`myproject`：

```bash
sudo nano /etc/nginx/sites-available/myproject
```

创建服务器代码块告诉Nginx监听默认的80端口。我们也需要告诉它使用这个代码块来处理访问服务器域名或者IP地址的请求：

```ini
server {
    listen 80;
    server_name server_domain_or_IP;    # 注意替换 server_domain_or_IP
}
```

我们需要添加的另一项是一个能够匹配所有请求的本地代码块。在代码块内，需要包含`uwsgi_params`文件，该文件指定了一些常见需要设置的uWSGI参数。然后向`uwsgi_pass`定义的socket传递请求：

```ini
server {
    listen 80;
    server_name server_domain_or_IP;    # 注意替换 server_domain_or_IP

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/user/myproject/myproject.sock; # 注意把user替换成自己的用户名
    }
}
```

实际上，这就是为我们的应用所要做的所有工作。编辑完成后保存并关闭该文件。

为了能够启用我们刚刚创建的Nginx服务块配置文件，还要把它链接到`sites-enabled`目录下：

```bash
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```

使用该目录下的文件，我们可以用以下命令测试语法是否有错误：

```bash
sudo nginx -t
```

如果返回的结果没有问题，我们可以重启Nginx进程以便重新加载新的配置文件：

```bash
sudo service nginx restart
```

现在你应该可以在浏览器中访问服务器域名或IP地址查看你的应用了：

![显示内容][3]

## 九、总结

本指南中，我们使用Python的虚拟环境创建了一个简单的Flask应用。我们创建了WSGI入口点，所有WSGI能支持的应用服务都能与它交互，然后配置了uWSGI服务来提供这个功能。之后，我们创建了Upstart脚本，能够在系统启动时自动启动服务。我们创建了Nginx服务块转发外部请求，能够把Web客户端的流量转发到应用服务中。

Flask是一个非常简单而又非常灵活的框架，这意味着你的应用功能不会过多的受限于它的结构和设计。你可以使用本指南中描述的内容为你的Flask应用提供服务支持。


  [1]: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04 "initial server setup guide"
  [2]: https://www.digitalocean.com/community/tutorials/how-to-set-up-uwsgi-and-nginx-to-serve-python-apps-on-ubuntu-14-04#definitions-and-concepts "How To Set Up uWSGI and Nginx to Serve Python Apps on Ubuntu 14.04"
  [3]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-28_000000_test_app.webp "显示内容"
