title: Linux上搭建Jekyll环境
date: 2013-12-05
tags:
- Jekyll
- Github
permalink: Setup-Linux-Jekyll-Environment
---

利用Github Pages搭建了[个人博客][g2ex]，文章可能不止一次地修改，就会造成多次commit，想着本地搭建环境，待文章彻底改好本地测试满意之后，再提交。搭建Github Pages的Jeykll环境总会遇到大大小小的错误，最常见的有`Error: Failed to build gem native extension`。今天总结一下Linux下本地Jekyll搭建的步骤和错误的解决办法，以备后需。

Github Pages基于Jekyll，Jekyll基于Ruby gem，Jekyll的正常工作需要系统中正确安装了Ruby、Rubygems。因为GFW的原因，导致 [rubygems.org][rubygemorg] 存放在 Amazon S3 上面的资源文件间歇性连接失败，大陆用户建议使用[淘宝Ruby更新源][rubytb]替换默认的更新源。如果你有翻墙之术，则可以跳过下面的第3步。

导致错误`Error: Failed to build gem native extension`的原因是已安装Ruby的版本太低，需要使用高版本的Ruby才能解决此问题。<sup>[[了解更多]][more]</sup>为了安装高版本的Ruby，我们先安装`RVM`，使用`RVM`来安装Ruby。

另外，如果你的网络供应商是“长城宽带”或者“宽带通”，那么你可能还会碰到`Hash Sum mismatch`问题或者运行`rvm requiresments`时发生下图类似的错误，这是因为这些网络提供商设置了透明缓存，即便更换系统更新源也无济于事，你有必要使用代理来解决这个问题。<sup>[[了解更多]][more2]</sup>

![运行`rvm requiresments`时发生的错误][Error1]

----------

下面开始在[Ubuntu][ubuntu] / [Linux Mint][mint] / [Elementary OS][elementary]上建本地Jekyll环境，这里以Elementary OS为例。

## 一、安装RVM

安装`RVM`的时候需要用到`curl`，所以先安装`curl`：

```bash
$ sudo apt-get install curl
```

安装完成后，使用`curl`下载`RVM`：

```bash
$ curl -L get.rvm.io | bash -s stable --auto
```

重新加载`.bash_profile`配置文件：

```bash
$ . ~/.bash_profile
```

安装`RVM`所依赖的包：

```bash
$ rvm requirements
```

## 二、安装Ruby

目前ruby的最新版本是ruby 2.0.0，版本可以从[Ruby官网][rubyorg]查找。

安装Ruby 2.0.0：

```bash
$ rvm install 2.0.0
```

安装完成开始使用`2.0.0`：

```bash
$ rvm use 2.0.0
```

查看一下Ruby的具体版本：

```bash
$ ruby -v
```

根据终端显示，说明目前安装的是`ruby 2.0.0-p353`：

```bash
ruby 2.0.0p353 (2013-11-22 revision 43784) [i686-linux]
```

下面设置系统默认使用的Ruby版本，注意最后要加上`-p353`：

```bash
$ rvm --default use 2.0.0-p353
```

## 三、更新Ruby源

首先使用以下命令检查本地gem源：

```bash
$ gem sources
```

终端会显示（有时候显示的可能是`http://rubygems.org/`）：

```bash
*** CURRENT SOURCES ***

https://rubygems.org/
```

使用以下命令把Ruby源更改为淘宝的Ruby源：

```bash
$ gem sources --remove https://rubygems.org/
$ gem sources -a http://ruby.taobao.org/
```

再使用`gem sources`命令检查一下，看源是否已经换成了淘宝的。

## 四、安装Jekyll

至此，Ruby环境已经搭建完成，可以安装Jekyll了：

```bash
$ gem install jekyll -V
```

安装需要一段时间，上条命令中`-V`参数可以查看详细下载和安装情况，也可以不带`-V`参数。

安装完成Jekyll后，进入博客目录，使用`jekyll serve`命令启动Jekyll服务，如果没有错误，就可以使用浏览器访问本地博客了。

## 五、另外

你可能需要`RDiscount`?

如果你的博客中使用了`RDiscount`作为`Markdown`的解释器，那么还需要安装`RDiscount`：

```bash
$ gem install rdiscount
```

RDiscount是使用C写的模版解释器，效率要比Maruku高很多。详情[点我了解更多][RdiscountVS]。

## 六、参考内容

1. http://jekyllrb.com/
2. https://www.ruby-lang.org/
3. http://rubygems.org/
4. http://ruby.taobao.org/
5. http://rubygems.org/gems/rdiscount
6. http://ryanbigg.com/2010/12/ubuntu-ruby-rvm-rails-and-you/
7. http://stackoverflow.com/questions/15796274/rdiscount-error-failed-to-build-gem-native-extension
8. http://stackoverflow.com/questions/373002/better-ruby-markdown-interpreter


[g2ex]: http://g2ex.me "G2ex"
[rubygemorg]: https://rubygems.org/ "Rubygems官网"
[rubytb]: http://ruby.taobao.org/ "Ruby淘宝更新源"
[rubyorg]: https://www.ruby-lang.org/ "Ruby官网"
[more]: http://stackoverflow.com/questions/15796274/rdiscount-error-failed-to-build-gem-native-extension "stackoverflow上的问题"
[more2]: http://forum.ubuntu.org.cn/viewtopic.php?f=52&t=423516&sid=7877f90e773fea818cbafa9e3fd2224f "关于引起更新源索引时Hash Sum mismatch问题的真正原因及解决方案"
[RdiscountVS]: http://stackoverflow.com/questions/373002/better-ruby-markdown-interpreter "stackoverflow上的问题"
[Error1]: https://i.imgur.com/iD40S5b.png "运行 rvm requiresments 时发生的错误"
[ubuntu]: http://www.ubuntu.com/ "Ubuntu官网"
[mint]: http://www.linuxmint.com/ "Linux Mint官网"
[elementary]: http://elementaryos.org/ "Elementary OS官网"