>#[原文出自：标哥的技术博客](http://www.henishuo.com/runtime-method-swizzling/)

#前言

在我学习runtime的method swizzling特性之前，有很多同事或者朋友经常在我耳边说起swizzling特性，一个个在我面前说这个东西千万不能用，会引起很多问题的。但是，在我学习完这一节的知识后，我终于明白其所以然。

学习完swizzling特性后，我很喜欢她。她就像一把双刃剑，用好了可以带你飞，乱用则会反伤。但是，我更相信她的强大，更相信自己够能驾驭她！一起来学习吧！

#Method Swizzling

试想一下，苹果的源码是闭源的，我们只有类名和类的属性、方法等声明，却看不到实现，这时候我们若想改变其中一个方法的实现，有哪些方案呢？笔者想到的有以下几种方案：

1. 继承于这个类，然后通过重写方法（很常用，比如基类控制器，可以在视图加载完成时做一些公共的配置等）
2. 通过类别重写方法，暴力抢先（此法太暴力，尽量不要这么做）
3. swizzling（本文特讲内容）

##Swizzling原理

在Objective-C中调用一个方法，其实是向一个对象发送消息，而查找消息的唯一依据是selector的名字。所以，我们可以利用Objective-C的runtime机制，实现在运行时交换selector对应的方法实现以达到我们的目的。

每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系。IMP有点类似函数指针，指向具体的Method实现。如果对Method不了解，请阅读笔者的文章：[runtime Method精讲](http://www.henishuo.com/runtime-method-in-detail/)。

我们先看看SEL与IMP之间的关系图（图片来源http://blog.csdn.net/yiyaaixuexi/article/details/9374411）：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/153752_G4aM_1463495.jpg)

从上图可以看出来，每一个SEL与一个IMP一一对应，正常情况下通过SEL可以查找到对应消息的IMP实现。

但是，现在我们要做的就是把链接线解开，然后连到我们自定义的函数的IMP上。当然，交换了两个SEL的IMP，还是可以再次交换回来了。交换后变成这样的，如下图（图片来源http://blog.csdn.net/yiyaaixuexi/article/details/9374411）：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/154037_8i4R_1463495.png)

从图中可以看出，我们通过swizzling特性，将selectorC的方法实现IMPc与selectorN的方法实现IMPn交换了，当我们调用selectorC，也就是给对象发送selectorC消息时，所查找到的对应的方法实现就是IMPn而不是IMPc了。

##在+load方法中交换

Swizzling应该在+load方法中实现，因为+load方法可以保证在类最开始加载时会调用。因为method swizzling的影响范围是全局的，所以应该放在最保险的地方来处理是非常重要的。+load能够保证在类初始化的时候一定会被加载，这可以保证统一性。试想一下，若是在实际时需要的时候才去交换，那么无法达到全局处理的效果，而且若是临时使用的，在使用后没有及时地使用swizzling将系统方法与我们自定义的方法实现交换回来，那么后续的调用系统API就可能出问题。

类文件在工程中，一定会加载，因此可以保证+load会被调用。

##不要在+initialize中交换

+initialize是类第一次初始化时才会被调用，因为这个类有可能一直都没有使用到，因此这个类可能永远不会被调用。

类文件虽然在工程中，但是如果没有任何地方调用过，那么是不会调用+initialize方法的。

##使用dispatch\_once保证只交换一次

方法交换应该要线程安全，而且保证只交换一次，除非只是临时交换使用，在使用完成后又交换回来。

最常用的用法是在+load方法中使用dispatch\_once来保证交换是安全的。因为swizzling会改变全局，我们需要在运行时采取相应的防范措施。保证原子操作就是一个措施，确保代码即使在多线程环境下也只会被执行一次。而diapatch\_once就提供这些保障，因此我们应该将其加入到swizzling的使用标准规范中。

##通用交换IMP写法

网上有很多的版本，但是有很多是不全面的，考虑的范围不够全面。下面我们来写一个通用的写法，现在扩展到NSObject中，因为NSObject是根类，这样其它类都可以使用了：

```
@interface NSObject (Swizzling)

+ (void)swizzleSelector:(SEL)originalSelector withSwizzledSelector:(SEL)swizzledSelector;

@end

#import "NSObject+Swizzling.h"
#import <objc/runtime.h>


// 实现代码如下
@implementation NSObject (Swizzling)

+ (void)swizzleSelector:(SEL)originalSelector withSwizzledSelector:(SEL)swizzledSelector {
  Class class = [self class];
  
  Method originalMethod = class_getInstanceMethod(class, originalSelector);
  Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
  
  // 若已经存在，则添加会失败
  BOOL didAddMethod = class_addMethod(class,
                                      originalSelector,
                                      method_getImplementation(swizzledMethod),
                                      method_getTypeEncoding(swizzledMethod));
  
  // 若原来的方法并不存在，则添加即可
  if (didAddMethod) {
    class_replaceMethod(class,
                        swizzledSelector,
                        method_getImplementation(originalMethod),
                        method_getTypeEncoding(originalMethod));
  } else {
    method_exchangeImplementations(originalMethod, swizzledMethod);
  }
}

@end
```

因为方法可能不是在这个类里，可能是在其父类中才有实现，因此先尝试添加方法的实现，若添加成功了，则直接替换一下实现即可。若添加失败了，说明已经存在这个方法实现了，则只需要交换这两个方法的实现就可以了。

尽量使用`method_exchangeImplementations`函数来交换，因为它是原子操作的，线程安全。尽量不要自己手动写这样的代码：

```
IMP imp1 = method_getImplementation(m1);
IMP imp2 = method_getImplementation(m2);
method_setImplementation(m1, imp2);
method_setImplementation(m2, imp1);
```

虽然`method_exchangeImplementations`函数的本质也是这么写法，但是它内部做了线程安全处理。

当然，我们也可以写成C语言函数，而不是归属于类的方法：

```
// C语言版
void swizzleSelector(Class class, SEL originalSelector, SEL swizzledSelector) {
  Method originalMethod = class_getInstanceMethod(class, originalSelector);
  Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
  
  // 若已经存在，则添加会失败
  BOOL didAddMethod = class_addMethod(class,
                                      originalSelector,
                                      method_getImplementation(swizzledMethod),
                                      method_getTypeEncoding(swizzledMethod));
  
  // 若原来的方法并不存在，则添加即可
  if (didAddMethod) {
    class_replaceMethod(class,
                        swizzledSelector,
                        method_getImplementation(originalMethod),
                        method_getTypeEncoding(originalMethod));
  } else {
    method_exchangeImplementations(originalMethod, swizzledMethod);
  }
}
```

##简单使用swizzling

最简单的方法实现交换如下：

```
Method originalMethod = class_getInstanceMethod([NSArray class], @selector(lastObject));

Method newMedthod = class_getInstanceMethod([NSArray class], NSSelectorFromString(@"hyb_lastObject"));

method_exchangeImplementations(originalMethod, newMedthod);

// NSArray提供了这样的实现
- (id)hyb_lastObject {
  if (self.count == 0) {
    NSLog(@"%s 数组为空，直接返回nil", __FUNCTION__);
    
    return nil;
  }
  
  return [self hyb_lastObject];
}
```

看到`hyb_lastObject`这个方法递归调用自己了吗？为什么不是调用`return [self lastObject]`？因为我们交换了方法的实现，那么系统在调用`lastObject`方法是，找的是`hyb_lastObject`方法的实现，而手动调用`hyb_lastObject`方法时，会调用`lastObject`方法的实现。不清楚？回到前面看一看那个交换IMP的图吧！

我们通过使用swizzling只是为了添加个打印？当然不是，我们还可以做很多事的。比如，上面我们还做了防崩溃处理。

##NSMutableArray扩展交换处理崩溃

还记得那些调用数组的`addObject:`方法加入一个nil值是的崩溃情景吗？还记得`[__NSPlaceholderArray initWithObjects:count:]`因为有nil值而崩溃的提示吗？还记得调用`objectAtIndex:`时出现崩溃提示empty数组问题吗？那么通过swizzling特性，我们可以做到不让它崩溃，而只是打印一些有用的日志信息。

我们先来看看NSMutableArray的扩展实现：

```
#import "NSMutableArray+Swizzling.h"
#import <objc/runtime.h>
#import "NSObject+Swizzling.h"

@implementation NSMutableArray (Swizzling)

+ (void)load {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    [self swizzleSelector:@selector(removeObject:)
     withSwizzledSelector:@selector(hyb_safeRemoveObject:)];
    
    [objc_getClass("__NSArrayM") swizzleSelector:@selector(addObject:)
     withSwizzledSelector:@selector(hyb_safeAddObject:)];
    [objc_getClass("__NSArrayM") swizzleSelector:@selector(removeObjectAtIndex:)
                            withSwizzledSelector:@selector(hyb_safeRemoveObjectAtIndex:)];
 
    [objc_getClass("__NSArrayM") swizzleSelector:@selector(insertObject:atIndex:)
     withSwizzledSelector:@selector(hyb_insertObject:atIndex:)];
    
    [objc_getClass("__NSPlaceholderArray") swizzleSelector:@selector(initWithObjects:count:) withSwizzledSelector:@selector(hyb_initWithObjects:count:)];
    
    [objc_getClass("__NSArrayM") swizzleSelector:@selector(objectAtIndex:) withSwizzledSelector:@selector(hyb_objectAtIndex:)];
  });
}

- (instancetype)hyb_initWithObjects:(const id  _Nonnull __unsafe_unretained *)objects count:(NSUInteger)cnt {
  BOOL hasNilObject = NO;
  for (NSUInteger i = 0; i < cnt; i++) {
    if ([objects[i] isKindOfClass:[NSArray class]]) {
      NSLog(@"%@", objects[i]);
    }
    if (objects[i] == nil) {
      hasNilObject = YES;
      NSLog(@"%s object at index %lu is nil, it will be filtered", __FUNCTION__, i);
      
//#if DEBUG
//      // 如果可以对数组中为nil的元素信息打印出来，增加更容易读懂的日志信息，这对于我们改bug就好定位多了
//      NSString *errorMsg = [NSString stringWithFormat:@"数组元素不能为nil，其index为: %lu", i];
//      NSAssert(objects[i] != nil, errorMsg);
//#endif
    }
  }
  
  // 因为有值为nil的元素，那么我们可以过滤掉值为nil的元素
  if (hasNilObject) {
    id __unsafe_unretained newObjects[cnt];
    
    NSUInteger index = 0;
    for (NSUInteger i = 0; i < cnt; ++i) {
      if (objects[i] != nil) {
          newObjects[index++] = objects[i];
      }
    }
    
    return [self hyb_initWithObjects:newObjects count:index];
  }

  return [self hyb_initWithObjects:objects count:cnt];
}


- (void)hyb_safeAddObject:(id)obj {
  if (obj == nil) {
    NSLog(@"%s can add nil object into NSMutableArray", __FUNCTION__);
  } else {
    [self hyb_safeAddObject:obj];
  }
}

- (void)hyb_safeRemoveObject:(id)obj {
  if (obj == nil) {
    NSLog(@"%s call -removeObject:, but argument obj is nil", __FUNCTION__);
    return;
  }
  
  [self hyb_safeRemoveObject:obj];
}

- (void)hyb_insertObject:(id)anObject atIndex:(NSUInteger)index {
  if (anObject == nil) {
    NSLog(@"%s can't insert nil into NSMutableArray", __FUNCTION__);
  } else if (index > self.count) {
    NSLog(@"%s index is invalid", __FUNCTION__);
  } else {
    [self hyb_insertObject:anObject atIndex:index];
  }
}

- (id)hyb_objectAtIndex:(NSUInteger)index {
  if (self.count == 0) {
    NSLog(@"%s can't get any object from an empty array", __FUNCTION__);
    return nil;
  }
  
  if (index > self.count) {
    NSLog(@"%s index out of bounds in array", __FUNCTION__);
    return nil;
  }
  
  return [self hyb_objectAtIndex:index];
}

- (void)hyb_safeRemoveObjectAtIndex:(NSUInteger)index {
  if (self.count <= 0) {
    NSLog(@"%s can't get any object from an empty array", __FUNCTION__);
    return;
  }
  
  if (index >= self.count) {
    NSLog(@"%s index out of bound", __FUNCTION__);
    return;
  }

  [self hyb_safeRemoveObjectAtIndex:index];
}

@end
```

然后，我们测试nil值的情况，是否还会崩溃呢？

```
NSMutableArray *array = [@[@"value", @"value1"] mutableCopy];
[array lastObject];
  
[array removeObject:@"value"];
[array removeObject:nil];
[array addObject:@"12"];
[array addObject:nil];
[array insertObject:nil atIndex:0];
[array insertObject:@"sdf" atIndex:10];
[array objectAtIndex:100];
[array removeObjectAtIndex:10];
  
NSMutableArray *anotherArray = [[NSMutableArray alloc] init];
[anotherArray objectAtIndex:0];
  
NSString *nilStr = nil;
NSArray *array1 = @[@"ara", @"sdf", @"dsfdsf", nilStr];
NSLog(@"array1.count = %lu", array1.count);
  
// 测试数组中有数组
NSArray *array2 = @[@[@"12323", @"nsdf", nilStr], @[@"sdf", @"nilsdf", nilStr, @"sdhfodf"]];
```

哈哈，都不崩溃了，而且还打印出崩溃原因。是不是很神奇？如果充分利用这种特性，是不是可以给我们带来很多便利之处？

上面只是swizzling的一种应用场景而已。其实利用swizzling特性还可以做很多事情的，比如处理按钮重复点击问题等。

#源代码

大家可以到笔者的GITHUB下载对应的源代码来测试：[https://github.com/CoderJackyHuang/RuntimeDemo](https://github.com/CoderJackyHuang/RuntimeDemo)

**喜欢就随手给个star吧！**

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

