>#CAAnimation解读

---
`CAAnimation`是一个抽象类，遵循了`CAMediaTiming`协议和`CAAction`协议！我们不要直接使用`CAAnimation`类，而是使用其子类：

* `CATransition`：提供渐变效果，如推拉`push`效果,消退`fade`效果,揭开`reveal`效果
* `CAAnimationGroup`：允许多个动画同时播放
* `CABasicAnimation`： 提供了对单一动画的实现
* `CAKeyframeAnimation`： 关键桢动画,可以定义动画路线
* `CAPropertyAnimation`：属性动画，通常不直接使用，而是使用`CABasicAnimation`子类

#创建对象

---
我们看到有一个工厂方法来创建`CAAnimation`对象，因此，我们通常都使用这个方法来创建动画：

```
+ (instancetype)animation;
```

当然不同类型的子类使用的方法不一样，对于继承于`CAPropertyAnimation`的子类，都可以通过属性路径来创建：

```
/* Creates a new animation object with its `keyPath' property set to
 * 'path'. */

+ (instancetype)animationWithKeyPath:(nullable NSString *)path;
```

#遵守了CAMediaTiming协议

---
这个协议是是用于配置动画的相关属性的，英文部分是官方的注释，中文部分为笔者的理解，下面一一讲解：

```
/* The begin time of the object, in relation to its parent object, if
 * applicable. Defaults to 0. */
// 获取或设置动画的开始时间，默认为0
@property CFTimeInterval beginTime;

/* The basic duration of the object. Defaults to 0. */
// 获取或设置动画的时长，也就是整个动画的总时长
@property CFTimeInterval duration;

/* The rate of the layer. Used to scale parent time to local time, e.g.
 * if rate is 2, local time progresses twice as fast as parent time.
 * Defaults to 1. */
// 获取或设置动画的播放速度，默认为1，若设置为2，则以两倍的速度播放。
// 如果设置小于1，则相当于慢放。
@property float speed;

/* Additional offset in active local time. i.e. to convert from parent
 * time tp to active local time t: t = (tp - begin) * speed + offset.
 * One use of this is to "pause" a layer by setting `speed' to zero and
 * `offset' to a suitable value. Defaults to 0. */
// 获取或设置当前播放的进度，默认为0。有这么一种使用场景：设置speed为0，
// 然后设置这个timeOffset为合适的值，就可以暂停动画了
@property CFTimeInterval timeOffset;

/* The repeat count of the object. May be fractional. Defaults to 0. */
// 获取或设置动画播放次数，默认为0表示只播放一次。
// 设置为HUGE_VALF表示无限制播放次数
@property float repeatCount;

/* The repeat duration of the object. Defaults to 0. */
// 获取或设置重复播放的动画时长，不要与repeatCount混合使用
@property CFTimeInterval repeatDuration;

/* When true, the object plays backwards after playing forwards. Defaults
 * to NO. */
// 获取或设置是否回放。
// 如果设置为YES，在动画播放完成时，就会以动画的效果回到起点
// 如果设置为NO，播放完成时，就会停留在终点
@property BOOL autoreverses;

/* Defines how the timed object behaves outside its active duration.
 * Local time may be clamped to either end of the active duration, or
 * the element may be removed from the presentation. The legal values
 * are `backwards', `forwards', `both' and `removed'. Defaults to
 * `removed'. */
// 获取或设置动画完成时的动作
// forwards表示动画完成时，也回到起点而不是留在终点
// backwards表示动画完成时，就停留在终点
// removed表示完成时就移除，默认就是removed
@property(copy) NSString *fillMode;
```

#遵守了CAAction协议

---
这个协议只有一个方法，我们可以调用此方法来触发指定的事件，这样接收者就可以接收到代理。

```
/* Called to trigger the event named 'path' on the receiver. The object
 * (e.g. the layer) on which the event happened is 'anObject'. The
 * arguments dictionary may be nil, if non-nil it carries parameters
 * associated with the event. */

- (void)runActionForKey:(NSString *)event 
                 object:(id)anObject
              arguments:(nullable NSDictionary *)dict;
```

#CAAnimationDelegate代理

---
`CAAnimation`为这么个属性：

```
/* The delegate of the animation. This object is retained for the
 * lifetime of the animation object. Defaults to nil. See below for the
 * supported delegate methods. */

@property(nullable, strong) id delegate;
```

我们只要指定了代理，就可以实现这两个代理方法：

```
/* Called when the animation begins its active duration. */
// 动画开始时的回调
- (void)animationDidStart:(CAAnimation *)anim;

/* Called when the animation either completes its active duration or
 * is removed from the object it is attached to (i.e. the layer). 'flag'
 * is true if the animation reached the end of its active duration
 * without being removed. */
// 动画停止的回调，可以通过flag判断动画是否是完成还是暂停
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag;
```

#removedOnCompletion属性

---

当我们动画完成时，如果希望动画就自动移除的话，我们可以设置此属性为`YES`，默认值为`YES`。如果我们想要循环或者执行多次动画，就将此属性设置为`NO`

```
/* When true, the animation is removed from the render tree once its
 * active duration has passed. Defaults to YES. */

// 当duration值已经达到时，是否将动画自动从渲染树上移除
@property(getter=isRemovedOnCompletion) BOOL removedOnCompletion;
```

#timingFunction属性

---
这个属性是用于指定动画移动的步调是什么样式，比如线性。

```
/* A timing function defining the pacing of the animation. Defaults to
 * nil indicating linear pacing. */

@property(nullable, strong) CAMediaTimingFunction *timingFunction;
```

关于`CAMediaTimingFunction`类，主要是这向个方法。当创建时，我们`+functionWithName:`工厂方法来创建系统已经提供的样式。

```
/* A convenience method for creating common timing functions. The
 * currently supported names are `linear', `easeIn', `easeOut' and
 * `easeInEaseOut' and `default' (the curve used by implicit animations
 * created by Core Animation). */

+ (instancetype)functionWithName:(NSString *)name;
```

其中这个`name`有这几个变量对应的：

```
// 线性动画
CA_EXTERN NSString * const kCAMediaTimingFunctionLinear
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

// 快速进入动画
CA_EXTERN NSString * const kCAMediaTimingFunctionEaseIn
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

// 快速出来动画
CA_EXTERN NSString * const kCAMediaTimingFunctionEaseOut
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

// 快速进入出来动画
CA_EXTERN NSString * const kCAMediaTimingFunctionEaseInEaseOut
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);
    
// 默认动画是curve动画，也就是曲线动画
CA_EXTERN NSString * const kCAMediaTimingFunctionDefault
	__OSX_AVAILABLE_STARTING (__MAC_10_6, __IPHONE_3_0);
```

如果我们想要让其移动动画是按贝塞尔曲线的路径行动，那么可以用这两个方法来创建：

```
/* Creates a timing function modelled on a cubic Bezier curve. The end
 * points of the curve are at (0,0) and (1,1), the two points 'c1' and
 * 'c2' defined by the class instance are the control points. Thus the
 * points defining the Bezier curve are: '[(0,0), c1, c2, (1,1)]' */

+ (instancetype)functionWithControlPoints:(float)c1x :(float)c1y :(float)c2x :(float)c2y;

- (instancetype)initWithControlPoints:(float)c1x :(float)c1y :(float)c2x :(float)c2y;
```

#[阅读原文](http://www.henishuo.com/caanimation-indtroduce-in-detail/)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/)**



