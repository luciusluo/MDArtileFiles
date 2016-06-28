#概述

让我们一直来刷算法题吧，本篇对应于剑指offser第四题[重建二叉树](http://www.nowcoder.com/practice/8a19cbe657394eeaac2f6ea9b0f6fcf6?tpId=13&tqId=11157&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)！正在找工作的，或者准备要找工作的，都好好刷一刷吧，对自己的思维有很大的帮助，对找工作更是好处多多啦！

#题目描述

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

例如：输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

#要求

时间限制：1秒  
空间限制：32768K

#算法分析

首先，我们需要明白基本概念。先序遍历二叉树，先访问根，再访问左孩子，最后才访问右孩子；后序遍历二叉树，先访问左孩子，再访问根，最后才访问右孩子。

算法的思想是：

先建立根，再建立左子树，最后建立右子树。每次递归，先序数组中的第一个元素一定是这次构建子树的根，然后在中序数组中查找根的位置，将中序数组中以根的中心，分成左、右两个中序数组，用于建立左、右子树；同样，在先序数组中，从第二个元素起，到中序中的根的索引处，作为先序左数组，用于构造左子树，而根索引处到最后，则是先序遍历中的右数组，用于构造右子树。

最后，每次递归都得到子树的根节点，同时又以根结点分别构造出先序左、右子树的数组和中序左、右子树的数组，就可以一直递归到所有子树建立完毕。

#C++

```
/**
 * Definition for binary tree
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    struct TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> in) {
        if (pre.size() == 0 || pre.size() != in.size()) {
            return NULL;
        }
        
        vector<int> pre_left;
        vector<int> pre_right;
        vector<int> in_left;
        vector<int> in_right;
        
        TreeNode *root = new TreeNode(pre.at(0));
        
        // 查找根在中序数组中的位置
        int rootIndex = 0;
        for (int i = 0; i < in.size(); ++i) {
            if (in.at(i) == pre.at(0)) {
                rootIndex = i;
                break;
            }
        }
        
        // 左子树部分
        for (int i = 0; i < rootIndex; ++i) {
            in_left.push_back(in.at(i));
            
            if (i + 1 < pre.size()) {
                pre_left.push_back(pre.at(i + 1));
            }
        }
        
        // 右子树部分
        for (int i = rootIndex + 1; i < in.size(); ++i) {
            in_right.push_back(in.at(i));
            pre_right.push_back(pre.at(i));
        }
        
        root->left = reConstructBinaryTree(pre_left, in_left);
        root->right = reConstructBinaryTree(pre_right, in_right);
        
        return root;
    }
};
```

#小结 

这道题若没有参考他人的代码，我也没能及时写出来。首先太久不看树方面的知识了，都快记不住了，所以首先百度了一下树的相关知识，然后再回头看看以前写过的树相关的遍历算法。在理解了先序、中序、后序遍历的概念后，再配合他人写的代码，就可以理解这道题如何实现了！