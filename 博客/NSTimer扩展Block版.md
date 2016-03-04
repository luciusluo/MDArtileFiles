>#NSTimer扩展block版

---
对于定时器，基本每个应用都使用需要到的，但是原生API使用起来并不是那么方便，还得处理各种回调，对于开发时相对复杂了。因此，尝试写下这个`block`版本的，以简化调用！！！

#头文件

---
下面是扩展的头文件，看看公开了哪些API：

```
//
//  NSTimer+Convenience.h
//  NSTimerBlockDemo
//
//  Created by huangyibiao on 15/3/25.
//  Copyright (c) 2015年 huangyibiao. All rights reserved.
//

#import <Foundation/Foundation.h>

@interface NSTimer (Convenience)

/**
 *  无参数无返回值Block
 */
typedef void (^HYBVoidBlock)(void);

/**
 *  创建Timer---Block版本
 *
 *  @param interval 每隔interval秒就回调一次callback
 *  @param repeats  是否重复
 *  @param callback 回调block
 *
 *  @return NSTimer对象
 */
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                    repeats:(BOOL)repeats
                                   callback:(HYBVoidBlock)callback;

/**
 *  创建Timer---Block版本
 *
 *  @param interval 每隔interval秒就回调一次callback
 *  @param count  回调多少次后自动暂停，如果count <= 0，则表示无限次，否则表示具体的次数
 *  @param callback 回调block
 *
 *  @return NSTimer对象
 */
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                    count:(NSInteger)count
                                   callback:(HYBVoidBlock)callback;

/**
 *  开始启动定时器
 */
- (void)fireTimer;

/**
 *  暂停定时器
 */
- (void)unfireTimer;

@end
```

#实现文件

---
重点是实现文件，这里使用了非常巧妙的方式：让`NSTimer`本身的`userInfo`属性来存储回调！

```
//
//  NSTimer+Convenience.m
//  NSTimerBlockDemo
//
//  Created by huangyibiao on 15/3/25.
//  Copyright (c) 2015年 huangyibiao. All rights reserved.
//

#import "NSTimer+Convenience.h"

@implementation NSTimer (Convenience)

+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                    repeats:(BOOL)repeats
                                   callback:(HYBVoidBlock)callback {
  return [NSTimer scheduledTimerWithTimeInterval:interval
                                          target:self
                                        selector:@selector(onTimerUpdateBlock:)
                                        userInfo:[callback copy]
                                         repeats:repeats];
}

+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                      count:(NSInteger)count
                                   callback:(HYBVoidBlock)callback {
  NSDictionary *userInfo = @{@"callback"     : [callback copy],
                             @"count"        : @(count)};
  return [NSTimer scheduledTimerWithTimeInterval:interval
                                          target:self
                                        selector:@selector(onTimerUpdateCountBlock:)
                                        userInfo:userInfo
                                         repeats:YES];
}

+ (void)onTimerUpdateBlock:(NSTimer *)timer {
  HYBVoidBlock block = timer.userInfo;
  
  if (block) {
    block();
  }
}

+ (void)onTimerUpdateCountBlock:(NSTimer *)timer {
  static NSUInteger currentCount = 0;
  
  NSDictionary *userInfo = timer.userInfo;
  HYBVoidBlock callback = userInfo[@"callback"];
  NSNumber *count = userInfo[@"count"];
  
  if (count.integerValue <= 0) {
    if (callback) {
      callback();
    }
  } else {
    if (currentCount < count.integerValue) {
      currentCount++;
      if (callback) {
        callback();
      }
    } else {
      currentCount = 0;
      
      [timer unfireTimer];
    }
  }
}

- (void)fireTimer {
  [self setFireDate:[NSDate distantPast]];
}

- (void)unfireTimer {
  [self setFireDate:[NSDate distantFuture]];
}

@end
```

代码中很关键的代码，获取回调，再获取已经回调了的次数，用于处理回调次数限制的情况：

```
NSDictionary *userInfo = timer.userInfo;
HYBVoidBlock callback = userInfo[@"callback"];
NSNumber *count = userInfo[@"count"];
```

#源代码

---
想要下载Demo？狂点这里：[https://github.com/CoderJackyHuang/HYBTimerExtension](https://github.com/CoderJackyHuang/HYBTimerExtension)