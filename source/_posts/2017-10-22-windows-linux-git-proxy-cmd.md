title: "Win/Linux 命令行、终端和 Git 代理设置"
date: 2017-10-22 0:0:0
tag:
- Proxy
- Git
permalink: windows-linux-git-proxy-cmd
---

本文整理了 Windows 命令行 和 Linux 终端以及 Git 中设置代理的命令。以本地 HTTP/HTTPS 代理 `127.0.0.1:8118` 和 SOCKS5 代理 `127.0.0.1:1080` 为例。

## Windows 命令行代理设置

HTTP 代理设置：
```
set http_proxy=http://127.0.0.1:8118
set https_proxy=http://127.0.0.1:8118
```

SOCKS5 代理设置：
```
set http_proxy=socks5://127.0.0.1:1080
set https_proxy=socks5://127.0.0.1:1080
```

可以通过 `echo %http_proxy%` 命令查看是否设置成功。

取消代理设置：
```
set http_proxy=
set https_proxy=
```


## Linux 终端代理设置

### 临时代理设置

Linux 终端设置 HTTP 代理（只对当前终端有效）：
```
$ export http_proxy=http://127.0.0.1:8118
$ export https_proxy=http://127.0.0.1:8118
```

Linux 中设置 SOCKS5 代理（只对当前终端有效）：
```
$ export http_proxy=socks5://127.0.0.1:1080
$ export https_proxy=socks5://127.0.0.1:1080
```

设置终端中的 wget、curl 等都走 SOCKS5 代理（只对当前终端有效）：
```
$ export ALL_PROXY=socks5://127.0.0.1:1080
```

Linux 终端中取消代理设置：
```
$ unset http_proxy
$ unset https_proxy
$ unset ALL_RPOXY
```

### 永久代理设置

将代理命令写入配置文件 `~/.profile` 或 `~/.bashrc` 或 `~/.zshrc` 中：
```
# HTTP 代理设置
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
```
或 
```
# SOCKS5 代理设置
export http_proxy=socks5://127.0.0.1:1080
export https_proxy=socks5://127.0.0.1:1080
```
或 
```
# 强制终端中的 wget、curl 等都走 SOCKS5 代理
export ALL_PROXY=socks5://127.0.0.1:1080
```

## Git 设置代理

代理格式 `[protocol://][user[:password]@]proxyhost[:port]`
参考 https://git-scm.com/docs/git-config

设置 HTTP 代理：
```
git config --global http.proxy http://127.0.0.1:8118
git config --global https.proxy http://127.0.0.1:8118
```

设置 SOCKS5 代理：
```
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
```

Git 取消代理设置：
```
git config --global --unset http.proxy
git config --global --unset https.proxy
```
