>#[原文出自：标哥的技术博客](http://www.henishuo.com/the-basics-of-swift/)

#前言

Swift是iOS、OS X和WatchOS平台新的开发语言。尽管如此，Swift有很多是与我们使用过的C和Objective-C开发经验是很像的。

Swift提供了自己版本的C和Objective-C基础数据类型，包括整型Int、浮点型Double和Float、Boolean值Bool和字符串类型String。Swift还提供了三个强大的基本集合类型Array、Set、Dictionary。

与C语言一样，Swift使用变量存储和通过唯一标识名获取值。Swift也有值不可变的，称为`constants`，它是比C的常量要强大得多。Swift通过使用常量可使代码更安全且更清晰。

除了我们熟悉的类型外，Swift引入了在Objective-C中没有的高级数据类型，比如元组（Tuples）。元组使我们可以创建和传一组类型相同或者不相同的值。我们可以使用元组作为函数返回值。

Swift还提供了可选类型（Optional Type），用于处理可空值。可行类型表示“它有值，且值为x或者它根本没有值”。使用Optionals与Objective-C中使用指针nil很像，但是Optionals可以是任意类型，而不仅仅是类。Optionals不仅仅是安全，而且比nil指针语意更清晰明了，它们是Swift最强大的特性之一。

Swift是类型安全的语言，这意味着我们所写的代码的类型必须是明确的，不能隐匿转换。比如，如果我们希望是Float类型，却传一个Int类型，那么是不可行，编译器会抱错。

>注明：Swift版本为2.1<br/>
>测试环境：xcode 7.2

#常量与变量

常量与变量都通过一个名称与之关联一个指定类型的值。常量的值一旦设置，不能再修改，而变量是可以随时修改值的。

##常量

声明常量使用关键字`let`来声明，声明时可不明确指定常量的数据类型，交由编译器来推断：

```
// 常量和变量必须在使用前声明，用let来声明常量，用var来声明变量
// 常量在声明时，要求初始化，且不能更改
// 如果是局部常量，就使用驼峰命名规则即可
let blog = "http://www.henishuo.com";

// 如果是全局的常量，个人习惯以小写k开头，后面单词首字母大写
// 本人不习惯看全大写，因此更喜欢这种风格
let kWebsiteTitle = "标哥的技术博客";
```

当然，有的时候我们是不能省略类型说明的，比如Float和Double类型，若我们期望的是Float类型，则必须指定数据类型为Float类型，否则默认编译器就会推断为Double类型：

```
// 指明数据类型
let maxCost: Float = 2000.0

// 最大并发数量设置为全局，则可以这么定义
// 可以指定类型为Int，则指定也没有关系，编译器会推断出来为Int
let kMaxCocurrentCount: Int = 4
```

可能注意到了代码句后面的分号可有可无，通常都不添加。

##变量

声明变量使用关键字`var`来声明。

###类型自动推断

```
// 类型自动推断
// 推出类型为Int <=> var x: Int = 1
var x = 1

// 推断出类型为String <=> var string: String = ...
var string = "编译器自动推断出类型String"
```

###一行声明多变量

```
// 可以一行声明多个变量
// 虽然可以一行声明多个变量，但是建议不要这么做，一行声明一个更可读
var x1 = 0, x2 = 1, x3 = 2

// 连续多个变量声明时，只需要对最后一个变量指定类型，其它几个变量类型也会被声明为指定的类型
var s1, s2, s3: String

// 抱错，类型不一致，s1是String类型
// s1 = 1
s1 = "s1"
print(s1)
```

虽然可以一行声明多个变量，也可以只指定最后一个变量的数据类型，但是不推荐这么做。在开发中一定要清晰，因此最好一行只声明一个变量。

###类型标注

```
// 变量声明时，可以明确指定类型
var xx: Int = 0
```

添加类型标注的方式是很简单的，就是在变量名的后面添加冒号跟着数据类型。比如上面的就是变量名xx后面跟着: Int。为了更美观，通常会在冒号后面添加一个空格。

###分号可有可无

```
// 一般情况下，在swift中分号是可有可无的
var seg = 1;
var seg1 = 1
```

但是，如果一行声明多个变量，像这样就必须添加分号：

```
var seg11 = 1; var seg111 = 2
```

如果不添加分号就会抱错，因为编译器也分不清了。

###整数

```
// Swift 提供了8，16，32和64位的有符号和无符号整数类型。分别为：
// (U)Int8,16,,32,64
//
// 定义有符号整数，基本都是使用Int就可以了
// Int
// 一般来说，你不需要专门指定整数的长度。Swift 提供了一个特殊的整数类型Int，
// 长度与当前平台的原生字长相同：
//
// 在32位平台上，Int和Int32长度相同。
// 在64位平台上，Int和Int64长度相同。
var number: Int = 10
print(Int.max)
print(Int.min)
```

###进制

```
// 一个十进制数，没有前缀
// 一个二进制数，前缀是0b
// 一个八进制数，前缀是0o
// 一个十六进制数，前缀是0x
var decInt = 17
var binInt = 0b10001
var octInt = 0o21
var hexInt = 0x11
```

###类型转换

Swift是强类型语言，不允许隐式转换，因此类型不相同时必须手动转换类型，不能赋值，否则编译器会报错：

```
var intType = 10
var doubleType = 10.0
// 报错，类型不一致，必须手动转换
//intType = doubleType
intType = Int(doubleType)
```

###类型别名

使用类型别名，通过关键字`typealias`来声明，格式为：typealias 类型别名 = 原类型名。使用类型别名的好处是使语义更明确。

```
typealias IntType = Int

// IntType其实就是Int
let valueObject: IntType = 10

print(valueObject)
```

###Bool值

Swift中的真假值用true、false表示，与objective-c中的YES、NO表示的意思一样的。

```
var isDoing: Bool = false

var isSuccessful = true
```

###Unicode字符变量

我们可以使用中文命名，也可以使用表情符号命名，但是真正开发中是不允许的，因为这会引来很多麻烦。再说，这么多年来都习惯了英文命名，用中文命名会让很多人无法接受的，而且也不好输出。

```
var 你好 = "你好世界"
var 🐶🐮 = "dogcow"
```

###打印

首先打印使用print函数，字符串中使用***\\(变量名)***：

```
var name = "标哥"
print("大家都叫我：\(name)")
```

###注释

在Swift中，对于单行功能注释通常使用`//`，对于公开的属性或者变量添加的注释通常使用`///`，通过代码段功能注释，通常使用`/* */`:

```
/// 注释风格一
// 注释风格二
/* 注释风格三 /*在swift中可以嵌套注释*/ */
var comment = "注释"
```

###元组

元组是使用圆括号来声明，元组中的元素的数据类型可以是任意的，不要求相同。访问元组的值，可以通过序列0,1,2...，也可以在声明时指定元组各个元素的名称，然后通过名称来获取值：

```
// swift中增加了元组类型，元组内的值可以是任意类型，并不要求是相同类型。
// 声明方式，直接用圆括号
let http404NotFound = (404, "Not Found")
// 可以用.0,1,2...访问值
print(http404NotFound.0)
print(http404NotFound.1)

// 如果只要其中一部分值，可以用_过滤
let (httpCode, _) = http404NotFound
print(httpCode)

// 可以给元组中的每个元素全名
let httpStatus = (code: 404, description: "Not Found")
print(httpStatus.code)
print(httpStatus.description)
```

当我们不需要接收某些值时，我们可以通过下划线`_`来过滤。这种用法是很常见的，使用非常广泛。

###Optionals

对于可选类型，官方的说明是："There is a value, and it equals x"或者"There isn't a value at all"。也就是说，它要么有值x，有么没有值。

使用`?`声明为可选类型，当它没有值是，这就是nil。只有声明为可选类型，值才可以设置为nil。

```
// 可选类型
// 用？表示可选类型，通过用可选绑定判断是否有值
var optionalValue: Int?
```

####If语句和强制拆包

我们可以使用`==`或者`!=`来判断：

```
var convertedNumber: Int? = 123

if convertedNumber != nil {
   print("value is \(convertedNumber!)")
}
```

####Optional Binding

通过`if let value = optionalValue {}`的方式来绑定值：

```
if let value = optionalValue {
  print("有值的")
} else {
  print("是空的")
}

optionalValue = 10
if let value = optionalValue {
  print("有值的")
} else {
  print("是空的")
}
```

####多个Optional Binding

如果有多个，则只需要一个let声明即可，当然也可以分别写，每个之间使用逗号来分开。其结果为，只有所有都是有值的，才会为true，否则为false：

```
var op1: Int?
var op2: Int?
op1 = 1
// 对于多个可选，只有都有值才会执行{}内的内容
if let lhs = op1, rhs = op2 {
  print("都有值")
} else {
  print("不都有值")
}

op2 = 2
if let lhs = op1, rhs = op2 {
  print("都有值")
} else {
  print("不都有值")
}
```

####Implicitly Unwrapped Optionals

隐式解析可选类型值，需要在声明是使用`!`而不是`?`，这是告诉编译器，一旦赋值，它都会有值，但是若在获取值时，它却没有值，则会崩溃。

```
// 如果变量一开始没有值，但是一旦赋值后，保证一直有值，则可以用!来声明
var name: String! // 后面会有值
name = "我会有值的"
print(name)// 使用时不用加!

// 可以使用！，也可以不使用，但是一定要保证有值，否则会crash
var lili: String = name!
print(lili)
```

###Error Handling

swift中提供了异常处理机制，使用throws抛出异常，交由外部处理。如下：

```
enum LessThanErrorType : ErrorType {
  case LessThanZero
}

func throwErrorFunc(number: Int) throws {
  if number <= 0 {
    throw LessThanErrorType.LessThanZero
  }
  
  print("number = \(number)")
}
```

外部就可以通过下面的方式来调用并捕获异常：

```
do {
  try throwErrorFunc(-1)
} catch {
   
}
```

如果要细分各种异常处理，则可以使用多个catch：

```
do {
  try throwErrorFunc(-1)
} catch LessThanErrorType.LessThanZero {

} 
// 若前一个不符合条件，则会进入这个捕获所有类型的异常处理
catch {

}
```

###断言（Assertions）

断言在开发中有一定的作用的，首先我们在代码调试的时候，可以添加断言来处理数据是否满足要求，比如必传字段出现空，则可以通过断点将不满足的条件抛出来。

```
// 断言通常是用于诊断条件是否满足，如果不能满足就会中断程序
var optoinalValue2 = "我不是空的"
assert(optoinalValue2.isEmpty == false)// 条件为真，跟什么都没做一样

// assertion failed: : file <EXPR>, line 86
assert(optoinalValue2.isEmpty == true)

// 我们也可以传失败信息
// assertion failed: optoinalValue2不为空: file <EXPR>, line 89
assert(optoinalValue2.isEmpty == true, "optoinalValue2不为空")

// 还提供了直接断言失败的API：
// fatal error: : file <EXPR>, line 92
assertionFailure()

// 也支持断言失败提示信息：
// fatal error: 反正是失败了: file <EXPR>, line 96
assertionFailure("反正是失败了")
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
