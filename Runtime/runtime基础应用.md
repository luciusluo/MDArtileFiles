
#引言

相信很多同学都听过运行时，但是我相信还是有很多同学不了解什么是运行时，到底在项目开发中怎么用？什么时候适合使用？想想我们的项目中，到底在哪里使用过运行时呢？还能想起来吗？另外，在面试的时候，是否经常有笔试中要求运用运行时或者在面试时面试官会问是否使用过运行时，又是如何使用的？

回想自己，曾经在面试中被面试官拿运行时刁难过，也在笔试中遇到过。因此，后来就深入地学习了`Runtime`机制，学习里面的API。所以才有了后来的组件封装中使用运行时。

相信我们都遇到过这样一个问题：我想在扩展（category）中添加一个属性，如果iOS是不允许给扩展类扩展属性的，那怎么办呢？答案就是**使用运行时机制**

#运行时机制

`Runtime`是一套比较底层的纯C语言的API, 属于C语言库, 包含了很多底层的C语言API。 
在我们平时编写的iOS代码中, 最终都是转成了runtime的C语言代码。

所谓运行时，也就是在编译时是不存在的，只是在运行过程中才去确定对象的类型、方法等。利用`Runtime`机制可以在程序运行时动态修改类、对象中的所有属性、方法等。

还记得我们在网络请求数据处理时，调用了`-setValuesForKeysWithDictionary:`方法来设置模型的值。这里什么原理呢？为什么能这么做？其实就是通过`Runtime`机制来完成的，内部会遍历模型类的所有属性名，然后设置与`key`对应的属性名的值。

我们在使用运行时的地方，都需要包含头文件：`#import <objc/runtime.h>`。如果是`Swift`就不需要包含头文件，就可以直接使用了。

#获取对象所有属性名


利用运行时获取对象的所有属性名是可以的，但是变量名获取就得用另外的方法了。我们可以通过`class_copyPropertyList`方法获取所有的属性名称。

下面我们通过一个`Person`类来学习，这里的方法没有写成扩展，只是为了简化，将获取属性名的方法直接作为类的实例方法：

##Objective-C版


```
@interface Person : NSObject {
  NSString *_variableString;
}

// 默认会是什么呢？
@property (nonatomic, copy) NSString *name;

// 默认是strong类型
@property (nonatomic, strong) NSMutableArray *array;

// 获取所有的属性名
- (NSArray *)allProperties;

@end
```

下面主要是写如何获取类的所有属性名的方法。注意，这里的`objc_property_t`是一个结构体指针`objc_property *`，因此我们声明的`properties`就是二维指针。在使用完成后，我们一定要记得释放内存，否则会造成内存泄露。这里是使用的是C语言的API，因此我们也需要使用C语言的释放内存的方法`free`。


```
/// An opaque type that represents an Objective-C declared property.
typedef struct objc_property *objc_property_t;
```

```
- (NSArray *)allProperties {
  unsigned int count;
  
  // 获取类的所有属性
  // 如果没有属性，则count为0，properties为nil
  objc_property_t *properties = class_copyPropertyList([self class], &count);
  NSMutableArray *propertiesArray = [NSMutableArray arrayWithCapacity:count];
  
  for (NSUInteger i = 0; i < count; i++) {
    // 获取属性名称
    const char *propertyName = property_getName(properties[i]);
    NSString *name = [NSString stringWithUTF8String:propertyName];
    
    [propertiesArray addObject:name];
  }
  
  // 注意，这里properties是一个数组指针，是C的语法，
  // 我们需要使用free函数来释放内存，否则会造成内存泄露
  free(properties);
  
  return propertiesArray;
}
```

现在，我们来测试一下，我们的方法是否正确获取到了呢？看下面的打印结果就明白了吧！

```
Person *p = [[Person alloc] init];
p.name = @"Lili";

size_t size = class_getInstanceSize(p.class);
NSLog(@"size=%ld", size);

for (NSString *propertyName in p.allProperties) {
  NSLog(@"%@", propertyName);
}
// 打印结果：
// 2015-10-23 17:28:38.098 PropertiesDemo[1120:361130] size=48
// 2015-10-23 17:28:38.098 PropertiesDemo[1120:361130] copiedString
// 2015-10-23 17:28:38.098 PropertiesDemo[1120:361130] name
// 2015-10-23 17:28:38.098 PropertiesDemo[1120:361130] unsafeName
// 2015-10-23 17:28:38.099 PropertiesDemo[1120:361130] array
```

##Swift版

对于Swift版，使用C语言的指针就不容易了，因为Swift希望尽可能减少C语言的指针的直接使用，因此在Swift中已经提供了相应的结构体封装了C语言的指针。但是看起来好复杂，使用起来好麻烦。看看Swift版的获取类的属性名称如何做：

```
class Person: NSObject {
  var name: String = ""
  var hasBMW = false
  
  override init() {
    super.init()
  }
  
  func allProperties() ->[String] {
    // 这个类型可以使用CUnsignedInt,对应Swift中的UInt32
    var count: UInt32 = 0
    
    let properties = class_copyPropertyList(Person.self, &count)
    
    var propertyNames: [String] = []
    
    // Swift中类型是严格检查的，必须转换成同一类型
    for var i = 0; i < Int(count); ++i {
      // UnsafeMutablePointer<objc_property_t>是
      // 可变指针，因此properties就是类似数组一样，可以
      // 通过下标获取
      let property = properties[i]
      let name = property_getName(property)
      
      // 这里还得转换成字符串
      let strName = String.fromCString(name);
      propertyNames.append(strName!);
    }
    
    // 不要忘记释放内存，否则C语言的指针很容易成野指针的
    free(properties)
    
    return propertyNames;
  }
}
```

关于Swift中如何C语言的指针问题，这里不细说，如果需要了解，请查阅相关文章。
测试一下是否获取正确：

```
let p = Person()
p.name = "Lili"

// 打印结果：["name", "hasBMW"]，说明成功
p.allProperties()
```

#获取对象的所有属性名和属性值


对于获取对象的所有属性名，在上面的`-allProperties`方法已经可以拿到了，但是并没有处理获取属性值，下面的方法就是可以获取属性名和属性值，将属性名作为`key`，属性值作为`value`。
##Objective-C版

```
- (NSDictionary *)allPropertyNamesAndValues {
  NSMutableDictionary *resultDict = [NSMutableDictionary dictionary];
  
  unsigned int outCount;
  objc_property_t *properties = class_copyPropertyList([self class], &outCount);
  
  for (int i = 0; i < outCount; i++) {
    objc_property_t property = properties[i];
    const char *name = property_getName(property);
    
    // 得到属性名
    NSString *propertyName = [NSString stringWithUTF8String:name];
    
    // 获取属性值
    id propertyValue = [self valueForKey:propertyName];
    
    if (propertyValue && propertyValue != nil) {
      [resultDict setObject:propertyValue forKey:propertyName];
    }
  }
  
  // 记得释放
  free(properties);
  
  return resultDict;
}
```
测试一下：
```
// 此方法返回的只有属性值不为空的属性
NSDictionary *dict = p.allPropertyNamesAndValues;
for (NSString *propertyName in dict.allKeys) {
  NSLog(@"propertyName: %@ propertyValue: %@", 
        propertyName, 
        dict[propertyName]);
}
```

看下打印结果，属性值为空的属性并没有打印出来，因此字典的key对应的value不能为nil：

```
propertyName: name propertyValue: Lili
```

##Swift版

```
func allPropertyNamesAndValues() ->[String: AnyObject] {
    var count: UInt32 = 0
    let properties = class_copyPropertyList(Person.self, &count)
    
    var resultDict: [String: AnyObject] = [:]
    for var i = 0; i < Int(count); ++i {
      let property = properties[i]
      
      // 取得属性名
      let name = property_getName(property)
      if let propertyName = String.fromCString(name) {
        // 取得属性值
        if let propertyValue = self.valueForKey(propertyName) {
          resultDict[propertyName] = propertyValue
        }
      }
    }
    
    free(properties)
    
    return resultDict
}
```

测试一下：

```
let dict = p.allPropertyNamesAndValues()
for (propertyName, propertyValue) in dict.enumerate() {
  print("propertyName: \(propertyName), propertyValue: \(propertyValue)")
}
```
打印结果与上面的一样，由于`array`属性的值为nil，因此不会处理。

```
propertyName: 0, propertyValue: ("name", Lili)
```

#获取对象的所有方法名


通过`class_copyMethodList`方法就可以获取所有的方法。

##Objective-C版

```
- (void)allMethods {
  unsigned int outCount = 0;
  Method *methods = class_copyMethodList([self class], &outCount);
  
  for (int i = 0; i < outCount; ++i) {
    Method method = methods[i];
    
    // 获取方法名称，但是类型是一个SEL选择器类型
    SEL methodSEL = method_getName(method);
    // 需要获取C字符串
    const char *name = sel_getName(methodSEL);
   // 将方法名转换成OC字符串
    NSString *methodName = [NSString stringWithUTF8String:name];
    
    // 获取方法的参数列表
    int arguments = method_getNumberOfArguments(method);
    NSLog(@"方法名：%@, 参数个数：%d", methodName, arguments);
  }
  
  // 记得释放
  free(methods);
}
```

测试一下：

```
[p allMethods];
```

调用打印结果如下，为什么参数个数看起来不匹配呢？比如`-allProperties`方法，其参数个数为0才对，但是打印结果为2。根据打印结果可知，无参数时，值就已经是2了。：

```
方法名：allProperties, 参数个数：2
方法名：allPropertyNamesAndValues, 参数个数：2
方法名：allMethods, 参数个数：2
方法名：setArray:, 参数个数：3
方法名：.cxx_destruct, 参数个数：2
方法名：name, 参数个数：2
方法名：array, 参数个数：2
方法名：setName:, 参数个数：3
```

##Swift版

```
func allMethods() {
  var count: UInt32 = 0
  let methods = class_copyMethodList(Person.self, &count)
  
  for var i = 0; i < Int(count); ++i {
    let method = methods[i]
    let sel = method_getName(method)
    let methodName = sel_getName(sel)
    let argument = method_getNumberOfArguments(method)
    
    print("name: \(methodName), arguemtns: \(argument)")
  }
  
  free(methods)
}
```

测试一下调用：

```
p.allMethods()
```
打印结果与上面的Objective-C版的一样。

#获取对象的成员变量

要获取对象的成员变量，可以通过`class_copyIvarList`方法来获取，通过`ivar_getName`来获取成员变量的名称。对于属性，会自动生成一个成员变量。

##Objective-C版

```
- (NSArray *)allMemberVariables {
  unsigned int count = 0;
  Ivar *ivars = class_copyIvarList([self class], &count);
  
  NSMutableArray *results = [[NSMutableArray alloc] init];
  for (NSUInteger i = 0; i < count; ++i) {
    Ivar variable = ivars[i];
    
    const char *name = ivar_getName(variable);
    NSString *varName = [NSString stringWithUTF8String:name];
    
    [results addObject:varName];
  }
  
  free(ivars);
  
  return results;
}
```

测试一下：

```  
for (NSString *varName in p.allMemberVariables) {
  NSLog(@"%@", varName);
}
```

打印结果说明属性也会自动生成一个成员变量：

```
2015-10-23 23:54:00.896 PropertiesDemo[46966:3856655] _variableString
2015-10-23 23:54:00.897 PropertiesDemo[46966:3856655] _name
2015-10-23 23:54:00.897 PropertiesDemo[46966:3856655] _array
```

##Swift版

`Swift`的成员变量名与属性名是一样的，不会生成下划线的成员变量名，这一点与Oc是有区别的。

```
func allMemberVariables() ->[String] {
  var count:UInt32 = 0
  let ivars = class_copyIvarList(Person.self, &count)
  
  var result: [String] = []
  for var i = 0; i < Int(count); ++i {
    let ivar = ivars[i]
    
    let name = ivar_getName(ivar)
    
    if let varName = String.fromCString(name) {
      result.append(varName)
    }
  }
  
  free(ivars)
  
  return result
}
```
测试一下：

```
let array = p.allMemberVariables()
for varName in array {
  print(varName)
}
```

打印结果，说明`Swift`的属性不会自动加下划线，属性名就是变量名：

```
name
array
```

#运行时发消息


iOS中，可以在运行时发送消息，让接收消息者执行对应的动作。可以使用`objc_msgSend`方法，发送消息。

##Objective-C版

```  
Person *p = [[Person alloc] init];
p.name = @"Lili";
objc_msgSend(p, @selector(allMethods));
```
这样就相当于手动调用`[p allMethods];`。但是编译器会抱错，问题提示期望的参数为0，但是实际上有两个参数。解决办法是，关闭严格检查：

![image](https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD2JLSfAicem2zkAHcrKXEbawib3bNlicY6DkibaItQCK8VyKGowW3u8lNTibGxdQ6pA1dMFSYoiacyu7ib7A/0?wx_fmt=png)

##Swift版

很抱歉，似乎在Swift中已经没有这种写法了。如果有，请告诉我。

#Category扩展"属性"



iOS的category是不能扩展存储属性的，但是我们可以通过运行时关联来扩展“属性”。

##Objective-C版

假设扩展下面的“属性”：

```
// 由于扩展不能扩展属性，因此我们这里在实现文件中需要利用运行时实现。
typedef void(^HYBCallBack)();
@property (nonatomic, copy) HYBCallBack callback;
```

在实现文件中，我们用一个静态变量作为key：

```
const void *s_HYBCallbackKey = "s_HYBCallbackKey";

- (void)setCallback:(HYBCallBack)callback {
  objc_setAssociatedObject(self, s_HYBCallbackKey, callback, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (HYBCallBack)callback {
  return objc_getAssociatedObject(self, s_HYBCallbackKey);
}
```

其实就是通过`objc_getAssociatedObject`取得关联的值，通过`objc_setAssociatedObject`设置关联。

##Swift版


Swift版的要想扩展闭包，就比OC版的要复杂得多了。这里只是例子，写了一个简单的存储属性扩展。

```

let s_HYBFullnameKey = "s_HYBFullnameKey"

extension Person {
  var fullName: String? {
    get { return objc_getAssociatedObject(self, s_HYBFullnameKey) as? String }
    set {
      objc_setAssociatedObject(self, s_HYBFullnameKey, newValue, .OBJC_ASSOCIATION_COPY_NONATOMIC)
    }
  }
}
```

#总结


在开发中，我们比较常用的是使用关联属性的方式来扩展我们的“属性”，以便在开发中简单代码。我们在开发中使用关联属性扩展所有响应事件、将代理转换成block版等。比如，我们可以将所有继承于UIControl的控件，都拥有block版的点击响应，那么我们就可以给UIControl扩展一个TouchUp、TouchDown、TouchOut的block等。

对于动态获取属性的名称、属性值使用较多的地方一般是在使用第三方库中，比如MJExtension等。这些三方库都是通过这种方式将Model转换成字典，或者将字典转换成Model。

#写在最后


如果文章中出现有疑问的地方，请在评论中评论，笔者会在第一时间回复您的！

阅读容易，写文章难！且看且珍惜！

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**阅读原文：**[http://www.henishuo.com/ios-runtime/](http://www.henishuo.com/ios-runtime/)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
