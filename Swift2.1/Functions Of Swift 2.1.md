#前言

Swift统一的函数语法足够灵活，可以用来表示任何函数，包括从最简单的没有参数名字的 C 风格函数，到复杂的带局部和外部参数名的Objective-C风格函数。参数可以提供默认值，以简化函数调用。参数也可以既当做传入参数，也当做传出参数，也就是说，一旦函数执行结束，传入的参数值可以被修改。

在 Swift 中，每个函数都有一种类型，包括函数的参数值类型和返回值类型。你可以把函数类型当做任何其他普通变量类型一样处理，这样就可以更简单地把函数当做别的函数的参数，也可以从其他函数中返回函数。函数的定义可以写在在其他函数定义中，这样可以在嵌套函数范围内实现功能封装。

>注明：Swift版本为2.1<br/>
>测试环境：xcode 7.2


#函数定义与调用（Defining and Calling Functions）

函数定义语法为：func 函数名(参数列表) ->返回值：

```
// 函数定义方式：func 函数名称(arg...) ->ReturnValue
func testFunc() ->Void {
  // 无参数，无返回值
}
```

若无返回值，可以直接省略`->Void`：

```
func testArgFunc(arg: Int) {
  // 无返回值可以使用Void，其实是空元组
}
```

我们先来看看官方定义的`Void`是什么：

```
/// The empty tuple type.
///
/// This is the default return type of functions for which no explicit
/// return type is specified.
public typealias Void = ()
```


它其实就是一个空元组。它是作为函数无显示指定返回类型时的默认返回值类型。

参数列表可以使用可变参数：

```
// 可变参数
func calSum(array: Int...) ->Int {
  var sum = 0
  
  for item in array {
    sum += item
  }
  
  return sum
}
```

函数调用就是通过我们的函数名称调用：

```
calSum(1, 2, 3) // 6
calSum(1, 2) // 3
```

swift支持函数重载：

```
// 函数重载
func calSum(array: [Int]) ->Int {
  var sum = 0
  
  for item in array {
    sum += item
  }
  
  return sum
}

// 调用方式与上面一样，只是参数类型不一样
calSum([1, 2, 3])// 6
calSum([1, 4, 5]) // 10
```

函数有多个参数时，使用逗号分开，像：`(arg1: Int, arg2, Int, arg3: Int)`：

```
func multipleFunc(arg1: Int, _ arg2: Int) ->Int {
  return 0
}
```

#多重返回值函数（Functions with Multiple Return Values）

当函数有多个返回值时，我们可以使用元组作为返回值，当然也可以使用数组、字典等，但是使用元组更简单，更明确：

```
func minMax(array: [Int]) -> (min: Int, max: Int) {
    var currentMin = array[0]
    var currentMax = array[0]
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    return (currentMin, currentMax)
}
```

#Optional返回值类型

当函数不一定有返回值时，可以返回可选类型，在类型后面添加个问号即可：

```
func minMax(array: [Int]) -> (min: Int, max: Int)? {
    if array.isEmpty { return nil }
    var currentMin = array[0]
    var currentMax = array[0]
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    return (currentMin, currentMax)
}

// 调用的时候，通过值绑定来判断是否有值
if let bounds = minMax([8, -6, 2, 109, 3, 71]) {
    print("min is \(bounds.min) and max is \(bounds.max)")
}
// prints "min is -6 and max is 109"
```

#指定外部参数名（Specifying External Parameter Names）

我们看看Dictonary的一个API：

```
public mutating func removeAll(keepCapacity keepCapacity: Bool = default)
```

我们可以看到这里一个参数有两个名字：keepCapacity，其中第一个叫外部参数名，第二个叫函数内部变量。

比如,with是外部参数名，用于在调用时显示的；而userId是函数内部变量名，用于内部使用：

```
func find(with userId: Int) ->User? {
  
}

if let user = find(with: 10) {
  // find
}
```

#默认参数值（Default Parameter Values）

默认参数要放在参数列表最后：

```
func someFunction(parameterWithDefault: Int = 12) {
    // function body goes here
    // if no arguments are passed to the function call,
    // value of parameterWithDefault is 12
}
someFunction(6) // parameterWithDefault is 6
someFunction() // parameterWithDefault is 12
```

#可变参数（Variadic Parameters）

可变参数可以接收多个参数：

```
// 可变参数  
// 可变参数接受0个或者多个指定类型的值。可变参数使用...表示  
// 函数最多只能有一个可变参数，并且如果有可变参数，这个可变参数必须出现在参数列表的最后  
// 如果参数列表中有一或多个参数提供默认值，且有可变参数，那么可变参数也要放到所有最后一个  
// 提供默认值的参数之后（先是默认值参数，才能到可变参数）  
func arithmeticMean(numbers: Double...) -> Double {  
  var total = 0.0  
  for number in numbers {  
    total += number  
  }  
  return total / Double(numbers.count)  
}  
  
arithmeticMean(1, 2, 3, ,4 5) // return 3.0  
arithmeticMean(1, 2, 3) // return 2.0  
```

#常量参数和变量参数（Constant and Variable Parameters）

函数参数默认是常量。试图在函数体中更改参数值将会导致编译错误。这意味着你不能错误地更改参数值。

如果要在函数内部修改参数值，那么需要手动声明为变量类型：

```
// 常量和变量参数  
// 在函数参数列表中，如果没有指定参数是常量还是变量，那么默认是let，即常量  
// 这里Str需要在函数体内修改，所以需要指定为变量类型，即用关键字var  
func alignRight(var str: String, count: Int, pad: Character) -> String {  
  let amountToPad = count - countElements(str)  
    
  // 使用_表示忽略，因为这里没有使用到  
  for _ in 1...amountToPad {  
    str = pad + str  
  }  
    
  return str   
} 
```


#输入输出参数（In-Out Parameters）

```
// 输入/输出参数  
// 有时候，在函数体内修改了参数的值，如何能直接修改原始实参呢？就是使用In-Out参数  
// 使用inout关键字声明的变量就是了  
// 下面这种写法，是不是很像C++中的传引用？  
func swap(inout lhs: Int, inout rhs: Int) {  
  let tmp = lhs  
  lhs = rhs  
  rhs = tmp  
}  

// 如何调用呢？调用的调用，对应的实参前面需要添加&这个符号  
// 因为需要修改，所以一定是var类型的  
var first = 3   
var second = 4  
// 这种方式会修改实参的值  
swap(&first, &second) // first = 4, second = 3  
```

#函数类型（Function Types）

```
// 使用函数类型  
// 这里是返回Int类型  
// 函数类型是：(Int, Int) -> Int   
func addTwoInts(first: Int, second: Int) -> Int {  
  return first + second  
}  

// 函数类型是：(Int, Int) -> Int   
func multiplyTwoInts(first: Int, second: Int) -> Int {  
  return first * second  
}  
```

#使用函数类型（Using Function Types）

函数类型也需要类型，也可以用于定义变量，只是它是函数指针类型变量：

```
var mathFunction: (Int, Int) -> Int = addTwoInts  

mathFunction(1, 2) // return 3  

mathFunction = multiplyTwoInts  

mathFunction(1, 2) // return 2  
```

#函数类型作为参数类型（Function Types as Parameter Types）

函数类型也是类型，也可以作为函数的形参：

```
// 函数类型可以作为参数  
func printMathResult(mathFunction: (Int, Int) -> Int, first: Int, second: Int) {  
  println("Result: \(mathFunction(first, second))")  
}  
  
printMathResult(addTwoInts, 3, 5) // prints "Result: 8"  
printMathResult(multiplyTwoInts, 3, 5) // prints "Result: 15"  
```

#函数类型作为返回类型（Function Types as Return Types）

函数类型也可以作为返回值类型：

```
// 函数作为返回类型  
func stepForward(input: Int) -> Int {  
  return input + 1  
}  
  
func stepBackward(intput: Int) -> Int {  
  return input - 1  
}  
  
func chooseStepFunction(backwards: Bool) -> ((Int) -> Int) {  
  return backwards ? stepBackward : stepForward  
}  
  
var currentValue = 3  
let moveNearerToZero = chooseStepFunction(currentValue > 0) // call stepBackward() function  
  
// 参数可以嵌套定义，在C、OC中是不可以嵌套的哦  
func chooseStepFunction(backwards: Bool) -> ((Int) -> Int) {  
  func stepForward(input: Int) -> Int {  
    return input + 1  
  }  
    
  func stepBackward(input: Int) -> Int {  
    return input + 1  
  }  
    
  return backwards ? stepBackward : stepForward   
}  
```

#嵌套函数（Nested Functions）

在Ojbective-C中，函数是不能嵌套定义的，但是在swift中是可以嵌套的：

```
func chooseStepFunction(backwards: Bool) -> (Int) -> Int {
    func stepForward(input: Int) -> Int { return input + 1 }
    func stepBackward(input: Int) -> Int { return input - 1 }
    return backwards ? stepBackward : stepForward
}

var currentValue = -4
let moveNearerToZero = chooseStepFunction(currentValue > 0)
// moveNearerToZero now refers to the nested stepForward() function
while currentValue != 0 {
    print("\(currentValue)... ")
    currentValue = moveNearerToZero(currentValue)
}
print("zero!")
// -4...
// -3...
// -2...
// -1...
// zero!
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