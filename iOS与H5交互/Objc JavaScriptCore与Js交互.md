

#前言

ObjectiveC与Js交互是常见的需求，可对于新手或者所谓的高手而言，其实并不是那么简单明了。这里只介绍iOS7.0后出来的JavaScriptCore framework。

代码有更新，提供两个页面切换，测试是否可以成功调用！

#关于JavaScriptCore

本教程中所涉及到的几种类型：

* **JSContext**, JSContext是代表JS的执行环境，通过-evaluateScript:方法就可以执行一JS代码
* **JSValue**, JSValue封装了JS与ObjC中的对应的类型，以及调用JS的API等
* **JSExport**, JSExport是一个协议，遵守此协议，就可以定义我们自己的协议，在协议中声明的API都会在JS中暴露出来，才能调用

#ObjC与JS交互方式

通过JSContext，我们有两种调用JS代码的方法：

* 1、直接调用JS代码
* 2、在ObjC中通过JSContext**注入模型**，然后调用模型的方法

##直接调用JS代码

```
 // 一个JSContext对象，就类似于Js中的window，
 // 只需要创建一次即可。
 self.jsContext = [[JSContext alloc] init];

 //  jscontext可以直接执行JS代码。
 [self.jsContext evaluateScript:@"var num = 10"];
 [self.jsContext evaluateScript:@"var squareFunc = function(value) { return value * 2 }"];
 // 计算正方形的面积
 JSValue *square = [self.jsContext evaluateScript:@"squareFunc(num)"];
 
 // 也可以通过下标的方式获取到方法
 JSValue *squareFunc = self.jsContext[@"squareFunc"];
 JSValue *value = [squareFunc callWithArguments:@[@"20"]];
 NSLog(@"%@", square.toNumber);
 NSLog(@"%@", value.toNumber);
```

这种方式是没有注入模型到JS中的。这种方式使用起来不太合适，通常在JS中有很多全局的函数，为了防止名字重名，使用模型的方式是最好不过了。通过我们协商好的模型名称，在JS中直接通过模型来调用我们在ObjC中所定义的模型所公开的API。

##通过注入模型的方式交互

首先，我们需要先定义一个协议，而且这个协议必须要遵守JSExport协议。

```
@protocol JavaScriptObjectiveCDelegate <JSExport>

// JS调用此方法来调用OC的系统相册方法
- (void)callSystemCamera;
// 在JS中调用时，函数名应该为showAlertMsg(arg1, arg2)
// 这里是只两个参数的。
- (void)showAlert:(NSString *)title msg:(NSString *)msg;
// 通过JSON传过来
- (void)callWithDict:(NSDictionary *)params;
// JS调用Oc，然后在OC中通过调用JS方法来传值给JS。
- (void)jsCallObjcAndObjcCallJsWithDict:(NSDictionary *)params;

@end
```

接下来，我们还需要定义一个模型：

```
// 此模型用于注入JS的模型，这样就可以通过模型来调用方法。
@interface HYBJsObjCModel : NSObject <JavaScriptObjectiveCDelegate>

@property (nonatomic, weak) JSContext *jsContext;
@property (nonatomic, weak) UIWebView *webView;

@end
```

实现这个模型：

```
@implementation HYBJsObjCModel

- (void)callWithDict:(NSDictionary *)params {
 NSLog(@"Js调用了OC的方法，参数为：%@", params);
}

// Js调用了callSystemCamera
- (void)callSystemCamera {
 NSLog(@"JS调用了OC的方法，调起系统相册");
 
 // JS调用后OC后，又通过OC调用JS，但是这个是没有传参数的
 JSValue *jsFunc = self.jsContext[@"jsFunc"];
 [jsFunc callWithArguments:nil];
}

- (void)jsCallObjcAndObjcCallJsWithDict:(NSDictionary *)params {
 NSLog(@"jsCallObjcAndObjcCallJsWithDict was called, params is %@", params);
 
 // 调用JS的方法
 JSValue *jsParamFunc = self.jsContext[@"jsParamFunc"];
 [jsParamFunc callWithArguments:@[@{@"age": @10, @"name": @"lili", @"height": @158}]];
}

- (void)showAlert:(NSString *)title msg:(NSString *)msg {
 dispatch_async(dispatch_get_main_queue(), ^{
   UIAlertView *a = [[UIAlertView alloc] initWithTitle:title message:msg delegate:nil cancelButtonTitle:@"Ok" otherButtonTitles:nil, nil];
   [a show];
 });
}

@end
```

接下来，我们在controller中在webview加载完成的代理中，给JS注入模型。

```
#pragma mark - UIWebViewDelegate
- (void)webViewDidFinishLoad:(UIWebView *)webView {
 self.jsContext = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
 // 通过模型调用方法，这种方式更好些。
  HYBJsObjCModel *model  = [[HYBJsObjCModel alloc] init];
 self.jsContext[@"OCModel"] = model;
 model.jsContext = self.jsContext;
 model.webView = self.webView;

 self.jsContext.exceptionHandler = ^(JSContext *context, JSValue *exceptionValue) {
   context.exception = exceptionValue;
   NSLog(@"异常信息：%@", exceptionValue);
 };
}
```

我们是通过webView的valueForKeyPath获取的，其路径为`documentView.webView.mainFrame.javaScriptContext`。
这样就可以获取到JS的context，然后为这个context注入我们的模型对象。

**注意：**每次页面载入都要更新jsContext，否则将没有注入。但是这种做法有一个不好的地方就是，页面没有载入完成并不能注入JS对象到JS端，那么H5页面想要在页面载入时就手动调用JS代码就不可以了！

我们先写两个JS方法：

```
 var jsFunc = function() {
   alert('Objective-C call js to show alert');
 }
 
 var jsParamFunc = function(argument) {
   document.getElementById('jsParamFuncSpan').innerHTML
   = argument['name'];
 }
```
 
这里我们定义了两个JS方法，一个是jsFunc，不带参数。
另一个是jsParamFunc，带一个参数。

接下来，我们在html中的body中添加以下代码：

```
<div style="margin-top: 100px">
<h1>Test how to use objective-c call js</h1>
<input type="button" value="Call ObjC system camera" onclick="OCModel.callSystemCamera()">
<input type="button" value="Call ObjC system alert" onclick="OCModel.showAlertMsg('js title', 'js message')">
</div>

<div>
<input type="button" value="Call ObjC func with JSON " onclick="OCModel.callWithDict({'name': 'testname', 'age': 10, 'height': 170})">
<input type="button" value="Call ObjC func with JSON and ObjC call js func to pass args." onclick="OCModel.jsCallObjcAndObjcCallJsWithDict({'name': 'testname', 'age': 10, 'height': 170})">
</div>

<div>
<span id="jsParamFuncSpan" style="color: red; font-size: 50px;"></span>
</div>
```

现在就可以测试代码了。

当我们点击第一个按钮：`Call ObjC system camera`时，
通过`OCModel.callSystemCamera()`，就可以在HTML中通过JS调用OC的方法。
在OC代码中，我们的`callSystemCamera`方法体中，添加了以下两行代码，就是获取HTML中所定义的JS就去jsFunc，然后调用它。

```
 JSValue *jsFunc = self.jsContext[@"jsFunc"];
 [jsFunc callWithArguments:nil];
```
 
这样就可以在JS调用OC方法时，也让OC反馈给JS。

看看下面传字典参数：

```
- (void)jsCallObjcAndObjcCallJsWithDict:(NSDictionary *)params {
 NSLog(@"jsCallObjcAndObjcCallJsWithDict was called, params is %@", params);
 
 // 调用JS的方法
 JSValue *jsParamFunc = self.jsContext[@"jsParamFunc"];
 [jsParamFunc callWithArguments:@[@{@"age": @10, @"name": @"lili", @"height": @158}]];
}
```

获取我们在HTML中定义的jsParamFunc方法，然后调用它并传了一个字典作为参数。

#小结

1. 在多个web页面之间，每次页面载入完成时，都要注入JS对象（通过JSContext），才能保证每个页面都可以让OC与JS端可以交互。
2. 通过JSContext页面可以让OC与JS交互，但是H5端并不能在页面载入完成前调用OC的代码，因此此时还没有注入JS对象。即使在webViewDidFinishLoad时就注入，对象也没有并不存在。



#源代码

好了，就讲这么多吧，如果想要Demo源代码，请到github下载：**[IOSCallJsOrJsCallIOS](https://github.com/CoderJackyHuang/IOSCallJsOrJsCallIOS)**

#推荐阅读

* [Swift JavaScriptCore与JS交互](http://www.henishuo.com/swift-js/)
* [WKWebView新特性及JS交互](http://www.henishuo.com/wkwebview-js/)


