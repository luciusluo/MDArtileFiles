>#ios开发必备第三方库



#引言

---
作为`iOS`开发人员，在开发`App`的过程中怎么会不使用第三方库呢？相信没有`App`是不使用第三方库的！相信自己在所做过的`App`中使用过哪些第三方库呢？

#网络库

---
说到网络库，这是开发必备，除非你的`App`玩单机。现在特别火也特别好用的网络库就数`AFNetworking`了。也有部分同志喜欢自己用原生的`NSURLConnection`来做，当然是可以的，只是要写起来也困难，维护起来更困难。

**猛点阅读**：[基于AFNetworking封装](http://www.henishuo.com/base-on-afnetworking-wrapper/)

#数据库

---
不是每个`App`都使用到数据库的，但是如果需要使用到数据库，我们还是需要使用第三方的。因为原来的`CoreData`真不好用。当前刚接触的时候，`FMDB`是特别火的，当然现在仍然有很多公司在使用，包括我们公司。

这里有一篇CSDN文章说得不错：[http://blog.csdn.net/xyz_lmn/article/details/9312837](http://blog.csdn.net/xyz_lmn/article/details/9312837)

#JSON与Model互转

---
从我开发公司的`App`以来，一直在寻找`JSON`与`Model`互转的第三方库，因为每次网络取回数据后再一个个解析取出来真的很麻烦很累。这里自然极力推荐的库就是`MJExtension`。

这个库简单易用，直接看一看文档就明白怎么用了：[https://github.com/CoderMJLee/MJExtension](https://github.com/CoderMJLee/MJExtension)

#图片下载

---
现在很多公司所开发的`App`中使用了`SDWebImage`，但是个人觉得使用`AFNetworking`这套网络库就可以了，这套库已经提供了对图片的下载和高效缓存。如果喜欢使用`SDWebImage`，可查看一下源代码及使用文档：[https://github.com/rs/SDWebImage](https://github.com/rs/SDWebImage)

事实上，本人现在直接使用`AFNetworking`的图片下载及缓存功能，无须再添加一个三方库。

#提示HUD

---
说到这个提示`HUD`，很多人都非常喜欢`MBProgressHUD`，其下载地址：[https://github.com/jdg/MBProgressHUD](https://github.com/jdg/MBProgressHUD)

但是，本人不太喜欢它，因为使用起来很麻烦。本人更推荐的是`SVProgressHUD`，以单例形式存活，任何时候直接调用，而且我们需要调用的`api`都是类方法，直接调用即可。其下载地址为：[https://github.com/TransitApp/SVProgressHUD](https://github.com/TransitApp/SVProgressHUD)

#自动布局

---
对于开发是使用`xib/storybard`的同学可跳过。这里介绍的是纯代码的自动布局，原生的代码自动布局是相当困难的，写起来很麻烦而且也很难记住。因此，我们需要一个第三方库对原生的约束`api`封装成简单易用的接口给我们使用。

这里本人极力推荐`Masonry`，其下载地址为：[https://github.com/SnapKit/Masonry](https://github.com/SnapKit/Masonry)

#侧滑菜单

---
对于使用侧滑风格的`app`，可使用`MMDrawerController`这套库，几行代码就可以实现了。其下载地址为：[https://github.com/mutualmobile/MMDrawerController](https://github.com/mutualmobile/MMDrawerController)

#CoverFlow效果

---
我想最有名的`CoverFlow`效果的第三方库就是`iCarousel`了。其下载地址：[https://github.com/nicklockwood/iCarousel](https://github.com/nicklockwood/iCarousel)

#日志

---
开发`App`怎么能没有日志呢？没有日志，如何去查看记录？现在特别火的日志库是`CocoaLumberjack `，其下载地址：[https://github.com/CocoaLumberjack/CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack)

#刷新

---
到目前为止，很多公司的`App`都采用了`MJRefresh`这个快速集成下拉刷新和上拉加载更多功能的库。这个库还支持自定义样式，因此可根据需求定制风格。其下载地址：[https://github.com/CoderMJLee/MJRefresh](https://github.com/CoderMJLee/MJRefresh)

#模糊效果

---
`iOS7`以后就有`UIVisualEffect`这个控件支持模糊效果。如果要支持iOS5.0以上版本，那就需要第三方库来支持了。支持静态、动态模糊效果，继承与UIView的模糊特效的`FXBlurView`就能满足我们的需求。其下载地址：[https://github.com/nicklockwood/FXBlurView](https://github.com/nicklockwood/FXBlurView)

#富文本

--
文字视图开源组件，是`UILabel`的替代元件，可以简单的方式展现渲染的属性字符串。另外，还支持链接，不管是手动还是使用`UIDataDetectorTypes`自动把电话号码、事件、地址以及其他信息变成链接。其下载地址：[https://github.com/mattt/TTTAttributedLabel](https://github.com/mattt/TTTAttributedLabel)

#TabBarController

---
`RDVTabBarController`可以方便设置底部菜单的文字图片，点击效果，小红点提示等等，但是没有原生的`UITabBar`过渡效果，因此笔者不是很喜欢。其下载地址：[https://github.com/robbdimitrov/RDVTabBarController](https://github.com/robbdimitrov/RDVTabBarController)

#福利

---
最近看到这有一篇文章收集了很全的第三方库，上边所推荐都是本人所用。点这里看更多第三方库：[http://www.52codes.net/article/465.html](http://www.52codes.net/article/465.html)

#关注我

---
如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：324400294**<br>
关注微信公众号：[**iOSDevShares**]()

