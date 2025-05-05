---
title: 学习日记 | 树的重心
date: 2025-04-23 21:00:00
tags:
  -数据结构
  -二叉树
---
## 一：树的重心是什么？
树的重心有四个定义（同样也是性质，可以相互推导）分别是一下  
1. 树的重心如果不唯一，那么至多有两个，并且这两个点相邻
2. 以重心为根时，最大子树的节点数量最小
3. 以树的重心为根时，所有子树的大小都不超过整棵树大小的一半。
4. 树中所有点到某个点的距离和中，到重心的距离和是最小的；如果有两个重心，那么到它们的距离和一样。
## 二：树的重心有什么作用？
1. 优化分治算法（点分治），从重心分割树，保证递归层数为 O(logn)。
2. 动态树的维护，在动态树（如 Link-Cut Tree）中，重心帮助快速合并或分裂子树。
## 三：树的重心怎么求？（重点）
目前总结来看是两种方法，但是其实核心还是等价的（用不同的定义）
### 1：计算以当前节点为根的最大子树的节点数量，并且实时比较更新最小值。
具体代码：
```cpp
void dfs(int u, int father)//以father为父节点的u节点的子树节点数量(包括自身)
{   
	childnode[u] = 1;
    //以下是求子树节点数量的经典操作，积累一下
	for (int x : vec[u])
	{
		if (x == father)continue;
		dfs(x, u);
		childnode[u] += childnode[x];
		maxnode[u] = max(maxnode[u], childnode[x]);
	}
	maxnode[u] = max(maxnode[u], n - maxnode[u]);//这里很容易忽略，由于这不是一个有向树，所以另外一边也是它的子树（这有点反直觉，但得好好注意）
	if (maxnode[u] < curMin || (maxnode[u] == curMin) && u < center)
	{
		curMin = maxnode[u];
		center = u;
	}
}
```
### 2.dfs递归，计算每个节点的子树大小并判断是否满足重心条件。 
具体代码：
```cpp
int dfs(int u, int parent) 
{
    int size = 1; // 当前子树大小
    bool is_centroid = true;
    for(int v:tree[u])
    {
        if(v==parent)continue;
        int sub_size=dfs(v,u);
        if(sub_size>n/2)
        {
            is_centroid=false;
            break;
        }
        size+=sub_size;
    }
    //这里又是那容易忽略的一步，要检查另外一边
    if (n - size > n / 2) is_centroid = false;
    
    if (is_centroid) centroid = u;
    return size;
}
```
## 总结
这是我第一次写个人博客嘿嘿，希望以后可以坚持下去。这次学习了树的重心，自己真正动手写一些东西之后，感觉真的不一样！xhy加油！
## 修正(2025-5-4)
在第一种方法求最大子树的节点数的最小值中有这样一段代码
maxnode[u] = max(maxnode[u], n - maxnode[u]);//这里很容易忽略，由于这不是一个有向树，所以另外一边也是它的子树（这有点反直觉，但得好好注意）
这里是有问题的，n-maxnode[u]应该要修改为n-childnode[u]。
核心是要深入理解maxnode和childnode分别表示的是什么？
maxnode是指以当前节点为根节点的子树的最大节点数量；
而childnode是指包括当前节点在内的以当前节点为根的所有节点数量总和。
详情可看grl老师的灵魂画图 🥰
## 扩展(2025-5-5)
当树的边权不全为1，而会是其他值的时候，就有另外一种新的做法：先来看具体代码
```cpp
#include <iostream>
#include <vector>
#include <climits>
using namespace std;

struct edge {
    int to, weight;
    edge(int t, int w) : to(t), weight(w) {}
};

vector<vector<edge>> tree;
vector<int> sub_w; // 子树边权总和（仅包含子树内部）
int total = 0;     // 整棵树的边权总和

void dfs_pre(int u, int fath) {
    for (auto& e : tree[u]) {
        if (e.to != fath) {
            dfs_pre(e.to, u);
            sub_w[u] += sub_w[e.to] + e.weight; // 包含当前边权
        }
    }
}

vector<int> dis;
int min_max = INT_MAX;
int center;

void dfs_cal(int u, int fath) {
    int max_sub = 0;
    for (auto& e : tree[u]) {
        if (e.to != fath) {
            // 动态调整公式
            dis[e.to] = dis[u] + (total - 2 * sub_w[e.to]) * e.weight;
            // 计算最大子树边权
            int cur = sub_w[e.to] + e.weight;
            max_sub = max(max_sub, cur);
        }
    }
    // 比较上方子树（总边权 - 当前子树边权 - 父边权）
    max_sub = max(max_sub, total - sub_w[u] - 0); // 0为父边权（已包含在total中）
    
    if (max_sub < min_max) {
        min_max = max_sub;
        center = u;
    }

    for (auto& e : tree[u]) {
        if (e.to != fath) {
            dfs_cal(e.to, u);
        }
    }
}

int main() {
    int n;
    cin >> n;
    tree.resize(n + 1);
    sub_w.resize(n + 1, 0);
    dis.resize(n + 1, 0);

    // 输入边并计算总边权
    for (int i = 0; i < n - 1; i++) {
        int x, y, w;
        cin >> x >> y >> w;
        tree[x].emplace_back(y, w);
        tree[y].emplace_back(x, w);
        total += w; // 正确统计单次边权
    }

    dfs_pre(1, 0);
    dfs_cal(1, 0);

    cout << "重心节点: " << center << endl;
    return 0;
}
```
核心点是两个dfs，首先第一个dfs_pre是为了初始化sub_w的数组，把以每个节点为根的子树边权和求出来(包括节点本身)
第二个dfs_cal是计算重心，其中动态调整公式是核心

