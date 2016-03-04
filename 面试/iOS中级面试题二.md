#前言

如果没有阅读过前一份中级面试题【[iOS中级面试题一](http://www.henishuo.com/ios-middle-interview-questions-one/)】，可以先阅读之。

本篇文章中所收集的面试题大部分来源来网络，小部分为笔者根据个人经验所出的面试题。所有题目都会提供参考答案，但是参考答案仅供参考，不保证一定正确，且答案不是唯一的，不同的面试官的知识面和技术水平不同也会出现不同的意见。

#1、iOS数据持久化存储方案有哪些？

**参考答案：**

* plist属性列表存储（如NSUserDefaults）
* 文件存储（如二进制数据写入文件存储，通过NSFileManager来操作将下载起来的二进制数据写一篇文件中存储）
* NSKeydeArchiver归档存储，常见的是自动化归档/解档处理，想要学习如何通过runtime实现自动化归档/解档，可以阅读文章：[学习通过runtime实现自动化归档/解档](http://www.henishuo.com/runtime-archive-unarchive-automaticly/)
* 数据库SQLite3存储（如FMDB、Core Data）


#2、沙盒的目录结构是怎样的？各自一般用于什么场合？

**参考答案：**

- Application：存放程序源文件，上架前经过数字签名，上架后不可修改
- Documents: 保存应⽤运行时生成的需要持久化的数据,iTunes同步设备时会备份该目        录。例如,游戏应用可将游戏存档保存在该目录
- tmp: 保存应⽤运行时所需的临时数据,使⽤完毕后再将相应的文件从该目录删除。应用        没有运行时,系统也可能会清除该目录下的文件。iTunes同步设备时 不会备份该目录 
- Library/Caches: 保存应用运行时⽣成的需要持久化的数据,iTunes同步设备时不会备份        该目录。⼀一般存储体积大、不需要备份的非重要数据，比如网络数据缓存存储到Caches下 
- Library/Preference: 保存应用的所有偏好设置，如iOS的Settings(设置) 应⽤会在该目录中查找应⽤的设置信息。iTunes同步设备时会备份该目录

#3、#define定义的宏和const定义的常量有什么区别？

**参考答案：**

* \#define定义宏的指令，程序在预处理阶段将用\#define所定义的内容只是进行了替换。因此程序运行时，常量表中并没有用\#define所定义的宏，系统并不为它分配内存，而且在编译时不会检查数据类型，出错的概率要大一些。
* const定义的常量，在程序运行时是存放在常量表中，系统会为它分配内存，而且在编译时会进行类型检查。
* \#define定义表达式时要注意“边缘效应”，例如如下定义：

```
#define N 2 + 3 // 我们预想的N值是5，我们这样使用N
int a = N / 2;  // 我们预想的a的值是2.5，可实际上a的值是3.5
```

#4、常见的出现内存循环引用的场景有哪些？

**参考答案：**

* 定时器（NSTimer）：NSTimer经常会被作为某个类的成员变量，而NSTimer初始化时要指定self为target，容易造成循环引用（self->timer->self）。 另外，若timer一直处于validate的状态，则其引用计数将始终大于0，因此在不再使用定时器以后，应该先调用invalidate方法
* block的使用：block在copy时都会对block内部用到的对象进行强引用(ARC)或者retainCount增1(非ARC)。在ARC与非ARC环境下对block使用不当都会引起循环引用问题，
一般表现为，某个类将block作为自己的属性变量，然后该类在block的方法体里面又使用了该类本身，简单说就是self.someBlock = ^(Type var){[self dosomething];或者self.otherVar = XXX;或者\_otherVar = ...};出现循环的原因是：self->block->self或者self->block->\_ivar（成员变量）
* 代理（delegate）：在委托问题上出现循环引用问题已经是老生常谈了，规避该问题的杀手锏也是简单到哭，一字诀：声明delegate时请用assign(MRC)或者weak(ARC)，千万别手贱玩一下retain或者strong，毕竟这基本逃不掉循环引用了！

#5、block中的weak self，是任何时候都需要加的么？

**参考答案：**

不是什么任何时候都需要添加的，不过任何时候都添加似乎总是好的。只要出现像self->block->self.property/self->_ivar这样的结构链时，才会出现循环引用问题。好好分析一下，就可以推断出是否会有循环引用问题。

#6、GCD的queue、main queue中执行的代码一定是在main thread么？

**参考答案：**

* 对于queue中所执行的代码不一定在main thread中。如果queue是在主线程中创建的，那么所执行的代码就是在主线程中执行。如果是在子线程中创建的，那么就不会在main thread中执行。
* 对于main queue就是在主线程中的，因此一定会在主线程中执行。获取main queue就可以了，不需要我们创建，获取方式通过调用方法dispatch_get_main_queue来获取。

#7、头文件中声明的成员变量（不是属性），外部可直接访问么？

**参考答案：**

外部不能直接访问头文件所声明的成员变量，需要提供成员变量的getter方法才能在外部访问。而属性已经直接给我们自动生成了getter方法，因此外部可以直接访问属性。

#8、TCP和UDP的区别是什么？

**参考答案：**

* TCP：面向连接、传输可靠（保证数据正确性，保证数据顺序传输）、用于传输大量数据(流模式)、速度慢，建立连接需要开销较多(时间，系统资源)。
* UDP：面向非连接、传输不可靠、用于传输少量数据(数据包模式)、速度快，传输的是报文。

#9、MD5和Base64的区别是什么，各自使用场景是什么？

**参考答案：**

做过加密相关的功能的，几乎都会使用到MD5和Base64，它们两者在实际开发中是最常用的。

* MD5：是一种不可逆的摘要算法，用于生成摘要，无法逆着破解得到原文。常用的是生成32位摘要，用于验证数据的有效性。比如，在网络请求接口中，通过将所有的参数生成摘要，客户端和服务端采用同样的规则生成摘要，这样可以防篡改。又如，下载文件时，通过生成文件的摘要，用于验证文件是否损坏。
* Base64：属于加密算法，是可逆的，经过encode后，可以decode得到原文。在开发中，有的公司上传图片采用的是将图片转换成base64字符串，再上传。在做加密相关的功能时，通常会将数据进行base64加密/解密。

#10、发送10个网络请求，然后再接收到所有回应之后执行后续操作，如何实现？


**参考答案：**

从题目分析可知，10个请求要全部完成后，才执行某一功能。比如，下载10图片后合成一张大图，就需要异步全部下载完成后，才能合并成大图。

做法：通过dispatch\_group\_t来实现，将每个请求放入到Group中，将合并成大图的操作放在dispatch\_group\_notify中实现。

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{ /*加载图片1 */ });
dispatch_group_async(group, queue, ^{ /*加载图片2 */ });
dispatch_group_async(group, queue, ^{ /*加载图片3 */ }); 
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    // 合并图片
});
```

#11、\_\_block和\_\_weak修饰符的区别？

**参考答案：**

* \_\_block不管是ARC还是MRC模式下都可以使用，它的作用是标识block外的变量在block内使用可以修改变量的值。
* \_\_weak只能在ARC模式下使用，表示弱引用，主要是用于防止循环引用而引入的。

#12、常见的Http状态码有哪些？

**参考答案：**

* 302是请求重定向。
* 500及以上是服务器错误，如503表示服务器找不到、3840表示服务器返回无效JSON。
* 400及以上是请求链接错误或者找不到服务器，如常见的404。
* 200及以上是正确，如常见的是200表示请求正常。
* 100及以上是请求接受成功。

更详细请阅读：[Http状态码详细说明](http://tool.oschina.net/commons?type=5)

#13、iOS与H5交互的方式有哪些？

**参考答案：**

* 通过协议在shouldStartRequest中捕获请求，获取scheme来判断预先定义的功能，然后调用native代码。比如，定义点击图片调用native来展示大图，那么js接收到点击时，重定向将图片的url添加上自定义scheme，如HYBImagePreview://这样。更详细请阅读我的文章：[Swift版](http://www.henishuo.com/html-img-preview/)、[Objective-C版本](http://www.henishuo.com/objc-html-img-preview/)
* 通过iOS7以后引入的JavascriptCore框架来实现，通过注入对象的方式来实现，维护起来更简单。更详细请阅读我的文章：[Oc版本与JS交互](http://www.henishuo.com/oc-js/)、[Swift版与JS交互](http://www.henishuo.com/swift-js/)、如何使用的是WKWebView，可阅读[WKWebView与JS交互](http://www.henishuo.com/wkwebview-js/)
* 通过WebViewJavascriptBridge这个第三方库来实现，具体百度吧！

#14、如何浅拷贝与深拷贝的理解？

**参考答案：**

所谓浅拷贝是指只复制指向对象的指针，而不复制引用对象本身（同一份内存），而所谓深拷贝是指复制引用对象本身（新创建了一份内存）。

打个比方：

浅复制好比是你的影子，当你挂了，影子也消失了。
深复制好比克隆你，即使你挂了，新克隆出来的人是新的生命，不会因为你挂了也跟着挂。

#15、使用是懒加载？常见场景？

**参考答案：**

所谓懒加载是指在真正需要到的时候才真正载入内存。常见的场景就是重写属性的getter方法，在getter方法中判断是否创建过或者加载过，若未创建过或者未加载过，则创建或者加载。

懒加载可以优化性能，有的功能一般情况下不会使用到，但是这些功能一使用就会占较大的内存，此时使用懒加载就非常的好了。

常见的写法：

```
- (NSMutableArray *)dataSource {
  if (_dataSource == nil) {
    _dataSource = [[NSMutableArray alloc] init];
    // 添加默认数据等
    // ...
  }
}
```

#16、一个tableView是否可以关联两个不同的数据源？

**参考答案：**

当然是可以关联多个不同的数据源，但是不同同时使用多个数据源而已。比如，一个列表有两个筛选功能，一个是筛选城市，一个是筛选时间，那么这两个就是两个数据源了。当筛选城市时，就会使用城市数据源；当筛选时间时，就会使用时间数据源。

#17、对象添加到通知中心中，当通知中心发通知时，这个对象却已经被释放了，可能会出现什么问题？

**参考答案：**

其实这种只是考查对通知的简单应用。通知是多对多的关系，主要使用场景是跨模块传值。当某对象加入到通知中心后，若在对象被销毁前不将该对象从通知中心中移除，当发送通知时，就会造成崩溃。这是很常见的。所以，在添加到通知中心后，一定要在释放前移除。

更详细请阅读笔者的文章：[iOS中的NSNotificationCenter](http://www.henishuo.com/ios-notification/)



#18、实现过框架或者库以供他人使用么？如果有，请谈一谈构建框架或者库时候的经验；如果没有，请设想和设计框架的public的API，并指出大概需要如何做、需要注意哪些问题，以使人人更容易地使用你的框架。

**参考答案：**

实现过的。本人开源过多个组件的，可到GITHUB查找：[标哥的GITHUB开源组件库](https://github.com/CoderJackyHuang)，每个开源组件都有详细的博文说明如何使用哦。当然，在公司也设计了相关的公共框架，不可公开！从以下角度出发来思考和设计公共框架：

* 确保外部调用简单，且保证有详细的头文件注释说明。
* 确保API编码规范，保证风格统一。
* 确保API易扩展，可以考虑预留参数
* 确保没有外部依赖或者依赖要尽可能的少，以保证公共库的纯洁（原则上不能有外部依赖）
* 确保易维护，不存在冗余API

#19、如何自动计算cell的高度？

**参考答案：**

笔者喜欢纯代码自动布局，一直使用Masonry这个第三方库来实现纯代码自动布局的，使用起来非常简单，而且效率也很高。开发起来，提高了开发效率。

关于Masonry自动计算行高，笔者提供了swift版和oc版本的扩展，这两个版本都提供了自动计算行高的功能，并且带有缓存功能，保证永远只计算一次行高，效率就会很高，一般的应用也就不会卡屏了。

学习[OC版本自动计算行高扩展](http://www.henishuo.com/masonry-cell-height-auto-calculate/)、[swift版自动计算行高扩展](http://www.henishuo.com/snapkit-auto-cell-height/)

实现原理：通过数据模型的id作为key，以确保唯一，如何才能保证复用cell时不会出现混乱。在配置完数据后，通过更新约束，得到最后一个控件的frame，就只可以判断cell实际需要的高度，并且缓存下来，下次再获取时，判断是否存在，若存在则直接返回。因此，只会计算一遍。

#20、UITableView是如何计算内容高度的？为什么初始化时配置数据时，获取行高的代理方法会调用数据条数次？

**参考答案：**

UITableView是继承于UIScrollView的，因此也有contentSize。要得到tableview的contentsize，就需要得到所有cell的高度，从而计算出总高度，才能得到contentsize。因此，在reloadData时，就会调用该代理方法数据条数次。

为了提高效率，笔者写了扩展用于自动计算行高的，并且带有缓存，以保证只会计算一次，防止卡屏。做到这一点，一般的应用就可以解决卡屏的问题了。对于富文本比较多的应用，还可以继续优化哦。

学习[OC版本自动计算行高扩展](http://www.henishuo.com/masonry-cell-height-auto-calculate/)、[swift版自动计算行高扩展](http://www.henishuo.com/snapkit-auto-cell-height/)

#关注我


关注                | 账号              | 备注
-------------      | -------------     | ----------------
Swift/ObjC技术群一  | 324400294         |  群一若已满，请申请群二
Swift/ObjC技术群二  | 494669518         | 群二若已满，请申请群三
Swift/ObjC技术群三  | 461252383         | 群三若已满，会有提示信息
关注微信公众号       | iOSDevShares      | 关注微信公众号，会定期地推送好文章
关注新浪微博账号      |  [标哥Jacky](http://weibo.com/u/5384637337) | 关注微博，每次发布文章都会分享到新浪微博
关注标哥的GitHub     | [CoderJackyHuang](https://github.com/CoderJackyHuang) | 这里有很多的Demo和开源组件
关于我               | [进一步了解标哥](http://www.henishuo.com/about-biaoge/) | 如果觉得文章对您很有帮助，可捐助我！


