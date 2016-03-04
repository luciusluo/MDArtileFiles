#引言

该文章与runtime相关，开始并没打算写，因为大神们写了好多runtime的文章，分析的都很全面、很深刻，再写也就是班门弄斧。但还是写了，因为我在看一个东西的时候偶尔发现了object_getClass(obj)与[OBJ class]返回的指针不同，感觉非常奇怪，因为它颠覆了我们对runtime中类结构模型的认识，后来在网上找了相关问题的答案，发现并没有，所以打算写一篇文章来和大家说说这个问题，我会讲一点点runtime，但不会系统的讲，目的就是为了能让大家理解我说的问题就可以了。

#runtime讲哪些东西？

runtime是很宽泛的概念，通常我们在讲runtime的时候大多侧重以下两方面：

1. 基于Class、Object的结构模型讲解。

2. 实践中基于runtime的api应用，这里讲的最多的就是基于method swizzling来实现AOP。

我遇到的问题就是与Class、Object的结构模型相冲突的，所以我们今天要讨论的是前者。

runtime是开源的，大家要想了解细节还是要大概的看看[源码](https://opensource.apple.com/tarballs/objc4/)

使用runtime的api要引入头文件：

```
#import <objc/runtime.h>
```

#先从runtime源码说起

我们先从runtime源码开始了解一些本质上的东西。

```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```

###类的本质就是结构

```
typedef struct objc_class *Class;
```

###Class的本质就是代表类的结构体的指针

```
struct objc_object {
     Class isa
}
typedef struct objc_object *id
```

###id也是一个结构体指针

```
@interface NSObject <NSObject> {
     Class isa
}
```

###NSObject本质上也是结构

这些都是runtime中的源码，没什么好说的，拿出来就是要帮助大家加深对runtime的理解，下面来说说objc_class结构里的isa和super_class之间的关系，然后深入理解一下Class，这与我遇到的问题是有关系的。其实runtime还有很多重要的概念，比如SEL、IMP、Method等...由于和我们今天讨论的问题没有关系，这里不再展开说明了，这几个概念都与消息转发关系密切，感兴趣大家可以看源码。

#runtime经典图分析

![image](http://www.henishuo.com/wp-content/uploads/2015/12/inherit.png)

相信了解runtime的朋友对这张图不陌生，我们在这里再看看这张图说明了什么：

###1. 横向看：

实例是对象，类也是对象(类对象)，meta类也是对象(原类对象)

这是很重要的一点，希望大家理解，我们这里忽略上下结构，先看左右结构，从左到右的指向就是之前介绍的runtime源码中objc_class结构里isa的指向，Instance指的是我们创建的对象，Subclass（class）就是创建该对象的那个类，注意：创建对象的类本身也是对象，称为类对象，类对象中存放的是描述实例相关的信息，例如实例的成员变量，实例方法。

类对象里的isa指针指向Subclass（meta），Subclass（meta）也是一个对象，是原类对象，原类对象中存放的是描述类相关的信息，例如类方法，在这一过程中，isa的两次指向很像很像，大家注意理解。

类本身作为一个对象这件事情其实还是值得我们花时间来想想的，我当时是在想NSTimer自释放问题的时候想到类对象的，从而才发现了本文讨论的问题。

由于Subclass（meta）在横向上已经没有可以指向的对象了，所以他们的isa指针统一指向纵向（继承关系）上的根meta class。而根meta class的isa则指向自己，我们后面会在代码中把这些结论性的东西验证了。

###2.纵向看：

superclass指针很容易理解，就是按照继承关系向上指的，一直到继承链的最上方，值得说的是Root class（class）的superclass指向是nil，Root class（meta）的superclass指向它的Root class （class），这个注意一下。

#从代码上理解上面的图

这里介绍几个runtime中的方法，还是看runtime源码：

```
Class object_getClass(id obj)
{
    if (obj) return obj->getIsa();
    else return Nil;
}
```

object_getClass()就是顺着isa的指向链找到对应的类，我们一会要验证这个isa的指向链是否与上面图中是一致的，就是用这个方法。

与之相关还有一个方法：object_setClass()，我们可以用该方法，简单的来看一下runtime的强大，它可以动态改变类。首先要知道我们在NSLog的时候用的%@打印对象的时候其实是调用该类的description方法，而且我们还知道，NSArray对象的description会把每个array里的元素都打印出来，而NSObject对象的description就仅仅打印类名和指针，下面通过一小段代码看看runtime的强大。

```
NSArray *tempObj = @[@"hello", @"erliangzi"];
NSLog(@"tempObj:%@", tempObj);
object_setClass(tempObj, [NSObject class]);
NSLog(@"tempObj:%@", tempObj);
2016-02-02 23:56:22.905 TimerDemo[1104:54722] tempObj:(
    hello,
    erliangzi
)
2016-02-02 23:56:22.906 TimerDemo[1104:54722] tempObj:<NSObject: 0x7ff580d06140>
```

是不是很不可思议？runtime就是这么强大！

```
+ (Class)class {
    return self;
}

- (Class)class {
    return object_getClass(self);
}
```

这是NSObject类里实例方法class与类方法class的实现，这里再强调一下：类方法是在meta class里的，类方法就是把自己返回，而实例方法中是返回实例isa的类，我们要验证这个isa的指向链的时候不能用这种方法，千万记住，为什么，一会说明。

看代码：

```
#import <objc/runtime.h>
#import "Person.h"


Person *obj = [Person new];
NSLog(@"instance         :%p", obj);
NSLog(@"class            :%p", object_getClass(obj));
NSLog(@"meta class       :%p", object_getClass(object_getClass(obj)));
NSLog(@"root meta        :%p", object_getClass(object_getClass(object_getClass(obj))));
NSLog(@"root meta's meta :%p", object_getClass(object_getClass(object_getClass(object_getClass(obj)))));
NSLog(@"---------------------------------------------");
NSLog(@"class            :%p", [obj class]);
NSLog(@"meta class       :%p", [[obj class] class]);
NSLog(@"root meta        :%p", [[[obj class] class] class]);
NSLog(@"root meta's meta :%p", [[[[obj class] class] class] class]);
Log输出：

2016-02-02 18:06:11.443 TimerDemo[1718:248402] instance         :0x7fc792530f20
2016-02-02 18:06:11.444 TimerDemo[1718:248402] class            :0x10ae0e178
2016-02-02 18:06:11.444 TimerDemo[1718:248402] meta class       :0x10ae0e150
2016-02-02 18:06:11.444 TimerDemo[1718:248402] root meta        :0x10b66a198
2016-02-02 18:06:11.444 TimerDemo[1718:248402] root meta's meta :0x10b66a198
2016-02-02 18:06:11.444 TimerDemo[1718:248402] ---------------------------------------------
2016-02-02 18:06:11.444 TimerDemo[1718:248402] class            :0x10ae0e178
2016-02-02 18:06:11.444 TimerDemo[1718:248402] meta class       :0x10ae0e178
2016-02-02 18:06:11.444 TimerDemo[1718:248402] root meta        :0x10ae0e178
2016-02-02 18:06:11.444 TimerDemo[1718:248402] root meta's meta :0x10ae0e178
```

分析：
注：Person是一个继承自NSObject的普通类，里面有个name属性。

1. 我们发现调用class方法的方式不能得到isa的指向链，但是第一次调用是正确的（class的输出都是0x10ae0e178），为什么？原因就是上面贴出来的class源码中，我们第一次调用的class是实例方法，会返回isa的类，但是第二次开始调用的就是类方法，返回的是本身，所以还是0x10ae0e178，以后无论怎么调用都是执行的类方法，返回的都是本身，所以，用class方法是得不到isa指向链的。

2. 用object_getClass()验证了我们Class、Object结构模型理论是对的，我们这里特意的打印了root meta class 的isa，发现果然指向是自己（0x10b66a198）。

3. 从打印结果我们能看到，类也是对象，meta类也是对象，都占有一块内存，而且我们会发现类对象、meta类对象、root meta类对象的指针都是用9位16进制数表示，而实例对象是用12位16进制数表示（这里用的是64位模拟器），为什么这些类对象的指针位数少？因为它们存在于段上，并不在栈或者堆上，黑魔法那篇文章说过段內存的事情。也就是说可以把这些类对象理解成单利，这是很重要的一点，希望大家理解，这一点可以让我们天马行空的想很多，比如可不可以把网络请求写在类对象里，嫩不能用类对象去解决自释放的问题，等等...这会是很有意思的思考。

我们理解了这个结构模型之后，看看我遇到的问题吧。

###问题来了

看代码：

```
#import <objc/runtime.h>

NSTimer *timer1 = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(test) userInfo:nil repeats:YES];
NSLog(@"instance       :%p", timer1);
NSLog(@"class          :%p", object_getClass(timer1));
NSLog(@"meta class     :%p", object_getClass(object_getClass(timer1)));
NSLog(@"root meta class:%p", object_getClass(object_getClass(object_getClass(timer1))));
NSLog(@"------------------------------");
NSLog(@"[NSTimer class]:%p", [NSTimer class]);
Log输出：

2016-02-02 18:19:11.643 TimerDemo[1745:255746] instance       :0x7fee8bc7a810
2016-02-02 18:19:11.644 TimerDemo[1745:255746] class          :0x10ece02c0
2016-02-02 18:19:11.644 TimerDemo[1745:255746] meta class     :0x10ece02e8
2016-02-02 18:19:11.644 TimerDemo[1745:255746] root meta class:0x10e895198
2016-02-02 18:19:11.644 TimerDemo[1745:255746] ------------------------------
2016-02-02 18:19:11.644 TimerDemo[1745:255746] [NSTimer class]:0x10ecdfe38
```

###问题来了

为什么[NSTimer class]:0x10ecdfe38与class:0x10ece02c0得到的指针不一样？

就是说为什么object_getClass(obj)与[OBJ class]返回的指针不同？

[NSTimer class]返回应该是类对象，object_getClass(timer1)返回的也应该是类对象，上面也说过，可以把类对象理解成单利，为什么指针不同？

如果用Person类做实验两者返回就是相同的，如果用系统其它类做实验两者返回还是不同的，它们本身之间就有矛盾，更重要的是，与我们刚刚理解的结构模型也是矛盾的，如何用这个模型理论去解释[NSTimer class]返回的这个指针？

感兴趣的朋友可以不往下看，自己想想为什么，其实很简单，但是没想到会是这样的，我当时就是这个感受。

###答案来了

答案非常简单，两个字：**类簇**

####看代码

```
NSTimer *timer1 = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(test) userInfo:nil repeats:YES];
NSLog(@"instance       :%@", timer1);
NSLog(@"class          :%@", object_getClass(timer1));
NSLog(@"meta class     :%@", object_getClass(object_getClass(timer1)));
NSLog(@"root meta class:%@", object_getClass(object_getClass(object_getClass(timer1))));
NSLog(@"------------------------------");
NSLog(@"[NSTimer class]:%@", [NSTimer class]);
```

代码没有变，只是我们这次不打印指针，打印对象的描述：

####Log输出：

```
2016-02-02 18:31:54.405 TimerDemo[1772:263501] instance       :<__NSCFTimer: 0x7ff83841e530>
2016-02-02 18:31:54.405 TimerDemo[1772:263501] class          :__NSCFTimer
2016-02-02 18:31:54.405 TimerDemo[1772:263501] meta class     :__NSCFTimer
2016-02-02 18:31:54.406 TimerDemo[1772:263501] root meta class:NSObject
2016-02-02 18:31:54.406 TimerDemo[1772:263501] ------------------------------
2016-02-02 18:31:54.406 TimerDemo[1772:263501] [NSTimer class]:NSTimer
```

我们发现我们之前结构模型的认识没有错，之所以矛盾是因为NSTimer是个类簇，它返回的并不是NSTimer对象，而是__NSCFTimer对象！我没有想到NSTimer也是类簇，我们熟悉的类簇是NSNumber，NSArray，NSDictionary、NSString...这说明大多数的OC类都是类簇实现的（连NSTimer也不放过），也说明为什么我们Person类是正常的，因为它不是类簇实现的。

这里还是简单的说说类簇的概念吧：一个父类有好多子类，父类在返回自身对象的时候，向外界隐藏各种细节，根据不同的需要返回的其实是不同的子类对象，这其实就是抽象类工厂的实现思路，iOS最典型的就是NSNumber。

```
NSNumber *intNum = [NSNumber numberWithInt:1];
NSNumber *boolNum = [NSNumber numberWithBool:YES];
NSLog(@"intNum :%@", [intNum class]);
NSLog(@"boolNum:%@", [boolNum class]);
2016-02-02 23:15:23.868 TimerDemo[1018:35735] intNum :__NSCFNumber
2016-02-02 23:15:25.027 TimerDemo[1018:35735] boolNum:__NSCFBoolean
```

这里的numberWithXXXX方法是类工厂返回的其实并不是NSNumber类，而是各个子类，NSCFNumber、NSCFBoolean。

这和我们上面问题中的NSTimer很像，类方法
scheduledTimerWithTimeInterval: target: selector: userInfo: repeats:并没有返回NSTimer对象，而是返回了它的子类__NSCFTimer对象。

有人可能要问，如何证明__NSCFTimer就是NSTimer的子类，如何证明类簇是真的？其实很简单：

```
NSLog(@"[NSTimer class]    :%p", [NSTimer class]);
NSLog(@"class_getSuperClass:%p", class_getSuperclass([timer1 class]));
2016-02-02 23:22:54.367 TimerDemo[1038:39690] [NSTimer class]    :0x109ee4e38
2016-02-02 23:22:54.367 TimerDemo[1038:39690] class_getSuperClass:0x109ee4e38
```

#总结
我把这个问题记录下来就是希望这篇文章对恰好遇到这个问题的朋友、有相同困惑的朋友一些帮助，如果是NSArray遇到了相同的问题我可能立马想到类簇，因为它还有可变的对象，但是NSTimer...

我猜中了开头，可我猜不着这结局。——紫霞仙子《大话西游》
我猜到了结局，但猜不到原因，谁让iOS是封闭的系统呢？

**欢迎大家和我交流沟通，文章中有任何错误和漏洞，恳请指正，谢谢。**

#关注我


**Swift/ObjC技术群一：[324400294(已满)]()**

**Swift/ObjC技术群二：[494669518]()**

**ObjC/Swift高级群：[461252383（注明年限，新手勿扰）]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)


#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)

