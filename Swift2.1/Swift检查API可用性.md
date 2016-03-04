#前言

我们在Swift开发中，为了适配不同的系统版本，我们必须要对API的兼容性做处理。因此这里总结一下在Swift开发中对API有效性的常用判断方式。

说明：本文中的Swift开发语言是基于Swift2.1语法的，若旧版本不支持，请参考相关文章。

#回顾Objective-C的检查方式

1. 方式一：通过获取iOS版本，然后判断是否是某个版本范围。如：

```
if ([UIDevice currentDevice].systemVersion.intValue >= 8) {
  // 调用8.0以后才支持的API
}
```

2. 方式二：通过条件编译的方式判断是否是某个系统版本之上。如：

```
#if __IPHONE_OS_VERSION_MIN_REQUIRED < __IPHONE_7_0
     // 调用7.0以后才支持的API
#endif
```

3. 方式三：通过-respondsToSelector:方法来判断。如：

```
if (self respondsToSelector:@selector(testAPI)) {
  // 调用
  [self testAPI];
}
```

是否还有更多方式呢？一时想不出来了，如果还有别的方式，请联系我，谢谢！

#学习Swift的检查方式

Swift2.0中，我们可以通过#available来判断是否可用。看看其语法如何：

语法规则：`#available(iOS v, OSX v1, *)`，其中参数不要求全部使用，可以直接使用`#available(iOS v, *)`或者`#availabel(OSX, *)`

1. 方式一：我们仍然可以使用respondsToSelector方法来判断。如：

```
if tableView.respondsToSelector(Selector("reloadData")) {
   print("support reloadData")
}
```

注意：Selector类型与Objective-C中的不同，不会自动提示。

2. 方式二：使用@available方式检查。如：

```
let interfaceOrientation : UIInterfaceOrientation        
if #available(iOS 8.0, *) {
    interfaceOrientation = UIInterfaceOrientation.Portrait
} else {
    interfaceOrientation = rootController.interfaceOrientation
}
```

如果是iOS8.0及其以上版本，就会执行第一个分支，否则执行else分支。

下面看看我们封装的一个显示alertview的API： 

```
if #available(iOS 8.0, *) {

 // prevent show too many times
 guard let _ = sg_hyb_alert_actionsheet_controller else {
   return
 }

 let alertController = UIAlertController(title: title, message: message, preferredStyle: preferredStyle)

 if let actions = alertActions {
   for alertAction in actions {
     alertController.addAction(alertAction)
   }
 }

 sg_hyb_alert_actionsheet_controller = alertController
 self.presentViewController(alertController, animated: true, completion: nil)
} else {
  UIAlertView(title: title, message: message, delegate: nil, cancelButtonTitle: ok).show() } 
```

这是因为在iOS8.0以后，有UIAlertController类，更简单地完成显示alert或者actionSheet。


##为Swift的API提供版本限制

要为Swift的api设置系统版本要求，我们可以通过以下方式：

```
@available(iOS 9.0, *)
func checkAvailable() {
 let store = EKStock()
 // ...
}
```

上面的API前面添加了 @available(iOS 9.0, *) ，就要求iOS9.0才能有效。因此在编译时会检查出来是否满足，如果不满足就会产生编译错误，如：

>‘checkAvailable’ is only available on iOS 9.0 or newer

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
