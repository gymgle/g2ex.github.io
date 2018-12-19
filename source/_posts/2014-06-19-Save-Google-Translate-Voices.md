title: 保存Google Translate的朗读声音
date: 2014-06-19 23:28
tags:
- Google
permalink: Save-Google-Translate-Voices
---

Google Translate的声音越来越好听了，怎么把她的朗读声音保存下来呢？

其实用不着录音软件，在Windows中通过查找Internet Explorer浏览器的临时文件就可以把朗读的MP3文件找出来。

1. 打开IE（这里是在Windows 7中使用的IE 11），在`工具`中找到`Internet 选项`，`浏览历史记录`的下面有个`删除`和`设置`按钮，为了后续步骤的方便，这里可以先点击`删除`按钮清空一下临时文件、历史记录和Cookie等，然后点击`设置`按钮，如下图所示：
![清空临时文件后点击设置][1]
点击`查看文件`就可以打开IE的临时文件目录了，如下图所示：
![查看临时文件目录][2]
IE的临时目录一般位于`C:\Users\你的用户名\AppData\Local\Microsoft\Windows\Temporary Internet Files`下。

2. 打开[Google Translate][3]，输入文字，点击朗读的小喇叭<i class='icon-volume-up'></i>，刷新一下IE的临时目录，多出来的MP3文件就是刚刚听到的声音了。如果IE临时目录中没有出现MP3文件，试着 **以管理员身份运行** IE。

3. 如果在Google Translate中输入的文字太多，在IE的临时目录下会生成多个MP3文件，可以巧用WinRAR把多个MP3文件合并为一个。


  [1]: https://i.imgur.com/OWAQ0pb.png "清空临时文件后点击设置"
  [2]: https://i.imgur.com/35UhL7O.png "查看临时文件目录"
  [3]: https://translate.google.com/ "Google Translate"