#前言

以下题目来自[招聘一个靠谱的iOS](https://github.com/ChenYilong/iOSInterviewQuestions/blob/master/01%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88/%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88%EF%BC%88%E4%B8%8B%EF%BC%89.md#25-_objc_msgforward%E5%87%BD%E6%95%B0%E6%98%AF%E5%81%9A%E4%BB%80%E4%B9%88%E7%9A%84%E7%9B%B4%E6%8E%A5%E8%B0%83%E7%94%A8%E5%AE%83%E5%B0%86%E4%BC%9A%E5%8F%91%E7%94%9F%E4%BB%80%E4%B9%88)，笔者整理了大部分，并将个人的理解和想法写出来，可能存在于原作者的参考答案不同的地方，还请大家各抒己见！

所有参考答案不代表一定正确，但是都会有明确的来源说明及参考答案。若与您的想法存在差异，请在评论中提出，或者加群交流，得到统一意见后，笔者会更新文章内容！

如果未阅读过上半部分，请先阅读[招聘一个靠谱的iOS（上）](http://www.henishuo.com/interview-part-one/)

#1、runloop和线程有什么关系？

正如其名，loop表示某种循环，和run放在一起就表示一直在运行着的循环。实际上，run loop和线程是紧密相连的，可以这样说run loop是为了线程而生，没有线程，它就没有存在的必要。Run loop是线程的基础架构部分， Cocoa 和 CoreFundation都提供了方便配置和管理线程的 run loop （以下都以 Cocoa 为例）。每个线程，包括程序的主线程都有与之相应的run loop。

**参考答案：**

* 主线程的run loop默认是启动的。iOS的应用程序里面，程序启动后会有一个如下的main()函数

```
int main(int argc, char * argv[]) {
  @autoreleasepool {
    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
  }
}
```

重点是UIApplicationMain()函数，这个方法会为main thread设置一个NSRunLoop对象，这就解释了：为什么我们的应用可以在无人操作的时候休息，需要让它干活的时候又能立马响应。

* 对非主线程来说，run loop默认是没有启动的，确切地说，当没有访问过非主线程的run loop时，run loop是不存在的，因为这是一种懒加载。如果你需要做更多的线程交互则可以手动配置和启动，如果线程只是去执行一个长时间的已确定的任务则不需要。

* 在任何一个Cocoa程序的线程中，都可以通过以下代码来获取到当前线程的run loop：

```
NSRunLoop *runloop = [NSRunLoop currentRunLoop];
```

如果想更深入地了解RunLoop，请参考[iOS之Run Loop详解](http://www.henishuo.com/ios-runloop-in-detail/)


#2、RunLoop的mode作用是什么？

**参考答案：**

mode主要是用来指定事件在运行循环中什么状态接收事件，也就是通过mode可以过滤不需要的状态所传过来的事件，分为：

* NSDefaultRunLoopMode（kCFRunLoopDefaultMode）：默认，空闲状态
* UITrackingRunLoopMode：ScrollView滑动时会切换到该Mode
* UIInitializationRunLoopMode：run loop启动时，会切换到该mode
* NSRunLoopCommonModes（kCFRunLoopCommonModes）：Mode集合

苹果公开提供的Mode有两个：

* NSDefaultRunLoopMode（kCFRunLoopDefaultMode）
* NSRunLoopCommonModes（kCFRunLoopCommonModes）

如果我们把一个NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环中的时候, ScrollView滚动过程中会因为mode的切换，而导致NSTimer将不会被调度。当我们滚动的时候，若希望不调度，那就应该使用默认模式。但是，如果希望在滚动时，定时器也要回调，那就应该使用common modes。

如果想更深入地了解RunLoop，请参考[iOS之Run Loop详解](http://www.henishuo.com/ios-runloop-in-detail/)

#3、在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？

**参考答案：**

RunLoop只能同时运行在一种mode下。因此，如果要换mode，那么当前的loop也需要暂停，并重启成新的mode。利用这个机制，ScrollView滚动过程中NSDefaultRunLoopMode（kCFRunLoopDefaultMode）的mode会切换到UITrackingRunLoopMode来保证ScrollView的流畅滑动，只能在NSDefaultRunLoopMode模式下处理的事件会影响scrllView的滑动。此时，需要希望列表滚动时也能回调timer，那就需要将mode设置为NSRunLoopCommonModes。

**原因：**如果我们把一个NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到运行循环中的时候, ScrollView滚动过程中会因为mode的切换，而导致NSTimer将不被调度。

同时因为mode还是可定制的，所以Timer回调受scrollView的滑动影响的问题可以通过将timer添加到NSRunLoopCommonModes（kCFRunLoopCommonModes）来解决。代码如下：

```
// 将timer添加到NSDefaultRunLoopMode中
// 默认会自动添加到Run Loop中，但是mode为NSDefaultRunLoopMode
[NSTimer scheduledTimerWithTimeInterval:1.0
     target:self
     selector:@selector(timerTick:)
     userInfo:nil
     repeats:YES];
     
// 先添加定时器
// 默认会自动添加到Run Loop中，但是mode为NSDefaultRunLoopMode
NSTimer *timer = [NSTimer timerWithTimeInterval:1.0
     target:self
     selector:@selector(timerTick:)
     userInfo:nil
     repeats:YES];

// 手动修改其mode为forMode:NSRunLoopCommonModes
// 在滚动时，定时器也可以正常回调了
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

如果想更深入地了解RunLoop，请参考[iOS之Run Loop详解](http://www.henishuo.com/ios-runloop-in-detail/)

#4、ObjC使用什么机制管理对象内存？

**参考答案：**

通过引用计数器(retainCount)的机制来决定对象是否需要释放。 每次runloop完成一个事件循环的时候，都会检查对象的retainCount是否为0，如果retainCount为0，说明该对象没有地方需要继续使用了，可以释放掉了，此时才会真正的dealloc。

#5、ARC通过什么方式帮助开发者管理内存？

**参考答案：**

ARC相对于MRC，不是在编译时添加retain/release/autorelease这么简单。应该是编译期和运行期两部分共同帮助开发者管理内存。

* 在编译期，ARC用的是更底层的C接口实现的retain/release/autorelease，这样做性能更好，也是为什么不能在ARC环境下手动retain/release/autorelease，同时对同一上下文的同一对象的成对retain/release操作进行优化（即忽略掉不必要的操作）
* ARC也包含运行期组件，这个地方做的优化比较复杂，但也不能被忽略，手动去做未必优化得好，因此直接交给编译器来优化，相信苹果吧！

#6、BAD_ACCESS在什么情况下出现？

**参考答案：**

这种问题是经常遇到的，在开发时经常会出现BAD_ACCESS。原因是访问了野指针，比如访问已经释放对象的成员变量或者发消息、死循环等。

举个例子，播放视频时，设置了代理，当快速进入播放再快速返回时，就有可能出现这种bug。

#7、苹果是如何实现autoreleasepool的？

**参考答案：**

autoreleasepool以一个队列数组的形式实现,主要通过下列三个函数完成.

* objc_autoreleasepoolPush
* objc_autoreleasepoolPop
* objc_autorelease


看函数名就可以知道，对autorelease分别执行push、pop操作。销毁对象时执行release操作。

举例说明：我们都知道用类方法创建的对象都是 Autorelease 的，那么一旦 Person 出了作用域，当在 Person 的 dealloc 方法中打上断点，我们就可以看到这样的调用堆栈信息：

![image](http://101.200.209.244/wp-content/uploads/2016/02/687474703a2f2f6936302e74696e797069632e636f6d2f31356d666a31312e6a7067.png)

#8、使用block时什么情况会发生引用循环，如何解决？

**参考答案：**

一个对象中强引用了block，在block中又使用了该对象，就会发生循环引用。 解决方法是将该对象使用\_\_weak或者\_\_block修饰符修饰之后再在block中使用。

```
__weak __typeof(self) weakSelf = self；
```

比如，controller中有成员变量_name：

```
__weak __typeof(_name) weakName = _name;
self.vc = [[HYBController alloc] init];
vc.successBlock = ^(NSString *name) {
  weakName = name;
  // 如果直接使用_name，则会造成循环引用
  // _name = name;
};
```

#9、在block内如何修改block外部变量？

**参考答案：**

默认情况下，在block中访问的外部变量是复制过去的，即：写操作不对原变量生效。但是你可以加上\__block来让其写操作生效，示例代码如下:

```
__block int a = 0;
void  (^foo)(void) = ^{ 
    a = 1; 
}

foo(); 
```

#10、使用系统的某些block api（如UIView的block版本写动画时），是否也考虑引用循环问题？

**参考答案：**

系统的某些block api中，UIView的block版本写动画时不需要考虑，但也有一些api需要考虑。所谓“引用循环”是指双向的强引用，

所以那些“单向的强引用”（block 强引用 self ）没有问题，比如这些：

```
[UIView animateWithDuration:duration animations:^{ [self.superview layoutIfNeeded]; }]; 
[[NSOperationQueue mainQueue] addOperationWithBlock:^{ self.someProperty = xyz; }]; 
[[NSNotificationCenter defaultCenter] addObserverForName:@"someNotification" 
                                                  object:nil 
                           queue:[NSOperationQueue mainQueue]
                                              usingBlock:^(NSNotification * notification) {
                                                    self.someProperty = xyz; 
                                                    }]; 
```
这些情况不需要考虑“引用循环”。

但如果你使用一些参数中可能含有成员变量的系统api，如GCD、NSNotificationCenter就要小心一点。比如GCD内部如果引用了 self，而且GCD的其他参数是成员变量，则要考虑到循环引用：

```
__weak __typeof(self) weakSelf = self;
dispatch_group_async(_operationsGroup, _operationsQueue, ^{
	__typeof __(self) strongSelf = weakSelf;
	[strongSelf doSomething];
	[strongSelf doSomethingElse];
});
```

类似的：

```
__weak __typeof(self) weakSelf = self;
_observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"testKey"
                                                            object:nil
                                                             queue:nil
                                                        usingBlock:^(NSNotification *note) {
  __typeof__(self) strongSelf = weakSelf;
  [strongSelf dismissModalViewControllerAnimated:YES];
}];
```
self --> _observer --> block --> self 显然这也是一个循环引用。

#11、GCD的队列（dispatch\_queue\_t）有哪些类型？

**参考答案：**

只有两种类型：

* 串行队列Serial Dispatch Queue
* 并行队列Concurrent Dispatch Queue

我们所熟知的主线程队列dispath\_get\_main\_queue就属于串行队列，而全局队列dispatch\_get\_global\_queue属于并行队列。

#12、如何用GCD同步若干个异步调用？

**参考答案：**

举例：根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图

使用Dispatch Group追加block到Global Group Queue,这些block如果全部执行完毕，就会执行Main Dispatch Queue中的结束处理的block。

当放到group中的所有请求都完成时，才会回调dispatch\_group\_notify的block：

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

#13、dispatch\_barrier\_async的作用是什么？

**参考答案：**

在并行队列中，为了保持某些任务的顺序，需要等待一些任务完成后才能继续进行，使用 barrier 来等待之前任务完成，避免数据竞争等问题。

barrier是屏障的意思，就是说在处理竞争资源时，使用dispatch\_barrier\_async函数会等待追加到Concurrent Dispatch Queue并行队列中的操作全部执行完之后，然后再执行 dispatch\_barrier\_async 函数追加的处理，等dispatch\_barrier\_async追加的处理执行结束之后，Concurrent Dispatch Queue才恢复之前的动作继续执行。

打个比方：比如你们公司周末跟团旅游，高速休息站上，司机说：大家都去上厕所，速战速决，上完厕所就上高速。超大的公共厕所，大家同时去，程序猿很快就结束了，但程序媛就可能会慢一些，即使你第一个回来，司机也不会出发，司机要等待所有人都回来后，才能出发。 dispatch\_barrier\_async 函数追加的内容就如同 “上完厕所就上高速”这个动作。

注意：使用 dispatch\_barrier\_async ，该函数只能搭配自定义并行队列 dispatch\_queue\_t 使用。不能使用： dispatch\_get\_global\_queue ，否则 dispatch\_barrier\_async 的作用会和 dispatch\_async 的作用一模一样。 

#14、苹果为什么要废弃dispatch\_get\_current\_queue？

**参考答案：**

dispatch\_get\_current\_queue容易造成死锁。详情点击该API查看官方注释。

#15、以下代码运行结果如何？

```
- (void)viewDidLoad {
    [super viewDidLoad];
    NSLog(@"1");
    
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2");
    });
    
    NSLog(@"3");
}
```

**参考答案：**

只输出：1，然后主线程锁死。也就是所谓的界面卡死，点击都没有反映，但是又不会闪退。在使用dispatch_sync时一定要小心再小心。

#16、若一个类有实例变量NSString *\_foo，调用setValue:forKey:时，是以foo还是\_foo作为key？

**参考答案：**

两者都可以。

#17、如何调试BAD\_ACCESS错误？


**参考答案：**

出现BAD\_ACCESS错误，通常是访问了野指针，比如访问了已经释放了的对象。快速定位问题的步骤有：

1. 重写对象的respondsToSelector方法，先找到出现EXEC_BAD_ACCESS前访问的最后一个object
2. 设置Enable Zombie Objects
3. 设置全局断点快速定位问题代码所在行，接收所有的异常
4. Xcode7已经集成了BAD_ACCESS捕获功能：Address Sanitizer，与步骤2一样设置

#18、 lldb（gdb）常用的调试命令？

**参考答案：**

* breakpoint 设置断点定位到某一个函数
* n 断点指针下一步，猜测是next的缩写
* po打印对象，笔者猜测po是print object的缩写
* p与po类似，只是打印出来的是地址
* bt 打印堆栈信息
* expr 执行某行代码，比如@import UIKit

#19、Apple用什么方式实现对一个对象的KVO？

Apple 的文档对 KVO 实现的描述：

>Automatic key-value observing is implemented using a technique called isa-swizzling... When an observer is registered for an attribute of an object the isa pointer of the observed object is modified, pointing to an intermediate class rather than at the true class ...

从Apple 的文档可以看出：Apple 并不希望过多暴露 KVO 的实现细节。不过，要是借助 runtime 提供的方法去深入挖掘，所有被掩盖的细节都会原形毕露：

当你观察一个对象时，一个新的类会被动态创建。这个类继承自该对象的原本的类，并重写了被观察属性的 setter 方法。重写的 setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象：值的更改。最后通过 isa 混写（isa-swizzling） 把这个对象的 isa 指针 ( isa 指针告诉 Runtime 系统这个对象的类是什么 ) 指向这个新创建的子类，对象就神奇的变成了新创建的子类的实例。借来的一张示意图，如下所示：

![image](http://101.200.209.244/wp-content/uploads/2016/02/687474703a2f2f6936322e74696e797069632e636f6d2f7379353775722e6a7067.png)

KVO 确实有点黑魔法：

>Apple 使用了 isa 混写（isa-swizzling）来实现 KVO 。

下面做下详细解释：

键值观察通知依赖于 NSObject 的两个方法: willChangeValueForKey: 和 didChangevlueForKey: 。在一个被观察属性发生改变之前， willChangeValueForKey: 一定会被调用，这就会记录旧的值。而当改变发生后， didChangeValueForKey: 会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。可以手动实现这些调用，但很少有人这么做。一般我们只在希望能控制回调的调用时机时才会这么做。大部分情况下，改变通知会自动调用。

比如调用 setNow: 时，系统还会以某种方式在中间插入 wilChangeValueForKey: 、 didChangeValueForKey: 和 observeValueForKeyPath:ofObject:change:context: 的调用。大家可能以为这是因为 setNow: 是合成方法，有时候我们也能看到人们这么写代码:

```
- (void)setNow:(NSDate *)aDate {
    [self willChangeValueForKey:@"now"]; // 没有必要
    _now = aDate;
    [self didChangeValueForKey:@"now"];// 没有必要
}
```

这是完全没有必要的代码，不要这么做，这样的话，KVO代码会被调用两次。KVO在调用存取方法之前总是调用 willChangeValueForKey: ，之后总是调用 didChangeValueForkey: 。怎么做到的呢?答案是通过 isa 混写（isa-swizzling）。第一次对一个对象调用 addObserver:forKeyPath:options:context: 时，框架会创建这个类的新的 KVO 子类，并将被观察对象转换为新子类的对象。在这个 KVO 特殊子类中， Cocoa 创建观察属性的 setter ，大致工作原理如下:

```
- (void)setNow:(NSDate *)aDate {
    [self willChangeValueForKey:@"now"];
    [super setValue:aDate forKey:@"now"];
    [self didChangeValueForKey:@"now"];
}
```
这种继承和方法注入是在运行时而不是编译时实现的。这就是正确命名如此重要的原因。只有在使用KVC命名约定时，KVO才能做到这一点。

KVO 在实现中通过 isa 混写（isa-swizzling） 把这个对象的 isa 指针 ( isa 指针告诉 Runtime 系统这个对象的类是什么 ) 指向这个新创建的子类，对象就神奇的变成了新创建的子类的实例。这在Apple 的文档可以得到印证：

>Automatic key-value observing is implemented using a technique called isa-swizzling... When an observer is registered for an attribute of an object the isa pointer of the observed object is modified, pointing to an intermediate class rather than at the true class ...

然而 KVO 在实现中使用了 isa 混写（ isa-swizzling） ，这个的确不是很容易发现：Apple 还重写、覆盖了 -class 方法并返回原来的类。 企图欺骗我们：这个类没有变，就是原本那个类。。。

但是，假设“被监听的对象”的类对象是 MYClass ，有时候我们能看到对 NSKVONotifying_MYClass 的引用而不是对 MYClass 的引用。借此我们得以知道 Apple 使用了 isa 混写（isa-swizzling）。具体探究过程可参考[这篇博文](https://www.mikeash.com/pyblog/friday-qa-2009-01-23.html)。

