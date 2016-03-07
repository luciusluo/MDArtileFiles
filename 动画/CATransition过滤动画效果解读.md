#前言

---
曾经，笔者对动画一无所知，当他人问起时，总是似懂非懂。每一次别人说起动画效果时，笔者都不好意思插话，因此懂得太少，只是会使用`UIView`的那几个添加动画的方法。现在，不再等待，一步一步地学习其基础知识并开始尝试写一些常用的动画效果。

如果您也一样迷茫，那就不要迷茫了，实践出真知！！！

#基础知识

---
我们直接看官方声明：

```
/** Transition animation subclass. **/

@interface CATransition : CAAnimation

/* The name of the transition. Current legal transition types include
 * `fade', `moveIn', `push' and `reveal'. Defaults to `fade'. */

@property(copy) NSString *type;

/* An optional subtype for the transition. E.g. used to specify the
 * transition direction for motion-based transitions, in which case
 * the legal values are `fromLeft', `fromRight', `fromTop' and
 * `fromBottom'. */

@property(nullable, copy) NSString *subtype;

/* The amount of progress through to the transition at which to begin
 * and end execution. Legal values are numbers in the range [0,1].
 * `endProgress' must be greater than or equal to `startProgress'.
 * Default values are 0 and 1 respectively. */

@property float startProgress;
@property float endProgress;

/* An optional filter object implementing the transition. When set the
 * `type' and `subtype' properties are ignored. The filter must
 * implement `inputImage', `inputTargetImage' and `inputTime' input
 * keys, and the `outputImage' output key. Optionally it may support
 * the `inputExtent' key, which will be set to a rectangle describing
 * the region in which the transition should run. Defaults to nil. */

@property(nullable, strong) id filter;

@end
```

`CATransition`类继承于`CAAnimation`类，提供的是过滤的效果，如`push`、`fade`、`reveal`等。

`type`属性是用于指定效果类型，当前官方提供的效果有`fade`, `moveIn`, `push`和`reveal`. 默认为`fade`。对于其它类型，如`cube`立体效果这种官方没有公开，也不清楚是否是使用私有。

`subtype`属性是可选的，主要用于指定动画的方向。比如动作类动画效果中，有从左边进入、从右边进入等效果。

```
@property float startProgress;
@property float endProgress;
```
这两个属性可以设置动画动作的进度，默认为0->1。

`filter`属性默认为`nil`，一旦设置了此属性，`type`和`subtype`就会被忽略。 这个属性意思就是滤镜的意思吧，它需要实现`inputImage`、`inputTargetImage`、`inputTime`、`outputImage`，当然还有一个可选的`inputExtent`，不要求实现。

>**更多基础知识，请参考：[CAAnimation精讲](http://www.henishuo.com/caanimation-indtroduce-in-detail/)**

#实战练习做动画

---
先看看我们做效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/transition.gif)


常用的`transition`动画几乎都有了，而且笔者在学习的同时，也这将些动画封装成了一个类方法，只需要一行代码就可以实现动画效果了哦！

##头文件声明

这里只公共了一个方法，并将常用的动画使用一个枚举类型来指定，不用再记着那些单词了。

```
//
//  HYBTransitionAnimation.h
//  CATransitionOfObjCDemo
//
//  Created by huangyibiao on 15/12/14.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSUInteger, HYBTransitionType) {
  kHYBTransitionFade = 1,     // 淡入淡出
  kHYBTransitionPush,         // 推进效果
  kHYBTransitionReveal,       // 揭开效果
  kHYBTransitionMoveIn,       // 慢慢进入并覆盖效果
  kHYBTransitionCube,         // 立体翻转效果
  kHYBTransitionSuckEffect,   // 像被吸入瓶子的效果
  kHYBTransitionRippleEffect, // 波纹效果
  kHYBTransitionPageCurl,     // 翻页效果
  kHYBTransitionPageUnCurl,   // 反翻页效果
  kHYBTransitionCameraOpen,   // 开镜头效果
  kHYBTransitionCameraClose,  // 关镜头效果
  kHYBTransitionCurlDown,     // 下翻页效果
  kHYBTransitionCurlUp,       // 上翻页效果
  kHYBTransitionFlipFromLeft, // 左翻转效果
  kHYBTransitionFlipFromRight,// 右翻转效果
  kHYBTransitionOglFlip       // 翻转
};

typedef NS_ENUM(NSUInteger, HYBTransitionSubtype) {
  kHYBTransitionSubtypeFromLeft = 1,  // 从左边进入
  kHYBTransitionSubtypeFromRight,     // 从右边进入
  kHYBTransitionSubtypeFromTop,       // 从顶部进入
  kHYBTransitionSubtypeFromBottom     // 从底部进入
};

@interface HYBTransitionAnimation : NSObject

+ (void)transitionForView:(UIView *)aView
                               type:(HYBTransitionType)type
                            subtype:(HYBTransitionSubtype)subtype
                           duration:(NSTimeInterval)duration;

@end
```

##实现文件

其实实现的代码也很简单，只是对枚举类型判断一下，然后添加动画：

```
//
//  HYBTransitionAnimation.m
//  CATransitionOfObjCDemo
//
//  Created by huangyibiao on 15/12/14.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import "HYBTransitionAnimation.h"

@implementation HYBTransitionAnimation

+ (void)transitionForView:(UIView *)aView
                               type:(HYBTransitionType)type
                            subtype:(HYBTransitionSubtype)subtype
                           duration:(NSTimeInterval)duration {
  NSString *animationType = nil;
  NSString *animationSubtype = nil;
  
  switch (subtype) {
    case kHYBTransitionSubtypeFromLeft:
      animationSubtype = kCATransitionFromLeft;
      break;
      case kHYBTransitionSubtypeFromRight:
      animationSubtype = kCATransitionFromRight;
      break;
      case kHYBTransitionSubtypeFromTop:
      animationSubtype = kCATransitionFromTop;
      break;
      case kHYBTransitionSubtypeFromBottom:
      animationSubtype = kCATransitionFromBottom;
      break;
    default:
      break;
  }
  
  switch (type) {
    case kHYBTransitionFade: {
      animationType = kCATransitionFade;
      break;
    }
    case kHYBTransitionPush: {
      animationType = kCATransitionPush;
      break;
    }
    case kHYBTransitionReveal: {
      animationType = kCATransitionReveal;
      break;
    }
    case kHYBTransitionMoveIn: {
      animationType = kCATransitionMoveIn;
      break;
    }
    case kHYBTransitionCube: {
      animationType = @"cube";
      break;
    }
    case kHYBTransitionSuckEffect: {
      animationType = @"suckEffect";
      break;
    }
    case kHYBTransitionRippleEffect: {
      animationType = @"rippleEffect";
      break;
    }
    case kHYBTransitionPageCurl: {
      animationType = @"pageCurl";
      break;
    }
    case kHYBTransitionPageUnCurl: {
      animationType = @"pageUnCurl";
      break;
    }
    case kHYBTransitionCameraOpen: {
      animationType = @"cameraIrisHollowOpen";
      break;
    }
    case kHYBTransitionCameraClose: {
      animationType = @"cameraIrisHollowClose";
      break;
    }
    case kHYBTransitionCurlDown: {
      [self animationForView:aView type:UIViewAnimationTransitionCurlDown duration:duration];
      break;
    }
    case kHYBTransitionCurlUp: {
      [self animationForView:aView type:UIViewAnimationTransitionCurlUp duration:duration];
      break;
    }
    case kHYBTransitionFlipFromLeft: {
      [self animationForView:aView type:UIViewAnimationTransitionFlipFromLeft duration:duration];
      break;
    }
    case kHYBTransitionFlipFromRight: {
      [self animationForView:aView type:UIViewAnimationTransitionFlipFromRight duration:duration];
      break;
    }
      case kHYBTransitionOglFlip:
      animationType = @"oglFlip";
      break;
    default: {
      break;
    }
  }
  
  if (animationType != nil) {
    CATransition *animation = [CATransition animation];
    animation.duration = duration;
    animation.type = animationType;
    
    if (animationSubtype != nil) {
      animation.subtype = animationSubtype;
    }
    
    animation.timingFunction = UIViewAnimationOptionCurveEaseInOut;
    
    [aView.layer addAnimation:animation forKey:@"animation"];
  }
}

+ (void)animationForView:(UIView *)aView
                    type:(UIViewAnimationTransition)type
                duration:(NSTimeInterval)duration {
  [UIView animateWithDuration:duration animations:^{
    [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
    [UIView setAnimationTransition:type forView:aView cache:YES];
  }];
}

@end
```

##解析 

我们的核心添加动画的代码是：

```
CATransition *animation = [CATransition animation];
animation.duration = duration;
animation.type = animationType;
    
if (animationSubtype != nil) {
  animation.subtype = animationSubtype;
}
    
animation.timingFunction = UIViewAnimationOptionCurveEaseInOut;
    
[aView.layer addAnimation:animation forKey:@"animation"];
```

系统提供给我们一些动画是在`UIView`上提供的方法，我们可以看看这个枚举：

```
typedef NS_ENUM(NSInteger, UIViewAnimationTransition) {
    UIViewAnimationTransitionNone,
    UIViewAnimationTransitionFlipFromLeft,
    UIViewAnimationTransitionFlipFromRight,
    UIViewAnimationTransitionCurlUp,
    UIViewAnimationTransitionCurlDown,
};
```

我们添加动画只需要调用`UIView`添加动画的方法就可以实现了：

```
[UIView animateWithDuration:duration animations:^{
	[UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
	[UIView setAnimationTransition:type forView:aView cache:YES];
}];
```

#测试效果

---
我们在`ViewController`这里尝试一下效果，这里只是使用定时器每一秒就自动切换一种效果：

```
//
//  ViewController.m
//  CATransitionOfObjCDemo
//
//  Created by huangyibiao on 15/12/14.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import "ViewController.h"
#import "HYBTransitionAnimation.h"

@interface ViewController ()

@property (nonatomic, assign) int subtype;
@property (nonatomic, strong) NSArray *array;

@property (nonatomic, strong) UIImage *img1;
@property (nonatomic, strong) UIImage *img2;
@property (nonatomic, assign) BOOL isImg1;
@property (nonatomic, assign) NSUInteger index;

@end

@implementation ViewController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.array = @[@(kHYBTransitionFade),
                 @(kHYBTransitionPush),
                 @(kHYBTransitionReveal),
                 @(kHYBTransitionMoveIn),
                 @(kHYBTransitionCube),
                 @(kHYBTransitionSuckEffect),
                 @(kHYBTransitionRippleEffect),
                 @(kHYBTransitionPageCurl),
                 @(kHYBTransitionPageUnCurl),
                 @(kHYBTransitionCameraOpen),
                 @(kHYBTransitionCameraClose),
                 @(kHYBTransitionCurlDown),
                 @(kHYBTransitionCurlUp),
                 @(kHYBTransitionFlipFromLeft),
                 @(kHYBTransitionFlipFromRight),
                 @(kHYBTransitionOglFlip)];
  
  self.img1 = [UIImage imageNamed:@"1.png"];
  self.img2 = [UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"3" ofType:@"jpg"]];
  
  self.view.backgroundColor = [UIColor colorWithPatternImage:self.img1];
  self.isImg1 = YES;
  
  [NSTimer scheduledTimerWithTimeInterval:1.0
                                   target:self
                                 selector:@selector(updateAnimation)
                                 userInfo:nil
                                  repeats:YES];
}

- (void)updateAnimation {
  
  if (self.index >= self.array.count) {
    self.index = 0;
  }
  
  HYBTransitionType type = [[self.array objectAtIndex:self.index++] intValue];
  static int s_subtypeValue = 0;
  HYBTransitionSubtype subtype = kHYBTransitionSubtypeFromTop;
  s_subtypeValue++;
  if (s_subtypeValue >= 4) {
    s_subtypeValue = 1;
  }
  
  subtype = (HYBTransitionSubtype)s_subtypeValue;
  
  [HYBTransitionAnimation transitionForView:self.view
                                       type:type
                                    subtype:subtype
                                   duration:1.0];
  
  if (self.isImg1) {
    self.view.backgroundColor = [UIColor colorWithPatternImage:self.img1];
  } else {
    self.view.backgroundColor = [UIColor colorWithPatternImage:self.img2];
  }
  
  self.isImg1 = !self.isImg1;
}

@end
```

#写在最后

---
学习不易，写代码不易，且看且珍惜！！！

学习不易，写代码不易，且看且珍惜！！！

学习不易，写代码不易，且看且珍惜！！！

#源代码

---
小伙伴们可以到笔者的`GITHUB`下载源代码：[CATransitionDemo请随手给一个star表示支持](https://github.com/CoderJackyHuang/CATransitionDemo)

#[阅读原文](http://www.henishuo.com/catransition-talk-in-detail/)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/)**

