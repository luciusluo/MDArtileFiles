#概述

让我们一直来刷算法题吧，本篇对应于剑指offser第二题[替换空格](http://www.nowcoder.com/practice/4060ac7e3e404ad1a894ef3e17650423?tpId=13&tqId=11155&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)！正在找工作的，或者准备要找工作的，都好好刷一刷吧，对自己的思维有很大的帮助，对找工作更是好处多多啦！

#题目描述

请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

#要求

时间限制：1秒   
空间限制：32768K

#C/C++

```
//length为牛客系统规定字符串输出的最大长度，固定为一个常数
void replaceSpace(char *str,int length) {
    char *temp = (char *)malloc(sizeof(char) * length);
    
    int k = 0;
    for (int i = 0; i < strlen(str); ++i) {
        char ch = str[i];
        if (ch == ' ') {
            temp[k++] = '%';
            temp[k++] = '2';
            temp[k++] = '0';
        } else {
            temp[k++] = ch;
        }
    }
    
    temp[k] = '\0';
    for (int i = 0; i <= k; ++i) {
        str[i] = temp[i];
    }
    
    free(temp);
}
```

时间复杂度：O (n)

#分析

上面的代码中，采用了空间换时间的办法，利用临时数组来记录，时间复杂度的O ( n )。如果我们不想用空间换时间，可以在原有字符串上操作，但是每次都需要移动位置以存放%20，时间复杂度变成了O  ( n^^2 )

#小结

对于刷题，都要注意要求中的时间和空间限制，搞ACM题是必须认真思考的，虽然牛客网这里并没有要求那么严格，数据的长度并不算大，所以可以通过。

训练这些算法题，可以让你在写代码时，都会考虑上限与下限，最大长度可以达到多长。这段时间的训练目标是刷完剑指offer100题！

