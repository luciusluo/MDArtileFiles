#前言

神奇的JQuery怎么设置checkbox状态时好时坏？明明同一行代码，断点跟踪确实执行了，但是有时候好使，有时候却没有生效。毕竟对JS不是很熟悉，只是通过JS来处理前端HTML的标签的状态设置时，通过JQuery有时候会更方便些的，但是发现更不好办。

今天尝试实现checkbox全选、全不选功能，与App开发中的效果是一样的，勾选全选则将所有的选项都选中；同样取消勾选某个子项也将全选设置为非选中状态；所有子选项都为选中状态时，将全选设置为选中状态。

由于对JS不是很熟悉，于是尝试各种百度、google，发现出来的文章都是坑爹啊。各种JQuery的，但是为什么我设置了就是没有作用的。起初以为是变量获取不到，于是断点跟踪，对象是取到了的，但是设置JQuery的方法来设置就是没有作用。

#搜到的处理方式

这里的checkbox的id为cbxSelectAll，于是尝试这么写：

```
$('#cbxSelectAll').attr('checked', true);
```

结果是无效的。再尝试修改为：

```
$('#cbxSelectAll').attr('checked', 'checked');
```

结果是第一次设置生效了，再设置就没有生效。坑爹，这到底是什么东西，怎么时好时坏呢？

然后在设置为false时，这么写：

```
$('#cbxSelectAll').attr('checked', false);
// 也没有作用
//$('#cbxSelectAll').attr('checked', '');
```

果然是都没有作用。但是通过下面的设置，可以取消选中：

```
$('#cbxSelectAll').removeAttr('checked');
```

难道是年代久远，这些方法已经不再有效了吗？

#最后解决办法

最后的解决办法还是放弃了JQuery，改用Javascript原生的Dom来设置。

下面是设置为全选或者取消全选状态的代码：

```
var checkboxes = document.getElementsByName('selectIds');

var selectedCount = 0;
var unSelectCount = 0;

for (var i = 0; i < checkboxes.length; i++) {
  var checkbox = checkboxes[i];
  
  if(checkbox.tagName == "INPUT" && checkbox.checked){
	selectedCount++;
  } else if (checkbox.tagName == "INPUT" && checkbox.checked == false) {
  	unSelectCount++;
  }
}

if (selectedCount == checkboxes.length) {
	document.getElementById('cbxSelectAll').checked = true;
} else if (unSelectCount != checkboxes.length) {
	document.getElementById('cbxSelectAll').checked = false;
}
```

#JQuery获取状态

JQuery通过checkbox的is函数来获取状态：

```
var isChecked = $('#cbxSelectAll').is(':checked');
```

之前尝试过使用attr函数来获取，但是获取的值显示为null：

```
// 显示为null，好生奇怪
var isChecked = $('#cbxSelectAll').attr('checked');
```

当然，我们也可以直接使用Javascript原生的Dom方式来获取，肯定是没有问题的：

```
var isChecked = document.getElementById('cbxSelectAll').checked;
```

#小结

玩前端JQuery果然要比玩原生的JavaScript要吃力些，虽然有很多时候可以使代码更方便书写。不过还是两者结合来做吧。

此篇文章只适合菜鸟参考，请不要YY就好了！