#序言

最近收到某某同学将去youku的iOS笔试题的邮件，希望笔者能整理一下，并提供参考答案。笔者决定整理出来，并分享给大家。当然，与此同时，也想看看youku的笔试题到底有多难，也考考自己有多少料吧！

如果所提供的参考答案有任何值得置疑的地方，请一定要在评论中指出！

#题照

![image](http://101.200.209.244/wp-content/uploads/2016/03/ios1.jpg)

#1、如何声明私有变量和私有方法？

**参考答案：**

* 声明私有变量可以通过@private关键字来声明。例如，这样就是私有的成员变量了：

```
@interface HYBTestModel : NSObject {
  @private NSString *_userName;
}
@end
```

* 没有关键字声明为私有方法，因为ObjC中也没有真正意义上的私有方法。我们要让方法成员私有，只能通过放在.m文件中定义而不暴露在外部。但是，如果有人知道内部此这么一个方法，那么也是可以访问的。

先说明：ObjC中没有绝对的私有变量和私有方法。

如何修改私有成员变量的值？

```
HYBTestModel *model = [[HYBTestModel alloc] init];

// 通过KVC可以轻松修改私有成员变量
// 自己加一个打印就可以看到有值了！
[model setValue:@"修改私有变量的值" forKey:@"_userName"];
```

那又如何访问私有成员变量？

```
Ivar userNameIvar = class_getInstanceVariable([model class], "_userName");
NSString *userName = object_getIvar(model, userNameIvar);
```

我们可以通过runtime来获取对象的成员变量Ivar，然后再通过object_getIvar来获取某个对象的成员变量的值。

看到这里，还相信ObjC中所谓私有变量吗？

#2、assign、retain、copy分别起什么作用？重写下面的属性的getter/setter方法

```
@property (nonatomic, retain) NSNumber *num;
```

**参考答案：**

从题目可知这问的是MRC下的问题。在MRC下：

* assign用于非对象类型，对于对象类型的只用于弱引用。
* retain用于对象类型，强引用对象
* copy用于对象类型，强引用对象。

重写setter/getter（如何重写getter和setter，是不会自动登录_num成员变量的，需要自己手动声明）：

```
- (NSNumber *)num {
   return _num;
}

- (void)setNum:(NSNumber *)aNum {
   if (_num != aNum) {
     [_num release];
     _num = nil;
     
     _num = [aNum retain];
   }
}
```

#3、如何声明一个delegate属性，为什么？

**参考答案：**

声明属性时要，在ARC下使用weak，在MRC下使用assign。比如：

```
@property (nonatomic, weak) id<HYBTestDelegate> delegate;
```

在MRC下，使用assign是因为没有weak关键字，只能使用assign来防止循环引用。在ARC下，使用weak来防止循环引用。

#4、autorelease的对象何时被释放

**参考答案：**

如果了解一点点Run Loop的知道，应该了解到：Run Loop在每个事件循环结束后会去自动释放池将所有自动释放对象的引用计数减一，若引用计数变成了0，则会将对象真正销毁掉，回收内存。

所以，autorelease的对象是在每个事件循环结束后，自动释放池才会对所有自动释放的对象的引用计数减一，若引用计数变成了0，则释放对象，回收内存。因此，若想要早一点释放掉auto release对象，那么我们可以在对象外加一个自动释放池。比如，在循环处理数据时，临时变量要快速释放，就应该采用这种方式：

```
for (int i = 0; i < 10000000; ++i) {
   @autoreleasepool {
      HYBTestModel *tempModel = [[HYBTestModel alloc] init];
      // 临时处理
      // ...
   } // 出了这里，就会去遍历该自动释放池了
}
```

#5、这段代码有问题吗？如何修改？

```
for (int i = 0; i < 10000; ++i) {
  NSString *str = @"Abc";
  str = [str lowercaseString];
  str = [str stringByAppendingString:@"xyz"];
  
  NSLog(@"%@", str);
}
```

**参考答案：**

这道题从语法上看没有任何问题的，当然，既然面试官出了这一道题，那肯定是有问题的。

问题出在哪里呢？语法没有错啊？内存最后也可以得到释放啊！为什么会有问题呢？是的，问题是挺大的。这对于不了解iOS的自动释放池的原理的人或者说内存管理的人来说，这根本看不出来这有什么问题。

**问题就出在内存得不到及时地释放**。为什么得不到及时地释放？因为Run Loop是在每个事件循环结束后才会自动释放池去使对象的引用计数减一，对于引用计数为0的对象才会真正被销毁、回收内存。

因此，对于这里的问题，一个for循环执行10000次，会产生10000个临时自动释放对象，一直放到自动释放池中管理，而内存却得不到及时回收。

然后，现象是**内存暴涨**。正确的写法：

```
for (int i = 0; i < 10000; ++i) {
  @autoreleasepool {
	NSString *str = @"Abc";
	str = [str lowercaseString];
	str = [str stringByAppendingString:@"xyz"];
	  
	NSLog(@"%@", str);
  }
}
```


#6、UIViewController的viewDidUnload、viewDidLoad和loadView分别什么时候调用？UIView的drawRect和layoutSubviews分别起什么作用？

**参考答案：**

第一个问题：

* 在控制器被销毁前会调用viewDidUnload（MRC下才会调用）
* 在控制器没有任何view时，会调用loadView
* 在view加载完成时，会调用viewDidLoad

第二个问题：

* 在调用setNeedsDisplay后，会调用drawRect方法，我们通过在此方法中可以获取到context（设置上下文），就可以实现绘图
* 在调用setNeedsLayout后，会调用layoutSubviews方法，我们可以通过在此方法去调整UI。当然能引起layoutSubviews调用的方式有很多种的，比如添加子视图、滚动scrollview、修改视图的frame等。

#7、自定义NSOperation，需要实现哪些方法？

**参考答案：**

* 对于自定义非并发NSOperation，只需要实现main方法就可以了。
* 对于自定义并发NSOperation，需要重写main、start、isFinished、isExecuting，还要注意在相关地方加上kvo的代码，通知其它线程，否则当任务完成时，若没有设置isFinished=YES，isExecuting=NO，任务是不会退队的。

更多内容看这里：[iOS NSOperation](http://101.200.209.244/ios-nsoperation-queue/)

#8、如何扩展ObjC里面类的方法？

**参考答案：**

不太清楚题目的语义，好像是说扩展类方法？通过category很容易做到，这里就不说了！

#9、用代码实现一个单例

**参考答案：**

随手写一个吧：

```
+ (instancetype)sharedInstance {
  static id s_manager = nil;
  
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    s_manager = [[HYBTestSingleton alloc] init];
  });
  
  return s_manager;
}
```

#10、用代码实现一个冒泡算法

**参考答案：**

冒泡算法的核心算法思想是每趟两两比较，将小的往上浮，大的往下沉，就像气泡一样从水底往水面浮。

```
void bubbleSort(int a[], int len) {
  for (int i = 0; i < len - 1; ++i) {
     // 从水底往水面浮，所以从最后一个开始
     for (int j = len - 1; j > i; j--) {
       // 后者比前者还小，将需要交换
       if (a[j] < a[j - 1]) {
          int temp = a[j];
          a[j] = a[j - 1];
          a[j - 1] = temp;
       }
     }
  }
}
```

更详细地可阅读：[冒泡排序](http://101.200.209.244/bubble-sort/)

#11、UITableView是如何重用cell的？

**参考答案：**

UITableView提供了一个属性：visibleCells，它是记录当前在屏幕可见的cell，要想重用cell，我们需要明确指定重用标识（identifier）。

当cell滚动出tableview可视范围之外时，就会被放到可重用数组中。当有一个cell滚动出tableview可视范围之外时，同样也会有新的cell要显示到tableview可视区，因此这个新显示出来的cell就会先从可重用数组中通过所指定的identifier来获取，如果能够获取到，则直接使用之，否则创建一个新的cell。

#12、如何更高效地显示列表

**参考答案：**

要更高效地显示列表（不考虑种种优化），可以通过以下方法处理（只是部分）：

* 提前根据数据计算好高度并缓存起来
* 提前将数据处理、I/O计算异步处理好，并保存结果，在需要时直接拿来使用

#13、Cocoa中MVC是怎么实现的？

**参考答案：**

这个问题三言两语讲不明白。简单来说，M对应于Model（数据层）、V对应于View（视图层）、C对应于Controller（控制器层）。

如下图：

![image](http://101.200.209.244/wp-content/uploads/2016/03/屏幕快照-2016-03-14-下午5.24.04-e1457947511155.png)

用户在V上操作，需要通过C更新M，然后将新的M交到C，C让M更新。

更详细可以阅读：[iOS中的MVC设计模式](https://liuzhichao.com/p/1379.html)

#14、描述KVC、KVO机制

**参考答案：**

KVC即是指NSKeyValueCoding，是一个非正式的Protocol，提供一种机制来间接访问对象的属性。KVO 就是基于KVC实现的关键技术之一。

KVO即Key-Value Observing，是建立在KVC之上，它能够观察一个对象的KVC key path值的变化。
当keypath对应的值发生变化时，会回调observeValueForKeyPath:ofObject:change:context:方法，我们可以在这里处理。

更详细的内容，请自行百度吧，现在笔者没有写相关文章！

#15、使用或了解哪些设计模式

**参考答案：**

笔者只说说我们在开发中真正常用到的设计模式（包括架构设计模式）：

* 单例设计模式
* MVC构架设计模式
* 工厂设计模式
* 观察者设计模式（比如KVC/KVO/NSNotification，也有人说不是设计模式）
* 代理设计模式


更详细可以看这里：[23种设计模式目录](observeValueForKeyPath:ofObject:change:context: )

#最后

真心挺累的，整理一篇真不容易，光打字都快累死了~喜欢就打赏支持一下，虽不多，至少给点力气~

#推荐阅读

* 【[iOS面试宝典](http://101.200.209.244/ios-interview-entrance/)】