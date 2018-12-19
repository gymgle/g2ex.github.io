title: Scrapy笔记
date: 2014-12-26
tags:
- Python
- Scrapy
permalink: Scrapy-Notes
---

以下是最近写爬虫使用Scrapy的一些笔记。

## 关于Scrapy

Scrapy官网：http://scrapy.org/

Scrapy中文文档：http://scrapy-chs.readthedocs.org/zh_CN/latest/

## urllib和urllib2调试设置

要想看到urllib的调试状态，需要设置httplib的debuglevel：
```python
import urllib, httplib
httplib.HTTPConnection.debuglevel = 1
response = urllib.urlopen("http://wap.shouji.com.cn/wap/wdown/softversion?id=43432")
```

可以看到这样的输出：
```html
send: 'GET /wap/wdown/softversion?id=43432 HTTP/1.0\r\nHost: wap.shouji.com.cn\r\nUser-Agent: Python-urllib/1.17\r\n\r\n'
reply: 'HTTP/1.1 200 OK\r\n'
header: Server: nginx
header: Date: Thu, 27 Nov 2014 04:07:38 GMT
header: Connection: close
header: Cache-Control: no-cache
header: Expires: Thu, 01 Dec 1994 16:00:00 GMT
header: Set-Cookie: JSESSIONID=abc3a8cR57Ob5fBwaBVNu; path=/
header: Content-Length: 0
header: X-Cache: MISS from alicdn.shouji.com.cn
```

urllib2开启调试的方法：
```python
import urllib2
httpHandler = urllib2.HTTPHandler(debuglevel=1)
httpsHandler = urllib2.HTTPSHandler(debuglevel=1)
opener = urllib2.build_opener(httpHandler, httpsHandler)
urllib2.install_opener(opener)
response = urllib2.urlopen("http://xxx.xx/xx")
```

在Python2.7中，urllib与urllib2提供的功能不同，最显著的两个不同是：
1. urllib2可以接受一个Request类的实例来设置URL请求的headers，urllib仅可以接受URL。这意味着，你不可以伪装你的User Agent字符串等。
2. urllib提供urlencode方法用来GET查询字符串的产生，而urllib2没有。这是为何urllib常和urllib2一起使用的原因。

## 指定Scrapy的Referer

在`settings.py`文件中添加：
```html
DEFAULT_REQUEST_HEADERS = {
    'Referer': 'http://www.google.com'
}
```

## 获取302重定向后的URL

网页中的一些下载链接是这种类型的：http://abc.com/xxx?xxx 点击后会跳转到真实的下载链接，这个跳转的过程大多数是临时重定向的过程，HTTP状态码为302。

urllib2能够处理HTTP的301、302重定向网页，使用`geturl()`方法：
```python
link = "http://resget.91.com/Soft/Controller.ashx?action=download&tpl=1&id=41033355"
response = urllib2.urlopen(link)
dllink = response.geturl()
print dllink
```

输出结果是真实的下载地址：
```html
http://bcs.apk.r1.91.com/data/upload/2014/10_01/20/com.chinamworld.main_200333.apk
```

## Scrapy的Item

这是Scrapy例子中的爬虫类：
```python
class DmozSpider(Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/",
    ]

    def parse(self, response):
        sel = Selector(response)
        sites = sel.xpath('//ul[@class="directory-url"]/li')
        items = []

        for site in sites:
            item = Website()
            item['name'] = site.xpath('a/text()').extract()
            item['url'] = site.xpath('a/@href').extract()
            item['description'] = site.xpath('text()').re('-\s[^\n]*\\r')
            items.append(item)

        return items
```

在爬虫类的parse()方法中，返回的items是这种类型的字典：{'name': ['xxx'], 'url': ['xxx'], 'description': ['xxx']}，字典中元素的值是列表的形式。因此想要在`piplines.py`中处理item的元素，需要将元素的值按照列表的方法处理。比如，要想以`utf-8`编码输出：
```python
print "".join(item["name"]).encode('utf-8')
print "".join(item["url"]).encode('utf-8')
print "".join(item["description"]).encode('utf-8')
```

## 判断第三方市场的APK是否正版

原思路：先从官网下载正版APK，计算并记录该APK的MD5等信息。然后从第三方市场使用`urllib`的`urlretrieve()`方法下载APK，计算其MD5值是否同正版的MD5相同。
缺点：每次都要从第三方市场下载APK后才能计算其MD5，如果APK很大，下载耗时，网络带宽浪费严重。

新思路：不下载APK的情况下取其MD5值。
实现原理：使用urllib.urlopen()返回的response的`info()`方法，`info()`中包含了多个字段，其中一项就有资源的MD5值，字段名为：`Content-MD5`。代码示例如下：
```python
import urllib

# 该链接为百度手机助手中招商银行3.0.1的下载链接
dllink = "http://gdown.baidu.com/data/wisegame/24796ec9b5284d40/zhaoshangyinxing_301.apk"
response = urllib.urlopen(dllink)
headers = response.info()
print headers
```

输出如下：
```html
Server: JSP3/2.0.4
Date: Thu, 04 Dec 2014 08:58:28 GMT
Content-Type: application/vnd.android.package-archive
Content-Length: 4964681
Connection: close
ETag: b21e117b24796ec9b5284d40d88c1976
Last-Modified: Fri, 28 Nov 2014 07:17:36 GMT
Expires: Thu, 04 Dec 2014 09:26:00 GMT
Age: 257312
Accept-Ranges: bytes
x-bs-version: 04FD2F39E44F0C2C1C44179DD633BA21
x-bs-request-id: MTAuNDIuMTMzLjU4OjgwODA6Mzg5NTYxMDc2OToyOC9Ob3YvMjAxNCAxNzoyNTozMCA=
x-bs-meta-crc32: 3911572026
Content-MD5: b21e117b24796ec9b5284d40d88c1976
x-bs-client-ip: MjcuMjIxLjQwLjE1NQ==
```

通过下载验证，`Content-MD5`字段就是该apk文件的MD5值，`Content-Length`字段是该apk文件的大小（bytes），`x-bs-meta-crc32`字段是该apk文件的CRC32校验码（十进制表示）。另外，在这里urllib也可以换为urllib2，得到的结果相同。

大多数情况下，使用urllib2请求APK文件的下载链接时都会返回`Content-MD5`字段，但是也有例外。所以在获取其MD5之前要进行一次判断，看headers.has_key("Content-MD5")返回True还是False。

## MySQLdb操作中的错误

### 问题1

```html
TypeError: %d format: a number is required, not str
```

解决办法：MySQL里不需要对对应的变量写%d,只写%s就可以了

### 问题2

当执行以下代码时：
```python
cur.execute("create table test(id int(2) not null primary key auto_increment, name varchar(40)) default charset=utf8")
```
如果表test存在，会出现警告：
```html
Warning: Table 'test' already exists
```

解决办法：其实这只是个警告，不是错误。可以通过设置`sql_notes`忽略这个警告：
```python
SET sql_notes = 0;      -- Temporarily disable the "Table already exists" warning
CREATE TABLE IF NOT EXISTS ...
SET sql_notes = 1;      -- And then re-enable the warning again
```

但是，最好的办法不是忽略警告，而是解决警告：
```python
try:
    conn = MySQLdb.connect(host="localhost",user="root",passwd="1234",db="test",port=3306,charset="utf8")
    cur = conn.cursor()
    cur.execute("show tables")
    rows = cur.fetchall()
    if (u'test',) in rows:
        # test already exists
        print "Table 'test' already exists."
    else:
        # Create table test
        cur.execute("create table test(id int(2) not null primary key auto_increment, name varchar(40)) default charset=utf8")
finally:
    conn.commit()
    if conn:
        cur.close()
        conn.close()
```

## 使用LAMPP中的MySQL服务

Ubuntu中安装了LAMPP，在Scrapy中使用MySQLdb时，提示找不到`mysqld.sock`：
> "Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)"

这是因为LAMPP中MySQL的`mysqld.sock`不在`/var/run/mysqld/`目录中，而是位于`/opt/lampp/var/mysql/`目录下，名字是`mysql.sock`（注意，少一个`d`，不是`mysqld.sock`）。当然我们可以使用`find`命令来查找一下`*.sock`：
```bash
$ sudo find /opt/lampp -name "*.sock"
```

可以看到`mysql.sock`的位置：
```bash
/opt/lampp/var/mysql/mysql.sock
```

解决办法：建立一个'/var/run/mysqld/mysqld.sock'指向'/opt/lampp/var/mysql/mysql.sock'的软连接：
```bash
$ cd /var/run
$ sudo mkdir mysqld # 先创建mysqld文件夹
$ cd mysqld
$ sudo ln -s /opt/lampp/var/mysql/mysql.sock mysqld.sock
```
