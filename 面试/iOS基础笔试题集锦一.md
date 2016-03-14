
#前言

以下部分题目来源于网络，笔者在此处收集起来，既是要巩固自我，也希望能够帮助到同样需要的人！参考答案均为笔者所写，其有疑问或者出错之处，请在评论中提出，谢谢！不喜勿喷！

##1. #import和#include的区别？

**参考答案：**

`#import`是`Objective-C`导入头文件的语法，可保证不会重复导入。<br>
`#include`是`C/C++`导入头文件的语法，如果是`Objective-C`与`C/C++`混编码，对于`C/C++`类型的文件，还是使用`#include`来引入，这种写法需要添加防重复导入的语法。

##2. @class的作用

`@class`一般用于头文件中通过前向声明，就可以声明了，但是在`.m`文件中还是需要使用`#import`进来的。它的作用只是前向声明。

##3. 用NSLog函数输出一个浮点类型，结果四舍五入，并保留一位小数

**参考答案：**

```
float money = 1.011;
NSLog(@"%.1f", money);
```
使用`%f`来格式化，其中要保留一位小数，因此再用`%.1f`就是保留一位。

##4.property属性的修饰符有什么样的作用

**参考答案：**

`property`是属性访问声明，扩号内支持以下几个属性：

* `getter=getName`、`setter=setName`：设置`setter`与`getter`的方法名
* `readwrite`、`readonly`：设置可供访问级别
* `assign`：方法直接赋值，不进行任何retain操作，为了解决原类型与环循引用问题
* `retain`：其`setter`方法对参数进行`release`旧值再`retain`新值，所有实现都是这个顺序
* `copy`：其`setter`方法进行`copy`操作，与`retain`处理流程一样，先对旧值`release`，再`copy`出新的对象，`retainCount`为1。这是为了减少对上下文的依赖而引入的机制。
* `nonatomic`：非原子性访问，不加同步， 多线程并发访问会提高性能。注意，如果不加此属性，则默认是两个访问方法都为原子型事务访问。

这里有一篇文章介绍：[iOS中的property的修饰符如何使用](http://www.henishuo.com/ios-property/)
 

##5. self.name=@"object"和name=@"object"有什么不同?

**参考答案：**

`self.name =”object”`：会调用对象的`setName()`方法；
`name = “object”`：会直接把`"object"`字符串赋值给当前对象的`name`属性。

##6. viewDidLoad、loadView和viewDidUnload何时调用

**参考答案：**

`viewDidLoad`在`view`加载完成时调用，`loadView`在`controller`的`view`为`nil`时调用。对于`viewDidUnload`现在已经不能直接调用了。

##7. objective-c中的可变与不可变词典

**参考答案：**

可变字典就是可以增、删、改操作的字典，对应于`NSMutableDictionary`类型。<BR>
不可变字典就是不能执行增、删、改操作的字典，对应于`NSDictionary`类型。

##8.Objective-C的内存管理

**参考答案：**
现在内存管理几乎都采用`ARC`，也就是`Automatic Reference Counting`，意思是自动引用计数。由编译器在编译时自动为添加`retain`、`release`等代码。

如果问的`MRC`，也就是`Manual Reference Counting`，意思是手动内存管理。

>黄金法则：谁使对象的引用计数+1，不再使用该对象时，谁就应该使该对象的引用计数-1。

##9. 自动生成getter/setter方法

**参考答案：**
对于以前的代码，那时还没有`property`，使用这样的方法来创建：

```
- (void)setName:(NSString *)aName;
- (NSString *)name;
```

在后面有了`property`，直接使用`@property (nonatomic, copy) NSString *name`这样的方法来声明，编译器会自动生成`getter/setter`方法并生成一个`_name`成员变量。

##10. 什么是MVC

**参考答案：**

我相信大部分人在被问到这个问题时，都会回答`M`就是`Model`，`V`就是`View`，`C`就是`Controller`。这都是停留在概念上的回答，明显没有什么工作经验。对于一个对框架和架构有一定的思想的人，回答时会从项目的耦合度、团队开发如何减少冲突、如何降低团队与团队之间的沟通成本、如何将`M`、`V`、`C`之间按照既定的标准建立沟通的桥梁。

`Model`用于处理数据，通常来说，`Model`中会包含多个字段，用于存储数据。但是，`Model`还会有一部分逻辑，比如说：

```
@interface TestModel: HYBBaseModel 

// 这个是接口返回的字段，1表示XXX，2表示YYY，3表示ZZZ
@property (nonatomic, assign) NSUInteger type;

// 这个不是接口返回的字段，但是由于`type`字段是一个数值，不是`view`需要显示的数据
// 所以我们最好将逻辑统一放到这里来，外部只管获取最终显示需要的值即可。即使哪天接口
// 返回的字段变化或者增加什么新的值，只需要处理这个模型内部就好了。 
@property (nonatomic, copy, readonly) NSString relationship;

@end
```

对于`View`，不应该包含逻辑，应该根据模型直接获取数据。

对于`Controller`，大部分交互逻辑都集中到了这里，所有`View`需要的数据，都是通过`Controller`提取`Model`然后交给`view`去显示数据。

##11. 重写getter/setter方法

假设声明属性：

```
@property (nonatomic, copy) NSString *blogName;
```

重写这个属性的`getter/setter`方法：

**参考答案：**

这里一旦连`getter`方法也重写，编译器不会给我们自动生成成员变量`_blogName`，因此我们需要在类的声明中添加一个成员变量`_blogName`：

```
@interface Demo () {
   NSString *_blogName;
}

@end
```

在自动内存管理下（`ARC`）：

```
- (void)setBlogName:(NSString *)aName {
   if (_blogName != aName) {
      _blogName = nil;
      _blogName = [aName copy];
   }
}

- (NSString *)blogName {
  return _blogName;
}
```

对于手动内存管理（`MRC`）：

```
- (void)setBlogName:(NSString *)aName {
   if (_blogName != aName) {
      [_blogName release];
      _blogName = nil;
      _blogName = [aName copy];
   }
}

- (NSString *)blogName {
  return _blogName;
}
```

##12. obj在编译时和运行时分别时什么类型的对象

如下面的代码，`obj`在编译时和运行时分别时什么类型的对象：

```
NSString *obj = [[NSData alloc] init];
```
**参考答案：**

在编译时，我们所声明的`obj`是`NSString *`类型，因此是`NSString`类型对象。在运行时，由于指针`obj`所指向的是`NSData`类型对象的内存，因此实际上是`NSData`类型的对象。在编译时，这一行代码会转换成类似这样：

```
NSString *obj = ((id (*)(id, SEL))objc_msgSend)([NSData class], @selector(alloc));
obj = ((id (*)(id, SEL))objc_msgSend)((id)obj, @selector(init));
```

由于在编译时，转换成`id`，因此可以用`NSString *`指向`NSData`对象，而`id`是具备运行时特性的，因此在链接时，通过`id`的`isa`指针可以找到其所属的类，因此最终类型还是通过`isa`确定其所属类型。

##13. id声明的对象有什么特性？


`id`类型可以指向任何类型的对象。

**参考答案：**

我们先看看其定义：

```
/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

可其定义可知`id`类型是一个指向`objc_object`结构体类型的指针，这个结构体只有一个指向对象无类的指针`isa`，因此`id`可以指向任何类型的对象，故其具备运行时特性。

##14. iOS设备性能测试

在实际开发中，我们经常需要对应用瘦身，因此对性能的检测是很重要的。

**参考答案：**

使用`Profile`-> `Instruments` ->`Time Profiler`可以检测性能。

##15. Objective-C中有私有方法、私有变量么？

我记得曾经我就被这么问过，不知道大家有没有遇到过。

**参考答案：**

在类的`.m`实现文件内声明，就可以作为私有方法、私有变量。但是，并不是绝对的私有，如果外部知道有这么个方法，一样可以调用，而且不会报错。就像苹果公司没有公开出来的`API`，只要我们通过其它方式了解到`api`就可以调用。于是苹果审核时经常由于使用了私有`api`而打回来了。

##16. 简述tableview的重用机制

曾经笔者面试时，也被问到这个问题。

**参考答案：**

```
[[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentifier]
```

* 这个方法就是重用机制的核心了。比如，有一个界面可显示10个cell，那么创建10次cell，并给`cell`指定同样的重用标识(当然，可以为不同显示类型的cell指定不同的标识)并且10个`cell`将全部都加入到`visiableCells`数组，`reusableTableCells`为空.

* 滚动`tableView`，当有一个`cell`完全移出屏幕时，这个`cell`就会被加入到`reusableTableCells `。而新出现的那个`cell`将加入到`visiableCells`，而这个`cell`就是被重用的。

如果要让`tableview`不重用，不设置`reuseIdentifier`就可以了。

##17. nil与NULL的区别

**参考答案：**

`nil`和`C`语言的`NULL`相同，在`objc/objc.h`中定义。`nil`表示`Objective-C`对象的值为空。在`C`语言中，指针的空值用`NULL`表示。在`Objective-C`中，`nil`对象调用任何方法表示什么也不执行，也不会崩溃。

更详细请阅读笔者的文章：[http://www.henishuo.com/nil-nil-null-nsnull-difference/](http://www.henishuo.com/nil-nil-null-nsnull-difference/)

##18. Category是什么，何时使用？

**参考答案：**

`Category`就是所谓的扩展。

有时我们需要在一个已经定义好的类中增加一些方法，而不想去重写该类，这时候使用扩展就很好。比如，当工程已经很大，代码量比较多，或者类中已经有很多方法，已经有其他代码调用了该类创建对象并使用该类的方法时，可以使用类别对该类扩充新的方法。

笔者所到公司之处，都会根据公司的`UI`风格定制一套`UI`组件，统一全局的风格。本人向来不喜欢用`xib/storyboard`开发，因为维护成本太高了。我们不能通过继承的方式定制各种组件吧？所以这个时候使用扩展是最佳时期。

##19. 什么是Delegate？常用场景？

**参考答案：**

`Delegate`就是所谓的代理，代理是一种设计模式。在`iOS`开发中，会使用到大量的代理，而代理设计模式是苹果中的标准设置模式。

常用场景有反向传值。比如：苹果的蓝牙，我们进入到下一个界面去打开或者关闭蓝牙，当操作之后需要将状态反馈到前一个界面，并更新显示。对于这种状态，使用代理设计模式是很标准的模式。

##20. 什么是单例，如何设计单例？

**参考答案：**

单例就是全局都只有一个对象存在，而且是在整个`App`运行过程中都存在。每个`App`都会有单例，比如`UIApplication`。而我们在做用户数据存储时，通常都会用单例存储，因为应用在所有操作中，经常要求先登录。

下面这种写法是最常用的写法，这个是线程安全的。

```
+ (instancetype)shared {
  static HYBUserManager *sg_userManager = nil;
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    if (sg_userManager == nil) {
      sg_userManager = [[HYBUserManager alloc] init];
    }
  });
  
  return sg_userManager;
}
```

##21. 什么是通知？

**参考答案：**

在`iOS`中，通知是非常常用的设计模式。它是多对多的关系。关于通知，由于这一节比较重要，单独写成一篇文章，请[阅读原文](http://www.henishuo.com/ios-notification/)

#写在最后

文章中难免有说得不合理的地方，如果您认为说法不正确或者哪里有错误的地方，请在评论中留言，笔者会在第一时间修正！！！

下一篇：[http://www.henishuo.com/objc-interview-two](http://www.henishuo.com/objc-interview-two/)

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

#支持并捐助

如果您觉得文章对您很有帮助，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
