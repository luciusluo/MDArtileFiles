#概述

剑指offer[合并两个有序的链表](http://www.nowcoder.com/practice/d8b6b4358f774294a89de2a6ac4d9337?tpId=13&tqId=11169&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)，通过递归与非递归来实现！算法写法可能有很多种，但是算法的核心思想是差不多的！

#题目描述

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

#要求

时间限制：1秒  
空间限制：32768k

#C++非递归做法

非递归的做法思想是：先判断两个链表头，以决定哪个链表头作为新链表的链表头指针；然后同时遍历两个链表，直到其中一个链表尾，比较时，值小的接入到新的链表中，同时向后移动该链表的头指针和新链表的指针；当遍历完毕后，哪个链表不为空，就直接将剩余部分接入到新链表尾即可。

```
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
        if (pHead1 == NULL) {
            return pHead2;
        }
        
        if (pHead2 == NULL) {
            return pHead1;
        }
        
        // 决定哪个头指针作为新链表的头指针
        ListNode *h = NULL;
        ListNode *p = NULL;
        if (pHead1->val > pHead2->val) {
            h = pHead2;
            pHead2 = pHead2->next;
        } else {
            h = pHead1;
            pHead1 = pHead1->next;
        }
        
        p = h;
        // 遍历两个链表
        while (pHead1 && pHead2) {
            // 小的接到新链表后面，大的不动
            if (pHead1->val > pHead2->val) {
                p->next = pHead2;
                pHead2 = pHead2->next;
            } else {
                p->next = pHead1;
                pHead1 = pHead1->next;
            }
            
            p = p->next;
        }
        
        if (pHead1) {
            p->next = pHead1;
        } else {
            p->next = pHead2;
        }
        
        return h;
    }
};
```

#C++递归做法

递归做法，其实就是每次判断哪个值小，值小的作为头，然后递归构造next部分。

```
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
        if (pHead1 == NULL) {
            return pHead2;
        }
        
        if (pHead2 == NULL) {
            return pHead1;
        }
       
        ListNode *mergeHead = NULL;
        if (pHead1->val > pHead2->val) {
            mergeHead = pHead2;
            mergeHead->next = Merge(pHead1, pHead2->next);
        } else {
            mergeHead = pHead1;
            mergeHead->next = Merge(pHead2, pHead1->next);
        }
        
        return mergeHead;
    }
};
```

#小结 

面试挺爱都查链表操作的，虽然我在iOS开发这么多的项目中，从来没有应用过链表操作。不过算法的思想可以帮助我们解决问题，做事情都提前先思考分析清楚，这是一种好的开发习惯！