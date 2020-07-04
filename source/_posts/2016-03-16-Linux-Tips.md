title: Linux Tips
date: 2016-03-16
tags:
- Linux
categories:
- Tips
permalink: Linux-Tips
---

## 常用技巧

`ps -aux`查看进程。

`netstat -tln`查看网络开放端口。

`Ctrl+H`查看Ubuntu里以`.`开头的隐藏文件。

在linux Terminal中使命令行程序后台运行，在命令后加个`&`即可，例如：`$ gedit &`。但即使这样，Terminal关闭后程序扔会退出。
使用`nohup`命令可实现关闭Terminal后程序继续运行，方法：`nohup <命令>  &`，例如：`nohup gedit &`。

`pip install xxx`安装xxx包，`pip install --upgrade xxx`升级包，`pip uninstall xxx`卸载包。

查看用户信息：`whoami`、`id`。

`apt-get install`安装软件时，默认会有提示用户确认，加上`-y`参数可以跳过交互模式直接确认。

在虚拟机中安装vmware tools的时候，加上`-d`参数可以按照默认设置安装，跳过交互模式。

`lsb_release -s -c`可以查看系统当前`发行版`的名称，比如在Ubuntu 14.04上执行该命令会输出`trusty`。

## 64位Ubuntu安装32位软件，需要安装32位的库来获得支持
```bash
sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0 lib32stdc++6
```

## Ubuntu 14.04 LTS安装shadowsocks-qt5
```bash
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```

## Kali中设置Terminal快捷键

设置-键盘-快捷键-自定义快捷键，命令中输入`x-terminal-emulator`，点击`添加`后单击该行右侧`禁用`处，按下自定义快捷键，比如`Ctrl + Alt + T`。

![x-terminal-emulator][1]

## Kali更新源以及使用HTTPS更新

以当前Kali Rolling 2016.1版本为例，编辑`/etc/apt/sources.list`，替换为中科大更新源：
```bash
deb http://mirrors.ustc.edu.cn/kali kali-rolling main contrib non-free
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main contrib non-free
```

因为中科大更新源支持**HTTPS**，通过**https**更新需要在**http**更新源下安装`apt-transport-https`包，然后把更新源中的**http**改为**https**。
```bash
deb https://mirrors.ustc.edu.cn/kali kali-rolling main contrib non-free
deb-src https://mirrors.ustc.edu.cn/kali kali-rolling main contrib non-free
```

PS：使用HTTPS更新再也不用怕ISP的缓存劫持了！

## Debian/Kali中图形化安装`deb`的工具`GDebi`
```bash
sudo apt-get install gdebi
```

## 环境变量设置

### 临时环境变量设置
以设置`http_proxy`为例：

```bash
export http_proxy=http://127.0.0.1:8787
```

取消环境变量：
`unset http_proxy`或`export http_proxy=`

### 用户级环境变量配置文件
```bash
~/.pam_environment
~/.profile
~/.bashrc
~/.bash_profile
~/.bash_login
```

### 系统级环境变量配置文件
```bash
/etc/environment
/etc/profile.d/*.sh
/etc/profile
/etc/profile.d/*
/etc/default/locale
/etc/bash.bashrc
```

更多参考[help.ubuntu.com][2]

## SSH的使用

生成rsa公私钥对之前判断文件是否存在：
```bash
cat ~/.ssh/id_rsa.pub
```

ssh-keygen生成rsa公私钥对：
```bash
ssh-keygen -t rsa -C "yourname@domain.com"
```

显示rsa公钥：
```bash
cat ~/.ssh/id_rsa.pub
```

复制到剪贴板（需安装`xclip`）：
```bash
xclip -sel clip < ~/.ssh/id_rsa.pub
```


  [1]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2016-03-16_085908.webp "自定义快捷键"
  [2]: https://help.ubuntu.com/community/EnvironmentVariables
