>#iOS App之间如何通信

---
假设需求是这样的：由一个app1跳转到app2之后，app2完成某项任务之后，怎么把app2的完成信息传到app1（自己的程序是app1），传的是什么类型的数据，怎么进行解析？

>#支持原创，请[阅读原文](http://www.henishuo.com/ios-app-communication/)

#逻辑

---
本文章使用TestApp1作为第一个app的URL Schemes，TestApp2为第二个app的URL Schemes。

#TestApp1工程配置

如下图，要适配`iOS9.0`：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-03-下午10.30.30.png)

对于`URL Schemes`中的`TestApp1`是本应用提供给其它应用调用的。

#TestApp2工程配置

如下图，要适配`iOS9.0`：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-03-下午10.30.48.png)

对于`URL Schemes`中的`TestApp2`是本应用提供给其它应用调用的。

#TestApp1工程中实现代码测试

---

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  // TestApp2是TestApp2这个app在info.plist中配置的URL Schemes
  if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"TestApp2://"]]) {
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"TestApp2://success=1&count=100"]];
  }
  
  return YES;
}

- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
  NSString *receText = [[url host] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
  NSLog(@"%@    %@",receText, url.absoluteString);
  
  
  return YES;
  
}
```

我们首先需要判断手机是否安装了应用`TestApp2`，通过`TestApp2`工程公开的`URL Schemes`来判断，即`TestApp2://`。需要传参数时，是通过URL参数来传的。如：TestApp2://success=1&count=100就是一个URL。

#TestApp2工程中实现代码测试

---
```
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
  NSString *receText = [[url host] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
  NSLog(@"%@    %@",receText, url.absoluteString);
  
  
  [self performSelector:@selector(goBackToApp1) withObject:nil afterDelay:2];
  
  return YES;
  
}

- (void)goBackToApp1 {
  if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"TestApp1://"]]) {
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"TestApp1://paySuccess=1"]];
  }
}
```

我们在`- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url`处理来自`TestApp1`应用的调用，获取到相应的参数了。

当我们处理数据完成，需要反馈给`TestApp1`时，就需要调用通过`TestApp1://paySuccess=1`调用回到`TestApp1`并将状态带回去。

#TestApp1打印日志

---
```
2015-12-03 22:30:10.250 TestApp1[9008:678123] paySuccess=1    TestApp1://paySuccess=1
```

说明参数从`TestApp2`正确的传过来了。

#TestApp2打印日志

---
```
2015-12-03 22:29:59.690 TestApp2[9004:677942] success=1&count=100    TestApp2://success=1&count=100
```

说明参数也能正确地从`TestApp1`传过来了。


#最后

---
最近不少朋友问到我应用之间如何相互调用，又如何传参数的问题，在这里统一讲解了。

#源代码

---
如果单看文章，看不太明白，可以到github下载源代码运行看看效果：[https://github.com/CoderJackyHuang/AppCommunicationDemo](https://github.com/CoderJackyHuang/AppCommunicationDemo)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/ios-app-communication/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/ios-app-communication/)**


