#前言

最近很多朋友看到了我的JS与OC/Swift交互的文章，经常有朋友问HTML中的图片在点击时如何在原生预览？这个问题其实很简单，只是一直没有动手去实践过。今天借着闲时，动手写了个demo，只是实现了点击接收，不实现预览功能，怎么预览就是大家要做的事了。

本篇文章只是说明通过怎样的方式接入，才能让iOS Native与Js交互，当然这里不会使用JavaScriptCore这个库，也不会使用任何第三方，如UIWebViewJavaScriptBridge。

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-01-上午11.24.32.png)

#Swift实现代码

实现思路：面对iOS与JS交互，最直接的就是UIWebView的代理方法。那么，提炼出来的问题就是：如何知道H5中的图片被点击了？在点击以后又如何在iOS端判断？针对第一个问题，最直接的方式就是注入图片点击的JS代码。针对第二个问题，最直接的方式就是通过注入特定的scheme来识别：

```
class ViewController: UIViewController, UIWebViewDelegate {
  var webView = UIWebView()
  
  override func viewDidLoad() {
    super.viewDidLoad()
    
    self.view.addSubview(self.webView)
    self.webView.delegate = self
    self.webView.frame = self.view.bounds
    self.webView.loadRequest(NSURLRequest(URL: NSURL(string: "http://www.cocoachina.com/programmer/20160113/14976.html")!))
  }
  
  
  // MARK: UIWebViewDelegate
  func webView(webView: UIWebView, shouldStartLoadWithRequest request: NSURLRequest, navigationType: UIWebViewNavigationType) -> Bool {
    let scheme = "hyb-image-preview:"
    // 由于我们注入了JS，在点击HTML中的图片时，就会跳转，然后在此处就可以拦截到
    if ((request.URL?.scheme.hasPrefix(scheme)) != nil) {
      // 获取原始图片的URL
      let src = request.URL?.absoluteString.stringByReplacingOccurrencesOfString(scheme, withString: "")
      if let imageUrl = src {
        // 原生API展开图片
        // 这里已经拿到所点击的图片的URL了，剩下的部分，自己处理了
        // 有时候会感觉点击无响应，这是因为webViewDidFinishLoad,还没有调用。
        // 调用很晚的原因，通常是因为H5页面中有比较多的内容在加载
        // 因此，若是原生APP与H5要交互，H5要尽可能地提高加载速度
        // 不相信？在webViewDidFinishLoad加个断点就知道了
        print(imageUrl)
      }
    }
    
    return true
  }
  
  func webViewDidFinishLoad(webView: UIWebView) {
    // 在H5页面加载完成时，注入图片点击的JS代码
    let js = "function addImgClickEvent() { " +
      "var imgs = document.getElementsByTagName('img');" +
      // 遍历所有的img标签，统一加上点击事件
      "for (var i = 0; i < imgs.length; ++i) {" +
      "var img = imgs[i];" +
      "img.onclick = function () {" +
      // 给图片添加URL scheme，以便在拦截时可能识别跳转
      "window.location.href = 'hyb-image-preview:' + this.src;" +
      "}" +
      "}" +
    "}"
    // 注入JS代码
    self.webView.stringByEvaluatingJavaScriptFromString(js)
    
    // 执行所注入的JS
    self.webView.stringByEvaluatingJavaScriptFromString("addImgClickEvent();")
  }
}
```

这里需要到写JS函数，如何JS都不懂，那就不知道怎么办了，当然让后台帮你写也是可以的。这里所实现的功能是非常简单的，只是往H5中注入了JS函数，然后执行所注入的JS函数，给所有的img标签添加上onclick事件处理，在img被点击时，重新载入图片就会回调我们UIWebView的shouldStartLoadWithRequest代理方法，然后在此处识别是否是我们所指定的scheme，如果是，则认为这个是我们要处理的图片预览。

#源代码

请大家到我的GITHUB地址下载吧：[https://github.com/CoderJackyHuang/HTMLImagePreviewDemo](https://github.com/CoderJackyHuang/HTMLImagePreviewDemo)

#推荐阅读

* [ObjC点击H5图片Native预览](http://www.henishuo.com/objc-html-img-preview/)
* [OC JavaScriptCore与js交互](http://www.henishuo.com/oc-js/)
* [WKWebView新特性及JS交互](http://www.henishuo.com/wkwebview-js/)
* [Swift JavaScriptCore与JS交互](http://www.henishuo.com/swift-js/)
* [iOS Native加载H5中的图片](http://www.henishuo.com/ios-native-h5-img/)




