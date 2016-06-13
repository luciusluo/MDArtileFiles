
#前言

开发中经常需要打印日志以查看数据是否正确，或者说查看数据的格式。但是，苹果对于我们的`NSDictionary`、`NSSet`、`NSArray`等值有中文时，打印出来的是`Unicode`编码，人类无法直接读懂，因此，笔者研究研究如何将打印出来的日志保持原有的格式化且能够将`Unicode`编码打印出来是正常人类可读懂的中文。

#当前版本

* 已升级至最新版：1.2

#实现原理

苹果给我们提供了本地化的方法，对于`NSDictionary`、`NSSet`、`NSArray`都可以重写该方法来实现：

##NSSet实现

对于`NSSet`实现如下：

```
- (NSString *)descriptionWithLocale:(id)locale indent:(NSUInteger)level {
  NSMutableString *desc = [NSMutableString string];
  
  NSMutableString *tabString = [[NSMutableString alloc] initWithCapacity:level];
  for (NSUInteger i = 0; i < level; ++i) {
    [tabString appendString:@"\t"];
  }
  
  NSString *tab = @"\t";
  if (level > 0) {
    tab = tabString;
  }
  [desc appendString:@"\t{(\n"];
  
  for (id obj in self) {
    if ([obj isKindOfClass:[NSDictionary class]]
        || [obj isKindOfClass:[NSArray class]]
        || [obj isKindOfClass:[NSSet class]]) {
      NSString *str = [((NSDictionary *)obj) descriptionWithLocale:locale indent:level + 1];
      [desc appendFormat:@"%@\t%@,\n", tab, str];
    } else if ([obj isKindOfClass:[NSString class]]) {
      [desc appendFormat:@"%@\t\"%@\",\n", tab, obj];
    } else {
      [desc appendFormat:@"%@\t%@,\n", tab, obj];
    }
  }
  
  [desc appendFormat:@"%@)}", tab];
  
  return desc;
}
```
我们这里根本缩进级别来生成对应的格式化字符串，以便解决多级嵌套时格式不太美观的问题。

##NSArray实现

对于`NSArray`的实现如下：

```
- (NSString *)descriptionWithLocale:(id)locale indent:(NSUInteger)level {
  NSMutableString *desc = [NSMutableString string];
  
  NSMutableString *tabString = [[NSMutableString alloc] initWithCapacity:level];
  for (NSUInteger i = 0; i < level; ++i) {
    [tabString appendString:@"\t"];
  }
  
  NSString *tab = @"";
  if (level > 0) {
    tab = tabString;
  }
  [desc appendString:@"\t(\n"];
  
  for (id obj in self) {
    if ([obj isKindOfClass:[NSDictionary class]]
        || [obj isKindOfClass:[NSArray class]]
        || [obj isKindOfClass:[NSSet class]]) {
      NSString *str = [((NSDictionary *)obj) descriptionWithLocale:locale indent:level + 1];
      [desc appendFormat:@"%@\t%@,\n", tab, str];
    } else if ([obj isKindOfClass:[NSString class]]) {
      [desc appendFormat:@"%@\t\"%@\",\n", tab, obj];
    } else {
      [desc appendFormat:@"%@\t%@,\n", tab, obj];
    }
  }
  
  [desc appendFormat:@"%@)", tab];
  
  return desc;
}
```
同样的道理，不过笔者对于字符串值，都加上了双引号，一眼就可以看出来是字符串类型。

##NSDictionary实现

对于NSDictonary的实现如下：

```
- (NSString *)descriptionWithLocale:(id)locale indent:(NSUInteger)level {
  NSMutableString *desc = [NSMutableString string];
  
  NSMutableString *tabString = [[NSMutableString alloc] initWithCapacity:level];
  for (NSUInteger i = 0; i < level; ++i) {
    [tabString appendString:@"\t"];
  }
  
  NSString *tab = @"";
  if (level > 0) {
    tab = tabString;
  }
  
  [desc appendString:@"\t{\n"];
  
  // 遍历数组,self就是当前的数组
  for (id key in self.allKeys) {
    id obj = [self objectForKey:key];
    
    if ([obj isKindOfClass:[NSString class]]) {
      [desc appendFormat:@"%@\t%@ = \"%@\",\n", tab, key, obj];
    } else if ([obj isKindOfClass:[NSArray class]]
               || [obj isKindOfClass:[NSDictionary class]]
               || [obj isKindOfClass:[NSSet class]]) {
      [desc appendFormat:@"%@\t%@ = %@,\n", tab, key, [obj descriptionWithLocale:locale indent:level + 1]];
    } else {
      [desc appendFormat:@"%@\t%@ = %@,\n", tab, key, obj];
    }
  }
  
  [desc appendFormat:@"%@}", tab];
  
  return desc;
}
```

字典打印出来的格式是{}这样的结构，我们一样要考虑嵌套情况，并且根据嵌套添加对应的级别缩进。

#升级版本到Version1.0

新增NSData类型自动打印出可视化内容，只要是新增类似这样的处理：

```
else if ([obj isKindOfClass:[NSData class]]) {
  // 如果是NSData类型，尝试去解析结果，以打印出可阅读的数据
  NSError *error = nil;
  NSObject *result =  [NSJSONSerialization JSONObjectWithData:obj
                                                      options:NSJSONReadingMutableContainers
                                                        error:&error];
  // 解析成功
  if (error == nil && result != nil) {
    if ([result isKindOfClass:[NSDictionary class]]
        || [result isKindOfClass:[NSArray class]]
        || [result isKindOfClass:[NSSet class]]) {
      NSString *str = [((NSDictionary *)result) descriptionWithLocale:locale indent:level + 1];
      [desc appendFormat:@"%@\t%@ = %@,\n", tab, key, str];
    } else if ([obj isKindOfClass:[NSString class]]) {
      [desc appendFormat:@"%@\t%@ = \"%@\",\n", tab, key, result];
    }
  } else {
    @try {
      NSString *str = [[NSString alloc] initWithData:obj encoding:NSUTF8StringEncoding];
      if (str != nil) {
        [desc appendFormat:@"%@\t%@ = \"%@\",\n", tab, key, str];
      } else {
        [desc appendFormat:@"%@\t%@ = %@,\n", tab, key, obj];
      }
    }
    @catch (NSException *exception) {
      [desc appendFormat:@"%@\t%@ = %@,\n", tab, key, obj];
    }
  }
}
```

我们只是尝试去解析可视化数据，若解析成功则显示出来，若解析失败就原样输出。

#测试

我们来测试一下效果，我们打印如下一个字典：

```
NSString *str = @"我是转换成data格式的字符串";
NSData *dataString = [NSData dataWithBytes:str.UTF8String length:str.length];
NSDictionary *dataSet = @{@"key": @"字典转成data",
                          @"key1": @"在set、数组、字典中嵌套"};
NSData *dataSetItem = [NSJSONSerialization dataWithJSONObject:dataSet
                                                      options:NSJSONWritingPrettyPrinted
                                                        error:nil];

NSMutableSet *set = [NSMutableSet setWithArray:@[@"可变集合",
                                                 @"字典->不可变集合->可变集合",
                                                 dataSetItem]];
NSDictionary *dict = @{@"name"  : @"标哥的技术博客",
                       @"title" : @"http://www.henishuo.com",
                       @"count" : @(11),
                       @"dataString" : dataString,
                       @"results" : [NSSet setWithObjects:@"集合值1", @"集合值2", set , nil],
                       @"summaries" : @[@"sm1",
                                        @"sm2",
                                        @{@"keysm": @{@"stkey": @"字典->数组->字典->字典"}},
                                        dataSetItem],
                       @"parameters" : @{@"key1" : @"value1",
                                         @"key2": @{@"key11" : @"value11",
                                                    @"key12" : @[@"三层", @"字典->字典->数组"]},
                                         @"key13": dataSetItem},
                       @"hasBug": @[@"YES",@"NO"],
                       @"contact" : @[@"关注博客地址：http://www.henishuo.com",
                                      @"QQ群: 324400294",
                                      @"关注微博：标哥Jacky",
                                      @"关注GITHUB：CoderJackyHuang"]};
NSLog(@"%@", dict);
```

打印效果如下：

```
2015-12-31 16:47:42.352 demo[58176:2693559] 	{
	hasBug = 	(
		"YES",
		"NO",
	),
	dataString = "我是转换成",
	title = "http://www.henishuo.com",
	count = 11,
	results = 	{(
		"集合值2",
		"集合值1",
			{(
			"可变集合",
			"字典->不可变集合->可变集合",
				{
				key = "字典转成data",
				key1 = "在set、数组、字典中嵌套",
			},
		)},
	)},
	summaries = 	(
		"sm1",
		"sm2",
			{
			keysm = 	{
				stkey = "字典->数组->字典->字典",
			},
		},
			{
			key = "字典转成data",
			key1 = "在set、数组、字典中嵌套",
		},
	),
	contact = 	(
		"关注博客地址：http://www.henishuo.com",
		"QQ群: 324400294",
		"关注微博：标哥Jacky",
		"关注GITHUB：CoderJackyHuang",
	),
	name = "标哥的技术博客",
	parameters = 	{
		key1 = "value1",
		key13 = 	{
			key = "字典转成data",
			key1 = "在set、数组、字典中嵌套",
		},
		key2 = 	{
			key11 = "value11",
			key12 = 	(
				"三层",
				"字典->字典->数组",
			),
		},
	},
}
```

#如何使用

首先要得到源代码，安装方式有两种，一种是直接使用`Cocoapods`：

```
pod 'HYBUnicodeReadable', '~> 1.0'
```

或者直接下载源代码[https://github.com/CoderJackyHuang/HYBUnicodeReadable](https://github.com/CoderJackyHuang/HYBUnicodeReadable)，放入工程中即可

>温馨提示：不需要引入头文件，即可达到效果！

#写在最后

如果在使用过程中出现任何bug，请反馈作者以便及时修正！谢谢您的支持！

**如果觉得赞，请给一个star**

