#概述

让我们一直来刷算法题吧，本篇对应于剑指offser第一题[二维数组中的查找](http://www.nowcoder.com/practice/abc3fe2ce8e146608e868a70efebf62e?tpId=13&tqId=11154&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)！正在找工作的，或者准备要找工作的，都好好刷一刷吧，对自己的思维有很大的帮助，对找工作更是好处多多啦！

#题目

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

输入描述:

```
array： 待查找的二维数组
target：查找的数字
```

输出描述:

```
查找到返回true，查找不到返回false
```

#要求

时间限制：1s  
空间限制：32768k

#分析

由题目可以分析出来，二维数据是有序的，同一行是递增的，同一列也是递增的，因此我们可以考虑从每行的最后一个与待查找元素比较，当该行最后一个元素之待查找元素相待， 则return true；当比待查找元素大时，表示要么在这一行，要么不存在。

#C++

```
bool Find(vector< vector<int> > array, int target) {
    for (int i = 0; i < array.size(); ++i) {
        vector<int> item = array[i];
         
        int last = 0;
        if (item.size() >= 1) {
           last = item[item.size() - 1];
        } else {
            continue;
        }
         
        // 直接命中
        if (last == target) {
            return true;
        }
        // 要么不存在，要么就在这一行
        else if (last > target) {
            int low = 0;
            int high = item.size() - 1;
            // 折半查找
            while (low <= high) {
                int mid = (low + high) / 2;
                if (item.at(mid) == target) {
                    return true;
                } else if (item.at(mid) > target) {
                    high = mid - 1;
                } else {
                    low = mid + 1;
                }
            }
        }
    }
     
    return false;
}
```

折半查找时间复杂度为：O ( log2^^n )，与行宽有关。因此，最终算法时间复杂度为：O (m log2 ^^n )，其中m表示列宽，n表示行宽。

#小结

这里没有直接使用双重循环遍历，如果采用双重循环遍历，应该从时间上是过不去的才对。这里单重遍历查找所在的行，然后对所在的行再折半查找，效率要提高了很多。