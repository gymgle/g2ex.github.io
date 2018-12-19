title: Ubuntu 14.04部署ownCloud 8.2
date: 2015-11-08 23:00:00
tags:
- ownCloud
- Ubuntu
- Apache
- MySQL
- PHP
permalink: Install-ownCloud-8-2-on-Ubuntu-14-04
---

为了更好地用户体验，[ownCloud从8.1版本开始不再把Windows列入原生支持的平台][1]。因此，最好在类Linux平台上部署ownCloud的服务器端。

ownCloud服务器端使用PHP编写，因此部署ownCloud需要Apache、MySQL/MariaDB（也支持SQLite、PostgreSQL或Oracle）、PHP的环境。类似我们搭建一个PHP Blog或CMS，可以从[Apache Friends][2]下载一套XAMPP快速安装，也可以按照本文中从Ubuntu软件源中单独安装Apache、MySQL和PHP，配置好环境后再安装ownCloud。

结合参考官网文档，在Ubuntu 14.04上部署ownCloud 8.2的步骤如下。

## 一、安装apapche2

```bash
sudo apt-get install apache2
```

## 二、安装PHP

### 1) 安装PHP

```bash
sudo apt-get install php5 php5-mysql
```

### 2) 安装ownCloud所需模块

```bash
sudo apt-get install php5-gd php5-json php5-curl php5-intl php5-mcrypt php5-imagick
```

## 三、安装MySQL

### 1) 安装MySQL服务端

```bash
sudo apt-get install mysql-server
```

### 2) 安全配置MySQL

MySQL的默认配置并不安全，因此需要使用`mysql_secure_installation`脚本对MySQL进行安全配置。配置项包括设置root密码（在安装MySQL时应该已经设置）、移除匿名用户、禁止远程root登录、移除test数据库和访问权限，最后重载授权表。

```bash
sudo mysql_secure_installation
```

### 3) 创建ownCloud数据库

下面命令中为ownCloud创建的用户`ownclouduser`和密码以及数据库`ownclouddb`会在之后首次配置ownCloud时使用。你也可以使用其他名字。

```bash
#mysql -u root -p
Enter password:

mysql> CREATE USER 'ownclouduser'@'localhost' IDENTIFIED BY 'YOURPASSWORD';
mysql> CREATE DATABASE ownclouddb;
mysql> GRANT ALL ON ownclouddb.* TO 'ownclouduser'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> exit
```

## 四、安装ownCloud 8

ownCloud官方介绍了[4种][3]针对不同环境安装ownCloud的方法：

* Archive File —— For server owners
* </> Web Installer —— For Shared hosts
* Packages —— For auto updates
* Applications —— For easy deployment

为了方便自动更新，本文使用`Packages`的方法安装ownCloud服务器端，该方法非常简单。

### 1) 下载ownCloud公钥

下载针对`xUbuntu 14.04`的`ownCloud公钥`到我们本地的可信任列表中：

```bash
wget -nv https://download.owncloud.org/download/repositories/8.2/xUbuntu_14.04/Release.key -O Release.key
sudo apt-key add - < Release.key
```

### 2) 添加更新源安装ownCloud

添加ownCloud仓库到Ubuntu更新源中，更新软件包列表后安装ownCloud：

```bash
sudo sh -c "echo 'deb http://download.owncloud.org/download/repositories/8.2/xUbuntu_14.04/ /' >> /etc/apt/sources.list.d/owncloud.list"
sudo apt-get update
sudo apt-get install owncloud
```

### 3) 配置ownCloud

现在，在浏览器中输入`http://localhost/owncloud`后应该可以看到ownCloud初始化配置界面了。创建一个管理员账户，数据库选择`MySQL/MariaDB`，分别输入在第三节中创建的ownCloud`数据库用户`、`密码`和`数据库名称`，点击`Finish setup`进入管理员页面。

![初始化配置ownCloud][4]

至此，ownCloud安装和初始化配置已经全部完成。

接下来，你可以根据需要修改ownCloud的设置，比如启用HTTPS、启用服务器文件加密（需先点击左上角`Files`->`Apps`->`Not enabled`，启用`Default encryption module`）等等。

ownCloud默认没有启用文件加密，`root`用户可以在`/var/www/owncloud/data/用户名/files`目录下查看用户的文件——未加密的明文文件！

## 五、参考内容

1. http://idroot.net/tutorials/install-owncloud-8-ubuntu-14-04/
2. https://download.owncloud.org/download/repositories/stable/owncloud/


  [1]: https://owncloud.org/blog/owncloud-server-8-1-will-not-support-windows-as-server-platform-natively/ "ownCloud Server 8.1 Will Not Support Windows as Server Platform Natively"
  [2]: https://www.apachefriends.org/ "Apache Friendslatform Natively"
  [3]: https://owncloud.org/install/#instructions-server "部署ownCloud"
  [4]: https://i.imgur.com/W6sMUZN.png "初始化配置ownCloud"