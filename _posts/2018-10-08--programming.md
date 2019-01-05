---
layout: post
title: "Programming Contest"
categories: Contest
tags: Programming
--- 

* content
{:toc}

I realize that I need to get more practice for programming, so here I will note some interesting problems. 





### **Problem**
输入一颗二叉树的跟节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径.(在返回值的list中，数组长度大的数组靠前)

```
struct TreeNode{
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL){}
}

class Solution{
	vector<vector<int>> result;  //在外部设置全局变量
	vector<int> path;
public:
	vector<vector<int>> FindPath(TreeNode* root, int expectNumber){
		if (root==NULL){
			return result;
		}
		
		path.push_back(root->val);　　
		//如果这句放在if后面，导致到达叶节点满足if条件，push进了path,但path还有一个值没有传入．
		expectNumber=expectNumber-root->val;　
		//如果expectNumber放在if后面，会出现在叶节点时此时expectNumber不为0，先进行判断再减值，进入下次递归导致root==NULL;

		if(expectNumber==0 && root->left==NULL && root->right==NULL){
			result.push_back(path);
		}

		FindPath(root->left, expectNumber);　　//调用带返回值的函数我们可以不用这个返回值
		FindPath(root->right, expectNumber);
		path.pop_back();　//深度遍历完一条路径要回退
		return result;
	}
}
```
