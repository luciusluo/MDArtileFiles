#前言


`iOS 7`之后，苹果提供了自定义转场动画的`API`，我们可以自己去定义任意动画效果。本篇为笔者学习`push`、`pop`自定义转场效果的笔记，如何有任何不正确或者有指导意见的，请在评论中留下您的宝贵意见！！！

**请注意：**如果要求支持`iOS 7`以下版本，则不可使用此效果。

#实现目标效果

我们本篇文章目标效果：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/transition-2.gif)


#视图切换种类

如下效果图，这是有两大类视图切换动画的，一种是交互式的，另一种就是自定义的。

![image](http://www.henishuo.com/wp-content/uploads/2015/12/iOS-7视图切换.png)

本篇只讲其中的`UIViewControllerAnimatedTransitioning`协议，来实现`push`、`pop`动画效果。另外的几个，后面会继续学习总结!!!

#协议

我们要实现`push`、`pop`自定义转场效果，我们必须要有一个遵守了`UIViewControllerAnimatedTransitioning`协议且实现其必须实现的代理方法的类。

我们先来学习`UIViewControllerAnimatedTransitioning`协议：

```
@protocol UIViewControllerAnimatedTransitioning <NSObject>

// This is used for percent driven interactive transitions, as well as for container controllers that have companion animations that might need to
// synchronize with the main animation.
// 
// 指定转场动画时长，必须实现，否则会Crash。
// 这个方法是为百分比驱动的交互转场和有对比动画效果的容器类控制器而定制的。
- (NSTimeInterval)transitionDuration:(nullable id <UIViewControllerContextTransitioning>)transitionContext;

// This method can only  be a nop if the transition is interactive and not a percentDriven interactive transition.
// 若非百分比驱动的交互过渡效果，这个方法只能为空
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext;


@optional

// This is a convenience and if implemented will be invoked by the system when the transition context's completeTransition: method is invoked.
- (void)animationEnded:(BOOL) transitionCompleted;

@end
```

我们要实现目标效果，就需要一个定义一个类遵守`UIViewControllerAnimatedTransitioning`协议并实现相应的代理方法。

#遵守UIViewControllerAnimatedTransitioning协议

下面，我们来定义一个转场类，这个类必须要遵守`UIViewControllerAnimatedTransitioning`协议，如下：

```
//
//  HYBControllerTransition.h
//  PushPopMoveTransitionDemo
//
//  Created by huangyibiao on 15/12/18.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSUInteger, HYBControllerTransitionType) {
  kControllerTransitionPush = 1 << 1,
  kControllerTransitionPop = 1 << 2
};

@interface HYBControllerTransition : NSObject <UIViewControllerAnimatedTransitioning>

+ (instancetype)transitionWithType:(HYBControllerTransitionType)transitionType
                          duration:(NSTimeInterval)duration;

@end
```

我们只需要公开一个工厂方法来生成对象即可，调用更简单些。第个参数用于指定是哪种类型，是`push`还是`pop`，第二个参数是用于指定动画时长。

#实现文件

我们一步步分析下面的关键代码：

```
//
//  HYBControllerTransition.m
//  PushPopMoveTransitionDemo
//
//  Created by huangyibiao on 15/12/18.
//  Copyright © 2015年 huangyibiao. All rights reserved.
//

#import "HYBControllerTransition.h"
#import "ViewController.h"
#import "DetailController.h"

@interface HYBControllerTransition ()

@property (nonatomic, assign) HYBControllerTransitionType transitionType;
@property (nonatomic, assign) NSTimeInterval duration;

@end

@implementation HYBControllerTransition

- (instancetype)init {
  if (self = [super init]) {
    self.transitionType = kControllerTransitionPush;
  }
  
  return self;
}

+ (instancetype)transitionWithType:(HYBControllerTransitionType)transitionType
                          duration:(NSTimeInterval)duration {
  HYBControllerTransition *transition = [[HYBControllerTransition alloc] init];
  transition.transitionType = transitionType;
  transition.duration = duration;
  
  return transition;
}

#pragma mark - UIViewControllerAnimatedTransitioning
- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext {
  switch (self.transitionType) {
    case kControllerTransitionPush: {
      [self push:transitionContext];
      break;
    }
    case kControllerTransitionPop: {
      [self pop:transitionContext];
      break;
    }
    default: {
      break;
    }
  }
}

- (NSTimeInterval)transitionDuration:(id<UIViewControllerContextTransitioning>)transitionContext {
  return self.duration;
}

- (void)animationEnded:(BOOL)transitionCompleted {
  NSLog(@"%s", __FUNCTION__);
}

#pragma mark - Private
- (void)pop:(id<UIViewControllerContextTransitioning>)transitionContext {
  DetailController *fromVC = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
  ViewController *toVC = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
  UIView *containerView = [transitionContext containerView];
 
  UIView *toImageView = toVC.isImg1 ? toVC.img1 : toVC.img2;
  
  UIView *tempView = containerView.subviews.lastObject;
  
  // 第一个view是fromVC.view
  // 第二个view是push进来时所生成的toImageView截图
  for (UIView *view in containerView.subviews) {
    NSLog(@"%@", view);
    if (fromVC.view == view) {
      NSLog(@"YES");
    }
  }
  
  toImageView.hidden = YES;
  tempView.hidden = NO;
  // 必须保证将toVC.view放在最上面，也就是第一个位置
  [containerView insertSubview:toVC.view atIndex:0];

  [UIView animateWithDuration:self.duration
                        delay:0.0
       usingSpringWithDamping:0.55
        initialSpringVelocity:1/ 0.55
                      options:0
                   animations:^{
                     fromVC.view.alpha = 0.0;
                     tempView.frame = [toImageView convertRect:toImageView.bounds toView:containerView];
  } completion:^(BOOL finished) {
    tempView.hidden = NO;
    toImageView.hidden = NO;
    [tempView removeFromSuperview];
    
    [transitionContext completeTransition:YES];
    
    for (UIView *view in containerView.subviews) {
      NSLog(@"%@", view);
      if (toVC.view == view) {
        NSLog(@"YES");
      }
    }
  }];
}

- (void)push:(id<UIViewControllerContextTransitioning>)transitionContext {
  ViewController *fromVC = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
  DetailController *toVC = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
  UIView *containerView = [transitionContext containerView];
  
  UIView *fromImageView = fromVC.isImg1 ? fromVC.img1 : fromVC.img2;
  UIView *tempView = [fromImageView snapshotViewAfterScreenUpdates:NO];
  tempView.frame = [fromImageView convertRect:fromImageView.bounds toView:containerView];
 
  UIView *toImageView = toVC.imgView;
  
  fromImageView.hidden = YES;
  toVC.view.alpha = 0.0;
  toImageView.hidden = YES;
  
  [containerView addSubview:toVC.view];
  [containerView addSubview:tempView];
  
  [UIView animateWithDuration:self.duration
                        delay:0.0
       usingSpringWithDamping:0.55
        initialSpringVelocity:1/ 0.55
                      options:0
                   animations:^{
                     toVC.view.alpha = 1.0;
                     tempView.frame = [toImageView convertRect:toImageView.bounds toView:containerView];
                   } completion:^(BOOL finished) {
                     tempView.hidden = YES;
                     toImageView.hidden = NO;

                     [transitionContext completeTransition:YES];
                   }];
}
@end
```

###分析Push动画

我们暂不细说`UIViewControllerContextTransitioning`协议，我们这里只使用到了`-containerView`这个代理方法，我们可以通过苹果提供的键来获取对应的控制器：

```
UIKIT_EXTERN NSString *const UITransitionContextFromViewControllerKey NS_AVAILABLE_IOS(7_0);
UIKIT_EXTERN NSString *const UITransitionContextToViewControllerKey NS_AVAILABLE_IOS(7_0);
```

我们可以看到这是在`iOS 7.0`以后才有的，因此系统版本要求是在`iOS 7`才能使用。

我们这里通过:

```
ViewController *fromVC = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
```
获取到了`fromVC`，也就是当前要从哪个控制器切换。

然后通过：

```
DetailController *toVC = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
```

获取到了`toVC`，也就是切换到哪一个控制器。

然后再通过

```
UIView *containerView = [transitionContext containerView];
```
获取`containerView`视图。这个是一个代理方法，可以获取到视图容器。


下面我们获取`fromVC`所点击的图片控件，然后通过`-snapshotViewAfterScreenUpdates:`将所点击的图片控件截图并用于切换使用，参数设置为`NO`，否则动画会很生硬。最后，我们还要将这个所点击的图片控件的坐标转换成容器视图的坐标：

```
UIView *fromImageView = fromVC.isImg1 ? fromVC.img1 : fromVC.img2;
UIView *tempView = [fromImageView snapshotViewAfterScreenUpdates:NO];
tempView.frame = [fromImageView convertRect:fromImageView.bounds toView:containerView];
```

将下来就是设置切换动画之前的状态：

```
UIView *toImageView = toVC.imgView;
  
fromImageView.hidden = YES;
toVC.view.alpha = 0.0;
toImageView.hidden = YES;
```

下面这两行是非常关键的，并且必须保证`tempView`在最上层，否则动画效果就没有了。先将目标控制器的视图添加到容器中，再添加源图片的截图到容器中，用于显示切换效果。

```
[containerView addSubview:toVC.view];
[containerView addSubview:tempView];
```

我们在动画中，将初始的截图的`frame`改变成最终的效果的`frame`即可达到我们的目标效果。另外要注意还需要将坐标转换成容器的坐标：

```
tempView.frame = [toImageView convertRect:toImageView.bounds toView:containerView];
```

当动画完成以后，一定要调用：`[transitionContext completeTransition:YES]`，设置切换动画已经完成，否则想要`pop`回去就不能了。

###分析pop动画

我们只讲不同于`push`部分的代码，我们添加了打印容器中的视图的代码：

```
// 第一个view是fromVC.view
// 第二个view是push进来时所生成的toImageView截图
for (UIView *view in containerView.subviews) {
	NSLog(@"%@", view);
	if (fromVC.view == view) {
	  NSLog(@"YES");
	}
}
```

打印结果：

```
<UIView: 0x7fae00514ef0; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x7fae00513320>>
YES
<_UIReplicantView: 0x7fae00566bd0; frame = (20 20; 335 627); hidden = YES; layer = <_UIReplicantLayer: 0x7fae00510520>>

<UIView: 0x7fae004563a0; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x7fae00422b20>>
YES
```

从打印结果分析出来，在`pop`之前，第一个就是`fromVC.view`，第二个就是上次`push`时的截图。在动画完成之后，只剩下`toVC.view`了。

我们还需要将`toVC.view`放在容器最上层：

```
// 必须保证将toVC.view放在最上面，也就是第一个位置
[containerView insertSubview:toVC.view atIndex:0];
```

**动画完成后，一定要将`tempView`移除：**

```
[tempView removeFromSuperview];
```

如果不移除，这个`tempView`会与所点击的图片控件等大小，位置一样，会挡住原图片控件，手势也就无法响应。我们一定要注意移除。动画完成后，也一定要设置`[transitionContext completeTransition:YES]`。


#写在最后


本篇文章的转场效果已经实现了，大家可以一起来学习哦！如果写有更多更好看的转场效果，请一定要分享出来哦！

#源代码


小伙伴们可以到笔者的`github`下载：[PushPopTransitionDemo](https://github.com/CoderJackyHuang/PushPopTransitionDemo)

**请随手给一个star吧！！！**

#推荐阅读

* [present/dismiss转场动画](http://www.henishuo.com/ios-7-presentdismiss-transitioning/)

