##前言

Swift与Js交互是常见的需求，可对于新手或者所谓的高手而言，其实并不是那么简单明了。这里只介绍iOS7.0后出来的JavaScriptCore framework。

##关于JavaScriptCore

本教程中所涉及到的几种类型：

* **JSContext**, JSContext是代表JS的执行环境，通过-evaluateScript:方法就可以执行一JS代码
* **JSValue**, JSValue封装了JS与ObjC中的对应的类型，以及调用JS的API等
* **JSExport**, JSExport是一个协议，遵守此协议，就可以定义我们自己的协议，在协议中声明的API都会在JS中暴露出来，才能调用


##Swift与JS交互方式

通过JSContext，我们有两种调用JS代码的方法：

* 1、直接调用JS代码
* 2、在Swift中通过JSContext**注入模型**，然后调用模型的方法


###直接调用JS代码
我们可以不通过模型来调用方法，也可以直接调用方法

```
let context = JSContext() 
context.evaluateScript(“var num = 10”)
context.evaluateScript(“function square(value) { return value * 2}”)

// 直接调用
let squareValue = context.evaluateScript(“square(num)”)
print(squareValue)

// 通过下标来获取到JS方法。
let squareFunc = context.objectForKeyedSubscript(“square”)
print(squareFunc.callWithArguments([“10”]).toString());
```

这种方式是没有注入模型到JS中的。这种方式使用起来不太合适，通常在JS中有很多全局的函数，为了防止名字重名，使用模型的方式是最好不过了。通过我们协商好的模型名称，在JS中直接通过模型来调用我们在Swift中所定义的模型所公开的API。

##注入模型的交互
首先，我们需要先定义一个协议，而且这个协议必须要遵守JSExport协议。

All methods that should apply in Javascript,should be in the following protocol.注意，这里必须使用@objc，因为JavaScriptCore库是ObjectiveC版本的。如果不加@objc，则调用无效果。

```
@objc protocol JavaScriptSwiftDelegate: JSExport { 
 func callSystemCamera();
 func showAlert(title: String, msg: String);
 func callWithDict(dict: [String: AnyObject]);
 func jsCallObjcAndObjcCallJsWithDict(dict: [String: AnyObject]);
}
```

接下来，我们还需要定义一个模型：

```
@objc classJSObjCModel: NSObject, JavaScriptSwiftDelegate { 
  weak var controller: UIViewController? 
  weak var jsContext: JSContext? 
  
  func callSystemCamera() {   
    print(“js call objc method: callSystemCamera”);
    let jsFunc = self.jsContext?.objectForKeyedSubscript(“jsFunc”); 
    jsFunc?.callWithArguments([]);  
  }

  func showAlert(title: String, msg: String) {     
    dispatch_async(dispatch_get_main_queue()) { () -> Void in   
      let alert = UIAlertController(title: title, message: msg, preferredStyle: .Alert)   
      alert.addAction(UIAlertAction(title: “ok”, style: .Default, handler: nil))      
      self.controller?.presentViewController(alert, animated: true, completion: nil)   
    }   
  }  
  
  // JS调用了我们的方法   
  func callWithDict(dict: [String : AnyObject]) {   
    print(“js call objc method: callWithDict, args: %@”, dict) 
  }
  
  // JS调用了我们的就去  
  func jsCallObjcAndObjcCallJsWithDict(dict: [String : AnyObject]) {     
    print(“js call objc method: jsCallObjcAndObjcCallJsWithDict, args: %@”, dict) 
    let jsParamFunc = self.jsContext?.objectForKeyedSubscript(“jsParamFunc”); 
    let dict = NSDictionary(dictionary: [“age”: 18, “height”: 168, “name”: “lili”])
    jsParamFunc?.callWithArguments([dict]) 
  }
}
```

接下来，我们在controller中在webview加载完成的代理中，给JS注入模型。

```
// MARK: - UIWebViewDelegate
func webViewDidFinishLoad(webView: UIWebView) {
  let context = webView.valueForKeyPath(“documentView.webView.mainFrame.javaScriptContext”) as? JSContextlet 
  model = JSObjCModel() 
  model.controller = self
  model.jsContext = context
  self.jsContext = context

  // 这一步是将OCModel这个模型注入到JS中，在JS就
  // 可以通过OCModel调用我们公暴露的方法了。
  self.jsContext?.setObject(model, forKeyedSubscript: “OCModel”)
  self.jsContext?.exceptionHandler = { (context, exception) in 
     print(“exception @”, exception) 
  }
}
```

我们是通过webView的valueForKeyPath获取的，其路径为documentView.webView.mainFrame.javaScriptContext。
这样就可以获取到JS的context，然后为这个context注入我们的模型对象。
我们先写两个JS方法：

```
var jsFunc = function() {
  alert(‘Objective-C call js to show alert’);
}

var jsParamFunc = function(argument) {
  document.getElementById(‘jsParamFuncSpan’).innerHTML
  = argument[‘name’];
}
```

这里我们定义了两个JS方法，一个是jsFunc，不带参数。
另一个是jsParamFunc，带一个参数。
接下来，我们在html中的body中添加以下代码：

`Test how to use objective-c call js`

现在就可以测试代码了。

当我们点击第一个按钮：Call ObjC system camera时，
通过OCModel.callSystemCamera()，就可以在HTML中通过JS调用OC的方法。
在Swift代码中，我们的callSystemCamera方法体中，添加了以下两行代码，就是获取HTML中所定义的JS就去jsFunc，然后调用它。

```
let jsFunc = self.jsContext?.objectForKeyedSubscript(“jsFunc”); jsFunc?.callWithArguments([]);
```

这样就可以在JS调用Siwft方法时，也让Swift反馈给JS。
注意：这里是通过objectForKeyedSubscript方法来获取变量jsFunc。
方法也是变量。看看下面传字典参数：

```
(void)jsCallObjcAndObjcCallJsWithDict:(NSDictionary )params {
    NSLog(@”jsCallObjcAndObjcCallJsWithDict was called, params is %@”, params);
    // 调用JS的方法
    JSValue jsParamFunc = self.jsContext[@”jsParamFunc”];
    [jsParamFunc callWithArguments:@[@{@”age”: @10, @”name”: @”lili”, @”height”: @158}]];
}
```

获取我们在HTML中定义的jsParamFunc方法，然后调用它并传了一个字典作为参数。

#源代码

好了，就讲这么多吧，如果想要Demo源代码，请到github：**[https://github.com/CoderJackyHuang/IOSCallJsOrJsCallIOS](https://github.com/CoderJackyHuang/IOSCallJsOrJsCallIOS)**

#推荐阅读

* [OC JavaScriptCore与js交互](http://www.henishuo.com/oc-js/)
* [WKWebView新特性及JS交互](http://www.henishuo.com/wkwebview-js/)

