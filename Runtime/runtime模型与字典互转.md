>#[原文出自：标哥的技术博客](http://www.henishuo.com/runtime-model-dictionary-convert/)

#前言

在开发中必不可少的模型与字典互转，但是一直以来都是使用他人的库，从来没有研究其原理或者说深究其所以然。现在，在这里我们一起来学习通过`runtime`完成模型与字典的互转。

#声明Model

在开始介绍详细API之前，我们先来声明一个模型类`HYBTestModel`，这个类提供了根据字典转换成模型类对象的功能，还提供了将模型类转换成字典的功能：

```
//
//  HYBTestModel.h
//  RuntimeDemo
//
//  Created by huangyibiao on 15/12/28.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import <Foundation/Foundation.h>

@protocol HYBEmptyPropertyProperty <NSObject>

// 设置默认值，若为空，则取出来的就是默认值
- (NSDictionary *)defaultValueForEmptyProperty;

@end

@interface HYBTestModel : NSObject <HYBEmptyPropertyProperty>

@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *title;
@property (nonatomic, strong) NSNumber *count;
@property (nonatomic, assign) int    commentCount;
@property (nonatomic, strong) NSArray *summaries;
@property (nonatomic, strong) NSDictionary *parameters;
@property (nonatomic, strong) NSSet *results;

@property (nonatomic, strong) HYBTestModel *testModel;

// 只读属性
@property (nonatomic, assign, readonly) NSString *classVersion;

// 通过这个方法来实现自动生成model
- (instancetype)initWithDictionary:(NSDictionary *)dictionary;

// 转换成字典
- (NSDictionary *)toDictionary;

// 测试
+ (void)test;

@end
```

我们这里声明了几种类型，主要是模型中的数组、字典、集合、对象属性，我们最后要转换对象成字典。

#实现代码

在我们讲解之前，先把代码全部放出来，我们再一一讲解：

```
#import "HYBTestModel.h"
#import <objc/runtime.h>
#import <objc/message.h>

@implementation HYBTestModel

- (instancetype)initWithDictionary:(NSDictionary *)dictionary {
  if (self = [super init]) {
    for (NSString *key in dictionary.allKeys) {
      id value = [dictionary objectForKey:key];
      
      if ([key isEqualToString:@"testModel"]) {
        HYBTestModel *testModel = [[HYBTestModel alloc] initWithDictionary:value];
        value = testModel;
        self.testModel = testModel;
        
        continue;
      }
      
      SEL setter = [self propertySetterWithKey:key];
      if (setter != nil) {
        ((void (*)(id, SEL, id))objc_msgSend)(self, setter, value);
      }
    }
  }
  
  return self;
}

- (NSDictionary *)toDictionary {
  unsigned int outCount = 0;
  objc_property_t *properties = class_copyPropertyList([self class], &outCount);
  
  if (outCount != 0) {
    NSMutableDictionary *dict = [[NSMutableDictionary alloc] initWithCapacity:outCount];
    
    for (unsigned int i = 0; i < outCount; ++i) {
      objc_property_t property = properties[i];
      const void *propertyName = property_getName(property);
      NSString *key = [NSString stringWithUTF8String:propertyName];
      
      // 继承于NSObject的类都会有这几个在NSObject中的属性
      if ([key isEqualToString:@"description"]
          || [key isEqualToString:@"debugDescription"]
          || [key isEqualToString:@"hash"]
          || [key isEqualToString:@"superclass"]) {
        continue;
      }
      
      // 我们只是测试，不做通用封装，因此这里不额外写方法做通用处理，只是写死测试一下效果
      if ([key isEqualToString:@"testModel"]) {
        if ([self respondsToSelector:@selector(toDictionary)]) {
          id testModel = [self.testModel toDictionary];
          if (testModel != nil) {
            [dict setObject:testModel forKey:key];
          }
          continue;
        }
      }
      
      SEL getter = [self propertyGetterWithKey:key];
      if (getter != nil) {
        // 获取方法的签名
        NSMethodSignature *signature = [self methodSignatureForSelector:getter];
        
        // 根据方法签名获取NSInvocation对象
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
        // 设置target
        [invocation setTarget:self];
        // 设置selector
        [invocation setSelector:getter];
        
        // 方法调用
        [invocation invoke];
        
        // 接收返回的值
        __unsafe_unretained NSObject *propertyValue = nil;
        [invocation getReturnValue:&propertyValue];
        
        //        id propertyValue = [self performSelector:getter];
        
        if (propertyValue == nil) {
          if ([self respondsToSelector:@selector(defaultValueForEmptyProperty)]) {
            NSDictionary *defaultValueDict = [self defaultValueForEmptyProperty];
            
            id defaultValue = [defaultValueDict objectForKey:key];
            propertyValue = defaultValue;
          }
        }
        
        if (propertyValue != nil) {
          [dict setObject:propertyValue forKey:key];
        }
      }
    }
    
    free(properties);
    
    return dict;
  }
  
  free(properties);
  return nil;
}

- (SEL)propertyGetterWithKey:(NSString *)key {
  if (key != nil) {
    SEL getter = NSSelectorFromString(key);
    
    if ([self respondsToSelector:getter]) {
      return getter;
    }
  }
  
  return nil;
}

- (SEL)propertySetterWithKey:(NSString *)key {
  NSString *propertySetter = key.capitalizedString;
  propertySetter = [NSString stringWithFormat:@"set%@:", propertySetter];
  
  // 生成setter方法
  SEL setter = NSSelectorFromString(propertySetter);
  
  if ([self respondsToSelector:setter]) {
    return setter;
  }
  
  return nil;
}

#pragma mark - HYBEmptyPropertyProperty
- (NSDictionary *)defaultValueForEmptyProperty {
  return @{@"name" : [NSNull null],
           @"title" : [NSNull null],
           @"count" : @(1),
           @"commentCount" : @(1),
           @"classVersion" : @"0.0.1"};
}

+ (void)test {
  NSMutableSet *set = [NSMutableSet setWithArray:@[@"可变集合", @"字典->不可变集合->可变集合"]];
  NSDictionary *dict = @{@"name"  : @"标哥的技术博客",
                         @"title" : @"http://www.henishuo.com",
                         @"count" : @(11),
                         @"results" : [NSSet setWithObjects:@"集合值1", @"集合值2", set , nil],
                         @"summaries" : @[@"sm1", @"sm2", @{@"keysm": @{@"stkey": @"字典->数组->字典->字典"}}],
                         @"parameters" : @{@"key1" : @"value1", @"key2": @{@"key11" : @"value11", @"key12" : @[@"三层", @"字典->字典->数组"]}},
                         @"classVersion" : @(1.1),
                         @"testModel" : @{@"name"  : @"标哥的技术博客",
                                          @"title" : @"http://www.henishuo.com",
                                          @"count" : @(11),
                                          @"results" : [NSSet setWithObjects:@"集合值1", @"集合值2", set , nil],
                                          @"summaries" : @[@"sm1", @"sm2", @{@"keysm": @{@"stkey": @"字典->数组->字典->字典"}}],
                                          @"parameters" : @{@"key1" : @"value1", @"key2": @{@"key11" : @"value11", @"key12" : @[@"三层", @"字典->字典->数组"]}},
                                          @"classVersion" : @(1.1)}};
  HYBTestModel *model = [[HYBTestModel alloc] initWithDictionary:dict];
  
  NSLog(@"%@", model);
  
  NSLog(@"model->dict: %@", [model toDictionary]);
}

@end
```

#细节讲解

注意到这一行代码了吗？这是将对应的值赋值给对应的属性：

```
((void (*)(id, SEL, id))objc_msgSend)(self, setter, value)
```

我们需要明确一点，`objc_msgSend`函数是用于发送消息的，而这个函数是可以很多个参数的，但是我们必须手动转换成对应类型参数，比如上面我们就是强制转换`objc_msgSend`函数类型为带三个参数且返回值为`void`函数，然后才能传三个参数。如果我们直接通过`objc_msgSend(self, setter, value)`是报错，说参数过多。

`(void (*)(id, SEL, id)`这是`C`语言中的函数指针，如果不了解`C`，没有关系，我们只需要记住参数列表前面是一个`(*)`这样的就是对应函数指针了。

##生成Setter方法

我们要通过`objc_msgSend`函数来发送消息给对象，然后通过属性的`setter`方法来赋值，那么我们要生成`setter`选择器：

```
- (SEL)propertySetterWithKey:(NSString *)key {
  NSString *propertySetter = key.capitalizedString;
  propertySetter = [NSString stringWithFormat:@"set%@:", propertySetter];
  
  // 生成setter方法
  SEL setter = NSSelectorFromString(propertySetter);
  
  if ([self respondsToSelector:setter]) {
    return setter;
  }
  
  return nil;
}
```

这里就是生成属性的`setter`选择器。我们知道，系统生成属性的`setter`方法的规范是`setKey:`这样的格式，因此我们只要按照同样的规则生成`setter`就可以了。另外我们还需要判断是否可以响应此`setter`方法。

##模型中有模型属性

对于本demo中，模型中还有模型属性，那么我们应该如何来赋值呢？其实，我们需要注意一点，一定要给模型属性分配内存，否则看起来赋值了，但是对象还是空。

```
if ([key isEqualToString:@"testModel"]) {
	HYBTestModel *testModel = [[HYBTestModel alloc] initWithDictionary:value];
	value = testModel;
	self.testModel = testModel;
	    
	continue;
}
```

这里是在`for`循环中的，我们一定要注意加上`continue`，否则下边可能会将其值设置为空哦。

##生成Getter方法

我们可以通过`NSSelectorFromString`函数来生成`SEL`选择器，当然我们也可以通过`@selector()`生成`SEL`选择器，但是我们这里只能使用前者：

```
- (SEL)propertyGetterWithKey:(NSString *)key {
  if (key != nil) {
    SEL getter = NSSelectorFromString(key);
    
    if ([self respondsToSelector:getter]) {
      return getter;
    }
  }
  
  return nil;
}
```

##模型转字典

首先，我们需要先获取所有的属性，以便获取属性值：

```
unsigned int outCount = 0;
objc_property_t *properties = class_copyPropertyList([self class], &outCount);
```

`objc_property_t`类型代表属性，它是一个结构体。通过函数`class_copyPropertyList`可以获取对象的属性列表及属性的个数。

在遍历属性列表时，我们通过这样获取名称：

```
objc_property_t property = properties[i];
const void *propertyName = property_getName(property);
NSString *key = [NSString stringWithUTF8String:propertyName];
```

由于继承于`NSObject`的对象，都有这几个属性，因此我们需要过滤掉：

```
// 继承于NSObject的类都会有这几个在NSObject中的属性
if ([key isEqualToString:@"description"]
  || [key isEqualToString:@"debugDescription"]
  || [key isEqualToString:@"hash"]
  || [key isEqualToString:@"superclass"]) {
	continue;
}
```

###调用getter方法获取值

我们要通过runtime获取值，常用的有两种方式：

* 通过performSelector方法
* 通过NSInvocation对象

下面是通过`NSInvocation`的方法，流程可以这样：先获取方法签名，然后根据方法签名生成`NSInvocation`对象，设置`target`、`SEL`，然后调用，最后获取返回值：

```
SEL getter = [self propertyGetterWithKey:key];
if (getter != nil) {
  // 获取方法的签名
  NSMethodSignature *signature = [self methodSignatureForSelector:getter];
  
  // 根据方法签名获取NSInvocation对象
  NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
  // 设置target
  [invocation setTarget:self];
  // 设置selector
  [invocation setSelector:getter];
  
  // 方法调用
  [invocation invoke];
  
  // 接收返回的值
  __unsafe_unretained NSObject *propertyValue = nil;
  [invocation getReturnValue:&propertyValue];
  
  //        id propertyValue = [self performSelector:getter];
}
```

如果是通过`performSelector`方式，一行代码就可以了，但是会提示有内存泄露的风险，通常使用上面这种方式，而不是直接使用下面这种方式：

```
id propertyValue = [self performSelector:getter];
```

我们这里还做了额外的处理，当属性值获取到是空的时候，我们可以通过协议指定默认值。当值为空时，我们就会使用默认值：

```
if (propertyValue == nil) {
  if ([self respondsToSelector:@selector(defaultValueForEmptyProperty)]) {
    NSDictionary *defaultValueDict = [self defaultValueForEmptyProperty];
    
    id defaultValue = [defaultValueDict objectForKey:key];
    propertyValue = defaultValue;
  }
}
```

###模型属性转换成字典

我们这里的模型有一个属性也是一个模型，因此我们需要额外处理一下：

```
// 我们只是测试，不做通用封装，因此这里不额外写方法做通用处理，只是写死测试一下效果
if ([key isEqualToString:@"testModel"]) {
  if ([self respondsToSelector:@selector(toDictionary)]) {
    id testModel = [self.testModel toDictionary];
    if (testModel != nil) {
      [dict setObject:testModel forKey:key];
    }
    continue;
  }
}
```

注意，这里是写死的哦，因为我们这里写死了`testModel`，只是为了简化，如果要封装成通用的方法，那么就需要做更多的工作了。不过我们这里的目的是学习如何转，而不是封装成能用的库。因此，研究明白其原理才是目的。

#测试


我们测试一下这样复杂的结构：

```
+ (void)test {
  NSMutableSet *set = [NSMutableSet setWithArray:@[@"可变集合", @"字典->不可变集合->可变集合"]];
  NSDictionary *dict = @{@"name"  : @"标哥的技术博客",
                         @"title" : @"http://www.henishuo.com",
                         @"count" : @(11),
                         @"results" : [NSSet setWithObjects:@"集合值1", @"集合值2", set , nil],
                         @"summaries" : @[@"sm1", @"sm2", @{@"keysm": @{@"stkey": @"字典->数组->字典->字典"}}],
                         @"parameters" : @{@"key1" : @"value1", @"key2": @{@"key11" : @"value11", @"key12" : @[@"三层", @"字典->字典->数组"]}},
                         @"classVersion" : @(1.1),
                         @"testModel" : @{@"name"  : @"标哥的技术博客",
                                          @"title" : @"http://www.henishuo.com",
                                          @"count" : @(11),
                                          @"results" : [NSSet setWithObjects:@"集合值1", @"集合值2", set , nil],
                                          @"summaries" : @[@"sm1", @"sm2", @{@"keysm": @{@"stkey": @"字典->数组->字典->字典"}}],
                                          @"parameters" : @{@"key1" : @"value1", @"key2": @{@"key11" : @"value11", @"key12" : @[@"三层", @"字典->字典->数组"]}},
                                          @"classVersion" : @(1.1)}};
  HYBTestModel *model = [[HYBTestModel alloc] initWithDictionary:dict];
  
  NSLog(@"%@", model);
  
  NSLog(@"model->dict: %@", [model toDictionary]);
}
```

#打印效果

模型转字典的效果：

```
2015-12-29 16:02:15.207 RuntimeDemo[40233:1083396] model->dict: 	{
	classVersion = "0.0.1",
	count = 11,
	parameters = 	{
		key1 = "value1",
		key2 = 	{
			key11 = "value11",
			key12 = 	(
				"三层",
				"字典->字典->数组",
			),
		},
	},
	results = 	{(
		"集合值2",
		"集合值1",
			{(
			"可变集合",
			"字典->不可变集合->可变集合",
		)},
	)},
	title = "http://www.henishuo.com",
	commentCount = 1,
	testModel = 	{
		classVersion = "0.0.1",
		count = 11,
		parameters = 	{
			key1 = "value1",
			key2 = 	{
				key11 = "value11",
				key12 = 	(
					"三层",
					"字典->字典->数组",
				),
			},
		},
		results = 	{(
			"集合值2",
			"集合值1",
				{(
				"可变集合",
				"字典->不可变集合->可变集合",
			)},
		)},
		title = "http://www.henishuo.com",
		commentCount = 1,
		name = "标哥的技术博客",
		summaries = 	(
			"sm1",
			"sm2",
				{
				keysm = 	{
					stkey = "字典->数组->字典->字典",
				},
			},
		),
	},
	name = "标哥的技术博客",
	summaries = 	(
		"sm1",
		"sm2",
			{
			keysm = 	{
				stkey = "字典->数组->字典->字典",
			},
		},
	),
}
```

转换是成功的，目的达到了！！！

>注意，这里打印显示这样的格式，中文也能正常显示出来，是笔者提供了这样一个库，详细请阅读[http://www.henishuo.com/ios-unicode-readable/](http://www.henishuo.com/ios-unicode-readable/)

#源代码下载

小伙伴们可以到GITHUB下载源代码：[https://github.com/CoderJackyHuang/RuntimeDemo](https://github.com/CoderJackyHuang/RuntimeDemo)

**给个star吧！！！**

#写在最后

本篇就介绍这么多，模型与字典互转的功能就可以自己尝试去实现了，对runtime的了解也更深入一步了！！！

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

#支持并捐助

如果您觉得文章对您很有帮助，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
