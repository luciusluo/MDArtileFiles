#前言

---
最近公司的客户端安全性出现了严重的问题，如今这个出解决方案并自我测试验证可行性的重任落在了我的身上，学习了很多他人的文章，再经过多次讨论，最后才确定最终解决方案。笔者在这里讲讲这一经历中所需要了解的知识。

`iOS`客户端想要加密传输数据以防被窃取，最可靠的方式莫过于使用公钥加密算法加密，使用`HTTPS`协议在整个传输过程中都使用了这个技术，对于未能使用`HTTPS`的`HTTP`接口，我们能否实现`RSA`加密呢？

`PHP`端实现解密可参考：[iOS与PHP端实现RSA非对称加密解密](http://orangeholic.iteye.com/blog/2161771)

>提示：本篇文章只讲`RSA`非对称加密相关知识。

#`RSA`非对称加密介绍

----
非对称加密算法可能是世界上最重要的算法，它是当今电子商务等领域的基石。简而言之，非对称加密就是指加密公钥和解密密钥是不同的，而且加密公钥和解密密钥是成对出现。非对称加密又叫公钥加密，也就是说成对的密钥，其中一个是对外公开的，所有人都可以获得，人们称之为公钥；而与之相对应的称为私钥，只有这对密钥的生成者才能拥有。

**公钥私钥具有以下重要特性：**

对于一个私钥，有且只有一个与之对应的公钥。公钥公开给任何人，私钥通常是只有生成者拥有。公/私钥通常是`1024`位或者`2048`位，越长安全系数越高，但是解密越困难。尽管拿到了公钥，如果没有私钥，要想解密那几乎是不可能的，至少现在在世界上还没有人公开出来说成功解密的。

非对称加密算法如此强大可靠，却有一个弊端，就是加解密比较耗时。因此，在实际使用中，往往与对称加密和摘要算法结合使用。对称加密很好理解，本篇文章不细读对称加密。我们再来看一下摘要算法。

#`RSA`非对称加密典型用法 

---
* 防止中间人攻击：将明文通过接收人的公钥加密，传输给接收人，因为只有接收人拥有对应的私钥，别人不可能拥有或者不可能通过公钥推算出私钥，所以传输过程中无法被中间人截获。只有拥有私钥的接收人才能解密得到原文。此用法通常用于交换对称密钥。比如，在登录时在客户端生成随机密钥，然后使用公钥加密传输到服务端，服务端使用私钥解密得到随机密钥，此密钥就用于后续的`AES`加密/解密使用。
* 身份验证和防止篡改：通常会将所有的参数按照某种规则拼接并排序，然后使用某种算法加密生成摘要，然后在服务端使用同样的规则生成摘要，比较两个摘要以确定身份和是否被篡改过。

###摘要算法

上面提到摘要算法，摘要算法是指可以将任意长度的文本，通过一个算法，得到一个固定长度的文本。这里文本不一定只是文本，可以是字节数据。所以摘要算法可以将很长的内容变成一个固定长度的东西。

**摘要算法具有以下重要特性：**

* 只要源内容不同，计算得到的结果，必然不同。
* 无法通过摘要算法可逆拿到源内容


典型的摘要算法有大名鼎鼎的`MD5`和`SHA`。摘要算法主要用于比对信息源是否一致，因为只要源内容发生变化，得到的摘要必然不同；而且通常结果要比源短很多，所以称为“摘要信息”。

###数字签名

理解了非对称加密和摘要算法，来看一下数字签名，实际上数字签名就是两者结合。现在假设我们需要传输好几个参数，如何确定接口中所传输的参数值是否被篡改过？这时候数字签名就可以解决这个问题了。

常用的解决办法：将参数按照指定的规则拼接（拼接规则要与服务端协商统一），然后排序（保证服务端的顺序与客户端的一致），然后通过摘要算法如`MD5`得到摘要信息，再通过`AES`加密摘要信息，服务端按照同样的规则，生成摘要，然后比对是否一致。

>温馨提示：专业叫数字签名，实际工作中大家都叫加盐防篡改。

###`CA`签发证书

实际上，数字证书就是通过数字签名实现的数字化的证书。在一般的证书组成部分中还加入了其他的信息，比如证书有效期，公司组织名称等，过了有效期需要重新签发。

跟现实生活中的签发机构一样，数字证书的签发机构也有若干，并有不同的用处。比如苹果公司就可以签发跟苹果公司有关的证书，而跟`web`访问有关的证书则是由几家全世界公认的机构进行签发。这些签发机构称为`CA`（`Certificate Authority`）。

对于被签发人，通常都是企业或开发者。对于需要搭建基于`SSL`的网站，那么需要从几家国际公认的`CA`去申请证书；对如需要开发`iOS`的应用程序，需要从苹果公司获得相关的证书。这些申请通常是企业或者开发者个人提交给`CA`的。当然申请所需要的材料、资质和费用都各不相同，是由这些`CA`制定的，比如苹果要求`$99`或者`$299`的费用。

`web`应用相关的`SSL`证书的验证方通常是浏览器；`iOS`各种证书的验证方是`iOS`设备。我们之所以必须从`CA`处申请证书，就是因为`CA`已经将整个验证过程规定好了。

#生成自签名证书

```
// 生成1024位私钥
openssl genrsa -out private_key.pem 1024

// 根据私钥生成CSR文件
openssl req -new -key private_key.pem -out rsaCertReq.csr

// 根据私钥和CSR文件生成crt文件
openssl x509 -req -days 3650 -in rsaCertReq.csr -signkey private_key.pem -out rsaCert.crt

// 为IOS端生成公钥der文件
openssl x509 -outform der -in rsaCert.crt -out public_key.der

// 将私钥导出为这p12文件
openssl pkcs12 -export -out private_key.p12 -inkey private_key.pem -in rsaCert.crt
```

注意：在生成私钥时，是需要密码的，一定要记住密码。

得到了`public_key.der`公钥和`private_key.p12`私钥，就可以进行加密和解密了。

```
NSString *encryptString = [self rsaEncryptText:@"123456好哇好哇哈"];
NSLog(@"加密：123456好哇好哇哈：%@", encryptString);
NSLog(@"解密结果为：%@", [self rsaDecryptWithText:encryptString]);
```

#公开HYBRSAEncrypt

请注意，代码中直接放到工程并不能直接使用，有部分转成`base64`字符串的方法是一个扩展，自己替换成你们自己的方法就可以了。代码有部分是参考他人的。如果觉得是您写的，或者与您的相同，不希望公开可给我邮件。

```
//
//  HYBRSAEncrypt.h
//
//  Created by huangyibiao on 15/12/16.
//  Copyright © 2015年 edu.huangyibiao.com. All rights reserved.
//

#import <Foundation/Foundation.h>

@interface HYBRSAEncrypt : NSObject

// 加密相关
- (void)loadPublicKeyWithPath:(NSString *)derFilePath;
- (void)loadPublicKeyWithData:(NSData *)derData;
- (NSString *)rsaEncryptText:(NSString *)text;
- (NSData *)rsaEncryptData:(NSData *)data;

// 解密相关
- (void)loadPrivateKeyWithPath:(NSString *)p12FilePath password:(NSString *)p12Password;
- (void)loadPrivateKeyWithData:(NSData *)p12Data password:(NSString *)p12Password;
- (NSString *)rsaDecryptText:(NSString *)text;
- (NSData *)rsaDecryptData:(NSData *)data;

@end
```

实现文件中，有几个`API`是本人的扩展方法，请替换成你们所封装的`API`：

```
//
//  HYBRSAEncrypt.m
//
//  Created by huangyibiao on 15/12/16.
//  Copyright © 2015年 edu.huangyibiao.com. All rights reserved.
//

#import "HYBRSAEncrypt.h"

@interface HYBRSAEncrypt () {
  SecKeyRef _publicKey;
  SecKeyRef _privateKey;
}

@end

@implementation HYBRSAEncrypt

- (void)dealloc {
  if (nil != _publicKey) {
    CFRelease(_publicKey);
  }
  
  if (nil != _privateKey) {
    CFRelease(_privateKey);
  }
}

#pragma mark - 加密相关
- (void)loadPublicKeyWithPath:(NSString *)derFilePath {
      NSData *derData = [[NSData alloc] initWithContentsOfFile:derFilePath];
  if (derData.length > 0) {
    [self loadPublicKeyWithData:derData];
  } else {
    NSLog(@"load public key fail with path: %@", derFilePath);
  }
}

- (void)loadPublicKeyWithData:(NSData *)derData {
  SecCertificateRef myCertificate = SecCertificateCreateWithData(kCFAllocatorDefault, (__bridge CFDataRef)derData);
  SecPolicyRef myPolicy = SecPolicyCreateBasicX509();
  SecTrustRef myTrust;
  OSStatus status = SecTrustCreateWithCertificates(myCertificate,myPolicy,&myTrust);
  SecTrustResultType trustResult;
  
  if (status == noErr) {
    status = SecTrustEvaluate(myTrust, &trustResult);
  }
  
  SecKeyRef securityKey = SecTrustCopyPublicKey(myTrust);
  CFRelease(myCertificate);
  CFRelease(myPolicy);
  CFRelease(myTrust);
  
  _publicKey = securityKey;
}

- (NSString *)rsaEncryptText:(NSString *)text {
  NSData *encryptedData = [self rsaEncryptData:[text hdf_toData]];
  NSString *base64EncryptedString = [NSString hdf_base64StringFromData:encryptedData
                                                                length:encryptedData.length];
  return base64EncryptedString;
}

- (NSData *)rsaEncryptData:(NSData *)data {
  SecKeyRef key = _publicKey;
  
  size_t cipherBufferSize = SecKeyGetBlockSize(key);
  uint8_t *cipherBuffer = malloc(cipherBufferSize * sizeof(uint8_t));
  size_t blockSize = cipherBufferSize - 11;       
  size_t blockCount = (size_t)ceil([data length] / (double)blockSize);
  
  NSMutableData *encryptedData = [[NSMutableData alloc] init] ;
  for (int i = 0; i < blockCount; i++) {
    size_t bufferSize = MIN(blockSize,[data length] - i * blockSize);
    NSData *buffer = [data subdataWithRange:NSMakeRange(i * blockSize, bufferSize)];
    OSStatus status = SecKeyEncrypt(key,
                                    kSecPaddingPKCS1,
                                    (const uint8_t *)[buffer bytes],
                                    [buffer length],
                                    cipherBuffer,
                                    &cipherBufferSize);
    if (status == noErr) {
      NSData *encryptedBytes = [[NSData alloc] initWithBytes:(const void *)cipherBuffer
                                                      length:cipherBufferSize];
      [encryptedData appendData:encryptedBytes];
    } else {
      if (cipherBuffer) {
        free(cipherBuffer);
      }
      
      return nil;
    }
  }
  
  if (cipherBuffer){
    free(cipherBuffer);
  }
  
  return encryptedData;
}

#pragma mark - 解密相关
- (void)loadPrivateKeyWithPath:(NSString *)p12FilePath password:(NSString *)p12Password {
  NSData *data = [NSData dataWithContentsOfFile:p12FilePath];
  
  if (data.length > 0) {
    [self loadPrivateKeyWithData:data password:p12Password];
  } else {
    NSLog(@"load private key fail with path: %@", p12FilePath);
  }
}

- (void)loadPrivateKeyWithData:(NSData *)p12Data password:(NSString *)p12Password {
  SecKeyRef privateKeyRef = NULL;
  NSMutableDictionary * options = [[NSMutableDictionary alloc] init];
  
  [options setObject:p12Password forKey:(__bridge id)kSecImportExportPassphrase];
  
  CFArrayRef items = CFArrayCreate(NULL, 0, 0, NULL);
  OSStatus securityError = SecPKCS12Import((__bridge CFDataRef)p12Data,
                                           (__bridge CFDictionaryRef)options,
                                           &items);
  
  if (securityError == noErr && CFArrayGetCount(items) > 0) {
    CFDictionaryRef identityDict = CFArrayGetValueAtIndex(items, 0);
    SecIdentityRef identityApp = (SecIdentityRef)CFDictionaryGetValue(identityDict,
                                                                      kSecImportItemIdentity);
    securityError = SecIdentityCopyPrivateKey(identityApp, &privateKeyRef);
    
    if (securityError != noErr) {
      privateKeyRef = NULL;
    }
  }
  
  _privateKey = privateKeyRef;
  
//  CFRelease(items);
}

- (NSString *)rsaDecryptText:(NSString *)text {
  NSData *data = [NSData hdf_base64DataFromString:text];
  
  NSData *decryptData = [self rsaDecryptData:data];
  
  NSString *result = [[NSString alloc] initWithData:decryptData encoding:NSUTF8StringEncoding];
  return result;
}

- (NSData *)rsaDecryptData:(NSData *)data {
  SecKeyRef key = _privateKey;
  
  size_t cipherLen = [data length];
  void *cipher = malloc(cipherLen);
  
  [data getBytes:cipher length:cipherLen];
  size_t plainLen = SecKeyGetBlockSize(key) - 12;
  
  void *plain = malloc(plainLen);
  OSStatus status = SecKeyDecrypt(key, kSecPaddingPKCS1, cipher, cipherLen, plain, &plainLen);
  
  if (status != noErr) {
    return nil;
  }
  
  NSData *decryptedData = [[NSData alloc] initWithBytes:(const void *)plain length:plainLen];
  
  return decryptedData;
}

@end
```

#写在最后

笔者经历了九九八十一难才调通，在前端实现`RSA`加密和解密，并验证成功。这里面关系到很多方面的知识，大家可以参考苹果官方关于安全性方面的文档。如果这里写得不好或者读完文章您还不懂怎么做，不要担心，这是很正常的。因为笔者也是经过了好几天的研究和讨论才在安全性方面有所了解。

#[阅读原文](http://www.henishuo.com/talk-about-rsa-encrypt-decrypt/)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/)**

