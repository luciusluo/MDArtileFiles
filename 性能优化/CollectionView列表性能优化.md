#概述

本篇一起来学习如何优化UICollectionView实现的网格布局，这里只是展示图片和文字，但是图片比较大，而且比较多。在优化之前，很明显的一卡一卡的。

在优化之后，FPS达到了平稳的58~60，快速滚动时，基本都是60，而且在优化后通过Core Animations检测已经没有离屏渲染、图层混合等。

#优化后的FPS

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160326-2@2x.png)

#效果图

![image](http://www.henishuo.com/wp-content/uploads/2016/03/griddemo.gif)

这是在前一篇讲如何实现网格布局的文章中的效果图，也是本篇文章要优化的内容。如果对网格布局不太懂，可以先阅读【[UICollectionView网格布局](http://www.henishuo.com/collectionview-grid-layout/)】

#优化图片

**优化第一步：**直接使用UIView，而不是UIImageView，这样更轻量：

```
@property (nonatomic, strong) UIView *imageView;
```

**优化第二步：**裁剪图片大小至控件的大小

裁剪前的效果图。裁剪前，很明显被压缩得很厉害，已经变形了，效果很差：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160326-0@2x.png)

裁剪后的效果图。裁剪后，图片大小按照一定的比例裁剪，且图片的大小与控件的大小正好一致，也就没有了color mismatch images的问题了：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160326-1@2x.png)

因为这里的背景的黑色，所以使用不透明。裁剪图片的代码：

```
- (UIImage *)clipImage:(UIImage *)image toSize:(CGSize)size {
  UIGraphicsBeginImageContextWithOptions(size, YES, [UIScreen mainScreen].scale);
  
  CGSize imgSize = image.size;
  CGFloat x = MAX(size.width / imgSize.width, size.height / imgSize.height);
  CGSize resultSize = CGSizeMake(x * imgSize.width, x * imgSize.height);
  
  [image drawInRect:CGRectMake(0, 0, resultSize.width, resultSize.height)];
  
  UIImage *finalImage = UIGraphicsGetImageFromCurrentImageContext();
  UIGraphicsEndImageContext();
  
  return finalImage;
}
```

异步去剪裁图片，在剪裁完成后再更新：

```
dispatch_async(dispatch_get_global_queue(0, 0), ^{
  UIImage *image = [UIImage imageNamed:model.imageName];
  image = [self clipImage:image toSize:self.imageView.frame.size];
  dispatch_async(dispatch_get_main_queue(), ^{
    model.clipedImage = image;
    self.imageView.layer.contents = (__bridge id _Nullable)(model.clipedImage.CGImage);
  });
});
```

**优化第三步：**缓存剪裁的图片

在实际开发中，我们如果要优化，可以将剪裁后的图片缓存到本地。不过这里为了简单，只是将裁剪后的图片缓存到内存中，避免重复裁剪。我们给model增加一个字段，记录剪裁后的图片，然后在加载图片的地方，判断一下：

```
if (model.clipedImage) {
	self.imageView.layer.contents = (__bridge id _Nullable)(model.clipedImage.CGImage);
} 
```

这里使用的是UIView来呈现图片，这样就更轻量一些。

到这里，已经优化得很不错了。

#优化UILabel

我们知道UILabel的图层混合问题，在中文情况下会出现图层混合。而非中文情况下，如果背景色与父视图的背景色不同或者设置为透明，都会引起图层混合，下面我们来优化一下。

**优化第一步：**设置背景色与父视图相同，对于非中文就可以解决图片混合问题

```
self.titleLabel.backgroundColor = [UIColor blackColor];
```

**优化第二步：**处理中文图层混合问题

```
self.titleLabel.layer.masksToBounds = YES;
```

#结尾

优化完之后，大家可以下载demo来运行看看，通过Instruments工具的Core Animations来检测一下，没有出现图层混合了，没有出现像素不对齐了，也没有离屏渲染。


#源代码

[CollectionViewDemos](https://github.com/CoderJackyHuang/CollectionViewDemos)

**提示：**本篇文章的demo对应于工程中的Demo1-GridCollectionView分组。


#推荐阅读

* [iOS性能优化之TimeProfiler](http://www.henishuo.com/ios-instrucments-optimize/)
* [Color Blended Layers](http://www.henishuo.com/color-blended-layers/)
* [Color Misaligned Images](http://www.henishuo.com/color-misaligned-images/)
* [Offscreen-Rendered](http://www.henishuo.com/offscreen-rendered/)