#前言


在开发中经常需要给已有的类添加方法和属性，但是`Objective-C`是不允许给已有类通过分类添加属性的，因为类分类是不会自动生成成员变量的。但是，我们可以通过运行时机制就可以做到了。

本篇文章适合新手阅读，手把手教你如何在项目中使用关联属性！

#API介绍

我们先看看Runtime提供的关联API，只有这三个API，使用也是非常简单的：

```
/** 
 * Sets an associated value for a given object using a given key and association policy.
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * @param value The value to associate with the key key for object. Pass nil to clear an existing association.
 * @param policy The policy for the association. For possible values, see “Associative Object Behaviors.”
 * 
 * @see objc_setAssociatedObject
 * @see objc_removeAssociatedObjects
 */
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)

/** 
 * Returns the value associated with a given object for a given key.
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * 
 * @return The value associated with the key \e key for \e object.
 * 
 * @see objc_setAssociatedObject
 */
id objc_getAssociatedObject(id object, const void *key)

/** 
 * Removes all associations for a given object.
 * 
 * @param object An object that maintains associated objects.
 * 
 * @note The main purpose of this function is to make it easy to return an object 
 *  to a "pristine state”. You should not use this function for general removal of
 *  associations from objects, since it also removes associations that other clients
 *  may have added to the object. Typically you should use \c objc_setAssociatedObject 
 *  with a nil value to clear an association.
 * 
 * @see objc_setAssociatedObject
 * @see objc_getAssociatedObject
 */
void objc_removeAssociatedObjects(id object)
```

实际上，我们几乎不会使用到`objc_removeAssociatedObjects`函数，这个函数的功能是移除指定的对象上所有的关联。既然我们要添加关联属性，几乎不会存在需要手动取消关联的场合。

#设置关联值（Setter）


对于设置关联，我们需要使用下面的API关联起来：

```
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
```

参数说明：

* `object`：与谁关联，通常是传`self`
* `key`：唯一键，在获取值时通过该键获取，通常是使用`static const void *`来声明
* `value`：关联所设置的值
* `policy`：内存管理策略，比如使用`copy`

#获取关联值（Getter）

如果我们要获取所关联的值，需要通过`key`来获取，调用如下函数：

```
id objc_getAssociatedObject(id object, const void *key)
```

参数说明：

* `object`：与谁关联，通常是传`self`，在设置关联时所指定的与哪个对象关联的那个对象
* `key`：唯一键，在设置关联时所指定的键

#关联策略

我们先看看设置关联时所指定的`policy`，它是一个枚举类型，看官方说明：

```
/**
 * Policies related to associative references.
 * These are options to objc_setAssociatedObject()
 */
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
```
我们说明一下各个值的作用：

* OBJC\_ASSOCIATION\_ASSIGN：表示弱引用关联，通常是基本数据类型，如`int`、`float`，非线程安全
* OBJC\_ASSOCIATION\_RETAIN\_NONATOMIC：表示强（strong）引用关联对象，非线程安全
* OBJC\_ASSOCIATION\_COPY\_NONATOMIC：表示关联对象copy，非线程安全
* OBJC\_ASSOCIATION\_RETAIN：表示强（strong）引用关联对象，是线程安全的
* OBJC\_ASSOCIATION\_COPY：表示关联对象copy，是线程安全的

#扩展属性

我们来写一个例子，扩展`UIControl`添加Block版本的`TouchUpInside`事件。

扩展头文件声明：

```
#import <UIKit/UIKit.h>

typedef void (^HYBTouchUpBlock)(id sender);

@interface UIControl (HYBBlock)

@property (nonatomic, copy) HYBTouchUpBlock hyb_touchUpBlock;

@end
```

扩展实现文件：

```
#import "UIControl+HYBBlock.h"
#import <objc/runtime.h>

static const void *sHYBUIControlTouchUpEventBlockKey = "sHYBUIControlTouchUpEventBlockKey";

@implementation UIControl (HYBBlock)

- (void)setHyb_touchUpBlock:(HYBTouchUpBlock)hyb_touchUpBlock {
  objc_setAssociatedObject(self,
                           sHYBUIControlTouchUpEventBlockKey,
                           hyb_touchUpBlock,
                           OBJC_ASSOCIATION_COPY);
  
  [self removeTarget:self
              action:@selector(hybOnTouchUp:)
    forControlEvents:UIControlEventTouchUpInside];
  
  if (hyb_touchUpBlock) {
    [self addTarget:self
             action:@selector(hybOnTouchUp:)
   forControlEvents:UIControlEventTouchUpInside];
  }
}

- (HYBTouchUpBlock)hyb_touchUpBlock {
  return objc_getAssociatedObject(self, sHYBUIControlTouchUpEventBlockKey);
}

- (void)hybOnTouchUp:(UIButton *)sender {
  HYBTouchUpBlock touchUp = self.hyb_touchUpBlock;
  
  if (touchUp) {
    touchUp(sender);
  }
}

@end
```

使用起来很简单吧！！！

#小结

本文章是专门介绍通过runtime如何给已有类添加扩展属性，如果文章中出现有疑问的地方，请在评论中评论，笔者会在第一时间回复您的！

本篇文章中所提到的只是最常见的添加关联属性的方式之一，对于生成只读的关联属性也是很常用的，自行实现一下吧！

