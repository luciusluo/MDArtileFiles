#概述

今天工程玩着玩着突然编译不了了，而且还弹出个窗口说没有权限。真是醉了，什么也没有做，就这样说我没有权限。弹出来是这样的：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160410-2@2x-e1460288188906.png)

这是什么东西？太影响写代码的心情了！果然Xcode搞的鬼啊。

#解决方案

尝试了好几种方案的，也许其中一种可以解决！

1. 重启Xcode试试
2. 删除到DerivedData试试
3. 重启电脑试试

笔者都尝试过了，最后是解决了！

如何删除掉DerivedData？看下面的图：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ2016041011-e1460288381568.png)

在哪里打开的？

**打开方式：**点击Xcode菜单->Window->Projects->找到对应的工程!

#结尾

记录下来吧，这种坑跳过了就不希望大家也跟着跳！