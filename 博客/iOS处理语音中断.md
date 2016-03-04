>#iOS在语音时处理中断

---
在语音时，如果当前用户正在播放音乐之类的，那么我们必须要中断之，在用户语音完成时，又要通知其中断已完成，以便恢复之前的音乐播放等。看代码吧：

下面的代码是中断处理：

```
if ([[[UIDevice currentDevice] systemVersion] compare:@"7.0"] != NSOrderedAscending) {
    __block BOOL    bCanRecord = NO;
    AVAudioSession  *audioSession = [AVAudioSession sharedInstance];
    
    if ([audioSession respondsToSelector:@selector(requestRecordPermission:)]) {
      [audioSession performSelector:@selector(requestRecordPermission:) withObject:^(BOOL granted) {
        if (granted) {
          bCanRecord = YES;
        } else {
          bCanRecord = NO;
          dispatch_async(dispatch_get_main_queue(), ^{
            [[[UIAlertView alloc] initWithTitle:nil
                           message             :@"需要访问您的麦克风。\n请启用麦克风-设置/隐私/麦克风"
                           delegate            :nil
                           cancelButtonTitle   :@"关闭"
                           otherButtonTitles   :nil] show];
          });
        }
      }];
    }
    
    if (!bCanRecord) {
      return;
    }
}
```

当我们语音完毕之后，我们需要通知中断已经完成，以便恢复原始状态，该播放音乐就播放：

```
[[AVAudioSession sharedInstance] setActive:NO
                                   withFlags:AVAudioSessionSetActiveOptionNotifyOthersOnDeactivation
                                       error:nil];
```

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**