---
title: 学习日记 | 最小生成树
date: 2025-05-09
tags:
  -学习
  -数据结构
  -图算法
---
# 写在前面
星辰为点⭐，爱意为边❤️，最小生成树绘出我们最纯粹的相遇，情牵一生。 
<!-- more -->  
# Prim算法
**算法核心**：从一个起始顶点开始，每次选择连接已选顶点集合和未选顶点集合的边中权重最小的边，逐步扩展生成树。时间复杂度为 O (E log V)，适合稠密图。
## 具体代码
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <limits>
using namespace std;
typedef pair<int,int> ii;//边的类型是顶点+权重
//prim具体实现
vector<ii>prim(vector<vector<ii>>&ajd,int start)
{
    int n=adj.size();
    vector<bool>vis(n,0);
    vector<ii>mst;
    priority_queue<ii, vector<ii>, greater<ii>> pq;
    pq.push({0,start});
    while(!q.empty())
    {
        int u = pq.top().second;
        int w = pq.top().first;
        pq.pop();

    if (visited[u]) continue;
    visited[u] = true;

    if (u != start) {
            mst.push_back({u, w}); // 加入最小生成树
    }

        // 遍历所有邻接边
    for (auto& edge : adj[u]) {
        int v = edge.first;
        int weight = edge.second;
        if (!visited[v]) {
            pq.push({weight, v});
        }
    }
    }
}
```
## 代码细节思考
1. 为什么pair中要把weight权重放在前面？这里只是为了偷懒方便，直接用greater<ii>来设置比较函数建立小顶堆
2. 这里还是一个模板(bfs+优先队列)
3. 为什么适合稠密图呢？因为这里是以点为核心来做，不论图中边有多少，时间复杂度只和点个数有关。
# Kruskai算法
将所有边按权重排序，每次选择权重最小的边，如果该边不形成环，则加入生成树。使用并查集 (Union-Find) 高效检测环。时间复杂度为 O (E log E)，适合稀疏图。
## 具体代码
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
vector<int>parent;
struct Edge
{
    int u,v,weight;
    Edge(int u,int v,int weight):u(u),v(v),weight(weight){}
}
struct Compare
{
    bool ()(Edge&a,Edge &b)
    {
        return a.weight>b.weight;
    }
}
void init(int n)
{
    parent.resize(n);
    for(int i=0;i<n;i++)
    {
        parent[i]=i;
    }
}
int find(int x)
{
    if(parent[x]!=x)
    {
        parent[x]=find(parent[x]);
    }
    return parent[x];
}
void unite(int x,int y)
{
    int rootx=find(x);
    int rooty=find(y);
    if(rootx!=rooty)
    {
        parent[rooty]=rootx;
    }
}
bool isconnected(int x,int y)
{
    return find(x)==find(y);
}
//上面都是并查集的实现，接下来是Kruskal算法
vector<Edge>Kruskal(vector<Edge>edges,int n)
{
    vector<Edge>mst;
    sort(edges.begin(),edges.end(),Compare);
    for(auto&edge:edges)
    {
        if(!isconnected(edge.u,edge.v))
        {
            unite(edge.u,edge.v);
            mst.push_back(edge);
            if(mst.size()==n-1)break;
        }
    }
    return mst;
}
```
## 代码细节分析
1. 这个算法是基于边来计算，所以与点的数量无关，可以适用于稀疏图（边多点少）
2. 这里的优化是用并查集做的，越发体现出并查集的重要性了

