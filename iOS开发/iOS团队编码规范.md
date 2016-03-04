
团队开发一定要有很好的编码规范才利于团队开发与快速维护。我相信大家在维护老代码的时候，都有这么一种心理：真想骂死那些没有留下一点点注释，而且代码风格极差的家伙，远看一朵云，近看却是一陀`XX`。

#模型(Model)规范


以下为笔者定义模型的代码规范：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/ModelStyle.gif)

对于定义模型类，属性名就跟接口中所返回的参数名一样就可以了，除非接口所定义的返回数据字段与我们`ios`的关键字相同而不得不修改为另外一个名字。

下面是笔者比较崇尚的一种写法。这里接口返回的字段是`type`，而这个字段所返回的值为`1,2,3`之类的，而视图显示的却是`本人`、`父亲`之类的。因此，我们应该将逻辑放到模型中，外部不能修改，只能获取，而视图这里只负责显示即可，就不需要去处理逻辑了。

```
// 患者类型，这个字段外部不要直接使用，改成直接使用relationship
@property (nonatomic, copy) NSString *type;
// 将获取逻辑放在统一一个位置，外部不能修改，取到直接用即可
@property (nonatomic, copy, readonly) NSString *relationship;
```

#视图(View)规范


以下是笔者定制公共`cell`的代码规范：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/cellStyle.gif)

我们对于定制一个`cell`之类的视图时，我们应该使用一个专门的`API`来配置数据，而不是公共各个控件，而外部又一个个地调用控件来配置数据，这样是很不合理的。

这里所定制的`cell`风格类似，但是是多个界面多个模型公共用，因此，笔者这里使用了的规则是：每个界面对应一个配置数据的`API`。

```
/**
 * 选择患者界面才能调用此方法来配置数据
 *
 * @param model 选择患者界面对应的数据模型
 */
- (void)confiSelectedPatientCellWithModel:(HYBJiaHaoSelectPatientModel *)model;

/*!
 *  @author 黄仪标, 15-11-11 11:11:35
 *
 *  配置既往疾病描述界面对应的cell
 *
 *  @param model 既往疾病模型
 */
- (void)configHistoryDiseaseCellWithModel:(HYBJiaHaoHistoryDiseaseModel *)model
                                   isInfo:(BOOL)isInfo;

/*!
 *  @author 黄仪标, 15-11-11 11:11:35
 *
 *  配置既往就诊医院界面对应的cell
 *
 *  @param model 既往就诊医院模型
 */
- (void)configHistoryHospitalCellWithModel:(HYBJiaHaoHistoryHospitalModel *)model;

/*!
 *  @author 黄仪标, 15-11-11 14:11:27
 *
 *  配置加号目的界面对应的cell
 *
 *  @param title 显示的标题
 */
- (void)configPurposeCellWithTitle:(NSString *)title;
```

像这里，我们在外部调用是很简洁的。如下：

```
#pragma mark - UITableViewDataSource
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
  HYBJiaHaoSelectItemCell *cell = [HYBJiaHaoSelectItemCell HYB_dequeueWithTableView:tableView];
  
  if (cell == nil) {
    cell = [HYBJiaHaoSelectItemCell HYB_cellWithDefaultStyle];
  }
  
  if (indexPath.row < self.datasources.count) {
    HYBJiaHaoHistoryHospitalModel *model = [self.datasources HYB_safeObjectAtIndex:indexPath.row];
    [cell configHistoryHospitalCellWithModel:model];
  }
  
  return cell;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
  return self.datasources.count;
}

#pragma mark - UITableViewDelegate
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
  if (indexPath.row < self.datasources.count) {
    HYBJiaHaoHistoryHospitalModel *model = [self.datasources HYB_safeObjectAtIndex:indexPath.row];
    
    return [HYBJiaHaoSelectItemCell HYB_heightForIndexPath:indexPath configBlock:^(UITableViewCell *cell) {
      HYBJiaHaoSelectItemCell *itemCell = (HYBJiaHaoSelectItemCell *)cell;
      [itemCell configHistoryHospitalCellWithModel:model];
    }];
  }
  
  return 0;
}
```
这里是通过`[cell configHistoryHospitalCellWithModel:model];`来配置数据，外部并没有任何的逻辑。而下面这里是自动计算`cell`的高度，这是使用了笔者为`UITableViewCell`扩展的`API`，基于`Masonry`

#控制器规范


以下是笔者的控制器类的效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/ControllerStyle.gif)

我们对于控制器类，我们对外的接口都是统一的。对外不公开属性，统一将所需要传的参数放到初始化`api`中，将外部所传的参数放到内部作为私有。这么做的好处是，所以需要维护这些代码的人，不需要去一个个查那一堆的属性到底哪个是必传，哪些是可以不传。如下：

```
/*!
 *  @author 黄仪标, 15-11-18 23:11:23
 *
 *  此API只针对添加患者信息界面
 *
 *  @param submitModel 传递参数，最后一步需要的模型
 *  @param addSuccess  添加成功后的回调
 *
 *  @return 控制器对象
 */
- (instancetype)initAddWithSubmitModel:(HYBJiaHaoSubmitModel *)submitModel
                          onAddSuccess:(HYBVoidBlock)addSuccess;

/*!
 *  @author 黄仪标, 15-11-14 21:11:40
 *
 *  进入编辑患者信息界面
 *
 *  @param submitModel  最后需要提交的模型
 *  @param isOldPatient 是否是老患者
 *  @param editSuccess  编辑保存成功后的回调
 *
 *  @return 控制器对象
 */
- (instancetype)initEditWithSubmitModel:(HYBJiaHaoSubmitModel *)submitModel
                            patientModel:(HYBJiaHaoSelectPatientModel *)patientModel
                           isOldPatient:(BOOL)isOldPatient
                          onEditSuccess:(HYBVoidBlock)editSuccess;
```

由于控制器类通常代理逻辑比较多，而且代码量也比较大，因此我们需要很明确的代码规范，以让我们快速的查找到我们需要查询的代码。

比如，这里是部分效果：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/屏幕快照-2015-11-18-下午11.11.37.png)

这里的分区就很清晰，当我们需要处理网络接口时，我们直接就可以查看`Network`这里，然后点击对应要处理的接口就可以直接跳转到接口这里，定位效率极高。

通常的风格写法如下：

```
#pragma mark - LifeCycle

#pragma mark - UI

#pragma mark - Network
#pragma mark -- 上传图片
#pragma mark -- 保存用户数据

#pragma mark - Delegate
#pragma mark -- UITableViewDataSource
#pragma mark -- UITableViewDelegate

#pragma mark - Getter/Setter

#pragma mark - Private
#pragma mark -- 进入用户编辑信息界面
#pragma mark -- 进入用户二维码界面
```

上面像`进入用户二维码界面`这种只是一个例子。`-`是一级，`--`是前者的子级，风格就很清晰了。

#善于重写Getter方法


在开发中，尽量不要使用`_name`这种类型的调用，而是声明为属性，直接使用`self.name`这样的写法。声明为属性，我们可以重写`getter`方法，而且就是所谓的`lazy loading`。如下就是一个例子，只有在使用到的时候，直接通过`self.yearSources`就可以直接使用了，而不需要再提供一个方法来初始化数据：

```
#pragma mark - Getter
#pragma mark -- 初始化年份
- (NSMutableArray *)yearSources {
  if (_yearSources == nil) {
    _yearSources = [[NSMutableArray alloc] init];
    
    int curYear = (int)[[NSDate date] HYB_year];
    for (int year = 1970; year <= curYear + 100; ++year) {
      [_yearSources addObject:@(year)];
    }
  }
  
  return _yearSources;
}
```

#善于重写setter方法


很多朋友不太喜欢重写`setter`方法，而是单独再提供一个`api`来更新数据。事实上，我们通过重写`setter`方法，可以给我们带来很大的便利。看下面的例子：

```
#pragma mark - Setter
- (void)setSelectedHospitalModel:(HYBJiaHaoConsultHistoryModel *)selectedHospitalModel {
  if (_selectedHospitalModel != selectedHospitalModel) {
    _selectedHospitalModel = selectedHospitalModel;
    
    self.hospitalTextField.text = selectedHospitalModel.hospitalName;
    self.departmentTextField.text = selectedHospitalModel.hospitalFacultyName;
  }
}
```
重写这个方法，就不需要额外提供一个方法来更新数据显示了。我们通过这种写法，由于下一个界面在选择以后，在`block`中返回了所选择的模型类，因此我们只需要调用`self. selectedHospitalModel = selectedModel;`就可以了，因此这个方法已经重写了而且也自动更新数据显示了。

#接口API


不要直接在控制器中调用`AFNetworking`的方法，将方法统一封装到一个网络层管理类。每个接口应该配上详细的说明，包括功能说明，参数说明，最好将接口文档对应于这个接口的链接添加上。如下图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/屏幕快照-2015-11-19-上午9.40.14.png)

维护的人只需要点击链接就可以去查找接口详细的使用教程。

>提示：如果不添加链接，应该将参数`param`有哪些字段，及其意思标明，维护的人方能直接看懂。


