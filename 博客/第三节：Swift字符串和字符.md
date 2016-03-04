>#Swift字符串和字符

欢迎一起来学习**Swift字符串和字符**章节的知识。在学习中遇到任何问题，请加QQ群：**324400294**

#介绍

`String`类型是一种快速、现代化的字符串实现，每一个字符串都是由编码无关的`Unicode`字符组成，并支持访问字符的多种`Unicode`表示形式，如：UTF8、UTF16等。字符串的内容都是由一个个的`Character`组成。

在开发中，对字符串、字符常用的操作有：

* 初始化空字符串
* 使用可变、不可变字符串
* 字符串是值类型
* 与`NSString`互转
* 计算字符数量
* 比较字符串
* 连接字符串和字符
* 获取、删除、添加、修改字符/字符串
* 分割字符串



>注意：
Swift的String类型与Foundation NSString类进行了无缝桥接，因此这两者可以直接互转，然后可调用对应的类型的所有API。

#初始化空串

初始化空字符串有三种方式，分别为：

* 直接使用双引号
* 使用`String`结构体的无参构造函数
* 使用`String`结构体的有参构造函数，但参数传双引号

```
var string = "" // 空字符串
string = String() // 空字符串
string = String("")// 空字符串
```

字符`Character`最常用的构造函数：

```
/// 根据一个只有一个字符的字符串创建一个`Character`
///
/// 要求参数s字符串中只能包含一个字符
public init(_ s: String)
```

对于初始化字符常用的方式：

```
// 对于构造函数：public init(_ s: String)
// 我们必须保证所传的参数`s`字符串中最多包含一个字符，否则会`Crash`掉
var c = Character("1")// OK
var cc = Character("10") // Factal error

// 上面使用构造的方法与下面这种直接赋值的方式是一样的，实际上也是通过转换成构造函数`public init(_ s: String)`来初始化
var c: Character = "A"

```

#可变、不可变字符串
对于`Swift`中的`String`字符串，若要声明不可变字符串，使用`let`关键字声明为常量字符串即可；若是要让字符串可变，使用`var`关键字声明就可以了。与`OjbectiveC`不一样，使用`NSString`声明为不可变字符串，使用`NSMutableString`声明为可变字符串。相比之下，自然是`Swift`的`String`结构体使用起来更简单。

对于不可变字符串，是不能执行增、删、改操作的，因此下面会报错：
Cannot use mutating member on immutable value: 'str' is a 'let' constant。意思是：`str`是用`let`声明的常量，不能对不可变值使用可变成员(方法、属性)。

```
let str = ""
str.append(Character("1"))
```

当我们修改为`var`声明字符串为变量`str`，就可以直接添加字符了：

```
var str = ""
str.append(Character("1")) // Ok 
```

对于字符`Character`与`String`一样，使用`let`声明为常量就不能再修改其值，使用`var`声明为变量就可以在运行时修改其值。

```
var cc = "A"
cc = "b"// Ok

let c = "C"
c = "c" // Factal error
```

#字符串是值类型
注意，`String`类型是结构体，因此它是值类型，而不是引用类型。所有在使用字符串的时候，不会有引用计数的说法了，在传值时，会进行值拷贝而不是引用。

```
string = "值拷贝"
func testStringCopy(var str: String) {
  str = "这里新值"
  print(str)
}
testStringCopy(string) // 这里新值
print(string) // 值拷贝
```

打印结果如下，说明这是值拷贝，外部的字符串变量`string`不会被函数内部的字符串变量`str`所修改：

```
这里新值
值拷贝
```

#与`NSString`互转
`Swift`中的`String`与`Foundation`中的`NSString`进行了无缝桥接，因此我们在使用中可以直接让这两种类型直接互转，不会出现失败。

```
var ss = "String中并不包含所有的NSString的方法"
var nss = ss as NSString // OK

var str = nss as String // OK
```

但是，我们不能直接将`String`转换成`NSMutableString`。由于`String`与`NSString`是无缝桥接的，因此转换必定成功，直接使用`as`转换即可。

>提示：类型转换使用关键字`as`，如果转换有可能失败，则需要使用`as?`。如果认为一定不会失败，我们可以使用强制转换`as!`，但是一旦失败就会崩溃。


#计算字符数量
计算字符串的字符的个数与`NSString`的不一样，没有`length`方法。在`String`中，对字符的操作，主要是通过其属性`characters`来操作。

```
let str = "计算字符串de个数"
print(str.characters.count) // 9
```

当前我们也可以通过直接转换成`NSString`再获取其长度：

```
let name = str as NSString
print(name.length) // 9
```

在实际开发中，本人不希望这样类型转换再操作，因此，我会扩展`String`，添加`length`方法：

```
extension String {
  func hyb_length() ->Int {
    return self.characters.count
  }
}
```

这样我们就可以直接使用：

```
print(str.hyb_length) // 9
```

> 注意：看到扩展的方法前面为什么要添加前缀呢？考虑到多人开发时，难免会冲突，因此使用前缀可以很好的解决冲突问题。比如，我们公司6个组同时对同一个App进行开发，只是各自分处不同的业务线，因此大家都会添加一些常用的扩展API，而且大家都会导入第三方库，经常出现同名的API，因此很容易造成冲突。前缀一般都使用`项目简写或者公司简写名`再加一个下划线。


#比较字符串
对于字符串`String`的比较，与`NSString`相比，简单了很多，因此`Swift`支持重载函数和符号函数，因此，就可以直接定义`==`、`!=`函数。判断是否相等，使用两个等号`==`，不相等就用`!=`就可以了。

```
if str != "谢谢" {
  print("不相等")
}

if "谢谢" == "谢谢" {
  print("肯定相等")
}
```

通常我们还会对字符串的前缀、后缀比较：

```
if str.hasPrefix("字符串") {
  print("有...")
}

if str.hasSuffix("连接") {
  print("有...")
}
```

还可以直接判断是否包含某个子串：

```
print(str.containsString("符"))// true
```

判断是否为空串，可以使用`==`判断，也可以直接获取其属性`isEmpty`，注意它不是一个函数:

```
print(str.isEmpty)// false

print(str == "") // false
```

虽然两者都有两样的效果，但是推荐使用`isEmpty`，可读性更强一些。

#连接字符串和字符

在`Swift`中，对`String`字符串类型操作，比`NSString`要简单多了，拼接字符串，可直接使用`+=`符号函数，也可以使用`\(变量名)`这种方式插入，还可以使用`append`方法添加字符。

```
var str = "字符串连接"

// str=字符串连接值拷贝,string=值拷贝
str += string 

// 使用\(变量、常量、函数返回值等)
str = "\(str)\(string)"

// 注意，由于"A"这么写编译器无法识别是字符还是字符串，
// 且append只能追加一个字符，不能追加字符串
//str.append("A")// 提示：mutating func append(c: Character)
str.append(Character("A"))// 传类型强转换就可以了

// 如果是追加字符串，可以使用`write`方法
str.write("顶起")
print(str)
```

#获取、删除、添加、修改字符/字符串
对于删除、添加、修改字符串，只是对可变字符串有效。

添加字符串的与上面的**`连接字符串和字符`**是一样的。关于`write`方法和`append`方法有多个函数，自己再深入学习。当然还有`insert`方法、`appendContentsOf`等

删除字符：

* 清空字符串可以使用`removeAll`等方法，也可以直接使用双引号：

```
// 传true参数，表示内容清除，但容量不变
str.removeAll(keepCapacity: true)
// 使用默认值，容量会清掉
str.removeAll()

// 也可以使用这样：
str = ""

// 还可以直接使用删除范围函数：
str.removeRange(Range(start: str.startIndex, end: str.endIndex))
```

* 删除部分字符：

```
// 删除第一个字符
str.removeAtIndex(str.startIndex)

// 删除第一个字符
str.characters.removeFirst()

// 删除最后一个字符
str.characters.removeLast()

// 删除某个函数
str.removeRange(Range(start: str.startIndex, end: str.endIndex))
```

* 修改字符串:

我们主要是使用`replaceRange`方法：

```
let range = str.rangeOfString("顶起")
str.replaceRange(range!, with: "被替换的值")
print(str)
```

* 获取字符串、字符

通过循环的方式来遍历：

```
for c in str.characters {
  print(c)
}
```

如果在遍历中需要知道其索引和值，可以通过快速枚举：

```
for (index, c) in str.characters.enumerate() {
  print("c = \(c), index= \(index)")
}
```

下面这种方式是非常少用的，因为非常不好用：

```
for var index = 0; index < str.characters.count; ++index {
   // 不能直接使用str[index]获取，也没有str.characters[index]
}
```

#分割字符串

最常用的方法就是`componentsSeparatedByString`，会返回一个数组：

```
str.componentsSeparatedByString("被")
```

#学习路线
关于学习`Swift`的路线说明，请阅读文章：[http://mp.weixin.qq.com/s?__biz=MzIzMzA4NjA5Mw==&mid=400386772&idx=1&sn=15328f06c1d3dd3a964d24cf1aad5c4d#wechat_redirect](http://mp.weixin.qq.com/s?__biz=MzIzMzA4NjA5Mw==&mid=400386772&idx=1&sn=15328f06c1d3dd3a964d24cf1aad5c4d#wechat_redirect)

请从第一节开始阅读！

#关注我
**公众号搜索「iOS开发技术分享」快速关注微信号：iOSDevShares**

**QQ群：324400294**

