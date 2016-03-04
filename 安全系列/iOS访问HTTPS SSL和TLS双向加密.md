
#前言


由于项目需求，访问服务是`https`的，并且使用的是`TLS`加密方式。

####关于https和ssl的原理，请到此处查看[http://blog.163.com/magicc_love/blog/static/185853662201321423527263/](http://blog.163.com/magicc_love/blog/static/185853662201321423527263/)


#使用MKNetworkit实现


下面说明使用MKNetworkit网络库实现的代码：

```
- (void)testClientCertificate {
  SecIdentityRef identity = NULL;
  SecTrustRef trust = NULL;
  NSString *p12 = [[NSBundle mainBundle] pathForResource:@"testClient" ofType:@"p12"];
  NSData *PKCS12Data = [NSData dataWithContentsOfFile:p12];
  
  [[self class] extractIdentity:&identity andTrust:&trust fromPKCS12Data:PKCS12Data];
  
  NSString *url = @"https://218.244.131.231/ManicureShop/api/order/pay/%@";
  NSDictionary *dic = @{@"request" : @{
                            @"orderNo" : @"1409282102222110030643",
                            @"type" : @(2)
                            }
                        };
  
  _signString = nil;
  NSData *postData = [NSJSONSerialization dataWithJSONObject:dic
                                                     options:NSJSONWritingPrettyPrinted
                                                       error:nil];
  NSString *sign = [self signWithSignKey:@"test" params:dic];
  NSMutableData *body = [postData mutableCopy];
  NSLog(@"%@", [[NSString alloc] initWithData:body encoding:NSUTF8StringEncoding]);
  url = [NSString stringWithFormat:url, sign];
  
  MKNetworkEngine *engine = [[MKNetworkEngine alloc] initWithHostName:@"218.244.131.231"];
  NSString *path = [NSString stringWithFormat:@"/ManicureShop/api/order/pay/%@", sign];
  MKNetworkOperation *op = [engine operationWithPath:path params:dic httpMethod:@"POST" ssl:YES];
  op.postDataEncoding = MKNKPostDataEncodingTypeJSON; // 传JOSN
  
  // 这个是app bundle 路径下的自签证书
  op.clientCertificate = [[[NSBundle mainBundle] resourcePath]
                          stringByAppendingPathComponent:@"testClient.p12"];
  // 这个是自签证书的密码
  op.clientCertificatePassword = @"testHttps";
  
  // 由于自签名的证书是需要忽略的，所以这里需要设置为YES，表示允许
  op.shouldContinueWithInvalidCertificate = YES;
  [op addCompletionHandler:^(MKNetworkOperation *completedOperation) {
    NSLog(@"%@", completedOperation.responseJSON);
  } errorHandler:^(MKNetworkOperation *completedOperation, NSError *error) {
    NSLog(@"%@", [error description]);
  }];
  
  [engine enqueueOperation:op];
  return;
}

// 下面这段代码是提取和校验证书的数据的
+ (BOOL)extractIdentity:(SecIdentityRef *)outIdentity
               andTrust:(SecTrustRef*)outTrust
         fromPKCS12Data:(NSData *)inPKCS12Data {
  OSStatus securityError = errSecSuccess;
  
  // 证书密钥
  NSDictionary *optionsDictionary = @{@"testHttps": (__bridge id)kSecImportExportPassphrase};
  CFArrayRef items = CFArrayCreate(NULL, 0, 0, NULL);
  securityError = SecPKCS12Import((__bridge CFDataRef)inPKCS12Data,
                                  (__bridge CFDictionaryRef)optionsDictionary,
                                  &items);
  
  if (securityError == 0) {
    CFDictionaryRef myIdentityAndTrust = CFArrayGetValueAtIndex (items, 0);
    const void *tempIdentity = NULL;
    tempIdentity = CFDictionaryGetValue (myIdentityAndTrust, kSecImportItemIdentity);
    *outIdentity = (SecIdentityRef)tempIdentity;
    const void *tempTrust = NULL;
    tempTrust = CFDictionaryGetValue (myIdentityAndTrust, kSecImportItemTrust);
    *outTrust = (SecTrustRef)tempTrust;
  } else {
    NSLog(@"Failed with error code %d",(int)securityError);
    return NO;
  }
  return YES;
}
```


#使用AFNetworking实现


下面说明一下使用AFNetworking网络库访问的方式：

```
- (void)testClientCertificate {
  SecIdentityRef identity = NULL;
  SecTrustRef trust = NULL;
  NSString *p12 = [[NSBundle mainBundle] pathForResource:@"testClient" ofType:@"p12"];
  NSData *PKCS12Data = [NSData dataWithContentsOfFile:p12];
  
  [[self class] extractIdentity:&identity andTrust:&trust fromPKCS12Data:PKCS12Data];
  
  NSString *url = @"https://218.244.131.231/ManicureShop/api/order/pay/%@";
  NSDictionary *dic = @{@"request" : @{
                            @"orderNo" : @"1409282102222110030643",
                            @"type" : @(2)
                            }
                        };
  
  _signString = nil;
  NSData *postData = [NSJSONSerialization dataWithJSONObject:dic
                                                     options:NSJSONWritingPrettyPrinted
                                                       error:nil];
  NSString *sign = [self signWithSignKey:@"test" params:dic];
  NSMutableData *body = [postData mutableCopy];
  NSLog(@"%@", [[NSString alloc] initWithData:body encoding:NSUTF8StringEncoding]);
  url = [NSString stringWithFormat:url, sign];
  
  AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
  manager.requestSerializer = [AFJSONRequestSerializer serializer];
  manager.responseSerializer = [AFJSONResponseSerializer serializer];
  [manager.requestSerializer setValue:@"application/json" forHTTPHeaderField:@"Accept"];
  [manager.requestSerializer setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
  manager.responseSerializer.acceptableContentTypes = [NSSet setWithArray:@[@"application/json",
                                                                            @"text/plain"]];
  manager.securityPolicy = [self customSecurityPolicy];
  
  [manager POST:url parameters:dic success:^(AFHTTPRequestOperation *operation, id responseObject) {
    NSLog(@"JSON: %@", responseObject);
  } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
    
    NSLog(@"Error: %@", error);
  }];
}

// 下面这段代码是处理SSL安全性问题的:
/**** SSL Pinning ****/
- (AFSecurityPolicy*)customSecurityPolicy {
  NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"testClient" ofType:@"cer"];
  NSData *certData = [NSData dataWithContentsOfFile:cerPath];
  AFSecurityPolicy *securityPolicy = [AFSecurityPolicy defaultPolicy];
  [securityPolicy setAllowInvalidCertificates:YES];
  [securityPolicy setPinnedCertificates:@[certData]];
  [securityPolicy setSSLPinningMode:AFSSLPinningModeCertificate];
  /**** SSL Pinning ****/
  return securityPolicy;
}
```

#温馨提示


为了实现访问`https tls`加密方式，我也费了不少时间来查，这里写下此文章，希望对大家有用！

#解答

下面是几个问题是笔者的`CSDN`博客上朋友们提出的部分问题，笔者在这里可以回答这几个问题。

###Quetion1: signWithSignKey这个方法找不到？求解?

请注意，这里的`signWithSignKey`只是一个加密算法，关于加密的算法，通常是前端与后端要采用统一的加密规则来实现的，因此大家不要想着让我把这个加密算法扔给你们就可以的。

解决办法：找到你们的后端开发人员，协调加密算法，写出伪代码，然后苹果、安卓和后台三端按照伪代码实现出自己的加密算法。

###Quetions2：您好.我想问一下，后台给我的证书我怎么弄到工程里面呢？

后台给你的不是`p12`证书就是`cer`证书，直接放到工程里面就可以了，然后引进工程中。引入正确才能读取下内容。

###Quetion3：博主好，直接把p12放在程序包中是否安全？

不用担心安全的问题，因此这个证书只是我们前端为了检验而已，而服务端还会再验证的。


#源代码


真的很抱歉，由于以前写这篇文章的时候，没有将代码传到`git`上，所以现在找不到源代码了。不过上面所贴出的代码就是核心代码了。

#关注我


关注                | 账号              | 备注
-------------      | -------------     | ----------------
Swift/ObjC技术群一  | 324400294         |  群一若已满，请申请群二
Swift/ObjC技术群二  | 494669518         | 群二若已满，请申请群三
Swift/ObjC技术群三  | 461252383         | 群三若已满，会有提示信息
关注微信公众号       | iOSDevShares      | 关注微信公众号，会定期地推送好文章
关注新浪微博账号      |  [标哥Jacky](http://weibo.com/u/5384637337) | 关注微博，每次发布文章都会分享到新浪微博
关注标哥的GitHub     | [CoderJackyHuang](https://github.com/CoderJackyHuang) | 这里有很多的Demo和开源组件
关于我               | [进一步了解标哥](http://www.henishuo.com/about-biaoge/) | 如果觉得文章对您很有帮助，可捐助我！

