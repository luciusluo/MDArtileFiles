#序言

以下所有题目均来源于网络，收集于此并提供参考答案，不代表绝对正确，但具备参考价值。如果文章中出现任何错误或者疑问，请在评论中指出或者直接联系笔者，笔者将会在第一时间修正。

C、C++的基础是非常重要的，面试题中也经常有很多的C/C++题，因此很有重要加固C/C++方面的基础。

#1. 打印结果

```
main() { 
  int a[5]={1,2,3,4,5};
  int *ptr=(int *)(&a+1);
  
  printf("%d,%d",*(a+1),*(ptr-1));
}
```

**答案：2,5**

**笔者解析：<br>**

首先，你定义了一个名为 `a` 的整数数组，包含5个元素，并初始化为 `{1, 2, 3, 4, 5}`。

接着，你定义了一个指针 `ptr`，它被初始化为 `(&a+1)`。这里需要注意 `(&a+1)` 的操作。

`&a` 表示数组 `a` 的地址，即 `&a` 是指向整个数组的指针。
`(&a+1)` 表示对数组指针进行加1操作。这实际上不是访问数组的下一个元素，而是将指针移动到数组的末尾之后的一个位置。
最后，你将 `ptr` 强制类型转换为 `int*` 类型的指针。这是因为 `(&a+1)` 返回的是一个指向整数的指针，所以需要将其显式转换为 `int*`。


`a+1`是指针`+1`，就指向了数组`a`的第二个元素，而当于`ptr`当前所指向的值。所以，`*(a+1)`的值就是`2`。

`ptr-1`是二维指针`-1`操作，因此指向了数组`a`最后一个元素的地址，所以`*(ptr-1)`的值为数组第一个元素值`5`。

#2. 输出结果



```
void Func (char str[100]) {   
	sizeof(str ) = ? 
} 

void *p = malloc(100); sizeof(p) = ?
```

**答案：4**<br>

**笔者解析：<br>**

这道题很常见,`Func(char str[100])`函数中数组名作为函数形参时，在函数体内，数组名失去了本身的内涵，仅仅只是一个指针；在失去其内涵的同时，它还失去了其常量特性，可以作自增、自减等操作，可以被修改。`32`位平台下，指针的长度（占用内存的大小）为`4`字节，所以`sizeof(str)` 和`sizeof(p)` 都为`4`。

#3. 用预处理指令#define声明一个常数，用以表明1年中有多少秒（忽略闰年问题）



**答案：**

```
#define SECONDS_IN_A_YEAR (60 * 60 * 24 * 365)UL
```

**笔者解析：**

其实这道题主要是考查是否够细心，因此类型为`int`的话，会`overflow`的。因此这里在后面添加了`UL`，表示无符号长整形。`#define`不能以分号结束，而且这里还要注意括号。

#4. 写“标准”宏MIN ，这个宏输入两个参数并返回较小的一个


**答案：**

```
#define MIN(A,B) （（A） <= (B) ? (A) : (B))
```

**笔者解析：**

这里也是考查是否够细心的细节。一定要注意，宏只是单纯的展开，因此一定要注意展开后是否正确。所以无论何时，都加上括号以明确其意途是很好的。

这个操作符存在`C`语言中的原因是它使得编译器能产生比`if-then-else`更优化的代码，了解这个用法是很重要的。 懂得在宏中小心地把参数用括号括起来  我也用这个问题开始讨论宏的副作用，例如：当你写下面的代码时会发生什么事？ 

```
result = MIN(*p++, b);
```

展开后是这样的：

```
result = ((*p++) <= (b) ? (*p++) : (b))
```

这个表达式会产生副作用，指针`p`会作三次`++`自增操作，而这并不是我们希望的。

#5. 数组和指针的区别


这种题连大学考试都很爱考吧。

**参考答案：**

* 数组可以申请在**栈区和数据区**；指针可以指向任意类型的内存块
* `sizeof`作用于数组时，得到的是数组所占的内存大小；作用于指针时，得到的都是`4`个字节的大小
* 数组名表示数组首地址，是常量指针，不可修改指向。比如不可以将`＋＋`作用于数组名上；普通指针的值可以改变，比如可将`＋＋`作用于指针上
* 用字符串初始化字符数组是将字符串的内容拷贝到字符数组中；用字符串初始化字符指针是将字符串的首地址赋给指针，也就是指针指向了该数组

#6. static的作用


这个关键在实际开发中挺常用的。当我们使用实例成员变量不好处理时，我们将声明为静态变量，因此它有以下特性。

**参考答案：**

* 函数体内`static`变量的作用范围为该函数体，不同于`auto`变量，该变量的内存只被分配一次，因此其值在下次调用时仍维持上次的值
* 在模块内的`static`全局变量可以被模块内所用函数访问，但不能被模块外其它函数访问
* 在模块内的`static`函数只可被这一模块内的其它函数调用，这个函数的使用范围被限制在声明它的模块内
* 在类中的`static`成员变量属于整个类所拥有，对类的所有对象只有一份拷贝
* 在类中的`static`成员函数属于整个类所拥有，这个函数不接收`this`指针，因而只能访问类的`static`成员变量。 

#7. 简述内存分区情况


如果问到这种问题，通常是想考考到内存的理解程度。

**参考答案：**

* 代码区：存放函数二进制代码
* 数据区：系统运行时申请内存并初始化，系统退出时由系统释放，存放全局变量、静态变量、常量
* 堆区：通过`malloc`等函数或`new`等操作符动态申请得到，需程序员手动申请和释放
* 栈区：函数模块内申请，函数结束时由系统自动释放，存放局部变量、函数参数

#8. ＃include <filename>和＃include ”filename”有什么区别?


当年上大学时，确实大学考试也考过，没想到到了社会还是有可能考。

**参考答案：**

`＃include <filename>`直接在库文件目录中搜索所包含的文件而`＃include ”filename”`在当前目录下搜索所包含的文件，如果没有的话再到库文件目录搜索。

#9. 下面四个修饰指针有什么区别?


当年在大学时期，我对这几个变量也是经常搞混的。

```
const char *p;
char const *p;  
char * const p;  
const char * const p;
```

**参考答案：**

* `const char *p`定义了一个指向不可变的字符串的字符指针，可以这么看：`const char *`为类型，`p`是变量。
* `char const *p`与上一个是一样的。
* `char * const p`定义了一个指向字符串的指针，该指针值不可改变，即不可改变指向。这么看：`char *`是类型，`const`是修饰变量`p`，也就是说`p`是一个常量
* `const char * const p`定义了一个指向不可变的字符串的字符指针，且该指针也不可改变指向。这一个就很容易看出来了。两个`const`分别修饰，因此都是不可变的。

#10. 用过哪些排序算法？


回答时不必按课本背诵有哪些算法，就说说简单的且常用的就可以了。

**参考答案：**

冒泡排序、插入排序。在开发中最常用的就是冒泡排序和插入排序了，不用说那么多高深算法，在平常的工作中，若非BAT，也没有这么严格要求什么多高的效率。能掌握这两种最常用的就基本可以了，搞`App`开发，若非大数据，并没有什么太高的要求。

***冒泡排序：***
双重循环就可以实现，在工作中常应用于模型排序。

```
void bubbleSort(int[] unsorted)
{
    for (int i = 0; i < unsorted.Length; i++) 
    {
        for (int j = i; j < unsorted.Length; j++)
         {
            if (unsorted[i] > unsorted[j]) {
                int temp = unsorted[i];
                unsorted[i] = unsorted[j];
                unsorted[j] = temp;
            }
        }
    }
}
```

***插入排序：***

```
void insertionSort(int[] unsorted)
{
    for (int i = 1; i < unsorted.Length; i++)
    {
        if (unsorted[i - 1] > unsorted[i])
        {
            int temp = unsorted[i];
            int j = i;
            
            while (j > 0 && unsorted[j - 1] > temp)
            {
                unsorted[j] = unsorted[j - 1];
                j--;
            }
            unsorted[j] = temp;
        }
    }
}
```

下面也是插入排序算法，这种写法可能会更好看一些。上面用`while`循环来查找和移动位置，不好看明白。

```
void insertSort(int *a,int n)
{
    int i,j,key;
    
    // 控制需要插入的元素
    for (i = 1; i < n; i++)
    {
	     // key为要插入的元素
        key = a[i];
        // 查找要插入的位置,循环结束,则找到插入位置
        for (j = i; j > 0 && a[j-1] > key; j--) 
        {
	         // 移动元素的位置,供要插入元素使用
            a[j] = a[j-1]; 
        }
        
        // 插入需要插入的元素
        a[j] = key; 
    }
}
```

这里还有咱群里的一位小伙伴写的`Swift`版的快速排序算法，可以看看：[快速排序](http://blog.csdn.net/y550918116j/article/details/48719281)

#11．堆与栈的区别

上次我面了一位来面试`iOS`的研究毕业生，什么是栈？结果答成了堆。想想可知大学里都干什么去了。

**参考答案：**

栈的空间由操作系统自动分配/释放，堆上的空间手动分配/释放。
栈空间是有限的，而堆是很大的自由存储区。
            
`C`中的`malloc`函数分配的内存空间是在堆上的,`C++`中对应的是`new`操作符。
程序在编译期对变量和函数分配内存都在栈上进行,且程序运行过程中函数调用时参数的传递也在栈上。

#12. 链表

链表是非常爱问的。这里不写出如何创建链表，如何合并链表的代码了，只是会给出思路。

**如何合并两张有序的链表？**

回答时不用说得那么详细，一般来说，面试官也不会记得那么清楚，简单说到点上就可以了。既然两张表都是有序的，那么就可以先找到哪个作为主链。然后将副链每个元素按照顺序接入到主链上就可以了。大家可以百度找一些文章阅读，以了解更深一些。

