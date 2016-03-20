#前言

`iOS`实现多线程的方式有三种，分别是`NSThread`、`NSOperation`、`GCD`。

关于`GCD`，请阅读[GCD深入浅出学习](http://www.henishuo.com/gcd-multiple-thread-learn/)

#简介

`NSOperation`封装了需要执行的操作和执行操作所需的数据，提供了并发或非并发操作，可以设置最大并发数，取消操作等。

`iOS`使用`NSOperation`的方式有两种：
* 直接使用系统提供的两个子类：`NSInvocationOperation`和`NSBlockOperation`
* 继承于`NSOperation`

这里所说的抽象类不是真正的抽象类，不像`C++`那种纯虚函数，不能实例化。在`Ojbective-C`中是没有纯虚函数的，因此它是可以实例化的。只是由于没有提供任务接口，因此实例化了也没有意义。

```
NSOperation *op = [[NSOperation alloc] init];
```

**注意：**我们不能直接使用`NSOperation`这个类，这个类相当于一个抽象类，不能直接实例化，必须重写`main`方法。

#NSOperation基类API

下面简单说明`NSOperation`所提供的一些操作。

###1.执行任务

`NSOperation`提供了`start`方法开启任务执行操作，`NSOperation`对象默认按***同步方式***执行,也就是在调用`start`方法的那个线程中直接执行。

```
if (!operation.isExecuting) {
  [operation start];
}
```

###2.判断是否是同步还是异步

`NSOperation`提供的`isConcurrent`可判断是同步还是异步执行。`isConcurrent`默认值为`NO`,表示操作与调用线程同步执行。不过这个方法在`7.0`之后就被废弃了，改成使用`isAsynchronous`判断了。

```
if ([UIDevice currentDevice].systemVersion.intValue >= 7.0) {
  if (operation.isAsynchronous) {
    NSLog(@"异步");
  } else {
    NSLog(@"同步");
  }
} else {
  if (operation.isConcurrent) {
    NSLog(@"异步");
  } else {
    NSLog(@"同步");
  }
}
```

###3.判断任务是否在执行中

`NSOperation`提供了`isExecuting`，可判断任务是否正在执行中。

```
if (!operation.isExecuting) {
  [operation start];
}
```

###4.判断任务是否已经准备好

`NSOperation`提供了`isReady`方法来获取任务是否已经为执行准备好。

```
if (!operation.isReady) {
  [operation start];
}
```

###5.判断任务已经已完成

`NSOperation`提供了`isFinished`，可判断任务是否已经执行完成。

```
if (operation.isFinished) {
  NSLog(@"finished");
}
```

###6.取消任务/判断任务状态

`NSOperation`提供了`isCancelled`，可判断任务是否已经执行完成，而要取消任务，可调用`cancel`方法。

```
if (!operation.isCancelled) {
	[operation cancel];
}
```

###7.任务完成回调

如果我们想在一个`NSOperation`执行完毕后做一些事情，可以调用`NSOperation`的`completionBlock`属性来设置在任务完成以后我们还想做的事情。

我们可以通过这种点语法设置：

```
operation.completionBlock = ^() {
  NSLog(@"任务执行完毕");
};
```

也可以通过中括号方式设置：

```
[operation setCompletionBlock:^{
  NSLog(@"任务执行完毕");
}];
```

###8.任务优先级

如下，`NSOperation`为我们提供了在`NSOperationQueue`调度队列中任务的优先级设置。

```
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
	NSOperationQueuePriorityVeryLow = -8L,
	NSOperationQueuePriorityLow = -4L,
	NSOperationQueuePriorityNormal = 0,
	NSOperationQueuePriorityHigh = 4,
	NSOperationQueuePriorityVeryHigh = 8
};

@property NSOperationQueuePriority queuePriority;
```

#NSInvocationOperation子类

`NSInvocationOperation`是继承于`NSOperation`，提供创建任务的方式是通过`selector`。

```
- (nullable instancetype)initWithTarget:(id)target selector:(SEL)sel object:(nullable id)arg;
- (instancetype)initWithInvocation:(NSInvocation *)inv NS_DESIGNATED_INITIALIZER;
```

对于第二个初始化方法已经被废弃了，第二个初始化方法是通过运行时的方式来添加任务的，操作起来比较复杂。第一种就是很普通的方式，是很常见的`target-action`设计模式。

```
NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(updateUI) object:nil];
// 开始执行任务(同步执行)
[operation start];
```

调用`start`方法是同步执行的。如果要异步执行，可以放到`NSOperationQueue`队列中，它就相当于一个线程池，而且任务一旦放进去，就会按照`FIFO`的原则严格执行任务。任务放到线程池中后，是否会马上执行，是根据当前所设置的并发数量决定的。

看看我们下载一个图片：

```
- (void)test1 {
  NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self
                                                                         selector:@selector(downloadImage:)
                                                                           object:@"图片的URL"];
  
  NSOperationQueue *queue = [[NSOperationQueue alloc]init];
  [queue addOperation:operation];
}

- (void)downloadImage:(NSString *)url {
  NSURL *nsUrl = [NSURL URLWithString:url];
  NSData *data = [[NSData alloc] initWithContentsOfURL:nsUrl];
  UIImage *image = [[UIImage alloc] initWithData:data];
  
  dispatch_async(dispatch_get_main_queue(), ^{
    self.imageView.image = image;
  });
}
```

我们需要注意，最后在更新`UI`的时候，一定要回到主线程，否则`UI`效果不会马上变化。当然，我们也可以使用别的方式回到主线程更新`UI`：

```
[self performSelectorOnMainThread:@selector(updateUI:) withObject:image waitUntilDone:YES];  

- (void)updateUI:(UIImage *) image{  
    self.imageView.image = image;  
}  
```

#NSBlockOperation子类 

`NSBlockOperation`是直接继承于`NSOperation`的子类，它能够并发地执行一个或多个`block`对象，所有的`block`都执行完之后,操作才算真正完成。

###添加任务

`NSBlockOperation`都是`block`任务，操作起来比较简洁一些。

```
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^(){
  NSLog(@"这是第一个任务在线程：%@执行，isMainThread: %d，isAync: %d",
        [NSThread currentThread],
        [NSThread isMainThread],
        [operation isAsynchronous]);
}];

__weak typeof(operation) weakOperation = operation;
[operation addExecutionBlock:^() {
  NSLog(@"这是第二个任务在线程：%@执行，isMainThread: %d，isAync: %d",
        [NSThread currentThread],
        [NSThread isMainThread],
        [weakOperation isAsynchronous]);
}];

[operation addExecutionBlock:^() {
  NSLog(@"这是第三个任务在线程：%@执行，isMainThread: %d，isAync: %d",
        [NSThread currentThread],
        [NSThread isMainThread],
        [weakOperation isAsynchronous]);
}];

[operation addExecutionBlock:^() {
  NSLog(@"这是第四个任务在线程：%@执行，isMainThread: %d，isAync: %d",
        [NSThread currentThread],
        [NSThread isMainThread],
        [weakOperation isAsynchronous]);
}];

// 开始执行任务
[operation start]
```

看看打印结果：

```
2015-11-24 17:15:49.489 TestGCD[42401:3307632] 这是第一个任务在线程：<NSThread: 0x7fb369e02c30>{number = 1, name = main}执行，isMainThread: 1，isAync: 0
2015-11-24 17:15:49.489 TestGCD[42401:3307797] 这是第二个任务在线程：<NSThread: 0x7fb369ca8880>{number = 2, name = (null)}执行，isMainThread: 0，isAync: 0
2015-11-24 17:15:49.489 TestGCD[42401:3307809] 这是第三个任务在线程：<NSThread: 0x7fb369ca92b0>{number = 3, name = (null)}执行，isMainThread: 0，isAync: 0
2015-11-24 17:15:49.489 TestGCD[42401:3307798] 这是第四个任务在线程：<NSThread: 0x7fb369f2acd0>{number = 4, name = (null)}执行，isMainThread: 0，isAync: 0
```

由此，我们可以看到第一个任务在主线程执行，第二、三、四个任务都是其它子线程完成的。这四个任务都是同步执行的。其中的`number`代表线程的`id`。

当我们把`[operation start]`这行改成这样：

```
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperation:operation];
[queue setMaxConcurrentOperationCount:2];
```

其打印结果如下：

```
2015-11-24 17:22:26.206 TestGCD[42558:3316054] 这是第三个任务在线程：<NSThread: 0x7f9000744750>{number = 4, name = (null)}执行，isMainThread: 0，isAync: 0
2015-11-24 17:22:26.206 TestGCD[42558:3316053] 这是第一个任务在线程：<NSThread: 0x7f900061b6f0>{number = 2, name = (null)}执行，isMainThread: 0，isAync: 0
2015-11-24 17:22:26.206 TestGCD[42558:3316055] 这是第二个任务在线程：<NSThread: 0x7f90006183b0>{number = 3, name = (null)}执行，isMainThread: 0，isAync: 0
2015-11-24 17:22:26.206 TestGCD[42558:3316062] 这是第四个任务在线程：<NSThread: 0x7f9000609680>{number = 5, name = (null)}执行，isMainThread: 0，isAync: 0
```

由于我们设置了最大并发数量为2，因此同时能执行的任务数量最多两个。而这四个任务都不是在主线程执行的，全部放到子线程中执行了。我们发现isAync都为0，也就是说`operation`的`isAsynchronous`方法返回都是`NO`。

**注意：**并发与异步不是同一个概念

要异步执行，可以这样：

```
[operation addExecutionBlock:^() {
  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"这是第四个任务在线程：%@执行，isMainThread: %d",
          [NSThread currentThread],
          [NSThread isMainThread]);
  });
}];
```

#自定义NSOperation

如果`NSInvocationOperation`和`NSBlockOperation`对象不能满足需求, 我们可以直接继承`NSOperation`, 并添加额外的功能。继承所需的工作量主要取决于你要实现非并发还是并发的`NSOperation`。定义同步的`NSOperation`要简单许多,只需要重载`-main`这个方法，在这个方法里面执行主任务,并正确地响应取消事件; 对于异步`NSOperation`, 必须重写`NSOperation`的多个基本方法进行实现（main、start）。

###自定义非并发NSOperation

1. 使用自己定义的同步operation，需要继承自NSOperation，并实现必要的方法：isFinished，isExecuting，main等，并实现KVO机制
2. 如果不想让你自定义的operation与其他operation进行异步操作,你可以手动开始（调用start方法），并且在operation的start方法里面简单的调用main方法。
3. 如果要自定义operation,需要继承资NSOperation。并且重载 isExecuting方法和isFinished方法。在这两个方法中,必须返回一个线程安全值（通常是个BOOL值）,这个值可以在 operation 中进行操作。
4. 一旦你的 operation 开始了,必须通过 KVO,告诉所有的监听者,现在该operation的执行状态。
5. 在 operation 的 main 方法里面,必须提供 autorelease pool,因为你的 operation 完成后需要销毁。


自定义同步NSOperation，只需要重写main方法即可。我们定义一个图片下载类来说明如何自定义非并发的`NSOperation`：

```
typedef void(^HYBDownloadReponse)(UIImage *image);

/*!
 *  @author 黄仪标, 15-11-24 22:11:50
 *
 *  下载operation
 */
@interface DownloadOperation : NSOperation

@property (nonatomic, copy) NSString *url;
@property (nonatomic, copy) HYBDownloadReponse responseBlock;

- (instancetype)initWithUrl:(NSString *)url completion:(HYBDownloadReponse)completion;

@end
```

下面看看实现方法怎么实现的，关键点在于`-main`方法：

```
//
//  DownloadOperation.m
//  TestGCD
//
//  Created by huangyibiao on 15/11/24.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import "DownloadOperation.h"

@interface DownloadOperation () {
  BOOL _isFinished;
  BOOL _isExecuting;
}

@end

@implementation DownloadOperation

- (instancetype)initWithUrl:(NSString *)url completion:(HYBDownloadReponse)completion {
  if (self = [super init]) {
    self.url = url;
    self.responseBlock = completion;
  }
  
  return self;
}

// 必须重写这个主方法
- (void)main {
  // 新建一个自动释放池，如果是异步执行操作，那么将无法访问到主线程的自动释放池
  @autoreleasepool {
    // 提供一个变量标识，来表示我们需要执行的操作是否完成了，当然，没开始执行之前，为NO
    BOOL taskIsFinished = NO;
    UIImage *image = nil;
    
    // while 保证：只有当没有执行完成和没有被取消，才执行我们自定义的相应操作
    while (taskIsFinished == NO && [self isCancelled] == NO){
      // 获取图片数据
      NSURL *url = [NSURL URLWithString:self.url];
      NSData *imageData = [NSData dataWithContentsOfURL:url];
      image = [UIImage imageWithData:imageData];
      NSLog(@"Current Thread = %@", [NSThread currentThread]);
      
      // 这里，设置我们相应的操作都已经完成，后面就是要通知KVO我们的操作完成了。
      
      taskIsFinished = YES;
    }
    
    // KVO 生成通知，告诉其他线程，该operation 执行完了
    [self willChangeValueForKey:@"isFinished"];
    [self willChangeValueForKey:@"isExecuting"];
    _isFinished = YES;
    _isExecuting = NO;
    [self didChangeValueForKey:@"isFinished"];
    [self didChangeValueForKey:@"isExecuting"];
    
    if (self.responseBlock) {
      dispatch_async(dispatch_get_main_queue(), ^{
        self.responseBlock(image);
      });
    }
  }
}

- (void)start {
  // 如果我们取消了在开始之前，我们就立即返回并生成所需的KVO通知
  if ([self isCancelled]){
    // 我们取消了该 operation，那么就要告诉KVO，该operation已经执行完成（isFinished）
    // 这样，调用的队列（或者线程）会继续执行。
    [self willChangeValueForKey:@"isFinished"];
    _isFinished = NO;
    [self didChangeValueForKey:@"isFinished"];
  } else {
    // 没有取消，那就要告诉KVO，该队列开始执行了（isExecuting）！那么，就会调用main方法，进行同步执行。
    [self willChangeValueForKey:@"isExecuting"];
    _isExecuting = YES;
	[NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
    [self didChangeValueForKey:@"isExecuting"];
  }
}

- (BOOL)isExecuting {
  return _isExecuting;
}

- (BOOL)isFinished {
  return _isFinished;
}

@end
```

我们测试一下：

```
DownloadOperation *operation = [[DownloadOperation alloc] initWithUrl:@"https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD0wkCgnmf6DVxI6fVewNS8rhtZb71v2DMpDy8jIdtviaetzicwQzTEoKKyHAN96Beibk2G61tZpezQ0Q/0?wx_fmt=png" completion:^(UIImage *image) {
  //    self.imageView.image = image;
}];

DownloadOperation *operation1 = [[DownloadOperation alloc] initWithUrl:@"https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD0wkCgnmf6DVxI6fVewNS8rhtZb71v2DMpDy8jIdtviaetzicwQzTEoKKyHAN96Beibk2G61tZpezQ0Q/0?wx_fmt=png" completion:^(UIImage *image) {
  //    self.imageView1.image = image;
}];

DownloadOperation *operation2 = [[DownloadOperation alloc] initWithUrl:@"https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD0wkCgnmf6DVxI6fVewNS8rhtZb71v2DMpDy8jIdtviaetzicwQzTEoKKyHAN96Beibk2G61tZpezQ0Q/0?wx_fmt=png" completion:^(UIImage *image) {
  //    self.imageView2.image = image;
}];

DownloadOperation *operation3 = [[DownloadOperation alloc] initWithUrl:@"https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD0wkCgnmf6DVxI6fVewNS8rhtZb71v2DMpDy8jIdtviaetzicwQzTEoKKyHAN96Beibk2G61tZpezQ0Q/0?wx_fmt=png" completion:^(UIImage *image) {
  //    self.imageView3.image = image;
}];

DownloadOperation *operation4 = [[DownloadOperation alloc] initWithUrl:@"https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD0wkCgnmf6DVxI6fVewNS8rhtZb71v2DMpDy8jIdtviaetzicwQzTEoKKyHAN96Beibk2G61tZpezQ0Q/0?wx_fmt=png" completion:^(UIImage *image) {
  //    self.imageView4.image = image;
}];

NSOperationQueue *queue = [[NSOperationQueue alloc]init];
[queue addOperation:operation];
[queue addOperation:operation1];
[queue addOperation:operation2];
[queue addOperation:operation3];
[queue addOperation:operation4];
[queue setMaxConcurrentOperationCount:2];
```

打印结果：

```
2016-03-14 16:15:18.777 DataAgorithmDemos[6390:408581] Current Thread = <NSThread: 0x7ff6c96a23d0>{number = 3, name = (null)}
2016-03-14 16:15:19.055 DataAgorithmDemos[6390:408668] Current Thread = <NSThread: 0x7ff6c972fa90>{number = 4, name = (null)}
2016-03-14 16:15:19.287 DataAgorithmDemos[6390:408669] Current Thread = <NSThread: 0x7ff6c941f3d0>{number = 5, name = (null)}
2016-03-14 16:15:19.441 DataAgorithmDemos[6390:408579] Current Thread = <NSThread: 0x7ff6c95472a0>{number = 6, name = (null)}
2016-03-14 16:15:19.520 DataAgorithmDemos[6390:408580] Current Thread = <NSThread: 0x7ff6c9519090>{number = 7, name = (null)}
```

但是当我们不是放在queue中，而是手动start：

```
DownloadOperation *operation = [[DownloadOperation alloc] initWithUrl:@"https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD0wkCgnmf6DVxI6fVewNS8rhtZb71v2DMpDy8jIdtviaetzicwQzTEoKKyHAN96Beibk2G61tZpezQ0Q/0?wx_fmt=png" completion:^(UIImage *image) {
  //    self.imageView.image = image;
}];
[operation start];

// 打印结果：
// 2016-03-14 16:18:30.146 DataAgorithmDemos[6427:413473] Current Thread = <NSThread: 0x7fb3434052f0>{number = 1, name = main}
```


###自定义并发NSOperation

头文件与同步的声明一样。下面是实现文件部分：

```
//
//  DownloadOperation.m
//  DataAgorithmDemos
//
//  Created by huangyibiao on 16/3/14.
//  Copyright © 2016年 huangyibiao. All rights reserved.
//

#import "DownloadOperation.h"

@interface DownloadOperation () {
@private BOOL _isFinished;
@private BOOL _isExecuting;
}

@end

@implementation DownloadOperation

- (instancetype)initWithUrl:(NSString *)url completion:(HYBDownloadReponse)completion {
  if (self = [super init]) {
    self.url = url;
    self.responseBlock = completion;
  }
  
  return self;
}

// 必须重写这个主方法
- (void)main {
  // 新建一个自动释放池，如果是异步执行操作，那么将无法访问到主线程的自动释放池
  @autoreleasepool {
    UIImage *image = nil;
    if (!self.isCancelled) {
      // 获取图片数据
      NSURL *url = [NSURL URLWithString:self.url];
      NSData *imageData = [NSData dataWithContentsOfURL:url];
      image = [UIImage imageWithData:imageData];
    }
    
    NSLog(@"currentThread: %@", [NSThread currentThread]);
    
    // 被取消，也可能发生在转换的地方
    if (self.isCancelled) {
      image = nil;
    }
    
    
    if (![self isCancelled] && self.responseBlock) {
      dispatch_async(dispatch_get_main_queue(), ^{
        self.responseBlock(image);
      });
    }
  }
}

// 与自定义同步NSOperation不同的是，必须要实现下面的方法
#if __IPHONE_OS_VERSION_MAX_ALLOWED < __IPHONE_7_0
- (BOOL)isConcurrent {
  return YES;
}

#else

- (BOOL)isAsynchronous {
  return YES;
}
#endif

@end
```

打印结果：

```
2016-03-14 15:58:02.610 DataAgorithmDemos[6154:386771] currentThread: <NSThread: 0x7fbc83c9c2c0>{number = 3, name = (null)}
2016-03-14 15:58:02.612 DataAgorithmDemos[6154:386769] currentThread: <NSThread: 0x7fbc83ca01b0>{number = 4, name = (null)}
2016-03-14 15:58:02.612 DataAgorithmDemos[6154:386824] currentThread: <NSThread: 0x7fbc83d24eb0>{number = 5, name = (null)}
2016-03-14 15:58:04.210 DataAgorithmDemos[6154:386837] currentThread: <NSThread: 0x7fbc83ca0b30>{number = 6, name = (null)}
2016-03-14 15:58:11.693 DataAgorithmDemos[6154:386768] currentThread: <NSThread: 0x7fbc83d1a650>{number = 7, name = (null)}
```

#最后

由于这方面的知识理解的深度不够，后面还会不断更新。希望大家多多交流，一起学习多线程方面的知识!

#参考

【[苹果官方英文文档](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW9)】