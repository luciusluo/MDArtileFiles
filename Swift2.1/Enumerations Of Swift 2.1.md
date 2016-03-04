>#[原文出自：标哥的技术博客](http://www.henishuo.com/enumeration-of-swift/)

#前言

枚举为一组相关的值定义了一个共同的类型，使你可以在你的代码中以类型安全的方式来使用这些值，当然还有一个很重要的是它可能智能提示。

在C语言中，枚举会为一组整型值分配相关联的名称。Swift中的枚举更加灵活，不必给每一个枚举成员提供一个值。如果给枚举成员提供一个值（原始值），则该值的类型可以是字符串、字符、整型值或浮点数。

此外，枚举成员可以指定任意类型的关联值存储到枚举成员中，每一个枚举成员都可以有适当类型的关联值。

在Swift中，枚举类型是一等（first-class）类型，拥有在传统上只被类（class）所支持的特性，例如计算型属性用于提供枚举值的附加信息、实例方法用于提供和枚举值相关联的功能等、定义构造函数来提供一个初始值、可在原始实现的基础上扩展功能、可遵守协议来提供标准的功能等。

#枚举语法（Enumeration Syntax）

使用enum关键词来创建枚举并且把它们的整个定义放在一对大括号内：

```
enum SomeEnumeration {
    // 枚举定义放在这里
}
```

例如，我们声明一个上、下、左、右方向的枚举：

```
enum DirectionOperation {
  case Up
  case Down
  case Left
  case Right
}

// 指定方向为上
var direction = DirectionOperation.Up
```

那么dir的数据类型是什么呢？它是DirectionOperation类型，因为swift中的枚举不会自动给成员赋值为0,1...，它们本身就已经有完整的类型，那就是DirectionOperation。当然我们也可以指定类型。

像`direction`已经被赋值过，其数据类型已经明确，当我们再修改值时，不用在前面指定类型，而是只用枚举值就可以了：

```
direction = .Down
```

多个成员值可以出现在同一行上，用逗号隔开：

```
enum Planet {
    case Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}
```

#Switch匹配枚举值

```
enum Direction  {
  case North
  case South
  case East
  case West
}

let direction = Direction.South
switch direction {
case .North:
  print("north")
case .South:
  print(direction)
case .East:
  print("east")
case .West:
  print("west")
}
```

当不需要匹配每个枚举成员的时候，你可以提供一个default分支来涵盖所有未明确处理的枚举成员：

```
let somePlanet = Planet.Earth
switch somePlanet {
case .Earth:
    print("Mostly harmless")
default:
    print("Not a safe place for humans")
}
// 输出 "Mostly harmless”
```

#关联值（Associated Values）

swift中的枚举与objective-c中的枚举有很大的区别，swift中的枚举可以做到很多的功能，相当于是强化版本。

对于关联值，其实就是将某个成员的值要求是某种类型值，它们可以是不同类型的组合，这看起来有点像元组，因为元组也是不要求类型相同的。

我们假设这么一个场景：cell上有两个view，可以同时展开、同时收起、可以一个展开一个收起，那么我们就可以通过枚举类型来完成这样一个标识。

我们可以这么声明（不要在乎这里的命名，只是为了写个例子）：

```
enum ExpandStatus {
  case View1Status(Bool)
  case View2Status(Bool)
}
```

那么如何使用呢？可以这样：

```
let status = ExpandStatus.View1Status(true)
switch status {
case .View1Status(let isExpand):
  if isExpand {
    // view1展开
  } else {
    // view1收起
  }
case .View2Status(let isExpand):
  if isExpand {
    // view2展开
  } else {
    // view2收起
  }
}
```

使用枚举的目的是什么？为什么不直接使用常量？为什么不使用其它方式？当然，使用枚举自然有它的好处，不然谁会用呢？有什么好处？在笔者看来，它有以下好处：

* 枚举类型可以很明确语义，通过看枚举定义就可以看出来这个枚举代表什么，它又有几中状态，都可以直接看到。也就是说，如果有逻辑在内，那么通过枚举有多少个case，基本就是有多少种状态。
* 枚举类型易扩展。想想这么一种场景：突然有一天产品经理说，这里再增加一种状态，那么通过枚举就很容易实现，直接添加一个case分支即可，然后在相应要处理代码逻辑的地方，添加上新的分支处理逻辑就OK了。
* 枚举有关联值，它可以像函数那样简单调用。有时候一种状态是由多个值组合而成时，使用枚举的关联值就可以实现，而且语义依然很明确。如果是使用函数调用，那么就优化而言，自然没有枚举那么好。另外，关联值可以直接使用，它相当于传参数，而且还是明确的类型的参数。
* 使用枚举易维护。为什么说易维护，大家看到枚举大概就知道这是干嘛用的，有几种情况了吧。比那些直接使用常量或者变量语义要强多了。

笔者所能想到的好处，暂时就想到这么多了。大家认为还有哪些好处，可以在评论中回复哦！笔者会整理到文章中的。

#原始值的隐式赋值（Implicitly Assigned Raw Values）

在使用原始值为整数或者字符串类型的枚举时，不需要显式地为每一个枚举成员设置原始值，Swift 将会自动为你赋值。

##原始值类型为整型

我们利用整型的原始值来表示每个行星在太阳系中的顺序：

```
enum Planet: Int {
    case Mercury = 1, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}
```

因为我们明确指定的Planet类型的原始值类型为Int，那么它的成员的原始值都会是Int类型，如果我们不指定，那么它的成员会被自动赋值。

```
// 1
let mercury = Planet.Mercury.rawValue

// 2
let Venus = Planet.Venus.rawValue
```

那么后面的几个的原始值依次为3,4,5...

##原始值类型为字符串

下面我们声明一个指定原始值类型为String，注意CompassPoint类型是枚举类型，而不是String类型，大家不要搞混了：

```
enum CompassPoint: String {
    case North, South, East, West
}
``

那么对应的原始值为：

```
// North
let north = CompassPoint.North.rawValue

// South
let south = CompassPoint.South.rawValue
```

#原始值初始化实例（Initializing from a Raw Value）

如果在定义枚举类型的时候使用了原始值，那么将会自动获得一个初始化方法，这个方法接收一个叫做rawValue的参数，参数类型即为原始值类型，返回值则是枚举成员或nil。你可以使用这个初始化方法来创建一个新的枚举实例。

```
// p 就是CompassPoint.North
let p = CompassPoint(rawValue: "North")
```

#递归枚举（Recursive Enumerations）

使用`indirect`关键字指定该分支可以递归，注意像下面所定义的Add表达式中的递归参数类型那就是枚举类型才能递归哦：

```
enum Expression {
  case Number(Int)
  indirect case Add(Expression, Expression)
  indirect case multiple(Expression, Expression)
  
  func getResultValue() ->Int {
    switch self {
    case .Number(let value):
      return value
    case .Add(let lhs, let rhs):
      return lhs.getResultValue() + rhs.getResultValue()
    case .multiple(let lhs, let rhs):
      return lhs.getResultValue() * rhs.getResultValue()
    }
  }
}
```

我们来使用一下，看看这递归枚举如何使用：

```
// 3
let one = Expression.Number(1)
let two = Expression.Number(2)
let addResult = Expression.Add(one, two)

// 1
print(one.getResultValue())
// 2
print(two.getResultValue())
// 3
print(addResult.getResultValue())
```

swift给我们提供了这样的功能，有什么用呢？直接使用函数不就完事了吗？干嘛非得还要再用递归枚举呢？

我认为这苹果如此设计自有其道理，笔者觉得它的妙用之处在于简单递归、语义明确。如果是复杂的递归，因为类型不同，使用枚举递归就不太合适了。

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
