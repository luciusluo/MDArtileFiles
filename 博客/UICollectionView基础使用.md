>#UICollectionView基础教程

---
这个控件，看起来与UITableView有点像，而且基本的用法也很相像哦！！！

我们来看看API:

```
#pragma mark - UICollectionViewDataSource  
// 指定Section个数  
- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView {  
  return 3;  
}  
  
// 指定section中的collectionViewCell的个数  
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section {  
  return 10;  
}  
  
// 配置section中的collectionViewCell的显示  
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {  
  CollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"CellIdentifier" forIndexPath:indexPath];  
  cell.backgroundColor = [UIColor redColor];  
  cell.textLabel.text = [NSString stringWithFormat:@"(%ld %ld)", indexPath.section, indexPath.row];  
    
  return cell;  
}  
```

看看直线布局的API：

```
#pragma mark - UICollectionViewDelegateFlowLayout  
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath {  
  return CGSizeMake(self.view.frame.size.width / 3 - 10, self.view.frame.size.width / 3 - 10);  
}  
  
// 设置每个cell上下左右相距  
- (UIEdgeInsets)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout insetForSectionAtIndex:(NSInteger)section {  
  return UIEdgeInsetsMake(5, 5, 5, 5);  
}  
  
// 设置最小行间距，也就是前一行与后一行的中间最小间隔  
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumLineSpacingForSectionAtIndex:(NSInteger)section {  
  return 10;  
}  
  
// 设置最小列间距，也就是左行与右一行的中间最小间隔  
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumInteritemSpacingForSectionAtIndex:(NSInteger)section {  
  return 10;  
}  
  
// 设置section头视图的参考大小，与tableheaderview类似  
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForHeaderInSection:(NSInteger)section {  
  return CGSizeMake(self.view.frame.size.width, 40);  
}  
  
// 设置section尾视图的参考大小，与tablefooterview类似  
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForFooterInSection:(NSInteger)section {  
  return CGSizeMake(self.view.frame.size.width, 40);  
}  
```

如果是固定的，可以使用全局属性：

```
NS_CLASS_AVAILABLE_IOS(6_0) @interface UICollectionViewFlowLayout : UICollectionViewLayout  
  
@property (nonatomic) CGFloat minimumLineSpacing;  
@property (nonatomic) CGFloat minimumInteritemSpacing;  
@property (nonatomic) CGSize itemSize;  
@property (nonatomic) CGSize estimatedItemSize NS_AVAILABLE_IOS(8_0); // defaults to CGSizeZero - setting a non-zero size enables cells that self-size via -perferredLayoutAttributesFittingAttributes:  
@property (nonatomic) UICollectionViewScrollDirection scrollDirection; // default is UICollectionViewScrollDirectionVertical  
@property (nonatomic) CGSize headerReferenceSize;  
@property (nonatomic) CGSize footerReferenceSize;  
@property (nonatomic) UIEdgeInsets sectionInset;  
  
@end  
```

常用到的代理方法：

```
#pragma mark - UICollectionViewDelegate  
// 允许选中时，高亮  
- (BOOL)collectionView:(UICollectionView *)collectionView shouldHighlightItemAtIndexPath:(NSIndexPath *)indexPath {  
  NSLog(@"%s", __FUNCTION__);  
  return YES;  
}  
  
// 高亮完成后回调  
- (void)collectionView:(UICollectionView *)collectionView didHighlightItemAtIndexPath:(NSIndexPath *)indexPath {  
  NSLog(@"%s", __FUNCTION__);  
}  
  
// 由高亮转成非高亮完成时的回调  
- (void)collectionView:(UICollectionView *)collectionView didUnhighlightItemAtIndexPath:(NSIndexPath *)indexPath {  
  NSLog(@"%s", __FUNCTION__);  
}  
  
// 设置是否允许选中  
- (BOOL)collectionView:(UICollectionView *)collectionView shouldSelectItemAtIndexPath:(NSIndexPath *)indexPath {  
  NSLog(@"%s", __FUNCTION__);  
  return YES;  
}  
  
// 设置是否允许取消选中  
- (BOOL)collectionView:(UICollectionView *)collectionView shouldDeselectItemAtIndexPath:(NSIndexPath *)indexPath {  
  NSLog(@"%s", __FUNCTION__);  
  return YES;  
}  
  
// 选中操作  
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath {  
  NSLog(@"%s", __FUNCTION__);  
}  
  
// 取消选中操作  
- (void)collectionView:(UICollectionView *)collectionView didDeselectItemAtIndexPath:(NSIndexPath *)indexPath {  
  NSLog(@"%s", __FUNCTION__);  
}
```  

这里只是简单介绍其API及基础用法！！！

#源代码

---
请到`github`下载`demo`：[https://github.com/CoderJackyHuang/CollectionViewDemo](https://github.com/CoderJackyHuang/CollectionViewDemo)