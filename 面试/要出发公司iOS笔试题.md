#前言

招聘高峰期来了，大家都非常积极地准备着跳槽，那么去一家公司面试就会有一堆新鲜的问题，可能不会，也可能会，但是了解不够深。本篇文章为群里的小伙伴们去**要出发**公司的笔试题，由笔者整理并提供笔者个人参考答案。注意，仅供参考，不代表绝对正确。

参考答案不唯一，大家可以根据自己的理解回答，没有必要跟笔者的一样。参考笔者的答案，也许给你带来灵感！

#题目照

![image](http://www.henishuo.com/wp-content/uploads/2016/02/goios.jpg)

#1、编程规范问题

这题看不清楚，不过可以看得出来是编程规范问题。所以呢，笔者也就没有办法说明哪些不合理了。不过笔者曾经为公司的出过一个编程规范文档，后来整理成文章GIF图分享给大家。

**参考答案：**

仅供参考：[标哥的编码规范](http://www.henishuo.com/ios-code-style/)

#2、请写出UIViewController的完整生命周期

**参考答案：**

下面是笔者通过打印，先出现ViewController，然后在点击ViewController上的按钮时，模态弹出了一个纯代码HYBViewController，其打印如下：

```
-[ViewController initWithCoder:]
-[ViewController loadView]
-[ViewController viewDidLoad]
-[ViewController viewWillAppear:]
-[ViewController viewDidAppear:]

// present HYBViewController
-[HYBViewController initWithNibName:bundle:]
-[HYBViewController init]
-[HYBViewController loadView]
-[HYBViewController viewDidLoad]
-[ViewController viewWillDisappear:]
-[HYBViewController viewWillAppear:]
-[HYBViewController viewDidAppear:]
-[ViewController viewDidDisappear:]
```

生命周期如下：

```
* xib/storyboard：-initWithCoder:，而非xib/storyboard的是-initWithNibName:bundle:然后-init
* -loadView
* -viewDidLoad
* -viewWillAppear:
* -viewDidAppear:
* -viewWillDisappear:
* -viewDidDisappear:
```

注意，当从ViewController进入到HYBViewController控制器时，注意出现顺序如下：

```
* -[ViewController viewWillDisappear:]
* -[HYBViewController viewWillAppear:]
* -[HYBViewController viewDidAppear:]
* -[ViewController viewDidDisappear:]
```

在HYBViewController完全出现后，才会调用前一个控制器的完全消失。像这种要不同控制器之间导航条隐藏与显示控制问题，就需要特别注意其生命周期的顺序。

**注意：**有朋友说这里是错的，不过笔者打印出来验证发现就是这样的顺序！用模态呈现测试!截图如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/vcprint-e1458225641298.png)

请大家看看哪里出错！

#请写出有多少有方法给UIImageView添加圆角？

**参考答案：**

1. 最直接的方法就是使用如下属性设置：

```
imgView.layer.cornerRadius = 10;
// 这一行代码是很消耗性能的
imgView.clipsToBounds = YES;
```

好处是使用简单，操作方便。坏处是离屏渲染（off-screen-rendering）需要消耗性能。对于图片比较多的视图上，不建议使用这种方法来设置圆角。通常来说，计算机系统中CPU、GPU、显示器是协同工作的。CPU计算好显示内容提交到GPU，GPU渲染完成后将渲染结果放入帧缓冲区。

简单来说，离屏渲染，导致本该GPU干的活，结果交给了CPU来干，而CPU又不擅长GPU干的活，于是拖慢了UI层的FPS（数据帧率），并且离屏需要创建新的缓冲区和上下文切换，因此消耗较大的性能。

2. 给UIImage添加生成圆角图片的扩展API：

```
- (UIImage *)hyb_imageWithCornerRadius:(CGFloat)radius {
  CGRect rect = (CGRect){0.f, 0.f, self.size};
  
  UIGraphicsBeginImageContextWithOptions(self.size, NO, UIScreen.mainScreen.scale);
  CGContextAddPath(UIGraphicsGetCurrentContext(),
                   [UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:radius].CGPath);
  CGContextClip(UIGraphicsGetCurrentContext());
  
  [self drawInRect:rect];
  UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
  
  UIGraphicsEndImageContext();
  
  return image;
}
```

然后调用时就直接传一个圆角来处理：

```
imgView.image = [[UIImage imageNamed:@"test"] hyb_imageWithCornerRadius:4];
```

这么做就是on-screen-rendering了，通过模拟器->debug->Color Off-screen-rendering看到没有离屏渲染了!（黄色的小圆角没有显示了，说明这个不是离屏渲染了）


3. 在画之前先通过UIBezierPath添加裁剪，但是这种不实用：

```
- (void)drawRect:(CGRect)rect {
  CGRect bounds = self.bounds;
  [[UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:8.0] addClip];

  [self.image drawInRect:bounds];
}
```

4. 通过mask遮罩实现

这里不细说了，个人感觉不如第二种好用、通用

若有更多，可以在评论中指出~

#4、请描述事件响应者链的工作原理

**参考答案：**

iOS使用hit-testing寻找触摸的view。 Hit-Testing通过检查触摸点是否在关联的view边界内，如果在，则递归地检查该view的所有子view。在层级上处于lowest（就是离用户最近的view）且边界范围包含触摸点的view成为hit-test view。确定hit-test view后，它传递触摸事件给该view。

官方小例子事件响应者链如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/hit_testing_2x.png)

* 触摸点在view A中，所以要先检查子view B和C。
* 触摸点不在view B中，但在C中，所以检查C的子view D和E。
* 触摸点不在D中，但在E中。View E是这个层级上处于lowest的view的边界范围包含触摸点，所以它成为了hit-test view。

Hit-test view是处理触摸事件的第一选择，如果hit-test view不能处理事件，该事件将从事件响应链中寻找响应器，直到系统找到一个处理事件的对象。若不能处理，则就有事件传递链了，继续看下面的事件传递链。


事件传递链如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/iOS_responder_chain.png)

左半图:

* initial view若不能处理事件，则传到其父视图view
* view若不能处理，则传到其父视图，因为它还不是最上层视图
* 这里view的父视图是view controller的view，因为这个view也不能处理事件，因此传给view controller
* 若view controller也不能处理此事件，则传到window
* 若window也不能处理此事件，则传到app单例对象Application
* 若UIApplication单例对象也不能处理，则表示无效事件

右半图：

* initial view一直传递直到最上层view(原话：A view passes an event up its view controller’s view hierarchy until it reaches the topmost view.)
* topmost view传递事件到它所在的控制器（原话：The topmost view passes the event to its view controller.）
* view controller传递事件到topmost view的父视图，重复前三步，走到到达root controller（原话：passes the event to its topmost view’s superview.
Steps 1-3 repeat until the event reaches the root view controller.）

* 由root控制器传递事件到window（原话：The root view controller passes the event to the window object.）
* 若window也不能处理此事件，则传到app单例对象Application
* 若UIApplication单例对象也不能处理，则表示无效事件


为了解答这个小题目，翻阅了官方文档，由于内容较多，这里不说那么多，若要了解更多，参考官方文档吧：[Event Handling Guide for iOS](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW2)

#5、如何避免使用block时发生循环引用

**参考答案：**

关于block循环引用问题是非常常见的，但是很多人没有深入研究过，xcode没有提示警告就以为没有形成循环引用了。笔者也见过很多高级iOS开发工程师的同事，使用block并不会分析是否形成循环引用。

推荐阅读[iOS Block循环引用精讲](http://www.henishuo.com/ios-block-memory-cycle/)

#6、请比较GCD与NSOperation的异同

**参考答案：**

* 相同点：GCD和NSOperation都是苹果提供的多线程实现方案。
* 不同点：GCD是轻量级的纯C写的多线程实现方案，使用起来非常方便，在开发中大量使用，但是对于取消和暂停线程就比较麻烦些。而NSOperation是面向对象的，兼容KVO，对于取消和暂停任务是非常容易的。

更详细地，推荐阅读：[iOS图解多线程](http://www.henishuo.com/ios-multithread-detail/)

#7、请写出NSTimer使用时的注意事项（两项即可）

说到NSTimer这个定时器类，要使用好它，还得了解Run Loop，因为在不同的run loop mode下，定时器不都会回调的。

mode主要是用来指定事件在运行循环中的优先级的，分为：

* NSDefaultRunLoopMode（kCFRunLoopDefaultMode）：默认，空闲状态
* UITrackingRunLoopMode：ScrollView滑动时会切换到该Mode
* UIInitializationRunLoopMode：run loop启动时，会切换到该mode
* NSRunLoopCommonModes（kCFRunLoopCommonModes）：Mode集合

苹果公开提供的Mode有两个：

* NSDefaultRunLoopMode（kCFRunLoopDefaultMode）
* NSRunLoopCommonModes（kCFRunLoopCommonModes）

如果我们把一个NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环中的时候, ScrollView滚动过程中会因为mode的切换，而导致NSTimer将不再被调度。当我们滚动的时候，也希望不调度，那就应该使用默认模式。但是，如果希望在滚动时，定时器也要回调，那就应该使用common mode。

如果想更深入地了解RunLoop，请参考[iOS之Run Loop详解](http://www.henishuo.com/ios-runloop-in-detail/)

如果想要销毁timer，则必须先将timer置为失效，否则timer就一直占用内存而不会释放。造成逻辑上的内存泄漏。该泄漏不能用xcode及instruments测出来。
另外对于要求必须销毁timer的逻辑处理，未将timer置为失效，若每次都创建一次，则之前的不能得到释放，则会同时存在多个timer的实例在内存中。

**参考答案：**

* 注意timer添加到runloop时应该设置为什么mode
* 注意timer在不需要时，一定要调用invalidate方法使定时器失效，否则得不到释放

#8、说说Core Animation是如何开始和结束动画的

笔者不是很清楚题目的真正要求，是想知道核心动画的哪些知识点。如何开始和结束动画，这核心动画有很多种，每种动画还有很大的区别。

**参考答案：**

动画的开始和结束都可以通过CAMediaTiming协议来处理，核心动画的基类是遵守了CAMediaTiming协议的，可以指定动画开始时间、动画时长、动画播放速度、动画在完成时的行为（停留在结束处、动画回到开始处、动画完成时移除动画）。

请参考笔者以前给公司所有团队分享的知识点：[说说Core Animation](http://www.henishuo.com/core-animation/)

#最后

学习完这份笔试题的相关知识点，再整理成本篇文章，真心是累！且看且珍惜吧！

若支持笔者，不防打赏表示表示~

又一个夜深人静了~风儿挺大的！打一把梦三准备sleep~



#原文

[阅读原文](http://www.henishuo.com/ios-needgo-interview/)

