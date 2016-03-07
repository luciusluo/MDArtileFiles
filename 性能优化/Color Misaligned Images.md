#序言

今天我们来学习学习Instruments之Core Animation之**Color Misaligned Images**。

官方说明是这样的：**Color Misaligned Images：** Places a magenta overlay over images where the source pixels are not aligned to the destination pixels.

意思就是当图片的源像素与目标像素不对齐是会放一个洋红色的层在图片上。当图片的像素大小与控件的大小不一致而导致需要缩放时，图片会呈现黄色。

本篇文章是记录笔者学习优化图片显示的笔记，同时希望能够帮助大家进一步学习，当然更希望大家多思考多评论，将自己的想法都在评论中写出来！

#Instruments Core Animation界面

我们先来说说Instruments中的选项之一：Core Animation。运行后，默认看到右下角是第一个按钮，勾选上用于设置监测FPS的：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160305-2@2x-e1457176662621.png)

而我们根本每一选项做优化，要看下图，每次只勾选一个，一步步优化：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160305-0@2x-1-e1457176729557.png)

#优化前

根据官方说明，图片像素不对齐（也就是图片带alpha通道）时，会在图片上面加一层洋红色来标识；而图片被缩放时，会加一层黄色来标识。那么先看看我们的demo中被优化前的UI：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160305-3@2x-e1457177300979.png)

笔者通过网上找了一些带alpha通道的png图片，果真被标识为洋红色了。不过奇怪的是，两个一样的图片，为什么显示成正方形的时候，并没有被标识为像素不对齐呢？

笔者尝试修改左上角的头像为长方形，不过并没有用，还是显示为黄色而非洋红色。而且，这设置图片的代码是一样一样的，结果还是没有变化修改成洋红色，这个问题笔者暂时找不到原因。

希望大家知道原因的，可以告知一声，谢谢！

在优化前，我们监测FPS如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/屏幕快照-2016-03-05-下午4.00.40-e1457177972685.png)

右边一列是FPS值的变化，我们是在滚动时的FPS，说明不做优化处理，对于滚动的流畅性影响是比较大的。

在优化前的代码是：

```
NSString *path = nil;
if ([model.headImg hasSuffix:@".png"]) {
	path = model.headImg;
} else {
	path = [[NSBundle mainBundle] pathForResource:model.headImg ofType:nil];
}
UIImage *image = [UIImage imageNamed:path];
self.headImageView.image = image;
```

对于UICollectionViewCell中显示的图片优化前是这样的：

```
NSString *imgName = self.model.imgs[indexPath.row];
NSString *path = nil;
if ([imgName hasSuffix:@".png"]) {
	path = imgName;
} else {
	path = [[NSBundle mainBundle] pathForResource:imgName ofType:nil];
}
UIImage *image = [UIImage imageNamed:path];
imgView.image = image;
```

#优化后

在优化后，我们通过真机监测FPS在滚动时的变化，可以看到是比较稳定地处于60左右，说明优化的效果是比较明显的：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/屏幕快照-2016-03-05-下午3.56.48-e1457178332527.png)

下面我们采用等比例的方式来生成新的图片并缓存起来，看到优化后的效果：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160305-4@2x-e1457178696977.png)

我们发现洋红色没有了，黄色也没有了。说明我们已经解决掉这个问题了。

###等比例缩放

那么我们采用等比例缩放图片的代码如下（给UIImage添加了一个扩展方法）：

```
// 等比缩放
- (UIImage *)hyb_cropEqualScaleImageToSize:(CGSize)size {
    CGFloat scale =  [UIScreen mainScreen].scale;
  
  // 这一行至关重要
  // 不要直接使用UIGraphicsBeginImageContext(size);方法
  // 因为控件的frame与像素是有倍数关系的
  // 比如@1x、@2x、@3x图，因此我们必须要指定scale，否则黄色去不了
  // 因为在5以上，scale为2，6plus scale为3，所生成的图是要合苹果的
  // 规格才能正常
  UIGraphicsBeginImageContextWithOptions(size, NO, scale);
  
  CGSize aspectFitSize = CGSizeZero;
  if (self.size.width != 0 && self.size.height != 0) {
    CGFloat rateWidth = size.width / self.size.width;
    CGFloat rateHeight = size.height / self.size.height;
    
    CGFloat rate = MIN(rateHeight, rateWidth);
    aspectFitSize = CGSizeMake(self.size.width * rate, self.size.height * rate);
  }
  
  [self drawInRect:CGRectMake(0, 0, aspectFitSize.width, aspectFitSize.height)];
  UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
  UIGraphicsEndImageContext();
  
  return image;
}
```

###直接缩放至指定大小

下图是不按等比例来缩放，而是直接生成指定大小的图，优化后的效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160305-1@2x-1-e1457179097762.png)

而我们生成指定大小的图的代码如下：

```
// 非等比缩放，生成的图片可能会被拉伸
- (UIImage *)hyb_cropEqualScaleImageToSize:(CGSize)size {
  CGFloat scale =  [UIScreen mainScreen].scale;
 
  UIGraphicsBeginImageContextWithOptions(size, NO, scale);
  
  [self drawInRect:CGRectMake(0, 0, size.width, size.height)];
  UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
  UIGraphicsEndImageContext();
  
  return image;
}
```

####配置用户头像

配置用户头像处的代码就修改成如下，换成在子线程里处理图片裁剪，生成与headImageView正好合适的图片的大小，这样就不会出现图片需要UIImageView压缩的问题，性能自然得到提升：

```
// 优化后
dispatch_async(dispatch_get_global_queue(0, 0), ^{
  NSString *path = nil;
  if ([model.headImg hasSuffix:@".png"]) {
    path = model.headImg;
  } else {
   path = [[NSBundle mainBundle] pathForResource:model.headImg ofType:nil];
  }
  UIImage *image = [[UIImage imageNamed:path] hyb_cropEqualScaleImageToSize:self.headImageView.frame.size];
  dispatch_async(dispatch_get_main_queue(), ^{
    self.headImageView.image = image;
  });
});
```

####配置CollectionViewCell

在优化后，我们在配置UICollectionView中的图片时，代码是这样的：

```
NSString *imgName = self.model.imgs[indexPath.row];
if ([self.model.cacheImages objectForKey:imgName]) {
  imgView.image = [self.model.cacheImages objectForKey:imgName];
} else {
  
  dispatch_async(dispatch_get_global_queue(0, 0), ^{
    
    NSString *path = nil;
    if ([imgName hasSuffix:@".png"]) {
      path = imgName;
    } else {
      path = [[NSBundle mainBundle] pathForResource:imgName ofType:nil];
    }

    UIImage *image = [[UIImage imageNamed:path] hyb_cropEqualScaleImageToSize:imgView.frame.size];
    dispatch_async(dispatch_get_main_queue(), ^{
      imgView.image = image;
      if (image) {
        [self.model.cacheImages setObject:image forKey:imgName];
      }
    });
  });
}
```

我们首先是放在异步线程去去处图片的裁剪生成新的合适的图片，然后再回到主线程来更新显示图片。这么处理就不会阻塞主线程了，流畅度是否好多了~

#注意事项

当我们设置不透明度为NO时，表示透明，这时候通过Color Blended Layers发现图片变成红色了，那是因为透明原因，将第二个参数设置成YES就可以解决了：

```
UIGraphicsBeginImageContextWithOptions(size, NO, scale);
```

#最后

对于图片优化问题，个人觉得应该要根据情况而定，如果是在列表中需要滚动的，希望滚动可以很流畅的，才需要这么去优化它。对于那些不是滚动的界面，没有必须非得去优化它。

请大家多多反馈意见，说说自己的想法，在遇到问题能够将解决方案告知笔者，共同进步！

眼前还有三个月就要去深圳谋发展了，写下本篇，就当是提前准备准备吧！

#总结

* 优化UIImageView显示出现图片大小与控件大小不一致时，可以采用重新生成新的正好合适的图片上这种方式。
* 同样像UIButton这样也需要显示图片时，也可以采用UIImageView这种方式
* 对新生成的图片，最好缓存起来

#源代码

笔者提供了小Demo的，这个小demo也是如何在cell中嵌入collectionview的使用。

GITHUB下载地址：[CoderJackyHuang's PerformanceDemo](https://github.com/CoderJackyHuang/PerformanceDemo.git)

#参考

* [UIKit性能调优实战讲解](http://www.jianshu.com/p/619cf14640f3)



