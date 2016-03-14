#前言

记得曾经有好多人问过我webview加载到的图片太大了，超过了屏幕的宽度，怎么办呢？在笔者写这篇文章之前，也没有真正去验证过是否可行，只是告诉大家通过JS去设置css样式。

今天，也在segmentFault上看到有人也提了这样一个问题，因此决定亲自去测试一下，并分享给大家！

#HTML代码

```
<html>
  <head></head>
  <body>
     <h1>Test Img how to auto fit webview</h1>
     <img src="http://pic.nipic.com/2007-12-15/2007121519169400_2.jpg"></img>
     <h2>Another image</h2>
     <img src="http://www.zg2sc.cn/upfile/pic/2014/05/14/5f80fc3a258940998f921953269d1960141058011295091_163227633232_2.jpg"></img>
     <h2>Third image</h2>
     <img src="http://www.jydoc.com/uploads/jydoc/p35501/20091292046379677801.jpg"></img>
  </body>
</html>
```

#适应前图片太大

在网上随便找了几张大图，在适应之前，图片是远远大过屏幕的大小的，我们这里通过JS来实现图片自适应大小。在适应屏幕大小之前的效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160307-0@2x-e1457355438219.png)

如何通过js实现图片自适应屏幕呢？

#适应后图片显示合适

图片自适应后的效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160307-1@2x-e1457355507458.png)

我们只需要在webview的代理中实现注入js代码来设置图片的css就可以了：

```
- (void)webViewDidFinishLoad:(UIWebView *)webView {
  NSString *js = @"function imgAutoFit() { \
     var imgs = document.getElementsByTagName('img'); \
     for (var i = 0; i < imgs.length; ++i) {\
        var img = imgs[i];   \
        img.style.maxWidth = %f;   \
     } \
  }";
  js = [NSString stringWithFormat:js, [UIScreen mainScreen].bounds.size.width - 20];
  
  [webView stringByEvaluatingJavaScriptFromString:js];
  [webView stringByEvaluatingJavaScriptFromString:@"imgAutoFit()"];
}
```

当然，我们这里获取了屏幕的宽度，然后设置成图片的最大宽度为屏幕的宽度-20。

#最后

如果我的问题不清楚，可以到<https://segmentfault.com/q/1010000002693677>看看问题，当然这里只是一种方式来实现，其实还有很多的方式可以实现的，不过都需要依靠JS来实现的。

比如，我们可以通过JS获取所有的图片的URL然后本地去加载图片，将图片裁剪成适应后应该的图片的大小，这样既可以缓存到本地，又可以自适应。

#源代码

请大家到我的GITHUB下载DEMO：[WebViewImgAutoFit](https://github.com/CoderJackyHuang/WebViewImgAutoFit.git)

喜欢就**star**一下!

