>#[原文出自：标哥的技术博客](http://www.henishuo.com/runtime-message-forwarding/)

#前言

本篇文章是研究消息转发的机制，苹果的消息转发机制就像一条链，消息传送链越长则消耗也越大，最好是在第一级就可以直接发送消息。

我们必须要先了解`objc_msgSend`函数调用的检测过程：

1. 第一步：检测这个`selector`是不是要忽略的。
2. 第二步：检测这个`target`是不是`nil`对象。`nil`对象执行任何一个方法不会`Crash`是因为会被忽略掉。
3. 第三步：查找这个类的`IMP`，也就是方法实现。先从方法缓存列表`cache`中查找，若找到则跳到对应的函数去执行；若找不到，则查找方法分发表。如果分发表找不到就到父类的分发表去找，直到找到或者查找到`NSObject`根类为止。
4. 第四步：前三步都找不到，则开始进入动态方法解析了

#动态解析

我们用一张图讲解动态解析的流程（图片源自网络）：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/msgresolve.png)

其流程是这样的：

1. 第一步：`+ (BOOL)resolveInstanceMethod:(SEL)sel`实现方法，指定是否动态添加方法。若返回NO，则进入下一步，若返回YES，则通过`class_addMethod`函数动态地添加方法，消息得到处理，此流程完毕。
2. 第二步：在第一步返回的是NO时，就会进入`- (id)forwardingTargetForSelector:(SEL)aSelector`方法，这是运行时给我们的第二次机会，用于指定哪个对象响应这个`selector`。不能指定为`self`。若返回nil，表示没有响应者，则会进入第三步。若返回某个对象，则会调用该对象的方法。
3. 第三步：若第二步返回的是nil，则我们首先要通过`- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`指定方法签名，若返回nil，则表示不处理。若返回方法签名，则会进入下一步。
4. 第四步：当第三步返回方法方法签名后，就会调用`- (void)forwardInvocation:(NSInvocation *)anInvocation`方法，我们可以通过`anInvocation`对象做很多处理，比如修改实现方法，修改响应对象等
5. 第五步：若没有实现`- (void)forwardInvocation:(NSInvocation *)anInvocation`方法，那么会进入`- (void)doesNotRecognizeSelector:(SEL)aSelector`方法。若我们没有实现这个方法，那么就会`crash`，然后提示打不到响应的方法。到此，动态解析的流程就结束了。

#验证动态解析

这里提供了三个小例子，验证解析流程。

1. 第一个例子：提供声明，但是不提供方法实现。验证当找不到方法的实现时，动态添加方法。
2. 第二个例子：不提供声明，将调用对象修改成其它类实例。验证修改处理消息的对象。
3. 第三个例子：不提供声明，不修改调用对象，但是修改调用的方法

##例子一

我们先声明一个`HYBDog`类，提供一个`eat`方法，但是我们只提供声明，却不实现这个方法。

###声明HYBDog类

```
@interface HYBDog : NSObject

// 我们只声明，而不实现
- (void)eat;

@end
```

###实现HYBDog类

我们不实现实例方法`-eat`，而是添加了一个`C`语言的`eat`方法，注意这个`eat`方法不是`HYBDog`的实例方法。然后我们看一下实现如下：

```
#import "HYBDog.h"
#import <objc/runtime.h>


@implementation HYBDog

// 第一步：实现此方法，在调用对象的某方法找不到时，会先调用此方法，允许
// 我们动态添加方法实现
+ (BOOL)resolveInstanceMethod:(SEL)sel {
  // 我们这里没有给dog声明有eat方法，因此，我们可以动态添加eat方法
  if ([NSStringFromSelector(sel) isEqualToString:@"eat"]) {
    class_addMethod(self, sel, (IMP)eat, "v@:");
    return YES;
  }
  
  return [super resolveInstanceMethod:sel];
}

// 这个方法是我们动态添加的哦
//
void eat(id self, SEL cmd) {
  NSLog(@"%@ is eating", self);
}

@end
```

由于我们没有提供`HYBDog`实例方法`-eat`，因此在调用此方法时，runtime会调用`+ (BOOL)resolveInstanceMethod:(SEL)sel`方法，允许我们动态添加方法。当然我们也可以返回NO。若返回NO，就继续往下传递。

我们这里通过`class_addMethod`动态地添加方法`eat`，这个`eat`方法是`C`函数。为什么这里的`void eat(id self, SEL cmd)`有两个参数呢？我们什么时候传有参数？这是不是很奇怪呢？其实是这样的，编译器在将函数转换成`objc_msgSend`函数调用时，都会自动添加上`id self, SEL cmd`这两个参数，因此我们就可以拿得到。

###测试例一

我们如此调用，由于我们声明了`-eat`方法，因此是可以调用的。但是我们却没有实现它，编译时是没有问题的，因为`objective-c`在编译时，只是转换成`objc_msgSend`函数，而对于实现是在链接时才会去调用。

```
HYBDog *dog = [[HYBDog alloc] init];
[dog eat];
```

其打印结果如下：

```
<HYBDog: 0x7fdf53f08750> is eating
```

说明它成功的添加了我们的C语言方法的实现作为`-eat`方法的实现。到此，例一就圆满验证通过了！

##例子二

我们声明一个`HYBPig`类，但是这个类不声明`-eat`方法。

###声明HYBPig类

```
@interface HYBPig : NSObject

@end
```

###实现HYBPig类

本例子要验证消息处理查找的流程，修改调用`-eat`方法的对象：

```
#import "HYBPig.h"
#import "HYBDog.h"

@implementation HYBPig

// 第一步，我们不动态添加方法，返回NO
+ (BOOL)resolveInstanceMethod:(SEL)sel {
  return NO;
}

// 第二步，备选提供响应aSelector的对象，我们不备选，因此设置为nil，就会进入第三步
- (id)forwardingTargetForSelector:(SEL)aSelector {
  return nil;
}

// 第三步，先返回方法选择器。如果返回nil，则表示无法处理消息
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
  if ([NSStringFromSelector(aSelector) isEqualToString:@"eat"]) {
    return [NSMethodSignature signatureWithObjCTypes:"v@:"];
  }
  
  return [super methodSignatureForSelector:aSelector];
}

// 第三步，只有返回了方法签名，都会进入这一步，这一步用户调用方法
// 改变调用对象等
- (void)forwardInvocation:(NSInvocation *)anInvocation {
  // 我们改变调用对象为dog
  [anInvocation invokeWithTarget:[[HYBDog alloc] init]];
}

@end
```

其实，如果我们调用`+ (BOOL)resolveInstanceMethod:(SEL)sel`返回NO，那么我们就没有必要实现它，因为默认就是返回NO。

如果调用`- (id)forwardingTargetForSelector:(SEL)aSelector`返回nil，那么我们也没有必要实现它，因为默认就是返回nil。

但是，对于`-methodSignatureForSelector:`方法，默认也是返回nil，若不返回某方法签名，那么`-forwardInvocation:`方法就不会被调用，此时也就崩溃了。

对于`-forwardInvocation:`方法中，我们修改了调用`-eat`方法的是实例为`HYBDog`实例。

###测试例二

由于我们没有声明`-eat`方法，因此不能通过直接调用。但是，我们可以通过`performSelector`来实现，当然也可以通过`objc_msgSend`函数实现：

```
HYBPig *pig = [[HYBPig alloc] init];
[pig performSelector:@selector(eat) withObject:nil afterDelay:0];
```

通过`objc_msgSend`也可以实现：

```
((void (*)(id, SEL))objc_msgSend)((id)pig, @selector(eat));
```

打印结果却是`HYBDog`，说明我们成功地修改调用对象了：

```
<HYBDog: 0x7f8d8bf7c510> is eating
```

##例子三

我们这里创建一个`HYBCat`类来测试，修改调用方法为其它方法。

###声明HYBCat类

```
@interface HYBCat : NSObject

@end
```

###实现HYBCat类

```
@implementation HYBCat

// 第一步：在没有找到方法时，会先调用此方法，可用于动态添加方法
// 我们不动态添加
+ (BOOL)resolveInstanceMethod:(SEL)sel {
  return NO;
}

// 第二步：上一步返回NO，就会进入这一步，用于指定备选响应此SEL的对象
// 千万不能返回self，否则就会死循环
// 自己没有实现这个方法才会进入这一流程，因此成为死循环
- (id)forwardingTargetForSelector:(SEL)aSelector {
  return nil;
}

// 第三步：指定方法签名，若返回nil，则不会进入下一步，而是无法处理消息
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
  if ([NSStringFromSelector(aSelector) isEqualToString:@"eat"]) {
    return [NSMethodSignature signatureWithObjCTypes:"v@:"];
  }
  
  return [super methodSignatureForSelector:aSelector];
}

// 当我们实现了此方法后，-doesNotRecognizeSelector:不会再被调用
// 如果要测试找不到方法，可以注释掉这一个方法
- (void)forwardInvocation:(NSInvocation *)anInvocation {
  
  // 我们还可以改变方法选择器
  [anInvocation setSelector:@selector(jump)];
  // 改变方法选择器后，还需要指定是哪个对象的方法
  [anInvocation invokeWithTarget:self];
}

- (void)doesNotRecognizeSelector:(SEL)aSelector {
  NSLog(@"无法处理消息：%@", NSStringFromSelector(aSelector));
}

- (void)jump {
  NSLog(@"由eat方法改成jump方法");
}

@end
```

当我们实现了`-doesNotRecognizeSelector:`就方法时，就不会因为找不到方法而崩溃了。我们这里将动态地将调用`-eat`方法修改为`-jump`方法，同时也要设置这个`-jump`是哪个对象的。

这里的注释已经说明得很清楚了，就不再细说了！

###测试例三

```
HYBCat *cat = [[HYBCat alloc] init];
[cat performSelector:@selector(eat) withObject:nil afterDelay:0];
```

打印结果为：

```
由eat方法改成jump方法
```

说明我们已经成功地动态地修改方法了。

#源代码

大家可以到GITHUB下载源代码：[https://github.com/CoderJackyHuang/RuntimeDemo](https://github.com/CoderJackyHuang/RuntimeDemo)

对应于`ForwardMessage`分组。**请随手给个star支持一下吧！**

#写在最后

文章中难免有说得不合理的地方，如果您认为说法不正确或者哪里有错误的地方，请在评论中留言，笔者会在第一时间修正！！！

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

#支持并捐助

如果您觉得文章对您很有帮助，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
