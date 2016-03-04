#前言


本文主要是说一些`iOS9`适配中出现的坑，如果只是要单纯的了解`iOS9`新特性可以看瞄神的开发者所需要知道的 `iOS 9 SDK`新特性。9月17日凌晨，苹果给用户推送了`iOS9`正式版，随着有用户陆续升级`iOS9`，也就逐渐的衍生出了一系列的问题，笔者也在赶忙为自己维护的`App`做适配，本文写的一些坑基本都是亲身体验了。

#兼容HTTP

一、`NSAppTransportSecurity`

`iOS9`让所有的`HTTP`默认使用了`HTTPS`，原来的`HTTP`协议传输都改成`TLS1.2`协议进行传输。直接造成的情况就是`App`发请求的时候弹出网络无法连接。解决办法就是在项目的`info.plist`文件里加上如下节点：

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919100632101-919817909.png)

`NSAppTransportSecurity`字典中的`key`： `NSAllowsArbitraryLoads`设置为`YES`。

这个子节点的意思是：是否允许任意加载？ 设为`YES`的话就将禁用了`AppTransportSecurity`转而使用用户自定义的设置，这个问题就解决了。 

上面说是苹果限制了`HTTP`协议，但是也并不是说所有的`HTTPS`都能完美适配`iOS9`了。

举个栗子，从`app`内起`webView`加载`https`的网页。新建个项目写几行起网页的代码

```
- (void)loadView{
    UIWebView *web = [[UIWebView alloc]initWithFrame:[UIScreen mainScreen].bounds];
    self.view = web;
}
- (void)viewDidLoad {
    [super viewDidLoad];
     
    UIWebView *web = (UIWebView *)self.view; //董铂然
    NSURL *url = [NSURL URLWithString:@"https://github.com/"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [web loadRequest:request];
}
```

中间的`url`就是我们想要加载的`https`地址，用`https://baidu.com/` 和 `https://github.com/` 分别试一下，结果不同

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919103956992-179392949.png)
![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919105348273-215889174.png)
  
`github`的网页能打开，百度的网页打不开，下面打印了一行`log`：

```
NSURLSession/NSURLConnection HTTP load failed (kCFStreamErrorDomainSSL, -9802)
```

原因是苹果的官方资料说首先必须要基于`TLS 1.2`版本协议。然后证书的加密的算法还需要达到`SHA256`或者更高位的`RSA`密钥或`ECC`密钥，如果不符合，请求将被中断并返回`nil`.

在浏览器中是可以直接查看这个网站的加密算法的，先点绿锁再点证书信息。

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919110309133-1945301697.png)

从右边两张图可以看出，`github`带`RSA`加密的`SHA-256`符合苹果的要求，所以才可以展示。

针对百度的情况可以在`info.plist`中配置如下，如果网站引用的比较多应该是需要针对每个网站进行配置。

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919113027601-1922047899.png)

`NSAppTransportSecurity`，`NSExceptionDomains`，`NSIncludesSubdomains`，`NSExceptionRequiresForwardSecrecy`，`NSExceptionAllowInsecureHTTPLoads`写在下面便于复制。

其中的`ForwardSecrecy`理解为超前的密码保护算法，在官方资料里有写，一共是`11`种。配置完毕百度可以访问。

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919112541304-15327845.png)   


#Bitcode


`bitcode`的理解应该是把程序编译成的一种过渡代码，然后苹果再把这个过渡代码编译成可执行的程序。`bitcode`也允许苹果在后期重新优化我们程序的二进制文件，有类似于`App`瘦身的思想。

用了`xcode7`的编译器编译之前没问题的项目可能会出现下列报错。

```
XXXX’ does not contain bitcode. You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target. for architecture arm64
```

问题的原因是：某些第三方库还不支持`bitcode`。要不然是等待库的开发者升级了此项功能我们更新库，要不就是把这个`bitcode`禁用。

禁用的方法就是找到如下配置，选为NO.(iOS中`bitcode`是默认`YES`，`watchOS`中`bitcodes`是不让改的必须`YES`。)

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150920182214226-557850566.png)

#设置信任


这一条只和企业级应用或`inhouse`有关，和`AppStore`渠道的应用无关。

在`iOS8`只是弹出一个窗问你是否需要让手机信任这个应用，但是在`iOS9`却直接禁止，如果真的想信任需要自己去手动开启。类似于Mac系统从未知开发者处下载的`dmg`直接打不开，然后要到系统偏好设置的安全性与隐私手动打开。 下图展示左边`iOS8`,右边`iOS9`：


![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919233010867-422003752.jpg) 

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919233043961-840859995.jpg)


用户需要去**设置---》通用---》描述文件**里面自行添加信任。

这种问题的处理方法也就两种：

* 1.提前周知暂时不要升级`iOS9`  
* 2.大多是公司员工使用的企业级应用，群发一个指导邮件。 

 

#字体

`iOS8`中，字体是`Helvetica`，中文的字体有点类似于“华文细黑”。只是苹果手机自带渲染，所以看上去可能比普通的华文细黑要美观。`iOS9`中，中文系统字体变为了专为中国设计的**“苹方”** 有点类似于一种word字体“幼圆”。字体有轻微的加粗效果，并且最关键的是字体间隙变大了！

所以很多原本写死了`width`的`label`可能会出现“...”的情况。

iOS8：

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919223903476-176844619.png)

iOS9 蛋疼：

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919223918101-1917717144.png)

上面这两张图也可以直观的看出同一个界面，同一个label的变化。

所以为了在界面显示上不出错，就算是固定长度的文字也还是建议使用sizetofit 或者ios向上取整 ceilf() 或者提前计算

```
CGSize size = [title sizeWithAttributes:@{NSFontAttributeName: [UIFont systemFontOfSize:14.0f]}];
CGSize adjustedSize = CGSizeMake(ceilf(size.width), ceilf(size.height));
```

#URL scheme


`URL scheme`一般使用的场景是应用程序有分享或跳其他平台授权的功能，分享或授权后再跳回来。

在`iOS8`并没有做过多限制，但是`iOS9`需要将你要在外部调用的`URL scheme`列为**白名单**，才可以完成跳转

如果iOS9没做适配 会报如下错误:

```
canOpenURL: failed for URL : "mqzone://qqapp" - error: "This app is not allowed to query for scheme mqzone"
```

具体的解决方案也是要在`info.plist`中设置`LSApplicationQueriesSchemes`类型为数组，下面添加所有你用到的`scheme`：

![image](http://images2015.cnblogs.com/blog/717809/201509/717809-20150919235751554-1405232517.png)

 

#statusbar


这个还好只是报一个警告，如果就是不管他，也不会出现问题。

```
<Error>: CGContextSaveGState: invalid context 0x0. If you want to see the backtrace, please set CG_CONTEXT_SHOW_BACKTRACE environmental variable.
```

以前我们为了能够实时的控制顶部`statusbar`的样式，可能会在喜欢使用

```
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent]
[[UIApplication sharedApplication] setStatusBarHidden:YES];
```

但是这么做之前需要将`info.plist`里面加上`View controller-based status bar appearance`的`BOOL`值设为`NO`，就是把控制器控制状态栏的权限给禁了，用`UIApplication`来控制。但是这种做法在`iOS9`不建议使用了，建议我们使用吧那个`BOOL`值设为YES，然后用控制器的方法来管理状态栏比如。

```
- (UIStatusBarStyle)preferredStatusBarStyle {
    return UIStatusBarStyleLightContent;
}
```

点进头文件可以验证刚才说法：

```
@property(readwrite, nonatomic,getter=isStatusBarHidden) BOOL statusBarHidden NS_DEPRECATED_IOS(2_0, 9_0, "Use -[UIViewController prefersStatusBarHidden]");
```

#didFinishLaunchingWithOptions


如果运行的时候报下列错误，那就是你的`didFinishLaunchingWithOptions`写的不对了

```
***** Assertion failure in -[UIApplication _runWithMainScene:transitionContext:completion:], /BuildRoot/Library/Caches/com.apple.xbs/Sources/UIKit_Sim/UIKit-3505.16/UIApplication.m:3294**
```

`iOS9`不允许在`didFinishLaunchingWithOptions`结束了之后还没有设置`window`的`rootViewController`。 也许是`xcode7`的编译器本身就不支持。

解决的方法当然就是先初始化个值，之后再赋值替换掉

```
UIWindow *window = [[UIWindowalloc] initWithFrame:[UIScreenmainScreen].bounds];
window.rootViewController = [[UIViewController alloc]init];
```

#tableView


虽然现在的`iOS9`已经推送正式版了，但是`iOS9`使用时还是会感觉到App比以前更加卡顿了，`tableView`拖动时卡顿显示的最为明显。 并且之前遇到一个`bug`，原本好的项目用xcode7一编译，`tableView`刷新出了问题 ，`[tableView reloadData]`无效 有一行`cell`明明改变了但是刷新不出来。 感觉可能是这个方法和某种新加的特性冲突了，猜测可能是`reloadData`的操作被推迟到下一个`RunLoop`执行最终失效。

解决的方法是，注释`[tableView reloadData]`，改用局部刷新，问题居然就解决了。

```
[self.tableView reloadSections:[NSIndexSet indexSetWithIndex:0] withRowAnimation:UITableViewRowAnimationNone];
```

#NSLocalizableString（XCode7问题）


如果你程序启动后出现主页面一片空白，或是报了以下的栈调用错误。那就是`NSLocalizableString`的死循环导致堆栈溢出了。

```
#0  0x003052a8 in -[NSLocalizableString length] ()
#1  0x003052cc in -[NSLocalizableString length] ()
#2  0x003052cc in -[NSLocalizableString length] ()
#3  0x003052cc in -[NSLocalizableString length] ()
#4  0x003052cc in -[NSLocalizableString length] ()
#5  0x003052cc in -[NSLocalizableString length] ()
#6  0x003052cc in -[NSLocalizableString length] ()
#7  0x003052cc in -[NSLocalizableString length] ()
#8  0x003052cc in -[NSLocalizableString length] ()
#9  0x003052cc in -[NSLocalizableString length] ()
#10 0x003052cc in -[NSLocalizableString length] ()
#11 0x003052cc in -[NSLocalizableString length] ()
#12 0x003052cc in -[NSLocalizableString length] ()
#13 0x003052cc in -[NSLocalizableString length] ()
#14 0x003052cc in -[NSLocalizableString length] ()
#15 0x003052cc in -[NSLocalizableString length] ()
#16 0x003052cc in -[NSLocalizableString length] ()
```

这个的解决方法就是找到特定的页面，然后将`English`前面的勾勾上。

![image](http://images2015.cnblogs.com/blog/717809/201510/717809-20151019171804395-1530027809.png)


#bundle identifier（Xcode7问题）


如果你遇到了在本地编译通过，但是在`CI`上打包失败。并且报的错误是和`bundle identifier`相关，那很有可能是你`plist`文件中写的`bundle identifier`没有起作用。

因为`xcode7`新增了此功能，在`target`下面的`BuildSetting`里面增加了`Product Bundle identifier`。苹果之后的做法应该是推荐在此处设置`bundle identifier`，此处的设置会比`info.plist`里面优先读取。

![image](http://images2015.cnblogs.com/blog/717809/201510/717809-20151019172627192-351688632.png)

如果你的`Bundle identifier`一直没变，可能不会发现此问题。如果改变了，你在`plist`中修改是无效的。

另一个做法就是在ci打包的配置`Execute shell`上增加以下代码

```
"Set :CFBundleIdentifier com.XXX.XXX" "XXX/Supporting Files/XXX-Info.plist"
```

#ActionSheet


`Actionsheet` 在iOS8的时候改了一次版，当时是和`AlertView`二合一，并且以`AlertViewController`作为载体，之后再`present`出来，这在当时，苹果应该是想统一各个控件的展示方式，但是很多人可能并没有在意因为直接`show`那个方法并没有废除，大家都觉得应该是新旧都能用，再加上有的公司可能自己还做了一定扩展，诸多原因导致还是用的旧方法。

![image](http://images2015.cnblogs.com/blog/717809/201511/717809-20151106102306149-1292091022.gif)

![image](http://images2015.cnblogs.com/blog/717809/201511/717809-20151106102318633-977085013.gif)

在iOS9上使用旧方法直接show，会出现左图的问题。如果用的是`AlertViewController`的方法则不会出现问题（右图）

   
我猜测可能是`sheet`的`windowLevel`比键盘低导致的。但是将优先级设到`10000`，然后显示在`keyWindow`上。

```
sheet.window.windowLevel = 10000;
[sheet showInView:[UIApplication sharedApplication].keyWindow];
```

然后没有效果，然后又查了下`stackoverflow` 有个方法能取出优先级最高的`window`

```
UIWindow *topWindow = [[[UIApplication sharedApplication].windows sortedArrayUsingComparator:^NSComparisonResult(UIWindow *win1, UIWindow *win2) {
    return win1.windowLevel - win2.windowLevel;
}] lastObject];
```

试了下还是没有效果。 应该键盘的优先级无论如何都是最高的， 想盖在键盘上面的方法行不通。

当然，如果更换的成本比较大，也并不是没有办法，直接设置弹`sheet`之前收回键盘就好了。

 
暂时遇到这些问题，感觉iOS9的出现让所有iOS开发都是菊花一紧，预祝所有的iOS都能及时的做好适配改完bug，下个版本一上线，所有问题都解决。

#推荐阅读

* [HTTPS接口加密和身份认证](http://www.henishuo.com/https-validation-check/)
* [iOS访问HTTPS SSL和TLS双向加密](http://www.henishuo.com/ios-https-tls-ssl/)

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


