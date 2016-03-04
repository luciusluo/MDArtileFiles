
#公司代码规范


| 版本  | 更改日期 | 更改内容 | 编制 | 审核 |
|:-------------:|:---------------:| -------------:|:-------------:|:-------------:|:-------------:|
|  v0.1   | 2015-06-05 | 依据Cocoa完善初稿 | 黄仪标| |
|  v0.2   | 2015-07-15 | 根据团队建议改进 | 黄仪标 | |
|  v0.3   | 2015-11-04 | 根据开发所遇到问题改进 | 黄仪标 |

 
#命名基础


在⾯面向对象软件库的设计过程中,开发人员经常忽视对类,⽅方法,函数,常量以及其他编程接⼝元素的命名。本节讨论大多数`Cocoa`接⼝的一些命名约定。

###一般性原则

最好是既清晰⼜又简短,但不要为简短⽽而丧失清晰性


| 代码 | 点评 |
|:-----|:------|
| insertObject:atIndex:| Good|
| insert:at:| 不清晰;要插⼊什么?“at”表⽰示什么?|
| removeObjectAtIndex:| Good|
| removeObject:|不错,因为⽅法是⽤用来移除作为参数的对象|
| remove:|不清晰;要移除什么?|
 
名称通常不缩写,即使名称很⻓,也要拼写完全

|代码|点评|
|:---|:---|:---|
|destinationSelection|Good|
|destSel|不清晰|
|setBackgroundColor:|Good|
|setBkgdColor:|不清晰|

###一致性

尽可能与`Cocoa`编程接⼝命名保持一致。如果你不太确定某个命名的⼀致性,请浏览头文件或参考文档中的范例，在使⽤多态方法的类中,命名的⼀致性⾮常重要。在不同类中实现相同功能的⽅法应该具有同的名称。

|代码|点评|
|:---|:---|:---|
|- (NSInteger)tag|在 NSView, NSCell, NSControl 中有定义|
|- (void)setStringValue:(NSString *)|在许多 Cocoa classes 中都有定义|
 
###前缀

所有类名、枚举、结构、`protocol`定义时，都加上全大写的`HYB`作为前缀
 
###后缀

所有`protocol`定义时，都加上后缀`Delegate`
。如，`HYBRefreshViewDelegate`，表示`RefreshView`的协议
 
###导入头文件

对于`Objective-C`的类文件，使用`#import`导入；对于`c,c++`文件，使用`#include`导入
 
###方法命名

方法名首字母小写，其他单词首字母大写,每个空格分割的名称以动词开头。如：

```
- (void)insertModel:(id)model atIndex:(NSUInteger)atIndex;
```

###属性、变量命名

每个属性命名都加上类型后缀，如，按钮就加上`Button`后缀，模型就加上`Model`后缀。eg:

```
@property (nonatomic, strong) UIButton *submitButton;
```

对于内部私有变量，也需要加上后缀并以下划线开头，如：`_titleLabel`, `_descLabel`, `_okButton`

###访问方法

访问⽅方法是对象属性的读取与设置⽅法。其命名有特定的格式,依赖于属性的描述内容。


1、如果属性是⽤用名词表达的,则命名格式为:     
  
```
- (type)noun;
- (void)setNoun:(type)aNoun;
```

  例如: 


``` - (NSString *)title;
- (void)setTitle:(NSString *)aTitle;
```

2、如果属性是⽤用形容词表达的,则命名格式为: 


```
- (BOOL)isAdjective;  - (void)setAdjective:(BOOL)isAdjective;

- (BOOL)isEditable;
- (void)setEditable:(BOOL)isEditable;
```


3、如果属性是⽤用动词表达的,则命名格式为: 

```
- (BOOL)verbObject;  - (void)setVerbObject:(BOOL)flag;
- 
- (BOOL)showsAlpha;
- (void)setShowsAlpha:(BOOL)flag;
```

####动词要⽤用现在时时态

* 不要使⽤用动词的过去分词形式作形容词使⽤用 
* 可以使⽤用情态动词(`can`, `should`, `will`等)来提⾼高清晰性,但不要使⽤用`do`或` does`

```
- (BOOL)openFile:(NSString *)fullPath withApplication:
(NSString *)appName andDeactivate:(BOOL)flag;

- (void)setAcceptsGlyphInfo:(BOOL)flag;
// 不好
- (BOOL)acceptsGlyphInfo;

- (void)setGlyphInfoAccepted:(BOOL)flag;
// 不好
- (BOOL)glyphInfoAccepted;

// 好
- (void)setCanHide:(BOOL)flag;
- (BOOL)canHide;
- 
// 好
- (void)setShouldCloseDocument:(BOOL)flag;
// 不好
- (BOOL)shouldCloseDocument;

// 好
- (void)setDoesAcceptGlyphInfo:(BOOL)flag;
- (BOOL)doesAcceptGlyphInfo;
```

只有在方法需要间接返回多个值的情况下,才使⽤`get`
像上面这样的方法,在其实现里应允许接受`NULL`作为其`in/out`参数,以表示调⽤者对⼀个或多个 返回值不感兴趣。

###委托方法

委托⽅方法是那些在特定事件发⽣生时可被对象调⽤用,并声明在对象的委托类中的⽅方法。它们有独特的命名约 定,这些命名约定同样也适⽤用于对象的数据源⽅方法。

1、名称以标示发送消息的对象的类名开头,省略类名的前缀并⼩小写第⼀个字⺟


```
- (BOOL)tableView:(NSTableView*)tableView shouldSelectRow:(int)row;
- (BOOL)application:(NSApplication *)sender 
           openFile:(NSString *)filename;
```

2、冒号紧跟在类名之后(随后的那个参数表⽰委派的对象)。该规则不适用于只有一个`sender`参数的⽅法:

```
- (BOOL)applicationOpenUntitledFile:(NSApplication *)sender;
```

3、上⾯的那条规则也不适⽤于响应通知的⽅法。在这种情况下,⽅方法的唯⼀参数表⽰通知对象

```
- (void)windowDidChangeScreen:(NSNotification *)notification;
```

4、⽤于通知委托对象操作即将发生或已经发⽣的方法名中要使⽤`did`或`will`

```
- (void)browserDidScroll:(NSBrowser *)sender; - (NSUndoManager *)windowWillReturnUndoManager:(NSWindow *)window;
```

5、用于询问委托对象可否执行某操作的⽅法名中可使⽤did或will,但最好使⽤should

```
- (BOOL)windowShouldClose:(id)sender;
```

###枚举常量

声明枚举类型时，命名以`HYB`为前缀，而枚举值以小写`k`开头，后面的单词首字母大写，其余小写。如：

```
typedef NS_ENUM(NSUInteger, HYBContentMode) {
  kContentModeScaleFit = 1 << 1,
  kContentModeScaleFill = 1 << 2
};
```

###const常量

以小写`k`开头，后面单词首字母大写，其余小写。如：

```
const float kMaxHeigt = 100.0f;
```

如果是静态常量，仅限本类内使用的，加上前缀`s_`，如果是整个工程共用，以`sg_`为前缀。如：

```
s_kMaxHeight; 
sg_kMaxHeight;
```
 
###其他常量

1、使用`#define`声明普通常量，以小写`k`开头，后面的单词首字母大写，其余小写。如:

```
#define kScreenWidth ([UIScreen mainScreen].bounds.size.width)
```

2、通知常量名，以Notification为后缀，如:

```
#define kLoginSuccessNotification @”HYBLoginSucessNotification”
```

###代码注释

####类注释

类头需要有注释,功能说明，作者等：如，

```
/**
 *   这里是功能说明
 *
 *  @author Huangyibiao
 *  @modify 如果有修改需要这行，加上修改人和修改日期
 */
@interface HYBUIMaker : NSObject
```

####方法注释
方法注释需要加上作者，功能说明，参数说明，返回值说明:

```
/**
 *  @author
 *  描述内容
 *
 *  @param string <#string description#>
 *  @param font   <#font description#>
 *
 *  @return <#return value description#>
 */
+ (CGSize)sizeWithString:(NSString*)string andFont:(UIFont *)font;
```

####块注释

对于块注释，有多种多种风格

#####风格一：

```
/////////////////////////////////////
// @name UIButton控件生成相关API
/////////////////////////////////////
```

#####风格二：

```
//
// 块功能说明
//
```

风格三：

```
/*
* 块功能说明
*
*/
```
 
###类内分块结构写法

有生命周期的类，从上到下结构为：

```
#pragma mark – life cycle
#pragma mark – Delegate
#pragma mark – CustomDelegate…
#pragma mark – Event
#pragma mark - Network
#pragma mark – Private
#pragma mark – Getter/Setter
```

#参考

本文档参考[Coding Guidelines for Cocoa](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ CodingGuidelines/CodingGuidelines.html)

#关注我


**Swift/ObjC技术群一：[324400294(已满)]()**

**Swift/ObjC技术群二：[494669518]()**

**ObjC/Swift高级群：[461252383（注明年限，新手勿扰）]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)


#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
