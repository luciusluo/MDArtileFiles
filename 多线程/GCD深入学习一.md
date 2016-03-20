
#引言

虽然`GCD`使用很广，而且在面试时也经常问与`GCD`相关的问题，但是我相信深入理解关于`GCD`知识的人肯定不多，大部分都是人云亦云，只是使用过`GCD`完成一些很简单的功能。当然，使用`GCD`完成一些简单的功能，通常已经能够满足我们的需求了。不过，笔者比较喜欢刨根问底，因此在这里记录下学习的过程。

`iOS`实现提供实现多线程的方案有：`NSThread`、`NSOperation`、`GCD`。

在`iOS`所有实现多线程的方案中，`GCD`应该是最有魅力的，而且使用起来也是最方便的，因为`GCD`是苹果公司为多核的并行运算提出的解决方案。

`GCD`是`Grand Central Dispatch`的简称，它是基于`C`语言的。使用`GCD`，我们不需要编写线程代码，其生命周期也不需要我们手动管理，定义想要执行的任务,然后添加到适当的调度队列，也就是`dispatch queue`。`GCD`会负责创建线程和调度任务，系统直接提供线程管理。

由于`GCD`是基于`C`语言的，因此使用起来对于没有学习过`C`语言的同学们，相对困难一些。不过，事实上使用是很简单，只要注意**死锁**等问题就好了。

#概念：队列（Queue）


我们需要了解队列的概念，`GCD`提供了`dispatch queues`来处理代码块，这些队列管理所提供给`GCD`的任务并用`FIFO`顺序执行这些任务。这样才能保证第一个被添加到队列里的任务会是队列中第一个开始的任务，而第二个被添加的任务将第二个开始，如此直到队列的终点。
 
#概念：调度队列(dispath queue)


所有的调度队列（`dispatch queues`）自身都是线程安全的，我们能从多个线程并行的访问它们。 `GCD`的优点是显而易见的。我们需要了解调度队列如何我们的代码的不同部分提供线程安全，以决定使用何种队列，在哪个线程上执行等。
 
`GCD`将长期运行的任务拆分成多个工作单元，并将这些单元添加到`dispath queue`中，系统会管理这些`dispath queue`，为我们在多个线程上执行工作单元，我们不需要手动启动和管理后台线程。

系统提供了许多预定义的`dispath queue`，包括始终在主线程上执行工作的`dispath queue`。我们可以创建自己的`dispath queue`，而且可以创建任意多个。`GCD`的`dispath queue`严格遵循`FIFO`(先进先出)原则，添加到`dispath queue`的工作任务将按照加入`dispath queue`的顺序启动。

#概念：串行（Serial）

我们在学习操作系统这门课程的时候，经常会提到串行。我们使用`GCD`，也会用到串行的概念。

所谓串行（Serial）执行，指同一时间每次只能执行一个任务。

#概念：并发（Concurrent）

说到串行，自然会想到并发。在操作系统这门课程中，这个概念是非常重要的。

所谓并发（Concurrent），指同一时间可以同时执行多个任务。

#概念：死锁（Deadlock）

操作系统这门课程中对死锁的介绍说明有很多。在实际开发中，也经常遇到死锁的问题。

所谓死锁（Deadlock）是指它们都卡住了，并等待对方完成或执行其它操作。第一个不能完成是因为它在等待第二个的完成。但第二个也不能完成，因为它在等待第一个的完成。

#概念：线程安全（Thread Safe）

还记得我们在写单例的时候都加了哪些代码吗？我们应该知道，既然要声明为单例，说明这是共享资源区，就会存在竞态条件，因此，我们必须保证只创建一次。

像这样添加了线程锁的：

```
@synchronized(<#token#>) {
  <#statements#>
}
```

还有这样用于创建单例的，以确保只执行一次：

```
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
  <#code to be executed once#>
});
```

#创建和管理dispatch queue

##1.获取全局并发调度队列

并发的调度队列可以同时并行地执行多个任务，但是并发队列也是队列，因此同样遵循着`FIFO`的原则来启动任务。因为并发执行任务与系统有关，其同时执行任务的数量是由系统根据应用和系统动态变化决定的。

现在`iOS`系统，为每个应用提供了四种并发的全局共享的调度队列，其区别在于优先级不一样。

```
/*
 * The global concurrent queues may still be identified by their priority,
 * which map to the following QOS classes:
 * 
 *  - DISPATCH_QUEUE_PRIORITY_HIGH:         QOS_CLASS_USER_INITIATED
 *  - DISPATCH_QUEUE_PRIORITY_DEFAULT:      QOS_CLASS_DEFAULT
 *  - DISPATCH_QUEUE_PRIORITY_LOW:          QOS_CLASS_UTILITY
 *  - DISPATCH_QUEUE_PRIORITY_BACKGROUND:   QOS_CLASS_BACKGROUND
 */  
```

我们不需要创建它，只需要直接获取就可以了，因为这是系统为我们提供的，而且这个还是全局共享的：

```
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```

第一个参数为优先级，就是上面提供的这四种。第二个参数没有使用到，这个参数是预留的，使用0即可，看官方说明：

```
 * @param flags
 * Reserved for future use. Passing any value other than zero may result in
 * a NULL return value.
```

`flags`就是第二个参数，也就是为未来预留的参数。看看苹果想得真够远的，为未来预留~~。

**注意：**虽然`dispatch queue`是引用计数的对象,但我们不需要`retain`和`release`这个全局的并发`queue`。因为这些`queue`对应用是全局的,`retain`和`release`调用会被忽略。


##2.创建串行调度队列

当任务需要按特定的顺序执行时,就需要使用串行调度队列（Dispatch Queue）,串行调度队列每次只能执行一个任务。

我们可以使用串行队列替代锁,保护共享资源等。和锁不一样的是,串行队列确保任务按指定的顺序执行，而且只要你异步地提交任务到串行队列,就永远不会产生死锁。

我们可以手动创建和管理串行队列，且可以创建很多个，但是我们不要创建很多个串行队列来执行很多的任务，当需要执行大量的任务时，应该交给全局并发队列来完成。从操作系统方面思考，虽然允许应用创建很多个串行队列，但是其优先级永远不会比系统级的高，因此当任务很多时，所要求的资源未必就可以提供。所以，任务量大时，应该交给系统提供的全局队列来完成才是最佳的。

使用下面的方法来创建串行队列，其中第一个参数是队列的名称，通常使用公司的反域名，如com.company.project。第二个参数是队列相关属性，通常都传`NULL`：

```
dispatch_queue_t sequalQueue = dispatch_queue_create("com.huangyibiao.helloworld", NULL);
```

##3.获取公共队列

应用提供了几下几种获取公共队列的方法：

* `dispatch_get_current_queue`：在`iOS 6.0`之后已经废弃，用于获取当前正在执行任务的队列，主要用于调试
* `dispatch_get_main_queue`： 最常用的，用于获取应用主线程关联的***串行调度队列***
* `dispatch_get_global_queue`：最常用的，用于获取应用全局共享的***并发队列***

对于后面这两个分别获取主线程的串行队列和获取应用全局共享的并发队列是非常常用的，当我们需要开一个线程并发地异步执行任务时，我们就会放到全局队列中。当我们在异步执行完成时，通常需要回到主线程更新`UI`显示。

##4.调度队列（Dispatch Queue）的内存管理

调度队列，即`Dispatch Queue`与其它类型的`dispatch`对象是引用计数的数据类型。当创建一个串行`dispatch queue`时,初始引用计数为`1`,我们可用`dispatch_retain`和`dispatch_release`函数来增加和减少引用计数。当引用计数为`0`时,系统会异步地销毁这个`queue`。

以上是对于普通创建的调度队列有用，但对于系统本身提供的全局并发队列和主线程串行队列则不需要我们手动内管其内存，系统会自动管理。

在使用全局并发队列时，我们只通过`dispatch_get_global_queue`方法来获取即可，我们不需要管理其引用。
在使用主线程串行队列时，我们只通过`dispatch_get_main_queue`方法来获取即可，我们也不需要管理其内存问题。

#添加任务到调度队列


要想让调度队列执行任务，那么我们就需要将任务添加到适当的调度队列中。在实际`iOS`开发中，我们通常配合`block`的使用，将任务封装到一个`block`中。

我们可以异步或者同步添加任务到队列中，但是我们应该尽可能地使用`dispatch_async`或`dispatch_async_f`。前者是提交一个`block`任务到队列中，后者是提供一个函数任务到队列中。基本上都是直接使用`dispatch_async`提交一个`block`到队列中，这代码写起来更加地简洁。

当然，我们也可以同步添加任务。有时候我们可能希望同步地调度任务,以避免竞争条件或其它同步错误。使用`dispatch_sync`或`dispatch_sync_f`函数同步地添加任务到Queue,这两个函数会阻塞当前调用线程,直到相应任务完成执行。在实际开发中，当需要同步执行任务时，大多是直接使用`dispatch_sync`这个提交`block`任务的方法，使用起来更简洁。

**注意：**当队列中有任务正在同步执行时，我们不能使用`dispatch_sync`或`dispatch_sync_f`同步调度新任务到当前正在执行的`queue`中。对于串行`queue`肯定会导致死锁，而对于并发`queue`也应该避免这么使用。原来我接手的项目中，有一个同步任务正在执行数据库操作，可是当我也需要操作数据时，调用其所提供的`api`，使用`dispatch_sync`将我的任务添加到队列中，结果导致了死锁，每次都`crash`。

为什么尽可能地添加异步执行的任务呢？因此同步任务会阻塞主线程，很可能导致事件响应不了。

我们看看如何简单地创建队列、异步、同步任务添加到队列：

```
dispatch_queue_t queue = dispatch_queue_create("com.huangyibiao.helloworld", NULL);

dispatch_async(queue, ^{  
    NSLog(@"开启了一个异步任务，当前线程：%@", [NSThread currentThread]);  
});  
  
dispatch_sync(queue, ^{  
    NSLog(@"开启了一个同步任务，当前线程：%@", [NSThread currentThread]);  
});  

// MRC下才能调用，对于ARC就不能添加这行代码了。
dispatch_release(queue);
```

由于这个串行调度队列是我们自己创建的，我们需要管理其内存。不过在实际开发中，使用自己创建创建的方式是比较少见的，通常都是直接使用系统为每个应用提供的全局共享并发队列异步执行任务，然后使用主线程串行队列更新界面。

#控制并发数

太多并发是会带来很多的风险的。在实际开发中，并不是并发数越多就越好，往往是需要控制其并发数量的。比如，在处理网络请求并发数时，通常会设置限制最大并发数为4左右。当并发数量大了，开销也会很大。学过操作系统应该清楚，并发量大了，临界资源访问操作就很难控制，控制不好就会导致死锁等。当我们需要执行循环异步处理任务时，可以考虑使用`dispatch_apply`来替代。请看下一节！

#并发地循环迭代任务

如果迭代执行的任务与其它迭代任务是独立无关的,而且循环迭代执行顺序也无关紧要的话,我们可以调用`dispatch_apply`或`dispatch_apply_f`函数来替换循环。前者是提交`block`任务，后者是提交函数任务到队列中。比如，我们需要上传多张图片，这些图片的上传是互不干扰的，迭代执行的顺序是不重要的，那么我们就可以使用`dispatch_apply`来替换掉`for`循环。

下面代码使用`dispatch_apply`替换了`for`循环,所传递的`block`必须包含一个`size_t`类型的参数,用来标识当前循环迭代。第一次迭代这个参数值为`0`,最后一次值为`count - 1`：

```
// 获得全局并发queue
dispatch_queue_t gqueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
size_t gcount = 10;
dispatch_apply(gcount, gqueue, ^(size_t i) {
	[self uploadImageWithIndex:(NSUInteger)(i)];
});

- (void)uploadImageWithIndex:(NSUInteger)imageIndex {
  NSLog(@"上传索引为%lu的图片", imageIndex);
}
```

打印结果说明顺序是不确定的，可看得出来这是并发执行的：

```
2015-11-24 00:06:11.692 TestGCD[27714:2678984] 上传索引为0的图片
2015-11-24 00:06:11.692 TestGCD[27714:2679067] 上传索引为3的图片
2015-11-24 00:06:11.692 TestGCD[27714:2678984] 上传索引为4的图片
2015-11-24 00:06:11.692 TestGCD[27714:2679064] 上传索引为2的图片
2015-11-24 00:06:11.692 TestGCD[27714:2678984] 上传索引为6的图片
2015-11-24 00:06:11.692 TestGCD[27714:2679065] 上传索引为1的图片
2015-11-24 00:06:11.693 TestGCD[27714:2678984] 上传索引为8的图片
2015-11-24 00:06:11.692 TestGCD[27714:2679067] 上传索引为5的图片
2015-11-24 00:06:11.693 TestGCD[27714:2679064] 上传索引为7的图片
2015-11-24 00:06:11.693 TestGCD[27714:2679065] 上传索引为9的图片
```

**注意**：`dispatch_apply`或`dispatch_apply_f`函数也是在所有迭代完成之后才会返回，因此这两个函数会阻塞当前线程。当我们在主线程中使用时，一定要小心，很容易造成事件无法响应，所以如果循环代码需要一定的时间执行,可考虑在另一个线程中调用这两个函数。如果所传递的参数是串行`queue`，而且正是执行当前代码的`queue`,就会产生死锁。

#主线程中执行任务

看看下面很常用的异步下载图片的代码：

```
// 异步下载图片  
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
    NSURL *url = [NSURL URLWithString:@"图片的URL"];  
    UIImage *image = [UIImage imageWithData:[NSData dataWithContentsOfURL:url]];  
      
    // 回到主线程显示图片  
    dispatch_async(dispatch_get_main_queue(), ^{  
        self.imageView.image = image;  
    });  
}); 
```

这里先将异步下载图片的任务放到`dispatch_get_global_queue`全局共享并发队列中执行，在完成以后，需要放在`dispatch_get_main_queue`回到主线程更新`UI`。

#暂停和继续queue

我们可以使用·dispatch_suspend·函数暂停一个`queue`以阻止它执行`block`对象;使用`dispatch_resume`函数继续`dispatch queue`。调用`dispatch_suspend`会增加`queue`的引用计数,调用`dispatch_resume`则减少`queue`的引用计数。当引用计数大于`0`时,`queue`就保持挂起状态。因此你必须对应地调用`dispatch_suspend`和`dispatch_resume`函数。挂起和继续是异步的,而且只在执行`block`之间生效，挂起一个`queue`不会导致正在执行的`block`停止。

```
dispatch_suspend(gqueue);
dispatch_resume(gqueue);
```

**注意：**`dispatch_suspend`和`dispatch_resume`是成对出现的。

#调度组（Dispatch Group）的使用


当我们需要下载多张图片并且图片要求这几张图片都下载完成以后才能更新UI，那么这种情况下，我们就需要使用`dispatch_group_t`来完成了。

像这样：

```
// 异步下载图片  
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
    // 下载第一张图片  
    UIImage *image1 = [self imageWithURLString:url1];  
      
    // 下载第二张图片  
    UIImage *image2 = [self imageWithURLString:url2];  
      
    // 回到主线程显示图片  
    dispatch_async(dispatch_get_main_queue(), ^{  
        self.imageView1.image = image1;  
        self.imageView2.image = image2;  
    });  
}); 
```

这段代码是不能做到的，但是，我们还是有办法做到的。`dispatch_group_t`就是很好的选择。对于调度组，所添加的任务可以是同步的，也可以是异步的，在最近任务全部完成后都会有回调。

首先，我们通过`dispatch_group_create`创建一个组，然后通过`dispatch_group_async`将任务分别添加到该组中。当组中的所有任务都完成以后，我们可以通过`dispatch_group_notify`得到回调，然后在主线程更新UI。

代码写法像下面这样：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// 异步下载图片
dispatch_async(queue, ^{
  // 创建一个组
  dispatch_group_t group = dispatch_group_create();
  
  __block UIImage *image1 = nil;
  __block UIImage *image2 = nil;
  
  // 分别将任务添加到组中
  dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    image1 = [self downloadImage:url1];
  });
  
  dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    image2 = [self downloadImage:url2];
  });
  
  // 等待组中的任务执行完毕,回到主线程执行block回调
  dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    self.imageView1.image = image1;
    self.imageView2.image = image2;
  });
});
```

#延迟执行

我们常见的延迟执行方法有：

方法一：使用`NSObject`的`api`，同步执行：

```
[self performSelector:@selector(myFunction) withObject:nil afterDelay:5.0];
```

方法二：使用`NSTimer`定时器，不过这种方法没必要。

方法三：使用`dispatch_after`方法异步延迟执行：

```
CGFloat time = 5.0f;
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(time * NSEC_PER_SEC)),
             dispatch_get_main_queue(), ^{
	// time秒后异步执行这里的代码...
    
});
```

#结尾

对于在实际开发中常用的差不多全了，其它比较偏的`API`就不说了，在开发中比较少用。文章中所描述的不代表一定正确，因此若大家在阅读时发现任何可疑的地方，请务必在评论中指出！
