#前言

今天在<http://www.reviewcode.cn/reviewer.html?id=CcPkVLo96sIPCVG2>收到一份订单，本来以为只是进行Code Review，没有想到是帮忙解BUG。针对这个问题，我写了一个demo，并写下本篇文章，提供解决的思路。

#问题描述如下

在tableview的cell的多选题的情况下，有互斥的功能，根据RecordTwoLevelModel里面的参数strMutex_id，就是跟这个选项互斥的选项的id，这个bug就是选项之间现在不能互斥。

更详细地问题描述可以看这里：<https://github.com/yidakouneihan/Sino/issues/1>。这是接收订单后对需求者所确认的问题描述。

#数据建模

```
@interface HYBTestModel : NSObject

// 问题id
@property (nonatomic, copy) NSString *qid;
@property (nonatomic, copy) NSString *questionSummary;
// 修改为strong声明，当然使用copy也没有关系，因为我重写了getter方法，
// 当然这里应该声明为readonly更佳一些。
@property (nonatomic, strong) NSMutableArray *optionalAnswers;

@end

@interface HYBOptionalAnswerModel : NSObject

// 选项答案id
@property (nonatomic, copy) NSString *aid;
// 选项答案内容描述
@property (nonatomic, copy) NSString *optionalAnswerSummary;

// 辅助字段，标识是否选中
@property (nonatomic, assign) BOOL isSelected;

// 互斥的选项，以英文逗号分割
@property (nonatomic, copy) NSString *strMutex_id;

- (NSArray *)mutexIds;

@end
```

这里是使用了两个模型，HYBTestModel是问题模型，HYBOptionalAnswerModel是可选答案的模型。它有一个互斥id字符中，用英文逗号分隔。

#A、B、C与D互斥

我们先看在做多选题时，最常见的就是四个选项中有一个选项是全不选，当选择全不选时，其它三个都要反选。当A、B或者C选中是，D一定要反选。如下效果图：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/multiselectabc.gif)

#关键代码

当选中某一个选项答案时，将所有与之互斥的反选。但是，当反选自己时，只是简单的反选，这样就可以解决互斥问题了。

```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
  HYBTestModel *model = [self.datasource objectAtIndex:indexPath.section];
  HYBOptionalAnswerModel *answerModel = [model.optionalAnswers objectAtIndex:indexPath.row];
  
  answerModel.isSelected = !answerModel.isSelected;
  
  // 这里假设的是互斥是单一的，也就是说不存在多个相互互斥的状态。这里只考虑A/B/C/D中A/B/C都到D互斥的情况
  if (answerModel.isSelected) {
    for (HYBOptionalAnswerModel *otherAnswerModel in model.optionalAnswers) {
      if (otherAnswerModel != answerModel && [answerModel.mutexIds containsObject:otherAnswerModel.aid]) {
        // 互斥
        otherAnswerModel.isSelected = !answerModel.isSelected;
      }
    }
  }
  
  [tableView reloadSections:[NSIndexSet indexSetWithIndex:indexPath.section]
           withRowAnimation:UITableViewRowAnimationFade];
}
```

#A与B互斥、C与D互斥

我们再来看看A与B互斥、C与D互斥的例子。如果选A就不能选B，选B就不能先A；同样选C就不能选D，选D就不能先C。但是AB与CD互不干扰。

![image](http://www.henishuo.com/wp-content/uploads/2016/03/multiselectab_cd.gif)

所处理的逻辑与上面的关键代码是一样的，因此这一段关键的代码的可以满足我们的需求了。

#提示

对于这种类型的问题，一定要采用模型法来处理。通过数据建模，可以非常方便地处理重用问题及多选问题。如果你还在通过记录indexPath、记录cell来处理这些问题，那不防试试笔者的方法吧。

大家可以到<http://www.reviewcode.cn/reviewer.html?id=CcPkVLo96sIPCVG2>看看吧！

#源代码

大家可以下载源代码来参考参考：[CoderJackHuang：MultiSelectMutexDemo](https://github.com/CoderJackyHuang/MultiSelectMutexDemo)

