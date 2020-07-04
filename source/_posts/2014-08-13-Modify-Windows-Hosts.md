title: Windows中修改HOSTS的方法
date: 2014-08-13 20:45
tags:
- Hosts
permalink: Modify-Windows-Hosts
---

Windows XP/7/8/8.1的hosts位于系统盘的`Windows\System32\drivers\etc`目录下。hosts文件默认的权限只有读取和执行，因此需要在修改hosts之前需要赋予当前用户`完全控制`的权限，之后才可以用编辑器（Notepad++/写字板等）写入`IP 域名`记录。

## Windows 7修改hosts操作步骤

1. 右击hosts文件选择`属性` | `安全`选项卡 | `编辑(E)` | 选中`Users (YourName\Users)` | 勾选`完全控制`，确定后便可以修改hosts文件。
![Windows 7修改hosts权限][1]

2. 修改后防止其他程序对hosts修改，只勾选`允许`列的`读取和执行`、`读取`。

## Windows XP修改hosts操作步骤

1. 点击`我的电脑`菜单栏中的`工具` | `文件夹选项` | `查看` | 把`使用简单文件共享(推荐)`前面的钩去掉。否则在下一步中不会看到hosts属性中的`安全`选项卡。
![勾掉简单文件共享][2]

2. 右击hosts文件选择`属性` | `安全`选项卡 | `用户和组名称(G)`中选择`Users (YourName\Users)` | `允许`列勾选`完全控制`，确定后便可以修改hosts文件。
![Windows XP修改hosts权限][3]

3. 修改之后为了防止其他程序对hosts修改，在`拒绝`列勾选`写入`。


  [1]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2014-08-13_201217.webp "Windows 7修改hosts权限"
  [2]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2014-08-13_210635.webp "勾掉简单文件共享"
  [3]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2014-08-13_200315.webp "Windows XP修改hosts权限"
