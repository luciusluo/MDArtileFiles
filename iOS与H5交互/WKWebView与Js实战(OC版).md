#前言

上一篇专门讲解了WKWebView相关的所有类、代理的所有API。那么本篇讲些什么呢？当然是实战了！

本篇文章教大家如何使用WKWebView去实现常用的一些API操作。当然，也会有如何与JS交互的实战。

如果还没有阅读过[WKWebView精讲(OC版)](http://www.henishuo.com/wkwebview-objc/)，请先阅读，不然有可能看不懂下面所讲的内容。

#效果图

![image](http://www.henishuo.com/wp-content/uploads/2016/03/wkjs.gif)

通过本篇文章，至少可以学习到：

* OC如何给JS注入对象及JS如何给IOS发送数据
* JS调用alert、confirm、prompt时，不采用JS原生提示，而是使用iOS原生来实现
* 如何监听web内容加载进度、是否加载完成
* 如何处理去跨域问题

#创建配置类

在创建WKWebView之前，需要先创建配置对象，用于做一些配置：

```
WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
```

###配置偏好设置

偏好设置也没有必须去修改它，都使用默认的就可以了，除非你真的需要修改它：

```
// 设置偏好设置
config.preferences = [[WKPreferences alloc] init];
// 默认为0
config.preferences.minimumFontSize = 10;
// 默认认为YES
config.preferences.javaScriptEnabled = YES;
// 在iOS上默认为NO，表示不能自动通过窗口打开
config.preferences.javaScriptCanOpenWindowsAutomatically = NO;
```


###配置web内容处理池

其实我们没有必要去创建它，因为它根本没有属性和方法：

```
// web内容处理池，由于没有属性可以设置，也没有方法可以调用，不用手动创建
config.processPool = [[WKProcessPool alloc] init];
```

###配置Js与Web内容交互

WKUserContentController是用于给JS注入对象的，注入对象后，JS端就可以使用：

```
window.webkit.messageHandlers.<name>.postMessage(<messageBody>) 
```

来调用发送数据给iOS端，比如：

```
window.webkit.messageHandlers.AppModel.postMessage({body: '传数据'});
```

AppModel就是我们要注入的名称，注入以后，就可以在JS端调用了，传数据统一通过body传，可以是多种类型，只支持NSNumber, NSString, NSDate, NSArray,NSDictionary, and NSNull类型。

下面我们配置给JS的main frame注入AppModel名称，对于JS端可就是对象了：

```
// 通过JS与webview内容交互
config.userContentController = [[WKUserContentController alloc] init];

// 注入JS对象名称AppModel，当JS通过AppModel来调用时，
// 我们可以在WKScriptMessageHandler代理中接收到
[config.userContentController addScriptMessageHandler:self name:@"AppModel"];
```

当JS通过AppModel发送数据到iOS端时，会在代理中收到：

```
#pragma mark - WKScriptMessageHandler
- (void)userContentController:(WKUserContentController *)userContentController
      didReceiveScriptMessage:(WKScriptMessage *)message {
  if ([message.name isEqualToString:@"AppModel"]) {
    // 打印所传过来的参数，只支持NSNumber, NSString, NSDate, NSArray,
    // NSDictionary, and NSNull类型
    NSLog(@"%@", message.body);
  }
}
```

所有JS调用iOS的部分，都只可以在此处使用哦。当然我们也可以注入多个名称（JS对象），用于区分功能。

#创建WKWebView

通过唯一的默认构造器来创建对象：

```
self.webView = [[WKWebView alloc] initWithFrame:self.view.bounds
                                configuration:config];
[self.view addSubview:self.webView];                      
```

###加载H5页面

```
NSURL *path = [[NSBundle mainBundle] URLForResource:@"test" withExtension:@"html"];
[self.webView loadRequest:[NSURLRequest requestWithURL:path]];
```

###配置代理 

如果需要处理web导航条上的代理处理，比如链接是否可以跳转或者如何跳转，需要设置代理；而如果需要与在JS调用alert、confirm、prompt函数时，通过JS原生来处理，而不是调用JS的alert、confirm、prompt函数，那么需要设置UIDelegate，在得到响应后可以将结果反馈到JS端：

```
// 导航代理
self.webView.navigationDelegate = self;
// 与webview UI交互代理
self.webView.UIDelegate = self;
```

###添加对WKWebView属性的监听

WKWebView有好多个支持KVO的属性，这里只是监听loading、title、estimatedProgress属性，分别用于判断是否正在加载、获取页面标题、当前页面载入进度：

```
// 添加KVO监听
[self.webView addObserver:self
             forKeyPath:@"loading"
                options:NSKeyValueObservingOptionNew
                context:nil];
[self.webView addObserver:self
             forKeyPath:@"title"
                options:NSKeyValueObservingOptionNew
                context:nil];
[self.webView addObserver:self
             forKeyPath:@"estimatedProgress"
                options:NSKeyValueObservingOptionNew
                context:nil];
  
```

然后我们就可以实现KVO处理方法，在loading完成时，可以注入一些JS到web中。这里只是简单地执行一段web中的JS函数：

```
#pragma mark - KVO
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary<NSString *,id> *)change
                       context:(void *)context {
  if ([keyPath isEqualToString:@"loading"]) {
    NSLog(@"loading");
  } else if ([keyPath isEqualToString:@"title"]) {
    self.title = self.webView.title;
  } else if ([keyPath isEqualToString:@"estimatedProgress"]) {
    NSLog(@"progress: %f", self.webView.estimatedProgress);
    self.progressView.progress = self.webView.estimatedProgress;
  }
  
  // 加载完成
  if (!self.webView.loading) {
    // 手动调用JS代码
    // 每次页面完成都弹出来，大家可以在测试时再打开
    NSString *js = @"callJsAlert()";
    [self.webView evaluateJavaScript:js completionHandler:^(id _Nullable response, NSError * _Nullable error) {
      NSLog(@"response: %@ error: %@", response, error);
      NSLog(@"call js alert by native");
    }];
    
    [UIView animateWithDuration:0.5 animations:^{
      self.progressView.alpha = 0;
    }];
  }
}

```

#WKUIDelegate

与JS原生的alert、confirm、prompt交互，将弹出来的实际上是我们原生的窗口，而不是JS的。在得到数据后，由原生传回到JS：

```
#pragma mark - WKUIDelegate
- (void)webViewDidClose:(WKWebView *)webView {
     NSLog(@"%s", __FUNCTION__);
}

// 在JS端调用alert函数时，会触发此代理方法。
// JS端调用alert时所传的数据可以通过message拿到
// 在原生得到结果后，需要回调JS，是通过completionHandler回调
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler {
  NSLog(@"%s", __FUNCTION__);
  UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"alert" message:@"JS调用alert" preferredStyle:UIAlertControllerStyleAlert];
  [alert addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    completionHandler();
  }]];
  
  [self presentViewController:alert animated:YES completion:NULL];
  NSLog(@"%@", message);
}

// JS端调用confirm函数时，会触发此方法
// 通过message可以拿到JS端所传的数据
// 在iOS端显示原生alert得到YES/NO后
// 通过completionHandler回调给JS端
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL result))completionHandler {
  NSLog(@"%s", __FUNCTION__);
  
  UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"confirm" message:@"JS调用confirm" preferredStyle:UIAlertControllerStyleAlert];
  [alert addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    completionHandler(YES);
  }]];
  [alert addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
    completionHandler(NO);
  }]];
  [self presentViewController:alert animated:YES completion:NULL];
  
  NSLog(@"%@", message);
}

// JS端调用prompt函数时，会触发此方法
// 要求输入一段文本
// 在原生输入得到文本内容后，通过completionHandler回调给JS
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(nullable NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * __nullable result))completionHandler {
  NSLog(@"%s", __FUNCTION__);
  
  NSLog(@"%@", prompt);
  UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"textinput" message:@"JS调用输入框" preferredStyle:UIAlertControllerStyleAlert];
  [alert addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
    textField.textColor = [UIColor redColor];
  }];
  
  [alert addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    completionHandler([[alert.textFields lastObject] text]);
  }]];
  
  [self presentViewController:alert animated:YES completion:NULL];
}
```

#WKNavigationDelegate

如果需要处理web导航操作，比如链接跳转、接收响应、在导航开始、成功、失败等时要做些处理，就可以通过实现相关的代理方法：

```
#pragma mark - WKNavigationDelegate
// 请求开始前，会先调用此代理方法
// 与UIWebView的
// - (BOOL)webView:(UIWebView *)webView 
// shouldStartLoadWithRequest:(NSURLRequest *)request 
// navigationType:(UIWebViewNavigationType)navigationType;
// 类型，在请求先判断能不能跳转（请求）
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
  NSString *hostname = navigationAction.request.URL.host.lowercaseString;
  if (navigationAction.navigationType == WKNavigationTypeLinkActivated
      && ![hostname containsString:@".baidu.com"]) {
// 对于跨域，需要手动跳转
    [[UIApplication sharedApplication] openURL:navigationAction.request.URL];
    
    // 不允许web内跳转
    decisionHandler(WKNavigationActionPolicyCancel);
  } else {
    self.progressView.alpha = 1.0;
    decisionHandler(WKNavigationActionPolicyAllow);
  }
  
    NSLog(@"%s", __FUNCTION__);
}

// 在响应完成时，会回调此方法
// 如果设置为不允许响应，web内容就不会传过来
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler {
  decisionHandler(WKNavigationResponsePolicyAllow);
  NSLog(@"%s", __FUNCTION__);
}

// 开始导航跳转时会回调
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(null_unspecified WKNavigation *)navigation {
    NSLog(@"%s", __FUNCTION__);
}

// 接收到重定向时会回调
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(null_unspecified WKNavigation *)navigation {
    NSLog(@"%s", __FUNCTION__);
}

// 导航失败时会回调
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error {
    NSLog(@"%s", __FUNCTION__);
}

// 页面内容到达main frame时回调
- (void)webView:(WKWebView *)webView didCommitNavigation:(null_unspecified WKNavigation *)navigation {
    NSLog(@"%s", __FUNCTION__);
}

// 导航完成时，会回调（也就是页面载入完成了）
- (void)webView:(WKWebView *)webView didFinishNavigation:(null_unspecified WKNavigation *)navigation {
    NSLog(@"%s", __FUNCTION__);
}

// 导航失败时会回调
- (void)webView:(WKWebView *)webView didFailNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error {
  
}

// 对于HTTPS的都会触发此代理，如果不要求验证，传默认就行
// 如果需要证书验证，与使用AFN进行HTTPS证书验证是一样的
- (void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential *__nullable credential))completionHandler {
    NSLog(@"%s", __FUNCTION__);
  completionHandler(NSURLSessionAuthChallengePerformDefaultHandling, nil);
}

// 9.0才能使用，web内容处理中断时会触发
- (void)webViewWebContentProcessDidTerminate:(WKWebView *)webView {
    NSLog(@"%s", __FUNCTION__);
}
```

#JS端代码

```
<!DOCTYPE html>
<html>
  <head>
    <title>iOS and Js</title>
    <style type="text/css">
      * {
        font-size: 40px;
      }
    </style>
  </head>
  
  <body>
    
    <div style="margin-top: 100px">
      <h1>Test how to use objective-c call js</h1><br/>
      <div><input type="button" value="call js alert" onclick="callJsAlert()"></div>
      <br/>
      <div><input type="button" value="Call js confirm" onclick="callJsConfirm()"></div><br/>
    </div>
    <br/>
    <div>
      <div><input type="button" value="Call Js prompt " onclick="callJsInput()"></div><br/>
      <div>Click me here: <a href="http://www.baidu.com">Jump to Baidu</a></div>
    </div>
    
    <br/>
    <div id="SwiftDiv">
      <span id="jsParamFuncSpan" style="color: red; font-size: 50px;"></span>
    </div>
    
    <script type="text/javascript">
      function callJsAlert() {
        alert('Objective-C call js to show alert');
        
        window.webkit.messageHandlers.AppModel.postMessage({body: 'call js alert in js'});
      }
    
    function callJsConfirm() {
      if (confirm('confirm', 'Objective-C call js to show confirm')) {
        document.getElementById('jsParamFuncSpan').innerHTML
        = 'true';
      } else {
        document.getElementById('jsParamFuncSpan').innerHTML
        = 'false';
      }
      
      // AppModel是我们所注入的对象
      window.webkit.messageHandlers.AppModel.postMessage({body: 'call js confirm in js'});
    }
    
    function callJsInput() {
      var response = prompt('Hello', 'Please input your name:');
      document.getElementById('jsParamFuncSpan').innerHTML = response;
      
       // AppModel是我们所注入的对象
      window.webkit.messageHandlers.AppModel.postMessage({body: response});
    }
   </script>
  </body>
</html>
```

#源代码

下载源代码：[WKWebViewH5ObjCDemo](https://github.com/CoderJackyHuang/WKWebViewH5ObjCDemo)

关注GITHUB：[CoderJackyHuang](https://github.com/CoderJackyHuang)

#推荐阅读

* [WKWebView精讲-OC版](http://www.henishuo.com/wkwebview-objc/)
* [WebViewJavascriptBridge详细使用](http://www.henishuo.com/webviewjavascriptbridge-detail-use/)
* [OC JavaScriptCore与js交互](http://www.henishuo.com/oc-js/)
* [Swift WKWebView新特性及JS交互](http://www.henishuo.com/wkwebview-js/)


