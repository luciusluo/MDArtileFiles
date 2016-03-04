#前言

以下问题为朋友们问我的问题，笔者在这里统一根据个人理解和所学提供参考，如有任何疑问或出错之处，请在评论处评论或者直接联系笔者。谢谢！

#1.两个App之间如何互调，又如何传参？


**具体问题如下：**

由一个app1跳转到app2之后，app2完成某项任务之后，怎么把app2的完成信息传到app1（自己的程序是app1），传的是什么类型的数据，怎么进行解析？

**参考答案：**

笔者针对这个问题，写了一篇文章并提供demo下载：[iOS App之间如何通信](http://www.henishuo.com/ios-app-communication/)

#2.Socket在传输过程中用的是什么类型的数据：结构体还是文本？如果是文本，那么怎么进行编码和解码？


**参考答案：**

笔者学习相关socket知识，写了这几篇文章，可以参考：

[iOS Socket理论知识](http://www.henishuo.com/ios-socket-theory/)

[iOS Socket UDP编程-C语言版](http://www.henishuo.com/ios-socket-udp-c-version/)

[iOS Socket TCP编程-C语言版](http://www.henishuo.com/ios-socket-tcp-c-version/)

[iOS Socket编程-Objective-C原生API版]()

[iOS Socket编程-Objective-C基于CocoaAsyncSocket版]()

[iOS Socket编程-Swift原生API版]()

[iOS Socket编程-Swift基于AsyncSocket版]()

#3、图片压缩处理问题

**参考答案：**

关于图片的压缩处理，在ios中常用的方法是先处理像素再处理尺寸。大家可以参考笔者所写的这篇处理压缩处理文章：

[iOS 图片压缩处理](http://www.henishuo.com/ios-image-compressed/)
#4、在做图片优化处理的时候，怎么将一个原图比较小（或大）的image放到一个固定的imageview中，如何减少失真度，除了这个，图片优化还有哪些方法？

**参考答案：**

最直接的办法就是使用UIImageView的contentMode，让其自动适应，但是这样会消耗一定的性能。

如果要优化，可以通过手动将原图处理成UIImageView大小的图，再给它呈现。
#5、如何高效的对各种控件（label、button）进行切圆角、透明度、加边框（备注：不使用layer的方法）？


5.多线程在实际现实中有哪些应用？（网络操作和大量图片处理不算）6.如果app比较大，怎么样减少app的大小？7.你在迭代开发中是怎么处理版本兼容问题？8.你和前端服务器是怎么进行交互的（我说我们平常用的get、post，他说不是。。。）？9.怎么用GCD加载多张图片之后，把图片放到融合到一张图片里？10.使用GCD的时候，如何在一个group里添加几个任务的依赖关系（这几个任务放在一个组中）11.使用SDWebImage的时候，从服务器请求回来的头像URL没有变化，但是用户已经修改过头像，由于缓存的原因，不能显示出最新修改的用户的头像。在不去掉缓存的条件下，如何显示出最新的头像，给出策略。

