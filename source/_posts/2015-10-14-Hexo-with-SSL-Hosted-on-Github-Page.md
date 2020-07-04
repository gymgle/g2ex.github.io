title: 为Github的Hexo博客启用SSL/TLS
date: 2015-10-14 19:44:00
tags:
- Hexo
- SSL
- Github
permalink: Hexo-with-SSL-Hosted-on-Github-Page
---

CloudFlare的免费套餐提供了SSL，可以用它为我们独立域名的博客启用HTTPS。本文以 http://g2ex.me 为例。

## 主要步骤

1. 注册CloudFlare，添加个人网站，获取CLoudFlare提供的`Nameservers`;
2. 修改自己的域名提供商，把`站点的Nameservers`修改为`CloudFlare提供的Nameservers`；
3. 等待CloudFlare添加的网站为激活状态，使用`https`打开个人网站；
4. 修改网站模版，使`http`跳转到`https`。

## 详细步骤

### 一、注册CloudFlare

首先注册CloudFlare，注册后按照提示`Add Websites`，输入域名后点击`Begin Scan`：

![Begin Scan][1]

到达最后一步，会提示把自己网站的域名`Name Server`更换为：

```html
charles.ns.cloudflare.com
ivy.ns.cloudflare.com
```

![CloudFlare Name Servers][2]

### 二、修改域名提供商的Nameservers

本站使用了Godaddy域名提供商，登录Godaddy，在域名的`SETTINGS`中，点击`Nameservers`下的`Manage`:

![Godaddy Nameservers][3]

勾选`Custom`并点击`ADD NAMESERVER`，添加上边CloudFlare给的两个Name Servers。

![自定义Nameserver][4]

![添加CloudFlare提供的Nameservers][5]

### 三、等待CloudFlare确认

稍等片刻（几分钟到十几分钟），在CloudFlare中点击`Recheck Nameservers`，可以看到网站已经处于激活状态了。

![CloudFlare网站激活][6]

之后，便可以用 https://g2ex.me 访问站点了。

### 四、强制跳转

至此，必须手动输入`https`前缀才能访问加密的站点，要想在任何情况下都以加密方式访问网站，可以在网站模版的头中加入`http`到`https`的强制跳转。

以当前Hexo的[NexT主题][7]为例，打开`layout`目录下的`_layout.swig`，在`<head>`标签中加入以下代码，注意把`yoursite.com`替换为你的域名，这里为`g2ex.me`。

```html
<script type="text/javascript">
    var host = "yoursite.com";
    if ((host == window.location.host) && (window.location.protocol != "https:"))
        window.location.protocol = "https";
</script>
```

修改后，使用Hexo重新部署到Github上，完毕。

## 参考文章

https://blog.xingoxu.com/setup-log/blog-setup/github-pages-ssl.html


  [1]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-14_115601.webp "Begin Scan"
  [2]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-14_115333.webp "CloudFlare Name Servers"
  [3]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-14_120422.webp "Godaddy Nameservers"
  [4]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-14_130458.webp "自定义Nameserver"
  [5]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-14_130854.webp "添加CloudFlare提供的Nameservers"
  [6]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2015-10-14_131608.webp "CloudFlare网站激活"
  [7]: https://github.com/iissnan/hexo-theme-next "Hexo NexT主题"
