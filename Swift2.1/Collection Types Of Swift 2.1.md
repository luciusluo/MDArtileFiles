>#[原文出自：标哥的技术博客](http://www.henishuo.com/collection-types-of-swift/)

#前言

Swift语言提供Array、Set和Dictionary三种基本的集合类型用来存储集合数据。数组是有序的数据集；集合是无序无重复的数据集；而字典是无序的键值对数组集。

![image](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Art/CollectionTypes_intro_2x.png)

Swift的Array、Set和Dictionary类型被实现为泛型集合。因此，它所存储的元素的类型必须是一致的，同样，我们取出来的数据类型也是明确的。

#集合的可变性（Mutability Of Collections）

如果创建一个Arrays、Sets或Dictionaries并且把它分配成一个变量，这个集合将会是可变的。这意味着我们可以在创建之后添加更多或移除已存在的数据项，或者改变集合中的数据项。

如果我们把Arrays、Sets或Dictionaries分配成常量，那么它就是不可变的，它的大小和内容都不能被改变。

若我们确定集合不需要改变，那么我们应该使用不可变集合，也就是使用`let`声明，那样编译器会为我们优化。

```
// 在swift中要让Array、Set、Dictionary类型的变量是可变的，用var声明就可以了。
// 如果不希望可变，用let声明即可。
let immutableArrray = [Int]()

// 提示：error: immutable value of type '[Int]' only has mutating
// members named 'append'
//immutableArrray.append(12)

// OK
var mutableArray = [Int]()
mutableArray.append(12)
```

#数组（Arrays）

数组使用有序列表存储同一类型的多个值，相同的值可以多次出现在一个数组的不同位置中。我们需要明确，数组是有序的，元素类型是相同的。

Swift中的Array与Foundation中的NSArray是桥接的，可以相互转换。

##创建一个空数组（Create An Empty Array）

由于Swift中的数组是泛型，因此必须指定元素的数据类型：

```
// 使用构造语法来创建一个由特定数据类型构成的空数组：
// 方括号中必须指定类型，因为Array是一种泛型
var emptyArray = [Int]()

// 也可以使用结构体Array的构造写法
var anotherEmptyArray = Array<Int>()

// 如果之前类型明确了，直接再用[]即可.一样是Int类型的元素，是可推断出来的
emptyArray = []
```

##创建带默认值的数组（Creating An Array With A Default Value）

Swift提供了如下方法，可以创建带有默认值的数组，这个构造方法是使用count个指定元素来初始化：

```
/// Construct a Array of `count` elements, each initialized to
/// `repeatedValue`.
public init(count: Int, repeatedValue: Element)
```

例如：

```
var threeDoubles = [Double](count: 3, repeatedValue:0.0)
// threeDoubles 是一种 [Double] 数组，等价于 [0.0, 0.0, 0.0]
```

##数组相加来创建数组（Creating An Array By Adding Two Array Togeter）

两个数组能够通过加号`+`来创建一个合并的数组，是因为苹果为我们提供了这样的方法：

```
public func +<RRC1 : RangeReplaceableCollectionType, RRC2 : RangeReplaceableCollectionType where RRC1.Generator.Element == RRC2.Generator.Element>(lhs: RRC1, rhs: RRC2) -> RRC1
```

这是个泛型函数，要求类型RRC1和RRC2是遵守`RangeReplaceableCollectionType`协议的，并且这两个集合的元素类型要求相同。而数组是遵守`RangeReplaceableCollectionType`协议的，因此可以通过此方法将两个数组相加。

例如：

```
// anotherThreeDoubles 被推断为 [Double]，等价于 [2.5, 2.5, 2.5]
var anotherThreeDoubles = Array(count: 3, repeatedValue: 2.5)

// sixDoubles 被推断为 [Double]，等价于 [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
var sixDoubles = threeDoubles + anotherThreeDoubles
```

##用数组字面量构造数组（Creating An Array With An Array Literal）

我们可以使用字面量数组来进行数组构造，这是一种用一个或者多个数值构造数组的简单方法。字面量是一系列由逗号分割并由方括号包含的数值：

```
// shoppingList 已经被构造并且拥有两个初始项。
var shoppingList: [String] = ["Eggs", "Milk"]
```

由于类型是明确的，因此我们可以交由编译器来推断类型数组的类型：

```
var shoppingList = ["Eggs", "Milk"]
```

##访问和修改数组（Accessing And Modifying An Array）

常见的数组访问和修改操作有：

* 获取数组元素个数
* 判断数组是否为空
* 数组追加元素
* 数组在某个位置插入元素
* 数组在某个范围插入元素

###数组元素个数

获取数组元素的个数，可通过`count`属性获取：

```
// 打印结果："The shopping list contains 2 items."
print("The shopping list contains \(shoppingList.count) items.")
```

###数组判断是否为空

判断数组是否为空，可以通过`isEmpty`属性判断，也可以通过`count`判断：

```
if shoppingList.isEmpty {
    print("The shopping list is empty.")
} else {
    print("The shopping list is not empty.")
}

// 或者通过count属性
if shoppingList.count == 0 {
    print("The shopping list is empty.")
} else {
    print("The shopping list is not empty.")
}
```

###数组末尾追加元素

常用的有以下几种方法：

* 通过`append()`追加
* 通过`appendContentsOf()`追加
* 通过`+=`追加

我们先来分析一下数组结构体所提供的方法：

```
/// Append the elements of `newElements` to `self`.
///
/// - Complexity: O(*length of result*).
public mutating func appendContentsOf<S : SequenceType where S.Generator.Element == Element>(newElements: S)
/// Append the elements of `newElements` to `self`.
///
/// - Complexity: O(*length of result*).
public mutating func appendContentsOf<C : CollectionType where C.Generator.Element == Element>(newElements: C)
```

我们可以看到对于第一个`appendContentsOf`方法要求参数`S`是遵守了`SequenceType`：


```
var array = [1, 2]

// 追加单个元素
array.append(3)

// 追加一个数组
array += [4, 5]

// 追加一个集合
array.appendContentsOf([6, 7])

// 追加一个序列：1,2,3,4,5,6
array.appendContentsOf(1...6)
```

###数组插入元素

数组提供了`insert(_:at:)`方法插入数据：

```
/// Insert `newElement` at index `i`.
///
/// - Requires: `i <= count`.
///
/// - Complexity: O(`count`).
public mutating func insert(newElement: Element, atIndex i: Int)
```

那么可以通过调用来插入数据：

```
var array = [1, 2]
array.insert(55, atIndex: 2)
```

###数组修改值

我们也可以通过下标获取或者修改数据，我们先看看声明：

```
// 这个方法是在index位置插入数据或者获取index位置的数据
public subscript (index: Int) -> Element

// 这个方法是在指定的range插入多个数据或者获取指定range的数据
public subscript (subRange: Range<Int>) -> ArraySlice<Element>
```

通过下标获取或者修改值：

```
// 修改元素值，可以通过索引
array[0] = 100
// 获取
print(array[0])

// 可以修改指定范围的值
array[1..<4] = [77, 88, 99]
// 获取
print(array[1..<4])
```

也可以调用`replaceRange(_:with:)`来替换值：

```
array.replaceRange(1...2, with: [10, 12])
```

###数组移除元素

先看看苹果所提供的方法：

```
// 移除最后一个元素
public mutating func removeLast() -> Element

// 某个指定位置的元素
public mutating func removeAtIndex(index: Int) -> Element

// 移除某个范围
public mutating func removeRange(subRange: Range<Self.Index>)

// 移除前n个
public mutating func removeFirst(n: Int)

// 移除第一个
public mutating func removeFirst() -> Self.Generator.Element

// 清空
public mutating func removeAll(keepCapacity keepCapacity: Bool = default)
```

看看如何调用：

```
// 删除第一个
let element = array.removeAtIndex(0)

// 删除最后一个
let lastElement = array.removeLast()

// 删除所有元素，但是内存不释放，空间容量保留
array.removeAll(keepCapacity: true)

// 删除所有元素，且不保留原来的空间
array.removeAll(keepCapacity: false)

// 移除位置1、2、3的元素
array.removeRange(1...3)

// 移除第一个元素
let first = array.removeFirst()

// 移除前三个元素
array.removeFirst(3)
```

###数组遍历

常用的数组遍历有几种。

第一种，只获取元素值的for-in遍历：

```
for item in array {
    print(item)
}
```

第二种，获取元素及索引的for-in遍历：

```
for (index, value) in array.enumerate() {
  print("index: \(index), value = \(value)")
}
```

#集合（Sets）

集合(Set)用来存储相同类型并且没有确定顺序的值。当集合元素顺序不重要时或者希望确保每个元素只出现一次时可以使用集合而不是数组。

为了存储在集合中，该类型必须是可哈希化的。也就是说，该类型必须提供一个方法来计算它的哈希值。一个哈希值是Int类型的，它和其他的对象相同，其被用来比较相等与否，比如a==b,它遵循的是a.hashValue == b.hashValue。Swift 的所有基本类型(比如String,Int,Double和Bool)默认都是可哈希化的，它可以作为集合的值或者字典的键值类型。没有关联值的枚举成员值(在枚举有讲述)默认也是可哈希化的。

使用自定义的类型作为集合的值或者是字典的键值类型时，需要使自定义类型服从Swift标准库中的Hashable协议。服从Hashable协议的类型需要提供一个类型为Int的取值访问器属性hashValue。这个由类型的hashValue返回的值不需要在同一程序的不同执行周期或者不同程序之间保持相同。 因为hashable协议服从于Equatable协议，所以遵循该协议的类型也必须提供一个"是否等"运算符(\==)的实现。这个Equatable协议需要任何遵循的\==的实现都是一种相等的关系。也就是说，对于a,b,c三个值来说，\==的实现必须满足下面三种情况：

* a\==a(自反性)
* a\==b 可推出 b\==a(对称性)
* a\==b&&b\==c 可推出 a\==c(传递性)

##空集合（Empty Set）

`Swift`中的`Set`类型被写为`Set<Element>`，这里的`Element`表示`Set`中允许存储的类型，和数组不同的是，集合没有等价的简化形式。

```
var emptySet = Set<Int>()
print(emptySet.count) // 0

// 当变量已经有明确的类型时，再用空数组来初始化[]，其类型也是可推断的
emptySet = []// 类型为Set<Int>()
```

##初始化集合（Initializing An Empty Set）

可以直接使用数组字面量来初始化集合：

```
// 用数组构造集合
var favSet: Set<String> = ["milk", "egg"]

// 如果元素都是相同类型，带有初始化，可以省略类型
var favSet1: Set = ["good milk", "bad egg"]
```

使用集合可以非常好地解决重复问题：

```
// 重复的数据会被自动去重
var favSet: Set<String> = ["milk", "egg", "egg", "milk"]
print(favSet.count)// 2
```

##访问集合（Accessing A Set）

获取集合元素的个数，可以通过`count`只读属性获取，其时间复杂度为O(1)，也就是常量，因此效率是极高的。想想时间复杂度能达到O(1)，说明内部并不是列表，而是类似字典那样的哈希表，不过具体内部是如何，这里不细说，还得去查看源代码才能知晓。

```
/// The number of members in the set.
///
/// - Complexity: O(1).
public var count: Int { get }
```

调用就很方便了，效率又非常地高：

```
let count = favSet.count
```

我们还可以通过`isEmpty`属性判断是否为空：

```
// 官方声明如下：
/// `true` if the set is empty.
public var isEmpty: Bool { get }
    
// 判断是否为空集
if favSet.isEmpty {
  print("empty set")
} else {
  print("non empty set")
}
```

判断元素是否在集合中：

```
// 判断是否包含元素
if favSet.contains("wahaha") {
  print("fav set contains wahaha")
} else {
  print("fav set doesn\'t not contain wahaha")
}
```

##遍历集合（Iterating Over A Set）

遍历很少能离开得了`for-in`语法：

```
// 遍历一个set。使用for-in
for value in favSet {
  print(value)// 这是无序的
}
```

遍历集合，同时使集合是有序的：

```
// 要使之有序
// sort方法
for value in favSet.sort({ (e, ee) -> Bool in
  return e > ee // 由我们指定排序类型desc（降序）
}) {
  // milk
  // egg
  print(value)
}

// 或者：
// 或者直接使用sort无参数（默认为asc升序）
for value in favSet.sort() {
  // egg
  // milk
  print(value)
}
```
当然还可以使用最原始的方法来遍历：

```
for var i = favSet.startIndex; i != favSet.endIndex; i = i.successor() {
  print("for-i-in" + favSet[i])
}
```

##集合基本操作（Fundamental Set Operations）

集合的基本操作有：

* 求交集：使用intersect(_:)方法根据两个集合中都包含的值创建的一个新的集合。
* 求非交集：使用exclusiveOr(_:)方法根据在一个集合中但不在两个集合中的值创建一个新的集合。
* 求并集：使用union(_:)方法根据两个集合的值创建一个新的集合。
* 求差集：使用subtract(_:)方法根据不在该集合中的值创建一个新的集合。

![image](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Art/setVennDiagram_2x.png)

```
let set1: Set = [1, 2, 3, 4, 5]
let set2: Set = [1, 4, 3, 8]

// 求两个集合的交集
// 3, 1, 4
print(set1.intersect(set2))

// 求两个集合的并集
// 1, 2, 3, 4, 5
print(set1.union(set2).sort())

// 求集合set1去掉set1与set2的交集部分
// 2, 5
print(set1.subtract(set2).sort())

// 求集合set1与set2的并集 去掉 集合set1与set2的交集 部分
// 2, 5, 8
print(set1.exclusiveOr(set2).sort())
```

###集合成员关系（Relationships Of Set members）

* 使用\==来判断两个集合是否包含全部相同的值。
* 使用isSubsetOf(_:)方法来判断是否为子集
* 使用isSupersetOf(_:)方法来判断是否为超集
* 使用isStrictSubsetOf(\_:)或者isStrictSupersetOf(_:)方法来判断一个集合是否是另外一个集合的子集合或者父集合并且两个集合并不相等。
* 使用isDisjointWith(_:)方法来判断两个集合是否不含有相同的值。

![image](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Art/setEulerDiagram_2x.png)

从图中可以看出：

1. 集合a全部包含b，也就是说b是a的子集
2. 集合a与集合b相交，但是a与b不完全包含
3. 集合c与集合b完全没有交集

```
let set1: Set = [1, 2, 3, 4, 5]
let set2: Set = [1, 4, 3, 8]

// false
// set1并没有全部被set2包含
print(set1.isSubsetOf(set2))

let set3: Set = [1, 2, 5]

// true
// set3全部在set1中
print(set3.isSubsetOf(set1))

// true
// set1真包含set3，因此set1是set3的超集
print(set1.isSupersetOf(set3))

// true
// set1是set3的严格超集，因为Set1真包含set3且set1 != set3
print(set1.isStrictSupersetOf(set3))

// true
// set1真包含set3，且set1 != set3，因此set3是set1严格的真子集
print(set3.isStrictSubsetOf(set1))

let set4: Set = [1, 5, 2]

// false
// 因为set3和set4的元素是一样的，即相等
print(set4.isStrictSubsetOf(set3))

// false
// 这两个集合中有相同的元素
print(set3.isDisjointWith(set4))

let set5:Set = [9, 6]

// true
// set5和set4没有相同的元素
print(set5.isDisjointWith(set4))
```

#字典（Dictonaries）

字典是一种存储多个相同类型的值的容器。每个值都关联唯一的键，键作为字典中的这个值数据的标识符。和数组中的数据项不同，字典中的数据项并没有顺序的。我们在需要通过key访问数据的时候使用字典，这种方法很大程度上和我们在现实世界中使用字典查字义的方法一样。

Swift中的Dictionary与Foundation中的NSDictionary是桥接的，可以互相转换使用。

我们先来看看字典的结构定义：

```
/// A hash-based mapping from `Key` to `Value` instances.  Also a
/// collection of key-value pairs with no defined ordering.
public struct Dictionary<Key : Hashable, Value> : CollectionType, DictionaryLiteralConvertible
```

它是基于哈希表从key到value实例的映射，它的键值是无序的。要求key是遵守Hashable的。
其正规语法应该是：`Dictonary<key, value>`。

##创建空字典（Creating An Empty Dictionary）

```
// 创建空字典
var emptyDict = [Int: String]()

// 或者
emptyDict = Dictionary<Int, String>()

// 当创建过emptyDict后，再用[:]又成为一个空字典
emptyDict = [:]
```

##初始化字典（Initializing A Dictionary）

```
// 多个键值对用逗号分割
// 通过类型推断是可以推断出来类型为[String: String]
var dict = ["A": "Apple", "B": "Bubble"]

// 手动指明数据类型
var airports: [String: String] = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
```

声明不可变字典：

```
// 如果不希望字典可变，用let声明即可
let immutableDict = ["A": "Apple", "B": "Bubble"]
// 无法调用，因为是不可变的
//immutableDict.updateValue("App", forKey: "A")
//immutableDict += ["AB:", "Apple Bubble"]
```

##访问和修改字典（Accessing And Modifying A Dictionary）

我们可以通过字典的方法和属性来访问和修改字典，或者通过使用下标语法。

判断是否为空字典：

```
if dict.isEmpty {
  print("dict is an empty dictionary")
} else {
  print("dict is not an empty dictionary")
}

// 或者通过获取字典元素的个数是否为0
if dict.count == 0 {
  print("dict is an empty dictionary")
} else {
  print("dict is not an empty dictionary")
}
```

可以直接通过下标获取元素：

```
// 通过key直接获取
var apple = dict["A"]

// 获取索引获取(key, value)，再获取值
// 由于字典中的(key, value)是元组，因此可以直接通过点语法获取
let startKeyValue = dict[dict.startIndex]
let value = startKeyValue.1

// 读取时，判断是否有值
if let value = dict["BB"] {
  print("value is \(value)")
} else {
  // 输入这行
  print("The key BB doesn\'t not exist in dict")
}
```

可以直接通过下标来修改或者增加值：

```
// 修改，如果不存在，则会增加，否则更新值
dict["A"] = "Good Apple"

// 或者用API，如果不存在，则会增加，否则只是更新值
if let oldValue = dict.updateValue("Dog", forKey: "D") {
  print("The old value is \(oldValue)")
} else {
  // 输出这行
  print("The key D does\'t not exist before update")
}
```

移除元素：

```
// 通过下标语法移除元素
dict["D"] = nil

// 或者通过调用removeValueForKey方法来移除
if let removedValue = dict.removeValueForKey("D") {
  print("The removed value is \(removedValue)")
} else {
  print("The key D doesn'\t exist in dict, can\' be removed.")
}

// 或者清空字典：
dict.removeAll()
```

##字典遍历（Iterate Over A Dictionary）

字典中每个元素是一个键值对，而这个键值对其实就是一个元组：

```
// 用for-in遍历，形式为(key, value)
for (key, value) in dict {
  print("\(key): \(value)")
}
```

可以通过遍历所有的key：

```
// 可以遍历key
for key in dict.keys {
  print("\(key): \(dict[key])")
}
```

也可以遍历所有的值：

```
for value in dict.values {
  print("\(value)")
}
```

##字典值或者键转换成数组

如果不清楚Array有这么一个构造函数，那么您可能会手动写代码去遍历，但是事实上我们可以直接使用构造函数来实现的。我们知道字典的keys和values属性返回来的是`LazyMapCollection<key, value>`类型，并不是我们直接需要到的数据，因此我们有的时候需要转换成我们直接能使用的数据，这时候就需要这么做的：

```
// 获取所有的值
let values = [String](dict.values)

// 获取所有的键
let keys = [String](dict.keys)
```

#写在最后

本篇博文是笔者在学习Swift 2.1的过程中记录下来的，可能有些翻译不到位，还请指出。另外，所有例子都是笔者练习写的，若有不合理之处，还望指出。

学习一门语言最好的方法不是看万遍书，而是动手操作、动手练习。如果大家喜欢，可以关注哦，尽量2-3天整理一篇Swift 2.1的文章。这里所写的是基础知识，如果您已经是大神，还请绕路！

>文章并非纯翻译，而是练习语法而记录下来的内容

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

#支持并捐助

如果您觉得文章对您很有帮助，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
