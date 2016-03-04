#前言

这里只是讲一个故事，一个发生在我身上的真实的故事。曾经，我以为搞加密很简单，不就是百度几个加密算法回来就可以了吗？百度上一大堆呢，要什么加密算法都有，有什么困难呢？是啊，理论上是可行的，可是，实际上却有很多的坑，跳了一个又一个，最后才发现大家已经跳进坑里面了。

这里所讲的故事是关于URL Encode的故事。想想我们iOS原来都是不管三七二十一，都是直接调用写一个URLEncode方法，然后在加密时就encode一下，服务端decode一下就可以了，但是我们要防篡改，使用了sign...

#Android端Encode问题

在安卓端，他们直接调用`URLEncoder.encode(text, encodeType)`这样的函数来进行encode，可是他们这个函数对空格进行encode后，得到的是+号，而不是%20。我们看到在浏览器里空格是转换成%20的。另外，安卓这个API并不是对所有的特殊字符都进行转码，这样就有问题了...生成sign签名时，如果都encode了，那么结果就会不一样

#iOS端Encode问题

在iOS端，我们直接使用系统NSString的API进行转码的话，不会对一些比较特殊的字符进行转码。系统NSString的转码API：

```
[str stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```
这样并不对特殊的字符，像`、!、:等字符并不会转码，那怎么办呢？我们通常会自己写一个API，调用C底层的API，可以指定对哪里特殊字符也转码(给NSString扩展)：

```
- (NSString *)hyb_URLEncode {
  NSString *newString =
  CFBridgingRelease(CFURLCreateStringByAddingPercentEscapes(kCFAllocatorDefault,
                                                            (CFStringRef)self,
                                                            NULL,
                                                            CFSTR(":/?#[]@!$ &'()*+,;=\"<>%{}|\\^~`"), CFStringConvertNSStringEncodingToEncoding(NSUTF8StringEncoding)));
  if (newString) {
    return newString;
  }
 
  return self;
}
```

这样就可以对所有特殊字符都转码了。而解码则是非常简单的，只是对空格进行特殊处理：

```
- (NSString *)hyb_URLDecode {
  NSString *input = self;
  NSMutableString *outputStr = [NSMutableString stringWithString:input];
  [outputStr replaceOccurrencesOfString:@"+"
                             withString:@" "
                                options:NSLiteralSearch
                                  range:NSMakeRange(0, [outputStr length])];
  
  return [outputStr stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
}
```

在iOS端，对空格转码得到%20，对+转码得到%2b，但是在服务端和android端对空格转码得到的是+，而对+转码得到的也是%2b。问题就出在这里了...


#怎么生成sign

通常的做法是对所有参数按key排序，然后拼接成a=x&b=y...这样的字符串，然后md5一下。但是如果encode一下，iOS端和安卓端出现不同的结果，那么服务端拿到以后是可以得到原串的，但是服务端encode一下所得到的结果会不一样，那么校验sign就会失败。

但是，如果不对每个value进行转码，在服务端就无法通过&来分割了，因为value中有&时，若不转码就会出问题，因此encode是必须的。


#如何解决

生成sign时，是遍历所有的key-value，然后拼接，最后md5。那么，生成sign时，我们只要不对value进行encode，而其他上传的参数值都encode，这样就可以解决我们的问题了。

解决方案：

生成sign时，遍历parameters，对每个value都不进行encode，直接拼接然后md5（前提是一定要Asc或者Desc排序一下）。

而其他参数在拼接时，就正常使用encode，一切就可以解决了！

#关注我


**Swift/ObjC技术群一：324400294(已满)**

**Swift/ObjC技术群二：494669518**

**ObjC/Swift高级群：461252383（注明年限，新手勿扰）**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**原文有更新，请阅读原文**：[http://www.henishuo.com/urlencode-space-special-handle/](http://www.henishuo.com/urlencode-space-special-handle/)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)





