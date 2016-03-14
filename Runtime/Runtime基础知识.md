#前言


学习`Objective-C`的运行时`Runtime`系统是很有必要的。个人觉得，得之可得天下，失之则失天下。

`Objective-C`提供了编译运行时，只要有可能，它都可以动态地运作。这意味着不仅需要编译器，还需要运行时系统执行编译的代码。运行时系统充当`Objective-C`语言的操作系统，有了它才能运作。

运行时系统所提供功能是非常强大的，在实际开发中是经常使用到的。比如，苹果不允许我们给`Category`追加扩展属性，是因为它不会自动生成成员变量，那么我们通过运行时就可以很好的解决这个问题。另外，常见的模型转字典或者字典转模型，对象归档等。后续我们再来学习如何应用，本节只是讲讲理论。

#与Runtime交互


`Objective-C`程序有有三种与`runtime`系统交互的级别：

1. 通过`Objective-C`源代码
2. 通过`Foundation`库中定义的`NSObject`提供的方法
3. 通过直接调用`runtime`方法

###通过Objective-C源代码

在大多数的部分，运行时系统会自动运行并在后台运行。我们使用它只是写源代码并编译源代码。当编译包含`Objective-C`类和方法的代码时，编译器会创建实现了语言动态特性的数据结构和函数调用。该数据结构捕获在类、扩展和协议中所定义的信息。

最重要的`runtime`函数是发消息函数，在编译时，编译器会转换成类似`objc_msgSend`这样的发送消息的函数。因此，我们通过写好源代码，编译器会自动帮助我们编译成`runtime`代码。

###通过NSObject提供的方法

在`Cocoa`编程中，大部分的类都继承于`NSObject`，也就是说`NSObject`通常是根类，大部分的类都继承于`NSObject`。有些特殊的情况下，`NSObject`只是提供了它应该要做什么的模板，却没有提供所有必须的代码。

有些`NSObject`提供的方法仅仅是为了查询运动时系统的相关信息，这此方法都可以反查自己。比如`-isKindOfClass:`和`-isMemberOfClass:`都是用于查询在继承体系中的位置。`-respondsToSelector:`指明是否接受特定的消息。`+conformsToProtocol:`指明是否要求实现在指定的协议中声明的方法。`-methodForSelector:`提供方法实现的地址。

###通过直接调用runtime函数

`runtime`库函数在`usr/include/objc`目录下，我们主要关注是这两个头文件：

```
#import <objc/runtime.h>
#import <objc/objc.h>
```

关于如何使用，后续的文章再细细讲解。

#消息（Message）


为什么叫消息呢？因为面向对象编程中，对象调用方法叫做发送消息。在编译时，应用的源代码就会被编将对象发送消息转换成`runtime`的`objc_msgSend`函数调用。

在`Objective-C`，消息在运行时并不要求实现。编译器会转换消息表达式：

```
[receiver message];
```

在编译时会转换成类似这样的函数调用：

```
objc_msgSend(receiver, selector);
```

具体会转换成哪个，我们来看看官方的原话：

```
When it encounters a method call, the compiler generates a call to one of the
 *  functions \c objc_msgSend, \c objc_msgSend_stret, \c objc_msgSendSuper, or \c objc_msgSendSuper_stret.
 *  Messages sent to an object’s superclass (using the \c super keyword) are sent using \c objc_msgSendSuper; 
 *  other messages are sent using \c objc_msgSend. Methods that have data structures as return values
 *  are sent using \c objc_msgSendSuper_stret and \c objc_msgSend_stret.
```

也就是说，我们是通过编译器来自动转换成运行时代码时，它会根据类型自动转换成下面的其它一个函数：

* objc\_msgSend：其它普通的消息都会通过该函数来发送
* objc\_msgSend\_stret：消息中需要有数据结构作为返回值时，会通过该函数来发送消息并接收返回值
* objc\_msgSendSuper：与objc\_msgSend函数类似，只是它把消息发送给父类实例
* objc\_msgSendSuper\_stret：与objc\_msgSend\_stret函数类似，只是它把消息发送给父类实例并接收数组结构作为返回值

另外，如果函数返回值是浮点类型，官方说明如下：

```
 * arm:    objc_msgSend_fpret not used
 * i386:   objc_msgSend_fpret used for `float`, `double`, `long double`.
 * x86-64: objc_msgSend_fpret used for `long double`.
 *
 * arm:    objc_msgSend_fp2ret not used
 * i386:   objc_msgSend_fp2ret not used
 * x86-64: objc_msgSend_fp2ret used for `_Complex long double`.
```
其实这是一个条件编译，我们不用担心是哪种处理器上，我们只需要调用`objc_msgSend_fpret`函数即可。

**注意事项：**一定要调用所调用的API支持哪些平台，乱调在导致部分平台上不支持而崩溃的。

当消息被发送到实例对象时，它是如何处理的：

![image](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Art/messaging1.gif)

我们的根类是`NSObject`，它会一层一层的传递，直接找到要处理该消息的对象，若都没有找到，正常情况下会出现`Unreconized selector ...`这样的崩溃提示了。

#Message Forwarding


当发送消息给一个不处理该消息的对象是错误的。然后在宣布错误之前，运行时系统给了接收消息的对象处理消息的第二个机会。

当某对象不处理某消息时，可以通过重写`-forwardInvocation:`方法来提供一个默认的消息响应或者避免出错。当对象中找不到方法实现时，会按照类继承关系一层层往上找。我们看看类继承关系图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/inherit.png)

所有元类中的`isa`指针都指向根元类，而根元类的`isa`指针则指向自身。根元类是继承于根类的，与根类的结构体成员一致，都是objc\_class结构体，不同的是根元类的`isa`指针指向自身，而根类的`isa`指针为`nil`

我们再看看消息处理流程：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/msgresolve.png)

当对象查询不到相关的方法，消息得不到该对象处理，会启动“消息转发”机制。消息转发还分为几个阶段：先询问`receiver`或者说是它所属的类是否能动态添加方法，以处理当前这个消息，这叫做“动态方法解析”，runtime会通过`+resolveInstanceMethod:`判断能否处理。如果runtime完成动态添加方法的询问之后，`receiver`仍然无法正常响应则Runtime会继续向receiver询问是否有其它对象即其它receiver能处理这条消息，若返回能够处理的对象，Runtime会把消息转给返回的对象，消息转发流程也就结束。若无对象返回，Runtime会把消息有关的全部细节都封装到`NSInvocation`对象中，再给`receiver`最后一次机会，令其设法解决当前还未处理的这条消息。

**提示：**消息处理越往后，开销也就会越大，因此最好直接在第一步就可以得到消息处理。

我们看看类结构体：

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
/* Use `Class` instead of `struct objc_class *` */
```

我们可以看到每个类结构体都会有一个`isa`指针，它是指向元类的。它还有一个父类指针`super_class`，指针父类。包含了类的名称`name`、类的版本信息`version`、类的一些标识信息`info`、实例大小`instance_size`、成员变量地址列表`ivars`、方法地址列表`methodLists`、缓存最近使用的方法地址`cache`、协议列表protocols`。

我们在使用时，经常使用到`Class`，它就是：

```
typedef struct objc_class *Class;
```

当类为根类时，它的`super_class`就会是`nil`。普通的`Class`存储的是实例成员，如`-`号方法、属性、成员变量，而`isa`则指向元类，而元类存储的是静态成员，如`+`号方法、`static`成员。

#Type Encoding



编码值       |   含意
----------- | -------------
c           |  代表char类型
i           |  代表int类型
s           |  代表short类型
l           |  代表long类型，在64位处理器上也是按照32位处理
q           |  代表long long类型
C           |  代表unsigned char类型
I           |  代表unsigned int类型
S           |  代表unsigned short类型
L           |  代表unsigned long类型
Q           |  代表unsigned long long类型
f           |  代表float类型
d           |  代表double类型
B           |  代表C++中的bool或者C99中的_Bool
v           |  代表void类型
*           |  代表char *类型
@           |  代表对象类型
\#          |  代表类对象 (Class)
:           |  代表方法selector (SEL)
[array type]|  代表array
{name=type...}| 代表结构体
(name=type...)| 代表union
bnum          | A bit field of num bits
^type         | A pointer to type
?             | An unknown type (among other things, this code is used for function pointers)

我们想要通过运行时处理各种类型，那么我们必须要知道哪种字符代表什么类型。

#写在最后


理论知识就写这么多吧，这篇文章只是讲讲一些比较基础的知识点，为后面的学习奠定基础。如果文章中出现有疑问的地方，请在评论中评论，笔者会在第一时间回复您的！

