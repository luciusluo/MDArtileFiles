#前言

前段时间，群里的小伙伴们经常问题UITableViewCell中要放一个UIWebView，怎么做呢？怎么算高度？怎么让它自适应？这一听感觉挺不好处理的。

因为UIWebView通过代理加载的话，还没有计算高度出来，cell的heightForRowAtIndexPath已经调用了。

基于此，笔者尝试学习了一下如何去计算其高度，并自适应。

注意：笔者只是抛砖引玉，仅仅处理了首次加载WebView得到的内容的高度，如果点击WebView里面的内容，页面变大或者变小，这里可没有处理刷新计算高度哦。这部分有点难度的，就交给大家自行深入吧~希望本博文可以给大家新的思想而已

#效果图

![image](http://www.henishuo.com/wp-content/uploads/2016/02/QQ20160229-0@2x.png)

#难点分析

* 性能问题：如果每一行cell的高度不一样，采用常规做法是很耗性能，然后高度不好处理。
* 自适应高度问题：如果每次都通过加载UIWebView内容成功后才reload行，从而计算高度，这是会进行死循环的。webview加载—>刷新当前行-->又重新加载webview-->又重复刷新高度-->死循环，当然也可以加载一次webview后拿到的高度跟当前刷新加载的高度比较，若相等则不必无限刷新，不过这么做有一个最大的问题，就是当前正常看着看着，突然又刷新变化了~

如果通过以下中的任何一个方法来加载HTML，那么就是常规的做法了，这种做法要在cell中使用，是比较困难而且比较复杂难以控制的：

```
- (void)loadRequest:(NSURLRequest *)request;
- (void)loadHTMLString:(NSString *)string baseURL:(NSURL *)baseURL;
- (void)loadData:(NSData *)data MIMEType:(NSString *)MIMEType textEncodingName:(NSString *)textEncodingName baseURL:(NSURL *)baseURL;
```
加载完了内容才能执行stringByEvaluatingJavaScriptFromString方法。

#设计思想

笔者没有使用自动布局来计算，这方面也让大家自行发挥吧。笔者只是通过常规的计算方式来学习一下基本的功能。

本demo的设计思想是通过js的方式来计算高度及替换HTML。

关键点如下：

* 获取HTML的高度：在cell的代理方法heightForRowAtIndexPath，我们要计算出HTML的高度，这样才能确定UIWebView的高度。
* 替换原来的HTML：在cell配置数据显示处，直接替换掉原来的HTML，而不是通过WebView所提供的API来加载HTML，这样就可以同步实现得到显示数据

另外，在计算高度是，如何才能计算HTML的内容的高度呢？那么我们必须借助一个UIWebView来处理了，因此，我们还要使用一个专门用来计算HTML高度的UIWebView。

#计算HTML高度的UIWebView

```
- (UIWebView *)webView {
  if (_webView == nil) {
    _webView = [[UIWebView alloc] init];
    CGSize size = CGSizeMake(self.view.frame.size.width - 20, 0);
    _webView.frame = CGRectMake(0, 0, size.width, size.height);
  }
  
  return _webView;
}
```

看到设置了大小了吗？我们必须指定webview的width否则下面计算的高度是不正确的。但是高度不要指定，设置为0就可以了（因为一旦指定，内容就会自动按照这个高度来自适应的，高度也就不用计算了）。

#计算HTML的高度

首先，我们在ViewController中添加了个属性webView，用于负责计算HTML的高度的。因此，将它作为属性，一直都需要使用。代码如下：

```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
  HYBTestModel *model = [self.datasource objectAtIndex:indexPath.row];
  CGFloat w = [UIScreen mainScreen].bounds.size.width;
  CGFloat h = [model.title sizeWithFont:[UIFont systemFontOfSize:16]
                      constrainedToSize:CGSizeMake(w, CGFLOAT_MAX)
                          lineBreakMode:NSLineBreakByWordWrapping].height;
  h += 10 + 20;
  
  if (model.webHeight <= 0) {
    // 注意不能使用双引号
    NSString *js = [NSString stringWithFormat:@"document.body.innerHTML = '%@'", model.html];
    [self.webView stringByEvaluatingJavaScriptFromString:js];
    
    NSLog(@"%@", [self.webView stringByEvaluatingJavaScriptFromString:@"document.body.innerHTML"]);
    
    NSString *heightJs = @"document.getElementsByTagName('div')[0].scrollHeight";
    CGFloat webHeight = [[self.webView stringByEvaluatingJavaScriptFromString:heightJs] floatValue];
    NSLog(@"%f", webHeight);

    self.webView.scrollView.contentSize = CGSizeMake(w - 20, webHeight);
    model.webHeight = webHeight + 40;
  }
  
  return h + model.webHeight + 40;
}
```

* 先通过document.body.innerHTML这段JS来替换用于计算HTML高度的UIWebView的HTML
* document.getElementsByTagName('div')[0].scrollHeight来获取可滚动范围的高度，注意，这个参数div不是固定的，要根据自己的HTML的内容来获取，不能直接获取body.scrollHeight，这是不对的。对于笔者的demo中，div是最上层标签，所以就可以通过它获取高度。
* 为什么要将计算的结果加上40？因为直接计算出来的是不够准备的，因为还有margin、padding，自己看看差不多就可以了。

#配置Cell

```
- (void)configCellWithModel:(HYBTestModel *)model {
  self.label.text = model.title;
  [self.label sizeToFit];
  
  NSString *js = [NSString stringWithFormat:@"document.body.innerHTML = '%@'", model.html];
 [self.webView stringByEvaluatingJavaScriptFromString:js];
  
  self.webView.frame = CGRectMake(10,
                                  self.label.frame.origin.y + self.label.frame.size.height + 20,
                                  self.label.frame.size.width,
                                  model.webHeight);
}
```

对于配置cell显示数据时，直接替换UIWebView里面的HTML就可以了，不需要通过代理回调才通知高度，如此就可以直接设置webview的frame了。

这里可没有再计算HTML的高度了，在计算高度的时候，直接给model添加了属性model.webHeight，用于记录所计算出来的高度，以免重复计算。

#源代码

大家可以到笔者的GITHUB下载源代码参考参考：[UITableViewCell嵌入UIWebView](https://github.com/CoderJackyHuang/UITableViewEmbedUIWebViewDemo)

#最后

笔者所提供的本demo这种做法不一定是最好的，如果大家想到更加优秀的做法，请在评论中回复，或者通过邮件、QQ、微博、群等方式通知笔者，共同进步~

