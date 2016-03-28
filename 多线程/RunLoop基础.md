#序言

RunLoop一直是比较高级而又比较神秘的技术，一直以来都没有深入去阅读过苹果给出的官方文档。本篇文章就讲讲苹果官方文档中所介绍的RunLoop，再加上其开源性，让我们一起深入去研究其特性及与线程的关系。

本篇主要是阅读官方文档所总结下来的知识点，有很大一部分是翻译过来的。

#什么是Run Loops

Run Loops是与线程想关联的基础部分。一个Run Loop就是事件处理循环，它是用来调度和协调接收到的事件处理。使用Run Loop的目的，就是使得线程有工作需要做时可以忙碌起来，而当没有事可做时，又可以使得线程睡眠。

Run Loop管理不都是自动的。我们必须手动设计线程代码，在合适的时候来启动Run Loop，并回应到来的事件。Cocoa和Core Foundation都提供了run loop对象来帮助我们配置和管理线程的run loop。我们的应用没有必要显式地创建这些对象；每个线程，包括应用程序的主线程，都有一个与之关联的run loop。只有子线程才需要显式地运行其run loop。App会将自动配置和来运行主线程的run loop的任务作为应用程序启动处理的一部分。

对于想要更深入地了解Run Loop Objects，阅读[NSRunLoop Class Reference](http://www.gnustep.org/resources/documentation/Developer/Base/Reference/NSRunLoop.html)、[CFRunLoop Reference](https://developer.apple.com/library/ios/documentation/CoreFoundation/Reference/CFRunLoopRef/index.html#//apple_ref/doc/uid/20001441)

#一个Run Loop的结构

Run Loop就像它的名字一样，它使得线程进入事件循环，能对到来的事件启动事件处理。你的代码中提供了流程控制说一句来实现run loop实实在在的循环部分，换句话说，你的代码提供了while或者for循环来驱动run loop。在你的循环中，你使用run loop对象在事件到达时，运行事件处理的代码并调起已安装的处理程序。

**Run Loop接收来自两种不同类型的源(sources)的事件**：

* 输入源：异步传递事件，通常是来自不同的线程或不同的应用的消息。输入源异步传递事件到对应的处理程序和在线程关联的NSRunLoop对象调起runUntilDate:方法来退出事件处理。
* Timer源：同步地传递事件，发生在每个定时器调用或周期性地调用。Timer源传递事件到他们的处理程序，但是不会调用run loop来退出处理。

这两种源在事件到达时都使用应用程序特定的处理程序来处理事件。

如下图所示，展示了run loop和不同的源的概述结构。

![image](http://www.henishuo.com/wp-content/uploads/2016/03/runloop.jpg)

除了处理输入源之外，run loops还发出关于run loop行为的通知。我们可以注册成为run loop的观察者，就可以接收这些通知和使用它在线程上做一些额外处理。我们可以使用Core Foundation在对应的线程上注册成为run loop的观察者。

#Run Loop Modes

Run Loop模式是一个监视输入源和定时器的集合和注册成为run loop的观察者的集合。每次要运行run loop，都需要显示或隐式地指定某种运行的mode。只有与这种指定的mode关联的源才会被监视和允许传递他们的事件，同样地，只有与这种模式关联的观察者都会收到run loop行为变化的通知。与其它模式想关联的源，直到随后在合适的模式通过循环后，都会接收到新的事件（比如，将timer加入run loop default模式下，当滚动时，timer不会收到回调，直到停止滚动回到default模式下）。

在我们的代码中，我们通过名称来唯一标识mode。在Cocoa和Core Foundation中都定义了default模式和几个常用的模式，都是通过字符串名称来指定。我们也可以自定义模式，但是我们需要手动添加至少一个input source/timers/observers。

我们可以通过使用mode来过滤掉我们不希望接收到来自不想要的通过run loop的源。大部分情况下，我们都是使用系统定义的default模式。对于子线程，我们可以使用自定义模式在关键性操作时阻止低优先级的源传递事件。

**注意：**Modes是通过事件源来区分，而不是事件类型来区分。比如说，我们不能使用mode来匹配只有mouse-down事件或者只有键盘事件。我们可以使用modes来监听不同系统的端口，临时挂起定时器，甚至改变正在被监视的sources和run loop观察者。

Mode              | 名称             | 说明
|--------------|------------------|----------------|
Default        | NSDefaultRunLoopMode(Cocoa) kCFRunLoopDefaultMode (Core Foundation) | 最常用的默认模式|
Connection | NSConnectionReplyMode (Cocoa)| 是针对NSConnection的，我们几乎不会使用到|
Modal | NSModalPanelRunLoopMode (Cocoa) | Cocoa uses this mode to identify events intended for modal panels.
Event tracking | NSEventTrackingRunLoopMode (Cocoa) | 在滚动时会进入此mode|
Common modes | NSRunLoopCommonModes (Cocoa) kCFRunLoopCommonModes (Core Foundation) | 公共模式，在任何模式下都可以接收|

#Input Sources

输入源异步传递事件到你的线程。事件的源由输入源的类型来决定，也就是两种源中的其中一种：

* Port-based：基于端口号的输入源监听应用程序的Mach端口。
* Custom Input Sources：自定义输入源监听自定义的事件源。

系统通常实现了这两种输入源。唯一的不同点是它们是如何被发出信号的。port-based源是由内核（kernel）自动发出信号，而custom sources必须手动从其它线程发出信号。

当我们创建输入源时，可以指定mode。Modes会影响任何时刻被监视的输入源。大部分情况下，我们都让run loop在default mode下运行，但是也可以指定自定义的mode。如果一个输入源不是当前所监视的model，它所产生的任何事件都会被保留直接进入正常的mode。

###Port-Based Sources

Cocoa和Core Foundation提供了内建支持，可以使用与port相关的对象和函数来创建基于端口的输入源。举个例子，在Cocoa中永远不需要手动创建输入源。我们只需要简单地创建一个port对象和使用NSPort的方法。port对象为我们处理所需要的输入源的创建和配置。

在Core Foundation中，我们必须手动创建port和source。在这两种情况下，我们可以使用与port opaque type关联的函数（CFMessagePortRef, or CFSocketRef) 来创建合适的对象。

###Custom Input Sources

在Core Foundation中，要创建自定义输入源，我们必须使用与CFRunLoopSourceRef关联的函数。我们配置自定义输入源可以使用几个回调函数。Core Foundation会在不同点回调这些函数来配置source，处理任何到达的事件和销毁已从run loop移除的source。

除了定义在事件到达时自定义源的行为之外，我们也必须定义事件传递机制。这部分源运行在单独的线程，负责提供输入源的数据，当数据准备好可以处理时，signaling（通知相关线程）这个消息。事件传递机制是我们自己来决定，但是不需要过于复杂。

####Cocoa Perform Selector Source

除了基于端口的源之外，Cocoa还定义了自定义输入源允许我们在任意线程上执行selector。就像port-based源一样，执行selector请求会在目标线程上序列化，以减少在同一个线程中出现多个方法同步执行的问题。与port-based源不同的是，执行selector源在执行完毕后会自动将自己从run loop中移除。

当执行在其它线程执行selector时，目标线程必须要有运行的run loop。当我们创建线程时，这意味着直到启动了run loop都会显式地执行selector代码。

Run Loop每次经过一个循环，就会处理队列中所有的selector，而不仅仅是处理一个。

方法 | 说明
|-----------------|-------------|
|  performSelectorOnMainThread:withObject:waitUntilDone: performSelectorOnMainThread:withObject:waitUntilDone:modes:               |   在主线程在下一个循环周期执行selector，会阻塞主线程，直到Selector执行完毕        |
|   performSelector:onThread:withObject:waitUntilDone: performSelector:onThread:withObject:waitUntilDone:modes:              |      在指定的线程执行selector，会阻塞指定的线程       |
|     performSelector:withObject:afterDelay: performSelector:withObject:afterDelay:inModes:            |              在当前线程执行selector，会在run loop进入下一个循环周期，在延迟指定的时间后，先执行。由于它要等待run loop进入下一个循环周期才能执行selector，这些方法提供了从当前执行的代码中自动最小延迟。队列中的selector会一个一个顺序地执行。
|  cancelPreviousPerformRequestsWithTarget: cancelPreviousPerformRequestsWithTarget:selector:object:               |   取消通过performSelector:withObject:afterDelay: or performSelector:withObject:afterDelay:inModes:执行的selector          |

#Timer Sources

Timer源在未来设定的时间会同步地传递事件到你的线程。Timers是线程通知自己去做一些事情的一种方式。比如说，搜索框可以使用定时器来初始化在一定时间就自动搜索，以便提供更多地联想词给用户。

尽管它发送基于时间的通知，但定时器并不是一种实时的机制。像输入源一样，定时器只有与run loop的mode一样才会发送通知。如果timer在run loop中并不是所被监视的mode，它不会触发定时器，直到run loop的mode与timer所支持的mode一样。

同样地，如果run loop正在处理中，timer已经fire了，这时候会被中断，直到下一次通过run loop才会调志处理程序。如果run loop已经不再运行了，则timer永远不会再fire。

我们可以配置timer只产生事件一次或者重复产生。重复的timer会自动根据调度的firing time自动调度，而不是真实的firing time。比如说，如果一个timer在特定的时间调度，然后每5秒重复一次。如果firing time被延迟导致缺少一或多次调用，那么timer在缺失的周期中只会调用一次。

#Run Loop Observers

与sources在适当时机异步或同步发出事件不同，observers在run loop本身执行期间，会在特定的地方发出。你可能需要到run loop observers去准备线程处理特定的事件或者在进入睡眠之前。我们可以通过以下事件来关联run loop observers:

* 进入run loop
* run loop将要处理timer
* run loop将要处理输入源
* run loop将要进入睡眠
* run loop被唤醒，但是还没有处理事件
* 退出run loop

我们可以通过Core Foundation来添加run loop observers。要创建run loop observer,可以通过CFRunLoopObserverRef来创建新的实例。这个类型会跟踪你所定义的回调函数和所感兴趣的活动。

与timers类型，run-loop observers可以使用一次或者重复多次。一次性的observer会在fire之后自动从run loop移除，而重复性的observer会继续持有。

#The Run Loop Sequence Of Events

本小节讲的是RunLoop事件顺序。每次运行它，你的线程的run loop处理待处理的事件和给所有attached observers发出通知。处理的顺序如下：

1. 通知observers run loop已经进入
2. 通知observers timers准备要fire
3. 通知observers有不是基于port-based的输入源即将要fire
4. fire任何已经准备好的non-port-based输入源
5. 如果port-based输入源准备好且等待fire，则立即处理这个事件。然后进入步骤9
6. 通知observers线程即将进入睡眠 
7. 让线程进入睡眠，直到以下任何一种事件到达：
   * port-based输入源有事件到达
   * timer fire
   * run loop超时
   * run loop被显式唤醒
8. 通知observers线程被唤醒
9. 处理待处理的事件：
   * 如果用户定义的timer fired了，处理timer事件并重新启动循环。进入步骤2
   * 如果输入源fired了，则传递事件
   * 如果run loop被显式唤醒，但是又未超时，则重启循环，进入步骤2
10. 通知observers run loop退出

由于observer对timer和输入源的通知会在事件真正发生之前被传递，这样就产生了间隙。如果这个间隙是很关键的，那么我们可以通过使用sleep和awake-from-sleep通知来帮助我们纠正这个时间间隔问题。

#When Would You Use A Run Loop?

什么时候应该使用run loop呢？

只有当我们需要创建子线程的时候，才会需要到显示地运行run loop。应用程序的主线程的run loop是应用启动的基础任务，在启动时就会自动启动run loop。所以我们不需要手动启动主线程的run loop。

对于子线程，我们需要确定线程是否需要run loop，如果需要，则配置它并启动它。我们并不问题需要启动run loop的。比如说，如果我们开一个子线程去执行一些长时间的和预先决定的任务，我们可能不需要启动run loop。Run loop是用于那么需要在线程中有更多地交互的场景。比如说，我们会在下面的任何一种场景中需要开启run loop:

* 使用端口源或者自定义输入源与其它线程通信
* 在线程中使用定时器
* 使用Cocoa中的任何performSelector...方法
* 保持线程来执行周期性的任务

#Using Run Loop Objects

Run Loop对象给添加输入源、定时器和观察者到run loop提供了主接口。每个线程都有一个单独的run loop与之关联（对于子线程，若没有调用过任何获取run loop的方法是不会有run loop的，只有调用过，才会创建或者直接使用）。

在Cocoa中，通过NSRunLoop来创建实例，在low-level应用中，可以使用CFRunLoopRef类型，它是指针。

###Getting A Run Loop Object

通过以下两种方式来获取run loop对象：

* 在Cocoa中，使用[NSRunLoop currentRunLoop]获取
* 使用CFRunLoopGetCurrent()函数获取

###配置RunLoop

在子线程运行run loop之前，你必须至少添加一种输入源或者定时器。如果run loop没有任何的源需要监视，它就会立刻退出。

除了添加sources之外，你还可以添加观察者来检测runloop不同的执行状态。要添加观察者，可以使用CFRunLoopObserverRef指针类型和使用CFRunLoopAddObserver函数来添加到run loop中。我们只能通过Core Foundation来创建run loop观察者，即使是Cocoa应用。

下面这段代码展示主线程如何添加观察者到run loop以及如何创建run loop观察者：

```
- (void)threadMain {
    // The application uses garbage collection, so no autorelease pool is needed.
    NSRunLoop* myRunLoop = [NSRunLoop currentRunLoop];
 
    // Create a run loop observer and attach it to the run loop.
    CFRunLoopObserverContext  context = {0, self, NULL, NULL, NULL};
    CFRunLoopObserverRef    observer = CFRunLoopObserverCreate(kCFAllocatorDefault,
            kCFRunLoopAllActivities, YES, 0, &myRunLoopObserver, &context);
 
    if (observer) {
        CFRunLoopRef    cfLoop = [myRunLoop getCFRunLoop];
        CFRunLoopAddObserver(cfLoop, observer, kCFRunLoopDefaultMode);
    }
 
    // Create and schedule the timer.
    [NSTimer scheduledTimerWithTimeInterval:0.1 target:self
                selector:@selector(doFireTimer:) userInfo:nil repeats:YES];
 
    NSInteger    loopCount = 10;
    do {
        // Run the run loop 10 times to let the timer fire.
        [myRunLoop runUntilDate:[NSDate dateWithTimeIntervalSinceNow:1]];
        loopCount--;
    } while (loopCount);
}
```

在为长时间存在的线程配置run-loop时，最好添加至少一个输入源来接收事件。尽管我们可以只用timer源，但是一旦timer调用后，经常会被invalidate，这会导致run loop退出。

#Starting the Run Loop

只有子线程才有可能需要启动run loop。Run loop必须至少有一种输入源或者timer源来监视。如果没有任何源，则run loop会退出。

下面的几种方式可以启动run loop：

* 无条件地：无条件进入run loop是最简单的方式，但也是最不希望这么做的，因为这样会导致run loop会进入永久地循环。可以添加、删除输入源和timer源，但是只能通过kill掉run loop才能停止。而且还不能使用自定义mode。
* 限时：与无条件运行run loop不同，最好是给run loop添加一个超时时间。
* 在特定的mode：除了添加超时时间，还可以指定mode。

下面是运行run loop的一段代码：

```
- (void)skeletonThreadMain {
    // Set up an autorelease pool here if not using garbage collection.
    BOOL done = NO;
 
    // Add your sources or timers to the run loop and do any other setup.
 
    do {
        // Start the run loop but return after each source is handled.
        SInt32    result = CFRunLoopRunInMode(kCFRunLoopDefaultMode, 10, YES);
 
        // If a source explicitly stopped the run loop, or if there are no
        // sources or timers, go ahead and exit.
        if ((result == kCFRunLoopRunStopped) || (result == kCFRunLoopRunFinished))
            done = YES;
 
        // Check for any other exit conditions here and set the
        // done variable as needed.
    } while (!done);
 
    // Clean up code here. Be sure to release any allocated autorelease pools.
}
```

#Exiting the Run Loop

有两种方法使run loop在处理事件之前，退出run loop：

* 给run loop设定超时时间
* 告诉run loop要stop

设定超时时间是比较推荐的。我们可以通过CFRunLoopStop函数来停止run loop。

#Thread Safety and Run Loop Objects

Core Foundation中的Run Loop API是线程安全的（以CF开头的API），而Cocoa中的NSRunLoop不是线程安全的。

#Configuring Run Loop Sources

下面是展示如何配置不同类型的输入源。

###Defining a Custom Input Source

创建自定义输入源涉及到以下部分：

* 想要处理的输入源的信息
* 让感兴趣的客户端知道如何联系输入源的调度程序 
* 执行任何客户端发送的请求处理程序 
* 使输入源失效的取消程序 

####Timer Source

```
NSRunLoop* myRunLoop = [NSRunLoop currentRunLoop];
 
// Create and schedule the first timer.
NSDate* futureDate = [NSDate dateWithTimeIntervalSinceNow:1.0];
NSTimer* myTimer = [[NSTimer alloc] initWithFireDate:futureDate
                        interval:0.1
                        target:self
                        selector:@selector(myDoFireTimer1:)
                        userInfo:nil
[myRunLoop addTimer:myTimer forMode:NSDefaultRunLoopMode];       
```

或者使用Core Foundation：

```
CFRunLoopRef runLoop = CFRunLoopGetCurrent();
CFRunLoopTimerContext context = {0, NULL, NULL, NULL, NULL};
CFRunLoopTimerRef timer = CFRunLoopTimerCreate(kCFAllocatorDefault, 0.1, 0.3, 0, 0,
                                        &myCFTimerCallback, &context);
 
CFRunLoopAddTimer(runLoop, timer, kCFRunLoopCommonModes);
```

#最后

本篇文章主要是官方文档的部分翻译版本，不过有很多无关的都省略了，而且有都转换成笔者的语言来表达出来，如果读不懂，最好还是去看官方文档吧。毕竟，英文与中文翻译不管怎么翻译都存在很大的问题。

#疑问

官方文档中提到，每个线程都有一个run loop与之关联。但是实质上子线程在没有访问过run loop时，是不存在的。当访问时，若不存在则创建run loop并放到全局数组中。笔者阅读过CFRunLoop.c的源代码，所以当有面试官问到这个问题时，一定要注意哦！

#参考

[官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW47)