#概述

让我们一直来刷算法题吧，本篇对应于剑指offser第七题[跳台阶](http://www.nowcoder.com/practice/8c82a5b80378478f9484d87d1c5f12a4?tpId=13&tqId=11161&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)！正在找工作的，或者准备要找工作的，都好好刷一刷吧，对自己的思维有很大的帮助，对找工作更是好处多多啦！

#题目简述

一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

#要求

时间限制：1s  
空间限制：32768K

#分析

题目中明确提示一次只可以跳1级或者2级台阶，假设一次跳1级，则后面的n-1级台阶可以有f(n-1)种跳法；假设一次跳2级，则后面的n-2级台阶可以有f(n-2)种跳法。那么总跳法有f(n) = f(n-1) + f(n-2)种跳法。当n=1时，只有一阶，那么跳法只有一种。当n=2时，f(2) = 2。

```
        | 1, (n=1)
f(n) =  | 2, (n=2)
        | f(n-1)+f(n-2) ,(n>2,n为整数)
```

其实跟斐波那契数列一样的！

#C++

```
class Solution {
public:
	 // 递归解法
    int jumpFloor(int number) {
        if (number == 1) {
            return 1;
        }
        
        if (number == 2) {
            return 2;
        }
        
        return jumpFloor(number - 1) + jumpFloor(number - 2);
    }
};
```

或者采用非递归做法：

```
class Solution {
public:
    int jumpFloor(int number) {
        if (number == 1) {
            return 1;
        }
        
        if (number == 2) {
            return 2;
        }
        
        int a = 1;
        int b = 2;
        int sum = 0;
        for (int i = 3; i <= number; ++i) {
            sum = a + b;
            a = b;
            b = sum;
        }
        
        return sum;
    }
};
```

#变态跳台阶

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

再试试再变态一点点的，扩展一下。一次可以1~n级，那么：

* f(1) = 1
* f(2) = 2
* f(3) = f(1) + f(2) = 3
* f(4) = f(1) + f(2) + f(3) = 2 * f(3) = 2 * 3 = 6
* f(5) = f(1) + f(2) + f(3) + f(4) = 2 * f(4) = 2 * 6 = 12

总跳法可以有：

```
       | 1 (n = 1)
f(n) = | 2 (n = 2)
       | 2 * f(n-1)
```

#C++

```
class Solution {
public:
    int jumpFloorII(int number) {
        if (number == 1) {
            return 1;
        }
        
        if (number == 2) {
            return 2;
        }
        
        return 2 * jumpFloorII(number - 1);
    }
};
```

#小结

数学分析是搞算法的基本功，其实刷算法题跟高考考试差不多，都是需要认真分析题目。所以，那些清华出身的计算机科班的尖子生出来搞算法，自然在我这种普通大学出身的要强！逻辑思维比不上人家！

