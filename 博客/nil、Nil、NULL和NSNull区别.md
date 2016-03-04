#前言

---
记得曾经有不少朋友问过笔者，在`Objective-C`中`nil`和`Nil`以及`NULL`的区别。最重要的是，在面试中还有不少朋友常会被问到。记得当年刚找工作的时候，笔者就被面试官问到过，现在笔者在这里统一详细说明。

#NULL

---
对于学习过`C/C++`语言的朋友，对`NULL`一定很熟悉吧？这就是在`C/C++`中的空指针。

在`C`语言中，`NULL`是无类型的，只是一个宏，它代表空。我们不研究`C++`中的`NULL`，因为在`C++11`以后又有了新的定义，我们不深究。

这就是`C`语言中所谓的`NULL`（`C++`的定义比较复杂，这里不说了）：

```
#if defined(__need_NULL)
#undef NULL
#ifdef __cplusplus
#  if !defined(__MINGW32__) && !defined(_MSC_VER)
#    define NULL __null
#  else
#    define NULL 0
#  endif
#else
#  define NULL ((void*)0)
#endif
```
这是在`stddef.h`头文件中声明的。这是使用了条件编译的，`__cplusplus`这个宏表示`C++`，对于我们`Objective-C`开发来说，`NULL`就表示`((void*)0)`

像`C`语言中，我们定义了一个指针，当我们使用完以后，通常会设置指向`NULL`。如果没有设置，这个指针就成了所谓的野指针，然后其它地方不小心访问了这个指针是很容易造成非法访问的，常见的表现就是崩溃了。

既然`Objective-C`是基于`C`语言的面向对象语言，那么也会使用到`C`语言类型的指针，比如使用`const char *`类型，判断是否为空时，是使用`p != NULL`来判断的。

#nil

---
对于我们学习`Objective-C`的人来说，这个是非常熟悉的。如下为官方定义：

```
#ifndef nil
# if __has_feature(cxx_nullptr)
#   define nil nullptr
# else
#   define nil __DARWIN_NULL
# endif
#endif
```

对于我们`Objective-C`开发来说，`nil`就是`__DARWIN_NULL`。看下官方定义：

```
#ifdef __cplusplus
#ifdef __GNUG__
#define __DARWIN_NULL __null
#else /* ! __GNUG__ */
#ifdef __LP64__
#define __DARWIN_NULL (0L)
#else /* !__LP64__ */
#define __DARWIN_NULL 0
#endif /* __LP64__ */
#endif /* __GNUG__ */
#else /* ! __cplusplus */
#define __DARWIN_NULL ((void *)0)
#endif /* __cplusplus */
```

这个也是条件编译的，那么对于我们`Objective-C`开发来说，`nil`就代表`((void *)0)`。

我们使用`nil`表示`Objective-C`对象为空，如`NSString *str = nil`。

#Nil

---
先看看官方是如何声明的：

```
#ifndef Nil
# if __has_feature(cxx_nullptr)
#   define Nil nullptr
# else
#   define Nil __DARWIN_NULL
# endif
#endif
```

根据条件，我们做`Objective-C`开发的，那么`Nil`也就是代表`__DARWIN_NULL`，而对于`__DARWIN_NULL`的声明如下：

```
#ifdef __cplusplus
#ifdef __GNUG__
#define __DARWIN_NULL __null
#else /* ! __GNUG__ */
#ifdef __LP64__
#define __DARWIN_NULL (0L)
#else /* !__LP64__ */
#define __DARWIN_NULL 0
#endif /* __LP64__ */
#endif /* __GNUG__ */
#else /* ! __cplusplus */
#define __DARWIN_NULL ((void *)0)
#endif /* __cplusplus */
```

这个也是条件编译的，那么对于我们`Objective-C`开发来说，`Nil`也就代表`((void *)0)`。

但是它是用于代表空类的。比如：

```
Class myClass = Nil;
```

#NSNull

---
先看看官方的声明：

```
NS_ASSUME_NONNULL_BEGIN

@interface NSNull : NSObject <NSCopying, NSSecureCoding>

+ (NSNull *)null;

@end

NS_ASSUME_NONNULL_END
```

由此我们可知，`NSNull`是继承于`NSObject`的类型。它是很特殊的类，它表示是空，什么也不存储，但是它却是对象，只是一个占位对象。

使用场景就不一样了，比如说服务端接口中让我们在值为空时，传空。

```
NSDictionry *parameters = @{@"arg1" : @"value1",
                            @"arg2" : arg2.isEmpty ? [NSNull null] : arg2};
```

这只是随手举的例子，当然我们也可以不传这人参数。如果我们要统一，比如通过`runtime`来动态将对象转成我们的参数时，那么可以统一将值为`nil`的都设置为`[NSNull null]`              

#区别

---
`NULL`、`nil`、`Nil`这三者对于`Objective-C`中值是一样的，都是`(void *)0`，那么为什么要区分呢？又与`NSNull`之间有什么区别：

* `NULL`是宏，是对于`C`语言指针而使用的，表示空指针
* `nil`是宏，是对于`Objective-C`中的对象而使用的，表示对象为空
* `Nil`是宏，是对于`Objective-C`中的类而使用的，表示类指向空
* `NSNull`是类类型，是用于表示空的占位对象，与`JS`或者服务端的`null`类似的含意

#写在最后

---
以上只是笔者个人见解，不代表百分百正确，如果疑问之处，请在评论处留言，笔者会回复！！！

以上只是笔者个人见解，不代表百分百正确，如果疑问之处，请在评论处留言，笔者会回复！！！

以上只是笔者个人见解，不代表百分百正确，如果疑问之处，请在评论处留言，笔者会回复！！！

#[阅读原文](http://www.henishuo.com/nil-nil-null-nsnull-difference/)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/)**

