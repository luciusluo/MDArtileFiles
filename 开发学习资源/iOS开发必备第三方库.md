#引言

作为iOS开发人员，在开发App的过程中怎么会不使用第三方库呢？相信没有App是不使用第三方库的！相信自己在所做过的App中使用过哪些第三方库呢？

##网络库

说到网络库，这是开发必备，除非你的App玩单机。现在特别火也特别好用的网络库就数AFNetworking了。也有部分同志喜欢自己用原生的NSURLConnection来做，当然是可以的，只是要写起来也困难，维护起来更困难。

笔者基于AFNetworking封装了一个网络常用API类，猛点阅读：[开源HYBNetworking基于AFN封装网络](http://www.henishuo.com/base-on-afnetworking-wrapper/)

##数据库

不是每个App都使用到数据库的，但是如果需要使用到数据库，我们还是需要使用第三方的。因为原来的CoreData真不好用。当前刚接触的时候，FMDB是特别火的，当然现在仍然有很多公司在使用，包括我们公司。

这里有一篇CSDN文章说得不错：[http://blog.csdn.net/xyz_lmn/article/details/9312837](http://blog.csdn.net/xyz_lmn/article/details/9312837)

##模型与字典互转/自动归档

从我开发公司的App以来，一直在寻找JSON与Model互转的第三方库，因为每次网络取回数据后再一个个解析取出来真的很麻烦很累。这里自然极力推荐的库就是MJExtension。

这个库简单易用，直接看一看文档就明白怎么用了：[https://github.com/CoderMJLee/MJExtension](https://github.com/CoderMJLee/MJExtension)

当然，后来出了个YYModel，笔者研究了一下，其实与MJExtension差不多，只是YYModel大部分都使用runtime最底层API，而MJExtension更多的是OC语法。在性能上，据说YYModel要比MJExtension要高，当然从原理上来分析应该会高一些。

想试试YYModel？试试吧：[YYModel](https://github.com/ibireme/YYModel)


##图片下载

现在很多公司所开发的App中使用了SDWebImage，但是个人觉得使用AFNetworking这套网络库就可以了，这套库已经提供了对图片的下载和高效缓存。如果喜欢使用SDWebImage，可查看一下源代码及使用文档：

[著名SDWebImage](https://github.com/rs/SDWebImage)

事实上，本人现在直接使用AFNetworking的图片下载及缓存功能，无须再添加一个三方库。

##提示HUD

说到这个提示HUD，很多人都非常喜欢MBProgressHUD，其下载地址：[MBProggressHUD](https://github.com/jdg/MBProgressHUD)

但是，本人不太喜欢它，因为使用起来很麻烦。本人更推荐的是SVProgressHUD，以单例形式存活，任何时候直接调用，而且我们需要调用的api都是类方法，直接调用即可。其下载地址为：[SVProgressHUD](https://github.com/TransitApp/SVProgressHUD)

#自动布局

对于开发是使用xib/storybard的同学可跳过。这里介绍的是纯代码的自动布局，原生的代码自动布局是相当困难的，写起来很麻烦而且也很难记住。因此，我们需要一个第三方库对原生的约束api封装成简单易用的接口给我们使用。

这里本人极力推荐Masonry，其下载地址为：[著名自动布局Masonry](https://github.com/SnapKit/Masonry)

扩展了自动计算行高：[开源HYBMasonryAutoCellHeight](http://www.henishuo.com/masonry-cell-height-auto-calculate/)

如果是swift开发，推荐SnapKit，另外笔者基于SnapKit扩展了一个自动计算行高：[HYBSnapkitAutoCellHeight开源自动算行高Swift版](http://www.henishuo.com/snapkit-auto-cell-height/)

不会用Masonry？看看笔者的14篇教程吧：[Masonry纯代码自动布局实战](http://www.henishuo.com/category/autolayout/)

##侧滑菜单

对于使用侧滑风格的app，可使用MMDrawerController这套库，几行代码就可以实现了。其下载地址为：[https://github.com/mutualmobile/MMDrawerController](https://github.com/mutualmobile/MMDrawerController)

##CoverFlow效果

我想最有名的CoverFlow效果的第三方库就是iCarousel了。其下载地址：[https://github.com/nicklockwood/iCarousel](https://github.com/nicklockwood/iCarousel)

##日志

开发App怎么能没有日志呢？没有日志，如何去查看记录？现在特别火的日志库是CocoaLumberjack，其下载地址：[https://github.com/CocoaLumberjack/CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack)

##刷新

到目前为止，很多公司的App都采用了MJRefresh这个快速集成下拉刷新和上拉加载更多功能的库。这个库还支持自定义样式，因此可根据需求定制风格。其下载地址：[https://github.com/CoderMJLee/MJRefresh](https://github.com/CoderMJLee/MJRefresh)

##模糊效果

iOS7以后就有UIVisualEffect这个控件支持模糊效果。如果要支持iOS5.0以上版本，那就需要第三方库来支持了。支持静态、动态模糊效果，继承与UIView的模糊特效的FXBlurView就能满足我们的需求。其下载地址：[https://github.com/nicklockwood/FXBlurView](https://github.com/nicklockwood/FXBlurView)

##富文本
 

文字视图开源组件，是UILabel的替代元件，可以简单的方式展现渲染的属性字符串。另外，还支持链接，不管是手动还是使用UIDataDetectorTypes自动把电话号码、事件、地址以及其他信息变成链接。其下载地址：[https://github.com/mattt/TTTAttributedLabel](https://github.com/mattt/TTTAttributedLabel)

##TabBarController

RDVTabBarController可以方便设置底部菜单的文字图片，点击效果，小红点提示等等，但是没有原生的UITabBar过渡效果，因此笔者不是很喜欢。其下载地址：[https://github.com/robbdimitrov/RDVTabBarController](https://github.com/robbdimitrov/RDVTabBarController)

##福利

最近看到这有一篇文章收集了很全的第三方库，上边所推荐都是本人所用。点这里看更多第三方库：[http://www.52codes.net/article/465.html](http://www.52codes.net/article/465.html)

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**原文有更新，阅读原文：**[http://www.henishuo.com/ios-thirdparty/](http://www.henishuo.com/ios-thirdparty/)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
