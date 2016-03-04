>#Swift通过类名动态创建对象


##前言
最近一些朋友问到我在Swift中如何通过类字符串名称的方式创建类实例的问题，起初以为与`Objective-C`的差不多吧，事实上还是有很大的差别的。下面是帮助朋友们之后，也随便总结而写下的文章。

> 注意：本篇文章中所涉及到的Swift代码都是Swift2.0的语法。

##先看ObjC中的方式
我们可以通过Class类型就可以调用alloc来分配内存，调用init方法来初始化。如：

```
Class cl = NSClassFromString(@"ViewController");
UIViewController *vc = [[cl alloc] init];
```

通常我们这么写法是用于循环创建的场景，通过公共基类接收，就可以指向所创建的对应的类名称的内存。

##Swift中的方式
今天是由于一位朋友突然询问我这么一个问题：
> swift中怎么通过类名称创建对象呢？
> 
> 一时并无法回答，因为一看到`Swift中NSClassFromString`返回的是`AnyClass`类型，而这个`AnyClass`类型为`public typealias AnyClass = AnyObject.Type`，这个`Type`具体是什么呢？为什么`option+点击`进不去，无法查看呢？`AnyObject`其实只是一个空协议，难道.Type是自动有的吗？这个本人也不清楚。


看看下面的方式：

```
var str = NSString.self()// 或者NSString.self.init()

str = "TestNSString"
print(str)
```

下面我们定义一个类：

```
class MyClass: NSObject {
 var member = 10
 
 override required init() {
   print("init")
 }
}
```

我们定义的类是继承于NSObject,这时我们这么测试：

```
// 打印出10
print( MyClass.self().member)
```
说明继承于NSObject后， 调用self()就可以创建对象了。

另外，我们使用NSClassFromString来创建试试：

```
 let className = NSStringFromClass(MyClass)
 print(className)
 let classType = NSClassFromString(className) as? MyClass.Type
 if let type = classType {
   let my = type.init()
   print(my.member)
 }
```

第一个打印就打印出`__lldb_expr_83.MyClass`。<br/>
第二个打印打印出`10`。type为MyClass.Type，通过MyClass.Type.init()是可以创建类对象的。

我们还可以通过MyClass.self.init()来创建对象，`print(MyClass.self.init().member)`打印出来也是10

注意：所创建的`MyClass`类中的`init`方法前面必须是`required`的，因为这么创建方式是使用meta type来创建的，如果不添加`required`，编译时就会报错。当我们修改继承方式，把`NSObject`改成`AnyObject`，其结果也一样，`AnyObject`只是协议，遵守协议。说明与继承方式无关。

##总结：
在Swift中，要创建对象有以下几种方式：

* 1、`NSString.self()// 或者NSString.self.init()`
* 2、`let myClass = MyClass.Type.init()`
* 3、`let myClass = MyClass.self.init()`
* 4、` let type = NSClassFromString("MyClass") as! MyClass.Type`然后通过`type.init()`来创建对象


> 参考：[http://stackoverflow.com/questions/24049673/swift-class-introspection-generics
](http://stackoverflow.com/questions/24049673/swift-class-introspection-generics
)

#关注我

**公众号搜索「iOS开发技术分享」快速关注微信号：iOSDevShares**

**QQ群：324400294**

