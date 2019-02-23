title: "Golang 项目被墙包的获取"
date: 2018-05-25 18:18:00
tag:
- vendor
- golang
- shadowsocks
- proxy
permalink: go-dep-tips

---

> Updated at 2019-02-23

因为 golang.org 被墙，go 项目打包 vendor 的时候，golang.org/x 的包是无法下载的。现在 `go mod` 成了官方主推的 vendor 管理工具，使用 go mod，可以在 `go.mod` 中使用 `replace` 替换成 github 上对应的库。例如：

```
replace (
	golang.org/x/net v0.0.0-20180821023952-922f4815f713 => github.com/golang/net v0.0.0-20180826012351-8a410e7b638d
	...
)
```

我们一些老项目还是用的 `dep` 工具。不管是 go mod 还是 dep 管理 vendor 依赖包，都可以用 VPN 或代理的方式去使用。

## Windows / Mac 平台

Windows / Mac 下使用 **Proxifier** 全局强制代理。

## Linux 平台

### 全局代理的方式

Linux 下可以通过设置 HTTP 代理，让 go mod、dep 命令（实际上是让 go、git 走代理）通过代理下载依赖的包。如果没有 HTTP 代理，只有 Shadowsocks 代理的话，可以安装 privoxy，把 SOCKS5 协议转换为 HTTP 协议，默认 privoxy 监听的是 8118 http 端口，以下脚本中的端口号根据自己的 HTTP 代理端口自行更改。（`shadowsocks-qt5` 版本的 SS 是直接支持 HTTP 代理的。）

下面的别名方法可以在终端中设置一键代理和取消代理：

在 `~/.bashrc` 或 `~/.bash_profile` 或 `~/.zshrc` （如果使用 zsh）中添加：
```
alias proxy="
    export http_proxy=http://127.0.0.1:8118/;
    export https_proxy=http://127.0.0.1:8118/;
    export all_proxy=http://127.0.0.1:8118/;
    export no_proxy=localhost,127.0.0.0/8,::1;
    export HTTP_PROXY=http://127.0.0.1:8118/;
    export HTTPS_PROXY=http://127.0.0.1:8118/;
    export ALL_PROXY=http://10.0.0.117:8118/;
    export NO_PROXY=localhost,127.0.0.0/8,::1"
alias unproxy="
    unset http_proxy;
    unset https_proxy;
    unset all_proxy;
    unset no_proxy;
    unset HTTP_PROXY;
    unset HTTPS_PROXY;
    unset ALL_PROXY;
    unset NO_PROXY"
```

添加完成后 `source ~/.bashrc` 或 `source ~/.bash_profile` 或 `source ~/.zshrc` 或 重启终端。需要用 go mod、dep 前执行 `proxy`，不需要代理了执行 `unproxy` 或直接退出当前终端。

### 只准对当前程序代理

使用 `proxychains-ng` 工具 https://github.com/rofl0r/proxychains-ng

```bash
$ proxychains4 -q go mod vendor
```

### 类似 VPN 全局的方式

使用 `sshuttle ` 工具 https://github.com/sshuttle/sshuttle

这也是我自己当前最常用的方式。sshuttle 通过设置本机
iptables 来实现本地请求转发到目标 ssh 主机，远程服务器不需要安装任何软件，只要你能 ssh 登录到远程服务器即可。

```bash
# 转发所有流量到远程主机
$ sshuttle -r username@ssh-server 0.0.0.0/0 -v

# 0.0.0.0/0 可以缩写成 0/0
$ sshuttle -r username@ssh-server 0/0 -v

# 只转发 192.168.*.* 网段的请求到远程主机
$ sshuttle -r username@ssh-server 192.168.0.0/16 -v

# -v 表示显示详情 
# -vv 表示更多的信息
```

---

实在嫌麻烦，就在国外的 VPS 上打包 vendor，然后把项目同步过来 - -!

> 本文更是对代理的一种讨论。