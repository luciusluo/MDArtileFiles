#前言

今天在处理进制转换时，突然看到iOS里面的一个很神奇的函数：strtoul，这可以进行进制的转换。

本篇文章的主题就是讲讲这个strtoul神奇的函数，如何帮助我们快速进行任意进制转换。

#strtoul

函数声明如下:

```
unsigned long strtoul(const char *, char **, int);
```

从这个函数名称，大概可以猜出来这个函数的意思是字符串转换成无符号长整型。如何看出来，这么看：str-to-ul，再补全就是string-to-unsign long。

这是C语言函数，参数说明如下：

* 参数一：const char *类型，表示字符串的起始地址
* 参数二：表示字符串有效数字的结束地址，传0或者NULL表示不接收
* 参数三：转换基数。当base=0,自动判断字符串的类型，并按10进制输出，例如"0xa",就会把字符串当做16进制处理，输出的为10

#例子

```
// s是15
unsigned long s =  strtoul([[@"F" substringWithRange:NSMakeRange(0, 1)] UTF8String], 0, 16);
```

#最后

经验+1，记录下来，下次可以直接拿过来用哦！