#前言

在开发中一定需要到调试跟踪，但是很多开发者虽然做过很多的项目，但是未必了解开发中有哪些调试命令可以帮助我们开发者更快更好地定位到问题所在。

本篇文章主要是讲解在开发中如何利用LLDB来Debug。首先会讲一些基础知识，主要是帮助新手们学习如何去调试。对于一些比较高级的操作，不会也没有关系，但是如果能够掌握得了的话，会更方便更快速地查找问题。

#初步认识LLDB

LLDB是XCode内置的为我们开发者提供的调试工具。至于还不懂什么是调试的，百度一下概念吧，笔者也不知如何描述。看看下图吧，应该就可以大概明白什么是调试了！

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-19-下午5.57.35.png)

我们加了断点，然后在运行到断点处就停了下来，接下来我们看到lldb这里了吗？我们可以通过lldb所提供的命令来操作。

#基本调试操作


从上图中，我们八个按钮，我们讲讲前五个按钮：

* 第一个按钮点击就会收起这一栏目了，也就看不见了。
* 第二个按钮：如果为蓝色，就是断点有效。如果点击它变成灰色，就是所有断点不起作用。
* 第三个按钮：是继续的意思，会让程序从断点处恢复继续往下运行，我们点了这个按钮后，应用就会恢复正常运行状态。
* 第四个按钮是：单步执行的意思，每点这个按钮一次，程序就会从我们断点开始的地方，向下执行一步。
* 第五个按钮是：进入执行的意思，简单来说就是如果我们当前的断点在一个函数调用上，把么断点会继续进入这个函数的内部进行调试。
* 第六个按钮是：跳出的意思, 就是如果我们当前在一个函数中，它会跳出当前的函数，回到函数的调用处。

#常用p、po、call命令

先看下图：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-19-下午9.45.34.png)


以下是输入help命令时打印出来的，可以看看这四者有什么不同：

```
p         -- ('expression --')  Evaluate an expression (ObjC++ or Swift) in
               the current program context, using user defined variables and
               variables currently in scope.
po        -- ('expression -O  -- ')  Evaluate an expression (ObjC++ or Swift)
               in the current program context, using user defined variables and
               variables currently in scope.
print     -- ('expression --')  Evaluate an expression (ObjC++ or Swift) in
               the current program context, using user defined variables and
               variables currently in scope.
call      -- ('expression --')  Evaluate an expression (ObjC++ or Swift) in
               the current program context, using user defined variables and
               variables currently in scope.
```

从官方的描述来看，p、print、call是一样的，但是po就不太一样了，输入一样但是输出不一样。po的输出的是具体对象的内容。

如果想要按照特定的格式来打印，如下：

```
(lldb) p/s blogName
(__NSCFConstantString *) $9 = @"标哥的技术博客"
(lldb) p/x blogName
(__NSCFConstantString *) $10 = 0x000000010921c0a8 @"标哥的技术博客"
(lldb) p/t blogName
(__NSCFConstantString *) $11 = 0b0000000000000000000000000000000100001001001000011100000010101000 @"标哥的技术博客"
(lldb) p/a blogName
(__NSCFConstantString *) $12 = 0x000000010921c0a8 @ @"标哥的技术博客"
```

关于这个规则问题，请查阅[打印输出格式化](https://sourceware.org/gdb/onlinedocs/gdb/Output-Formats.html)

#lldb声明变量

我们可以使用e命令定义变量，然后在调试中使用。看如下的例子：

```
(lldb) e NSString *$str = @"http://www.henishuo.com"
(lldb) po $str
http://www.henishuo.com

(lldb) e int $count = 10
(lldb) p $count
(int) $count = 10
(lldb) e NSArray *itemArray = @[@"Test", @"Demo", @"huangyibiao"]
(lldb) po $count
10
```

我们使用e声明了$str变量，然后下面就可以使用了。我们看到通过p命令打印出来的都是$开头的变量了吗？我们在声明和使用时也需要加上$符号，与PHP一样！

在调试时，有时候想临时计算一下某个值来比较时，就可以通过这种方式来实现了，再也不用到源代码处添加上声明实现然后添加一句打印了，是否便利了很多？

#调用变量的API

当我们在断点处，定义了blogName变量了，因此我们可以通过调试命令来调用

```
(lldb) po [blogName uppercaseString]
标哥的技术博客

(lldb) po [blogName substringFromIndex:2]
的技术博客

```

#强转返回值类型

当我们调用API返回值类型不指定时，有时候所打印出来的东西是我们看不懂的，比如下面的获取结果应该是“标”字，但是不同类型打印结果却不一样：

```
(lldb) po [blogName characterAtIndex:0]
26631

(lldb) po (unsigned int)[blogName characterAtIndex:0]
26631

(lldb) po (char)[blogName characterAtIndex:0]
'\a'

(lldb) po (NSString *)[blogName characterAtIndex:0]
0x0000000000006807

(lldb) po (unichar)[blogName characterAtIndex:0]
U+6807 u'标'
```

#加断点

如果我们不是在一开始就添加所有的断点，而在调试开始后，想给其它地方加个断点，那么我们可以很方便地通过命令添加断点：

```
(lldb) b   33
Breakpoint 9: where = OCLLDBDebugDemo`-[ViewController onButtonClicked:] + 53 at ViewController.m:33, address = 0x000000010921a6d5
```

这是在当前类文件下的33行添加一个断点，添加成功后会有提示，如这里的提示就是成功地在33行添加了断点。当然，添加断点的方式也有好几种，如：

```
(lldb) b  -[ViewController onButtonClicked:]
Breakpoint 4: where = OCLLDBDebugDemo`-[ViewController onButtonClicked:] + 53 at ViewController.m:33, address = 0x000000010921a6d5
```

实际也是在33行添加断点。不过我们若要使用动态添加断点，就使用b命令加行号就可以了，这种最简单了。

#设置断点触发条件

看下图，笔者是怎么设置触发条件的：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-19-下午10.46.58.png)

我们在NSLog这一行，设置了条件，只有当条件满中时，才会进入断点，不过这里并没有让它进入断点，而条件满足时就发出声音并打印提示语。

这种应用场景主要是在循环遍历数据时，想要断点跟踪就只能通过这种方式了，除非添加NSLog打印，但是这种需要手动添加代码，在调试时才想到要添加一些打印语句，这时候又得重新运行，这太慢了。如果懂得如何设置断点条件，那么就能满足我们的需求了，直接可以设置条件。

#常用打印视图层次结构

当我们想要知道某个视图的结构时，可以通过调用recursiveDescription方法来打印出来，那么其结构就一目了然了：

```
(lldb) po [self.view recursiveDescription]
<UIView: 0x7fdd1052af10; frame = (0 0; 320 568); autoresize = W+H; layer = <CALayer: 0x7fdd1052b290>>
   | <UIButton: 0x7fdd10529070; frame = (66 183; 188 40); opaque = NO; autoresize = RM+BM; layer = <CALayer: 0x7fdd1051bff0>>
   |    | <UIButtonLabel: 0x7fdd104162f0; frame = (41.5 11; 105 18); text = '标哥的技术博客'; alpha = 0.2; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x7fdd10412590>>
   |    |    | <_UILabelContentLayer: 0x7fdd12804f30> (layer)
   | <_UILayoutGuide: 0x7fdd1052b300; frame = (0 0; 0 20); hidden = YES; layer = <CALayer: 0x7fdd1052b710>>
   | <_UILayoutGuide: 0x7fdd1052c070; frame = (0 568; 0 0); hidden = YES; layer = <CALayer: 0x7fdd1052c200>>
```

#临时刷新界面UI

本demo中，最开始按钮的背景颜色是blueColor，现在我们要在调试过程中修改其背景色为红色，并刷新界面。执行下面的命令行，App界面的按钮背景颜色是：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-19-下午11.00.55.png)

```
(lldb) e ((UIButton *)sender).backgroundColor = [UIColor redColor]
(UICachedDeviceRGBColor *) $41 = 0x00007fdd10715b00
(lldb) e (void)[CATransaction flush]
```

执行上面的命令后，App界面的按钮背景颜色是：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-19-下午11.00.43.png)

这种做法很有用的哦。当我们在调试UI时，因为颜色类似而不容易区分出来，但是我们可以在调试时通过这样的方式来修改背景色，就不用给源代码写相应的代码来重新运行看效果了。

在调试下运行上面的命令后，按钮的背景颜色就变成了红色了！

#修改变量值

本小节通过使用expr在调试过程中修改变量的值，感谢我的大徒弟在看完之后根据自己的经验总结出这一条技巧，现在也分享出来给大家~

使用很简单，就是这样的规则：

```
expr variable = newValue
```

如下图，这是笔者所测试的小例子：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-21-下午5.29.28.png)


#最后

写下本篇文章的主要目的是小徒弟不太懂调试，写下此篇文章以帮助小徒弟同时也帮助大家更好地在开发中学会去调试代码。其实还有很多的调试命令，但是不常用，这里就不一一列出来讲解了，大家若想了解更多，可以输入help查看！

#关注我


关注                | 账号              | 备注
-------------      | -------------     | ----------------
Swift/ObjC技术群一  | 324400294         |  群一若已满，请申请群二
Swift/ObjC技术群二  | 494669518         | 群二若已满，请申请群三
Swift/ObjC技术群三  | 461252383         | 群三若已满，会有提示信息
关注微信公众号       | iOSDevShares      | 关注微信公众号，会定期地推送好文章
关注新浪微博账号      |  [标哥Jacky](http://weibo.com/u/5384637337) | 关注微博，每次发布文章都会分享到新浪微博
关注标哥的GitHub     | [CoderJackyHuang](https://github.com/CoderJackyHuang) | 这里有很多的Demo和开源组件
关于我               | [进一步了解标哥](http://www.henishuo.com/about-biaoge/) | 如果觉得文章对您很有帮助，可捐助我！



