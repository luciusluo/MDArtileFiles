>#[原文出自：标哥的技术博客](http://www.henishuo.com/control-flow-of-swift/)

#前言

Swift提供了类似C语言的流程控制结构，包括可以多次执行任务的for和while循环。还有基于特定条件选择执行不同代码分支的if、guard和switch语句，还有控制流程跳转到其他代码的break和continue语句。

Swift增加了for-in循环，用来更简单地遍历数组、字典、区间、字符串和其他序列类型。

Swift的switch语句比C语言中更加强大。在C语言中，如果某个case不小心漏写了break，这个case就会贯穿至下一个case，而Swift无需写break，所以不会发生这种贯穿的情况。case 还可以匹配更多的类型模式，包括区间匹配（range matching）、元组（tuple）和特定类型的描述。switch的case语句中匹配的值可以是由case体内部临时的常量或者变量决定，也可以由where分句描述更复杂的匹配条件。

#for循环（For Loops Statement）

* for：与C语言一样的for循环
* for-in：快速遍历集合、序列等

for-in遍历range（其中...表示闭区间[1,5]）：

```
for index in 1...5 {
    print("\(index) times 5 is \(index * 5)")
}
// 1 times 5 is 5
// 2 times 5 is 10
// 3 times 5 is 15
// 4 times 5 is 20
// 5 times 5 is 25

// 它可以转换for循环为：
for var index = 1; index <= 5; ++index {
   // ...
}
```

若我们不要获取值，可以用下划线（_）过滤，这种用法很常见：

```
let base = 3
let power = 10
var answer = 1

// 三个点表示全闭区间[1, power]
for _ in 1...power {
    answer *= base
}

// 两个点加一个<就是左闭右开[1, 5)
var sum = 0
for _ in 1..<5 {
  sum += 1
}
```

常见的遍历数组方法：

```
let names = ["Anna", "Alex", "Brian", "Jack"]
for name in names {
    print("Hello, \(name)!")
}

// or 
for (name, index) in names.enumerate() {
   // 也是很常用的
}
```

常见的遍历字典：

```
let numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
for (animalName, legCount) in numberOfLegs {
    print("\(animalName)s have \(legCount) legs")
}

// 我们知道字典中的键值对是用元组表示的，所以可以直接通过元组来访问
```

#while循环（While Loop Statement）

* while循环，每次在循环开始时计算条件是否符合；
* repeat-while循环，每次在循环结束时计算条件是否符合。用法与Objective-C中的do-while完全一样。

注意：没有do-while而是repeat-while

```
var index = 10
while index > 0 {
  index--
}
print(index)// 0

index = 0
repeat {// 没有do-while了，只有repeat-while
  print("test repeat")
  index++
} while index < 3
```

#if条件语句（If Condition Statement）

If条件为真假值，要么为true，要么为false。

```
// 条件只有index值为3，结果才为true，才会打印。
if index == 3 {
  print("3")
}
```

#guide语句（Guide Condition Statement）

`guide`语法与if不同，如果条件为真，则不进入else分支，否则进入。`guide`语义是守卫的意思，也就是说，只要满足条件，什么事都没有，否则就会进入else分支。

```
// 在函数内部，判断必传参数为空时，直接退出函数，这种用法很常用。
guard let name = dict["name"] else {
	return
}
```

#switch语句（Switch Statement）

swift中的Switch分支与Objective-C中的switch有很多不同的地方：

* swift中不需要为每个case手动写break
* swift中case支持区间匹配
* swift中的case支持元组
* swift中的case支持值绑定
* swift中的case支持where条件过滤
* swift中的case可以放置多个值

不用手写break，也不会隐式贯穿：

```
var value = 1
switch value {
case 1:
  print("only print 1")
case 2:
  print("only print 2")
default:
  print("on matched value")
}
// "only print 1\n"
```

支持匹配区间：

```
let approximateCount = 62
let countedThings = "moons orbiting Saturn"
var naturalCount: String
switch approximateCount {
case 0:
    naturalCount = "no"
case 1..<5:
    naturalCount = "a few"
case 5..<12:
    naturalCount = "several"
case 12..<100:
    naturalCount = "dozens of"
case 100..<1000:
    naturalCount = "hundreds of"
default:
    naturalCount = "many"
}
print("There are \(naturalCount) \(countedThings).")
// 输出 "There are dozens of moons orbiting Saturn."
```

支持元组，对于不需要获取的值，可以用`_`过滤：

```
let httpError = (404, "Http Not Found")
switch httpError {
case let (code, _) where code == 404:
  print(httpError.1)
case let (code, msg) where code == 502:
  print(msg)
  
default:
  print("default")
}
```

支持值绑定：

```
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
// 输出 "on the x-axis with an x value of 2"
```

支持where过滤条件：

```
let point = (1, 3)
switch point {
case let (x, y) where x > y:
  print("x > y")
case let (x, _) where x > 2:
  print("x > 2")
case let (1, y) where y > 4:
  print("y > 4 and x == 1")
case let (x, y) where x >= 1 && y <= 10:
  print("ok")// ok
default:
  print("error")
}
```

支持一个case多个值：

```
let numberSymbol: Character = "三"  // 简体中文里的数字 3
var possibleIntegerValue: Int?
switch numberSymbol {
case "1", "١", "一", "๑":
    possibleIntegerValue = 1
case "2", "٢", "二", "๒":
    possibleIntegerValue = 2
case "3", "٣", "三", "๓":
    possibleIntegerValue = 3
case "4", "٤", "四", "๔":
    possibleIntegerValue = 4
default:
    break
}
```

#控制转移语句（Control Transfer Statements）

swift有五种控制转移语句：

* continue：跳过本次循环，直接进入下一循环
* break：中断最近的循环或者中断某个标签（下一小节说明）
* fallthrough：用于switch分支贯穿分支
* return：用于函数返回
* throw：用于抛出异常

用`continue`跳过不满足条件的：

```
let puzzleInput = "great minds think alike"
var puzzleOutput = ""
for character in puzzleInput.characters {
    switch character {
    case "a", "e", "i", "o", "u", " ":
        continue
    default:
        puzzleOutput.append(character)
    }
}
print(puzzleOutput)
// 输出 "grtmndsthnklk"
```

用break退出循环：

```
for index in 1...5 {
  if index >= 3 {
    break
  }
}

// 在swift中用break，就会直接退出该swift语句
index = 10
for index in 20..<100 {
	switch index {
		case let x where x < 40:
		print(x)
	case let x where x > 100:
		break
	default:
		break
}
```

用fallthrough贯穿swift的case：

```
let integerToDescribe = 5
var description = "The number \(integerToDescribe) is"
switch integerToDescribe {
case 2, 3, 5, 7, 11, 13, 17, 19:
    description += " a prime number, and also"
    fallthrough
default:
    description += " an integer."
}
print(description)
// 输出 "The number 5 is a prime number, and also an integer."
```

#标签语句

比如有时候需要在满足某个条件的时候就跳去执行某段代码，那么这时候用标签语句就很好用：

语法如下：

```
label name: while condition {
    statements
}
```

官方的一个例子：

```
gameLoop: while square != finalSquare {
    if ++diceRoll == 7 { diceRoll = 1 }
    switch square + diceRoll {
    case finalSquare:
        // 到达最后一个方块，游戏结束
        break gameLoop
    case let newSquare where newSquare > finalSquare:
        // 超出最后一个方块，再掷一次骰子
        continue gameLoop
    default:
        // 本次移动有效
        square += diceRoll
        square += board[square]
    }
}
print("Game over!")
```

#检查API可用性

语法如下：

```
if #available(iOS 9, OSX 10.10, *) {
    // 在 iOS 使用 iOS 9 的 API, 在 OS X 使用 OS X v10.10 的 API
} else {
    // 使用先前版本的 iOS 和 OS X 的 API
}
```

详细如何使用，请阅读文章：[Swift检测API可用性](http://www.henishuo.com/swift-api-available/)

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


