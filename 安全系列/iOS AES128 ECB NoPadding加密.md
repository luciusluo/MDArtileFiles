#前言

谈谈AES加密，网上有很多的版本，当我没有真正在加密安全问题前，总以为百度出来某个AES加密算法就可以直接使用，实际上当我真正要做加密时，遇到了很多的坑，原来不是拿过来就能用的。写下本篇文章，记录下曾经遇到的坑，严防以后再出现同样的坑。

#AES规则 

原输入数据不够16字节的整数位时，就要补齐。因此就会有padding，若使用不同的padding，那么加密出来的结果也会不一样。

#AES加密算法

苹果提供给我们的API只有这一个函数用来加密或者解密：

```
CCCryptorStatus CCCrypt(
    CCOperation op,         /* kCCEncrypt, etc. */
    CCAlgorithm alg,        /* kCCAlgorithmAES128, etc. */
    CCOptions options,      /* kCCOptionPKCS7Padding, etc. */
    const void *key,
    size_t keyLength,
    const void *iv,         /* optional initialization vector */
    const void *dataIn,     /* optional per op and alg */
    size_t dataInLength,
    void *dataOut,          /* data RETURNED here */
    size_t dataOutAvailable,
    size_t *dataOutMoved)
    __OSX_AVAILABLE_STARTING(__MAC_10_4, __IPHONE_2_0);
```

* 其中第一个`CCOperation`只有两个值，要么是`kCCEncrypt`表示加密，要么是`kCCDecrypt`表示解密。

* 第二个参数表示加密的算法，它只有以下向种类型：

```
enum {
    kCCAlgorithmAES128 = 0,
    kCCAlgorithmAES = 0,
    kCCAlgorithmDES,
    kCCAlgorithm3DES,       
    kCCAlgorithmCAST,       
    kCCAlgorithmRC4,
    kCCAlgorithmRC2,   
    kCCAlgorithmBlowfish    
};
typedef uint32_t CCAlgorithm;
```

我们这里使用的是`kCCAlgorithmAES128`表示使用AES128位加密。

* 第三个参数表示选项，这里使用的是`kCCOptionECBMode`，表示ECB：

```
enum {
    /* options for block ciphers */
    kCCOptionPKCS7Padding   = 0x0001,
    kCCOptionECBMode        = 0x0002
    /* stream ciphers currently have no options */
};
typedef uint32_t CCOptions;
```

* 第四个参数表示加密/解密的密钥。
* 第五个参数keyLength表示密钥的长度。
* 第六个参数iv是个固定值，通过直接使用密钥即可。大家一定要注视这个参数，如果安卓、服务端和iOS端不统一，那么加密结果就会不一样，解密可能能解出来，但是解密后在末尾会出现一些\0、\t之类的。
* 第七个参数dataIn表示要加密/解密的数据。
* 第八个参数dataInLength表示要加密/解密的数据的长度。
* 第九个参数dataOut用于接收加密后/解密后的结果。
* 第十个参数dataOutAvailable表示加密后/解密后的数据的长度。
* 第十一个参数dataOutMoved表示实际加密/解密的数据的长度。（因为有补齐）



#加密算法

依赖于第三方库：[GTMBase64](https://github.com/r258833095/GTMBase64)，这个库已经几年没有维护了，现在还是MRC版本，要使用请到GITHUB查看使用教程，那里有ARC接入说明：

```
+ (NSString *)hyb_AESEncrypt:(NSString *)plainText password:(NSString *)key {
  if (key == nil || (key.length != 16 && key.length != 32)) {
    return nil;
  }
  
  char keyPtr[kCCKeySizeAES128+1];
  memset(keyPtr, 0, sizeof(keyPtr));
  [key getCString:keyPtr maxLength:sizeof(keyPtr) encoding:NSUTF8StringEncoding];
  
  
  char ivPtr[kCCBlockSizeAES128+1];
  memset(ivPtr, 0, sizeof(ivPtr));
  [key getCString:ivPtr maxLength:sizeof(ivPtr) encoding:NSUTF8StringEncoding];
  
  NSData* data = [plainText dataUsingEncoding:NSUTF8StringEncoding];
  NSUInteger dataLength = [data length];
  
  int diff = kCCKeySizeAES128 - (dataLength % kCCKeySizeAES128);
  unsigned long newSize = 0;
  
  if(diff > 0) {
    newSize = dataLength + diff;
  }
  
  char dataPtr[newSize];
  memcpy(dataPtr, [data bytes], [data length]);
  for(int i = 0; i < diff; i++) {
    // 这里是关键，这里是使用NoPadding的
    dataPtr[i + dataLength] = 0x0000;
  }
  
  size_t bufferSize = newSize + kCCBlockSizeAES128;
  void *buffer = malloc(bufferSize);
  memset(buffer, 0, bufferSize);
  
  size_t numBytesCrypted = 0;
  
  CCCryptorStatus cryptStatus = CCCrypt(kCCEncrypt,
                                        kCCAlgorithmAES128,
                                        kCCOptionECBMode,
                                        [key UTF8String],
                                        kCCKeySizeAES128,
                                        ivPtr,
                                        dataPtr,
                                        sizeof(dataPtr),
                                        buffer,
                                        bufferSize,
                                        &numBytesCrypted);
  
  if (cryptStatus == kCCSuccess) {
    NSData *resultData = [NSData dataWithBytesNoCopy:buffer length:numBytesCrypted];
    return [GTMBase64 stringByEncodingData:resultData];
  }
  
  free(buffer);
  return nil;
}
```

对于加密算法，大家一定要注意，保证iOS、安卓、服务端的加密规则是一定的，建议统一使用No Padding的，这里使用No Padding是这样的：

```
for(int i = 0; i < diff; i++) {
	// 这里是关键，这里是使用NoPadding的
	dataPtr[i + dataLength] = 0x0000;
}
```

其实所谓Padding就是指在位数不够需要补齐时，使用什么来填充，而No Padding就是使用16个0，对应0x0000.如果三端不统一，加密出来就算能解密，也会出现一些奇怪的字符，甚至会有部分乱码。

另外，这里使用的是`kCCOptionECBMode`，也就是ECB。在安卓端和PHP端，也得使用ECB。在调试过程中，发现PHP使用CBC解密不了IOS端的。于是改成了使用ECB。

#解密算法

依赖于第三方库：[GTMBase64](https://github.com/r258833095/GTMBase64)，这个库已经几年没有维护了，现在还是MRC版本，要使用请到GITHUB查看使用教程，那里有ARC接入说明：

```
+ (NSString *)hyb_AESDecrypt:(NSString *)encryptText password:(NSString *)key {
  if (key == nil || (key.length != 16 && key.length != 32)) {
    return nil;
  }
  
  char keyPtr[kCCKeySizeAES128 + 1];
  memset(keyPtr, 0, sizeof(keyPtr));
  [key getCString:keyPtr maxLength:sizeof(keyPtr) encoding:NSUTF8StringEncoding];
  
  char ivPtr[kCCBlockSizeAES128 + 1];
  memset(ivPtr, 0, sizeof(ivPtr));
  [key getCString:ivPtr maxLength:sizeof(ivPtr) encoding:NSUTF8StringEncoding];
  
  NSData *data = [GTMBase64 decodeData:[encryptText dataUsingEncoding:NSUTF8StringEncoding]];
  NSUInteger dataLength = [data length];
  size_t bufferSize = dataLength + kCCBlockSizeAES128;
  void *buffer = malloc(bufferSize);
  
  size_t numBytesCrypted = 0;
  CCCryptorStatus cryptStatus = CCCrypt(kCCDecrypt,
                                        kCCAlgorithmAES128,
                                        kCCOptionECBMode,
                                        [key UTF8String],
                                        kCCBlockSizeAES128,
                                        ivPtr,
                                        [data bytes],
                                        dataLength,
                                        buffer,
                                        bufferSize,
                                        &numBytesCrypted);
  if (cryptStatus == kCCSuccess) {
    NSData *resultData = [NSData dataWithBytesNoCopy:buffer length:numBytesCrypted];
    
    NSString *decoded=[[NSString alloc] initWithData:resultData encoding:NSUTF8StringEncoding];
    return decoded;
  }
  
  free(buffer);
  return nil;
}
```

解密时也得跟加密一样指定为ECB，否则解出来会出现乱码，或者末尾会出现\0、\t之类的符号。

#写在最后

开发中总会遇到各种坑，网上查了很多的资料，但是终究没有说明解决的办法，而是只将自己的代码放出来。对于刚接触这方面知识的开发人员来说，是很懵懂的。甚至很多新手会觉得系统就是这样的，我也没办法。其实总会有解决办法的，关键在于与其他各端统一连调。

#关注我


**Swift/ObjC技术群一：324400294(已满)**

**Swift/ObjC技术群二：494669518**

**ObjC/Swift高级群：461252383（注明年限，新手勿扰）**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**原文有更新，请阅读原文**：[http://www.henishuo.com/ios-aes128-ecb-nopadding/](http://www.henishuo.com/ios-aes128-ecb-nopadding/)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
