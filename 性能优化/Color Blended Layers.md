#前言

最近笔者也是对性能优化方面特别感兴趣，为此做一性能优化相关的专题，记下学习的笔记，后来者都可以参考参考。

本篇文章专门学习**Color Blended Layers**这一项优化。翻译过来，大概的意思就是图层颜色混合。

官方的说明是：

Shows blended view layers. Multiple view layers that are drawn on top of each other with blending enabled are highlighted in red. Reducing the amount of red in your app when this option is selected can dramatically improve your app’s performance. Blended view layers often cause slow table scrolling.

#基本概念

我们要明白**像素**的概念。屏幕上每一个点都是一个像素点，颜色由R、G、B、alpha组成。如果某一块区域上覆盖了多层layer，最后所计算出来的显示的颜色效果，会受到这些layer的共同影响。

**举个例子：**

上层是蓝色(RGB=0,0,1),透明度为50%，下层是红色(RGB=1,0,0)。那么最终的显示效果是紫色(RGB=0.5,0,0.5)。

这种颜色的混合(blended)需要消耗一定的GPU资源，在实际开发中可能不止只有两层。如果只想显示最上层的颜色，可以把它的透明度设置为100%，这样GPU会忽略下面所有的layer，从而节约了很多不必要的运算。

#Core Animation Instrument

类似如下图，可以通过Instruments来选择Core Animation查看FPS：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/1430211590521452.png)

注意：若要看FPS，需要在**真机**上运行。

#Demo优化前

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160305-0@2x-e1457152933753.png)

在优化之前，我们通过模拟器->Debug->Color Blended Layers可看到上图的效果。我们发现有好多混合的地方，这都是影响滚动时流畅性的。

#Demo优化后

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160305-1@2x-e1457153005355.png)

优化后，我们看到没有中文标签变成绿色了，而有中文的标签还是混合后的颜色。关于有中文的标签如何去优化，这个问题现在没有办法，如果大家有什么好的方法，请一定要告诉笔者，成分感谢！(补充：已经找到解决办法，看文章末尾)

#优化的代码

在优化前，cell的标题的背景颜色并没有手动设置，而是使用了默认的颜色，这时候即使我们的标题是没有中文的，也会出现混合。而笔者只是添加了一行代码：

```
self.titleLabel.backgroundColor = self.contentView.backgroundColor;
```

这么一行代码，使得标签的背景色和父视图的背景颜色一样，就只可以解决混合的问题（中文除外）。
如何我们设置背景色为clear，一样会出现混合，即使父视图也是clear。

同样，我们在配置CollectionViewCell的时候，也是这么处理：

```
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {
  UICollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"identifeir"
                                                                         forIndexPath:indexPath];
  UIImageView *imgView = [cell.contentView viewWithTag:100];
  if (imgView == nil) {
    cell.contentView.backgroundColor = [UIColor whiteColor];
    
    imgView = [[UIImageView alloc] init];
    imgView.frame = CGRectMake(0,
                               0,
                               cell.contentView.bounds.size.width,
                               cell.contentView.bounds.size.height - 20);
    [cell.contentView addSubview:imgView];
    imgView.tag = 100;
    imgView.layer.shadowPath = nil;
    
    UILabel *titleLabel = [[UILabel alloc] init];
    titleLabel.numberOfLines = 0;
    titleLabel.text = @"www.henishuo.com";
    [cell.contentView addSubview:titleLabel];
    titleLabel.tag = 101;
    titleLabel.font = [UIFont systemFontOfSize:12];
    titleLabel.frame = CGRectMake(0, imgView.frame.size.height, imgView.frame.size.width, 20);
    titleLabel.backgroundColor = cell.contentView.backgroundColor;
  }
  
  NSString *imgName = self.model.imgs[indexPath.row];
  NSString *path = [[NSBundle mainBundle] pathForResource:imgName ofType:@"jpg"];
  imgView.image = [UIImage imageNamed:path];
  
  return cell;
}
```

我们设置cell的背景色、collectionview的背景色都为白色，然后titleLabel的背景色与父视图的背景色设置成一样，这样就可以解决混合问题，中文文本除外。

我们尝试设置opaque、alpha都会是图层混合，因此最关键的还是backgroundColor这一关键属性。

#UILabel中文会变红的解决方案

在微博发出来后，终于有高人指出原因。虽然设置了背景色，但在iOS8及其以上版本，用UILabel显示中文却出现了像素混合的情况，这是为什么呢？我们来看看UILabel在iOS8前后的变化。

* 在iOS8以前的版本：UILabel使用的是CALayer作为底图层
* 在iOS8及其以上版本：UILabel的底图层变成了\_UILabelLayer，绘制文本也有所改变

我们可以通过打印日志可以看到这个\_UILabelLayer。

那么如何解决呢？我们仔细地看一看那个UILabel变红区域，我们发现文字并不是紧紧地靠着边界，而是与边界有一定的距离，其实这是超出了显示的内容的范围，从而引起了像素混合。

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160308-0@2x.png)

看到标签显示中文也变成绿色了吗？哈哈，这是因为添加了一行代码而解决的。将UILabel沿着bounds来裁剪，这样就不会超出来了，问题也就解决了：

```
self.descLabel.layer.maskToBounds = YES;
```

不过在iOS8之前，我们不用设置它的。更详细地请看后面的参考列表。

#最后

上面讲的只是UILabel的，但是实际开发中还会有很多的控件的，那么我们也要注意处理。这里只是抛砖引玉，希望得到大家的评论，分别提出更多更好的方法。

当某天找到中文显示图层颜色混合的解决方案的时候，会更新本篇Instruments文章的！

#提示

如果在使用Instucments时，真机不能点击启动，请阅读文章：

* [Instruments启动按钮不可点解决办法](http://www.henishuo.com/Instruments-cannot-click/)

#总结

* 尽量设置控件的backgroundColor属性与父视图的背景色一样，如果UI效果不允许，那么可以尝试别的方式来处理
* 尽量不要直接使用clear背景色，因为还是会图层颜色混合，除非真的必须这么做。

#源代码

笔者提供了小Demo的，这个小demo也是如何在cell中嵌入collectionview的使用。

GITHUB下载地址：[CoderJackyHuang's PerformanceDemo](https://github.com/CoderJackyHuang/PerformanceDemo.git)

#参考

* [UIKit性能调优实战讲解](http://www.jianshu.com/p/619cf14640f3)
* [iOS图形性能](http://www.cocoachina.com/ios/20150429/11712.html)
* [UILabel在iOS8下的Color Blended Layers](http://www.jianshu.com/p/db6602413fa3)


