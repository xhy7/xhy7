---
title: 日记 | 日子又开始变得浑圆饱满
date: 2025-04-29
tags:
  -日记
---
# 讨厌的调休
虽然周一的课还比较好，但是还是不喜欢只有一天的周末（以及连续上两天同样的课）
体育课要上点心啦aaa，50m和跳远成绩都不好，足球还磕磕绊绊的。
似乎已经好久没上过高数课了，但高数的学习也不能落下，每天学至少30min的高数，也是给期末复习减小一点压力。
TED演讲结课了，本来可以开开心心的结束这门通识课，但最后个人pre的时候老师计我用时太长，把我喊停的操作让我有点点小意见。  
但我个人也有原因：准备不够用心，有点依赖PPT，没有体现出个人演讲的特点。
唉，还是希望能在最后拿到一个不错的分数吧。（虔诚拜三拜）
# 高数的表格积分
这是一种简化分部积分的一个方法。核心点就是选好被积分和被求导的函数。
要求让被积分的函数能够一直积下去，而被求导的函数最终能够得到常数项。
（不要问我为什么下学期还在补上学期的内容，问就是还不会）
csdn上也有一篇讲的很全面的
[分部积分的快速运算：表格法](https://blog.csdn.net/LoraRae/article/details/121276931)
感觉csdn上的学习资源真的挺多的，至少在国内还是独一档的存在。可惜它的槽点也太多了= =  
希望它的管理者能好好听一听用户的反馈吧。
# 数据结构-LCA
LCA的确是树上问题一个很重要的板块，在很多问题的子问题中都会要用到寻找最近父节点。
但是它的求解方法也有点多，所以我向ds求助总结归纳后，得到以下结论：
## 倍增法
```cpp
void dfs(int u, int fath)
{
	fa[u][0] = fath;
	depth[u] = depth[fath] + 1;
	for (int i = 1; i <= 16; i++)
	{
		fa[u][i] = fa[fa[u][i - 1]][i - 1];
	}
	for (int v : graph[u])
	{
		if (v == fath)continue;
		else dfs(v, u);
	}
}
int getLCA(int x, int y)
{
	if (depth[x] < depth[y])swap(x, y);//保证x深度更深
	//xy跳到同一深度
	for (int i = 16; i >= 0; i--)
	{
		if (depth[fa[x][i]] >= depth[y])
			x = fa[x][i];
	}
	if (x == y)return x;
	for (int i = 16; i >= 0; i--)
	{
		if (fa[x][i] != fa[y][i])
		{
			x = fa[x][i];
			y = fa[y][i];
		}
	}
	//上面两个for循环都是从大步长开始跳的，思考为什么
	return fa[x][0];
    //想想这里为什么返回的是x的父节点
}
```
## tarjan算法
**核心**
通过dfs遍历树，利用并查集动态维护节点的祖先关系。当访问完某节点的所有子树后，将该节点和父节点合并。查询时，若另一节点已被访问，则其所在集合的代表元素即为LCA
**特点**
1. 离线算法：预先存储所有查询，统一处理
2. 时间复杂度：O(n+q)
3. 空间复杂度：O(n+q)
```cpp
//初始化
vector<int> adj[N];         // 邻接表存树
vector<pair<int, int>> query[N]; // query[u]存储与u相关的查询（查询编号，另一节点）
int fa[N];                  // 并查集父节点数组
bool vis[N];                // 访问标记数组
int ans[M];                 // 存储所有查询结果
//并查集
int find(int x) {           // 路径压缩
    return fa[x] == x ? x : fa[x] = find(fa[x]);
}

void merge(int x, int y) {  // 合并操作
    int fx = find(x), fy = find(y);
    if(fx != fy) fa[fy] = fx;
}
//dfs核心逻辑
void tarjan(int u) {
    vis[u] = true;          // 标记已访问
    for(int v : adj[u]) {   // 遍历子节点
        if(!vis[v]) {
            tarjan(v);
            merge(u, v);    // 关键：合并子节点到当前节点
        }
    }
    // 处理所有与u相关的查询
    for(auto [v, id] : query[u]) {
        if(vis[v]) {        // 若另一节点已访问
            ans[id] = find(v); // LCA为v所在集合的代表节点
        }
    }
}
//main函数
int main() {
    // 初始化并查集
    for(int i=1; i<=n; i++) fa[i] = i;
    
    // 添加双向查询（重要！）
    for(int i=0; i<q; i++) {
        int u, v;
        cin >> u >> v;
        query[u].emplace_back(v, i);
        query[v].emplace_back(u, i);
    }
    
    // 从根节点开始DFS
    tarjan(root);
    
    // 输出结果
    for(int i=0; i<q; i++) 
        cout << ans[i] << endl;
}
```
**关键细节**
1. 合并时机：子节点回溯时立即合并到父节点
2. 双向存储查询：每个查询(u,v)都要同时存入u和v的查询列表
3. 离线处理本质：所有查询必须在dfs前预先存储，无法动态处理新查询
**疑问点分析**
1. 这里回溯合并的目的是保证已处理子树的节点集合始终指向
2. 代表节点会被动态更新为当前路径上的最低层祖先
![lca解法总结](/img/LCA.png "xhy")
## 小flag--往后安排
每天晚上九点五十左右开始听一听算法课程，好好补一补差距。
详情请上bilbili搜索up主“左程云”，讲的真的很好！
# 校新总结会
让美好的瞬间定格在当下吧，谁能想到我们能一起并肩走到这么远呢~
![计科辩珍贵合影](/img/校新总结1.jpg "xhy")
![计科辩珍贵合影](/img/校新总结2.jpg "xhy")
![计科辩珍贵合影](/img/校新总结3.jpg "xhy")
qq学姐的一句话很有道理：**辩论其实真的不花时间，只是把你每天用来玩手机的1h拿来备赛罢了**
but，我的实际情况好像是把学习的1h来备赛，讨论完回去该铲还是铲嘿嘿![alt text](image.png)
# 写在最后
把每天过的充实，毕竟，治愈自己最好的方法，就是忙碌和睡觉。:blush:


