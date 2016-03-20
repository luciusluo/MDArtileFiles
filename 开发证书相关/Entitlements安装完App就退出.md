#前言

最近Debug真机调试经常出现这个问题：

The entitlements specified in your application’s Code Signing Entitlements file do not match ...

实际上应用已经安装到手机上了，但是突然就中断了，导致无法调试。不过手机上是可以继续使用的。

每次安装完就弹出来这样的窗口：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/屏幕快照-2016-03-17-下午12.36.41-e1458189505162.png)

#解决办法

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160317-0@2x-e1458189555709.png)

如上图，在Targets中的Info中的Build选项卡中的Code Signing Entitlements在Debug下去掉这一项的值，重新运行就可以了。

#最后

坑不能反复的跳，记录起来以防自己再跳进去！





