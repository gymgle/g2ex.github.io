title: VPN Gate免费L2TP/IPsec代理
date: 2014-09-01 
tags:
- VPN
permalink: Get-VPN-Gate-L2TP-IPsec-IP
---

## VPN Gate 说明

[VPN Gate][1] 提供了多种协议的公共代理服务器，其中L2TP/IPsec的代理使用起来最为方便，不需要VPN客户端，可以在Windows、Mac、iOS、Android上使用。

在 [VPN Gate 列表中][2] 找支持L2TP/IPsec的代理IP，详细的配置方法参考[《通过使用 L2TP/IPsec VPN 协议连接到 VPN Gate》][3]。如果无法打开该页面，点击截图：[https://i.imgur.com/1tsnYZA.png][4]。

## 获取VPN Gate L2TP/IPsec IP地址的办法

**由于VPN Gate被墙**，所以想了一个办法在GAE上部署小程序用于获取VPN Gate的L2TP/IPsec代理IP：~~http://getvpngate.appspot.com/~~。

**但是appspot也被墙了**，万万没想到有人做了appspot的反向代理，把`xxx.appspot.com`写成`xxx.appsp0t.com`就可以访问了：[http://getvpngate.appsp0t.com][5]，可以把它加入收藏夹！

如果提示`No L2TP IP`，试着多刷新几次！

## 关于GAE小程序

GAE程序非常简单，使用Google App Engine Launcher新建一个Python 2.7的App，然后修改`main.py`：

```python
import webapp2
import re
import urllib2

class MainHandler(webapp2.RequestHandler):
    def get(self):
        html = urllib2.urlopen("http://www.vpngate.net/cn/").read()
        ips = re.findall(r"<td class='vg_table_row_1'><b><span style='font-size: 12pt;'>(.+?)</span></b></td><td class='vg_table_row_1' style='text-align: right;'><b><span style='font-size: 10pt;'>(.+?)</span></b><BR><span style='font-size: 9pt;'>(.+?)</span><BR>(.+?)<b>L2TP/IPsec<BR>连接指南</b>", html)
        if ips:
            self.response.write(ips[0][0])
        else:
            self.response.write("No L2TP IP")

app = webapp2.WSGIApplication([
    ('/', MainHandler)
], debug=True)
```

部署到GAE之前修改`app.yaml`中的`application: `为自己的app id。

项目Github：[https://github.com/gymgle/VPNGateL2TP][6]


  [1]: http://www.vpngate.net/cn/
  [2]: http://www.vpngate.net/cn/#LIST
  [3]: http://www.vpngate.net/cn/howto_l2tp.aspx
  [4]: https://i.imgur.com/1tsnYZA.png
  [5]: http://getvpngate.appsp0t.com
  [6]: https://github.com/gymgle/VPNGateL2TP