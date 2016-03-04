#前言

想要通过runtime发送消息，就必须要掌握runtime如何发送消息，是调用哪个函数？又是如何调用的？本篇文章只是记录笔者学习`objc_msgSend`函数的使用笔记，若有误解之处，还请指出。谢谢！

#objc_msgSend

我们先来看看官方函数`objc_msgSend`的声明：

```
/* Basic Messaging Primitives
 *
 * On some architectures, use objc_msgSend_stret for some struct return types.
 * On some architectures, use objc_msgSend_fpret for some float return types.
 * On some architectures, use objc_msgSend_fp2ret for some float return types.
 *
 * These functions must be cast to an appropriate function pointer type 
 * before being called. 
 */
void objc_msgSend(void /* id self, SEL op, ... */ )
```

从这个函数的注释可以看出来了，这是个最基本的用于发送消息的函数。另外，这个函数并不能发送所有类型的消息，只能发送基本的消息。比如，在一些处理器上，我们必须使用`objc_msgSend_stret`来发送返回值类型为结构体的消息，使用`objc_msgSend_fpret `来发送返回值类型为浮点类型的消息，而又在一些处理器上，还得使用`objc_msgSend_fp2ret`来发送返回值类型为浮点类型的消息。

最关键的一点：无论何时，要调用`objc_msgSend`函数，必须要将函数强制转换成合适的函数指针类型才能调用。

从`objc_msgSend`函数的声明来看，它应该是不带返回值的，但是我们在使用中却可以强制转换类型，以便接收返回值。另外，它的参数列表是可以任意多个的，前提也是要强制函数指针类型。

#学习使用

我们建立一个类，就专门学习如何运用`objc_msgSend`函数来发送消息。我们建立了一个`HYBMsgSend`类来学习。

##1.创建并初始化对象

我们一直以来都是使用类似这样的`[[HYBMsgSend alloc] init]`来创建并初始化对象吧，其实在编译时，这一行代码也会转换成类似如下的代码：

```
// 1.创建对象
HYBMsgSend *msg = ((HYBMsgSend * (*)(id, SEL))objc_msgSend)((id)[HYBMsgSend class], @selector(alloc));

// 2.初始化对象
msg = ((HYBMsgSend * (*)(id, SEL))objc_msgSend)((id)msg, @selector(init));
```

要发送消息给`msg`对象，并将创建的对象返回，那么我们需要强转函数指针类型。`(HYBMsgSend * (*)(id, SEL)`这是带一个对象指针返回值和两个参数的函数指针，这样就可以调用了。

##2.发送无参数无返回值消息

我们先定义一个方法：

```
- (void)noArgumentsAndNoReturnValue {
  NSLog(@"%s was called, and it has no arguments and return value", __FUNCTION__);
}
```

然后，我们发送消息，测试一下结果。

```
  // 2.调用无参数无返回值方法
((void (*)(id, SEL))objc_msgSend)((id)msg, @selector(noArgumentsAndNoReturnValue));
```

结果如下，说明成功地接收到消息并处理了：

```
-[HYBMsgSend noArgumentsAndNoReturnValue] was called, and it has no arguments and return value
```

##3.带参数不带返回值消息

我们定义一个方法只带参数，不带返回值：

```
- (void)hasArguments:(NSString *)arg {
  NSLog(@"%s was called, and argument is %@", __FUNCTION__, arg);
}
```

然后尝试发送消息试试：

```
// 3.调用带一个参数但无返回值的方法
((void (*)(id, SEL, NSString *))objc_msgSend)((id)msg, @selector(hasArguments:), @"带一个参数，但无返回值");
```

同样，我们也是需要强转函数指针类型，否则会报错的。其实，只有调用runtime函数来发送消息，几乎都需要强转函数指针类型为合适的类型。

其打印结果说明成功接收到消息并处理了：

```
-[HYBMsgSend hasArguments:] was called, and argument is 带一个参数，但无返回值
```

##4.带返回值不带参数消息

当消息带有返回值时，我们如何接收呢？我们先声明一个方法，让它带有返回值，但是不带参数：

```
- (NSString *)noArgumentsButReturnValue {
  NSLog(@"%s was called, and return value is %@", __FUNCTION__, @"不带参数，但是带有返回值");
  return @"不带参数，但是带有返回值";
}
```

然后发送消息，并接收返回值：

```
// 4.调用带返回值，但是不带参数
NSString *retValue = ((NSString * (*)(id, SEL))objc_msgSend)((id)msg, @selector(noArgumentsButReturnValue));
NSLog(@"5. 返回值为：%@", retValue);
```

打印结果说明成功地发送了消息并得到处理，且成功地获取到了返回值：

```
-[HYBMsgSend noArgumentsButReturnValue] was called, and return value is 不带参数，但是带有返回值
```

##5.带参数带返回值的消息

定义一个带参数带普通返回值的方法：

```
- (int)hasArguments:(NSString *)arg andReturnValue:(int)arg1 {
  NSLog(@"%s was called, and argument is %@, return value is %d", __FUNCTION__, arg, arg1);
  return arg1;
}
```

发送消息，并接收返回值：

```
// 6.带参数带返回值
int returnValue = ((int (*)(id, SEL, NSString *, int))
                 objc_msgSend)((id)msg,
                               @selector(hasArguments:andReturnValue:),
                               @"参数1",
                               2016);
NSLog(@"6. return value is %d", returnValue);
```

其结果如下，说明调用成功，参数也传过去了，返回值也接收到了：

```
-[HYBMsgSend hasArguments:andReturnValue:] was called, and argument is 参数1, return value is 2016
6. return value is 2016
```

##6.动态添加方法再调用

我们声明一个C语言函数：

```
// C函数
int cStyleFunc(id receiver, SEL sel, const void *arg1, const void *arg2) {
  NSLog(@"%s was called, arg1 is %@, and arg2 is %@",
        __FUNCTION__,
        [NSString stringWithUTF8String:arg1],
        [NSString stringWithUTF8String:arg1]);
  return 1;
}
```

这个函数并不属性对象方法，因此我们不能直接调用，但是我们可以动态地添加方法到对象中，然后再发送消息：

```
int returnValue = ((int (*)(id, SEL, NSString *, int))
                 objc_msgSend)(msg,
                               @selector(hasArguments:andReturnValue:),
                               @"参数1",
                               2016);
NSLog(@"6. return value is %d", returnValue);
NSLog(@"%s", @encode(const void *));
// 7.动态添加方法，然后调用C函数
class_addMethod(msg.class, NSSelectorFromString(@"cStyleFunc"), (IMP)cStyleFunc, "i@:r^vr^v");
returnValue = ((int (*)(id, SEL, const void *, const void *))
             objc_msgSend)((id)msg,
                           NSSelectorFromString(@"cStyleFunc"),
                           "参数1",
                           "参数2");
```

**提示：**"v@:r^vr^v"，其中`i`代表返回类型`int`，@代表参数接收者，:代表SEL，r^v是const void *

##7.带浮点返回值的消息

对于发送带浮点返回值类型的消息，我们可以使用`objc_msgSend_fpret`也可以使用`objc_msgSend`。不过这两个函数返回结果是一样的。笔者并不是很清楚，这两个函数的主要区别是什么。根据注释说明，只是说有一些处理器上，需要使用`objc_msgSend_fpret`来发送带浮点返回值类型的消息。

```
float retFloatValue = ((float (*)(id, SEL))objc_msgSend_fpret)((id)msg, @selector(returnFloatType));
NSLog(@"%f", retFloatValue);
  
retFloatValue = ((float (*)(id, SEL))objc_msgSend)((id)msg, @selector(returnFloatType));
NSLog(@"%f", retFloatValue);
```

##8.带结构体返回值的消息

对于返回值类型为结构体的消息，我们必须使用`objc_msgSend_stret`而不能直接使用`objc_msgSend`函数，否则会`crash`：

```
// 9.返回结构体时，不能使用objc_msgSend，而是要使用objc_msgSend_stret，否则会crash
CGRect frame = ((CGRect (*)(id, SEL))objc_msgSend_stret)((id)msg, @selector(returnTypeIsStruct));
NSLog(@"9. return value is %@", NSStringFromCGRect(frame));
```

#源代码

大家可以到GITHUB下载源代码：[https://github.com/CoderJackyHuang/RuntimeDemo](https://github.com/CoderJackyHuang/RuntimeDemo)

对应于`Message`分组。**请随手给个star支持一下吧！**

#写在最后

文章中难免有说得不合理的地方，如果您认为说法不正确或者哪里有错误的地方，请在评论中留言，笔者会在第一时间修正！！！

#关注我


关注                | 账号              | 备注
-------------      | -------------     | ----------------
Swift/ObjC技术群一  | 324400294         |  群一若已满，请申请群二
Swift/ObjC技术群二  | 494669518         | 群二若已满，请申请群三
Swift/ObjC技术群三  | 461252383         | 群三若已满，会有提示信息
关注微信公众号       | iOSDevShares      | 关注微信公众号，会定期地推送好文章
关注新浪微博账号      |  [标哥Jacky](http://weibo.com/u/5384637337) | 关注微博，每次发布文章都会分享到新浪微博
关注标哥的GitHub     | [CoderJackyHuang](https://github.com/CoderJackyHuang) | 这里有很多的Demo和开源组件
关于我               | [进一步了解标哥](http://www.henishuo.com/about-biaoge/) | 如果觉得文章对您很有帮助，可捐助我！


