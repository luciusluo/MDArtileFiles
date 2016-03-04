#前言

最新整理的笔试题，由群里某某群友提供的题目，笔者整理并在此提供参考答案。

招聘高峰期来了，大家都非常积极地准备着跳槽，那么去一家公司面试就会有一堆新鲜的问题，可能不会，也可能会，但是了解不够深。本篇文章为群里的小伙伴们去某公司的笔试题，由笔者整理并提供笔者个人参考答案。注意，仅供参考，不代表绝对正确。

参考答案不唯一，大家可以根据自己的理解回答，没有必要跟笔者的一样。参考笔者的答案，也许给你带来灵感！

#题照（前五题）

![image](http://www.henishuo.com/wp-content/uploads/2016/02/3CE5311C01B70EAC002440345CFE9AE7.jpg)

#1、链表不具备的特点是（）

A. 可随机访问任何一个元素  
B. 插入，删除操作不需要移动元素  
C. 无需事先估计存储空间大小  
D. 所欲存储空间可以是不连续的  

这道题是考大学时所学的链表知识，其实笔者也忘了很多了。因为在大学时，曾经自己写过很多链表相关的代码，再加上经常帮同学调试，以及帮助同学讲解链表相关知识点，因此记忆较深。以下答案纯属个人认知，非百度而来，若有不对之处，请指出。

**参考答案：**  （A）  

链表不同于数组。链表之所有叫链表，就是像一条链一样，要过到某个节点处，就得遍历着找；而数组才具备随机访问任何一个元素的能力，数组可以通过索引直接访问元素，时间复杂度为常量，效率非常高，因此在某些场合上，我们需要数组这样的数据结构。

B. 链表的插入、删除都不需要移动元素，只需要修改指针的指向就可以了，因为链表上的每个节点都是动态分配的，分配在堆上，通过指针来指向每个节点的内存区，要获取某个节点的值，是需要遍历一遍才能找到对应的节点的。

C. 因为链表上的每个节点是分配在堆上，需要开发人员手动申请内存空间的，因此不像数组在定义时就要指定存储空间大小。对于链表，需要增加一个节点时，直接在堆上申请。当需要删除某个节点时，可以直接将该节点的内存给释放掉。

D. 因为链接中的节点都是存储在堆上的，而每个节点之间都有一个指向前一个节点和后一个节点的指针，只要知道链表头指针，就可以通过遍历查找到任何一个节点。因此，链表不同于数组，数组是要连续的内存存储空间，才能保证以常量时间复杂度快速访问任意元素；而链表不要求每个节点是连接，在堆上申请的内存空间很难得到连续的，而且空间产生内存碎片。

#2、关于多线程和多进程编程，下面描述正确的是（）

A. 多进程里，子进程可获取父进程的所有堆和栈的数据；而线程会与同进程的其他线程共享数据，拥有自己的栈空间。  
B. 线程因为有自己的独立栈空间且共享数据，所有执行的开销相对较大，同时不利于资源管理和保护。  
C. 线程的通信速度更快，切换更快，因为他们在同一地址空间内。  
D. 线程使用公共变量/内存时需要使用同步机制，因为他们在同一地址空间内。

#3、设两个变量a=19;b=29;在不创建新实例的情况下使a、b的值互换

**参考答案：**  

这道题要求不创建新的实例，只有a、b两个变量，要交换这两个变量的值，通常的做法是使用临时变量来临时存储，但是现在要求不使用新的实例，那么有什么办法呢？

方法就是通过位运算来操作：

```
a = a ^ b;
b = a ^ b;
a = a ^ b;
```

对于题目中的a = 19，也就是对应二进制**00010011**；而b=29，也就是对应二进制**00011101**

* 第一步：a = 00010011 ^ 00011101 => 00001110，将a、b的值都记录下来了
* 第二步：b = 00001110 ^ 00011101 => 00010011（值为19，也就是b得到了原来的a的值）
* 第三步：a = 00001110 ^ 00010011 => 00011101 (值为29，也就是a得到了原来的b的值)

注意，^符号表示按位异或。所谓按位异或是指对应位置上的二进制数值相同为0，不同为1。

#4、使用block时什么情况会发生引用循环，如何解决？

**参考答案：**  

笔者之前写过这篇文章讲了讲开发中常见的内存循环引用的案例：

[iOS Block循环引用精讲](http://www.henishuo.com/ios-block-memory-cycle/)


#5、为什么要序列化，对象序列化方式

**参考答案：**  

笔者也不是很确定，iOS里的序列化是指归档、JSON序列化吗？实际上归档也就是将对象转换成XML、JSON序列化也就是将对象转换data。

**将对象JSON序列化：**

```
NSLog(@"%s", __FUNCTION__);
NSDictionary *dict = @{@"key"  : @"value",
                     @"key1" : @"value1",
                     @"key2" : @"value2"};
NSData *data = [NSJSONSerialization dataWithJSONObject:dict options:NSJSONWritingPrettyPrinted error:nil];
NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
```

**将对象归档：**需要遵守NSCoding协议，实现如下方法：

```
- (void)encodeWithCoder:(NSCoder *)aCoder {
  [aCoder encodeObject:self.title forKey:@"title"];
}
```

如果所谓的序列化不是指这两种，还请高人指点。

#题照（后三题）

![image](http://www.henishuo.com/wp-content/uploads/2016/02/9C6A94935AA1D3D3CA0ACE0D56C0F904.jpg)

#7、简述如何处理UI与耗时操作的通信，有哪些方式及各自的优缺点

**参考答案：**  

* 将耗时的计算和IO操作放在子线程去处理，然后到主线程更新UI。优点是
* 采用预加载方式，将耗时操作提前处理。优点是可让UI更流畅；缺点是内存会增多，控制加载逻辑比较复杂。
* 采用延迟加载方式，将耗时操作而不立刻使用时，采用延迟加载。优点是界面可提高流畅度；缺点是在需要显示时还需要加载才能显示，需要稍稍等待。

不知道说得合不合适，还请高人提出还有哪些方式？

#8、如何优化一个TableView

**参考答案：**  

* 若高度一定，直接使用rowHeight属性而不是使用heightForRowAtIndexPath方法，以减少调用的消耗。若高度是不固定的，heightForRowAtIndexPath所计算的高度应该缓存起来，每次数据源发生变化时，比如删除、插入、更新行都会重新请求所有的高度。若有100个行，就会有调用100次，因为将高度缓存起来是应该的。同理，heightForHeaderInSection、heightForFooterInSection也应该缓存起来。

* 不要在tableView:cellForRowAtIndexPath:中做太多的计算和IO操作，比如可以将需要的计算提前计算好、IO操作也提前计算好。它应该直接调用来显示就可以。

* 将计算行高的时间提前到从服务器获取数据的时候，计算完了高度一并写回数据库或者通过转型为model，将高度放到模型中。但是，最好将高度缓存起来。若一个model的数据有不同的状态，比如展开与收起状态，应该也将高度都缓存起来。注意使用异步去计算，计算完成后再回到主线程显示。

* 在设置显示图片时，不要直接设置UIImageView的contentMode属性自动适应，图片变形会计算transform，压缩时会乘以一个矩阵，消耗性能。对于要求性能较高的app，应该将得到的图片经过处理成UIImageView大小后再呈现。

* 不要将视图的opaque属性设置为NO，默认为YES,它表示不透明度。当opque为NO的时候，图层的半透明取决于图片和其本身合成的图层为结果。

* layer添加圆角是比较耗时的，这样会离屏渲染，需要牺牲更多的性能。比如，图片显示有圆角时，可以通过core graphics来生成带圆角的图片等。

* 手动绘制cell。绘制cell不建议使用UIView，建议使用CALayer。 UIView的绘制是建立在CoreGraphic上的，其使用的是CPU。CALayer使用的是Core Animation，CPU、GPU都可以使用且由系统自动决定使用哪一个。UIView的绘制，使用的是自下向上的一层一层的绘制，而后渲染。Layer处理的是纹理，利用GPU的 Texture Cache和独立的浮点数计算单元可以加速纹理的处理。

* 重用cell。防止重复的绘制，减少渲染次数，可提高性能。

* 减少subviews的数量。尽量放在同一层view上显示。

* 尽量少动态给cell添加子view。用addView给Cell动态添加View，可以初始化时就添加，然后通过hide来控制是否显示。


想要更深入，不防看看大牛的文章吧：[iOS 保持界面流畅的技巧](http://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

#9、假设让你设计一关于指示器的开源库，请设想和设计框架的public API，并指出大概需要如何做、需要注意一些什么方面来使别人容易使用这个框架

**参考答案：**

设计API的基本准则是：

* 简单易用
* 易扩展
* 单一功能


**注意事项：**

* 尽可能不要依赖第三方库（除非不使用第三库需要非常大的工作量）。
* 每个API的功能应该是单一的，只做一件事。
* 注意性能、内存问题
* 注意多个HUD显示、关闭的切换问题
* 样式问题

**分析：**

要想让全工程使用起来非常方便，那最好的方式就是使用单例，比如SVProgressHUD就是通过单例的方式来操作的。将单例封闭在内部，外部并不知道是单例，而外部的调用全是通过类方法的形式来调用，代码调用是非常简化的。使用单例的优点是方便管理和调用。缺点就是一直占用内存而不释放。

当然，我们也可以通过正常的对象创建，在哪里使用就在哪里创建一个对象，自己来管理。这样的方式在使用的地方不是那么方便，还需要再单独进行一层封装，以方便直接调用。但这种方式的好处就是在不需要使用的时候可以释放掉；缺点就是如果同时创建了多个HUD来显示时，需要调用者使用代码逻辑来控制之前的显示与隐藏或者切换文本等。

笔者觉得，使用单例方式更方便调用一些，外部也不用通过逻辑来管理多个HUD的显示与隐藏问题，都封装到内部，由封装库的人来维护。

假设设计一个HYBProgressHUB的小例子，随手写的，没有真实写完整：

```
typedef NS_ENUM(NSUInteger, HYBProgressHUBMaskType) {
  kHYBProgressHUBClear,
  kHYBProgressHUBBlack,
  kHYBProgressHUBUserEnabled
};

@interface HYBProgressHUD : NSObject

+ (void)setDefaultMaskType:(HYBProgressHUBMaskType)defaultMaskType;

+ (void)show;
+ (void)showWithText:(NSString *)text;
+ (void)showWithImage:(NSString *)image;
+ (void)showWithText:(NSString *)text image:(UIImage *)image;

+ (void)showWithText:(NSString *)text maskType:(HYBProgressHUBMaskType)maskType;
+ (void)showWithImage:(NSString *)image  maskType:(HYBProgressHUBMaskType)maskType;
+ (void)showWithText:(NSString *)text image:(UIImage *)image  maskType:(HYBProgressHUBMaskType)maskType;

+ (void)dismiss;
+ (void)dismissWithText:(NSString *)text;
+ (void)dismissWithImage:(UIImage *)image;
+ (void)dismissWithText:(NSString *)text image:(UIImage *)image;

@end
```

将单例放在实现文件中，并没有暴露出来：

```
@implementation HYBProgressHUD

+ (instancetype)sharedInstance {
 __block HYBProgressHUD *singleTon = nil;
  
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    singleTon = [[[self class] alloc] init];
  });
  
  return singleTon;
}

@end
```

因为根据不同的需求，不同的场景，show会有很多种形式，比如不可点击、可点击、是否半透明等，因此API也应该针对单一性原则提供对应的API。但是，一个App中大部分的HUD显示的样式、方式是相同的，因此，我们可以提供一个默认的全局的呈现样式，比如调用show这个API要显示的样式就是默认的样式，若不调用，内部也会给一个默认值。

对于dismiss也一样，可以是直接隐藏，也可以在隐藏之前先提示点什么，比如纯文本提示、纯图片提示、文本和图片组合提示等。

当然，我们还可以扩展API设置dismiss的默认显示图片和文字的样式：

```
typedef NS_ENUM(NSUInteger, HYBProgressHUBDismissType) {
  kHYBProgressHUBDismissSuccess,
 kHYBProgressHUBDismissFailed,
  kHYBProgressHUBDismissWarnings,
  kHYBProgressHUBDismissTimeout
};

+ (void)dismissWithText:(NSString *)text type:(HYBProgressHUBDismissType)type;
```

比如在所有的网络请求的地方，就可以写一个通用的API，在成功与失败回调处，统一处理显示不同的类型提示。

**提示：**

上面所描述内容为笔者纯手工尝试设计的，不代表足够合理，写出来的最主要目的是抛砖引玉，当然我更希望能够有大神出来指出其中的不足或者加以补充，让后来者少走弯路。

#最后

本来不太想整理这份笔试题的，但是对于其中的几道题确实很需要认真地思考，另外也希望得到大神们的评论，指出其中的缺陷，或者能够加以补充。请各位大神们，注入新鲜的血液吧！


#参考

* [iOS 保持界面流畅的技巧](http://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
* [UITableView优化技巧](http://www.cocoachina.com/ios/20150602/11968.html)

