#前言

PHP中的变量用一个美元符号后面跟变量名来表示。变量名是区分大小写的。

变量名与PHP中其它的标签一样遵循相同的规则。一个有效的变量名由字母或者下划线开头，后面跟上任意数量的字母，数字，或者下划线。按照正常的正则表达式，它将被表述为：'[a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*'。

$this是一个特殊的变量，它不能被赋值。

#基础

以$开头，跟着字母或者下划线，后面可以是数字，甚至是中文。不过要按照命名规范，使用驼峰命名.

```
<?php
$var = 'Bob';
$Var = 'Joe'; // 区分大小写
echo "$var, $Var";      // 输出 "Bob, Joe"

$4site = 'not yet';     // 非法变量名；以数字开头
$_4site = 'not yet';    // 合法变量名；以下划线开头
$i站点is = 'mansikka';  // 合法变量名；可以用中文
?>

```

使用引用赋值，简单地将一个 & 符号加到将要赋值的变量前（源变量）。例如，下列代码片断将输出“My name is Bob”两次：

```
<?php
$foo = 'Bob';              // 将 'Bob' 赋给 $foo
$bar = &$foo;              // 通过 $bar 引用 $foo
$bar = "My name is $bar";  // 修改 $bar 变量
echo $bar;
echo $foo;                 // $foo 的值也被修改

$vbar = &(24 * 7);  // 非法; 引用没有名字的表达式
?>
```

默认值：

```
// 未赋值和无上下文声明的变量，默认值为NULL
var_dump($unset_var);

// 可根据上下文
echo($unset_bool ? "true\n" : "false\n");

// String usage; outputs 'string(3) "abc"'
$unset_str .= 'abc';
var_dump($unset_str);

// Integer usage; outputs 'int(25)'
$unset_int += 25; // 0 + 25 => 25
var_dump($unset_int);

// Float/double usage; outputs 'float(1.25)'
$unset_float += 1.25;
var_dump($unset_float);

// Array usage; outputs array(1) {  [3]=>  string(3) "def" }
$unset_arr[3] = "def"; // array() + array(3 => "def") => array(3 => "def")
var_dump($unset_arr);

// Object usage; creates new stdClass object (see http://www.php.net/manual/en/reserved.classes.php)
// Outputs: object(stdClass)#1 (1) {  ["foo"]=>  string(3) "bar" }
$unset_obj->foo = 'bar';
var_dump($unset_obj);
```

判断是否有值：

```
<?php
print isset($a); // $a is not set. Prints false. (Or more accurately prints ''.)
$b = 0; // isset($b) returns true (or more accurately '1')
$c = array(); // isset($c) returns true
$b = null; // Now isset($b) returns false;
unset($c); // Now isset($c) returns false;
?>
```

#预定义变量

```
$_GET      // 所有GET请求参数
$_POST     // 所有POST请求参数
$_ENV      // 环境变量
$_SERVER  
$_COOKIE   // cookie
$_REQUEST  // 所有请求参数
$_SESSION  // session值
```

#写在最后

本文只是笔者学习PHP的笔记！当然也欢迎高手留言，求指导速成方法。倘若大家觉得对自己也有用，欢迎大家一起来学习PHP，一起讨论PHP语法知识，一起练习PHP。


#关注我

**PHP学习交流群：536643087**


