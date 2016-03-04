#前言

PHP脚本可放置于文档中的任何位置。本篇文章主要包括以下内容：

* PHP标记
* 从HTML中分离
* 指令分隔符
* 注释

#PHP标记

PHP脚本以<?php 开头，以 ?> 结尾：

```
<?php 
   echo "Hello world!";
?>
```

当解析一个文件时，PHP会寻找起始和结束标记，也就是 <?php 和 ?>，这告诉PHP开始和停止解析二者之间的代码。此种解析方式使得PHP可以被嵌入到各种不同的文档中去，而任何起始和结束标记之外的部分都会被PHP解析器忽略。

PHP也允许使用短标记 <? 和 ?>，**但不鼓励使用**。只有通过激活php.ini中的 short_open_tag配置指令或者在编译PHP时使用了配置选项 --enable-short-tags 时才能使用短标记。

如果文件内容是**纯PHP**代码，最好在**文件末尾删除PHP结束标记**。这可以避免在 PHP结束标记之后万一意外加入了空格或者换行符，会导致PHP开始输出这些空白，而脚本中此时并无输出的意图。

例如，下面的是在一个文件中的，这个文件只有PHP代码，此时建义不要加上?>结束标识：

```
<?php  // Html safe containers 

echo "Hello world<br>";
echo "test<br>";
echo "www.henishuo.com"; 
```

#从HTML中分离

凡是在一对开始和结束标记之外的内容都会被PHP解析器忽略，这使得PHP文件可以具备混合内容。可以使PHP嵌入到HTML文档中去，如下例所示：

```
<p>This is going to be ignored by PHP and displayed by the browser.</p>

<?php echo 'While this is going to be parsed.'; ?>

<p>This will also be ignored by PHP and displayed by the browser.</p>
```

条件分离：

```
<?php $expression = true ?>

<?php if ($expression == true) {
	echo "expression is true";
} ?>

<?php if ($expression == true): ?>
  This will show if the expression is true.
<?php else: ?>
  Otherwise this will show.
<?php endif; ?>
```

#指令分隔符

同C语言一样，PHP需要在每个语句后用分号结束指令。一段PHP代码中的结束标记隐含表示了一个分号；在一个PHP代码段中的最后一行可以不用分号结束。如果后面还有新行，则代码段的结束标记包含了行结束。

```
<?php
    echo "This is a test";
?>

<?php echo "This is a test" ?>

<?php echo 'We omitted the last closing tag';?>
```

#注释

PHP支持C、C++、Unix Shell风格（Perl 风格）的注释。例如:

```
<?php
    echo "This is a test"; // This is a one-line c++ style comment
    
    /* This is a multi line comment
       yet another line of comment 
     */   
    echo "This is yet another test";
    
    echo 'One Final Test'; # This is a one-line shell-style comment
?>
```

可以使用`//`或者`#`注释单行，可以使用`/**/`注释段。

#写在最后

本文只是笔者学习PHP的笔记！当然也欢迎高手留言，求指导速成方法。倘若大家觉得对自己也有用，欢迎大家一起来学习PHP，一起讨论PHP语法知识，一起练习PHP。


#关注我

**PHP学习交流群：536643087**

**原文有更新，请阅读原文：**[http://www.henishuo.com/php-base-programmar/](http://www.henishuo.com/php-base-programmar/)



