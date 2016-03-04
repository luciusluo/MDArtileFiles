>#修改UITabbarItem字体样式

---

#`Objective-C`版本
如果是使用`ObjectiveC`来开发，那么可以这么全局修改：

```
NSDictionary *attributes = @{NSFontAttributeName:[UIFont systemFontOfSize:0.5]};
    [[UIBarButtonItem appearance] setTitleTextAttributes:attributes
                                                forState:UIControlStateNormal];
```

如果是修改单个`UIBarButtonItem`对象，可以直接设置：

```
[item setTitleTextAttributes:attributes forState:UIControlStateNormal];
[item setTitleTextAttributes:attributesHighlighted forState:UIControlStateSelected];

```

其中`item`是`UIBarButtonitem`对象。

#`Swift`版本

对于`Swift`版本，其实也是一样的，这种方式是全局的设置方法：

```
// 这里样式要改成什么样，自己需要的
let normalAttributes = [NSFontAttributeName: UIFont.systemFontOfSize(15)]
let highlightedAttributes = [NSFontAttributeName: UIFont.systemFontOfSize(15)];
UIBarButtonItem.appearance().setTitleTextAttributes(normalAttributes, forState: .Normal)
UIBarButtonItem.appearance().setTitleTextAttributes(highlightedAttributes, forState: .Normal)
```

如果只是想对单个设置样式，可以这样：

```
let normalAttributes = [NSFontAttributeName: UIFont.systemFontOfSize(15)]
let highlightedAttributes = [NSFontAttributeName: UIFont.systemFontOfSize(15)];

item.setTitleTextAttributes(normalAttributes, forState: .Normal)
item.setTitleTextAttributes(highlightedAttributes, forState: .Normal)

```

其中，`item`是`UIBarButtonItem`对象。

>#Enjoy it!