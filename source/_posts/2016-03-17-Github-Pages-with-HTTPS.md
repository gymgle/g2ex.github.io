title: Github Pages站点HTTPS使用技巧
date: 2016-03-17 10:15:00
tag:
- Github
- Hexo
permalink: Github-Pages-HTTPS-Tips
---

其实早在两年前的2014年3月，Github Pages就开始支持HTTPS了。部署好你的站点，不需要多做什么就可以直接使用`https://yourname.github.io`访问了。

不过Github Pages的HTTPS使用有两个需要注意的地方：
1. 它只支持Github二级域名`yourname.github.io`的形式，不支持自定义的独立域名；
2. 它同时支持HTTP和HTTPS的访问，但不能强制`HTTP`跳转到`HTTPS`；

第一个问题的解决办法，可以参考本站去年10月份的一篇[《为Github的Hexo博客启用SSL/TLS》][1]，使用CloudFlare的免费套餐提供的SSL为独立域名的博客启用HTTPS连接。

关于第二个问题，如果你不想申请独立域名，同时只保留HTTPS访问，那么可以参考本文以下两个办法。

## 一、强制HTTP跳转到HTTPS

因为Github Pages托管的是静态站点，所以无法在HTML中使用`<meta refresh>`标签动态跳转，只能在页面中使用JavaScript。

找到网站主题中的模版，以Hexo的NexT主题为例，打开该主题`layout`目录下的`_layout.swig`，因为`_layout.swig`会被所有页面引用，只需在它的`<head>`标签中加入以下代码：

```html
<script type="text/javascript">
    var host = "yourname.github.io";
    if ((host == window.location.host) && (window.location.protocol != "https:"))
        window.location.protocol = "https";
</script>
```

如果你的Github Pages站点没有模版，那就只好在所有页面的`<head>`标签中都加入上面的JavaScript代码了。

## 二、搜索引擎优化

使用[canonical][2]告诉搜索引擎收录你站点的HTTPS版本。

其实强制HTTP跳转到HTTPS就已经解决基本问题了，搜索引擎优化这个步骤是一个锦上添花的过程。既然行动了，那就做得完美吧！

同样以NexT主题为例，在_layout.swig的<head>标签中加入以下代码：
```html
<link rel="canonical" href="https://yourname.github.io" />
```

## 参考文章
https://konklone.com/post/github-pages-now-sorta-supports-https-so-use-it


  [1]: https://g2ex.github.io/2015/10/14/Hexo-with-SSL-Hosted-on-Github-Page/
  [2]: https://support.google.com/webmasters/answer/139066