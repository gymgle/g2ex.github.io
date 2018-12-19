title: "微信公众平台开发（一）"
date: 2016-09-25 22:45:00
tag:
- Django
- Python
- WeChat
- Media Platform
- Official Accounts
permalink: WeChat-Media-Platform-Development-Intro
---

## 简介

本文总结了使用 `Django` 开发微信公众号的**后台接入过程**的步骤和经验，主要包括：

1. Django 的安装、配置和应用开发，开发和测试环境为 `Ubuntu 14.04 LTS`；
2. 公众号后台接入过程，并通过微信公众平台的验证；
3. 开发过程中遇到的坑和解决方法。

在后续文章中，会总结基于 [`wechat-python-sdk`][1] 开发微信公众平台的经验。

## 安装 Django

### 选择 Python 版本

根据 Django 版本选择 Python。请注意，代码随 Django 版本不同可能有所变化，如果你想正确运行本文的代码，请确认使用 Django `1.10.1` 版本。这里使用了 Ubuntu 默认的 Python `2.7` 版本。

Django version | Python versions
---------------|----------------
1.8 | 2.7, 3.2 (until the end of 2016), 3.3, 3.4, 3.5
1.9, 1.10 | 2.7, 3.4, 3.5
1.11 | 2.7, 3.4, 3.5, 3.6
2.0 | 3.5+


### 安装 pip

这里使用 pip 安装 Django，pip 已经默认安装在了 Python 2 >= 2.7.9 或 Python 3 >= 3.4 中。如果需要安装 pip，可以使用 Ubuntu 包管理器：

```python
$ sudo apt-get install python-pip python-dev
```

### 安装 virtualenv

virtualenv 使得不同版本的 Python 开发更加方便，如果你不需要 virtualenv，可以跳过该步骤。

```python
$ sudo pip install virtualenv
```

### 安装 Django

```python
$ mkdir Workspace && cd Workspace # 创建并进入工作目录，可省略
$ virtualenv venv                 # 创建 python 虚拟环境，这里使用默认 Python 版本，也可以使用 -p 参数指定 Python 版本。未使用 virtualenv 请省略此步骤
$ source ./venv/bin/activate      # 激活 virtualenv。未使用 Virtualenv 请省略此步骤
$ pip install Django
```

**PS:** Django 的其他安装方法请参考官方文档：https://docs.djangoproject.com/en/1.10/intro/install/

## 创建 Django 项目

创建一个名为 `mysite` 的 Django 项目：

```python
$ django-admin startproject mysite
```
测试一下 Django 项目是否能够正常运行：

```python
$ cd mysite
$ python manage.py runserver
```
正常情况下，会在终端中输出以下内容，打开浏览器访问 `http://127.0.0.1:8000/` 可以看到 `It worked!` 运行结果。

```python
September 25, 2016 - 21:09:58
Django version 1.10.1, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
接下来，按 `CONTROL-C` 停止 Django 服务，创建 `wechat` app：

```python
$ python manage.py startapp wechat
```

> **项目 ( Projects ) vs. 应用 ( apps )**
项目与应用之间有什么不同之处？应用是一个提供功能的 Web 应用 – 例如：一个博客系统、一个公共记录的数据库或者一个简单的投票系统。 项目是针对一个特定的 Web  网站相关的配置和其应用的组合。一个项目可以包含多个应用。一个应用可以在多个项目中使用。

## 配置微信公众号开发者模式

主要注意两点：

1. 进入微信公众平台，填写服务器配置，在 URL 中填入你的服务器 IP 地址或者域名，这里在 IP 地址后加上 `wechat/`，在 Django 的应用中去解析 `wechat/`。注意不要忘记 URL 最后的反斜杠 `/`，否则 Django 不能正确解析微信的验证信息。
2. 定义你自己的 `Token`。

![微信公众号服务器配置][2]

此时点击「提交」按钮，是无法通过验证的，因为我们还没编写 `wechat` 应用。

## 编写 Django 应用

在这一节，修改已经创建的 `wechat` 应用，接入微信公众平台，并通过微信公众平台的验证。验证是指验证消息的确来自微信服务器。提取验证消息中的 `signature`、`timestamp`、`nonce` 和 `echostr`，计算对比 `signature` 字段是否正确。详细过程请参考公众平台官方文档：https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319

现在，`mysite` 项目的目录应该是这样的：

```shell
mysite
├── manage.py
├── mysite
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── wechat
    ├── admin.py
    ├── apps.py
    ├── __init__.py
    ├── models.py
    ├── tests.py
    └── views.py
```

接入微信公众平台主要涉及三个文件：`mysite/urls.py`、`wechat/urls.py` 和 `wechat/views.py`：

* `mysite/urls.py` 负责解析公众平台 URL 请求中的 `wechat/`，指向 `wechat` 应用中定义的 `urls`；
* `wechat/urls.py` 负责解析 Django 提取的 `wechat/` 后的 URL，需要自己手动创建；
* `wechat/views.py` 负责实现对公众平台消息的验证；

### 创建 **wechat/urls.py**

在 `wechat` 目录中创建 `urls.py`，添加以下内容：

```pyton
from django.conf.urls import url, include
from . import views

urlpatterns = [
    url(r'^$', views.WeChat),
]
```

你也许已经注意到了首字母大写的 `WeChat`，这是接下来我们要在 `wechat/views.py` 中编写的公众平台验证方法。

### 修改 **mysite/urls.py**

为 `mysite/urls.py` 添加 `wechat/` 的解析，修改后的 `mysite/urls.py` 为：

```python
from django.conf.urls import url
from django.contrib import admin
from django.conf.urls import include

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^wechat/', include('wechat.urls')),
]
```

### 修改 **wechat/views.py**

添加 `WeChat()` 方法，用于完成对公众平台消息的验证，修改后的 `wechat/views.py` 内容为：

```python
from django.shortcuts import render
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt
import hashlib

@csrf_exempt
def WeChat(request):
    token = 'xxx'    # 填入你的 Token

    signature = request.GET.get('signature', '')
    timestamp = request.GET.get('timestamp', '')
    nonce = request.GET.get('nonce', '')

    # 计算排序后的哈希值并比较
    tmp_sig = hashlib.sha1(''.join(sorted([token, timestamp, nonce]))).hexdigest()
    if tmp_sig == signature:
        return HttpResponse(request.GET.get('echostr', ''))

    return HttpResponse('error')
```

### 运行 Django 应用

至此，`wechat` 应用编写完成。下面，我们需要验证一下应用是否能够正确运行。

启动 Django：

```python
$ python manage.py runserver 0.0.0.0:80
```

如果出现了以下提示，则说明运行成功了。

```
Django version 1.10.1, using settings 'mysite.settings'
Starting development server at http://0.0.0.0:80/
Quit the server with CONTROL-C.
```

在提示信息中，也许还包括以下提示：

```
You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
```
按照要求运行一下 `python manage.py migrate` 即可。

在微信公众平台服务器配置中点击「提交」按钮提交配置信息，显示「提交成功」的消息后说明已经正确接入微信公众平台了。

![成功接入微信公众平台][3]

## 注意事项

1. 如果正式部署 Django，建议配合 `Nginx` 或 `Apache` 作为前端代理；
2. Django 默认开启了调试模式，正式部署项目时为了安全务必**关闭调试**，具体方法是把 `mysite、settings.py` 中的 `DEBUG = True` 改为 `DEBUG = False`；

## 项目源码

源码已托管至 Github https://github.com/gymgle/Gnotes/tree/master/WeChat/wechat-mp-check-signature-demo/mysite

## 踩过的坑

1. **`from django.conf.urls.defaults import * ImportError: No module named defaults`**
因为 `django.conf.urls.default` 在 Django 1.6 中被移除，建议改为 `from django.conf.urls import patterns, url, include`，而在 Django 1.9 中，`django.conf.urls.patterns()` 也被移除了，不需要再导入 patterns，只在 `urls.py` 中 `from django.conf.urls import url, include`：

    ```python
    urlpatterns = [
       url(...),
    ]
    ```

2. **`AttributeError: 'WSGIRequest' object has no attribute 'REQUEST'`**
 `request.REQUEST.get('signature', '')` 的用法已经过时，改为 `request.GET.get("page")` 或 `request.POST.get("page")`

3. **`CommandError: You must set settings.ALLOWED_HOSTS if DEBUG is False.`**
修改 `settings.py`，把 `ALLOWED_HOSTS = []` 改为：`ALLOWED_HOSTS = ['*']`

4. **微信公众平台配置正常，但自己的服务器一直收不到用户信息**
在微信公众号配置页面，注意 URL 最后带上 `/`：`http://xxx/wechat/`
因为 Django 会解析的形式是 `wechat/?x=xx&x=xx&...`

5. **Unknown command: 'syncdb'**
因为在Django 1.9 及以后的版本中使用 `migrate` 代替了 `syscdb`。

6. **怎么设置 Django 1.10 的时区和语言**
修改 `settings.py`：

    ```
    LANGUAGE_CODE = 'zh-Hans'
    TIME_ZONE = 'Asia/Shanghai'
    ```

7. **Django 1.10 INSTALLED_APPS 的设置问题**
在 `settings.py` 中：

    ```
    INSTALLED_APPS = [
        ...
        'wechat.apps.WechatConfig', # 不再是直接写 wechat，否则导入 models 时会出错
    ]
    ```

## 参考内容

* Django 官方文档 https://www.djangoproject.com/start/
* Django 1.8.2 文档 http://python.usyiyi.cn/django_182/index.html
* pip 安装文档 https://pip.pypa.io/en/stable/installing/
* Virtualenv 安装文档 https://virtualenv.pypa.io/en/stable/installation/
* 微信公众平台官方文档 https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319


  [1]: http://wechat-python-sdk.com/ "wechat-python-sdk"
  [2]: https://i.imgur.com/i62SYEb.png "微信公众号服务器配置"
  [3]: https://i.imgur.com/OEU2KDM.png "成功接入微信公众平台"