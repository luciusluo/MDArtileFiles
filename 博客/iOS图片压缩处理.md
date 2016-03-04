>#iOS图片压缩处理

---
首先，我们必须明确图片的**压缩**其实是两个概念：

1. “压” 是指文件体积变小，但是像素数不变，长宽尺寸不变，那么质量可能下降。
2. “缩” 是指文件的尺寸变小，也就是像素数减少，而长宽尺寸变小，文件体积同样会减小。

>#支持原创，请[阅读原文](http://www.henishuo.com/ios-image-compressed/)

#图片“压”处理

---
对于“压”的功能，我们可以使用`UIImageJPEGRepresentation`或`UIImagePNGRepresentation`方法实现，如：

```
NSData *imgData = UIImageJPEGRepresentation(image, 0.5);
```
第一个参数是图片对象，第二个参数是压的系数，其值范围为0~1。

>`UIImageJPEGRepresentation`方法的官方注释是：return image as JPEG. May return nil if image has no CGImageRef or invalid bitmap format. compression is 0(most)..1(least)

###关于PNG和JPEG格式压缩
1. `UIImageJPEGRepresentation`函数需要两个参数:图片的引用和压缩系数而`UIImagePNGRepresentation`只需要图片引用作为参数.

2. `UIImagePNGRepresentation(UIImage \*image)`要比`UIImageJPEGRepresentation(UIImage* image, 1.0)`返回的图片数据量大很多.

同样的一张照片, 使用`UIImagePNGRepresentation(image)`返回的数据量大小为`199K`,而 `UIImageJPEGRepresentation(image, 1.0)`返回的数据量大小只为`140K`,比前者少了`59K`.

如果对图片的清晰度要求不是极高,建议使用`UIImageJPEGRepresentation`，可以大幅度降低图片数据量.比如,刚才拍摄的图片,通过调用`UIImageJPEGRepresentation(image, 1.0)`读取数据时,返回的数据大小为`140K`,但更改压缩系数为`0.5`再读取数据时,返回的数据大小只有`11K`,大大压缩了图片的数据量,而且清晰度并没有相差多少,图片的质量并没有明显的降低。因此，在读取图片数据内容时，建议优先使用`UIImageJPEGRepresentation`,并可根据自己的实际使用场景,设置压缩系数,进一步降低图片数据量大小。

>提示：压缩系数不宜太低，通常是0.3~0.7，过小则可能会出现黑边等。

我们看一下笔者使用`UIImageJPEGRepresentation`的数据表：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-01-下午5.52.352.png)

笔者统计了`iphone`设备上的全屏图和原始图在压缩前和压缩后的大小，我们需要根据图片压缩后在PC上的清晰度来决定最终选择哪个压缩系数。

#图片“缩”处理

---
通过`[sourceImage drawInRect:CGRectMake(0, 0, targetWidth, targetHeight)]`可以进行图片“缩”的功能。如下是笔者对图片尺寸缩的`api`：

```
/*!
 *  @author 黄仪标, 15-12-01 16:12:01
 *
 *  压缩图片至目标尺寸
 *
 *  @param sourceImage 源图片
 *  @param targetWidth 图片最终尺寸的宽
 *
 *  @return 返回按照源图片的宽、高比例压缩至目标宽、高的图片
 */
- (UIImage *)compressImage:(UIImage *)sourceImage toTargetWidth:(CGFloat)targetWidth {
  CGSize imageSize = sourceImage.size;
  
  CGFloat width = imageSize.width;
  CGFloat height = imageSize.height;
  
  CGFloat targetHeight = (targetWidth / width) * height;
  
  UIGraphicsBeginImageContext(CGSizeMake(targetWidth, targetHeight));
  [sourceImage drawInRect:CGRectMake(0, 0, targetWidth, targetHeight)];
  
  UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
  UIGraphicsEndImageContext();
  
  return newImage;
}
```

我们对图片只“压”而不缩，有时候是达不到我们的需求的。因此，适当地对图片“缩”一“缩“尺寸，就可以满足我们的需求。

>提示：我们获取到的全屏图的宽高不是指设备的宽高，通常会比设置的宽要大，高度可能相等。我们可以拍全景图，那么宽就很大了。


#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**

