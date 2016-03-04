>#[原文出自：标哥的技术博客](http://www.henishuo.com/runtime-method-in-detail/)


#前言

本篇文章只讲`Method`的特性及相关方法，不讲`Method Swizzling`特性。关于`Method Swizzling`特性，我们放在单独的一篇文章来细讲，因为这一节非常重要。

#Method类型

Method类型是一个`objc_method`结构体指针，而结构体`objc_method`有三个成员：

```
/// An opaque type that represents a method in a class definition.
typedef struct objc_method *Method;

struct objc_method {
    SEL method_name; 		// 方法名称
    char *method_typesE;	// 参数和返回类型的描述字串
    IMP method_imp;			// 方法的具体的实现的指针
} 
```

#Method所有方法

下面是官方所提供的所有Method的方法，我们一一说明其用途，看代码注释：

```
// 函数调用，但是不接收返回值类型为结构体
method_invoke
// 函数调用，但是接收返回值类型为结构体
method_invoke_stret
// 获取函数名
method_getName
// 获取函数实现IMP
method_getImplementation
// 获取函数type encoding
method_getTypeEncoding
// 复制返回值类型
method_copyReturnType
// 复制参数类型
method_copyArgumentType
// 获取返回值类型
method_getReturnType
// 获取参数个数
method_getNumberOfArguments
// 获取函数参数类型
method_getArgumentType
// 获取函数描述
method_getDescription
// 设置函数实现IMP
method_setImplementation
// 交换函数的实现IMP
method_exchangeImplementations
```

#获取函数列表

我们尝试获取函数列表，并细说函数的参数type encoding、返回值类型等。我们先写以下几个方法：

```
- (int)testInstanceMethod:(NSString *)name andValue:(NSNumber *)value {
  NSLog(@"%@", name);
  
  return value.intValue;
}

- (NSArray *)arrayWithNames:(NSArray *)names {
  NSLog(@"%@", names);
  return names;
}

- (void)getMethods {
  unsigned int outCount = 0;
  Method *methodList = class_copyMethodList(self.class, &outCount);
  
  for (unsigned int i = 0; i < outCount; ++i) {
    Method method = methodList[i];
    
    SEL methodName = method_getName(method);
    NSLog(@"方法名：%@", NSStringFromSelector(methodName));
    
    // 获取方法的参数类型
    unsigned int argumentsCount = method_getNumberOfArguments(method);
    char argName[512] = {};
    for (unsigned int j = 0; j < argumentsCount; ++j) {
      method_getArgumentType(method, j, argName, 512);
      
      NSLog(@"第%u个参数类型为：%s", j, argName);
      memset(argName, '\0', strlen(argName));
    }
    
    char returnType[512] = {};
    method_getReturnType(method, returnType, 512);
    NSLog(@"返回值类型：%s", returnType);
    
    // type encoding
    NSLog(@"TypeEncoding: %s", method_getTypeEncoding(method));
  }
  
  free(methodList);
}
```

然后，我们写一个测试方法来调用一下：

```
+ (void)test {
  HYBMethodLearn *m = [[HYBMethodLearn alloc] init];
  [m getMethods];
  
  // 这就是为什么有四个参数的原因
  int returnValue = ((int (*)(id, SEL, NSString *, NSNumber *))objc_msgSend)((id)m, @selector(testInstanceMethod:andValue:), @"标哥的技术博客", @100);
  NSLog(@"return value is %d", returnValue);
}
```

其打印结果如下：

```
方法名：getMethods
第0个参数类型为：@
第1个参数类型为：:
返回值类型：v
TypeEncoding: v16@0:8
方法名：testInstanceMethod:andValue:
第0个参数类型为：@
第1个参数类型为：:
第2个参数类型为：@
第3个参数类型为：@
返回值类型：i
TypeEncoding: i32@0:8@16@24
方法名：arrayWithNames:
第0个参数类型为：@
第1个参数类型为：:
第2个参数类型为：@
返回值类型：@
TypeEncoding: @24@0:8@16
标哥的技术博客
return value is 100
```

从打印结果中，可以看到好多好奇怪的符号。

##获取函数名

我们通过`method_getName()`函数来获取方法`SEL`：

```
SEL methodName = method_getName(method);
NSLog(@"方法名：%@", NSStringFromSelector(methodName));
```

##为什么多了两个参数

通过`method_getNumberOfArguments`获取到函数的所有参数类型，从上面我们所定义的函数中，比如`getMethods`明明没有参数，为什么会打印出来两个参数呢？而`- (int)testInstanceMethod:(NSString *)name andValue:(NSNumber *)value`明明只有两个参数，为什么有四个参数呢？下面我们来细说：

首先，对于第一个方法，它在编译时会被转换成类似这样：

```
((void (*)(id, SEL))objc_msgSend)((id)m, @selector(getMethods));
```

这样一看，知道是有两个参数了吧？

同样的道理，对于第二个方法，在编译时编译器会将其转换成类似这样：

```
// 这就是为什么有四个参数的原因
int returnValue = ((int (*)(id, SEL, NSString *, NSNumber *))
                 objc_msgSend)((id)m,
                               @selector(testInstanceMethod:andValue:),
                               @"标哥的技术博客",
                               @100);
```

##函数编码

通过`method_getTypeEncoding`获取函数的编码，其结果是一串值。

* 第一个方法的编码为：v16@0:8
* 第二个方法的编码为：i32@0:8@16@24
* 第三个方法的编码为：@24@0:8@16

这么一看，可以看出来什么呢？从这几个值可以看出：

* 第一个位置是返回值类型，比如第一个方法返回值是V，第二个的是i，第三个的是@
* 第二/三个位置是第一/二个参数，参数列表从左到右算。分别是@ :，@ :，@ :，都是对象，其实第一个和第二个参数是固定的，第一个是接收消息的对象，而第二个是方法选择器SEL。
* 如果还有其它参数，依次...

但是类型后面跟着的数字是什么呢？其实笔者也不清楚，文档没有明确说明，不过从其打印结果可以看得出来其规律。比如第一个方法的：@的偏移为0、:的偏移为8、v的偏移为16，其它方法也是类似。

#method_invoke

除了使用`objc_msgSend`函数之外，还可以使用`method_invoke`，如：

```
// 获取方法
Method method = class_getInstanceMethod([self class], @selector(testInstanceMethod:andValue:));
  
// 调用函数
returnValue = ((int (*)(id, Method, NSString *, NSNumber *))method_invoke)((id)m, method, @"测试使用method_invoke", @11);
NSLog(@"call return vlaue is %d", returnValue);

// 与下面的调用可以得到同样的效果
int returnValue = ((int (*)(id, SEL, NSString *, NSNumber *))
                 objc_msgSend)((id)m,
                               @selector(testInstanceMethod:andValue:),
                               @"标哥的技术博客",
                               @100);
NSLog(@"return value is %d", returnValue);
```

#Type Encoding

下面是官方给出的所有类型编码，数据类型的编码最终值会有可能是下面中的多个的组合：

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

#源代码

大家可以到我的github下载本教程的demo：[RuntimeDemo](https://github.com/CoderJackyHuang/RuntimeDemo)

**喜欢就加个star**


#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)


