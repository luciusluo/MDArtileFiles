#概述

本篇一起来学习GCD队列相关知识及如何使用。一直以来都是看到过别人这么用，说实在的，还真没有学过文档，也没有深入研究过其所以然。今天一起来看看苹果的GCD队列相关知识，扫一扫盲区吧！

学习完本篇，您会对以下知识点更加理解：

* 队列
* 串行队列
* 并发队列
* GCD全局队列
* GCD主队列
* 创建串行队列
* 创建并发队列

#队列基础知识

在大学学习过队列、栈数据结构吧？如果学习过，应该是非常容易理解的。不管是什么队列，一定是FIFO队列，即先进先出。

所以，请大家记住了：不管是串行队列（SerialQueue）还是并发队列（ConcurrencyQueue），都是FIFO队列。也就意味着，任务一定是一个一个地，按照先进先出的顺序来执行。

**串行队列：**在创建队列时，传参数DISPATCH\_QUEUE\_SERIAL表示创建串行队列。任务会一个一个地执行，只有前一个任务执行完成，才会继续执行下一个任务。**串行执行并不是同步执行的意思，一定要注意区分**

**并发队列：**在创建队列时，传参数DISPATCH\_QUEUE\_CONCURRENT表示创建并发队列。并发队列会尽可能多地创建线程去执行任务。并发队列中的任务会按入队的顺序执行任务，但是哪个任务先完成是不确定的。

#队列类型

苹果提供了以下队列：

1. 全局队列：苹果预定义的全局并发队列，只能通过苹果提供的API来获取，可以设置优先级。
2. 主队列：在应用启动的时候，就会自动创建与主线程关联的串行队列，我们也可能获取，不能手动创建。
3. 手动创建串行队列
4. 手动创建并发队列

##全局队列

全局队列的第二个参数用于设置优先级，只有下面四个选项：

```
/*
 #define DISPATCH_QUEUE_PRIORITY_HIGH 2
 #define DISPATCH_QUEUE_PRIORITY_DEFAULT 0
 #define DISPATCH_QUEUE_PRIORITY_LOW (-2)
 #define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN
 */
```

我们通常使用默认选项，所以很多时候看到的都是传0。下面我们来看看创建四个任务放到并发队列中异步地执行：

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  NSLog(@"1");
});
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  // 睡眠2秒
  sleep(2);
  NSLog(@"2");
});

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  // 睡眠3秒
  sleep(1);
  NSLog(@"3");
});
```

打印结果如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160406-1@2x-e1459928644406.png)

打印出来的队列是通过dispatch\_get\_current\_queue来获取当前正在执行任务的队列，也就是主队列了。不过这个API已经在6.0以后被标识为DEPRECATED(废弃)了。

上面的代码中，我们在并发队列中添加了三个任务，其中任务1是直接执行，任务2是在异步执行过程中被睡眠2秒，任务3在异步执行过程中被睡眠1秒，结果任务3先于任务2执行完成。说明并发执行任务并不需要等待其他任务先执行完。对于这三个任务，是互不干扰的！

当然，全局队列可以指定任务执行的优先级的，比如下面：

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{
  NSLog(@"4");
});

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
  NSLog(@"3");
});

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  NSLog(@"2");
});

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
  NSLog(@"1");
});
```

打印如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160406-4@2x-e1459930188377.png)

任务四优先级最高，所以在并发队列中优先执行。

**提示：**暂时忽略dispatch\_async，后续文件会学习

##主队列

主队列是应用程序启动时，由系统预先创建的，与主线程相关联的队列。我们只能通过系统API来获取主队列，不能手动创建它。下面我们来看看主队列这个串行队列的执行顺序如何：

```
dispatch_async(dispatch_get_main_queue(), ^{
	sleep(2);
	NSLog(@"main 1");
});
  
dispatch_async(dispatch_get_main_queue(), ^{
	NSLog(@"main 2");
});
  
dispatch_async(dispatch_get_main_queue(), ^{
	sleep(1);
	NSLog(@"main 3");
});
```

打印结果：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160406-0@2x-e1459928691992.png)

从打印结果可看到执行的顺序是按入队的顺序来执行的。虽然让任务1睡眠2秒再执行，其他任务也只能等待任务1完成，才能继承执行任务2，在任务2执行完成，才能执行任务3。

从打印结果可以看到线程号是固定的，说明都在同一个线程中执行，而这个线程就是主线程。任务只能一个一个地执行。

**提示：**暂时忽略dispatch\_async，后续文件会学习

##创建串行队列

通过dispatch\_queue\_create函数来创建队列，参数一是一个C语言的字符串，是队列的标签，也就是名称，通常是采用com.<organization>.<function>这样的格式。参数二是指定串行队列还是并发队列。

* 创建串行队列传：DISPATCH\_QUEUE\_SERIAL（也就是NULL）
* 创建并发队列传：DISPATCH\_QUEUE\_CONCURRENT


```
dispatch_queue_t serialQueue = dispatch_queue_create("com.huangyibiao.serial-queue", 
DISPATCH_QUEUE_SERIAL);

dispatch_async(serialQueue, ^{
  NSLog(@"s1");
});
dispatch_async(serialQueue, ^{
  sleep(2);
  NSLog(@"s2");
});
dispatch_async(serialQueue, ^{
  sleep(1);
  NSLog(@"s3");
});
```

打印结果如下：


![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160406-2@2x-e1459929519843.png)

从打印结果可以看出来，任务全在同一个线程中执行，但是并不是在主线程，而是在子线程执行。不过任务执行只有顺序地执行，任务没有执行完毕之前，下一个任务是不能开始的。

##创建并发队列

通过dispatch\_queue\_create函数来创建队列，参数一是一个C语言的字符串，是队列的标签，也就是名称，通常是采用com.<organization>.<function>这样的格式。参数二是指定串行队列还是并发队列。

* 创建串行队列传：DISPATCH\_QUEUE\_SERIAL（也就是NULL）
* 创建并发队列传：DISPATCH\_QUEUE\_CONCURRENT

一起来看看串行队列是否需要等待任务执行完成，下一个任务才能开始：

```
dispatch_queue_t concurrencyQueue = dispatch_queue_create("com.huangyibiao.concurrency-queue",                            
DISPATCH_QUEUE_CONCURRENT);

dispatch_async(concurrencyQueue, ^{
  NSLog(@"s1");
});
dispatch_async(concurrencyQueue, ^{
  sleep(2);
  NSLog(@"s2");
});
dispatch_async(concurrencyQueue, ^{
  sleep(1);
  NSLog(@"s3");
});
```

打印结果：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160406-3@2x-e1459929896759.png)

从打印结果可以看出来，任务在三个子线程中执行，且互不干扰，不需要等待其他任务完成，就可以并发地分别去执行！

#结尾

本篇到此为止，量不多，应该比较好吸收。文中若有不正确之处，请在评论中指出！看到这里，对GCD中的队列知识应该已经掌握得差不多了！

#参考

* [苹果官方API文档](https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_get_current_queue)
* [苹果官方多线程编程文档](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW3)