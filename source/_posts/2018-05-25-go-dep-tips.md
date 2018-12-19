title: "go dep 获取被墙的包"
date: 2018-05-25 18:18:00
tag:
- dep
- golang
- shadowsocks
- proxy
permalink: go-dep-tips

---

执行 `dep init` 时被墙的包无法获取，需要设置代理。

Windows / Mac 下使用 **Proxifier** 全局强制代理。

Linux 下可以通过设置 HTTP 代理，让 dep 命令（实际上是让 go、git 走代理）通过代理下载依赖的包。如果没有 HTTP 代理，只有 Shadowsocks 代理的话，可以安装 privoxy，把 SOCKS5 协议转换为 HTTP 协议，默认 privoxy 监听的是 8118 http 端口，以下脚本中的端口号根据自己的 HTTP 代理端口自行更改。

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

添加完成后 `source ~/.bashrc` 或 `source ~/.bash_profile` 或 `source ~/.zshrc` 或 重启终端。需要用 dep 前执行 `proxy`，不需要代理了执行 `unproxy` 或直接退出当前终端。

---

实在嫌麻烦，就在国外的 VPS 上执行 dep 后，把项目同步过来 - -!

> 当然，这里一键设置代理的技巧不仅限于使用 dep。