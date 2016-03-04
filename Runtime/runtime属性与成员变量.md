#前言

在runtime中，`objc_property_t`代表属性，`Ivar`代表成员变量。本篇讲解这两大类型的具体实现、区别及各自常用的操作。

#objc\_property\_t属性

`objc_property_t`代表属性，而它又是一个结构体指针：

```
/// An opaque type that represents an Objective-C declared property.
typedef struct objc_property *objc_property_t;
```

`objc_property`是内置的类型，与之关联的还有一个`objc_property_attribute_t`，它是属性的`attribute`，也就是其实是对属性的详细描述，包括属性名称、属性编码类型、原子类型/非原子类型等。它的定义如下：

```
/// Defines a property attribute
typedef struct {
    const char *name;           /**< The name of the attribute */
    const char *value;          /**< The value of the attribute (usually empty) */
} objc_property_attribute_t;
```

其中，`value`通常是空的，但是对于类型是有值的。

##定义测试类

我们先定义一个用于测试的类，用于研究属性及成员变量：

```
@interface HYBPropertyLearn : NSObject {
  float _privateName;
  float _privateAttribute;
}

@property (nonatomic, copy) NSString *title;
@property (nonatomic, strong) NSArray *names;
@property (nonatomic, assign) int count;
@property (nonatomic, weak) id delegate;
@property (atomic, strong) NSNumber *atomicProperty;

+ (void)test;

@end
```
##objc\_property\_t与objc\_property\_attribute\_t

我们先定义获取属性的方法：

```
- (void)getAllProperties {
  unsigned int outCount = 0;
  objc_property_t *properties = class_copyPropertyList(self.class, &outCount);
  
  for (unsigned int i = 0; i < outCount; ++i) {
    objc_property_t property = properties[i];
     const char *propertyName = property_getName(property);
    
   const char *propertyAttributes = property_getAttributes(property);
    NSLog(@"%s  %s", propertyName, propertyAttributes);
    
    unsigned int count = 0;
    objc_property_attribute_t *attrbutes = property_copyAttributeList(property, &count);
    for (unsigned int j = 0; j < count; ++j) {
      objc_property_attribute_t attribute = attrbutes[j];
      
      const char *name = attribute.name;
      const char *value = attribute.value;
        NSLog(@"name: %s   value: %s", name, value);
    }
    
    free(attrbutes);
  }
  
  free(properties);
}
```

1. 通过`class_copyPropertyList`函数来获取属性列表，其中属性列表是使用`@property`声明的列表，对于直接使用声明为成员变量的，都不会出现在属性列表中，这也是正常的。
2. 我们通过runtime提供的`property_getName`函数来获取属性名称。
3. 若有获取属性的详细描述，可通过runtime提供的`property_getAttributes`函数来获取。
4. 若有获取属性中的`objc_property_attribute_t`列表，可以通过`property_copyAttributeList`函数来获取。
5. 若有获取单独的`objc_property_attribute_t`的`name`或者`value`，直接使用点语法即可，它是一个结构体。

对于上面的方法，其打印结果为：

```
title  T@"NSString",C,N,V_title
name: T   value: @"NSString"
name: C   value: 
name: N   value: 
name: V   value: _title
names  T@"NSArray",&,N,V_names
name: T   value: @"NSArray"
name: &   value: 
name: N   value: 
name: V   value: _names
count  Ti,N,V_count
name: T   value: i
name: N   value: 
name: V   value: _count
delegate  T@,W,N,V_delegate
name: T   value: @
name: W   value: 
name: N   value: 
name: V   value: _delegate
atomicProperty  T@"NSNumber",&,V_atomicProperty
name: T   value: @"NSNumber"
name: &   value: 
name: V   value: _atomicProperty
```

从打印结果可以分析出来:

* 通过`property_getName`获取得的是属性的名称，比如上面的title。
* 通过`property_getAttributes`函数获取到的是属性具体的描述，如`T@"NSString",C,N,V_title`其中`T`是固定的.
* 通过`objc_property_attribute_t`结构体可以直接获取`name`和`value`

下面详细地说明`property_getAttributes`获取到的属性的语义：

* 第一：T总是第一个，它代表类型。 对于类类型，它都是T@"类型"，其中@表示对象类型；对于id类型，它就是@；而对于基本数据类型，它都是`T`加上编码类型（@encode(type)），比如int类型就是Ti。
* 第二：表示属性编码类型，比如是C表示copy，&表示strong，W表示weak等。待会再详细地说明。若是基本类型，它默认是assign，因此此时它是空的。
* 第三：指定是nonatomic还是atomic，若是nonatomic，则用N表示，若是atomic，则值为空。比如上面的count属性的详细描述为：Ti,N,V\_count，它表示int类型、nonatomic、成员变量名为\_count；而上面的atomicProperty属性的详细描述为:T@"NSNumber",&,V\_atomicProperty，它表示类型为NSNumber且为对象类型、strong、atomic、成员变量名为\_atomicProperty。
* 第四：在详细描述中，属性名称是V开头，后面跟着成员变量名称。

下面再详细地说明通过`property_copyAttributeList`所获取到的`name`和`value`：

title属性的详细描述T@"NSString",C,N,V\_title通过解析得到：

```
// T的值@表示对象，"NSString"表示类型字符串
name: T   value: @"NSString"
// C表示copy，值为空
name: C   value: 
// N表示nonatomic，值为空
name: N   value: 
// V表示成员变量名
name: V   value: _title
```

names属性的详细描述T@"NSArray",&,N,V\_names通过解析得到：

```
// T的值中@表示对象类型，类型名称为"NSArray"
name: T   value: @"NSArray"
// &表示strong类型，在MRC下表示retain，值为空
name: &   value: 
// N表示nonatomic，值为空
name: N   value: 
// V表示成员变量名，值为_names
name: V   value: _names
```

count属性的详细描述Ti,N,V\_count通过解析得到：

```
// T的值表示基本类型，i表示int
name: T   value: i
// N表示nonatomic，值为空
name: N   value: 
// V表示成员变量名，值为_count
name: V   value: _count

// 由于类型为基本类型，指定的是assign，它是默认的，因此所解析中并没有这个name，它会省略掉
```

delegate属性的详细描述T@,W,N,V\_delegate通过解析得到：

```
// 我们所声明的delegate属性是id类型，因此也是类类型
// T的值@表示对象，但是它是id类型，没有具体的类类型名称，因此为空
name: T   value: @
// W表示weak
name: W   value: 
// N表示nonatomic
name: N   value:
// V表示成员变量名，值为_delegate 
name: V   value: _delegate
```

atomicProperty属性的详细描述T@"NSNumber",&,V\_atomicProperty通过解析得到：

```
// 声明为：
// @property (atomic, strong) NSNumber *atomicProperty;

// T的值@表示对象类型，类型为`NSNumber`
name: T   value: @"NSNumber"
// &表示strong类型，在MRC下表示retain
name: &   value: 
// V表示成员变量名，值为_atomicProperty
name: V   value: _atomicProperty

// 由于它是atomic原子操作，因此没有N这个name
```

通过上面的获取所有的属性，并不能获取到所有的成员变量，只能获取到声明为属性而自动创建对应的成员变量，这样是不能满足需求的，因此下面我们来学习一上成员变量。

#Ivar

成员变量通过`Ivar`表示，它是`objc_ivar`结构体指针：

```
/// An opaque type that represents an instance variable.
typedef struct objc_ivar *Ivar;
```

而`objc_ivar`结构的定义为：

```
struct objc_ivar {
    // 成员变量名
    char *ivar_name                 OBJC2_UNAVAILABLE;  
    // 成员变量encode类型
    char *ivar_type                 OBJC2_UNAVAILABLE;  
    // 基地址偏移字节
    int ivar_offset                 OBJC2_UNAVAILABLE;  
#ifdef __LP64__
    int space                       OBJC2_UNAVAILABLE;
#endif
}
```

我们来写一个获取所有成员变量的方法：

```
- (void)getAllMemberVariables {
  unsigned int outCount = 0;
  Ivar *ivars = class_copyIvarList(self.class, &outCount);
  
  for (unsigned int i = 0; i < outCount; ++i) {
    Ivar ivar = ivars[i];
    const char *name = ivar_getName(ivar);
    const char *type = ivar_getTypeEncoding(ivar);
   
    NSLog(@"name: %s encodeType: %s", name, type);
  }
  
  free(ivars);
}
```

其打印结果为：

```
name: _privateName encodeType: f
name: _privateAttribute encodeType: f
name: _count encodeType: i
name: _title encodeType: @"NSString"
name: _names encodeType: @"NSArray"
name: _delegate encodeType: @
name: _atomicProperty encodeType: @"NSNumber"
```

从打印结果可以看出来了，不管是属性定义还是直接声明为成员变量，都可以获取到变量名称。通过获取到所有的成员变量名称，那么我们就可以生成getter、setter方法，做我们想做的事情了。

#参考

关于encode，苹果提供了所有的类型，请阅读官方文档：[Type Encoding](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)

#源代码

大家可以到我的github下载本教程的demo：[RuntimeDemo](https://github.com/CoderJackyHuang/RuntimeDemo)

**喜欢就加个star**

#关注我

如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

#支持并捐助

如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
