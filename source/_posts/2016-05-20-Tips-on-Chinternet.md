title: "我的科学上网技巧"
date: 2016-05-20 23:30:00
tag:
- Shadowsocks
- Tor
- GFW
permalink: Tips-on-Chinternet
---

尽管也称互联网，但在国内，如若有以下一个或多个需求，你就不得不学习一下科学上网了：
1. 访问的网站或服务被（GFW）墙；
2. 网络提供商（ISP）劫持了网络流量；
3. 需要匿名（隐藏真实 IP）的情况，如渗透测试；
4. 躲避网络监控；
5. 其他情况。

这里分享一下我的科学上网技巧，使用到的工具软件可能有 `Shadowsocks`、`Lantern`、`Privoxy`、`Tor`。根据不同的目的选择不同的工具。

## 一、绕过 GFW 和 ISP 劫持

最简单和不折腾的方法是购买 VPN，次之是购买一台海外 VPS，自己安装 VPN 或 Shadowsocks 服务端。VPN 的优点是可以全局翻墙，Shadowsocks 虽然可以设置全局模式，但是对于不支持代理的本地应用是没有办法的，当然这种情况下可以试试 `ProxifierPE` 强制所有连接通过代理上网。

使用 Shadowsocks 可能遇到的另一个问题是，因为 Shadowsocks 使用的是 SOCKS5 类型的代理，当本地应用只支持 HTTP/HTTPS 类型代理时，就需要自己解决 HTTP 转换为 SOCKS5 的问题。当然，这个问题使用 `Privoxy` 就能很好地解决。

本节主要介绍如何组合使用 Shadowsocks 和 Privoxy，并假设你已经安装并配置好了 Shadowsocks（Linux 中可以安装 shadowsocks-qt5 或 命令行版本的 Shadowsocks）。

Shadowsocks 与 Privoxy 组合使用的示意图如下所示：

![Shadowsocks_Privoxy][1]

假设你已经配置好了 Shadowsocks，接下来配置 Privoxy。
**Windows** 系统中右键点击 Privoxy 托盘图标，依次点击 `Edit` - `Main Configuration` 打开配置文件；**Linux** 系统中 Privoxy 的配置文件位于`/etc/privoxy/config`。
配置文件修改为：
```ini
# 把 HTTP 流量转发到本机 127.0.0.1:1080 的 Shadowsocks
forward-socks5 / 127.0.0.1:1080 .

# 可选，默认只监听本地连接 127.0.0.1:8118
# 可以允许局域网中的连接
listen-address 0.0.0.0:8118
```

启动 Shadowsocks 和 Privoxy 后，把本地应用的代理设置为 HTTP/HTTPS 类型的 `127.0.0.1:8118`，就可以绕过 GFW 和 ISP 了。如果局域网中的其他 PC 或手机也希望使用该电脑上网（假设该电脑 IP 地址为 `192.168.1.10`），把它们的代理设置为 `19.168.1.10:8118` 即可。

**Tips**

目前，有一个讨巧的的办法替代这两者的组合：使用 [Lantern][2] —— 一款[开源][3]的安全上网工具。它使用的是 HTTP 类型代理，本地端口为 `8787`，也就是说，把本地应用的代理设置为 `127.0.0.1:8787` 就可以使用 HTTP 类型的代理了。而且使用它，也无需自己购买 VPS。

Lantern 默认非全局代理，可以在设置中改为全局模式。

另一方面，Lantern 只监听本机 `127.0.0.1:8787` 的连接，如果局域网中的电脑或手机也想通过这台电脑翻墙（假设该电脑 IP 地址为 192.168.1.10），那么也需要配合 Privoxy 使用。这时，Privoxy 的配置应该如下：
```ini
# 监听局域网中连接到本地 8118 端口的连接，转发给 8787 端口的 Lantern
forward / 127.0.0.1:8787
listen-address 127.0.0.1:8118    # 可改为 0.0.0.0:8118 允许局域网的连接
```

## 二、匿名上网

网络中保持匿名的办法是使用 [Tor][4]，匿名的意思是隐藏你当前的 IP 地址。Internet 上有很多志愿者运行着 Tor 中继节点，Tor 能保证从出发点的流量至少经过三个不同的中继节点到达目的地址，而且这三个不同的中继不会每次都相同。

形象地说，你想把一封匿名信交给小明，在大街上随便找了一陌生人 A 让 A 帮忙转交，A 走了一段路程后随便找了一个陌生人 B 让 B 帮忙转交，B 走了一段路程后随便找了一个陌生人 C 让 C 帮忙转交，最终 C 按照信封上的地址找到了小明并把信交给了他。小明只知道是 C 转交了这封信，至于是谁写的这封信，他就无从得知了。A、B、C 分别对应着 Tor 网络的中继节点，这种投递匿名信的方式就起到了隐藏 IP 地址的效果。

如下图所示，使用 Tor 从本机经过三跳访问了 Google。

![Tor_Browser][5]

### Tor 浏览器

为了方便使用 Tor，Tor 开发者把 Tor 集成到了定制版的 Firefox 中，简单设置一下就能正常使用。

Tor 浏览器专为大陆等网络环境加入了流量混淆的选项。首次打开浏览器时会弹出 Tor 状态检查，点击`设置`配置 Tor 网桥，勾选`互联网提供商（ISP）是否对 Tor 网络连接进行了封锁或审查`中的`是`，把下一步中的网桥类型选择`meek-amazon`或`meek-azure`。这两者在大陆没被完全封锁，因此可以用来做跳板网桥。

不过随着网络环境的恶化，Tor 提供的网桥类型都不可用时，就需要使用自己的 Shadowsocks 或 Lantern 代理了。

首次打开 Tor 浏览器，在 Tor 设置中勾选`互联网提供商（ISP）是否对 Tor 网络连接进行了封锁或审查`中的`否`，在下一步`是否需要本地代理访问互联网？`中选择`是`，下一步中设置你的代理：

* 使用 Shadowsocks 则设置为 SOCKS5 类型的 `127.0.0.1:1080`；
* 使用 Lantern 则设置为 HTTP/HTTPS 类型的 `127.0.0.1:8787`；

如果通过局域网中其他计算机的配置联网，把 `127.0.0.1` 改为那台计算机的 IP 地址。

使用了代理的 Tor 浏览器原理示意图如下（以 Tor + Shadowsocks 组合为例）：

![Tor_Shadowsocks][6]

### Tor Expert Bundle

Tor 浏览器适用于使用浏览器匿名上网的场景，如果打算让本地应用（如其他浏览器、Linux 中的 Terminal 等）也使用 Tor 隐藏 IP 地址，那么就需要自己手动配置 Tor 了。从 [Tor 官网下载][7] 操作系统对应的 `Expert Bundle`，它只包含 Tor 工具，不含浏览器。

假设你已经配置好了 Shadowsocks，接下来，为 Tor 配置 Shadowsocks 代理。

**Windows** 系统中，打开 `%AppData%/tor` 目录（如果不存在则创建该目录），新建 `torrc` 文件，内容如下：
```ini
## 通过 SOCKS5 代理
SOCKS5Proxy 127.0.0.1:1080
## 如果使用 HTTP/HTTPS 代理
# HTTPSProxy 127.0.0.1:8118

## 如果只允许特定端口的网络连接，如 80 和 443
ReachableAddresses *:80,*:443
ReachableAddresses reject *:*
```

**Linux** 系统中，Tor 的配置文件位于 `/etc/tor/torrc`，配置内容同上。

启动 Shadowsocks 和 Tor，因为 Tor 监听的是 SOCKS5 类型的本地 `9050` 端口，把需要匿名的本地应用代理设置为 SOCKS5 类型，代理地址设置为 `127.0.0.1:9050`，实现匿名上网。

还是老问题，如果本地应用只支持 HTTP/HTTPS 代理类型，那么仍需要使用 Privoxy，修改其配置文件为：
```ini
forward-socks5 / 127.0.0.1:9050 .
listen-address 127.0.0.1:8118    # 可改为 0.0.0.0:8118 允许局域网的连接
```

这样一来，本地应用的代理设置为 HTTP/HTTPS 类型的 `127.0.0.1:8118` 就可以实现匿名上网了。

Privoxy + Tor + Shadowsocks 组合使用示意图如下所示：

![此处输入图片的描述][8]


### One More Tip

使用 Tor 匿名访问网络除了应用于渗透测试，另一个应用场景是编写爬虫变换 IP 地址爬取站点，减小被网站屏蔽的可能性。

在 Linux Terminal 中，使用 export 命令设置代理，可以只在当前 Terminal 中生效，
```bash
export http_proxy=http://127.0.0.1:8118
export https_proxy=https://127.0.0.1:8118
```

## 三、总结

接下来简要归纳上述内容，给出每种组合的配置内容。

### Privoxy + Shadowsocks 组合 —— 翻墙

配置 Privoxy，Linux 系统中位于 `/etc/privoxy/config`：
```ini
forward-socks5 / 127.0.0.1:1080 .
listen-address 127.0.0.1:8118    # 可改为 0.0.0.0:8118 允许局域网的连接
```

本地代理需设置为 `HTTP/HTTPS` 类型的 `127.0.0.1:8118`。
如果不使用 Privoxy，本地代理需设置为 `SOCKS5` 类型的 `127.0.0.1:1080`。

### Privoxy + Tor + Shadowsocks 组合 —— 匿名上网

配置 Privoxy，Linux 系统中位于 `/etc/privoxy/config`：
```ini
forward-socks5 / 127.0.0.1:9050 .
listen-address 127.0.0.1:8118    # 可改为 0.0.0.0:8118 允许局域网的连接
```

配置 Tor，Windows 系统中位于 `%AppData%/tor/torrc`，Linux 系统中位于 `/etc/tor/torrc`：

```ini
SOCKS5Proxy 127.0.0.1:1080    # Shadowsocks 代理地址
ReachableAddresses *:80,*:443
ReachableAddresses reject *:*
```

本地代理需设置为 `HTTP/HTTPS` 类型的 `127.0.0.1:8118`。
如果不使用 Privoxy，本地代理需设置为 `SOCKS5` 类型的 `127.0.0.1:9050`。

### Privoxy + Tor + Lantern 组合 —— 匿名上网

配置 Privoxy，Linux 系统中位于 `/etc/privoxy/config`：
```ini
forward-socks5 / 127.0.0.1:9050 .
listen-address 127.0.0.1:8118    # 可改为 0.0.0.0:8118 允许局域网的连接
```

配置 Tor，Windows 系统中位于 `%AppData%/tor/torrc`，Linux 系统中位于 `/etc/tor/torrc`：
```ini
HTTPSProxy 127.0.0.1:8787    # Lantern 代理地址
ReachableAddresses *:80,*:443
ReachableAddresses reject *:*
```

本地代理需设置为 `HTTP/HTTPS` 类型的 `127.0.0.1:8118`。
如果不使用 Privoxy，本地代理需设置为 `SOCKS5` 类型的 `127.0.0.1:9050`。

------
EOF


  [1]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2016-05-20_chinternet.webp "Shadowsocks Privoxy 组合使用示意图"
  [2]: https://www.getlantern.org/ "Lantern"
  [3]: https://github.com/getlantern/lantern "Lantern Github 主页"
  [4]: https://www.torproject.org/ "Tor 官网"
  [5]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2016-05-20_150316.webp "Tor Browser"
  [6]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2016-05-20_chinternet_02.webp "Tor 浏览器 + Shadowsocks 组合使用示意图"
  [7]: https://www.torproject.org/download/ "Tor 下载"
  [8]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2016-05-20_chinternet_03.webp "Privoxy + Tor + Shadowsocks 组合使用示意图"
