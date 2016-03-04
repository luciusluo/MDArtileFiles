>#ios属性修饰

##引言
很多刚接触iOS的朋友对属性的`@property`的可选参数如何使用，什么情况下使用哪种选项不了解，也问了我很多这方面的知识，虽然知道怎么用，但是有些说不出其区别。在这里，再次深入学习一遍，对`copy/strong/weak/__weak/__strong/assign`的使用场景总结总结。如果有说得不对的地方，请指出。如果有疑问，请私聊我，或者直接回复我。

##自动引用计数
>原文档关于自动引用说明：Automatic Reference Counting (ARC) is a compiler feature that provides automatic memory management of Objective-C objects. Rather than having to think about retain and release operations, ARC allows you to concentrate on the interesting code, the object graphs, and the relationships between objects in your application.
>
>翻译过来就是：Automatic Reference Counting (ARC)是一个编译器的特性，提供了对iOS对象的自动内存管理,ARC在编译期间自动在适当的地方添加ObjC对象的retain和release操作代码，而不需要我们关心。

ARC在编译期间，根据Objective-C对象的存活周期，在适当的位置添加retain和release代码。从概念上讲，ARC与手动引用计数内存管理遵循同样的内存管理规则，但是**ARC也无法防止循环强引用**。

ARC还引入了新的修饰符来修饰变量和声明属性。

* 声明***变量***的修饰符：`__strong`, `__weak`, `__unsafe_unretained`, `__autoreleasing`;
* 声明***属性***的修饰符：`strong`, `weak`, `unsafe_unretained`。
* 对象和Core Foundation-style对象直接的转换修饰符号：`__bridge`，`__bridge_retained`或`CFBridgingRetain`, `__bridge_transfer`或` CFBridgingRelease`。
* 对于线程的安全，有`nonatomic`，这样效率就更高了，但是不是线程的。如果要线程安全，可以使用`atomic`，这样在访问是就会有线程锁。

> 记住内存管理法则：谁使对象的引用计数+1，不再引用时，谁就负责将该对象的引用计数-1。

下面我们来声明一个`Person`类来学习：

```
@interface Person : NSObject

// 注意：苹果有命名规范的，命名属性时，不能以copy开头。
// 如果下面的属性声明为copyString，会编译不通过。
@property (nonatomic, copy) NSString *copiedString;

// 默认会是什么呢？
@property (nonatomic) NSString *name;
// 默认是strong类型
@property (nonatomic) NSArray *array;

@end
```
如果属性没有指定类型，默认是什么呢？其实是`strong`。如果证明呢？验证方法：分别将`array`属性的类型分别设置为`weak`, `assign`,`strong`,不设置，这四种情况的结果分别是：第一种打印为空，第二种直接直接崩溃，第三种和最后一种是可以正常使用。如下面的验证代码：

```
  Person *lili = [[Person alloc] init];
  lili.name = @"LiLi";
  lili.copiedString = @"LiLi\' father is LLL";
  lili.array = @[@"谢谢", @"感谢"];
  
  NSArray *otherArray = lili.array;
  lili = nil;
  NSLog(@"%@", otherArray);
```

再继续添加下面的代码。默认声明变量的类型为`__strong`类型，因此上面的`NSArray *otherArray = lili.array;`与`__strong NSArray *otherArray = lili.array;`是一样的。如果我们要使用弱引用，特别是在解决循环强引用时就特别重要了。我们可以使用`__weak`声明变量为弱引用，这样就不会增加引用计数值。

```
  __strong NSArray *strongArray = otherArray;
  otherArray = nil;
  // 打印出来正常的结果。
  NSLog(@"strongArray = %@", strongArray);
  
  __weak NSArray * weakArray = strongArray;
  strongArray = nil;
  // 打印出来：null
  NSLog(@"weakArray: %@", weakArray);
```

####xib/storybard连接的对象为什么可以使用weak

```
@property (nonatomic, weak) IBOutlet UIButton *button;
```
像上面这行代码一样，在连接时自动生成为`weak`。因为这个button已经放到view上了，因此只要这个View不被释放，这个button的引用计数都不会为0，因此这里可以使用`weak`引用。

如果我们不使用`xib/storyboard`，而是使用纯代码创建呢？

```
@property (nonatomic, weak) UIButton *button;
```

> 补充：有朋友反映这里说得不够详细，感谢这位朋友的反馈。
> 
> 使用`weak`时，由于`button`在创建时，没有任何强引用，因此就有可能提前释放。Xcode编译器会告诉我们，这里不能使用`weak`。因此我们需要记住，只要我们在创建以后需要使用它，我们必须保证至少有一个强引用，否则引用计数为0，就会被释放掉。对于上面的代码，就是由于在创建时使用了`weak`引用，因此`button`的引用计数仍然为0，也就是会被释放，编译器在编译时会检测出来的。

这样写，在创建时通过`self.button = ...`就是出现错误，因为这是弱引用。所以我们需要声明为强引用，也就是这样：

```
@property (nonatomic, strong) UIButton *button;
```

####block声明使用copy

在使用block时，尽量使用`typedef`来起一个别名，这样更容易阅读。使block作为属性时，使用`copy`。

```
typedef void (^HYBTestBlock)(NSString *name);

@property (nonatomic, copy) HYBTestBlock testBlock;
```

####字符串
对于字符串，通常都是使用`copy`的方式。虽然使用`strong`似乎也没有没有问题，但是事实上在开发中都会使用`copy`。为什么这么做？因为对于字符串，我们希望是一次内容的拷贝，外部修改也不会影响我们的原来的值，而且NSString类遵守了`NSCopying`, `NSMutableCopying`, `NSSecureCoding`协议。

下面时使用`copy`的方式，验证如下：

```
  NSString *hahaString = @"哈哈";
  NSString *heheString = [hahaString copy];
  // 哈哈, 哈哈
  NSLog(@"%@, %@", hahaString, heheString);
  heheString = @"呵呵";
  // 哈哈, 呵呵
  NSLog(@"%@, %@", hahaString, heheString);
```
我们修改了`heheString`，并不会影响到原来的`hahaString`。`copy`一个对象变成新的对象(新内存地址) 引用计数为1 原来对象计数不变。

####属性声明修饰符
属性声明修饰符有：strong, weak, unsafe_unretained, readWrite，默认strong, readWrite的。

* strong：strong和retain相似,只要有一个strong指针指向对象，该对象就不会被销毁
* weak：声明为weak的指针，weak指针指向的对象一旦被释放，weak的指针都将被赋值为nil；
* unsafe\_unretained：用unsafe_unretained声明的指针，指针指向的对象一旦被释放，这些指针将成为野指针。

```
@property (nonatomic, copy) NSString *name;
// 一旦所指向的对象被释放，就会成为野指针
@property (nonatomic, unsafe_unretained) NSString *unsafeName;

lili.name = @"Lili";
lili.unsafeName = lili.name;
lili.name = nil;
// unsafeName就变成了野指针。这里不会崩溃，因为为nil.
NSLog(@"%@", lili.unsafeName);
```

####深拷贝与浅拷贝
关于浅拷贝，简单来说，就像是人与人的影子一样。而深拷贝就像是梦幻西游中的龙宫有很多个长得一样的龙宫，但是他们都是不同的精灵，因此他们各自都是独立的。

我相信还有不少朋友有这样一种**误解**：浅拷贝就是用`copy`，深拷贝就是用`mutableCopy`。如果有这样的误解，一定要更正过来。`copy`只是不可变拷贝，而`mutableCopy`是可变拷贝。比如，`NSArray *arr = [modelsArray copy]`，那么`arr`是不可变的。而`NSMutableArray *ma = [modelsArray mutableCopy]`，那么`ma`是可变的。

```
lili.array = [@[@"谢谢", @"感谢"] mutableCopy];
NSMutableArray *otherArray = [lili.array copy];
lili.array[0] = @"修改了谢谢";
// 打印： 谢谢 修改了谢谢
// 说明数组里面是字符串时，直接使用copy是相当于深拷贝的。
NSLog(@"%@ %@", otherArray[0], lili.array[0]);
```
这里就是浅拷贝，但是由于数组中的元素都是字符串，因此不会影响原来的值。

数组中是对象时：

```
  NSMutableArray *personArray = [[NSMutableArray alloc] init];
  Person *person1 = [[Person alloc] init];
  person1.name = @"lili";
  [personArray addObject:person1];
  
  Person *person2 = [[Person alloc] init];
  person2.name = @"lisa";
  [personArray addObject:person2];
  
  // 浅拷贝
  NSArray *newArray = [personArray copy];
  Person *p = newArray[0];
  p.name = @"lili的名字被修改了";
  
  // 打印结果：lili的名字被修改了
  // 说明这边修改了，原来的数组对象的值也被修改了。虽然newArray和personArray不是同一个数组，不是同一块内存，
  // 但是实际上两个数组的元素都是指向同一块内存。
  NSLog(@"%@", ((Person *)(personArray[0])).name);
```

深拷贝，其实就是对数组中的所有对象都创建一个新的对象：

```
  NSMutableArray *personArray = [[NSMutableArray alloc] init];
  Person *person1 = [[Person alloc] init];
  person1.name = @"lili";
  [personArray addObject:person1];
  
  Person *person2 = [[Person alloc] init];
  person2.name = @"lisa";
  [personArray addObject:person2];
  
  // 深拷贝
  NSMutableArray *newArray = [[NSMutableArray alloc] init];
  for (Person *p in personArray) {
    Person *newPerson = [[Person alloc] init];
    newPerson.name = p.name;

    [newArray addObject:newPerson];
  }
  Person *p = newArray[0];
  p.name = @"lili的名字被修改了";
  
  // 打印结果：lili
  NSLog(@"%@", ((Person *)(personArray[0])).name);
```

##Getter/Setter
在ARC下，getter/setter的写法与MRC的不同了。我面试过一些朋友，笔试这关就写得很糟（不包括算法）。通常在笔试时都会让重写一个属性的Getter/Setter方法。

```
@property (nonatomic, strong) NSMutableArray *array;

- (void)setArray:(NSMutableArray *)array {
  if (_array != array) {
    _array = nil;
    
    _array = array;
  }
}
```

如果是要重写getter就去呢？就得增加一个变量了，如果同时重写getter/setter方法，就不会自动生成_array变量，因此我们可以声明一个变量为`_array`:

```
- (void)setArray:(NSMutableArray *)array {
  if (_array != array) {
    _array = nil;
    
    _array = array;
  }
}

- (NSMutableArray *)array {
  return _array;
}
```


##总结
关于属性的这些选项的学习，做一下总结：

* 所有的属性，都尽可能使用`nonatomic`，以提高效率，除非真的有必要考虑线程安全。
* **NSString**：通常都使用`copy`，以得到新的内存分配，而不只是原来的引用。
* **strong**：对于继承于NSObject类型的对象，若要声明为强使用，使用`strong`，若要使用弱引用，使用`__weak`来引用，用于解决循环强引用的问题。
* **weak**：对于xib上的控件引用，可以使用weak，也可以使用strong。
* **__weak**：对于变量的声明，如果要使用弱引用，可以使用__weak，如：__weak typeof(Model) weakModel = model;就可以直接使用weakModel了。
* **__strong**：对于变量的声明，如果要使用强引用，可以使用__strong，默认就是__strong，因此不写与写__strong声明都是一样的。
* **unsafe_unretained**：这个是比较少用的，几乎没有使用到。在所引用的对象被释放后，该指针就成了野指针，不好控制。
* **__unsafe_unretained**：也是很少使用。同上。
* **__autoreleasing**：如果要在循环过程中就释放，可以手动使用__autoreleasing来声明将之放到自动释放池。

##参考资料
官方文档关于自动引用计数内存管理介绍：[https://developer.apple.com/library/mac/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html](https://developer.apple.com/library/mac/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)

>**公众号搜索「iOS开发技术分享」快速关注微信号：iOSDevShares QQ群：324400294**

![image](https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD0wkCgnmf6DVxI6fVewNS8ruMXjnY6iazpgH0p4hDn1vrIxN0H4pAnxDLIXlOzpjicwWgmaSDu5W0Zw/0?wx_fmt=jpeg)
