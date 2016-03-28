#前言

在开发中，基本都需要到注册，而注册通常要输入一些身份信息，当然前端也是需要验证是否是合法的身份证才能提交。这篇文章只是放出个人的代码，不代表全正确！！！

#扩展NSString代码

扩展NSString，添加以下方法：

```
- (BOOL)hyb_isValidPersonID {
  return [NSString hyb_isValidPersonID:self];
}

+ (BOOL)hyb_isValidPersonID:(NSString *)personId {
  // 判断位数
  if (personId.length != 15 && personId.length != 18) {
    return NO;
  }
  NSString *carid = personId;
  long lSumQT = 0;
  // 加权因子
  int R[] ={7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2 };
  // 校验码
  unsigned char checkers[11]={'1','0','X', '9', '8', '7', '6', '5', '4', '3', '2'};
  
  // 将15位身份证号转换成18位
  NSMutableString *str = [NSMutableString stringWithString:personId];
  if (personId.length == 15) {
    [str insertString:@"19" atIndex:6];
    long p = 0;
    const char *personId = [str UTF8String];
    
    for (int i = 0; i<= 16; i++) {
      p += (personId[i] - 48) * R[i];
    }
    
    int o = p % 11;
    NSString *string_content = [NSString stringWithFormat:@"%c", checkers[o]];
    [str insertString:string_content atIndex:[str length]];
    carid = str;
  }
  
  // 判断地区码
  NSString * sProvince = [carid substringToIndex:2];
  if (![self _areaCode:sProvince]) {
    return NO;
  }
  
  // 判断年月日是否有效
  // 年份
  int strYear = [[self _substringWithString:carid begin:6 end:4] intValue];
  // 月份
  int strMonth = [[self _substringWithString:carid begin:10 end:2] intValue];
  // 日
  int strDay = [[self _substringWithString:carid begin:12 end:2] intValue];
  
  NSTimeZone *localZone = [NSTimeZone localTimeZone];
  NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
  [dateFormatter setDateStyle:NSDateFormatterMediumStyle];
  [dateFormatter setTimeStyle:NSDateFormatterNoStyle];
  [dateFormatter setTimeZone:localZone];
  [dateFormatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
  NSDate *date=[dateFormatter dateFrostr:[NSString stringWithFormat:@"%d-%d-%d 12:01:01",
                                              strYear, strMonth, strDay]];
  if (date == nil) {
    return NO;
  }
  
  const char *pid  = [carid UTF8String];
  // 检验长度
  if(18 != strlen(pid)) return NO;
  // 校验数字
  for (int i = 0; i < 18; i++) {
    if ( !isdigit(pid[i]) && !(('X' == pid[i] || 'x' == pid[i]) && 17 == i) ) {
      return NO;
    }
  }
  
  // 验证最末的校验码
  for (int i = 0; i <= 16; i++) {
    lSumQT += (pid[i]-48) * R[i];
  }
  
  if (checkers[lSumQT%11] != pid[17] ) {
    return NO;
  }
  return YES;
}


#pragma mark - Private
+ (NSString *)_substringWithString:(NSString *)str begin:(NSInteger)begin end:(NSInteger )end {
  return [str substringWithRange:NSMakeRange(begin, end)];
}

/**
 * 功能:判断是否在地区码内
 * 参数:地区码
 */
+ (BOOL)_areaCode:(NSString *)code {
  NSMutableDictionary *dic = [[NSMutableDictionary alloc] init];
  [dic setObject:@"北京" forKey:@"11"];
  [dic setObject:@"天津" forKey:@"12"];
  [dic setObject:@"河北" forKey:@"13"];
  [dic setObject:@"山西" forKey:@"14"];
  [dic setObject:@"内蒙古" forKey:@"15"];
  [dic setObject:@"辽宁" forKey:@"21"];
  [dic setObject:@"吉林" forKey:@"22"];
  [dic setObject:@"黑龙江" forKey:@"23"];
  [dic setObject:@"上海" forKey:@"31"];
  [dic setObject:@"江苏" forKey:@"32"];
  [dic setObject:@"浙江" forKey:@"33"];
  [dic setObject:@"安徽" forKey:@"34"];
  [dic setObject:@"福建" forKey:@"35"];
  [dic setObject:@"江西" forKey:@"36"];
  [dic setObject:@"山东" forKey:@"37"];
  [dic setObject:@"河南" forKey:@"41"];
  [dic setObject:@"湖北" forKey:@"42"];
  [dic setObject:@"湖南" forKey:@"43"];
  [dic setObject:@"广东" forKey:@"44"];
  [dic setObject:@"广西" forKey:@"45"];
  [dic setObject:@"海南" forKey:@"46"];
  [dic setObject:@"重庆" forKey:@"50"];
  [dic setObject:@"四川" forKey:@"51"];
  [dic setObject:@"贵州" forKey:@"52"];
  [dic setObject:@"云南" forKey:@"53"];
  [dic setObject:@"西藏" forKey:@"54"];
  [dic setObject:@"陕西" forKey:@"61"];
  [dic setObject:@"甘肃" forKey:@"62"];
  [dic setObject:@"青海" forKey:@"63"];
  [dic setObject:@"宁夏" forKey:@"64"];
  [dic setObject:@"新疆" forKey:@"65"];
  [dic setObject:@"台湾" forKey:@"71"];
  [dic setObject:@"香港" forKey:@"81"];
  [dic setObject:@"澳门" forKey:@"82"];
  [dic setObject:@"国外" forKey:@"91"];
  
  if ([dic objectForKey:code] == nil) {
    return NO;
  }
  return YES;
}
```

#温馨提示

这段代码是严格地验证15位和18位的身份证的，并且x/X是不同的，x是不能通过的。

