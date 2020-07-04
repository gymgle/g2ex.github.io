title: "WebP 更新"
date: 2020-07-04 17:34:00
tag:
- webp
permalink: update-with-webp

---

腾出时间，把博客中的图片转换成了 WebP 格式。

同时，把图床从 imgur 换到了 Github，开启了 jsDelivr 加速。加速规则很简单：

```
# 假如 Bob 的 Github 账号中 Repo 仓库下有一张图片 path2pic.webp
# 如果直接引用 Github 中的图片，链接为：
https://github.com/Bob/Repo/blob/master/path2pic.webp

# jsDelivr 加速后的链接为：
https://cdn.jsdelivr.net/gh/Bob/Repo/path2pic.webp
```

整理记录博客的过程，也是对自己成长的一次梳理。

二十岁，对互联网世界刚刚开始探索，每有新发现，折腾之余又乐于分享。分享时往往站在一个初入门的视角，难免有些拖沓冗余，从图片的使用量也能看出来，前期多于这几年。

三十多岁后，发现了技术的相通性，思想借鉴，用在新的场景下，就换成了一个新名词。新鲜东西的出现，总能看到历史上的影子，探索时也少有了年轻时的激情。

二十岁，痴迷技术，也从技术和新发现中获取成就感。三十多岁，发现技术只是技术，最感谢兴趣的其实是钱。

去年公司解散，同学帮忙内推到大厂，开启了 995 的工作，更没有时间来整理博客。今天腾出来时间整理一番，啰嗦一下以此纪念。

图片转换 WebP 写了个批量处理的脚本，附上[链接](https://github.com/gymgle/gnotes/blob/master/Python/pic2webp.py) 。

