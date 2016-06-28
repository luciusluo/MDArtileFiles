#概述

让我们一直来刷算法题吧，本篇对应于剑指offser第三题[从尾到头打印链表](http://www.nowcoder.com/practice/d0267f7f55b3412ba93bd35cfa8e8035?tpId=13&tqId=11156&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)！正在找工作的，或者准备要找工作的，都好好刷一刷吧，对自己的思维有很大的帮助，对找工作更是好处多多啦！

#题目描述

输入一个链表，从尾到头打印链表每个节点的值。 

输入描述:

```
输入为链表的表头
```

输出描述:

```
输出为需要打印的“新链表”的表头
```

#要求

时间限制：1s  
空间限制：32768K

#C++

```
/**
*  struct ListNode {
*        int val;
*        struct ListNode *next;
*        ListNode(int x) :
*              val(x), next(NULL) {
*        }
*  };
*/
class Solution {
public:
    vector<int> printListFromTailToHead(struct ListNode* head) {
        vector<int> result;
        struct ListNode *node = head;
        
        while (node != NULL) {
            result.insert(result.begin(), node->val);
            node = node->next;
        }
        
        return result;
    }
};
```

#分析

其实这道题没有什么难点，只是要注意是从尾到头打印，因此需要逆序。还有就是考一下C++中的vector这个向量类的使用，由于是大学学习的，虽然那里玩得还算可以应付过去，但是隔这么多年，都不记得了，只是百度百科看一下里面的API才想起来！