title: CryptDB的安装与使用
date: 2015-10-23 22:00:00
tags:
- CryptDB
- MySQL
- Ubuntu
permalink: CryptDB-Notes
---

## 一、CryptDB介绍

CryptDB是来自MIT的一个开源项目，它不是某种数据库，而是加密数据库查询技术的一种，可以在加密的数据库（目前支持MySQL）上进行简单的操作。正常说来，一个应用是直接连接数据库的，配置了CryptDB后，CryptDB作为应用和数据库的中间代理，以明文的方式与应用交互，以密文的方式与数据库交互。其原理示意图如下：

![CryptDB原理图][1]

CryptDB首次解决了实用性的问题，它将数据嵌套进多个加密层，每层使用不同的密钥，这些加密密钥与用户的密码有关，即便是数据库管理员也不能访问这些加密的数据，这也防止了因数据库泄露导致用户信息泄露的问题。虽然目前支持的SQL语句有限，还没有到真正投入使用的程度，但其性却非常出众。Google也根据CryptDB的设计开发了[Encrypted BigQuery client][2]。

CryptDB官网的介绍如下：

> Online applications are vulnerable to theft of sensitive information because adversaries can exploit software bugs to gain access to private data, and because curious or malicious administrators may capture and leak data. CryptDB is a system that provides practical and provable confidentiality in the face of these attacks for applications backed by SQL databases. It works by executing SQL queries over encrypted data using a collection of efficient SQL-aware encryption schemes. CryptDB can also chain encryption keys to user passwords, so that a data item can be decrypted only by using the password of one of the users with access to that data. As a result, a database administrator never gets access to decrypted data, and even if all servers are compromised, an adversary cannot decrypt the data of any user who is not logged in. An analysis of a trace of 126 million SQL queries from a production MySQL server shows that CryptDB can support operations over encrypted data for 99.5% of the 128,840 columns seen in the trace. Our evaluation shows that CryptDB has low overhead, reducing throughput by 14.5% for phpBB, a web forum application, and by 26% for queries from TPC-C, compared to unmodified MySQL. Chaining encryption keys to user passwords requires 11-13 unique schema annotations to secure more than 20 sensitive fields and 2-7 lines of source code changes for three multi-user web applications.

## 二、安装CryptDB

安装系统：Ubuntu 12.04/13.04/14.04 x64

注：`gcc`和`MySQL`可以事先安装，也可以在安装CryptDB时自动安装。如果不更改CryptDB的默认密码`letmein`，那么配置`MySQL`时`root`用户密码也要设置为`letmein `。

### 1) 安装Git和ruby

安装`git`是为了获取官网的源码，又因为CryptDB的安装脚本使用ruby语言编写，因此也需要安装`ruby`。

```bash
$ sudo apt-get install git ruby
```

### 2) 克隆CrpytDB代码

这里把CryptDB项目克隆到了用户主目录下，后续步骤中也安装到了这个目录。根据个人喜好，可以自定义安装目录。

```bash
$ git clone -b public git://g.csail.mit.edu/cryptdb
```

### 3) 安装CrpytDB

安装非常简单，执行安装脚本，按照提示，等待完成。

```bash
$ cd cryptdb
$ sudo ./scripts/install.rb .
```

如果安装过程中出现问题，请翻阅下文中的`可能出现的问题`一节。

### 4) 添加环境变量`EDBDIR`到`.bashrc`

编辑用户主目录`/home/yourname/`下的`.bashrc`，把下面的代码添加到最后，注意把`/full/path/to/cryptdb/`替换为`CryptDB`的安装目录。后续步骤中出现的`/path/to/cryptdb/`也请注意替换。

```bash
export EDBDIR=/full/path/to/cryptdb/
```

## 三、CryptDB的使用

源码`doc/README`文档中介绍了CryptDB的三种使用方法，这里只介绍`Proxy`方法。其他两种分别是`Tests`和`Shell`方法，使用方法请参考CryptDB的说明文档。

### 1) 启用Proxy

MySQL使用本地`3306`端口，CryptDB使用本地`3307`端口，CryptDB把`3307`端口的数据处理后通过`3306`端口与MySQL交互。

打开Terminal（我们称其为`Terminal 1`），替换CryptDB路径，复制后按`Shift + Insert`粘贴到Terminal中并执行：

```bash
/path/to/cryptdb/bins/proxy-bin/bin/mysql-proxy  \
   --plugins=proxy --event-threads=4             \
   --max-open-files=1024                         \
   --proxy-lua-script=$EDBDIR/mysqlproxy/wrapper.lua \
   --proxy-address=127.0.0.1:3307                \
   --proxy-backend-addresses=localhost:3306
```

如果执行成功，`Terminal 1`中会显示：
```bash
2015-10-22 23:26:30: (critical) plugin proxy 0.8.4 started
```

### 2) 连接到CryptDB

接下来另外打开一个Terminal（我们称其为`Terminal 2`）连接到本机`3307`端口的CryptDB。如果想直连MySQL，使用默认的`mysql -u root -p`即可。

```bash
mysql -u root -pletmein -h 127.0.0.1 -P 3307
# 或者隐藏密码登录
mysql -u root -p -h 127.0.0.1 -P 3307
Enter password:
```

默认用户名密码：`root`/`letmein`，如果登陆出现问题，请确认`MySQL` `root`用户的密码与`CryptDB`的密码相同。

### 3) CryptDB使用演示

现在，Ubuntu中已经有两个Terminal了，一个启用3307端口代理的`Terminal 1`，另一个是与CryptDB建立连接的`Terminal 2`。以下在`Terminal 2`中查询、创建数据库表、增加记录、查询记录，可以看到`Terminal 1`中会有CryptDB对应的操作。

Terminal 2，查询数据库：
```SQL
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| cryptdb_udf        |
| mysql              |
| performance_schema |
| remote_db          |
+--------------------+
5 rows in set (0.02 sec):
```

Terminal 1，CryptDB查询数据库结果：
```SQL
QUERY: show databases	
NEW QUERY: show databases
```

Terminal 2，创建数据库：
```SQL
mysql> create database test;
Query OK, 1 row affected (0.05 sec)
```

Terminal 1，CryptDB创建数据库结果：
```SQL
QUERY: create database test	
NEW QUERY: create database `test`
ENCRYPTED RESULTS:
+
|
+
+
```

Terminal 2，打开刚创建的数据库：
```SQL
mysql> use test;
Database changed
```

Terminal 1，CryptDB打开数据库结果：
```SQL
QUERY: show tables	
NEW QUERY: show tables
ENCRYPTED RESULTS:
+--------------------+
|Tables_in_test      |
+--------------------+
+--------------------+
```

Terminal 2，新建数据库表：
```SQL
mysql> create table users(id int(2) not null primary key auto_increment,username varchar(40),password varchar(40));
Query OK, 0 rows affected (0.08 sec)
```

Terminal 1，CryptDB新建数据库表结果：
```SQL
QUERY: create table users(id int(2) not null primary key auto_increment,username varchar(40),password varchar(40))	
NEW QUERY: create table table_XOJUPFKJFH (MZJAQVYXMSoPLAIN INT(2) unsigned not null auto_increment, ZJPTNSDEPOoEq VARBINARY(80), UBPDDGBBFPoOrder BIGINT unsigned, cdb_saltOLGYFXFRJM BIGINT(8) unsigned, NCWFXJZAQOoEq VARBINARY(80), QKRGQVOTWToOrder BIGINT unsigned, cdb_saltUWACWHCBBL BIGINT(8) unsigned, PRIMARY KEY index_16383471738825684568 (MZJAQVYXMSoPLAIN)) AUTO_INCREMENT=0 ENGINE=InnoDB
ENCRYPTED RESULTS:
+
|
+
+
```

Terminal 2：表中增加记录：
```SQL
mysql> insert into users(username,password) values("foo","123456");
Query OK, 1 row affected (0.05 sec)
```

Terminal 1，CryptDB增加记录结果：
```SQL
QUERY: insert into users(username,password) values("foo","123456")	
NEW QUERY: insert into `test`.`table_XOJUPFKJFH` (`test`.`table_XOJUPFKJFH`.`ZJPTNSDEPOoEq`, `test`.`table_XOJUPFKJFH`.`UBPDDGBBFPoOrder`, `test`.`table_XOJUPFKJFH`.`cdb_saltOLGYFXFRJM`, `test`.`table_XOJUPFKJFH`.`NCWFXJZAQOoEq`, `test`.`table_XOJUPFKJFH`.`QKRGQVOTWToOrder`, `test`.`table_XOJUPFKJFH`.`cdb_saltUWACWHCBBL`, `test`.`table_XOJUPFKJFH`.`MZJAQVYXMSoPLAIN`) values ('?\Z??-;??|?m???]??\\pV??O?GP$?u???t???????????T??p', 6053682719380228167, 11367024404434184659, '?T?? r?????\nc??O$??R\n??bU?????o?*)FD??\n|?*?9????', 2215019363748817985, 12229882942316980603, '\'0\'')
ENCRYPTED RESULTS:
+
|
+
+
```

Terminal 2，查询表中的记录：
```SQL
mysql> select * from users;
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
| 1    | foo      | 123456   |
+------+----------+----------+
1 row in set (0.02 sec)
```

Terminal 1，CryptDB查询记录结果：
```SQL
QUERY: select * from users	
NEW QUERY: select `test`.`table_XOJUPFKJFH`.`MZJAQVYXMSoPLAIN`,`test`.`table_XOJUPFKJFH`.`ZJPTNSDEPOoEq`,`test`.`table_XOJUPFKJFH`.`cdb_saltOLGYFXFRJM`,`test`.`table_XOJUPFKJFH`.`NCWFXJZAQOoEq`,`test`.`table_XOJUPFKJFH`.`cdb_saltUWACWHCBBL` from `test`.`table_XOJUPFKJFH`
ENCRYPTED RESULTS:
+--------------------+--------------------+--------------------+--------------------+--------------------+
|MZJAQVYXMSoPLAIN    |ZJPTNSDEPOoEq       |cdb_saltOLGYFXFRJM  |NCWFXJZAQOoEq       |cdb_saltUWACWHCBBL  |
+--------------------+--------------------+--------------------+--------------------+--------------------+
|1                   |????-;??|?m???]??\pV??O?GP$?u???t???????????T??p|11367024404434184659|?T?? r??????c??O$??R???bU?????o?*)FD???|?*?9????|12229882942316980603|
+--------------------+--------------------+--------------------+--------------------+--------------------+
```

从CryptDB对应的结果可以看出，不仅数据库中的记录是加密的，表中的字段也是加密的。另外，直连到`3306`端口的MySQL，也能看到test数据库和表中的内容是加密的。

## 四、修改CryptDB密码

根据不同运行CryptDB的方式，密码保存的文件有所不同，上一节中使用代理的方式开启CryptDB服务，这种方式的密码保存在`cryptdb\mysqlproxy\wrapper.lua`中。修改该文件，把默认的`letmein`替换为自己的密码。

```lua
os.getenv("CRYPTDB_PASS") or "letmein" # 替换letmein
```

> PS：关于另外两种方式的说明
使用`Tests`方式运行CryptDB，修改：`test/test_utils.hh`
使用`Shell`方式运行CryptDB，修改：`main/cdb_test.cc`

修改完CryptDB的密码，别忘了修改MySQL的密码，这两者的密码要相同：
```SQL
$ mysqladmin -u root -p password # 修改MySQL的密码
Enter password:                  # 输入当前密码
New password:                    # 输入新密码
Confirm new password:            # 再次输入新密码
```

## 五、重新编译CryptDB

如果修改了源码，在cryptdb目录中执行`make`就可以重新编译CryptDB。如果修改了UDFs，还需要以root权限执行`make install`。

## 六、可能出现的问题

### 1) 安装时出错

**错误描述：**

```bash
/home/yourname/cryptdb/mysql-src/sql/sql_yacc.yy:30:23: error: ‘yythd’ was not declared in this scope
#define YYTHD ((THD *)yythd)
^
/home/yourname/cryptdb/mysql-src/sql/sql_yacc.yy:37:14: note: in expansion of macro ‘YYTHD’
#define Lex (YYTHD->lex)
^
/home/yourname/cryptdb/mysql-src/sql/sql_yacc.yy:14618:23: note: in expansion of macro ‘Lex’
LEX *lex= Lex;
^
make[2]: *** [sql/CMakeFiles/sql.dir/sql_yacc.cc.o] Error 1
make[1]: *** [sql/CMakeFiles/sql.dir/all] Error 2
make: *** [all] Error 2
./scripts/install.rb:176:in `pretty_execute': `make` failed (RuntimeError)
	from ./scripts/install.rb:169:in `>'
	from ./scripts/install.rb:135:in `fn'
	from ./scripts/install.rb:281:in `<main>'
```

**原因：**

对于出现的`yythd`错误，是因为安装CrpytDB时把`bison`自动更新到了最新版（bison 3 导致上述问题），需要去掉CryptDB安装脚本`install.rb`中的`bison`自动更新，手动安装`bison 2`。

**解决步骤：**

**(1) 去掉`script/install.rb`中的`bison`：**

打开`install.rb`，查找`bison`，可以看到`bison`位于`apt-get`中，说明安装时会自动更新`bison`。删除该位置的`bison`，保存`install.rb`。

**(2) 手动安装`bison 2`：**

```bash
wget http://launchpadlibrarian.net/140087283/libbison-dev_2.7.1.dfsg-1_amd64.deb
wget http://launchpadlibrarian.net/140087282/bison_2.7.1.dfsg-1_amd64.deb
dpkg -i libbison-dev_2.7.1.dfsg-1_amd64.deb
dpkg -i bison_2.7.1.dfsg-1_amd64.deb
```

### 2) 删除数据库出错

**错误描述：**
```bash
QUERY: select @@version_comment limit 1	
main/Connect.cc:112 (execute): mysql_query: Unknown database 'yourdroppedDB'
main/Connect.cc:113 (execute): on query: USE `yourdroppedDB `
mysql-proxy: main/rewrite_main.cc:126: std::map<std::basic_string<char>, int> collectTableNames(const string&, const std::unique_ptr<Connect>&): Assertion `c->execute("USE " + quoteText(db_name))' failed.
Aborted (core dumped)
```

**原因：**
如果使用`3306`端口的MySQL而不是通过CryptDB删除加密数据库，会导致CryptDB仍然查询已经被删除的数据库而出现上述问题。

**解决办法：**
删除`cryptdb\shadow`目录下的所有文件。

## 七、参考

https://css.csail.mit.edu/cryptdb/
CryptDB源码中`doc/README`文档
http://whitehatty.com/2012/09/30/cryptdb-howto-compile-on-ubuntu-linux-12-04/


  [1]: https://i.imgur.com/j10zY12.jpg "CryptDB原理图"
  [2]: https://github.com/google/encrypted-bigquery-client "Google Encrypted BigQuery"