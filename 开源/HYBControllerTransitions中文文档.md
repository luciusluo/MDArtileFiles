#中文文档

**当前版本：**1.2.0  
**项目目标：**一键式集成常用的转场动画，一个API轻松实现转场功能，无须了解转场知识！


HYBControllerTransitions是自定义围场动画API封装类库，使用简便。使用者不需要了解太多转场动画知识，即可轻松接入项目使用。

这是一个给开发者们提供自定义push、pop、dismiss和present转场动画的开源组件。现在库中支持泡泡放大缩小围场、模态半屏转场和移动切换转场（KeyNote某转场效果）。对于开发者们来说，这是一个很不错的开源库。开发者不需要了解转场知识，只需要简单一个API就可以实现对应的功能。

如果想要更深入地学习，请在源码中查看对应的注释说明！

![image](http://www.henishuo.com/wp-content/uploads/2016/03/logo.png)


#开源目标

在设计此类库时，希望以最简洁的API来实现转场功能。如果不懂任何转场知识的人，也能轻松使用，那才算达到目标。

因此，如果您没有学习过相关转场方面的知识，请不要担心，这个类库不需要您掌握太多转场的知识，只需要懂基本的OC语法即可轻松接入项目使用。

公共的属性及API都封装在HYBBaseTransition类型中，它遵守了需要实现转场的所有协议并实现之。如果目前类库中所提供的效果不满足您的需求，您可以直接继承于HYBBaseTransition于实现您希望得到的效果。

#版本变化

###Version 1.2.0

* 修正iOS7.0下显示问题
* 增加淡入淡出围场动画效果，类名为HYBEaseInOutTransition，Demo使用对应于Demo5

###Version 1.1.0

* 解决push/pop模式下，在push后，进入后台再进入前台时，会出现异常。原因是导航的代理交给了转场对象后，一直持有，因此才出现此问题。现在在poped的时候，会自动清除掉导航代理。如果在使用转场之前，已经有其它对象持有导航代理，应该在poped之后，交代理交还给原来的代理对象！

###Version 1.0.1

* 完善文档及去掉部分demo无关代码


#支持弹簧动画效果

默认转场动画使用的不是弹簧动画效果，如果觉得弹簧动画效果更佳，仅需要设置为YES即可。

```
/**
 *  Default is NO, if set to YES, it will be presented and dismissed with
 *  spring animation.
 */
@property (nonatomic, assign) BOOL animatedWithSpring;
```

当然，使用到弹簧自然需要设置其参数。不过作者都提供有默认值，都是经过调试过得到的值。如果觉得默认值所得到的效果不够好，请自行调整参数：

```
/**
 * The initial Spring velocity, Only when animatedWithSpring is YES, it will take effect.
 * Default is 1.0 / 0.5. If you don't know, just use the default value.
 */
@property (nonatomic, assign) CGFloat initialSpringVelocity;

/**
 *  The Spring damp, Only when animatedWithSpring is YES, it will take effect.
 *
 *  Default is 0.5. If you don't know, just use the default value.
 */
@property (nonatomic, assign) CGFloat damp;
```

#转场类型动画

目前所支持的动画效果有以下：

* Bubble Effect Transition：泡泡放大、缩小的动画效果，仅支持模态呈现present/dismiss。
* Modal Effect Transition：半屏呈现转场，支持设置缩放系数、呈现高度等，仅支持模态呈现present/dismiss。
* Move Push/Pop Transition：移动切换转场效果，仅支持push/pop模式。

##Buble Effect Transition

present时，以泡泡圆形放大；dismiss时，以泡泡圆形缩小至起点。当需要实现此转场效果时，请使用HYBBubbleTransition类型。效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/bubble.gif)

###如何使用

使用起来非常简单，只需要一个API且只在一处调用即可。比如，现在有HYBModalBubbleViewController，它有一个点击事件，在点击后会回调onPresent函数，然后配置如下即可实现转场：

```
- (void)onPresent {
  HYBBubbleFromBottomController *vc = [[HYBBubbleFromBottomController alloc] init];
  vc.modalPresentationStyle = UIModalPresentationCustom;
  
  // Remember to own it strongly
  // Because delegate is weak reference, and it will be released after out of the function body.
  self.bubbleTransition = [[HYBBubbleTransition alloc] initWithPresented:^(UIViewController *presented, UIViewController *presenting, UIViewController *source, HYBBaseTransition *transition) {
    // You need to cast type to the real subclass type.
    HYBBubbleTransition *bubble = (HYBBubbleTransition *)transition;
   
    // If you want to use Spring animation, set to YES.
    // Default is NO.
//    bubble.animatedWithSpring = YES;
    bubble.bubbleColor = presented.view.backgroundColor;
    
    // 由于一个控制器有导航，一个没有，导致会有64的误差，所以要记得处理这种情况
    CGPoint center = [self.view viewWithTag:1010].center;
    center.y += 64;
    
    bubble.bubbleStartPoint = center;
  } dismissed:^(UIViewController *dismissed, HYBBaseTransition *transition) {
    // Do nothing and it is ok here.
    // If you really want to do something, here you can set the mode.
    // But inside the super class, it is set to be automally.
    // So you do this has no meaning.
    transition.transitionMode = kHYBTransitionDismiss;
  }];
  vc.transitioningDelegate = self.bubbleTransition;
  
  [self presentViewController:vc animated:YES completion:NULL];
}
```

这里会present HYBBubbleFromBottomController这个控制器类，但是HYBBubbleFromBottomController对象什么也不需要做，就可以直接实现了转场。

是不是真的很简单呢？

如果想要了解更多功能功能，请在源代码中查看类中所提供的所有公开属性，几乎都有默认值，若不需要修改，直接使用默认值就可以了。

**注意事项：**一定要将代理设置为对应的转场类对象。而且一定要在当前控制器强引用该转场类对象，因为设置为转场代理，只是弱使用，如果没有强引用，它就会被释放掉：

```
// 代理不再是设置为self，而是设置为转场对象
vc.transitioningDelegate = self.bubbleTransition;
```

##Modal Effect Transition

当需要实现半屏呈现且带缩放效果的转场动画时，可以使用此HYBModalTransition类。它仅支持present/dismiss模式。它提供了更多的属性设置，稍候讲解。效果如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/modal.gif)


###如何使用

使用非常简单。假设在HYBModalHalfController有一个点击事件，点击后会回调onPresent函数，只需要在此函数中实现即可：

```
- (void)onPresent {
  HYBModalHalfDetailController *vc = [[HYBModalHalfDetailController alloc] init];
  
  self.transition = [[HYBModalTransition alloc] initWithPresented:^(UIViewController *presented, UIViewController *presenting, UIViewController *source, HYBBaseTransition *transition) {
    HYBModalTransition *modal = (HYBModalTransition *)transition;
    modal.scale = (CGPoint){0.95, 0.95};
    
    // If you don't specify, it will use default value
//    modal.presentedHeight = 350.0;
    
    // If you don't want to, set to YES or do no set.
    modal.shouldDismissOnTap = YES;
    
    // Default is NO, if set to YES, it will use spring animation.
    modal.animatedWithSpring = YES;
    
    // Default is YES. including navigation bar when take snapshots.
    // When has navigation bar, if set to NO, it looks not so good.
//    modal.scapshotIncludingNavigationBar = NO;
  } dismissed:^(UIViewController *dismissed, HYBBaseTransition *transition) {
    // do nothing
    // 注释掉也没有关系，内部已经自动设置了。
    transition.transitionMode = kHYBTransitionDismiss;
  }];
  
  vc.transitioningDelegate = self.transition;
  [self presentViewController:vc animated:YES completion:NULL];
}
```

对于HYBModalHalfDetailController控制器类，什么也不需要操作。这是不是有点太过于简单了？对于dismissed这个block，其实这里完全可以设置为nil，因为不需要做任何操作，默认就自动设置了mode。

###支持带导航截图

如果当前控制器类有导航，最好还是连导航条也一起生成截图，这样效果会好很多。默认就是YES。如果不希望如此，请手动设置为N：

```
/**
 *  Whether to include navigation bar when take snapshots.
 *	Default is YES. If NO, it has only the presenting view.
 */
@property (nonatomic, assign) BOOL scapshotIncludingNavigationBar;
```

###支持手势点击自动Dismiss

在弹出来之后，默认是添加了点击手势，可以自动dismiss。默认为YES,如果不希望如此，请手动设置为NO：

```
/**
 *	When tap on the presenting view, should it automatically is dismissed.
 *
 *  Default is YES.
 */
@property (nonatomic, assign) BOOL shouldDismissOnTap;
```

###支持设置缩放、高度

都提供了默认值，但是如果想要调整到一个让您满意的效果，也许这些属性就可以帮助您实现：

```
/**
 *	Make the from view scale to the specified scale.
 *
 *  Default is (0.9, 0.9)
 */
@property (nonatomic, assign) CGPoint scale;

/**
 *	The height for destination view to present.
 *  
 *  Default is half of destination view, it means desView.frame.size.height / 2
 */
@property (nonatomic, assign) CGFloat presentedHeight;
```

##Move Push/Pop Transition

类似于KeyNote的神奇移动效果push、pop动画。效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/move.gif)

使用起来也是很简单的，不同于present/dismiss模式，由于push的时候还不能得到转场过去的目标视图，比如效果图中的切换过去后看到的图片控件。在创建控制器类的时候，控件是不存在的，因此只能使用其它的办法来实现。

这里所采用的方案是通过给UIViewController添加扩展属性，如此就可以在创建UI的地方，将目标控件赋值。扩展所添加的属性为：

```
/**
 * Set a target view to show. When push, it will transition to
 * the target view. and when poped, it will pop from the target view.
 */
@property (nonatomic, strong, nonnull) UIView *hyb_toTargetView;
```

###如何使用

假设当前控制器类为HYBMoveViewController，它有一个collectionview，呈风格布局显示一个图片列表。当点击cell的时候，就切换（push）到HYBMoveDetailController控制器中显示更多内容。

实现如下：

```
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath {
  HYBMoveDetailController *vc = [[HYBMoveDetailController alloc] init];
  HYBGridModel *model = self.datasource[indexPath.item];
  vc.image = model.clipedImage;
  
  self.transition = [[HYBMoveTransition alloc] initWithPushed:^(UIViewController *fromVC, UIViewController *toVC, HYBBaseTransition *transition) {
    HYBMoveTransition *move = (HYBMoveTransition *)transition;
    HYBGridCell *cell = (HYBGridCell *)[collectionView cellForItemAtIndexPath:indexPath];
    move.targetClickedView = cell.imageView;
    
    move.animatedWithSpring = YES;
  } poped:^(UIViewController *fromVC, UIViewController *toVC, HYBBaseTransition *transition) {
    // Do nothing, unless you really need to.
  }];
  
  self.navigationController.delegate = self.transition;
  [self.navigationController pushViewController:vc animated:YES];
}
```

在点击的时候，需要创建一个转场对象，然后我们将导航类的代理设置为这个转场对象，如下：

```
self.navigationController.delegate = self.transition;
```

这样就可以实现内部自动处理了。当push的时候，会回调pushed闭包，在这里返回了转场对象，需要提供相关属性设置，才能实现。一定要传targetClickedView属性，这是被点击的控件，也是用于切换效果用的。

**注意：**如果导航的代理原来是其它类对象持有，在poded闭包中应该要设置成原来的导航代理。

在HYBMoveDetailController控制器中，在viewDidLoad这里创建UI的地方，创建了一个图片控件，然后如些设置：

```
// You must specify a target view with this.
self.hyb_toTargetView = imgView;
```

OK，到此就实现好功能了！

是否足够简单？

#如何安装

支持pod，可直接使用pod添加以下代码到Podfile中：

```
pod 'HYBControllerTransitions', '~> 1.0.0'
```

如果您的工程不支持Pod，呆直接将HYBControllerTransitions目录放到您的工程中即可！

#源代码

如果不想使用pod来安装，或者想要看看demo，请到此处下载：[HYBControllerTransitions](https://github.com/CoderJackyHuang/HYBControllerTransitions)

#致谢

非常感谢[andreamazz](https://github.com/andreamazz)，学习了很多他的开源作品！

#MIT LICENSE

Copyright (c) 2016 CoderJackyHuang. All rights reserved.

Permission is hereby granted, free of charge, to any person obtaining a  
copy of this software and associated documentation files (the "Software"),  
to deal in the Software without restriction, including  
without limitation the rights to use, copy, modify, merge, publish,  
distribute, sublicense, and/or sell copies of the Software, and to  
permit persons to whom the Software is furnished to do so, subject to  
the following conditions:  

The above copyright notice and this permission notice shall be included  
in all copies or substantial portions of the Software.  

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS  
OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF  
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY  
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,  
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE  
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.  

