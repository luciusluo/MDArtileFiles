#引言

一直听说`WKWebView`比`UIWebView`强大许多，可是一直没有使用到，今天花了点时间看写了个例子，对其API的使用有所了解，为了日后能少走弯路，也为了让大家更容易学习上手，这里写下这篇文章来记录如何使用以及需要注意的地方。

**温馨提示：**本人在学习使用过程中，确实有此体会，`WKWebView`的确比`UIWebView`强大很多，与JS交互的能力显示增强，在加载速度上有所提升。

#WKWebView新特性

* 性能、稳定性、功能大幅度提升
* 允许JavaScript的Nitro库加载并使用（UIWebView中限制）
* 支持了更多的HTML5特性
* 高达60fps的滚动刷新率以及内置手势
* GPU硬件加速
* KVO
* 重构UIWebView成14类与3个协议，[查看官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/WebKit/ObjC_classic/index.html)

#准备工作

首先，我们在使用的地方引入模块：

```
import Webkit
```

在学习之前，建议大家先点击`WKWebView`进去阅读里面的相关API，读完一遍，有个大概的印象，学习起来就很快了。

其初始化方法有：

```
public init()
public init(frame: CGRect)
public init(frame: CGRect, configuration: WKWebViewConfiguration)
```

加载HTML的方式与`UIWebView`的方式大致相同。其中，`loadFileURL`方法通常用于加载服务器的HTML页面或者JS，而`loadHTMLString`方法通常用于加载本地HTML或者JS：

```
public func loadRequest(request: NSURLRequest) -> WKNavigation?
 
// 9.0以后才支持   
@available(iOS 9.0, *)
public func loadFileURL(URL: NSURL, allowingReadAccessToURL readAccessURL: NSURL) -> WKNavigation?
 
// 通常用于加载本地HTML或者JS
public func loadHTMLString(string: String, baseURL: NSURL?) -> WKNavigation?
    
// 9.0以后才支持
@available(iOS 9.0, *)
public func loadData(data: NSData, MIMEType: String, characterEncodingName: String, baseURL: NSURL) -> WKNavigation
```

与之交互用到的三大代理：

* WKNavigationDelegate，与页面导航加载相关
* WKUIDelegate，与JS交互时的ui展示相关，比较JS的alert、confirm、prompt
* WKScriptMessageHandler，与js交互相关，通常是ios端注入名称，js端通过window.webkit.messageHandlers.{NAME}.postMessage()来发消息到ios端

#创建WKWebView

首先，我们在`ViewController`中先遵守协议：

```
class ViewController: UIViewController, WKScriptMessageHandler, WKNavigationDelegate, WKUIDelegate
```

我们可以先创建一个`WKWebView`的配置项`WKWebViewConfiguration`，这可以设置偏好设置，与网页交互的配置，注入对象，注入js等：

```
// 创建一个webiview的配置项
let configuretion = WKWebViewConfiguration()
    
// Webview的偏好设置
configuretion.preferences = WKPreferences()
configuretion.preferences.minimumFontSize = 10
configuretion.preferences.javaScriptEnabled = true
// 默认是不能通过JS自动打开窗口的，必须通过用户交互才能打开
configuretion.preferences.javaScriptCanOpenWindowsAutomatically = false
    
// 通过js与webview内容交互配置
configuretion.userContentController = WKUserContentController()
    
// 添加一个JS到HTML中，这样就可以直接在JS中调用我们添加的JS方法
let script = WKUserScript(source: "function showAlert() { alert('在载入webview时通过Swift注入的JS方法'); }",
  injectionTime: .AtDocumentStart,// 在载入时就添加JS
  forMainFrameOnly: true) // 只添加到mainFrame中
configuretion.userContentController.addUserScript(script)
    
// 添加一个名称，就可以在JS通过这个名称发送消息：
// window.webkit.messageHandlers.AppModel.postMessage({body: 'xxx'})
configuretion.userContentController.addScriptMessageHandler(self, name: "AppModel")
```

创建对象并遵守代理：

```
self.webView = WKWebView(frame: self.view.bounds, configuration: configuretion)

self.webView.navigationDelegate = self
self.webView.UIDelegate = self
```

加载我们的本地HTML页面：

```
let url = NSBundle.mainBundle().URLForResource("test", withExtension: "html")
self.webView.loadRequest(NSURLRequest(URL: url!))
self.view.addSubview(self.webView);
```

我们再添加前进、后退按钮和添加一个加载进度的控制显示在Webview上：

```
self.progressView = UIProgressView(progressViewStyle: .Default)
self.progressView.frame.size.width = self.view.frame.size.width
self.progressView.backgroundColor = UIColor.redColor()
self.view.addSubview(self.progressView)
    
self.navigationItem.leftBarButtonItem = UIBarButtonItem(title: "上一个页面", style: .Done, target: self, action: "previousPage")
self.navigationItem.rightBarButtonItem = UIBarButtonItem(title: "下一个页面", style: .Done, target: self, action: "nextPage")
```

#页面前进、后退

对于前进后退的事件处理就很简单的，要注意判断一下是否可以后退、前进才调用：

```
func previousPage() {
	if self.webView.canGoBack {
	  self.webView.goBack()
	}
}
  
func nextPage() {
	if self.webView.canGoForward {
	  self.webView.goForward()
	}
}
```

当然除了这些方法之处，还有重新载入等。

#WKWebView的KVO

对于`WKWebView`，有三个属性支持KVO，因此我们可以监听其值的变化，分别是：`loading`,`title`,`estimatedProgress`，对应功能表示为：是否正在加载中，页面的标题，页面内容加载的进度（值为0.0~1.0）

```
// 监听支持KVO的属性
self.webView.addObserver(self, forKeyPath: "loading", options: .New, context: nil)
self.webView.addObserver(self, forKeyPath: "title", options: .New, context: nil)
self.webView.addObserver(self, forKeyPath: "estimatedProgress", options: .New, context: nil)
```

然后就可以重写监听的方法来处理。这里只是取页面的标题，更新加载的进度条，在加载完成时，手动调用执行一个JS方法：

```
// MARK: - KVO
override func observeValueForKeyPath(keyPath: String?, ofObject object: AnyObject?, change: [String : AnyObject]?, context: UnsafeMutablePointer<Void>) {
  if keyPath == "loading" {
    print("loading")
  } else if keyPath == "title" {
    self.title = self.webView.title
  } else if keyPath == "estimatedProgress" {
    print(webView.estimatedProgress)
    self.progressView.setProgress(Float(webView.estimatedProgress), animated: true)
  }
  
  // 已经完成加载时，我们就可以做我们的事了
  if !webView.loading {
    // 手动调用JS代码
    let js = "callJsAlert()";
    self.webView.evaluateJavaScript(js) { (_, _) -> Void in
      print("call js alert")
    }
    
    UIView.animateWithDuration(0.55, animations: { () -> Void in
      self.progressView.alpha = 0.0;
    })
  }
}
```

#WKUIDelegate
我们看看`WKUIDelegate`的几个代理方法，虽然不是必须实现的，但是如果我们的页面中有调用了js的alert、confirm、prompt方法，我们应该实现下面这几个代理方法，然后在原来这里调用native的弹出窗，因为使用`WKWebView`后，HTML中的alert、confirm、prompt方法调用是不会再弹出窗口了，只是转化成ios的native回调代理方法：

```
// MARK: - WKUIDelegate
// 这个方法是在HTML中调用了JS的alert()方法时，就会回调此API。
// 注意，使用了`WKWebView`后，在JS端调用alert()就不会在HTML
// 中显示弹出窗口。因此，我们需要在此处手动弹出ios系统的alert。
func webView(webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: () -> Void) {
  let alert = UIAlertController(title: "Tip", message: message, preferredStyle: .Alert)
  alert.addAction(UIAlertAction(title: "Ok", style: .Default, handler: { (_) -> Void in
    // We must call back js
    completionHandler()
  }))
  
  self.presentViewController(alert, animated: true, completion: nil)
}

func webView(webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: (Bool) -> Void) {
  let alert = UIAlertController(title: "Tip", message: message, preferredStyle: .Alert)
  alert.addAction(UIAlertAction(title: "Ok", style: .Default, handler: { (_) -> Void in
    // 点击完成后，可以做相应处理，最后再回调js端
    completionHandler(true)
  }))
  alert.addAction(UIAlertAction(title: "Cancel", style: .Cancel, handler: { (_) -> Void in
    // 点击取消后，可以做相应处理，最后再回调js端
    completionHandler(false)
  }))
  
  self.presentViewController(alert, animated: true, completion: nil)
}

func webView(webView: WKWebView, runJavaScriptTextInputPanelWithPrompt prompt: String, defaultText: String?, initiatedByFrame frame: WKFrameInfo, completionHandler: (String?) -> Void) {
  let alert = UIAlertController(title: prompt, message: defaultText, preferredStyle: .Alert)
  
  alert.addTextFieldWithConfigurationHandler { (textField: UITextField) -> Void in
    textField.textColor = UIColor.redColor()
  }
  alert.addAction(UIAlertAction(title: "Ok", style: .Default, handler: { (_) -> Void in
     // 处理好之前，将值传到js端
    completionHandler(alert.textFields![0].text!)
  }))
  
  self.presentViewController(alert, animated: true, completion: nil)
}

func webViewDidClose(webView: WKWebView) {
  print(__FUNCTION__)
}
```

#WKScriptMessageHandler
接下来，我们看看`WKScriptMessageHandler`，这个是注入js名称，在js端通过`window.webkit.messageHandlers.{InjectedName}.postMessage()`方法来发送消息到native。我们需要遵守此协议，然后实现其代理方法，就可以收到消息，并做相应处理。这个协议只有一个方法：

```
// MARK: - WKScriptMessageHandler
func userContentController(userContentController: WKUserContentController, didReceiveScriptMessage message: WKScriptMessage) {
  print(message.body)
  // 如果在开始时就注入有很多的名称，那么我们就需要区分来处理
  if message.name == "AppModel" {
    print("message name is AppModel")
  }
}
```
这个方法是相当好的API，我们给js注入一个名称，就会自动转换成js的对象，然后就可以发送消息到native端。

#WKNavigationDelegate
还有一个非常关键的代理`WKNavigationDelegate`，这个代理有很多的代理方法，可以控制页面导航。

其调用顺序为：
1、这个代理方法是用于处理是否允许跳转导航。对于跨域只有Safari浏览器才允许，其他浏览器是不允许的，因此我们需要额外处理跨域的链接。

```
// 决定导航的动作，通常用于处理跨域的链接能否导航。WebKit对跨域进行了安全检查限制，不允许跨域，因此我们要对不能跨域的链接
// 单独处理。但是，对于Safari是允许跨域的，不用这么处理。
func webView(webView: WKWebView, decidePolicyForNavigationAction navigationAction: WKNavigationAction, decisionHandler: (WKNavigationActionPolicy) -> Void) {
	print(__FUNCTION__)
	    
	let hostname = navigationAction.request.URL?.host?.lowercaseString
	    
	print(hostname)
	// 处理跨域问题
	if navigationAction.navigationType == .LinkActivated && !hostname!.containsString(".baidu.com") {
	  // 手动跳转
	  UIApplication.sharedApplication().openURL(navigationAction.request.URL!)
	  
	  // 不允许导航
	  decisionHandler(.Cancel)
	} else {
	  self.progressView.alpha = 1.0
	  
	  decisionHandler(.Allow)
	}
}
```

2、开始加载页面内容时就会回调此代理方法，与`UIWebView`的`didStartLoad`功能相当

```
func webView(webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!) {
	print(__FUNCTION__)
}
```

3、决定是否允许导航响应，如果不允许就不会跳转到该链接的页面。

```
func webView(webView: WKWebView, decidePolicyForNavigationResponse navigationResponse: WKNavigationResponse, decisionHandler: (WKNavigationResponsePolicy) -> Void) {
	print(__FUNCTION__)
	decisionHandler(.Allow)
}
```

4、Invoked when content starts arriving for the main frame.这是API的原注释。也就是在页面内容加载到达mainFrame时会回调此API。如果我们要在mainFrame中注入什么JS，也可以在此处添加。

```
func webView(webView: WKWebView, didCommitNavigation navigation: WKNavigation!) {
  print(__FUNCTION__)
}
```

5、加载完成的回调

```
func webView(webView: WKWebView, didFinishNavigation navigation: WKNavigation!) {
  print(__FUNCTION__)
}
```

如果加载失败了，会回调下面的代理方法：

```
func webView(webView: WKWebView, didFailNavigation navigation: WKNavigation!, withError error: NSError) {
  print(__FUNCTION__)
}
```

其实在还有一些API，一般情况下并不需要。如果我们需要处理在重定向时，需要实现下面的代理方法就可以接收到。

```
func webView(webView: WKWebView, didReceiveServerRedirectForProvisionalNavigation navigation: WKNavigation!) {
  print(__FUNCTION__)
}
```

如果我们的请求要求授权、证书等，我们需要处理下面的代理方法，以提供相应的授权处理等：

```
func webView(webView: WKWebView, didReceiveAuthenticationChallenge challenge: NSURLAuthenticationChallenge, completionHandler: (NSURLSessionAuthChallengeDisposition, NSURLCredential?) -> Void) {
    print(__FUNCTION__)
	completionHandler(.PerformDefaultHandling, nil)
}
```

当我们终止页面加载时，我们会可以处理下面的代理方法，如果不需要处理，则不用实现之：

```
func webViewWebContentProcessDidTerminate(webView: WKWebView) {
    print(__FUNCTION__)
}
```

#源代码

具体代码已经发布到github：[https://github.com/CoderJackyHuang/WKWebViewTestDemo](https://github.com/CoderJackyHuang/WKWebViewTestDemo)

#总结
苹果已经向我们提供了`WKWebView`，拥有`UIWebView`的所有功能，且还提供更多的功能，明示是为了替代`UIWebView`，但是`WKWebView`要在ios8.0之后才能使用，因此，如果我们使用Swift来开发应用，兼容版本从8.0开始时，可以直接使用`WKWebView`。

我们可以发现，苹果提供了更多简便的方式让native与js交互更加方便，通过让native注入名称，然后在js端自动转换成js的对象，就可以在js端通过对象的方式来发送消息到native端。如此一来，就简化了js与native的交互了。


#推荐阅读

* [Swift JavaScriptCore与JS交互](http://www.henishuo.com/swift-js/)
* [OC JavaScriptCore与js交互](http://www.henishuo.com/oc-js/)


