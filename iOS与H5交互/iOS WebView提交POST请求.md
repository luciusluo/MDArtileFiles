#前言

一直以来，没有遇到过这样的BUG，于是才想到如何去解决！BUG是这样的：iOS中的Webview加载的是H5页面，正常情况下请求都是GET请求，但是对于表单提交却要求是POST请求，因此当我们重新创建一个Request来reload请求时，会自动变成GET请求，导致POST参数丢失。


#如何解决

NSMutableRequest类提供了这几个属性：

```
// GET/POST
@property (copy) NSString *HTTPMethod;
// POST BODY
@property (nullable, copy) NSData *HTTPBody;
```

下面是我的解决方案：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/屏幕快照-2016-01-27-上午9.40.17.png)

我们创建一个可变的request，然后重新load一下。


#关注我


**Swift/ObjC技术群一：324400294(已满)**

**Swift/ObjC技术群二：494669518**

**ObjC/Swift高级群：461252383（注明年限，新手勿扰）**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**原文有更新，请阅读原文**：[http://www.henishuo.com/webview-post/](http://www.henishuo.com/webview-post/)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)

