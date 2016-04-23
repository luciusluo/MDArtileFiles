#前言

`iOS 7`以后提供了自定义转场动画的功能，我们可以通过遵守协议完成自定义转场动画。本篇文章讲解如何实现自定义`present`、`dismiss`自定义动画。

#效果图

本篇文章实现的动画切换效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/modal.gif)

#视图切换种类

如下效果图，这是有两大类视图切换动画的，一种是交互式的，另一种就是自定义的。

![image](http://www.henishuo.com/wp-content/uploads/2015/12/iOS-7视图切换.png)

本篇只讲其中的`UIViewControllerAnimatedTransitioning`协议，来实现`present`、`dismiss`动画效果。另外的几个，后面会继续学习总结!!!

#协议

我们要实现`present`、`dismiss`自定义转场效果，我们必须要有一个遵守了`UIViewControllerAnimatedTransitioning`协议且实现其必须实现的代理方法的类。

我们先来学习`UIViewControllerAnimatedTransitioning`协议：

```
@protocol UIViewControllerAnimatedTransitioning <NSObject>

// This is used for percent driven interactive transitions, as well as for container 
// controllers that have companion animations that might need to
// synchronize with the main animation.
// 
// 指定转场动画时长，必须实现，否则会Crash。
// 这个方法是为百分比驱动的交互转场和有对比动画效果的容器类控制器而定制的。
- (NSTimeInterval)transitionDuration:(nullable id <UIViewControllerContextTransitioning>)transitionContext;

// This method can only  be a nop if the transition is interactive 
// and not a percentDriven interactive transition.
// 若非百分比驱动的交互过渡效果，这个方法只能为空
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext;


@optional

// This is a convenience and if implemented will be invoked by the system 
// when the transition context's completeTransition: method is invoked.
- (void)animationEnded:(BOOL) transitionCompleted;

@end
```

我们要实现目标效果，就需要一个定义一个类遵守`UIViewControllerAnimatedTransitioning`协议并实现相应的代理方法。

#遵守UIViewControllerAnimatedTransitioning协议


下面，我们来定义一个转场类，这个类必须要遵守`UIViewControllerAnimatedTransitioning`协议，如下：

##头文件

```
//
//  HYBModalTransition.h
//  PresentDismissTransitionDemo
//
//  Created by huangyibiao on 15/12/21.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSUInteger, HYBModalTransitionType) {
  kHYBModalTransitionPresent = 1 << 1,
  kHYBModalTransitionDismiss = 1 << 2
};

@interface HYBModalTransition : NSObject <UIViewControllerAnimatedTransitioning>

/*!
 *  @author 黄仪标, 15-12-21 11:12:44
 *
 *  指定动画类型
 *
 *  @param type          动画类型
 *  @param duration      动画时长
 *  @param presentHeight 弹出呈现的高度
 *  @param scale         fromVC的绽放系数
 *
 *  @return 
 */
+ (HYBModalTransition *)transitionWithType:(HYBModalTransitionType)type
                                  duration:(NSTimeInterval)duration
                             presentHeight:(CGFloat)presentHeight
                                     scale:(CGPoint)scale;

@end
```
我们只公开了一个方法来创建，指定动画类型，动画时长，呈现的高度，缩放系数。

##实现文件

```
//
//  HYBModalTransition.m
//  PresentDismissTransitionDemo
//
//  Created by huangyibiao on 15/12/21.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import "HYBModalTransition.h"

@interface HYBModalTransition ()

@property (nonatomic, assign) HYBModalTransitionType type;
@property (nonatomic, assign) CGFloat presentHeight;
@property (nonatomic, assign) CGPoint scale;
@property (nonatomic, assign) NSTimeInterval duration;

@end

@implementation HYBModalTransition

+ (HYBModalTransition *)transitionWithType:(HYBModalTransitionType)type
                                  duration:(NSTimeInterval)duration
                             presentHeight:(CGFloat)presentHeight
                                     scale:(CGPoint)scale {
  HYBModalTransition *transition = [[HYBModalTransition alloc] init];
  
  transition.type = type;
  transition.presentHeight = presentHeight;
  transition.scale = scale;
  transition.duration = duration;
  
  return transition;
}

#pragma mark - UIViewControllerAnimatedTransitioning
- (void)animationEnded:(BOOL)transitionCompleted {
  NSLog(@"%s", __FUNCTION__);
}

- (NSTimeInterval)transitionDuration:(id<UIViewControllerContextTransitioning>)transitionContext {
  return self.duration;
}

- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext {
  switch (self.type) {
    case kHYBModalTransitionPresent: {
      [self present:transitionContext];
      break;
    }
    case kHYBModalTransitionDismiss: {
      [self dismiss:transitionContext];
      break;
    }
    default: {
      break;
    }
  }
}

#pragma mark - Private
- (void)present:(id<UIViewControllerContextTransitioning>)transitonContext {
  UIViewController *fromVC = [transitonContext viewControllerForKey:UITransitionContextFromViewControllerKey];
  UIViewController *toVC = [transitonContext viewControllerForKey:UITransitionContextToViewControllerKey];
  UIView *containerView = [transitonContext containerView];
  
  // 对fromVC.view的截图添加动画效果
  UIView *tempView = [fromVC.view snapshotViewAfterScreenUpdates:NO];
  tempView.frame = fromVC.view.frame;
  
  // 对截图添加动画，则fromVC可以隐藏
  fromVC.view.hidden = YES;
  
  // 要实现转场，必须加入到containerView中
  [containerView addSubview:tempView];
  [containerView addSubview:toVC.view];
  
  // 我们要设置外部所传参数
  // 设置呈现的高度
  toVC.view.frame = CGRectMake(0,
                               containerView.frame.size.height,
                               containerView.frame.size.width,
                               self.presentHeight);
  
  // 开始动画
  __weak __typeof(self) weakSelf = self;
  [UIView animateWithDuration:self.duration delay:0.0 usingSpringWithDamping:0.5 initialSpringVelocity:1.0 / 0.5 options:0 animations:^{
    // 在Y方向移动指定的高度
    toVC.view.transform = CGAffineTransformMakeTranslation(0, -weakSelf.presentHeight);
    
    // 让截图缩放
    tempView.transform = CGAffineTransformMakeScale(weakSelf.scale.x, weakSelf.scale.y);
  } completion:^(BOOL finished) {
    if (finished) {
      [transitonContext completeTransition:YES];
    }
  }];
}

- (void)dismiss:(id<UIViewControllerContextTransitioning>)transitonContext {
  UIViewController *fromVC = [transitonContext viewControllerForKey:UITransitionContextFromViewControllerKey];
  UIViewController *toVC = [transitonContext viewControllerForKey:UITransitionContextToViewControllerKey];
  UIView *containerView = [transitonContext containerView];
  
  // 取出present时的截图用于动画
  UIView *tempView = containerView.subviews.lastObject;
  
  // 开始动画
  [UIView animateWithDuration:self.duration animations:^{
    toVC.view.transform = CGAffineTransformIdentity;
    fromVC.view.transform = CGAffineTransformIdentity;
 
  } completion:^(BOOL finished) {
    if (finished) {
      [transitonContext completeTransition:YES];
      toVC.view.hidden = NO;
      
      // 将截图去掉
      [tempView removeFromSuperview];
    }
  }];
}

@end
```

我们这里就不细讲了，因为在[iOS 7 push/pop转场动画](http://www.henishuo.com/ios7-pushpop-transitioning/)中已经讲过了。大家若未看过，可以先阅读。

#测试效果


我们要设置一下被`present`的控制器的代理，在`-viewDidLoad:`时添加如下代码：

```
// 配置一下代理防呈现样式为自定义
self.transitioningDelegate = self;
self.modalPresentationStyle =  UIModalPresentationCustom;
```

同时，还需要遵守协议并实现协议`UIViewControllerTransitioningDelegate`，这个是控制器转场动画实现的代理：

```
#pragma mark - UIViewControllerTransitioningDelegate
- (id<UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source {
   return [HYBModalTransition transitionWithType:kHYBModalTransitionPresent duration:0.5 presentHeight:350 scale:CGPointMake(0.9, 0.9)];
}

- (id<UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed {
  return [HYBModalTransition transitionWithType:kHYBModalTransitionDismiss duration:0.25 presentHeight:350 scale:CGPointMake(0.9, 0.9)];
}
```

我们设置`present`、`dismiss`自定义对象，就可以实现我们的动画了。

想要实现什么样的动画，都可以在`HYBModalTransition`类里面实现，没有实现不了，只有想不到！！！

#写在最后

初步研究这方面的知识，最近要做技术分享，正好也好好研究研究，如果写得不好，希望大家不要笑话！！！

其它转场效果，后续会分享出来！！！

其它转场效果，后续会分享出来！！！

其它转场效果，后续会分享出来！！！

#源代码

小伙伴们可以到笔者的github下载：[ModalTransitionDemo](https://github.com/CoderJackyHuang/ModalTransitionDemo)

**请随手给一个star吧！！！**

#推荐阅读

* [iOS 7 push/pop转场动画一](http://www.henishuo.com/ios7-pushpop-transitioning/)

