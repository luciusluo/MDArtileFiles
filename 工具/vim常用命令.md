#前言

vim在开发中是必备神器，不管是做服务端开发、前端开发，那都是必备神器。如果没使用过vim，那不用继续往下看了！

我主要是做iOS开发，但是我的Xcode一定要安装XVim插件，因为它能让我在Xcode中使用vim的命令，开发效率明显提高。

其实，不只是在开发中使用vim，在使用linux相关操作时，也经常需要使用vim命令的。

写下此篇，目的只是为了给自己整理和记录相关命令笔记。高手请飘过！

#常用命令

**复制区块：**

在visual模式下，将光标定位到某行，复制一部分内容，然后ctrl+v就会出现VISUAL BLOCK，然后下/右移动将某个范围都选择。

**对字符操作：**

x, X                 在一行字当中，x为向后删除一个字符 (相当亍 [del] 按键)，X为向前删除一个字符(相当亍 [backspace] 亦即是退格键) (常用)

nx                   n 为数字，连续向后删除 n 个字符。丼例来说，我要连续删除 10 个字符， 『10x』。

d$                   删除游标所在处，到该行行尾的所有字符

d0                   删除游标所在处 ，到该行行首的所有字符
 对行操作：

dd                    删除游标所在的那一整列(常用)

ndd                  n 为数字。删除光标所在的行向下n行，例如 20dd 则是删除 20行 (常用)

d1G                 删除光标所在到第一行的所有数据

dG                   删除光标所在到最后一行的所有数据

**复制相关：**

yy                 复制游标所在癿那一行(常用)

y1G               复制光标所在列到第一列癿所有数据

yG                复制光标所在列到最后一列癿所有数据

y0                复制光标所在癿那个字符到该行行首癿所有数据

y$                复制光标所在癿那个字符到该行行尾癿所有数据


**替换相关：**

r   取代光标处的字符（替换单个）
R   进入替换模式（替换多个）
cc  与dd相同，相当于删除行
S   与dd相同，相当于删除行
cw  相当于删除一个单词
C   相当于删除当前光标处到行尾的字符
c0  相当于删除当前光标处到行首的字符
c^  与c0相同

**批量替换：**

```
// 替换所有行中的所有replaced_source_pattern为target_pattern
:%s/replaced_source_pattern/target_pattern/g

// 替换当前行第一个replaced_source_pattern为target_pattern
:s/replaced_source_pattern/target_pattern/

// 替换当前行所有replaced_source_pattern为target_pattern
:s/replaced_source_pattern/sky/g 

// 替换第n行开始到最后一行中每一行的第一个replaced_source_pattern为target_pattern
:n,$s/replaced_source_pattern/target_pattern/

// 替换第n行开始到最后一行中每一行所有replaced_source_pattern为target_pattern
:n,$s/replaced_source_pattern/target_pattern/g 

// 替换每一行的第一个replaced_source_pattern为target_pattern
// 相当于:g/replaced_source_pattern/s//target_pattern/
:%s/replaced_source_pattern/target_pattern/
```

>n 为数字，若n为.，表示从当前行开始到最后一行

但是，如果我们的文本中就有/怎么办？我们可以使用#来作为分割符的。
可以使用#作为分隔符，此时中间出现的/不会作为分隔符，只是作为普通文本处理：

```
:s#hyb#huangyibiao
```

将当前行的hyb替换成huangyibiao

例如，要将所有的test替换成test1，:%s/test/test1/g

#持续更新

今天收集到自己常用的命令就写这么多了，后续用到再收集记录下来

#关注我


**Swift/ObjC技术群一：[324400294(已满)]()**

**Swift/ObjC技术群二：[494669518]()**

**ObjC/Swift高级群：[461252383（注明年限，新手勿扰）]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**原文有更新，请阅读原文：**[http://www.henishuo.com/vim-xcode/](http://www.henishuo.com/vim-xcode/)


#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)

