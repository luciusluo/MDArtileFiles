>#iOS使用present控制器时要过好一会才出现

---
最近很多朋友老问为什么为什么为什么使用模态呈现控制器的时候出现卡频？什么是卡频，笔者也不明白你说的是什么，太不专业了。

其实这只是停顿一会才出现而已。

#解决方案

在调用的地方，增加以下代码：

```
dispatch_async(dispatch_get_main_queue(), ^{
	// 做你要做的事
});
```


#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/)**



