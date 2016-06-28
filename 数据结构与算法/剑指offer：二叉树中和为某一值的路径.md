#概述

剑指offer[二叉树中和为某一值的路径](http://www.nowcoder.com/practice/b736e784e3e34731af99065031301bca?tpId=13&tqId=11177&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)这一道题主要还是考递归的解法。和为某一值的路径，因此我们必须要从根不断地遍历访问值并累加求和，若为叶子结点且累加之和等于指定的值，说明这是一条路径。

#题目描述

输入一颗二叉树和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。

#要求

时间限制：1秒  
空间限制：32768K

#分析

题目要求我们输入所有的路径，并且说明了路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。因为我们是从根往下的，因此与前序遍历一样，我们就按照前序遍历来访问结点。

#C++

```
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    vector<vector<int> > FindPath(TreeNode* root,int expectNumber) {
        vector<vector<int>> resultPaths;
        vector<int> path;
        int sum = 0;
        find(root, expectNumber, path, sum, resultPaths);
        
        return resultPaths;
    }
    
    // 前序遍历查找
    void find(TreeNode *root, int expectNumber, vector<int> &path, int sum, vector<vector<int>>&paths) {
        if (root == NULL) {
            return;
        }
        
        sum += root->val;
        path.push_back(root->val);
        // found
        if (root->left == NULL && root->right == NULL && expectNumber == sum) {
            paths.push_back(path);
            // 千万别忘了重置这个累加值
            sum = 0;
        }
        
        // 递归查找左子树
        if (root->left != NULL) {
            find(root->left, expectNumber, path, sum, paths);
        }
        
        // 递归查找右子树
        if (root->right != NULL) {
            find(root->right, expectNumber, path, sum, paths);
        }
        
        path.pop_back();
    }
};
```

我们这里采用的是引用传值，效率会高一些。当找到叶子结点且累加之和与指定值相等时，表示找到一条路径，则放到路径表中，并且一定要重置累加值。


#小结 

这道题参考别人的思路之后才写出来的，自己写的时候就卡在了path.pop_back和sum=0重置这里！

