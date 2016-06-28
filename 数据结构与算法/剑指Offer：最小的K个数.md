#概述

剑指Offer[最小的K个数](http://www.nowcoder.com/practice/6a296eb82cf844ca8539b57c23e6e9bf?tpId=13&tqId=11182&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)算法题，可以采用的方法比较多，本篇采用最小堆排序来查找。

#题目描述

输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

#要求

时间限制：1秒  
空间限制：32768K

#分析

从N个数中查找最小的前K个数，最常规的办法就是进行全排序，所有取前K个最小的即可，但是这样做的话，效率是比较低下的，很有可能过不了。

而题目中要求取的是前k个最小的数，那么我们可以采用最小堆排序，每次调整完堆，堆顶都是本次调整的堆中最小的数，因此只需要调整K次就可以得到最终答案了。

同样，如果是取最大的K个数，就采用最大堆排序就可以了！


#C++

```
class Solution {
public:
    vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
        vector<int> result;  
        
        if (k > input.size() || k <= 0) {
            return result;
        }
        
        if (k == input.size()) {
            return input;
        }
        
        heapSort(input, k, result);
        
        return result;
    }
    
    void heapSort(vector<int> &input, int k, vector<int> &result) {
        initMinHeap(input);
        
        for (int i = input.size() - 1; i > 0; --i) {
            // 将堆顶元素与当前堆的最后一个元素对比，如果比堆顶元素大，则调整
            if (input.at(0) < input.at(i)) {
                int tmp = input.at(0);
                input.at(0) = input.at(i);
                input.at(i) = tmp;
                
                result.push_back(tmp);
            }
            
            if (result.size() >= k) {
                return;
            }
            
            adjustMinHeap(input, i, 0);
        }
    }
    
    void initMinHeap(vector<int> &input) {
        for (int i = (input.size() - 1) / 2; i >= 0; --i) {
            adjustMinHeap(input, input.size(), i);
        }
    }
    
    void adjustMinHeap(vector<int> &input, int len, int parentNodeIndex) {
        if (len < 1) {
            return;
        }
        
        int targetIndex = -1;
        int leftIndex = 2 * parentNodeIndex  + 1;
        int rightIndex = 2 * parentNodeIndex + 2;
        
        if (leftIndex >= len) {
            return;
        }
        
        // 只有左孩子
        if (rightIndex >= len) {
            targetIndex = leftIndex;
        } else {
            targetIndex = input.at(leftIndex) < input.at(rightIndex) ? leftIndex : rightIndex;
        }
        
        if (input.at(targetIndex) < input.at(parentNodeIndex)) {
            int tmp = input.at(targetIndex);
            input.at(targetIndex) = input.at(parentNodeIndex);
            input.at(parentNodeIndex) = tmp;
            
            // 将叶子结点与根结点交换之后，有可能导致不再满足最小堆的性质，因此需要再调整一下
            adjustMinHeap(input, len, targetIndex);
        }
    }
};
```

#小结

本题做法可以有很多种，这里只是其中一种。我们还可以用简单选择排序算法进行K次排序就可以选择出来了！