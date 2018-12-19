title: VMware Workstation 10安装Mac OS X Mountain Lion 10.8.5
date: 2014-10-12
tags:
- Mac
permalink: VMware-Workstation-10-Install-Mac-OS-X-Mountain-Lion-10.8.5
---

## 关于原版OS X Mountain Lion 10.8.5

Mac OS X Mountain Lion 10.8.5作为Mountain Lion的最后一个稳定版本值得我们收藏。可能大家有所不知，10.8.5版本是分为两个Build的，一个是在2013年9月13日发布的`10.8.5 Build 12F37`，另一个是2013年10月3日发布的`10.8.5 Build 12F45`。也就是说，`10.8.5 Build 12F45`才是Mountain Lion的最终版本。[`OS X Mountain Lion的维基百科`][1]

不幸的是，网友们和论坛中分享的[OS X Mountain Lion 10.8.5 正式版 原版完整DMG安装镜像][2]大多数是Build 12F37版本（从发帖日期就可以看出来），网上搜索到的种子文件也是Build 12F37的种子。要想下载原版Build 12F45，可以搜索`OSX1085-12F45-ESD.dmg`，或者从这里下载：http://pan.baidu.com/s/1f68Vv

## 怎么知道下载了哪个版本？

通过文件的MD5等校验值来辨别。使用软件：[Hash][3]或者[HashTab][4]。

**OS X Mountain Lion 10.8.5 Build 12F37.dmg 信息如下：**
大小: 4469250353 字节
MD5: 5568B4DDE00A64F765EF00858B538078
SHA1: ECF68C2119C71825839D2A58E0D619E9CCF7C026
CRC32: F4DFCE4D
**从中提取出的InstallESD.dmg：**
MD5: 2C77151BE45C820B02A9ACE05434693D
SHA1: 2919B519142E2119197BFFD678F15F603E84970F
CRC32: A9DCAE18

**OSX1085-12F45-ESD.dmg 信息如下：**
大小: 4448808132 字节
MD5: 3FCEBFC81D00767D1ACEF1CB166F88CC
SHA1: 98E52D0FC443940265780539A311833EE5814DDD
CRC32: C82F14C1
**从中提取出的InstallESD.dmg：**
大小: 4434015077 字节
MD5: 69FA8DBBC2AA6668534CC863AC9B9F28
SHA1: B8FA63882F06B52EB73F6ECC6661858DE32E70E9
CRC32: ED788FDE

## VMware安装OS X 10.8.5注意事项

这里使用的VMware Workstation版本是10.0.3，VMware 9也可以。

1. 电脑CPU需要支持虚拟化技术，同时BIOS开启Virtualization Technology选项。使用[`securable`][5]软件来检测。
对于Intel CPU可以在这里查看自己的型号是否支持虚拟化技术：http://ark.intel.com/Products/VirtualizationTechnology

2. 安装`unlock-all-v120`，为Vmware Workstation添加Mac OS X支持。

3. 制作原版InstallESD.ISO镜像
**Build 12F45镜像的制作方法：**
① 使用7-zip打开`OSX1085-12F45-ESD.dmg`，提取出`InstallESD.dmg`；
② 使用UltraISO，把`InstallESD.dmg`转换为`标准ISO`文件；
**Build 12F37镜像的制作方法：**
① 使用7-zip打开`Install OS X Mountain Lion 10.8.5 Build 12F37.dmg`，提取出`2.hfs`；
② 使用7-zip打开`2.hfs`，提取出`InstallESD.dmg`；
③ 使用UltraISO，把`InstallESD.dmg`转换为`标准ISO`文件；

4. VMware中安装OS X
Vmware Workstation新建虚拟机向导，使用自定义，选Apple Mac OS X 10.8，内存大于2G，虚拟磁盘存储为单个文件，其余可以全部默认。把换后的`InstallESD.dmg`加载到VMware的CD/DVD中，开始安装直至完成。

5. 安装VMware Tools
找到VMware Workstation安装目录下的`darwin.iso`，加载到VMware的CD/DVD中，在OS X虚拟机中安装VMware Tools。

## 附件下载

> `securable`下载链接：http://pan.baidu.com/s/1hqxf91i 密码：`wpbs`
`unlock-all-v120.zip`下载链接：http://pan.baidu.com/s/1o69eCUe 密码：`7jsl`
`Hash v1.04`下载链接：http://pan.baidu.com/s/1c0my352 密码：`2hia`
`HashTab v5.2.0.14`下载链接：http://pan.baidu.com/s/1pJ2rbqN 密码：`yuta`

## 其他问题

问题1：不能下载安装OS X所需的附加组件 / 无法下载安装MAC OS X所需的其他组件
原因之一：OS X 10.8.5安装镜像有问题，请确认下载的原版镜像或者从中提取的`InstallESD.dmg`MD5是否正确。如果使用U盘安装，确认镜像没问题后换一个U盘试试。
原因之二：苹果系统版本不对。

## 扩展阅读

VMware Workstation 9安装MAC OS 10.8全程图解：http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1130227


  [1]: https://en.wikipedia.org/wiki/OS_X_Mountain_Lion
  [2]: http://bbs.pcbeta.com/viewthread-1404580-1-1.html
  [3]: http://keir.net/hash.html
  [4]: http://implbits.com/products/hashtab/
  [5]: https://www.grc.com/securable.htm