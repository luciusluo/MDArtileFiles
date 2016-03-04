>#[原文出自：标哥的技术博客](http://www.henishuo.com/runtime-archive-unarchive-automaticly/)

#前言

善用runtime，可以解决自动归档解档。想想以前归档是手动写的，确实太麻烦了。现在有了runtime，我们可以做到自动化了。本篇文章旨在学习如何通过runtime实现自动归档和解档，因此不会对所有类型适用，而是对我们指定的几种类型适用。

#定义模型

我们这里只是写一个例子，用于学习如何用runtime实现自动归档以及解档，因此，我们需要定义一个模型类，然后在里面实现自动归档和解档。

我们这里只处理了普通的几种类型，这里只测试`int`、`NSString`、`const void *`、`NSNumber`、`float`类型。对于`const void *`是不支持`kvc`的，因此我们是不能通过`kvc`完成的，但是我们可以通过runtime发送消息实现。

##声明头文件
 
我们首先得要定义一个类，声明一下我们要测试的类型。首先，我们必须要遵守协议`NSCoding`，这是归档必须要遵守的：

```
@interface HDFArchiveModel : NSObject <NSCoding>

@property (nonatomic, assign) int    referenceCount;
@property (nonatomic, copy) NSString *archive;
@property (nonatomic, assign) const void *session;
@property (nonatomic, strong) NSNumber *totalCount;
// 注意，这里只是为了测试一下属性使用下划线的情况
@property (nonatomic, assign) float  _floatValue;

+ (void)test;

@end
```

##实现代码

遵守了`NSCoding`协议之后，我们就可以在实现文件中实现`-encodeWithCoder:`方法来归档和`-initWithCoder:`解档。

实现代码如下：

```
#import <objc/runtime.h>
#import <objc/message.h>

@implementation HDFArchiveModel

- (void)encodeWithCoder:(NSCoder *)aCoder {
  unsigned int outCount = 0;
  Ivar *ivars = class_copyIvarList([self class], &outCount);
  
  for (unsigned int i = 0; i < outCount; ++i) {
    Ivar ivar = ivars[i];
    
    // 获取成员变量名
    const void *name = ivar_getName(ivar);
    NSString *ivarName = [NSString stringWithUTF8String:name];
    // 去掉成员变量的下划线
    ivarName = [ivarName substringFromIndex:1];
    
    // 获取getter方法
    SEL getter = NSSelectorFromString(ivarName);
    if ([self respondsToSelector:getter]) {
      const void *typeEncoding = ivar_getTypeEncoding(ivar);
      NSString *type = [NSString stringWithUTF8String:typeEncoding];
      
      // const void *
      if ([type isEqualToString:@"r^v"]) {
        const char *value = ((const void *(*)(id, SEL))(void *)objc_msgSend)((id)self, getter);
        NSString *utf8Value = [NSString stringWithUTF8String:value];
        [aCoder encodeObject:utf8Value forKey:ivarName];
        continue;
      }
      // int
      else if ([type isEqualToString:@"i"]) {
        int value = ((int (*)(id, SEL))(void *)objc_msgSend)((id)self, getter);
        [aCoder encodeObject:@(value) forKey:ivarName];
        continue;
      }
      // float
      else if ([type isEqualToString:@"f"]) {
        float value = ((float (*)(id, SEL))(void *)objc_msgSend)((id)self, getter);
        [aCoder encodeObject:@(value) forKey:ivarName];
        continue;
      }
      
     id value = ((id (*)(id, SEL))(void *)objc_msgSend)((id)self, getter);
      if (value != nil && [value respondsToSelector:@selector(encodeWithCoder:)]) {
        [aCoder encodeObject:value forKey:ivarName];
      }
    }
  }
  
  free(ivars);
}

- (instancetype)initWithCoder:(NSCoder *)aDecoder {
  if (self = [super init]) {
    unsigned int outCount = 0;
    Ivar *ivars = class_copyIvarList([self class], &outCount);
    
    for (unsigned int i = 0; i < outCount; ++i) {
      Ivar ivar = ivars[i];
      
      // 获取成员变量名
      const void *name = ivar_getName(ivar);
      NSString *ivarName = [NSString stringWithUTF8String:name];
      // 去掉成员变量的下划线
      ivarName = [ivarName substringFromIndex:1];
      // 生成setter格式
      NSString *setterName = ivarName;
      // 那么一定是字母开头
      if (![setterName hasPrefix:@"_"]) {
        NSString *firstLetter = [NSString stringWithFormat:@"%c", [setterName characterAtIndex:0]];
        setterName = [setterName substringFromIndex:1];
        setterName = [NSString stringWithFormat:@"%@%@", firstLetter.uppercaseString, setterName];
      }
     setterName = [NSString stringWithFormat:@"set%@:", setterName];
      // 获取getter方法
      SEL setter = NSSelectorFromString(setterName);
      if ([self respondsToSelector:setter]) {
        const void *typeEncoding = ivar_getTypeEncoding(ivar);
        NSString *type = [NSString stringWithUTF8String:typeEncoding];
        NSLog(@"%@", type);
        
        // const void *
        if ([type isEqualToString:@"r^v"]) {
          NSString *value = [aDecoder decodeObjectForKey:ivarName];
          if (value) {
           ((void (*)(id, SEL, const void *))objc_msgSend)(self, setter, value.UTF8String);
          }
  
          continue;
        }
        // int
        else if ([type isEqualToString:@"i"]) {
          NSNumber *value = [aDecoder decodeObjectForKey:ivarName];
          if (value != nil) {
            ((void (*)(id, SEL, int))objc_msgSend)(self, setter, [value intValue]);
          }
          continue;
        } else if ([type isEqualToString:@"f"]) {
          NSNumber *value = [aDecoder decodeObjectForKey:ivarName];
          if (value != nil) {
            ((void (*)(id, SEL, float))objc_msgSend)(self, setter, [value floatValue]);
          }
          continue;
        }

        // object
        id value = [aDecoder decodeObjectForKey:ivarName];
        if (value != nil) {
          ((void (*)(id, SEL, id))objc_msgSend)(self, setter, value);
        }
      }
    }
    
    free(ivars);
  }
  
  return self;
}

+ (void)test {
  HDFArchiveModel *archiveModel = [[HDFArchiveModel alloc] init];
  archiveModel.archive = @"标哥学习自动归档";
  archiveModel.session = "http://www.henishuo.com";
  archiveModel.totalCount = @(123);
  archiveModel.referenceCount = 10;
  archiveModel._floatValue = 10.0;
  
  NSString *path = NSHomeDirectory();
  path = [NSString stringWithFormat:@"%@/archive", path];
  [NSKeyedArchiver archiveRootObject:archiveModel
                              toFile:path];
  
  HDFArchiveModel *unarchiveModel = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
  
}

@end
```

#自动归档解析

我们这里获取对象的成员变量，通过`class_copyIvarList`可以得到所有成员变量：

```
unsigned int outCount = 0;
Ivar *ivars = class_copyIvarList([self class], &outCount);
```

我们获取到的属性所生成的成员变量是带下划线，因此我们需要在获取到成员变量名称后，需要去掉下划线：

```
// 获取成员变量名
const void *name = ivar_getName(ivar);
NSString *ivarName = [NSString stringWithUTF8String:name];
// 去掉成员变量的下划线
ivarName = [ivarName substringFromIndex:1];
```

接下来，在归档的时候我们通过属性的getter方法来获取值，然后归档。但是，成员变量是是有类型的，并不是所有类型都可以归档，比如`const void *`就不支持归档，那么我们需要根据类型转换成支持归档的类型再存储。另外，我们可以通过`ivar_getTypeEncoding`函数获取成员变量的类型，但是这个类型不一定是我们常见的`NSString`之类的，可能会出现`r^v`这样代表`const void *`。这些都是runtime系统所定义的，因此它是固定的，我们只要去按照苹果给出的类型编码表就可以知道哪些字符代表什么类型了。

```
const void *typeEncoding = ivar_getTypeEncoding(ivar);
NSString *type = [NSString stringWithUTF8String:typeEncoding];

// const void *
if ([type isEqualToString:@"r^v"]) {
  const char *value = ((const void *(*)(id, SEL))(void *)objc_msgSend)((id)self, getter);
  NSString *utf8Value = [NSString stringWithUTF8String:value];
  [aCoder encodeObject:utf8Value forKey:ivarName];
  continue;
}
// int
else if ([type isEqualToString:@"i"]) {
  int value = ((int (*)(id, SEL))(void *)objc_msgSend)((id)self, getter);
  [aCoder encodeObject:@(value) forKey:ivarName];
  continue;
}
// float
else if ([type isEqualToString:@"f"]) {
  float value = ((float (*)(id, SEL))(void *)objc_msgSend)((id)self, getter);
  [aCoder encodeObject:@(value) forKey:ivarName];
  continue;
}

id value = ((id (*)(id, SEL))(void *)objc_msgSend)((id)self, getter);
if (value != nil && [value respondsToSelector:@selector(encodeWithCoder:)]) {
  [aCoder encodeObject:value forKey:ivarName];
}
```

像上面，我们只额外处理了`const void *`、`int`、`float`类型，当我们调用`objc_msgSend`函数时，一定要转换类型，如果类型不匹配无法转换则是会崩溃的。从这里看到`objc_msgSend`函数是不是很强大？是的，真的太强大了。一会可以是有返回值的，一会可以是带多个参数的，还可以是不带返回值的。

最后，我们就可以调用归档方法来实现了归档：

```
[NSKeyedArchiver archiveRootObject:archiveModel
                          toFile:path];
```

到此，我们归档的工作已经完成了。如果要写一个通用的自动归档扩展，那么我们就需要处理完苹果中所有的数据类型，包括基本类型、C指针类型等。

#自动解档解析 

不知道这里叫解档是否合适，但是似乎已经习惯这么叫了。要实现解档，其实就是实现`NSCoding`协议中的`-initWithCoder:`方法。

首先我们要生成`setter`方法，但是`setter`方法的生成是有规则的。若属性名称不带下划线，那么生成的`setter`方法就是`set`+属性名称，其中需要将属性名称的首字母变成大写。若属性名称带下划线，则不需要处理。我们看看如何生成：

```
// 那么一定是字母开头
if (![setterName hasPrefix:@"_"]) {
	NSString *firstLetter = [NSString stringWithFormat:@"%c", [setterName characterAtIndex:0]];
	setterName = [setterName substringFromIndex:1];
	setterName = [NSString stringWithFormat:@"%@%@", firstLetter.uppercaseString, setterName];
}
setterName = [NSString stringWithFormat:@"set%@:", setterName];
```

我们知道，苹果中的变量只是是字母、数字和下划线，其中第一个字符只能是字母或者是下划线。因此，上面判断第一个是否是下划线，若不是下划线，则说明一定是字母。然后我们需要获取首字母，以便转换成大写字母。当然，上面替换，我们也可以这样写：

```
[setterName stringByReplacingCharactersInRange:NSMakeRange(0, 0) withString:firstLetter.uppercaseString];
```

接下来就是判断属性的类型，然后发送消息。这里只说明设置`const void *`属性值：

```
if ([type isEqualToString:@"r^v"]) {
  NSString *value = [aDecoder decodeObjectForKey:ivarName];
  if (value) {
   ((void (*)(id, SEL, const void *))objc_msgSend)(self, setter, value.UTF8String);
  }
  
  continue;
}
```

我们一定要强转`objc_msgSend`函数为`(void (*)(id, SEL, const void *))`这样的类型，然后其中第三个参数就是我们要设置的属性的值的类型。

最后，我们就可以通过解档方法来实现解档了：

```
HDFArchiveModel *unarchiveModel = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
```

#源代码

小伙伴们可以到我的GITHUB下载demo看看效果：[https://github.com/CoderJackyHuang/RuntimeDemo](https://github.com/CoderJackyHuang/RuntimeDemo)

**喜欢，就给个star吧！**

#写在最后

在学习通过runtime自动归档和解档的过程中，也遇到了不少问题。尤其是在生成setter方法时，由于调用了`capitalizedString`方法，结果将属性名称中只有首字母大写，其它全变成了小写，导致笔者调试了好久才发现这个小细节。这再次说明了，细节决定成败！！！

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

#支持并捐助

如果您觉得文章对您很有帮助，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)


