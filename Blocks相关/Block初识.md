# 一、概述

Block是C级别的语法和运行时特性。Block比较类似C函数，但是Block比之C函数，其灵活性体现在栈内存、堆内存的引用，我们甚至可以将一个Block作为参数传给其他的函数或者Block（嵌套）。在实际开发中，Block是使用非常广泛的，可以说它与GCD是绝配。如果GCD没有了Block，也许一切都不一样了！

# 二、声明和使用Block

我们使用^操作符来声明一个block变量和表明block的开始。Block体是由花括号{开始到花括号}结束。比如下面一个小例子：

```
int multiplier = 7;
int (^myBlock)(int) = ^(int num) {
    return num * multiplier;
}; // 变量声明是需要;的哦！
```

描述如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160325-0@2x-e1458870374485.png)

可能您也注意到了，block可以使用同一作用域范围的变量，比如上面的multiplier与myBlock变量是同一作用域的。

如果我们声明block为变量，那么我们可以像函数调用一样来调用它：

```
int multiplier = 7;
int (^myBlock)(int) = ^(int num) {
    return num * multiplier;
};
 
printf("%d", myBlock(3));
// prints "21"
```

# 三、直接使用block

在很多时候，我们并不需要声明block变量；相反，我们更多地是直接简单地写block内联体作为参数。下面我们通过qsort_b这个c函数来实现快速排序，它的最后一个参数是一个闭包，用于决定ASC还是DSC排序：

```
char *names[4] = {"henishuo.com", "weibo.com/huangyibiao520", "GITHUB", "CSDN"};

qsort_b(names, 4, sizeof(char *), ^int(const void *name1, const void *name2) {
	char *lhs = *(char **)name1;
	char *rhs = *(char **)name2;
	    
	return strcmp(lhs, rhs);
});

for (NSUInteger i = 0; i < 4; ++i) {
	NSLog(@"%s", names[i]);
}
```

# 四、Cocoa中的Block

在Cocoa中有很多的方法是使用block作为参数的，最典型的就是集合对象的操作。比如，最常见的就是NSArray的排序API，需要指定一个排序规则，都是通过block来实现的。比如下面的排序：

```
NSArray *stringsArray = @[ @"string 1",
                           @"String 21",
                           @"string 12",
                           @"String 11",
                           @"String 02" ];

NSStringCompareOptions comparisonOptions = NSCaseInsensitiveSearch
| NSNumericSearch
| NSWidthInsensitiveSearch
| NSForcedOrderingSearch;

NSLocale *currentLocale = [NSLocale currentLocale];

NSArray *finderSortArray = [stringsArray sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
  NSRange string1Range = NSMakeRange(0, [obj1 length]);
  return [obj1 compare:obj2 options:comparisonOptions range:string1Range locale:currentLocale];
}];

NSLog(@"finderSortArray: %@", finderSortArray);

// 打印结果如下:
/*
2016-03-25 10:08:18.880 BlockDemos[8077:1168458] finderSortArray: (
    "string 1",
    "String 02",
    "String 11",
    "string 12",
    "String 21"
)
*/
```

# 五、\_\_block变量

block可以在block体内修改有有效作用域范围的外部变量的值，它是block非常强大的特性。对于需要修改的外部变量，需要使用\_\_block来标识。

```
__block int count = 0;
void (^TestBlock)(BOOL) = ^(BOOL isOk) {
  if (isOk) {
	 // 如果没有加__block来声明，编译不通过
    count++;
  }
};

TestBlock(YES);
NSLog(@"count = %d", count);
```

# 六、typedef block别名

因为把block全写出来不太方便，而且在开发中使用block作为函数的参数时，直接将整个block写在参数位置，可读性明显降低，那么有什么办法可以解决这个问题呢？

答案就是使用typedef给block起一个别名。

下面是我的项目中经常使用的几个block别名：

```
// 请求成功的回调
typedef void (^HYBResponseSuccessBlock)(id response);
// 请求失败的回调
typedef void (^HYBResponseFailureBlock)(NSError *error);
// 状态标识公用block
typedef void (^HYBStatusBlock)(BOOL status);
```

这么写有什么好处？语义清晰了，而且书写起来也简单多了。

# 七、block作为参数返回值

有时候，我们会直接返回一个block供外部调用，这时候就需要它了：

```
typedef void (^HYBStatusBlock)(BOOL);
- (HYBStatusBlock)testStatus {
  HYBStatusBlock block = ^(BOOL status) {
    NSLog(@"status is %d", status);
  };
  
  // 将block copy到堆上
  return [block copy];
}

HYBStatusBlock block = [self testStatus];
block(YES);

// status is 1
```

# 八、block作为成员变量

在开发中，我们见到很多block作为类的成员变量，用来代替代理的。比如，从控制器A进入到控制器B，在B操作完成后需要反馈给A，这时候使用block就很简化了！

在A控制器中调起B控制器：

```
B *b = [[B alloc] initWithCallback:^(BOOL isSuccess) {
   // B在完成时，反馈到A
   // 刷新状态
}];

[self.navigationController pushViewController:b animated:YES];
```

在B中声明了一个block属性：

```
// 为什么使用copy？这是拷贝到堆内存上
@property (nonatomic, copy) HYBStatusBlock callback;
```

然后在B操作完成时，回调A

```
if (self.callback) {
  self.callback(YES);// NO
}
```

# 源代码

本篇中大分部代码都可以到GITHUB下载：[BlockDemos](https://github.com/CoderJackyHuang/BlockDemos)

# 结尾

本篇文章只是抛砖引玉，初步学习block的基础知识。部分是参考官方文档，当然大部分是根据开发中常见的和常用的总结出来！

天天使用block，但是一直以来没有深入地研究过block的机制，后续会继续深入，一步步剖析block的本质！

# 参考

[官方文档：Getting Started With Blocks](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Blocks/Articles/bxGettingStarted.html#//apple_ref/doc/uid/TP40007502-CH7-SW1)


