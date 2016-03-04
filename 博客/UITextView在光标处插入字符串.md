>#UITextView在光标处插入字符串

---
下面是某次需求中要求在光标处插入所导入的字符串，并且以`，`分开。
这里添加了比较多的逻辑处理，过滤一些字符。另外，在6.0系统上获取`selectedRange.location`会出现`NSNotFound`等，因此还处理添加特殊处理。下面的代码是兼容到`iOS 6.0`的。

```
#pragma mark -- 更新插入数据到光标处
- (void)updateTextViewTextInsertedString:(NSString *)text {
  if (kIsEmptyString(text)) {
    return;
  }
  
  // 获得光标所在的位置
  NSUInteger location = self.diseaseDescTextView.selectedRange.location;
  if (location == NSNotFound || location >= self.diseaseNameTextField.text.length) {
    if (kIsEmptyString(self.diseaseNameTextField.text)) {
      text = [text substringFromIndex:1];
    }

    NSString *currentText = self.diseaseDescTextView.text;
    if (kIsEmptyString(currentText)) {
      currentText = @"";
    }
    self.diseaseDescTextView.text = [NSString stringWithFormat:@"%@%@",
                                     currentText,
                                     text];
  [self textViewDidChange:self.diseaseDescTextView];
    return;
  }
  
  // 如果光标之前没有内容，去掉前面的逗号
  if (kIsEmptyString([self.diseaseDescTextView.text substringToIndex:location])) {
    if ([text hasPrefix:@"，"]) {
      if (text.length == 1) {
        text = @"";
      } else {
        text = [text substringFromIndex:1];
      }
    }
  }
  
  if (kIsEmptyString(self.diseaseDescTextView.text)) {
    self.diseaseDescTextView.text = text;
    [self textViewDidChange:self.diseaseDescTextView];
    return;
  }
  
  if (!kIsEmptyString([self.diseaseDescTextView.text substringFromIndex:location])) {
    text = [NSString stringWithFormat:@"%@，", text];
  }
  
  NSString *preText = [self.diseaseDescTextView.text substringToIndex:location];
  if (kIsEmptyString(preText)) {
    preText = @"";
  }
  
  NSString *lastText = [self.diseaseDescTextView.text substringFromIndex:location];
  if (kIsEmptyString(lastText)) {
    lastText = @"";
  }
  
  NSString *result = [NSString stringWithFormat:@"%@%@%@",
                      preText,
                      text,
                      lastText];
  
  self.diseaseDescTextView.text = result;
  [self textViewDidChange:self.diseaseDescTextView];
  
  // 调整光标
  self.diseaseDescTextView.selectedRange = NSMakeRange(location + text.length + 1, 1);
}
```

看不懂的可以在评论中回复，笔者会收到邮件通知。

#[阅读原文](http://www.henishuo.com/uitextview-cursor-insert-string/)

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**