#概述

最近对直播特别感兴趣，但苦于没有很好的教程，入门比较困难。刚刚阅读完一遍苹果的HTTP Live Streaming官方文档，找了半天没找到文档中所提到的Streaming Tool下载地址。今天再阅读时，才发现其下载地址，因此记录下来，以方便后来者更容易学习！

对于直播和视频、音频方面，笔者也是菜鸟一枚，未曾学习过。不过后续会充分利用业余时间研究直播、音/视频相关的技术，希望能遇到在这些方面有所建树的朋友，一起研究，一起分享！

记：后续笔者的所有学习研究的笔记都会以文章的形式分享出来，欢迎大家一起加入**直播、音/视频**技术探索行伍中！

#下载Streaming Tool

本篇没有记任何技术知识，只是教大家下载Streaming Tool，也就是需要使用到的音、视频需要用到的分片器。

###第一步：登录官网

打开链接：[HTTP Live Streaming](https://developer.apple.com/streaming/)，然后看到右边栏的Downloads，如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160428-0@2x-e1461814655342.png)

###第二步：登录开发者中心

点击**[Sign in]**就可以进入下载页面，如下所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160428-1@2x-e1461814768436.png)

###第三步：安装工具

安装过程极其简单，双击.dmg文件即可。安装过程中，一路点击下一步，最后输入电脑的密码就可以安装成功了！

#查看手册

在安装Tool完成以后，我们在终端输入媒体文件分片器使用手册命令：

```
$ man mediafilesegmenter
```

然后出现如下的信息，说明安装成功了，就可以直接通过它查看如何使用这个工具了！

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160428-2@2x.png)

#Streaming Tool Github下载

如果你没有开发者账号怎么办？不用着急，笔者已经为大家想到了，所以提供了GITHUB下载地址：

* [HLS Streaming Tool Download](https://github.com/CoderJackyHuang/HLSStreamTools)

#小结

这只是刚刚准备，后面的路还很长，也会很辛苦。虽然现在不是做这方面的工作，不过笔者会坚持下去，以后会慢慢转向找这方面的工作！

如果有对这些特别感兴趣的朋友，可以与笔者一起学习、分享！



