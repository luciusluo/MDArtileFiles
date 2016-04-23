#概述

本篇来研究一下GCD中的子队列如何设置优先级。我们知道全局队列可以有四种优先级可以设置，而我们自己创建的队列并没有参数可以指定优先级，那么我们有办法做到按优先级来执行任务吗？

答案是肯定的。既然苹果只提供了全局队列的优先级，那么我们可以通过将我们手动创建的队列作为全局队列的子队列，并可以设置优先级，我们的问题就可以解决了。

通过本篇文章，您将学习到以下知识点：

* 如何创建子队列
* 如何给子队列添加优先级

#如何设置子队列

苹果提供了一个设置子队列的API：**dispatch\_set\_target\_queue**，通过它可以设置调整目标队列，比如我们可以设置目标队列为全局队列，那么这个全局队列可以先设置优先级，如此就可以解决子队列优先级的问题。比如下面这样设置：

```
// 将serialQueue放到优先级为LOW的全局队列中作为子队列，
// 那么子队列的优先级也会跟着成为LOW优先级
dispatch_set_target_queue(serialQueue, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0));
```

#探索串行子队列优先级

下面我们来探索一下我们手动创建的串行队列，放到优先级为LOW的全局队列中，然后再给全局队列添加几个优先级为DEFAULT的任务，看看任务执行的顺序：

```
 dispatch_queue_t serialQueue = dispatch_queue_create("com.huangyibiao.serial_queue",
                                                      DISPATCH_QUEUE_SERIAL);
// 将serialQueue放到全局队列中作为子队列，这样优先级就是使用默认
dispatch_set_target_queue(serialQueue, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0));
  
dispatch_async(serialQueue, ^{
	NSLog(@"s1");
});
dispatch_async(serialQueue, ^{
	NSLog(@"s2");
});
dispatch_async(serialQueue, ^{
	NSLog(@"s3");
});
  
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	NSLog(@"优先级高-4");
});
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	NSLog(@"优先级高-5");
});
```

打印效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160407-0@2x-e1459998088289.png)

我们把串行队列放到了优先级为LOW的全局队列中，另外还把任务4、5放到优先级为DEFAULT的全局队列中，所以优先执行任务4、5，最后才执行优先级为低的串行队列中的任务。

所以这里设置了优先级后，队列就不一定是按FIFO规则了，出队的顺序就变成了按优先级了。只有当所有的任务都是同一个优先级的情况下，才是FIFO。

#探索并发子队列优先级

下面我们来探索一下我们手动创建的并发队列，放到优先级为LOW的全局队列中，然后再给全局队列添加几个优先级为DEFAULT的任务，看看任务执行的顺序：

```
dispatch_queue_t concurrencyQueue = dispatch_queue_create("com.huangyibiao.concurrency_queue",
                                                          DISPATCH_QUEUE_CONCURRENT);

// 放到全局队列，设置优先级为LOW
dispatch_set_target_queue(concurrencyQueue,
                          dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0));

dispatch_async(concurrencyQueue, ^{
  NSLog(@"c1");
});
dispatch_async(concurrencyQueue, ^{
  NSLog(@"c2");
});
dispatch_async(concurrencyQueue, ^{
  NSLog(@"c3");
});
dispatch_async(concurrencyQueue, ^{
  NSLog(@"c4");
});
dispatch_async(concurrencyQueue, ^{
  NSLog(@"c5");
});


dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  NSLog(@"g1");
});
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  NSLog(@"g2");
});
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  NSLog(@"g3");
});
```

打印效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160407-1@2x.png)

从打印结果来看，任务g1、g2、g3是相同优先级且是最高的，所以这三个任务优先执行，并不是按FIFO来出队执行了。子队列被优先放入全局队列中，按道理说它应该优先出列然后执行任务，但是现在并不是这样子。所以FIFO是有条件的，那就是优先级相同的情况下才会这样。如果出现优先级不同，则会按优先级高的先出队执行。

并发队列执行任务的顺序是不确定的。对于同一优先级的任务，他们出队的顺序一定是FIFO，先进先出，但是先执行的顺序是不确定的！

从效果图可以看出来，同一优先级的g1、g2、g3表现出了执行的顺序是乱的。同样，同一优先级的c1~c5也表现也执行的顺序是乱的，也就是随机性！

#结尾

本篇探索了如何添加子队列，如何实现子队列的优先级设置。设置子队列可以通过**dispatch\_set\_target\_queue**这个API来实现，而优先级设置可以通过全局队列来来设置优先级！


#源代码

GITHUB下载源代码[GCDDemos-Demo2](https://github.com/CoderJackyHuang/GCDDemos.git)

#参考

* [苹果官方API文档](https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_get_current_queue)
* [苹果官方多线程编程文档](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW3)