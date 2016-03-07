#前言

有时候使用UITableView所实现的列表，会使用到header view，但是又不希望它粘在最顶上而是跟随滚动而消失或者出现，下面的代码片段就是实现此功能：

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {  
    if (scrollView == _tableView) {  
        CGFloat sectionHeaderHeight = 36;
        
        if (scrollView.contentOffset.y <= sectionHeaderHeight && scrollView.contentOffset.y >= 0) {  
            scrollView.contentInset = UIEdgeInsetsMake(-scrollView.contentOffset.y, 0, 0, 0);  
        } else if (scrollView.contentOffset.y >= sectionHeaderHeight) {  
            scrollView.contentInset = UIEdgeInsetsMake(-sectionHeaderHeight, 0, 0, 0);  
        }  
    }  
}  
```

其中，header view是指tableview的代理方法：

```
- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section
```

#说明

* sectionHeaderHeight 的值要根据自己的而定
* \_tableView 如果一个类里有多个表格，要明确指明要去掉哪一个表格头的粘性


