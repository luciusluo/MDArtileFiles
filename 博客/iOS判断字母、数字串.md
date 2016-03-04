>#iOS判断字母、数字串

---
在项目开发中，经常需要到这些`api`，下面这三个方法是笔者自己封装的。

以下为`NSString`类的扩展方法，分别是判断字符串是否只是包含字母、是否只包含数字、是否只包含字母和数字：

```
- (BOOL)hyb_isOnlyLetters {
  NSCharacterSet *letterCharacterset = [[NSCharacterSet letterCharacterSet] invertedSet];
  return ([self rangeOfCharacterFromSet:letterCharacterset].location == NSNotFound);
}

- (BOOL)hyb_isOnlyNumbers {
  NSCharacterSet *numSet = [[NSCharacterSet characterSetWithCharactersInString:@"0123456789"] invertedSet];
  return ([self rangeOfCharacterFromSet:numSet].location == NSNotFound);
}

- (BOOL)hyb_isOnlyAlphaNumeric {
  NSCharacterSet *numAndLetterCharSet = [[NSCharacterSet alphanumericCharacterSet] invertedSet];
  return ([self rangeOfCharacterFromSet:numAndLetterCharSet].location == NSNotFound);
}
```

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**