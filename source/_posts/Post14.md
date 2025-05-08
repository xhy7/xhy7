---
title: 学习日记 | 哈夫曼编码树
date: 2025-05-08
tags:
  -学习
  -数据结构
---
# 核心思想
**将高频字符用短编码，低频用长编码，实现数据压缩**
# 代码
## 二叉哈夫曼树的建树
```cpp
// 节点定义
struct Node {
    char ch;        // 字符（叶子节点有效）
    int weight;     // 权重（出现频率）
    Node *left, *right; 
};

// 优先队列比较规则（最小堆）
struct Compare {
    bool operator()(Node* a, Node* b) {
        return a->weight > b->weight; // 小顶堆
    }
};

// 构建树过程
Node* buildTree(map<char, int> freqMap) {
    priority_queue<Node*, vector<Node*>, Compare> pq;
    for (auto& [ch, w] : freqMap) 
        pq.push(new Node{ch, w, nullptr, nullptr});

    while (pq.size() > 1) { // 合并到只剩一个根节点
        Node* a = pq.top(); pq.pop();
        Node* b = pq.top(); pq.pop();
        Node* parent = new Node{'\0', a->w + b->w, a, b}; 
        pq.push(parent);
    }
    return pq.top();
}
```
## 优先队列
这里用到了小顶堆来取每次最小的weight，需要自定义比较函数。同时定义也需要改一改
priority_queue<Node*,vector<Node*>,Compare>pq;
这里有一篇讲的很清楚的csdn[优先队列](https://blog.csdn.net/c20182030/article/details/70757660)
## 哈夫曼编码生成
**编码规则：左走记'0'右走记'1'，到叶子时路径就是编码**
具体代码：
```cpp
void generateCode(Node*root,string path,map<char,string>&codeMap)
{
    if(!root->left&&!root->right)
    {
        codeMap[root->ch]=path;
        return;
    }
    generateCode(root->left,path+'0',codeMap);
    generateCode(
    root->right,path+'1',codeMao
    );
}
```
有一篇知乎专栏上的文章，讲的也特别好[哈夫曼树](https://zhuanlan.zhihu.com/p/626456605)
# k叉哈夫曼树
当需要更高压缩率的时候需要用到k叉
与二叉的区别：
1. 节点结构
子节点要用vector来存储
```cpp
struct KNode {
    int weight;
    vector<KNode*> children; // K个子节点
};
```
2. 虚拟节点补充
当(n-1)%(k-1)!=0的时候要补充虚拟节点
```cpp
int m=k-((n-1)%(k-1))-1;
for(int i=0;i<m;i++)
{
    pq.push(new Knode{0,{}});
}
```
3. 合并逻辑
每次要合并k个节点
```cpp
while(pq.size()>1)
{
    KNode*parent=new KNode(0,{});
    for(int i=0;i<k;i++)
    {
        parent->weight+=pq.top()->weight;
        parent->children.push_back(pq.top());
        pq.pop();
    }
    po.push(parent);
}
```
# 荷马史诗
**追逐影子的人，自己就是影子** ⭐⭐⭐
## 题目
是一道很有感觉的题目。
![荷马史诗](/img/荷马史诗.png "洛谷")
## 代码
```cpp
#include<iostream>
#include<queue>
#include<algorithm>
#define ll long long
using namespace std;
struct node
{
    ll weight, height;
    node(ll w, ll h) :weight(w), height(h) {}
};
struct Compare
{
    bool operator()(node a, node b)
    {
        if (a.weight != b.weight)
            return a.weight > b.weight;
        else
            return a.height > b.height;
    }//这里的排序很重要，当ab的weight相等时，要把高度更小的放在顶
};
int main()
{
    priority_queue<node, vector<node>, Compare>q;
    ll n, k, ans = 0;
    cin >> n >> k;
    for (int i = 1; i <= n; i++)
    {
        ll w;
        cin >> w;
        q.push(node(w,1));
    }
    while ((q.size() - 1) % (k - 1) != 0)
    {
        q.push(node(0,1));
    }//添加虚拟节点
    while (q.size() > 1)
    {
        ll h = -1, w = 0;
        for (int i = 1; i <= k; i++)
        {
            node temp = q.top(); q.pop();
            h = max(h, temp.height);
            w += temp.weight;
        }
        ans += w;
        q.push(node(w,h+1));//此时加了一层height
    }
    cout << ans << endl;
    cout << q.top().height - 1;
    return 0;
}
```
作为我洛谷做的第一道蓝题，it deserves！💌
## 演好自己的角色
和这句话相似地，库切曾经说过：你内心肯定有着某种火焰🔥，能把你和他人区分开来。  
演好自己的角色，我的生命里才能始终跳动着灵魂的火焰。🎆
