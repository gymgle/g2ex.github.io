title: Python基础笔记
date: 2014-12-01
tags:
- Python
permalink: Basic-Python-Notes
---

最近要写爬虫，复习了Python的基础知识，记录了学习[零基础学Python][1]过程中的一些笔记。

## 取名的学问

* 文件名：全小写，可使用下划线
* 函数名：小写，可以用下划线分割单词增加可读性
* 函数的参数：如果一个函数的参数名称和保留的关键字冲突，通常使用一个后缀下划线好于使用缩写或奇怪的拼写
* 变量：变量名全部小写，由下划线链接各个单词

## 转义字符

```python
>>> print 'what\'s your name?'   #使用转义字符"\"
what's your name?
>>> print "what's your name?"   #双引号包裹单引号，单引号是字符
what's your name?
>>> print 'what "is your" name' #单引号包裹双引号，双引号是字符
what "is your" name
```

## 字符串操作

1. 用"+"连接
2. 用占位符连接

```python
>>> print "one is %d"%1
one is 1

>>> a = "py"
>>> b = "thon"
>>> print "%s%s"%(a,b)  #注
python
```

字符串长度：len(object)

字符串大小写：

```python
S.upper() #S中的字母大写
S.lower() #S中的字母小写
S.capitalize() #首字母大写
S.title() #所有单词首字母大写
S.istitle() #所有单词首字母是否大写的，且其它为小写
S.isupper() #S中的字母是否全是大写
S.islower() #S中的字母是否全是小写
```

字符串截取：a[n,m]，其中n < m，得到的字符是从a[n]开始到a[m-1]

去字符串两头空格：

```python
S.strip() #去掉字符串的左右空格
S.lstrip() #去掉字符串的左边空格
S.rstrip() #去掉字符串的右边空格
```

> `#!/usr/bin python`：表示python解释器在/usr/bin里面。`#!/usr/bin/env python`表示要通过系统搜索路径寻找python解释器。不同系统，可能解释器的位置不同，所以这种方式能够让代码更将拥有可移植性。以上是对Unix系列操作系统而言。

## 列表list []

类似字符串可以根据编号取元素，list.append(X)追加元素X，len(L)获取L中元素个数，list.extend(L)合并list。

list.count(X)元素X在list中出现的次数。

list.index(X)元素X在list中的位置。

list.insert(i, x)在list的第i个元素之前插入x，如果i==len(list)，意思是在后面追加，就等同于list.append(x)。

删除元素：list.remove(x)，list.pop([i]) # [ ]内为可选

排序：list.sort(cmp=None, key=None, reverse=False)，sorted(iterable[, cmp[, key[, reverse]]]

一个非常重要的帮手：help(list)、dir(list)（注：dir()要更简洁）

list与str的区别：list是可以改变的，str不可变。

list和str转化：`str.split()`这个内置函数实现的是将str转化为list。逆运算是`"[sep]".join(list)`把list中的元素使用sep分隔符组合为字符串。

list解析：
```python
>>> squares = [x**2 for x in range(1,10)]
>>> squares
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

## 字典dict {}

* dict是可变的
* dict可以存储任意数量的Python对象
* dict可以存储任何python数据类型
* dict以：key:value，即“键：值”对的形式存储数据，每个键是唯一的
* dict也被称为关联数组或哈希表

获取键、值：`d.keys()`、`d.values()`
获取键值对：`d.items()`

键值对的个数`len(d)`

删除键值对：`d.pop(key)`、`del d[key]`

`d.update(d2)`把d2合并到d中。

## 元组 tuple ()

元组不能修改。

分别用list()和tuple()能够实现两者的转化：
```python
>>> t
(1, '23', [123, 'abc'], ('python', 'learn'))
>>> tls = list(t) #tuple-->list
>>> tls
[1, '23', [123, 'abc'], ('python', 'learn')]
 
>>> t_tuple = tuple(tls) #list-->tuple
>>> t_tuple
(1, '23', [123, 'abc'], ('python', 'learn'))
```

tuple用在哪里？
> Tuple 比 list 操作速度快。如果您定义了一个值的常量集，并且唯一要用它做的是不断地遍历它，请使用 tuple 代替 list。
如果对不需要修改的数据进行 “写保护”，可以使代码更安全。使用 tuple 而不是 list 如同拥有一个隐含的 assert 语句，说明这一数据是常量。如果必须要改变这些值，则需要执行 tuple 到 list 的转换 (需要使用一个特殊的函数)。
Tuples 可以在 dictionary 中被用做 key，但是 list 不行。实际上，事情要比这更复杂。Dictionary key 必须是不可变的。Tuple 本身是不可改变的，但是如果您有一个 list 的 tuple，那就认为是可变的了，用做 dictionary key 就是不安全的。只有字符串、整数或其它对 dictionary 安全的 tuple 才可以用作 dictionary key。
Tuples 可以用在字符串格式化中，后面会用到。

## 集合 set {}

set不含键值对，set中的元素不可重复。

创建set：
```python
>>> s1 = set("abbc")  #把str中的字符拆解开,形成set.特别注意观察:abbc中有两个b
>>> s1 #但是在s1中,只有一个b,也就是不能重复
set(['a', 'b', 'c'])
 
>>> s2 = set([123,"google","face","book","facebook","book"]) #通过list创建set.不能有重复,元素可以是int/str
>>> s2
set(['facebook', 123, 'google', 'book', 'face']) #元素顺序排列不是按照指定顺序
 
>>> s3 = {"facebook",123} #通过{}直接创建
>>> s3
set([123, 'facebook'])
```
通过{}无法创建含有list/dict元素的set。

更改set：`set.add(x)`增加元素x，`set.update(s2)`把s2合并到当前set中。

set删除：`set.pop()` `set.remove(obj)` `set.discard(obj)`。set.pop()是从set中任意选一个元素，删除并将这个值返回。但是，不能指定删除某个元素。

`set.clear()`清空set。

冻结的集合：
```python
>>> f_set = frozenset("abc") #看这个名字就知道了frozen，冻结的set
>>> f_set
frozenset(['a', 'b', 'c'])
>>> f_set.add("python") #报错
```

子集与超集：set.issubset(s2)
```python
>>> a
set(['a', 'b', 'c'])
>>> c
set(['a', 'b'])
>>> c<a #c是a的子集
True
>>> c.issubset(a) #或者用这种方法，判断c是否是a的子集
True
>>> a.issuperset(c) #判断a是否是c的超集
True
```

并集：`a | b`或者`​a.union(b)`

交集：`a & b`或者`a.intersection(b)`

A相对B的差（补）：`a - b`或者`a.difference(b)`

-A、B的对称差集（(A-B)∪(B-A)）：`a.symmetric_difference(b)`

## Python数据类型总结

对象类型 | 举例
--- | ---
int/float | 123, 3.14
str | 'abc'
list | [1, [2, 'three'], 4]
dict | {'name': 'bill', 'lang':'python'}
tuple | (1, 2, 'three')
set | set('abc'), {'a', 'b', 'c'}

多用`dir(object)`或`help(object)`查看帮助。

## 坑爹的字符编码

`encode()`和`decode()`

python中如何避免中文是乱码？

经验一：在开头声明：`# -*- coding: utf-8 -*-`或者`# coding: utf-8`

经验二：遇到字符（节）串，立刻转化为unicode，不要用str()，直接使用unicode()：
```python
unicode_str = unicode('中文', encoding='utf-8')
print unicode_str.encode('utf-8')
```

经验三：如果对文件操作，打开文件的时候，最好用codecs.open，替代open：
```python
import codecs
codecs.open('filename', encoding='utf8')
```

关于print：
```python
print line,     #后面加一个逗号，就去掉了原来默认增加的\n了
```

## 文件

### 打开文件的模式
r    以读方式打开文件，可读取文件信息。
w    以写方式打开文件，可向文件写入信息。如文件存在，则清空该文件，再写入新内容
a    以追加模式打开文件（即一打开文件，文件指针自动移到文件末尾），如果文件不存在则创建
r+    以读写方式打开文件，可对文件进行读和写操作。
w+    消除文件内容，然后以读写方式打开文件。
a+    以读写方式打开文件，并把文件指针移到文件尾。
b    以二进制模式打开文件，而不是以文本模式。该模式只对Windows或Dos有效，类Unix的文件是用二进制模式进行操作的。

文件相关信息
```python
>>> f = open("131.txt","a")
>>> f.name
'131.txt'
>>> f.mode      #显示当前文件打开的模式
'a'
>>> f.closed    #文件是否关闭，如果关闭，返回True；如果打开，返回False
False
>>> f.close()   #关闭文件的内置函数
>>> f.closed
True
```

文件状态信息
```python
>>> import os
>>> file_stat = os.stat("131.txt")      #查看这个文件的状态
>>> file_stat                           #文件状态是这样的。从下面的内容，有不少从英文单词中可以猜测出来。
posix.stat_result(st_mode=33204, st_ino=5772566L, st_dev=2049L, st_nlink=1, st_uid=1000, st_gid=1000, st_size=69L, st_atime=1407897031, st_mtime=1407734600, st_ctime=1407734600)
 
>>> file_stat.st_ctime                  #这个是文件创建时间
1407734600.0882277                      #换一种方式查看这个时间
>>> import time
>>> time.localtime(file_stat.st_ctime)  #这回看清楚了
time.struct_time(tm_year=2014, tm_mon=8, tm_mday=11, tm_hour=13, tm_min=23, tm_sec=20, tm_wday=0, tm_yday=223, tm_isdst=0)
```

## 格式化

两个变量的值调换：
```python
>>> a = 1
>>> b = 2
>>> a, b = b, a
```

上面本质上是序列赋值。如果左边的变量是序列，右边的对象也是序列，两者将一一对应地进行赋值。

eval()是把字符串中符合python表达式的东西计算出来。
```python
>>> 3+4         #这是一个表达式，python会根据计算法则计算出结果来
7
>>> "3+4"       #这是一个字符串，python就不计算里面的内容了，虽然里面是一个符合python规范的表达式
'3+4'
>>> eval("3+4")　#这里就跟上面不一样了，就把字符串里面的表达式计算出来了
7
```

exec(),这个函数专门来执行字符串或文件里面的python语句：
```python
>>> exec "print 'hello'"
hello
>>> "print 'hello'"
"print 'hello'"
```

print详解：
```python
>>> a = 3.1415926
>>> print "%d"%a    #%d只能输出整数,int类型
3
>>> print "%f"%a　　#%f输出浮点数
3.141593
>>> print "%.2f"%a　#按照要求输出小数位数
3.14
>>> print "%.9f"%a  #如果要求的小数位数过多，后面就用0补全
3.141592600
>>> b = 3
>>> print "%4d"%b   #如果是整数，这样写要求该整数占有四个位置，于是在前面增加三个空格
   3                #而不是写成0003的样式
```

换一种范式：
```python
>>> import math     #引入数学模块
>>> print "PI=%f"%math.pi #默认，将圆周率打印成这个样子
PI=3.141593
>>> print "PI=%10.3f"%math.pi　#约束一下，这个的含义是整数部分加上小数点和小数部分共计10位，并且右对齐
PI=     3.142
>>> print "PI=%-10.3f"%math.pi　#要求显示的左对齐，其余跟上面一样
PI=3.142
>>> print "PI=%06d"%int(math.pi) #整数部分的显示，要求共6位,这样前面用0补足了。
PI=000003
```

%r是万能的吗？
```python
>>> import math
>>> print "PI=%r"%math.pi
PI=3.141592653589793
>>> print "Pi=%r"%int(math.pi)
Pi=3
```

占位符%s调用的是str()函数把对象转化为str类型，而%r是调用了repr()将对象转化为字符串。

格式化基本操作：用format
```python
>>> #先做一个字符串模板
>>> template = "My name is {0}. My website is {1}. I am writing {2}."
 
>>> #用format依次对应模板中的序号内容
>>> template.format("g2ex","g2ex.github.io","python")
'My name is g2ex. My website is g2ex.github.io. I am writing python.'
```

除了可以按照对应顺序（类似占位符了）填充模板中的位置之外，还能这样，用关键字来指明所应该田中的内容。此外，关键字和位置编号可以混用。

## 小函数

lambda：后面直接跟变量，变量后面是冒号，冒号后面是表达式，表达式计算结果就是本函数的返回值。
```python
lambda arg1, arg2, ...argN : expression using arguments
```

map：map()是python的一个内置函数，它的基本样式是：map(func, seq)，func是一个函数，seq是一个序列对象。

reduce：
```python
>>> reduce(lambda x,y: x+y,[1,2,3,4,5])
15
```

filter：
```python
>>> numbers = range(-5,5)
>>> numbers
[-5, -4, -3, -2, -1, 0, 1, 2, 3, 4]
 
>>> filter(lambda x: x>0, numbers)
[1, 2, 3, 4]
 
>>> [x for x in numbers if x>0] #与上面那句等效
[1, 2, 3, 4]
 
>>> filter(lambda c: c!='p', 'python') #能不能对应上面文档说明那句话呢？
'ython' #“If iterable is a string or a tuple, the result also has that type;”
```

## 函数参数
```python
>>> def foo(x,y,z,*args,**kargs):
...     print x
...     print y
...     print z
...     print args
...     print kargs
...
>>> foo('abc',2,"python")
abc
2
python
()
{}
>>> foo(1,2,3,4,5)
1
2
3
(4, 5)
{}
>>> foo(1,2,3,4,5,name="abc")
1
2
3
(4, 5)
{'name': 'abc'}
```

## 类

类构造函数中的第一个参数self用来接收实例化过程中传入的所有数据，这些数据是通过构造函数后面的参数导入的。self是一个实例（准确说是实例的引用变量）。

id()内置函数可以查看object在内存中的地址。

## import模块

当前目录下编写abc.py，使用import abc导入模块，文件名（不含扩展名.py）就是模块名。

import模块的时候，Python会把模块.py编译为模块.pyc，并不是每次import的时候都编译，只有改动了模块.py后在import时Python自动编译。如果改动了模块.py但没有再次import，可以使用内置函数reload(模块)重新加载。

## 目录

`os.path.<attribute>`

常用方法
```python
os.path.abspath(filename) 文件的绝对路径
os.path.isfile(filename) 判断在该路径中，是否存在那个文件，如果存在则返回True，否则False
os.path.split() 参数是目录加文件名，就可以将路径和文件名分开
os.path.exists(path) 判断目录是否存在
os.path.isabs(path) 判断path是否为绝对路径
os.paht.isdir(path) 判断path是否为存在的目录
os.path.join(path1, path2, ..., filename) 将多个路径组合
```

## MySQL

```bash
$ sudo apt-get install mysql-server # Ubuntu中安装MySQL
$ sudo service mysql start # 启动MySQL
# 初次使用设置密码
$ mysql -u root
mysql> GRANT ALL PRIVILEGES ON *.* TO root@localhost IDENTIFIED BY "123456";
# 以后使用密码123456登陆
$ mysql -u root -p
Enter password:
```

常用命令：
```bash
mysql> help;
mysql> show databases;
mysql> create database test character set utf8;
mysql> use test;
mysql> show tables;
mysql> create table users(id int(2) not null primary key auto_increment,username varchar(40),password text,email text)default charset=utf8;
mysql> desc users; # 显示表users的结构
mysql> insert into users(username,password,email) values("foo","123456","foo@gmail.com");
mysql> select * from users;
```

## Python数据库操作

```bash
$ sudo apt-get install python-mysqldb # 安装python-MySQLdb
```

```python
# python中导入
>>> import MySQLdb
# conn是MySQLdb.connect()的实例对象，拥有`commit()` `roolback()` `cursor([cursorclass])`属性
>>> conn = MySQLdb.connect(host="localhost",user="root",passwd="123456",db="databasename",port=3306,charset="utf8")
>>> cur = conn.cursor() # 使用游标
# cursor执行命令的方法：
# execute(query, args):执行单条sql语句。query为sql语句本身，args为参数值的列表。执行后返回值为受影响的行数
# executemany(query, args):执行单条sql语句，但是重复执行参数列表里的参数，返回值为受影响的行数
>>> cur.execute("insert into users (username,password,email) values (%s,%s,%s)",("abc","123456","abc@gmail.com"))
1L
>>> cur.executemany("insert into users (username,password,email) values (%s,%s,%s)",(("google","111222","g@gmail.com"),("facebook","222333","f@face.book"),("github","333444","git@hub.com"),("docker","444555","doc@ker.com")))
4L
# 使用cur.execute()之后必须使用commit()才能提交到数据库中
>>> conn.commit()
# 查询
>>> cur.execute("select * from users")
5L
# 查询记录赋值给lines，可以使用fetchall(self)接收全部返回结果行；fetchmany(size=None)接收size条返回结果行；fetchone()返回一条结果行；scroll(value, mode='relative')移动指针到某一行，如果mode='relative'，则表示从当前所在行移动value条，如果mode='absolute'，则表示从结果集的第一行移动value条。
>>> lines = cur.fetchall()
# 当使用cur = conn.cursor(cursorclass=MySQLdb.cursors.DictCursor)时，cur.fetchall()系列返回的元组中的元素是一个个字典，不指定cursorclass返回的元组中的元素是一个个元组
# 更新数据
>>> cur.execute("update users set username=%s where id=2",("mypython"))
```

## MySQL高级操作

在conn = MySQLdb.connect()中，如果不指定数据库，可以使用conn.select_db("test")选择test数据库，也可以新建数据库。
```python
>>> cur = conn.cursor()
>>> cur.execute("create database newtest")
1L
>>> cur.execute("create table newusers (id int(2) primary key auto_increment, username varchar(20), age int(2), email text)")
0L
# 可以查看表
>>> cur.execute("show tables")
1L
>>> cur.fetchall()
((u'newusers',),)
# 最后要关闭一切
>>> cur.close()
>>> conn.close()
```

## 关于数据库乱码问题

这个问题是编写web时常常困扰程序员的问题，乱码的本质来自于编码格式的设置混乱。所以，要特别提醒诸位注意。在用python-mysqldb的时候，为了放置乱码，可以做如下统一设置：

1. Python文件设置编码utf-8
2. MySQL数据库charset=utf8
3. Python连接MySQL是加上参数charset=utf8
4. 设置Python的默认编码为utf-8：sys.setdefaultencoding(utf-8)

代码示例：
```python
#encoding=utf-8
 
import sys
import MySQLdb
 
reload(sys)
sys.setdefaultencoding('utf-8')
 
db=MySQLdb.connect(user='root',charset='utf8')
```

MySQL的配置文件设置也必须配置成utf8 设置 MySQL 的 my.cnf 文件，在 [client]/[mysqld]部分都设置默认的字符集（通常在/etc/mysql/my.cnf)：
```python
[client] default-character-set = utf8
[mysqld] default-character-set = utf8
```

  [1]: http://looly.gitbooks.io/python-basic/