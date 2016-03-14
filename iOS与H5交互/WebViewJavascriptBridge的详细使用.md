#前言

WebViewJavascriptBridge是支持到iOS6之前的版本的，用于支持native的iOS与javascript交互。如果需要支持到iOS6之前的app，使用它是很不错的。本篇讲讲WebViewJavascriptBridge的基本原理及详细讲讲如何去使用，包括iOS端的使用和JS端的使用。

经过多番百度、Google，发现WebViewJavascriptBridge的资源讲解不是翻译官方文档就是直接说官方提供的demo。但是笔者在写这个demo时也遇到了不少问题，也想看看大家是怎么使用的，特别是JS端，弄了好久都没有回调，原来是因为log。

写下本篇文章，希望大家少走弯路吧！

#本Demo效果图

![image](http://www.henishuo.com/wp-content/uploads/2016/03/bridge.gif)

#iOS与H5交互的方案

纵观所有iOS与H5交互的方案，有以下几种：

* 第一种：有很多的app直接使用在webview的代理中通过拦截的方式与native进行交互，通常是通过拦截url scheme判断是否是我们需要拦截处理的url及其所对应的要处理的功能是什么。任意版本都支持。
* 第二种：iOS7之后出了JavaScriptCore.framework用于与JS交互，但是不支持iOS6，对于还需要支持iOS6的app，就不能考虑这个了。若需要了解，看最后的推荐阅读。
* 第三种：WebViewJavascriptBridge开源库使用，本质上，它也是通过webview的代理拦截scheme，然后注入相应的JS。
* 第四种：react-native，这个没玩过（与前三种不同）。

本篇文章专讲讲WebViewJavascriptBridge。

#WebViewJavascriptBridge的基本原理

我们看看WebViewJavascriptBridge.m中Webview代理拦截的代码：

```
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    if (webView != _webView) { return YES; }
    NSURL *url = [request URL];
    __strong WVJB_WEBVIEW_DELEGATE_TYPE* strongDelegate = _webViewDelegate;
    if ([_base isCorrectProcotocolScheme:url]) {
        if ([_base isBridgeLoadedURL:url]) {
            [_base injectJavascriptFile];
        } else if ([_base isQueueMessageURL:url]) {
            NSString *messageQueueString = [self _evaluateJavascript:[_base webViewJavascriptFetchQueyCommand]];
            [_base flushMessageQueue:messageQueueString];
        } else {
            [_base logUnkownMessage:url];
        }
        return NO;
    } else if (strongDelegate && [strongDelegate respondsToSelector:@selector(webView:shouldStartLoadWithRequest:navigationType:)]) {
        return [strongDelegate webView:webView shouldStartLoadWithRequest:request navigationType:navigationType];
    } else {
        return YES;
    }
}
```

在拦截后，通过先通过-isBridgeLoadedURL:方法判断URL是否是需要bridge的URL，若是，则通过injectJavascriptFile方法注入JS；否则判断URL是否是队列消息，若是，则执行查询命令JS并刷新消息队列；最后，URL被识别为未知的消息。

#JS端如何使用

下面是本demo的HTML完整代码：

```
<!doctype html>
<html>
  <head>
  <meta name="viewport" content="user-scalable=no, width=device-width, initial-scale=1.0, maximum-scale=1.0">
    <style type='text/css'>
      html { font-family:Helvetica; color:#222; }
      h1 { color:steelblue; font-size:24px; margin-top:24px; }
      button { margin:0 3px 10px; font-size:12px; }
      .logLine { border-bottom:1px solid #ccc; padding:4px 2px; font-family:courier; font-size:11px; }
    </style>
  </head>
  
  <body>
    <h1>WebViewJavascriptBridge Demo</h1>
    
    <script>
      window.onerror = function(err) {
        log('window.onerror: ' + err)
      }
    
      /*这段代码是固定的，必须要放到js中*/
      function setupWebViewJavascriptBridge(callback) {
        if (window.WebViewJavascriptBridge) { return callback(WebViewJavascriptBridge); }
        if (window.WVJBCallbacks) { return window.WVJBCallbacks.push(callback); }
        window.WVJBCallbacks = [callback];
        var WVJBIframe = document.createElement('iframe');
        WVJBIframe.style.display = 'none';
        WVJBIframe.src = 'wvjbscheme://__BRIDGE_LOADED__';
        document.documentElement.appendChild(WVJBIframe);
        setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
      }
    
      /*与OC交互的所有JS方法都要放在此处注册，才能调用通过JS调用OC或者让OC调用这里的JS*/
      setupWebViewJavascriptBridge(function(bridge) {
       var uniqueId = 1
       function log(message, data) {
         var log = document.getElementById('log')
         var el = document.createElement('div')
         el.className = 'logLine'
         el.innerHTML = uniqueId++ + '. ' + message + ':<br/>' + JSON.stringify(data)
         if (log.children.length) {
            log.insertBefore(el, log.children[0])
         } else {
           log.appendChild(el)
         }
       }
       /* Initialize your app here */
       
       /*我们在这注册一个js调用OC的方法，不带参数，且不用ObjC端反馈结果给JS：打开本demo对应的博文*/
       bridge.registerHandler('openWebviewBridgeArticle', function() {
          log("openWebviewBridgeArticle was called with by ObjC")
       })
       /*JS给ObjC提供公开的API，在ObjC端可以手动调用JS的这个API。接收ObjC传过来的参数，且可以回调ObjC*/
       bridge.registerHandler('getUserInfos', function(data, responseCallback) {
         log("Get user information from ObjC: ", data)
         responseCallback({'userId': '123456', 'blog': '标哥的技术博客'})
       })
                                   
       /*JS给ObjC提供公开的API，ObjC端通过注册，就可以在JS端调用此API时，得到回调。ObjC端可以在处理完成后，反馈给JS，这样写就是在载入页面完成时就先调用*/
       bridge.callHandler('getUserIdFromObjC', function(responseData) {
         log("JS call ObjC's getUserIdFromObjC function, and js received response:", responseData)
       })

       document.getElementById('blogId').onclick = function (e) {
         log('js call objc: getBlogNameFromObjC')
         bridge.callHandler('getBlogNameFromObjC', {'blogURL': 'http://www.henishuo.com'}, function(response) {
                          log('JS got response', response)
                          })
       }
     })
       
    </script>
    
    <div id='buttons'></div> <div id='log'></div>
    
    <div>
       <input type="button" value="getBlogNameFromObjC" id="blogId"/>
    </div>
  </body>
</html>
```

在JS端，嵌入步骤是：

* 第一步：将下面的代码放在JS中：

```
/*这段代码是固定的，必须要放到js中*/
function setupWebViewJavascriptBridge(callback) {
    if (window.WebViewJavascriptBridge) { return callback(WebViewJavascriptBridge); }
    if (window.WVJBCallbacks) { return window.WVJBCallbacks.push(callback); }
    window.WVJBCallbacks = [callback];
    var WVJBIframe = document.createElement('iframe');
    WVJBIframe.style.display = 'none';
    WVJBIframe.src = 'wvjbscheme://__BRIDGE_LOADED__';
    document.documentElement.appendChild(WVJBIframe);
    setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
}
```
这段代码是固定的，必须要放到js中。

* 第二步：在下面的方法体里写相关JS代码：

```
setupWebViewJavascriptBridge(function(bridge) {
    /* Initialize your app here */
    所有与iOS交互的JS代码放这里！
})
```

###JS如何调用iOS代码

通过bridge.callHandler来调用：

```
bridge.callHandler('getBlogNameFromObjC', 
                 {'blogURL': 'http://www.henishuo.com'}, 
                 function callback(response) {
                   log('JS got response', response)
                 }
})
```

其中，各参数说明如下：

* getBlogNameFromObjC：是iOS端register的handleName，在iOS端注册后，JS就可以直接通过这个handleName与iOS交互。比如，当点击某个按钮时，就执行上面这一段代码，结果就是js端将参数传给了iOS端，iOS端收到参数，然后通过回调给js，js会收到response，然后打印出来
* {'blogURL': 'http://www.henishuo.com'}：这是JSON字符串，传到iOS端会被WebViewJavascriptBridge自动转换成id对象，然后在回调处看到的就是字典对象了。
* callback：这是个js函数名，在iOS端收到回调后，拿到了参数，然后通过闭包回调反馈给js端，这个反馈就是通过callback函数来传值给js。

###JS端加入WebViewJavascriptBridge代码注意事项

如果在下面的函数体内有任何错误，都不会有打印日志，也不会有任何回调：

```
setupWebViewJavascriptBridge(function(bridge) {
    /* Initialize your app here */
    所有与iOS交互的JS代码放这里！
})
```

因此，如果遇到什么也没有输出，说明你写错了。另外，上面有demo中，log函数是自定义的，不是系统的，因此如果没有加入这个函数的定义，调用它也会导致不能交互。

#iOS端如何使用

* 第一步：开启日志

```
// 开启日志，方便调试
[WebViewJavascriptBridge enableLogging];
```

* 第二步：给ObjC与JS建立桥梁

```
// 给哪个webview建立JS与OjbC的沟通桥梁
self.bridge = [WebViewJavascriptBridge bridgeForWebView:webView];

// 设置代理，如果不需要实现，可以不设置
[self.bridge setWebViewDelegate:self];
```

* 第三步：注册HandleName，用于给JS端调用iOS端

```
// JS主动调用OjbC的方法
// 这是JS会调用getUserIdFromObjC方法，这是OC注册给JS调用的
// JS需要回调，当然JS也可以传参数过来。data就是JS所传的参数，不一定需要传
// OC端通过responseCallback回调JS端，JS就可以得到所需要的数据
[self.bridge registerHandler:@"getUserIdFromObjC" handler:^(id data, WVJBResponseCallback responseCallback) {
	NSLog(@"js call getUserIdFromObjC, data from js is %@", data);
	if (responseCallback) {
	  // 反馈给JS
	  responseCallback(@{@"userId": @"123456"});
	}
}];
  
[self.bridge registerHandler:@"getBlogNameFromObjC" handler:^(id data, WVJBResponseCallback responseCallback) {
	NSLog(@"js call getBlogNameFromObjC, data from js is %@", data);
	if (responseCallback) {
	  // 反馈给JS
	  responseCallback(@{@"blogName": @"标哥的技术博客"});
	}
}];
```


* 第四步：直接调用JS端注册的HandleName

```
[self.bridge callHandler:@"getUserInfos" data:@{@"name": @"标哥"} responseCallback:^(id responseData) {
	NSLog(@"from js: %@", responseData);
}];
```

#源代码

大家可以到GITHUB下载源代码，喜欢就随手给个Star：[WebViewJavascriptBridgeDemo](https://github.com/CoderJackyHuang/WebViewJavascriptBridgeDemo)

#最后

由于很多朋友不太会使用这个库，再加上群里的小伙伴们经常问这个问题，因此决定写一篇文章来教大家如何去使用。其实在此之前，笔者也没有使用过这个东西。

使用这个库有什么缺点？最明显的就是要固定加入相关代码，而且JS代码要放在固定的函数内添加。那么安卓端是否也支持这样呢？如果不支持，那么H5端要分别判断ios、安卓？由于没有在公司使用过，因此只能带着这些疑问，究竟是否通用。

如果大家在公司使用过，那么一定了解安卓是否兼容。还请大家给笔者在评论中说说！谢谢！

#推荐阅读

* [OC JavaScriptCore与js交互](http://www.henishuo.com/oc-js/)
* [Swift JavaScriptCore与js交互](http://www.henishuo.com/category/ios-h5-js/)
* [WKWebView新特性及JS交互](http://www.henishuo.com/wkwebview-js/)
* [ObjC点击H5图片Native预览](http://www.henishuo.com/objc-html-img-preview/)

#阅读原文

[阅读原文](http://www.henishuo.com/webviewjavascriptbridge-detail-use/)
