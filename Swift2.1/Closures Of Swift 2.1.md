#前言

闭包是自包含的功能代码块，可以在代码中使用或者用来作为参数传值。
在Swift中的闭包与C、OC中的blocks和其它编程语言（如Python）中的lambdas类似。
闭包可以捕获和存储上下文中定义的的任何常量和变量的引用。这就是所谓的变量和变量的自封闭，因此命名为”闭包“。

Swift还会处理所有捕获的引用的内存管理，全局函数和嵌套函数其实就是特殊的闭包。闭包的形式主要有：

1. 全局函数都是闭包，是特殊的闭包，有名字但不能捕获任何值。
2. 嵌套函数都是闭包，且有名字，也能捕获封闭函数内的值。
3. 闭包表达式都是无名闭包，使用轻量级语法，可以根据上下文环境捕获值。


Swift中的闭包有很多优化的地方:

* 根据上下文推断参数和返回值类型
* 从单行表达式闭包中隐式返回（也就是闭包体只有一行代码，可以省略return）
* 可以使用简化参数名，如$0, $1(从0开始，表示第i个参数...)
* 提供了尾随闭包语法(Trailing closure syntax)

>Swift版：2.1<br>
>Xcode:  7.2

#sort函数（Standard Library Functions）
 
 Swift标准库提供了名为sort的方法，会根据您提供的用于排序的闭包函数将已知类型数组中的值进行排序。一旦排序完成，sort(\_:)方法会返回一个与原数组大小相同,包含同类型元素且元素已正确排序的新数组。原数组不会被sort(_:)方法修改。
 
下面用Swift标准库中的sort方法来一步步简化闭包写法:

```
// sort函数需要两个参数
// 参数一：数组
// 参数二：一个闭包：带有两个参数，这两个参数类型与数组中的元素类型相同，返回值是Bool
var names = ["Swift", "Arial", "Soga", "Donary"]
```

##第一种方式：使用函数

```
func backwards(firstString: String, secondString: String) -> Bool {
	return firstString > secondString // 升序排序
}

// 这里第二个参数，传了一个函数
// reversed is equal to ["Swift", "Soga", "Donary", "Arial"]
var reversed = sort(nams, backwards)
```
 
##第二种方式：使用闭包方式

完整闭包写法是在花括号内有参数列表和返回值，用关键字in表明闭包体的开始。闭包的语法为：

```
{ (parameters) -> returnType in
    statements
}
```

其中`parameters`是参数列表，`->returnType` 是返回值类型，`in`是闭包体的开始。

###完整闭包写法

完整的闭包写法是不带任何缺省的：

```
// (firstString: String, secondString: String) 闭包参数列表
// -> Bool 指明闭包返回值类型是Bool
// in关键字表明闭包体的开始
reversed = sort(names, { (firstString: String, secondString: String) -> Bool in
	return firstString > secondString
})
```

###简化一

这里可以进一步简化写法，因为闭包代码比较短，可以写到一行上：

```
reversed = sort(names, { (firstString: String, secondString: String) -> Bool in return firstString > secondString})
```

###简化二

下面再进一步简化写法 ：根据环境上下文自动推断出类型

参数列表都没有指明类型，也没有指明返回值类型，这是因为swift可以根据上下文推测出：

```
// firstString和secondString的类型会是names数组元素的类型，而返回值类型会根据return语句结果得到
reversed = sort(names, { firstString, secondString in return firstString > secondString})
```

###简化三

再进一步简化：隐式返回（单行语句闭包）

因为闭包体只有一行代码，可以省略return：

```
reversed = sort(names, { firstString, secondString in firstString > secondString})
```

###简化四

再进一步简化：使用简化参数名（$i,i=0,1,2...从0开始的）

```
// Swift会推断出闭包需要两个参数，类型与names数组元素相同
reversed = sort(names, { $0 > $1 })	
```

最简单的一种写法：使用操作符，它是符号函数，也需要函数：

```
reversed = sort(names, >)	
```

#尾随闭包（Trailing Closures）

如果函数需要一个闭包参数作为参数，且这个参数是最后一个参数，而这个闭包表达式又很长时，使用尾随闭包是很有用的。尾随闭包可以放在函数参数列表外，也就是括号外。如果函数只有一个参数，那么可以把括号()省略掉，后面直接跟着闭包。

我们来看看数组提供的`map`方法，它需要一个参数，而且这个参数是一个闭包，那么它就属于尾随闭包了：

```
// Array的方法map()就需要一个闭包作为参数
let strings = numbers.map { // map函数后面的()可以省略掉
  (var number) -> String in
  var output = ""
  while number > 0 {
    output = String(number % 10) + output 
	number /= 10
  }
  return output
}
```

#捕获值（Capturing Values）

闭包可以根据环境上下文捕获到定义的常量和变量。闭包可以引用和修改这些捕获到的常量和变量，就算在原来的范围内定义为常量或者变量已经不再存在。在Swift中闭包的最简单形式是嵌套函数:

```
func increment(#amount: Int) -> (() -> Int) {
  var total = 0
  func incrementAmount() -> Int {
    total += amount // total是外部函数体内的变量，这里是可以捕获到的
	return total
  }
  return incrementAmount // 返回的是一个嵌套函数（闭包）
}	
	
// 闭包是引用类型，所以incrementByTen声明为常量也可以修改total
let incrementByTen = increment(amount: 10) 
incrementByTen() // return 10,incrementByTen是一个闭包

// 这里是没有改变对increment的引用，所以会保存之前的值
incrementByTen() // return 20	
incrementByTen() // return 30	

let incrementByOne = increment(amount: 1)
incrementByOne() // return 1
incrementByOne() // return 2	
incrementByTen() // return 40 
incrementByOne() // return 3
```

#闭包是引用类型（Closures Are Reference Types）

函数或闭包赋值给一个常量还是变量，实际上都是将常量或变量的值设置为对应函数或闭包的引用，也就是说将闭包赋值给了两个不同的常量或变量，两个值都会指向同一个闭包：

```
// 将函数或者闭包赋值给常量或者变量，都是对同一个函数或者闭包的引用
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen()

// 此时，alsoIncrementByTen和anotherIncrementByTen都是同一个函数或者闭包的引用 
let anotherIncrementByTen = alsoIncrementByTen
anotherIncrementByTen()
```

#非逃逸闭包(Nonescaping Closures)

当一个闭包作为参数传到一个函数中，但是这个闭包在函数返回之后才被执行，我们称该闭包从函数中逃逸。当你定义接受闭包作为参数的函数时，在参数名之前标注`@noescape`，用来指明这个闭包是不允许“逃逸”出这个函数的。将闭包标注`@noescape`能使编译器知道这个闭包的生命周期。

标注为`@noescape`，那么闭包只能在函数体中被执行，不能脱离函数体执行，所以编译器可以明确知道运行时的上下文，从而可以进行一些优化。

我们看看swift标准库函数`sort(_:)`，其定义就添加了`@noescape`，表示这个闭包不能逃逸，而只有在函数内使用：

```
public func sort(@noescape isOrderedBefore: 
               (Self.Generator.Element, Self.Generator.Element) -> Bool) 
               -> [Self.Generator.Element]
```

函数前面添加了`@noescape`，是因为这个函数可以确保在排序结束之后就没用了。

#逃逸闭包（Escaping Closures）

既然有非逃逸闭包，自然也需要逃逸闭包，因为在开发中通常需要将所传过来的闭包存储下来，以便在真正需要回调的时候调用。

那么，所谓逃逸闭包就是使函数调用返回后，依然可以使用这个闭包。想想这么一种场景：A界面传过来一个闭包，在B界面执行C操作时，才回调该闭包，以便A可以得到反馈。这就是所谓的反向传值。这里传闭包是在A界面初始化的时候，这时候闭包并不会调用，而是在B界面执行C操作时才真正地调用，因此这时候需要将闭包逃逸，才能在后续可以继续使用。

要使闭包出了函数还可以继续使用，那么我们可以将闭包引用赋值给外部变量。比如，A界面传过来的闭包可以在B界面通过属性引用A所传过来的闭包，那么在B执行C操作时，就可以通过属性引用A所传过来的闭包，然后回调之。

以下只是一个例子，随手写出，不代表运行能通过：

```
let b = BController(callback: {
   // Do what you need to do
} 
self.navigationController?.pushViewController(b, animated: true)

// B界面需要存储下来这个闭包
var callback: (() ->Void)?

init(callback: () ->Void) {
  // 存储下来
  self.callback = callback
}

// 执行C操作
if let block = self.callback {
   block()
}

```

#自动闭包（Autoclosures）

自动闭包是一种自动创建的闭包，用于包装传递给函数作为参数的表达式。这种闭包不接受任何参数，当它被调用的时候，会返回被包装在其中的表达式的值。这种便利语法让你能够用一个普通的表达式来代替显式的闭包，从而省略闭包的花括号。

有声明属性的时候，有一种属性叫自动计算属性，它其实就使用了自动闭包。动闭包让你能够延迟求值，因为代码段不会被执行直到你调用这个闭包。延迟求值对于那些有副作用和代价昂贵的代码来说是很有益处的，因为你能控制代码什么时候执行。

下面这段代码其实就是使用了自动闭包，因为它不需要参数，其实只需要实现体：

```
func find(closure: () ->String) {
  print(closure())
}

// { "找不到啊找不到" }会被编译器自动转化为@autoclosure
find({ "找不到啊找不到" })
```

不过我们也可以手动声明为`@autoclosure`，它主要用在延迟执行代码段。过度使用`@autoclosure`会让你的代码变得难以理解。上下文和函数名应该能够清晰地表明求值是被延迟执行的。

`@autoclosure`特性暗含了`@noescape`特性，这个特性在非逃逸闭包一节中有描述。如果你想让这个闭包可以“逃逸”，则应该使用`@autoclosure(escaping)`特性.

```
// customersInLine is ["Barry", "Daniella"]
// 存储闭包的数组
var customerProviders: [() -> String] = []

// @autoclosure(escaping)来声明自动闭包可逃逸
func collectCustomerProviders(@autoclosure(escaping) customerProvider: () -> String) {
    customerProviders.append(customerProvider)
}

// 只是添加自动闭包到数组中，并没有执行
collectCustomerProviders(customersInLine.removeAtIndex(0))
collectCustomerProviders(customersInLine.removeAtIndex(0))

print("Collected \(customerProviders.count) closures.")
// prints "Collected 2 closures."

// 这个时候才是遍历数组来获取闭包，并执行代码段
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// prints "Now serving Barry!"
// prints "Now serving Daniella!"
```

#写在最后

本篇博文是笔者在学习Swift 2.1的过程中记录下来的，可能有些翻译不到位，还请指出。另外，所有例子都是笔者练习写的，若有不合理之处，还望指出。

学习一门语言最好的方法不是看万遍书，而是动手操作、动手练习。如果大家喜欢，可以关注哦，尽量2-3天整理一篇Swift 2.1的文章。这里所写的是基础知识，如果您已经是大神，还请绕路！

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

#支持并捐助

如果您觉得文章对您很有帮助，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
