title: "微信公众平台开发（二）"
date: 2016-09-28 20:55:00
tag:
- Django
- Python
- WeChat
- Media Platform
- Official Accounts
permalink: WeChat-Media-Platform-Development-with-SDK
---

## 简介

在上一篇的 [微信公众平台开发（一）][1] 中总结了微信公众平台的接入过程，服务器端使用 Python 的 Django 框架编程实现微信公众平台消息的验证，从中可以了解 Django 的安装过程和公众平台的接入步骤。

在这一篇中，将总结使用 `wechat-python-sdk` 进行公众平台的开发过程。`wechat-python-sdk` 封装了与微信公众平台之间的交互，使得开发者**「可以专注于业务逻辑本身，而不是浪费精力在与微信服务器复杂的交互中」**，目前支持订阅号、服务号的官方接口。

`wechat-python-sdk` 官网：http://wechat-python-sdk.com/

官网中已经对 SDK 有了详细的使用说明，本文的重点是介绍在 Django 中如何使用 `wechat-python-sdk`，开发一个简单的消息回复程序。

## 预备条件

### 安装 Django

安装 Django，主要命令如下，详细过程请参考 [微信公众平台开发（一）][1] 。

```bash
$ sudo apt-get install python-pip python-dev

$ sudo pip install virtualenv     # 若不使用 virtualenv 可省略
$ mkdir Workspace && cd Workspace # 创建并进入工作目录，可省略
$ virtualenv venv                 # 创建 python 虚拟环境，这里使用默认 Python 版本，也可以使用 -p 参数指定 Python 版本。未使用 virtualenv 请省略此步骤
$ source ./venv/bin/activate      # 激活 virtualenv。未使用 Virtualenv 请省略此步骤

$ pip install Django
```

### 安装 wechat-python-sdk

通过 pip 安装：

```bash
$ pip install wechat-sdk
```

也可以通过 easy_install 或直接下载源码安装，请参考官网安装文档： http://wechat-python-sdk.com/

## 理解 wechat-python-sdk

对于刚刚接触 SDK 的同学来说，首先应该阅读 [快速上手 - SDK 快速介绍][2]，了解 SDK 的两个主要类：**微信配置类** `WechatConf`、**官网接口类** `WechatBasic`。

接下来阅读 [快速上手 - 官方接口][3] 来学习如何使用官方接口进行开发，了解验证如何 **验证服务器请求有效性**、**解析接收到的 XML 消息**、**获取解析后的信息**、**回复消息**。

至此，基本就可以使用 SDK 进行开发了。

## Django 应用中使用 wechat-python-sdk

修改 微信公众平台开发（一） 中的 `wechat` Django 应用，把它改造成使用 `wechat-python-sdk` 的版本。

修改 `wechat/views.py`，导入 `wechat_sdk`。首先初始化 `WechatConf` 类传入基本信息。然后在 `WeChat()` 方法中实例化 `WechatBasic` 类，使用 `check_signature()` 验证公众平台发来的消息合法性，使用 `parse_data()` 提取消息，并用 `isinstance()` 判断消息类型，使用 `response_text()` 组装文字消息回复微信用户。

修改后的文件如下：

```python
# coding: utf-8

import sys
reload(sys)
sys.setdefaultencoding('utf8')

from django.http.response import HttpResponse, HttpResponseBadRequest
from django.views.decorators.csrf import csrf_exempt
from wechat_sdk import WechatConf
from wechat_sdk import WechatBasic
from wechat_sdk.exceptions import ParseError
from wechat_sdk.messages import (TextMessage, VoiceMessage, ImageMessage, VideoMessage, LinkMessage, LocationMessage, EventMessage, ShortVideoMessage)

# 传入基本信息
conf = WechatConf(
    token='your_token', 
    appid='your_appid', 
    appsecret='your_appsecret', 
    encrypt_mode='safe',  # 可选项：normal/compatible/safe，分别对应于 明文/兼容/安全 模式
    encoding_aes_key='your_encoding_aes_key'  # 如果传入此值则必须保证同时传入 token, appid
)

@csrf_exempt
def WeChat(request):
    signature = request.GET.get('signature')
    timestamp = request.GET.get('timestamp')
    nonce = request.GET.get('nonce')
    # 实例化 WechatBasic 类
    wechat_instance = WechatBasic(conf=conf)
    # 验证微信公众平台的消息
    if not wechat_instance.check_signature(signature=signature, timestamp=timestamp, nonce=nonce):
        return HttpResponseBadRequest('Verify Failed')
    else:
        if request.method == 'GET':
           response = request.GET.get('echostr', 'error')
        else:
            try:
                wechat_instance.parse_data(request.body)
                message = wechat_instance.get_message()
                # 判断消息类型
                if isinstance(message, TextMessage):
                    reply_text = 'text'
                elif isinstance(message, VoiceMessage):
                    reply_text = 'voice'
                elif isinstance(message, ImageMessage):
                    reply_text = 'image'
                elif isinstance(message, LinkMessage):
                    reply_text = 'link'
                elif isinstance(message, LocationMessage):
                    reply_text = 'location'
                elif isinstance(message, VideoMessage):
                    reply_text = 'video'
                elif isinstance(message, ShortVideoMessage):
                    reply_text = 'shortvideo'
                else:
                    reply_text = 'other'
                response = wechat_instance.response_text(content=reply_text)
            except ParseError:    
                return HttpResponseBadRequest('Invalid XML Data')
        return HttpResponse(response, content_type="application/xml")
```

## 项目源码

源码已托管至 Github https://github.com/gymgle/Gnotes/tree/master/WeChat/wechat-mp-wechat-python-sdk-demo/mysite

## 踩过的坑

1. **安装 wechat-sdk 时出现 `error: Setup script exited with error: command 'x86_64-linux-gnu-gcc' failed with exit status 1`**
因为需要安装 `python-dev`：

    ```bash
    apt-get install python-dev
    ```
    最好在安装 pip 时同时安装 `python-dev`：

    ```bash
    apt-get install python-pip python-dev
    ```

## 参考内容

* wechat-python-sdk 官网 http://wechat-python-sdk.com/


  [1]: https://g2ex.github.io/2016/09/25/WeChat-Media-Platform-Development-Intro/
  [2]: http://wechat-python-sdk.com/quickstart/intro/
  [3]: http://wechat-python-sdk.com/quickstart/official/