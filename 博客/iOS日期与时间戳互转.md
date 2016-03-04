>#iOS时间戳与日期互转

---
下面这两个方法是给`NSDate`的扩展方法：

##将`NSDate`对象转换成时间戳

事实上，由日期类转换成时间戳是很容易的，是相对于1970年的：

```
- (NSString *)hyb_toTimeStamp {
  return [NSString stringWithFormat:@"%lf", [self timeIntervalSince1970]];
}
```

将时间戳转换成日期对象也很容易，也要基于1970年计算：

```
+ (NSDate *)hyb_toDateWithTimeStamp:(NSString *)timeStamp {
  NSString *arg = timeStamp;
  if (![timeStamp isKindOfClass:[NSString class]]) {
    arg = [NSString stringWithFormat:@"%@", timeStamp];
  }
  NSTimeInterval time = [timeStamp doubleValue];
  return [NSDate dateWithTimeIntervalSince1970:time];
}
```

#测试

---
###转换成时间戳

```
NSDate *date = [NSDate date];
NSString timeStamp = [date hyb_toTimeStamp];
```

###转换成日期

```
NSDate *date = [NSDate hyb_toDateWithTimeStamp:timeStamp];
```

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**