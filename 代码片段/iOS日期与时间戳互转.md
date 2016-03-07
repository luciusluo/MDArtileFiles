#前言

在开发中经常操作日期与时间戳，这只是日期与时间戳互转的代码片段：

下面这两个方法是给`NSDate`的扩展方法：


#将NSDate对象转换成时间戳

事实上，由日期类转换成时间戳是很容易的，是相对于1970年的：

```
- (NSString *)hyb_toTimeStamp {
  return [NSString stringWithFormat:@"%lf", [self timeIntervalSince1970]];
}
```

#将时间戳转换成NSDate对象

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

#如何使用

日期对象转换成时间戳：

```
NSDate *date = [NSDate date];
NSString timeStamp = [date hyb_toTimeStamp];
```

时间戳转换成日期：

```
NSDate *date = [NSDate hyb_toDateWithTimeStamp:timeStamp];
```

#注意事项

可能有人使用这么转换（网上有人发帖问4s上出现问题而其它手机没有问题）：

```
NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
[formatter setDateStyle:NSDateFormatterMediumStyle];
[formatter setTimeStyle:NSDateFormatterShortStyle];

NSDate *confromTimesp = [NSDate dateWithTimeIntervalSince1970:[self.appointmentServiceTime integerValue]/1000];

[formatter setDateFormat:@"YYYY.MM.dd"];
NSString *str = [formatter stringFromDate:confromTimesp];
```

其实问题出现在`[self.appointmentServiceTime integerValue]`，换成doubleValue就都一样了。因为4s上integerValue是int\_32，而是5s上integerValue是int\_64，所以时间戳这个值是大于32位int值的范围的，所以4s上有问题。

**注意：**如果后台给你返回来的是时间戳，请不要使用integerValue转换，一定要使用doubleValue。

