>#Swift自动引用计数

##引言
Swift使用自动引用计数（ARC）机制来处理内存。通常情况下，Swift内存管理机制会自动管理内存，无须我们考虑内存的管理。ARC会在类的实例不再被使用（也就是没有引用）时，会自动释放其占用的内存。

可是，在少数情况下，ARC需要更多地了解我们代码之间的联系，才能正确管理内存。本篇文章就这少数情况而讨论和分析其应用场景及如何更好地解决循环引用的问题。

>注意:ARC仅应用于类的实例。结构体和枚举类型是值类型，不是引用类型，也不是通过引用的方式存储和传递。

##ARC的工作机制
创建类实例时，该实例就拥有了一块内存，存储相关信息，且会使该实例的引用计数+1，如下面的person实例当前的引用计数就为1，当前person对象只有一个引用。

```
class Person: NSObject {
  deinit {
    print("deinit")
  }
}

var person: Person? = Person()
```

如果有多个引用，则该实例就不会被释放，看下面的`mary`和`lili`都引用了`person`,使其所指向的内存引用计数+2：

```
var mary = person
var lili = person
```

现在我们将`mary`,`lili`设为`nil`，则person所指向的内存块引用计数-2，但是并不为0，因此还不会释放。

```
mary = nil
lili = nil
```

现在我们再将`person`设为`nil`，会如何呢？自然会得到释放了，因此就打印`deinit`出来了。

```
person = nil
```

为了确保使用中的实例不会被销毁，ARC会跟踪和计算每一个实例正在被多少属性，常量和变量所引用。哪怕实例的引用数为1，ARC都不会销毁这个实例。

无论将实例赋值给属性、常量或变量，它们都会创建此实例的**强引用**。之所以称之为**“强”**引用，是因为它会将实例牢牢的保持住，只要强引用还在，实例是不允许被销毁的。

##场景一：类实例间的循环强引用
下面看一种这么一种情形：人可以有公寓，公寓可以属于某个人。

```
class Person: NSObject {
   var name: String
  
  // 不一定人人都有公寓，因此设置为可选类型更合适
  var apartment: Apartment?
  
  init(name: String) {
    self.name = name
  }
  
  deinit {
    print("deinit person: \(self.name)")
  }
}

class Apartment: NSObject {
   var unit: String
  // 公寓也可能还没有拥有者，设置为可选类型
  // 注意，弱引用只能声明为变量，不能声明为常量
  weak var owner: Person?
  
  init(unit: String) {
    self.unit = unit
  }
  
  deinit {
    print("deinit apartment: \(self.unit)")
  }
}

var mary: Person? = Person(name: "Mary")
var yaya: Apartment? = Apartment(unit: "yaya")

// 下面是建立两者之间的关联关系
mary?.apartment = yaya
yaya?.owner = mary
```

现在将这两个对象的指向都修改为nil:

```
// 现在释放，然后就会得到打印结果：
// deinit person: Mary
// deinit apartment: yaya
mary = nil
yaya = nil
```
说明这两个对象都得到正确的释放了。这与我们预期的是一致的。

下面换一种写法：如果我们将`var apartment: Apartment?`也声明为`weak`呢？也就是改成`weak var apartment: Apartment?`，现在我们只将`yaya`改为`nil`会如何呢？

```
yaya = nil
print(mary?.apartment?.unit)
```
* 当`apartment`声明为`var apartment: Apartment?`，其打印结果为`Optional("yaya")`
* 当`apartment`声明为`weak var apartment: Apartment?`，其打印结果为`nil`。

虽然这么做两者都得到释放，但是这跟我们预期的结果不一致。这是因为双方都是弱引用，当`yaya =nil`时，其引用计数值为0，因此会被释放。这时候再去访问，就不存在了。如果其类型不是可选类型，释放后再访问，会导致`crash`的。

如果两个类实例之间都是可选类型，使用`weak`关键字来处理弱引用更好，但应该是**一强一弱引用**。

##场景二：类实例与属性的循环强引用
下面的例子定义了两个类，Customer和CreditCard，模拟了银行客户和客户的信用卡。这两个类中，每一个都将另外一个类的实例作为自身的属性。这种关系可能会造成循环强引用

```
class Customer {
  let name: String
  var card: CreditCard?
  
  init(name: String) {
    self.name = name
  }
  
  deinit {
    print("\(name) is being deinitialized")
  }
}

class CreditCard {
  let number: UInt64
  
  // 使用unowned关键字声明为无主引用，可以声明为变量或者常量，但是不能声明为可选类型
  // 对于无主引用，永远都是有值的，因此不可声明为可选类型。
  unowned let customer: Customer
  
  init(number: UInt64, customer: Customer) {
    self.number = number
    self.customer = customer
  }
  
  deinit {
    print("Card #\(number) is being deinitialized")
  }
}

var john: Customer? = Customer(name: "John")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)
```

现在我们将john置为nil，其结果如何呢？

```
// 打印结果：
// John is being deinitialized
// Card #1234567890123456 is being deinitialized
john = nil
```
根据打印结果，我们可以看到这两个实例都得到正确的释放了。使用unowned关键字声明为无主引用，可以声明为变量或者常量，但是不能声明为可选类型。对于无主引用，永远都是有值的，因此不可声明为可选类型。

>注意:
如果试图在实例被销毁后，访问该实例的无主引用，会触发运行时错误。使用无主引用，你必须确保引用始终指向一个未销毁的实例。还需要注意的是,如果试图访问实例已经被销毁的无主引用，Swift确保程序会直接崩溃，而不会发生无法预期的行为。

##场景三：闭包引起的循环强引用
下面我们定义`DemoView`和`DemoController`类，前者有一个闭包作为属性，用于反向传值到后者。

```
typealias DemoClosure = (isSelected: Bool) ->Void
class DemoView: UIView {
  var closure: DemoClosure?
  
  func callback(selected: Bool) {
    if let callback = closure {
      callback(isSelected: selected)
    }
  }
  
  deinit {
    print("demo view deinit")
  }
}
class DemoController: UIViewController {
  var demoView: DemoView?
  var name: String = "DemoControllerName"
  
  override func viewDidLoad() {
    super.viewDidLoad()
    
    demoView = DemoView()
    self.view.addSubview(demoView!)
    // 注意，这里使用了[unowned self]无主引用
    demoView?.closure = { [unowned self](isSelcted: Bool) in
      // 闭包内，强引用了self，也就是DemoController
      print("\(self.name)")
    }
    
    
    // 测试调用
    self.passToDmeoView(true)
  }
  
  func passToDmeoView(selected: Bool) {
    demoView?.callback(selected)
  }
  
  deinit {
    print("DemoController deinit")
  }
}
```

对比一下添加了`[unowned self]`与未添加的打印结果：

* 不添加`[unowned self]`，直接使用`self.name`时，当返回上一个界面时，由于循环强引用，DemoController对象和DemoView者得不到释放。
* 添加了`[unowned self]`后，打印结果说明可以得到释放，如下：

```
DemoController deinit
demo view deinit
```

我们在使用闭包时，一定要注意循环强引用的问题，否则很容易千万内存泄露。除了使用无主引用之外，还有`weak`关键字来声明使用弱使用，比如：

```
lazy var closure: (Int, String) -> String = {
    [unowned self, weak delegate = self.delegate] (index: Int, title: String) -> String in
    // closure body goes here
    print("\(self.name)")
    delegate.callBack(title)
}
```
为了防止循环强引用，对于`weak delegate = self.delegate!`也需要使用弱引用。

>对于闭包内的引用，何时使用**弱引用**，何时使用**无主引用**呢？
>
>* 在闭包和捕获的实例***总是互相引用时并且总是同时销毁***时，将闭包内的捕获定义为**无主引用**，如上面的`closure`闭包中的`self`与`closure`问题互相引用且同时销毁。
>* 在被捕获的引用可能会变为nil时，将闭包内的捕获定义为弱引用。弱引用总是可选类型，并且当引用的实例被销毁后，弱引用的值会自动置为nil。
>* 注意：如果被捕获的引用绝对不会变为nil，应该用无主引用，而不是弱引用。如果上面的`self.delegate`不是可选类型，那么其值会一直存在，我们就不应该使用`weak`引用，而是使用`unowned`引用。


##总结
解决循环强引用的类型有两种：

* 使用`weak`关键字声明为弱引用。使用`weak`的弱引用只能声明为变量（不能声明为常量），表明其值能在运行时被修改。
* 使用`unowned`关键声明为无主引用。无主引用总是被定义为非可选类型，因为其永远是有值的，可声明为常量或变量，值不能为nil。
* 常见的循环强引用场景有：类实例与类属性间循环强引用、类实例与类实例间循环强引用、类与闭包循环强引用等。


##关注我
**公众号搜索「iOS开发技术分享」快速关注，微信号：iOSDevShares**

**QQ群：324400294**

