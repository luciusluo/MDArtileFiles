#前言

欢迎一起来学习**Swift基础部分**章节的知识。在学习中遇到任何问题，请加QQ群：324400294

#常量、变量

使用`let`关键字来声明常量，声明时需要给定确定的值，且不可再修改值类型，也不能修改值，否则会在编译前提示错误。
 
```
let constant: Int = 10
// 这里修改值就会报错，常量不能修改其值
constant = 11
```

使用`var`关键字来声明变量，在实际开发中，声明时最好给一个初始值。声明后，我们仍然可以修改其值。

```
var variable: Int = 10
// 这是没有问题的，变量是可以随时修改其值的
variable = 11
```

**温馨提示：**声明变量和变量的方式是: `var/let name: Type`来声明，可以指定初始值的方式：`var/let name: Type = initValue`

#类型推断

`Swift`是推荐类型推断的，也就是不需要我们在声明时指定具体的类型，只要赋值就可以根据值来自动推断出其数据类型。像上面的`constant`，我们可以不指定类型为`Int`，直接使用`let constant = 10`也是一样的，会根据数值`10`推断出其类型为`Int`。

注意，不是所有情况下都可以使用类型推断的，一定是我们知道已经知道其具体类型且我们希望是这种类型的情况下才使用类型推断。例如，如果我们想要声明一个`Float`类型的变量，而我们声明时这样：

```
let floatValue = 10.0
```

`floatValue`的类型是什么类型呢？是不是真的是`Float`类型？不是的，实际上是`Double`类型。因此，像这种情况下，我们就不能通过类型推断来推断具体类型了。需要加上类型声明：

```
let floatValue: Float = 10.0
// 或者这样
let floatValue = Float(10.0)
```

**温馨提示：**在开发中，我们会大量地使用类型推断，因为更简洁，且更容易阅读。

#类型转换

`Swift`是强类型语言，比`ObjectiveC`还要强，对于类型转换，永远不会隐式转换，因此在需要类型转换的地方，我们必须手动添加代码转换。

```
var doubleValue = 10.0

// 提示Double不能转换成Int类型，会抱错
//var intValue: Int = doubleValue

// 可通过使用构造方法来构造转换类型
var intValue = Int(doubleValue)
```

例子中，声明的`doubleValue`为`Double`类型，而声明的`intValue`为`Int`类型，要将一个`Double`类型的值赋值给一个`Int`类型的变量是不可行的，不会隐式发生类型转换。在`ObjectiveC`中，会进行隐式转换，可是`Swift`的强类型检查在编译阶段就会检查出来，然后抱错，要求我们必须手动处理转换问题。对于普通类型，我们可以使用其构造方法来转换。对于特殊类型，我们就要特殊处理了。

#类型别名

使用`typealias`关键字给一个已经存在的类型指定别名。指定别名后，就可以使用该别名来声明常量或者变量。

```
typealias AudioSample = Int
```		

对于这里，我们给`Int`类型起了一个别名（一个更有意义的名字）`AudoSample`，那么我们就可以直接使用`AudioSample`来声明变量或者常量了。

```
// 相当于let audioRate: Int = 1
let audioRate: AudioSample = 1

// 相当于var audioVoice: Int = 10
var audioVoice: AudioSample = 10
```

**温馨提示：**声明类型别名的方式为： 
 
`typealias` 类型别名 = 已存在类型名

#布尔类型

在`Swift`中布尔类型是使用`Bool`，其值只有真与假，也就是`true`或者`false`。
注意，不再是`YES`或者`NO`。

```
let boolValue: Bool = true
let isSuccess = false
```

**提示：**在开发中，给布尔类型命名时，尽量使用语意比较清晰的名称。

推荐使用`is`、`should`、`can`等开头。如：`isSuccessful`、`shouldUpdateWhenFinished`、`canTransfer`等，表示是否成功、是否应该在完成时自动更新、能否执行转换。

#元组类型

之前在`ObjectiveC`中，是没有元组的，但是在`Swift`中有了元组类型。声明元组的方式很简单，圆括号包起来，且元素的类型可以是任意类型。元组内的元素，可以通过.加上0，1，2...来访问元素，如果有名称，可以直接使用名称获取值。

```
let httpError = (404, "Http Not Found!")

// 也可以给元素指定名称
let httpError1 = (errorCode: 404, errorMessage: "Http Not Found")

// 可以通过.加上0，1，2...来访问元素，
// 如果有名称，可以使用名称获取值
httpError.0 // 404
httpError1.errorCode // 404
httpError.1 // Http Not Found
httpError1.errorMessage // Http Not Found
httpError1.1
```

**温馨提示：**对于元组，使用场景比较多的是在函数返回值需要返回多个值但使用数组又不是很好时，会优先考虑使用元组。在表示一个名称有多种说法时，我们会使用元组，如上面的`httpError`，当然也可以使用字典来表示，但是不如元组更简单明了。

#可选链

对于声明可选类型可以用`?`或者`!`,对于这一知识是比较多的，对于刚学习`Swift`的同志们，接受这一思想似乎有些困难，因此这里只指出，但不细说。具体会在下一章节详细说明。

#断言
`Swift`中的断言与`ObjectiveC`中的断言有很多不一样的地方，使用起来差不多。

**什么是断言？**

断言通常是用于诊断条件是否满足，如果不能满足就会中断程序。使用`assert`函数来添加断言。

如果条件为真，那么就什么也不做，可以继续往下走。如果条件为假，那么就会抛出crash信息，程序中断退出。

```
var valueId = ""

// 这里会crash，因为条件为假
// 崩溃信息为：assertion failed: : file <EXPR>, line 86
assert(valueId.isEmpty == false)
```

`assert`函数的方法为：

```
public func assert(@autoclosure condition: () -> Bool, @autoclosure _ message: () -> String = default, file: StaticString = default, line: UInt = default)
```
对于此方法，我们有很多个缺省值的参数，因此我们也可以传我们自定义的参数。

```
// assertion failed: optoinalValue2不为空: file <EXPR>, line 89
assert(optoinalValue2.isEmpty == true, "optoinalValue2不为空")
```

`Swift`还提供了直接断言失败的API：

```
// fatal error: : file <EXPR>, line 92
assertionFailure()
```

当前这个断言失败的API也可以传参数，指定错误信息。指定特定的错误信息，目的是让我们开发人员更精准地定位到出错的问题所在。

```
// fatal error: 反正是失败了: file <EXPR>, line 96
assertionFailure("反正是失败了")
```

#异常处理
对于异常处理，后面还有一章节专门详解，这里只是简单过一下。
在函数返回值之前参数之后添加关键字`throws`就可以将异常抛出来，让外部捕获处理。外部调用时可以使用`do { try ...} catch { ... }`来处理。调用方式为：`try 方法名()`。

```
func canThrowAnErrow() throws ->Bool {
  // 这个函数有可能抛出错误
  return true
}

// 在调用的地方使用do{try ...} catch {}来处理异常
do {
  try canThrowAnErrow()
  // 继续做其它事...
} catch {
  // 可以分别捕获不同的异常处理，这里直接捕获所有异常
}
```

**提示：**如果确定不会抛异常，我们可以直接使用`try! 方法名()`来调用，但是一旦抛出异常，就会直接`crash`掉，因此要慎用`try!`，尽量使用`try`来处理异常。

#最后

本篇文章是挺久以前写的了，今天只是更新一下文章的格式，如果文中出现比较旧的知识，大家可以在评论中指出。毕竟Swift更新太快了，笔者现在也没有及时地跟进！
