>#Swift可选链

欢迎一起来学习**Swift可选链**章节的知识。在学习中遇到任何问题，请加QQ群：**324400294**

#介绍
可选链，英文叫`Optional Chaining`，是表示变量、属性等值可为空，也就是值可能为`nil`。若有值，则可成功获取值；若无值，则返回`nil`。多个连续的调用可以被链接在一起形成一个调用链，如果其中任何一个节点为空（nil）将导致整个链调用失败。

在开发中会大量地使用可选类型属性、变量等，因为在语义上表示值可能有也可能没有。善于使用`Optional`，可简化开发。

>注意： `Swift`的可选链调用和`ObjectiveC`中的消息为空有点像，但是`Swift`可以使用在任意类型中，并且能够检查调用是否成功。
>

#Optional枚举

下面是`Optional`检举类型的声明：

```
public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
    case None
    case Some(Wrapped)
    /// Construct a `nil` instance.
    public init()
    /// Construct a non-`nil` instance that stores `some`.
    public init(_ some: Wrapped)
    /// If `self == nil`, returns `nil`.  Otherwise, returns `f(self!)`.
    @warn_unused_result
    public func map<U>(@noescape f: (Wrapped) throws -> U) rethrows -> U?
    /// Returns `nil` if `self` is nil, `f(self!)` otherwise.
    @warn_unused_result
    public func flatMap<U>(@noescape f: (Wrapped) throws -> U?) rethrows -> U?
    /// Create an instance initialized with `nil`.
    public init(nilLiteral: ())
}
```

其中无值用表示`None`表示，而有值用`Some`表示有值。

```
var count: Optional<Int>
```
这么声明与下面的声明是一样的：

```
var count: Int?
```

#如何声明Optional
声明为可选，可以使用`?`或者`!`，如果使用`?`，其值为空时，调用也没有关系，只是什么也没有做而已。但是，如果使用`!`声明，如果其值为空，我们调用就会造成`crash`。对于`!`表示选择编译器，其一定有值。

```
var optionalValue: String?

// Optional("nonnull")
print(optionalValue)

var name: String!

// Crash原因：取值是，发现是nil，而声明是就告诉过编译器，不可能为空
// fatal error: unexpectedly found nil while unwrapping an Optional value
print(name)
```

#使用!强制UnWrapper
如果我们在使用中，需要`unwrapper`呢？如果我们直接使用`?`来获取，也可以直接使用`!`强制获取。

```
class Person {
    var residence: Residence?
}

class Residence {
    var numberOfRooms = 1
}
```

我们声明一个`Person`对象：

```
let john = Person()
let residence = Residence()
biaoge.residence = residence
```

获取`numberOfRooms`的值时，我们可以直接使用`!`取，这样取出来的值就不是可选的。像下面这样，那么`roomCount`的类型为`Int`：

```
let roomCount = john.residence!.numberOfRooms
```

当然我们也可以使用`?`获取，则这样获取的话，`roomCount`的类型就是`Int?`

```
let roomCount = john.residence?.numberOfRooms
```

对于可选类型，我们通常使用下面的方式：

```
if let roomCount = john.residence?.numberOfRooms {
    print("John's residence has \(roomCount) room(s).")
} else {
    print("Unable to retrieve the number of rooms.")
}
```
这种方式是非常常用的，在开发中也会到处看到类似这样的代码。尝试去获取值，如果成功地取得值，就会是`true`，否则就会是`false`。

#可选赋值

对于上面的`john.residence?.numberOfRooms`，我们可以设置其值：

```
john.residence?.numberOfRooms = 10
```
由于`residence`是可选类型，我们在使用时，添加上`?`，就算`residence`是空的，这么设置也不会有问题。当然我们也可以使用`!`来告诉编译器，`residence`一定不会为空，但是一旦其值为空就会Crash。所以，无论何时，我们直接使用`?`就好了，不用使用`!`。

如果有很多个可空类型，一样都使用`?`就好了。类似下面这样：

```
// 只是假设
let testId = john?.residence?.test?.id
```

#访问下标
这里我们设计一个可空数组，然后通过访问下标的方式获取值。由于`array`是可空数组，不能直接访问，因此需要通过`?`或者`!`访问。

```
let array: [Int]? = [1, 2]
// 如果这里直接使用array[0]来抱错的
let first = array?[0]
```

当然我们也可以直接使用`!`的方式来访问，因为这里我们确定`array[0]`一定不会越界。下面这么写也是正确的。

```
let fiset = array![0]
```

#可选参数和返回值

在设计函数的时候，我们经常是不确定其一定有值的，因此我们会返回可空类型。

```
func funcName(name: String?) ->String? {
  return nil
}
```

#学习路线
关于学习`Swift`的路径说明，请阅读文章：[http://mp.weixin.qq.com/s?__biz=MzIzMzA4NjA5Mw==&mid=400386772&idx=1&sn=15328f06c1d3dd3a964d24cf1aad5c4d#wechat_redirect](http://mp.weixin.qq.com/s?__biz=MzIzMzA4NjA5Mw==&mid=400386772&idx=1&sn=15328f06c1d3dd3a964d24cf1aad5c4d#wechat_redirect)

#关注我
>**公众号搜索「iOS开发技术分享」快速关注微信号：iOSDevShares**
>**QQ群：324400294**

