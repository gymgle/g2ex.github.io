title: "Python3 urllib 获取重定向链接并下载文件"
date: 2015-10-17 17:25:00
tags:
- Python
- urllib
- MSE
permalink: Python-urllib
---

## 一、说明

计划在能上网的计算机中定期下载微软杀毒软件MSE（Microsoft Security Essentials）的离线更新包，方便其他不能上网的计算机更新病毒库。可以用Python编写下载脚本，然后在计算机中添加`任务计划`定期执行，以达到这个目的。

但在[MSE离线更新包][1]页面中，64-bit MSE更新包的下载链接为：http://go.microsoft.com/fwlink/?LinkID=121721&arch=x64 ，点击链接后服务器会HTTP 302跳转，然后返回真实的下载链接，如下图所示。

![302跳转到真实下载链接][2]

随着离线包版本的更新，真实的下载链接会有所变动。可以用Python3 `urllib`的`response.geturl()`获取302跳转后的链接，从而下载更新包并保存到本地。

## 二、代码实现

代码比较简单，在此抛砖引玉。

```python
'''
DownloadMSEUpdatePackage.py: Download MSE offline update package
'''

# coding: utf-8

import os
import logging
import urllib.request

def get_download_url(url):
    '''
    获取跳转后的真实下载链接
    :param url: 页面中的下载链接
    :return: 跳转后的真实下载链接
    '''
    req = urllib.request.Request(url)
    req.add_header('User-Agent','Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko')
    response = urllib.request.urlopen(req)
    dlurl = response.geturl()     # 跳转后的真实下载链接
    return dlurl

def download_file(dlurl):
    '''
    从真实的下载链接下载文件
    :param dlurl: 真实的下载链接
    :return: 下载后的文件
    '''
    req = urllib.request.Request(dlurl)
    req.add_header('User-Agent','Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko')
    response = urllib.request.urlopen(req)
    return response.read()

def save_file(dlurl, dlfolder):
    '''
    把下载后的文件保存到下载目录
    :param dlurl: 真实的下载链接
    :param dlfolder: 下载目录
    :return: None
    '''
    os.chdir(dlfolder)              # 跳转到下载目录
    filename = dlurl.split('/')[-1] # 获取下载文件名
    dlfile = download_file(dlurl)
    with open(filename, 'wb') as f:
        f.write(dlfile)
        f.close()
    return None

if __name__ == '__main__':
    # 设置log
    LOG_FILE = 'E:\\Software\\MSE\\update.log'
    logging.basicConfig(level = logging.DEBUG,
                       format = '%(asctime)s - %(filename)s:%(lineno)s - %(name)s - %(message)s',
                       filename = LOG_FILE,
                       filemode = 'a')

    url = 'http://go.microsoft.com/fwlink/?LinkID=121721&arch=x64'
    dlfolder = 'E:\\Software\\MSE' # 下载目录
    logging.debug('获取下载链接...')
    dlurl = get_download_url(url)  # 真实下载链接
    logging.debug('开始下载...')
    save_file(dlurl, dlfolder)     # 下载并保存文件
    logging.debug('下载完毕.')
```

## 三、创建任务计划程序

把代码保存为`DownloadMSEUpdatePackage.py`，Windows`计算机`右键`管理`打开`任务计划程序`，`创建基本任务`或`创建任务`，设置触发器为每天执行，触发操作为启动程序`python yourdirectory\DownloadMSEUpdatePackage.py`。

![设置触发器][3]

![设置触发操作][4]

如果使用Linux，可以用`crontab`命令制定计划任务，`crontab`命令格式与详细例子请见 http://blog.csdn.net/ethanzhao/article/details/4406017 。


  [1]: http://www.microsoft.com/security/portal/definitions/adl.aspx "MSE离线更新包"
  [2]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-17_164832_1.webp "302跳转到真实下载链接"
  [3]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-17_165928.webp "设置触发器"
  [4]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-17_165941.webp "设置触发操作"
