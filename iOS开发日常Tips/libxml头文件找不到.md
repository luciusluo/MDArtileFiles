#概述

关于使用到libxml2.tbd这个库时，要注意添加搜索头文件路径。写下此篇，希望遇到的人儿能够少走弯路！


#报错信息

当出现以下任何一种报错信息时，请记得添加相应的库及搜索路径：

* **libxml/parser.h**: No such file or directory
* **libxml/tree.h**: No such file or directory
* **libxml/catalog.h**: No such file or directory

其实都是使用了libxml2.tbd库。

#解决

**第一步：**  
检查是否已添加libxml2.tbd库，如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/05/QQ20160505-0@2x.png)

**第二步：**  

检查头文件路径是否已添加或者已否正确：

![image](http://www.henishuo.com/wp-content/uploads/2016/05/QQ20160505-1@2x.png)

#小结

问题虽然简单，但是这是相对的！希望能够帮助新手们，少走一些弯路！


