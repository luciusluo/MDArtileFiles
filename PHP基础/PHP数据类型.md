#前言

PHP支持8种原始数据类型。

1、四种标量类型：

* boolean（布尔型）
* integer（整型）
* float（浮点型，也称作 double)
* string（字符串）

2、两种复合类型：

* array（数组）
* object（对象）

3、最后是两种特殊类型：

* resource（资源）
* NULL（无类型）

实际上 double 和 float 是相同的，由于一些历史的原因，这两个名称同时存在。变量的类型通常不是由程序员设定的，确切地说，是由 PHP 根据该变量使用的上下文在运行时决定的。

#Boolean类型

这是最简单的类型。boolean表达了真假值，可以为TRUE或FALSE。两个都不区分大小写。

```
<?php

$action = true;

// == 是一个操作符，它检测两个变量是否相等，并返回一个布尔值
if ($action == "show_version") {
    echo "The version is 1.23";
}

// 这样做是不必要的
if ($show_separators == TRUE) {
    echo "<hr>\n";
}

// 因为可以使用下面这种简单的方式：
if ($show_separators) {
    echo "<hr>\n";
}
?>
```

当转换为boolean时，以下值被认为是FALSE：

* 布尔值 FALSE 本身
* 整型值 0（零）
* 浮点型值 0.0（零）
* 空字符串，以及字符串 "0"
* 不包括任何元素的数组
* 不包括任何成员变量的对象（仅 PHP 4.0 适用）
* 特殊类型 NULL（包括尚未赋值的变量）
* 从空标记生成的 SimpleXML 对象

如下：

```
<?php

var_dump((bool) "");        // bool(false)
var_dump((bool) 1);         // bool(true)
var_dump((bool) -2);        // bool(true)
var_dump((bool) "foo");     // bool(true)
var_dump((bool) 2.3e5);     // bool(true)
var_dump((bool) array(12)); // bool(true)
var_dump((bool) array());   // bool(false)
var_dump((bool) "false");   // bool(true)

?>
```

#Integer整型

整型数的字长和平台有关，尽管通常最大值是大约二十亿（32 位有符号）。64 位平台下的最大值通常是大约 9E18。PHP 不支持无符号整数。Integer 值的字长可以用常量 PHP_INT_SIZE来表示，自 PHP 4.4.0 和 PHP 5.0.5后，最大值可以用常量 PHP_INT_MAX 来表示。

```
<?php
$a = 1234; // 十进制数
$a = -123; // 负数
$a = 0123; // 八进制数 (等于十进制 83)
$a = 0x1A; // 十六进制数 (等于十进制 26)
?>
```

PHP 中没有整除的运算符。1/2 产生出 float 0.5。值可以舍弃小数部分强制转换为 integer，或者使用 round() 函数可以更好地进行四舍五入。

#Float浮点型

浮点型（也叫浮点数 float，双精度数 double 或实数 real）可以用以下任一语法定义：

```
<?php
$a = 1.234; 
$b = 1.2e3; 
$c = 7E-10;
?>
```

**比较浮点数**：

```
<?php
$a = 1.23456789;
$b = 1.23456780;
$epsilon = 0.00001;

if(abs($a-$b) < $epsilon) {
    echo "true";
}
?>
```

某些数学运算会产生一个由常量 NAN 所代表的结果。此结果代表着一个在浮点数运算中未定义或不可表述的值。任何拿此值与其它任何值进行的松散或严格比较的结果都是 FALSE。

由于 NAN 代表着任何不同值，不应拿 NAN 去和其它值进行比较，包括其自身，应该用 is_nan() 来检查。

#String 字符串

一个字符串 string 就是由一系列的字符组成，其中每个字符等同于一个字节。这意味着 PHP只能支持256的字符集，因此不支持Unicode。

>string 最大可以达到 2GB。


**单引号：**

```
<?php
	echo 'this is a simple string';
?>
```

**双引号：**

```
<?php
	echo 'this is a simple \\n string';
?>
```

**Heredoc：**

```
$str = <<<EOT
<p>This is a test</p>
EOT;

echo $str;
```

**Nowdoc：**

```
$str1 = <<<EOD
Example of string
spanning multiple lines
using nowdoc syntax.
EOD;

echo $str1;
```

使用花括号{}来标识变量：

```
$str = "This is my test";
// 其实不加{}有时候也没有关系，解析器大多时候可以识别的，不过最好加上
echo "string {$str} ";
```

使用dot（.）来连接字符串：

```
echo "<br>".$str1."str".$str;
```

字符串操作：

```
<?php
// 取得字符串的第一个字符
$str = 'This is a test.';
$first = $str[0];

// 取得字符串的第三个字符
$third = $str[2];

// 取得字符串的最后一个字符
$str = 'This is still a test.';
$last = $str[strlen($str)-1]; 

// 修改字符串的最后一个字符
$str = 'Look at the sea';
$str[strlen($str)-1] = 'e';

?>
```

#Array数组

PHP中的数组实际上是一个有序映射。映射是一种把 values 关联到 keys 的类型。此类型在很多方面做了优化，因此可以把它当成真正的数组，或列表（向量），散列表（是映射的一种实现），字典，集合，栈，队列以及更多可能性。由于数组元素的值也可以是另一个数组，树形结构和多维数组也是允许的。

```
<?php
$array = array(
    "foo" => "bar",
    "bar" => "foo",
);

// 自 PHP 5.4 起
$array = [
    "foo" => "bar",
    "bar" => "foo",
];
?>
```

没有指定key时，默认为整型：

```
<?php
$array = array("foo", "bar", "hallo", "world");
var_dump($array);
?>
```

输出：

```
array(4) {
  [0]=>
  string(3) "foo"
  [1]=>
  string(3) "bar"
  [2]=>
  string(5) "hallo"
  [3]=>
  string(5) "world"
}
```

#Object对象

要创建一个新的对象，使用 new 语句实例化一个类：

```
<?php
class Foo
{
   function do_foo() 
   {
   	 echo "do_foo";
   }	
}

$foo = new Foo();
$foo->do_foo();?>
```

#NULL

特殊的 NULL 值表示一个变量没有值。NULL 类型唯一可能的值就是 NULL。

在下列情况下一个变量被认为是 NULL：

* 被赋值为 NULL。
* 尚未被赋值。
* 被unset()。

在判断是否为NULL，使用is_null判断；要设置为NULL，使用unset函数。

#写在最后

本文只是笔者学习PHP的笔记！当然也欢迎高手留言，求指导速成方法。倘若大家觉得对自己也有用，欢迎大家一起来学习PHP，一起讨论PHP语法知识，一起练习PHP。


#关注我

**PHP学习交流群：536643087**

**原文有更新，请阅读原文：**[http://www.henishuo.com/php-data-struct/](http://www.henishuo.com/php-data-struct/)

