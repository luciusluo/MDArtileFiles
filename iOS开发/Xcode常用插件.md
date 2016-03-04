#Xcode插件神器

作为`iOS`开发人员，不了解些常用的插件，不使用插件，开发效率怎么会够快呢？那么问题来了，现在的你，使用过哪些`xcode`插件？如果没有使用过插件，那么很遗憾，您错过了很多好用的工具！

#插件管理器


既然使用`xcode`插件，就应该想想有没有插件管理器呢？是的，有一个灰常有名的插件管理器叫`Alcatraz`，关于这个插件管理器如何安装，请参考[官方文档](https://github.com/alcatraz/Alcatraz)

如果您已经安装过这个插件管理器，那么恭喜您可以直接看下面的内容了！！！

安装完成以后，需要重启`Xcode`后，然后在`Xcode`的菜单栏上就可以看到这样`package manager`:

![image](http://henishuo.com/wp-content/uploads/2015/11/屏幕快照-2015-11-11-下午10.56.44.png)

##第一神器：注释


开发必须有规范，不然维护成本就会提高。那么写`api`时就应该配上非常明确的注释，而有一个插件`VVDocument`就是一个注释神器。有了它，我们只需要使用`///`就会自动触发生成格式化的注释。看下面的例子，我们写好这个`API`后，直接输入`///`，就会生成下面这样的注释！！！当然格式是可以调整的，这个插件提供了一个配置界面，可以修改触发方式以及生成的样式等。

```
/*!
 *  @author 黄仪标, 15-11-11 23:11:04
 *
 *  <#Description#>
 *
 *  @param patientModel <#patientModel description#>
 *  @param resultBlock  <#resultBlock description#>
 *
 *  @return <#return value description#>
 */
- (instancetype)initWithPatientModel:(HYBPatientModel *)patientModel resultBlock:(HYBResultBlock)resultBlock;
```		

看下图，就是当前我的配置界面，如果想要修改就可以在这个界面直接修改！！！

![image](http://henishuo.com/wp-content/uploads/2015/11/屏幕快照-2015-11-11-下午11.04.02.png)

##第二神器：XVim


我相信对于做过`Web`开发的人员，对`vim`这个工具是相当熟悉的吧。说真的，刚开始我也觉得这个东西不好用，不过那是因为不会用。后来看到有个同事是后端转`iOS`的，他一直在使用`XVim`插件操作好快，于是就想学习一下这个东西怎么用。

事实如此，真的是相当棒的插件。现在我的`Xcode`一直都有这个插件，而且对这个`vim`已经熟悉了，其常用的操作命令都记住了。

现在，本人也在学习`HTML5`，使用`Sublime Text3`开发工具，这个也是神器，支持很多的插件，而且也支持`vim`，简单是爽死了！！！如果您也在使用，一定要学习这个工具如何使用。

如果不想使用`Alcatraz`插件管理器来插件，可以直接到[https://github.com/JugglerShu/XVim](https://github.com/JugglerShu/XVim)下载运行。

##第三神器：XToDo


首先，其开源github地址为：[https://github.com/trawor/XToDo](https://github.com/trawor/XToDo)

如果想要下载运行安装，可以直接下载然后用`xcode`运行。

我们在开发时，经常使用`#warning`来添加提醒信息，但是实际上很多项目里面有很多这样的信息，这让我们非常难找。有了`XToDo`这个神器，我们可以通过这个插件所提供的工具，直接查看。

支持的写法：`TODO`,`FIXME`,`???`,`!!!!`。看到这几个应该可以猜得出来是什么意思了吧。没错，就是事项的意思。

我们在代码中可以这样添加：

```
// TODO: 在上线前需要将这个值设置为111（假设）
const NSUInteger kAppInterfaceVersion = 111;

// FIXME: 这里是写死的假数据
NSString *title = @"假数据";

// ???: 这里是什么意思？
NSString *value = [self test];

// !!!!: 警告区
NSString *warningVersion = @"1"
```

##第四神器：Cocoapods


现在新的项目中几乎都使用了`Cocoapods`来管理第三方库了，因此，这个插件也是必备神器啊！关于这个`Cocoapods`怎么使用，请阅读这篇文章：[http://www.henishuo.com/cocoapods-use/)，这篇文章介绍了其基本使用，并且也教大家让自己的开源项目也支持`Cocoapods`。

![image](http://henishuo.com/wp-content/uploads/2015/11/屏幕快照-2015-11-11-下午11.24.33.png)

有了这个插件，就可以通过直观的界面来操作了。当然，喜欢使用命令的也是可以的，本人就更喜欢直接操作命令。

##第五神器：DXXcodeConsoleUnicodePlugin


你知道吗？为什么`Xcode`控制台`Console`打印出来的`JSON`数据中有中文时都是看不懂的字符？这让人非常难受，只能通过断点调试才能单步进去看到这个值。那么现在有了这个神器就不用这么麻烦了！！！直接就可以打印出来看了！！！

##第六神器：FuzzyAutocomplete


这个`FuzzyAutocomplete`可是相当好用的家伙，可以自动匹配所有的变量、函数名等，而且不要求顺序。比如，`Xcode`自带的智能提示，我们只能是顺序的写了前面的字符才能匹配出来提示。那么这个神器就不一样了，不要求记得`API`的写法顺序，只要记住其中几个字母，就可以匹配出来了，然后选择就可以了。

看下图，是不是很明智：

![image](http://henishuo.com/wp-content/uploads/2015/11/屏幕快照-2015-11-11-下午11.31.23.png)

##第七神器：GitDiff


对于项目使用了`git`这个来管理版本的开发人员来说，这可就是一个神器了。我们在文件中发动了任何地方，在左边的代码行号这里都会有相应颜色显示，一看就可以看出来了。

当然，对于不是使用`git`来管理的人来说，这个插件就没有必要了。

看下图的左边，是不是不一样了：

![image](http://henishuo.com/wp-content/uploads/2015/11/屏幕快照-2015-11-11-下午11.38.11.png)

还可以点击还原：

![image](http://henishuo.com/wp-content/uploads/2015/11/屏幕快照-2015-11-12-下午2.19.57.png)

##第八神器：PrettyPrintJSON


开发一定需要到调试接口，那么打印出来的JSON数据又是乱乱的，根本不能直观看出来是什么结构嘛。那么安装这个东西就好办了，直接可以显示出很好的结构。当然我们可以使用浏览器插件：`JSON-handle`插件，这个是`google`浏览器的插件，有了这个东西，将接口放到浏览器时，返回的`JSON`数据会自动格式化。

##第九神器：SCXcodeSwitchExpander


这个插件也是好东西哦，当我们定义了枚举结构时，我们使用`SCXcodeSwitchExpander`插件就相当容易了。

我们定义一个枚举：

```
typedef NS_ENUM(NSUInteger, HYBErrorType) {
   kErrorTypeNetworkFail,
   kErrorTypeNetworkTimeout,
   kErrorTypeArgumentLess
} 
```

当我们在使用时，我们声明一个枚举变量，然后输入`switch (枚举变量)`就会自动地展开了：

``` 
HYBErrorType errorType;
switch (errorType) {
	case kErrorTypeNetworkFail:
	  break;
	case kErrorTypeNetworkTimeout:
	  break;
	case kErrorTypeArgumentLess:
	  break;
	default:
	  break;
}
```
这是不是很方便呢？必须的！！！

##第十神器：Auto-Import


这个`Auto-Import`插件是可以快速导入头文件的插件。这个就不多说了,看图吧！！！

![image](http://www.henishuo.com/wp-content/uploads/2016/02/demo.gif)

其实到现在我也没有这么使用过。所以只能排第十了！！！

#Xcode升级后插件失效解决方案


这里有一个脚本可以刷新所有的插件，下载[https://github.com/cikelengfeng/RPAXU](https://github.com/cikelengfeng/RPAXU)，按照文档说明运行脚本即可。亲测可用！！！

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


