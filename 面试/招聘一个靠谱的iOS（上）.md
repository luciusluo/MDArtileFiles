#前言

以下题目来自[招聘一个靠谱的iOS](https://github.com/ChenYilong/iOSInterviewQuestions/blob/master/01%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88/%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88%EF%BC%88%E4%B8%8A%EF%BC%89.md#2-%E4%BB%80%E4%B9%88%E6%83%85%E5%86%B5%E4%BD%BF%E7%94%A8-weak-%E5%85%B3%E9%94%AE%E5%AD%97%E7%9B%B8%E6%AF%94-assign-%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C)，笔者整理了大部分，并将个人的理解和想法写出来，可能存在于原作者的参考答案不同的地方，还请大家各抒己见！

**所有参考答案不代表一定正确，但是都会有明确的来源说明及参考答案。若与您的想法存在差异，请在评论中提出，或者加群交流，得到统一意见后，笔者会更新文章内容！**

#1、什么情况使用weak关键字，相比assign有什么不同？

**使用weak关键字的主要场景：**

* 在ARC下,在有可能出现循环引用的时候往往要通过让其中一端使用weak来解决，比如: delegate代理属性，通常就会声明为weak。
* 自身已经对它进行一次强引用，没有必要再强引用一次时也会使用weak。比如：自定义 IBOutlet控件属性一般也使用weak，当然也可以使用strong。

**相比assign不同之处：**

* weak关键字只能用于对象，对于基本类型不能使用
* assign既可以用于对象，也可以用于基本类型，但是只是简单地进行赋值操作而已

#2、怎么用copy关键字？

**分析：**

copy关键字只能应用于对象，不能用于基本类型。copy属性会复制一份，并且强引用之，但是对于集合类型，通常并不能达到深拷贝的目的。NSString、NSArray、NSDictionary等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，当然很多时候都使用了strong来声明。block也使用copy关键字来声明。

**参考答案：**

* copy关键字只能应用于对象，不能用于基本类型
* 对于字符串，理应始终使用copy，虽然使用strong一般情况下也没有关系
* 对于不可变集合类型，有可变和不可变类型，若要防止外部的修改影响所传过来的值，应该使用copy来声明，虽然大多情况下使用strong一定问题都没有。不过，实际开发中，我见到的几乎都是使用strong来声明的，包括笔者在内。
* 对于可变集合类型，都应该使用strong来声明，不能使用copy，因为copy会生成一个不可变的类型，而不是可变的。
* 对于block，都应该使用copy来声明，原因是block来捕获上下文的信息。具体请参考：[【官方文档】Objects Use Properties to Keep Track of Blocks](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8-SW12)

#3、这个写法会出什么问题：@property (copy) NSMutableArray *array;

**参考答案：**

* 没有指明为nonatomic，因此就是atomic原子操作，会影响性能。该属性使用了同步锁，会在创建时生成一些额外的代码用于帮助编写多线程程序，这会带来性能问题，通过声明nonatomic可以节省这些虽然很小但是不必要额外开销。在我们的应用程序中，几乎都是使用nonatomic来声明的，因为使用atomic并不能保证绝对的线程安全，对于要绝对保证线程安全的操作，还需要使用更高级的方式来处理，比如NSSpinLock、@syncronized等
* 因为使用的是copy，所得到的实际是NSArray类型，它是不可变的，若在使用中使用了增、删、改操作，则会crash

例如：

```
// 使用copy声明，生成的实际上是NSArray类型
@property (copy) NSMutableArray *mutableArray;

NSMutableArray *array = [NSMutableArray arrayWithObjects:@1,@2,nil];
self.mutableArray = array;

// 调用删除操作，会崩溃，因为所生成的对象类型实际上是NSArray类型，是不可变的
[self.mutableArray removeObjectAtIndex:0];

// Crash信息
-[__NSArrayI removeObjectAtIndex:]: unrecognized selector sent to instance 0x7fcd1bc30460
```

#4、@property的本质是什么？ivar、getter、setter是如何生成并添加到这个类中的？

**@property的本质：**

@property = ivar（实例变量） + getter（取方法） + setter（存方法）

“属性” (property)有两大概念：ivar（实例变量）、存取方法（access method ＝ getter + setter）

**ivar、getter、setter如何生成并添加到类中：**

这是编译器自动合成的，通过@synthesize关键字指定，若不指定，默认为@synthesize propertyName = _propertyName;若手动实现了getter/setter方法，则不会自动合成。

现在编译器已经默认为我们添加@synthesize propertyName = _propertyName;因此不再需要手动添加了，除非你真的要改成员变量名。

生成getter方法时，会判断当前属性名是否有\_，比如声明属性为@property (nonatomic, copy) NSString *\_name;那么所生成的成员变量名就会变成\__name，如果我们要手动生成getter方法，就要判断是否以\_开头了。

不过，命名都要有规范，是不允许声明属性是使用\_开头的，不规范的命名，在使用runtime时，会带来很多的不方便的。

如果想了解更多关于runtime方面的知识，请阅读[runtime专题](http://www.henishuo.com/category/runtime/)

#5、@protocol和category中如何使用 @property


**参考答案：**

* 在protocol中使用@property只会生成setter和getter方法声明，我们使用属性的目的是希望遵守我协议的对象能实现该属性
* category使用@property也是只会生成setter和getter方法的声明，如果我们真的需要给category增加属性的实现，需要借助于运行时的两个函数：

```
objc_setAssociatedObject
objc_getAssociatedObject
```

如果想了解更多关于runtime方面的知识，请阅读[runtime关联属性](http://www.henishuo.com/runtime-association/)

#6、runtime如何实现weak属性？

**参考答案：**

* 通过关联属性来实现：

```
// 声明一个weak属性，这里假设delegate，其实weak关键字可以不使用，
// 因为我们重写了getter/setter方法
@property (nonatomic, weak) id delegate;

- (id)delegate {
  return objc_getAssociatedObject(self, @"__delegate__key");
}

// 指定使用OBJC_ASSOCIATION_ASSIGN，官方注释是：
// Specifies a weak reference to the associated object.
// 也就是说对于对象类型，就是weak了
- (void)setDelegate:(id)delegate {
  objc_setAssociatedObject(self, @"__delegate__key", delegate, OBJC_ASSOCIATION_ASSIGN);
}
```

* 通过objc_storeWeak函数来实现，不过这种方式几乎没有遇到有人这么使用过，因为这里不细说了。

#7、@property中有哪些属性关键字，后面可以有哪些修饰符？

**属性可以拥有的特质分为四类:**

* 原子性：nonatomic声明为非原子操作，atomic声明为原子操作。在默认情况下，由编译器合成的方法会通过锁定机制确保其原子性(atomicity)。如果属性具备 nonatomic 特质，则不使用同步锁。请注意，尽管没有名为“atomic”的特质(如果某属性不具备 nonatomic 特质，那它就是“原子的” ( atomic) )，但是仍然可以在属性特质中写明这一点，编译器不会报错。若是自己定义存取方法，那么就应该遵从与属性特质相符的原子性。
* 读/写权限：readwrite(读写)、readonly (只读)
* 内存管理相关：assign、strong、 weak、unsafe_unretained、copy
* 方法名：getter=<name> 、setter=set<Name>。getter=<name>的样式：

```
@property (nonatomic, getter=isOn) BOOL on;
```

* 不常用的：nonnull、null_resettable、nullable

#8、weak属性需要在dealloc中置nil么？

**参考答案：**

对于weak声明的属性，都不需要在dealloc中指定为nil，在ARC下，编译器会自动帮助我们处理。即使编译器不帮助我们处理，我们也不需要手动在dealloc中设置为nil。

#9、@synthesize和@dynamic分别有什么作用？

**分析：**

@property有两个对应的词，一个是@synthesize，，另一个是@dynamic。如果 @synthesize和@dynamic都没写，那么默认的就是@syntheszie var = _var;这两个关键字都是为@property关键字工作的。

**参考答案：**

* @synthesize的语义是如果你没有手动实现setter方法和getter方法，那么编译器会自动为你加上这两个方法。
* @dynamic告诉编译器：属性的setter与getter方法由用户自己实现，不自动生成。假如一个属性被声明为@dynamic var，然后你没有提供@setter方法和@getter 方法，编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺setter方法会导致程序崩溃；或者当运行到someVar = var 时，由于缺getter方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

#10、ARC下不显式指定任何属性关键字时，默认的关键字都有哪些？

**参考答案：**


* 对于基本数据类型默认关键字是：atomic,readwrite,assign
* 对于普通的Objective-C对象：atomic,readwrite,strong

参考链接：

* [Objective-C ARC: strong vs retain and weak vs assign](http://stackoverflow.com/a/15541801/3395008)
* [Variable property attributes or Modifiers in iOS](http://rdcworld-iphone.blogspot.in/2012/12/variable-property-attributes-or.html)

#11、objc中向一个nil对象发送消息将会发生什么？

**参考答案：**

在Objective-C中向nil发送消息是完全有效的，只是在运行时不会有任何作用，因为在运行时调用时，objc_msgSend函数传过去的receiver是nil，而内部会判断receiver是否为nil，若为nil则什么也不干。同样，若cmd也就是selector为nil，也是什么也不干。

#12、objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？

**参考答案：**

实际上，编译器在编译时会转换成objc_msgSend，大概会像这样：

```
((void (*)(id, SEL))(void)objc_msgSend)((id)obj, sel_registerName("foo"));
```

也就是说，[obj foo];在objc动态编译时，会被转换为：objc_msgSend(obj, @selector(foo));这样的形式，但是需要根据具体的参数类型及返回值类型进行相应的类型转换。

#13、什么时候会报unrecognized selector的异常？

**参考答案：**

下面只讲述对象方法的解析过程：

* 第一步：+ (BOOL)resolveInstanceMethod:(SEL)sel实现方法，指定是否动态添加方法。若返回NO，则进入下一步，若返回YES，则通过class_addMethod函数动态地添加方法，消息得到处理，此流程完毕。
* 第二步：在第一步返回的是NO时，就会进入- (id)forwardingTargetForSelector:(SEL)aSelector方法，这是运行时给我们的第二次机会，用于指定哪个对象响应这个selector。不能指定为self。若返回nil，表示没有响应者，则会进入第三步。若返回某个对象，则会调用该对象的方法。
* 第三步：若第二步返回的是nil，则我们首先要通过- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector指定方法签名，若返回nil，则表示不处理。若返回方法签名，则会进入下一步。
* 第四步：当第三步返回方法方法签名后，就会调用- (void)forwardInvocation:(NSInvocation *)anInvocation方法，我们可以通过anInvocation对象做很多处理，比如修改实现方法，修改响应对象等
* 第五步：若没有实现- (void)forwardInvocation:(NSInvocation *)anInvocation方法，那么会进入- (void)doesNotRecognizeSelector:(SEL)aSelector方法。若我们没有实现这个方法，那么就会crash，然后提示打不到响应的方法。到此，动态解析的流程就结束了。

更新详细地，请阅读[runtime message forwarding](http://www.henishuo.com/runtime-message-forwarding/)

#14、一个objc对象的isa的指针指向什么？有什么作用？

**参考答案：**

先阅读下图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/inherit.png)

从图中可以清晰地看出，一个对象的isa指针指向的是他所属的类(class)，用于查找对象上的方法。而类对象的isa指针又指向它的元类(meta class)，无类的isa也指向根类的元类，而根类的元类的isa又指向根类本身。

#15、下面的代码输出什么？

```
@implementation Son : Father

- (id)init {
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}

@end

// 输出
NSStringFromClass([self class]) = Son
NSStringFromClass([super class]) = Son
```

这个题目主要是考察关于Objective-C中对self和super的理解。我们都知道：self是类的隐藏参数，指向当前调用方法的这个类的实例。那super呢？

很多人会想当然的认为“super和self类似，应该是指向父类的指针吧！”。这是很普遍的一个误区。其实 super是一个 Magic Keyword，它本质是一个编译器标示符，和self 是指向的同一个消息接受者！他们两个的不同点在于：super会告诉编译器，调用class 这个方法时，要去父类的方法，而不是本类里的。

上面的例子不管调用[self class]还是[super class]，接受消息的对象都是当前 Son ＊xxx 这个对象。

当使用self调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；而当使用super时，则从父类的方法列表中开始找。然后调用父类的这个方法。

通过self来调用方法时，会转换成：

```
id objc_msgSend(id self, SEL op, ...)
```

而通过super调用方法时，会转换成：

```
id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
```

而第一个参数是 objc_super 这样一个结构体，其定义如下:

```
struct objc_super {
       __unsafe_unretained id receiver;
       __unsafe_unretained Class super_class;
};
```

#16、runtime如何通过selector找到对应的IMP地址？

如下图所示，每个selector都与对应的IMP是一一对应的关系，通过selector就可以直接找到对应的IMP：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/153752_G4aM_1463495.jpg)

#17、objc中的类方法和实例方法有什么本质区别和联系？

**类方法：**

* 类方法是属于类对象的（所谓的类对象，不是class instance）
* 类方法只能通过类对象调用
* 类方法中的self是类对象
* 类方法可以调用其他的类方法
* 类方法中不能访问成员变量
* 类方法中不定直接调用对象方法

**实例方法：**

* 实例方法是属于实例对象的
* 实例方法只能通过实例对象调用
* 实例方法中的self是实例对象
* 实例方法中可以访问成员变量
* 实例方法中直接调用实例方法
* 实例方法中也可以调用类方法(通过类名)

#18、\_objc\_msgForward函数是做什么的，直接调用它将会发生什么？

**参考答案：**

\_objc\_msgForward是IMP类型，用于消息转发的：当向一个对象发送一条消息，但它并没有实现的时候，\_objc\_msgForward会尝试做消息转发。

```
IMP msgForward =  _objc_msgForward;
```

如果手动调用_objc_msgForward，将跳过查找IMP的过程，而是直接触发“消息转发”，进入如下流程：

* 第一步：+ (BOOL)resolveInstanceMethod:(SEL)sel实现方法，指定是否动态添加方法。若返回NO，则进入下一步，若返回YES，则通过class_addMethod函数动态地添加方法，消息得到处理，此流程完毕。
* 第二步：在第一步返回的是NO时，就会进入- (id)forwardingTargetForSelector:(SEL)aSelector方法，这是运行时给我们的第二次机会，用于指定哪个对象响应这个selector。不能指定为self。若返回nil，表示没有响应者，则会进入第三步。若返回某个对象，则会调用该对象的方法。
* 第三步：若第二步返回的是nil，则我们首先要通过- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector指定方法签名，若返回nil，则表示不处理。若返回方法签名，则会进入下一步。
* 第四步：当第三步返回方法方法签名后，就会调用- (void)forwardInvocation:(NSInvocation *)anInvocation方法，我们可以通过anInvocation对象做很多处理，比如修改实现方法，修改响应对象等
* 第五步：若没有实现- (void)forwardInvocation:(NSInvocation *)anInvocation方法，那么会进入- (void)doesNotRecognizeSelector:(SEL)aSelector方法。若我们没有实现这个方法，那么就会crash，然后提示打不到响应的方法。到此，动态解析的流程就结束了。

更新详细地，请阅读[runtime message forwarding](http://www.henishuo.com/runtime-message-forwarding/)

#19、runtime如何实现weak变量的自动置nil？

**参考答案：**

runtime对注册的类会进行布局，对于weak对象会放入一个hash表中。 用weak指向的对象内存地址作为key，当此对象的引用计数为0的时候会dealloc。假如weak指向的对象内存地址是a，那么就会以a为键，在这个 weak 表中搜索，找到所有以a为键的weak对象，从而设置为nil。

weak修饰的指针默认值是nil（在Objective-C中向nil发送消息是安全的）

#20、能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

**不能向编译后得到的类中增加实例变量**

因为编译后的类已经注册在runtime中，类结构体中的objc\_ivar\_list实例变量的链表 和instance\_size实例变量的内存大小已经确定，同时runtime会调用class\_setIvarLayout或 class\_setWeakIvarLayout来处理strong和weak引用。所以不能向存在的类中添加实例变量；

**能向运行时创建的类中添加实例变量**

运行时创建的类是可以添加实例变量，调用class_addIvar函数。但是得在调用 objc\_allocateClassPair之后，objc\_registerClassPair之前，原因同上。

#关注我


关注                | 账号              | 备注
-------------      | -------------     | ----------------
Swift/ObjC技术群一  | 324400294         |  群一若已满，请申请群二
Swift/ObjC技术群二  | 494669518         | 群二若已满，请申请群三
Swift/ObjC技术群三  | 461252383         | 群三若已满，会有提示信息
关注微信公众号       | iOSDevShares      | 关注微信公众号，会定期地推送好文章
关注新浪微博账号      |  [标哥Jacky](http://weibo.com/u/5384637337) | 关注微博，每次发布文章都会分享到新浪微博，即可时时阅读文章
关注标哥的GitHub     | [CoderJackyHuang](https://github.com/CoderJackyHuang) | 这里有很多的Demo和开源组件，大家可以关注哦！
关于我               | [进一步了解标哥](http://www.henishuo.com/about-biaoge/) | 大家若对笔者感兴趣，可以关注我哦！如果觉得文章对您很有帮助，可捐助我！