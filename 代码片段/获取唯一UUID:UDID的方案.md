#概述

如何保证获取到的UUID能够唯一标识每一台设备呢？我们知道通过UIDevice可以获取到UUIDString，但是如果App被删除了然后重新安装，就会得到不同的UUIDString，这并不是我们希望的。

那么，有什么办法可以解决这个问题呢？这里不说5.0之前的一切，只说6.0之后的如何做到。

下面提供的只是代码片段，不是完整的代码！

#案例

苹果在iOS6.0版本之后，在UIDevice提供了以下属性：

```
@property(nullable, nonatomic,readonly,strong) NSUUID      *identifierForVendor 
```

通过这个属性，就可以获取到UUID：

```
/* Return a string description of the UUID, such as "E621E1F8-C36C-495A-93FC-0C247A3E6E5F" */
@property (readonly, copy) NSString *UUIDString;
```

我们来操作一下，先运行某个App，然后打印UUIDString：

```
B907009B-8C63-4CA8-B3FB-B2724AE96DD5
```

然后删除掉这个App，再重新安装，然后再打印：

```
B907009B-8C63-4CA8-B3FB-B2724AE96DD5
```

为什么是一样的呢？不是说会变吗？是的，因为手机上还安装了其它跟这个App同属于同一开发商的App。现在，我们把所有与它是同一开发商的App全部删除，再重新安装，再打印如下：

```
1A73D3A3-3CD6-4655-8566-042FE7C8B2AC
```

果然发生了变化了。

#官方文档

**苹果官方文档这么说的**：


The value of this property is the same for apps that come from the same vendor running on the same device. A different value is returned for apps on the same device that come from different vendors, and for apps on different devices regardless of vendor.

The value of this property may be nil if the app is running in the background, before the user has unlocked the device the first time after the device has been restarted. If the value is nil, wait and get the value again later.

The value in this property remains the same while the app (or another app from the same vendor) is installed on the iOS device. The value changes when the user deletes all of that vendor’s apps from the device and subsequently reinstalls one or more of them. Therefore, if your app stores the value of this property anywhere, you should gracefully handle situations where the identifier changes.

**意思大概就是说**：

在同一设备上运行来源于同一开发商的App，获取到的UUIDString属性是同一个值。当在同一设备上运行来源于不同的开发商的App，所获取到的UUIDString是不同的。在不同的设备上，不管是否同属于同一个开发商，得到的UUIDString都会不同。

当设备重启后，若用户第一次未解锁设备，而app在后台运行时，这个UUIDString可能为nil。如果值为nil，请等待并在稍候重新获取。

当app或者另外来源于同一开发商的app被安装到同一设备上，这个UUIDString会保持一致（比如上面的小例子，打印出来就是一致的）。当用户删除掉设备上所有同一开发商的app后，重新安装其中某一个app，这时候所获取到的UUIDString就会发生变化。因此，不管app存储将这个UUID存储到哪里，你都应该手动处理这种改变。

那么如何解决这种改变呢？

#解决方案

解决方案就是能所生成的UUIDString存储到KeyChain中，使用同一个access group、同一个identifier。每次获取UUID，都先从KeyChain中获取，若为空，则通过UIDevice获取UUIDString并存储到KeyChaing，代码版本如下：

```
+ (NSString *)UUID {
    //MARK:此处的 @"5CKSJSE23P.com.haodf.mainGroup" 字串不能动，否则会导致获取的值错误或者 nil
    KeychainItemWrapper *wrapper = [[KeychainItemWrapper alloc] initWithIdentifier:@"HuangyibiaoAppID" accessGroup:@"com.huangyibiao.test.group"];
    NSString *UUID = [wrapper objectForKey:(__bridge id)kSecValueData];
    
    if (UUID.length == 0) {
      UUID = [[[UIDevice currentDevice] identifierForVendor] UUIDString];
      [wrapper setObject:UUID forKey:(__bridge id)kSecValueData];
    }
  
  return UUID;
}
```

请自行修改~

