title: 利用Hosts屏蔽广告
date: 2013-08-19
tags:
- Hosts
- QQ
- Skype
- Android
permalink: Anti-AD-in-Hosts
---

## Hosts屏蔽QQ广告

```ini
0.0.0.0	fodder.qq.com
0.0.0.0	adshmct.qq.com
0.0.0.0	hm.l.qq.com
0.0.0.0	adshmmsg.qq.com
0.0.0.0	q.i.gdt.qq.com
0.0.0.0	v.gdt.qq.com
```

因为QQ客户端在权限足够的情况下会主动修改hosts文件，所以修改完hosts后需要把权限关掉。修改hosts的方法参考《[Windows中修改HOSTS的方法][1]》。

以下是[木鱼][2]修改hosts并清理QQ广告缓存的脚本，把脚本保存为`.bat`文件，Win7使用管理员权限运行。更详细的介绍，请访问[木鱼的网站][2]。

```bash
@ECHO OFF

ECHO 本批处理仅限于Windows7/8/8.1 或以上系统、并且未关闭UAC的情况下使用
ECHO 或请在运行后手动锁定HOSTS权限，否则QQ可能会通过篡改HOSTS文件的方式让屏蔽失效
ECHO.
ECHO 请关闭正在运行的QQ或TM，按任意键继续
PAUSE 1>NUL 2>NUL
taskkill /im qq.exe
taskkill /im tm.exe

ECHO 正在写入HOSTS文件……如果杀毒软件报警，请允许写入
ECHO 0.0.0.0	fodder.qq.com >>%SYSTEMROOT%\SYSTEM32\DRIVERS\ETC\HOSTS
ECHO 0.0.0.0	adshmct.qq.com >>%SYSTEMROOT%\SYSTEM32\DRIVERS\ETC\HOSTS
ECHO 0.0.0.0	hm.l.qq.com >>%SYSTEMROOT%\SYSTEM32\DRIVERS\ETC\HOSTS
ECHO 0.0.0.0	adshmmsg.qq.com >>%SYSTEMROOT%\SYSTEM32\DRIVERS\ETC\HOSTS
ECHO 0.0.0.0	q.i.gdt.qq.com	>>%SYSTEMROOT%\SYSTEM32\DRIVERS\ETC\HOSTS
ECHO 0.0.0.0	v.gdt.qq.com	>>%SYSTEMROOT%\SYSTEM32\DRIVERS\ETC\HOSTS

ECHO 正在删除本地缓存……
cd "%AppData%\Tencent\QQ\Misc"
for /d %%i in (com.tencent.advertisement*) do rd "%%i" /s /q
cd "%AppData%\Tencent\Users\"
del misc.db /s /q

ECHO.
ECHO 已经搞定了！ o(∩_∩)o
ECHO by 木鱼(2013.10.31)
PAUSE 1>NUL 2>NUL
```

## Hosts屏蔽阿里旺旺广告

```ini
0.0.0.0 click.tanx.com
0.0.0.0 p.tanx.com
```

## Hosts屏蔽Skype广告

```ini
0.0.0.0 rad.msn.com
0.0.0.0 live.rads.msn.com
0.0.0.0 ads1.msn.com
0.0.0.0 static.2mdn.net
0.0.0.0 g.msn.com
0.0.0.0 a.ads2.msads.net
0.0.0.0 b.ads2.msads.net
0.0.0.0 ac3.msn.com
```

## 更多去广告Hosts

更多广告的hosts可以从[huhamhire-hosts][3]中提取。

## 额外的福利

```ini
# Hosts for Android SDK Manager：
64.15.120.23 dl.google.com
64.15.120.23 dl-ssl.google.com
```

`update 2014-10-24`


  [1]: http://g2ex.farbox.com/2014-08-13-modify-windows-hosts
  [2]: http://www.fishlee.net/soft/tools/
  [3]: https://code.google.com/p/huhamhire-hosts/