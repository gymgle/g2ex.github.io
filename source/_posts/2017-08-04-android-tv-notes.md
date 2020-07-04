title: "Android 电视折腾记"
date: 2017-08-04 0:0:0
tags:
- Kodi
- Youtube
- Shadowsocks
- Android
- TV
permalink: android-tv-notes
---

要从 BBC 放出的几段《[Planet Earth II][1]》说起，突发奇想地打算在小米电视上看 YouTube 视频，电视的系统基于 Android 6.0.1，最终通过 Kodi + Youtube 插件成功实现目的。

## 0x00 介绍

Kodi 原名 XBMC，是一个开源跨平台的多媒体播放平台，支持视频、音乐、图片、直播、本地和在线媒体、网络服务等等。Kodi 最让人称奇的是它众多的插件，通过对应的插件，网络上的各种资源就可以完美地在 Kodi 中播放。

不过，在国内，要想观看 YouTube，还要考虑的一个问题是让电视自动翻墙。

这里以小米电视为例，当然也可以是其他 Android 电视，也可以是各种盒子。

## 0x01 让电视科学上网

有以下选择（前提是你有一台海外 VPS，搭建了 Shadowsocks 服务端，或者是配置了 VPN）：

1. 家用路由器翻墙。目前用的是小米路由器，需要刷开发版 ROM 获取 root 权限，才能安装 Shadowsocks 客户端实现自动翻墙，比较麻烦；现在小米 WiFi App 可以设置`智能 VPN`，支持 `选择地址限流` 和 `选择设备限流`，如果选择电视限流的话，电视相当于是全局 VPN 了。
2. 不想折腾路由器的，可以用一台局域网电脑作为家庭代理，安装上 Shadowsocks 和 Privoxy（支持局域网的 SOCKS/http/https 代理）。按照这种思路最好弄个树莓派做家庭代理。
3. 电视上安装翻墙 App，可选 Shadowsocks 和 Postern。

这里选择在电视安装 [Postern App][2]。最主要原因是 Postern 的自动翻墙配置利用了 GEOIP 库可以精准地实现「国内流量直连，国外流量走代理」。Shadowsocks 设置里因为用 `8.8.8.8` DNS（或其他） 去解析域名，国内的某些提供了海外加速的服务就会被解析到国外 IP 上，反而更慢了。另外，Postern 还能在配置里过滤广告。

具体的过程：

1. 下载 Postern App https://github.com/postern-overwal/postern-stuff
2. 下载自动翻墙配置文件 https://github.com/postern-overwal/postern-stuff/blob/master/postern-rule-geoip.conf
3. 把配置文件中的代理改为自己的：

  ```
  [Proxy]
  Proxy = shadowsocks,xxx.xxx.xxx.xxx,1080,aes-256-cfb,password
  ```

4. 小米电视的文件浏览器只能看到视频、图片格式的文件，所以要安装一个文件管理器。下载 MiXplorer https://forum.xda-developers.com/showthread.php?t=1523691 也可以使用你习惯的 App。
5. 把 Postern、修改的 postern-rule-geoip.conf、MiXplorer 放进 U 盘，插到电视上。在电视上安装 Postern 和 MiXplorer。
6. 使用 MiXplorer 把 U 盘里的 postern-rule-geoip.conf 复制到电视内部存储随便一个位置，点击该配置文件，选择使用 Postern 打开，就能把配置文件导入到 Postern 中了。

> 之所以用 MiXplorer 把 U 盘中的配置文件复制到电视内部存储中，是因为在 U 盘中直接打开配置文件，是无法选择使用 Postern 打开的 :(

> 另外，Postern 不仅支持 Shadowsocks，也支持 SSH、SOCKS5、HTTP/HTTPS 代理类型

## 0x02 安装 Kodi 和插件

1. 去[官网][3]下载 Android 平台下的 Kodi。要根据你电视的 CPU 类型选择，一般来说 ARM 系列比较常见。

  ![下载 Kodi][4]

2. 通过 U 盘，在电视上安装 Kodi App。
3. 第一次打开 Kodi 界面是英文，如果想切换中文，需要先在 `Interface settings` 的 `Skin` 里，把 `Fonts` 由默认改为 `Arial based`，否则切换成中文后，中文字体无法显示。然后在 `Interface settings` 的 `Regional` 中把 `Language` 改为 `Chinese (Simple)`。

  ![设置字体][5]

4. 在 Kodi 主界面左侧导航栏，依次点击 `插件` -> `下载` -> `视频插件`，在列表中找到 `YouTube`，然后点击安装。
5. 打开 Postern，在 Kodi 主界面的视频导航栏中，就可以看到视频插件中有刚刚安装的 YouTube 了。点击 `Sign in` 登录 YouTube，需要电脑上打开 youtube.com/activate 输入 8 个字符的验证码去授权电视上的 YouTube 登录。
6. 登录后就可以看到自己 YouTube 账号里的所有视频了，Enjoy~

  ![Kodi Youtube][6]

  ![Kodi Youtube][7]

## 0x03 总结

Kodi 已经可以称作平台级的多媒体工具了，通过插件，还能把玩 Udacity、Vimeo、500px、iPhoto 等等。Kodi 媒体库里能添加本地、家庭路由器硬盘、网络服务中的视频、音乐和图片，也能观看点视直播，可玩性极高。

常见的智能电视和电视盒子，大部分是基于 Android 系统，只是没有手机的屏幕触碰，只能通过遥控器去点按，操作起来费劲。不过可以用蓝牙鼠标连接电视，操作要方便一些。


  [1]: https://www.youtube.com/playlist?list=PLtra-MWzIvZGdqzuA59Jp0dZVzpmNZyT0
  [2]: https://github.com/postern-overwal/postern-stuff
  [3]: https://kodi.tv/download
  [4]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2017-08-04_112344.webp
  [5]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2017-08-04_132530.webp
  [6]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2017-08-04_132531_MITVScreenshot_org.xbmc.kodi.webp
  [7]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2017-08-04_132532_MITVScreenshot_org.xbmc.kodi.webp
