---
title: 学习日记 | 和洛谷拼了
date: 2025-05-04
tags:
  -学习
  -数据结构
  -洛谷
---
**于洛谷的算法星辰中，调试出你的最优解**
# 第六周习题
## P1012 拼数 tags(串，模拟)
核心点就是写一个cmp比较函数，然后用sort排序
```cpp
bool cmp(string a,string b) 
{ 
    return (a+b > b+a);
}
```
简单题，不多说
## P1928 外星人密码 tags(串，递归)
看到这道题就很伤心，期中考试考了这个原题我还没写出来，关键是递归函数的返回值(要不要返回，返回什么)没有想清楚，同时什么时候开始递归也没有确定
下面是具体代码：
```cpp
string decode() {
    char c;
    string res;
    while(cin.get(c) && c != '\n') {
        if(c == '[') {
            int k = 0;
            while(cin.peek() >= '0' && cin.peek() <= '9') {
                cin.get(c);
                k = k * 10 + (c - '0');
            }
            string sub = decode();
            while(k--) res += sub;
        }
        else if(c == ']') {
            return res;
        }
        else {
            res += c;
        }
    }
    return res;
}
```
这里这个递归函数把输入也包括在内
关键点：
1. 递归函数需要返回值string
2. 三个条件判断句对应的三种情况
3. 这里用的cin.get(c)是一个特殊的输入流，不能改为getchar(c)，否则后面的cin.peek()就会出错
反思：
其实这道题就体现了递归的核心思想：把一个问题分解成相似的小问题，然后向下逐步解决，并把返回值向上返回。
## P1019 拼字符串 tags(串，搜索)
这道题的核心是把多个字符串首尾相连拼在一起，用到的核心函数是string.substr()，就是把一个字符串对应一段位置的字符copy出来。
核心dfs代码：
```cpp
void dfs(string temp)
{
	res=max(res,(int)temp.size());
	for(int i=0;i<n;i++)
	{
		if(vis[i]>=2)
		continue;
		for(int j=1;j<min(temp.size(),s[i].size());j++)//用min可以减小循环量
		{
			if(temp.substr(temp.size()-j)==s[i].substr(0,j))//temp的后j个和s[i]的前j个匹配上了，其中到字符串的最后end()可省略
			{
				vis[i]++;
				dfs(temp+s[i].substr(j));
				vis[i]--;
			}
		}
	}
}
```
简单题，不多说
## B3712 敌人的朋友是敌人 tags(数组，模拟)
这里直接放一下原码
```cpp
#include <iostream>
#include <vector>
#include <unordered_set>
#include <algorithm>

using namespace std;
using ll = long long;

// 生成唯一的键，保证a <= b
ll getKey(int a, int b) {
    if (a > b) swap(a, b);
    return (ll)a * 1000000 + b;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, p, q;
    cin >> n >> p >> q;

    unordered_set<ll> friend_pairs, enemy_pairs;
    vector<vector<int>> friends(n + 1); // friends[u]存储u的朋友
    vector<vector<int>> enemies(n + 1); // enemies[u]存储u的敌人

    // 处理朋友关系
    for (int i = 0; i < p; ++i) {
        int u, v;
        cin >> u >> v;
        ll key = getKey(u, v);
        friend_pairs.insert(key);
        friends[u].push_back(v);
        friends[v].push_back(u);
    }

    // 处理敌人关系
    for (int i = 0; i < q; ++i) {
        int u, v;
        cin >> u >> v;
        ll key = getKey(u, v);
        enemy_pairs.insert(key);
        enemies[u].push_back(v);
        enemies[v].push_back(u);
    }

    unordered_set<ll> exclude_pairs;

    // 正确方法：遍历每个可能的中间人w
    for (int w = 1; w <= n; ++w) {
        const auto& friend_list = friends[w];
        const auto& enemy_list = enemies[w];

        // 生成所有 (u, v) 对，其中u是w的朋友，v是w的敌人
        for (int u : friend_list) {
            for (int v : enemy_list) {
                ll key = getKey(u, v);
                if (!friend_pairs.count(key) && !enemy_pairs.count(key)) {
                    exclude_pairs.insert(key);
                }
            }
        }

        // 生成所有 (u, v) 对，其中u是w的敌人，v是w的朋友
        for (int u : enemy_list) {
            for (int v : friend_list) {
                ll key = getKey(u, v);
                if (!friend_pairs.count(key) && !enemy_pairs.count(key)) {
                    exclude_pairs.insert(key);
                }
            }
        }
    }

    // 计算总握手次数
    ll total = (ll)n * (n - 1) / 2;
    ll handshake = total - q - exclude_pairs.size();

    cout << handshake << endl;

    return 0;
}
```
由于那周有点小偷懒，这道题没有写，于是直接看看AI的标准码来临时抱抱佛脚
# 第七周习题
## P3395 路障 tags(队列，bfs)
一个简单的bfs题目，但很多细节要注意
因为和时间有关，所以struct中要加上time这个变量
核心bfs代码
```cpp
//vis[x][y]=i表示在第i秒，坐标为x,y的位置被放上了路障
bool bfs()
{
    vis[1][1]=-1;
    ss.push(stu(1,1,0));
    while(!ss.empty())
    {
        stu k=ss.front();
        ss.pop();
        if(k.x==n&&k.y==n)return 1;
        for(int i=0;i<4;i++)
        {
            int nowx=k.x+dir[i][0];
            int nowy=k.y+dir[i][1];
            int nowtime=k.time+1；
            if(vis[nowx][nowy])continue;
            if(check(nowx,nowy))continue;
            //注意这里要check一下，不然越界了
            if(vis[nowx][nowy]<nowtime&&vis[nowx][nowy]>0)continue;
            //这里有点难理解，表示x,y坐标处在这之前已经被放上了路障，同时要保证>0
            vis[nowx][nowy]=-1;//把这里设置为以访问
            ss.push(stu(nowx,nowy,nowtime));
        }

    }
}
```
同时这里可以扩展学习一下快读方法：
```cpp
inline int read() {
    char ch = getchar(); int x = 0, f = 1;
    while(ch < '0' || ch > '9') {if(ch == '-') f = -1; ch = getchar();}
    while('0' <= ch && ch <= '9') {x = x * 10 + ch - '0'; ch = getchar();}
    return x * f;
}
```
主要是读很大的数字，比cin快在getchar()没有缓冲区
## P1016 旅行家的预算 tags(贪心，递归)
4个关键点：
1. 枚举途中经过的加油站，每次经过一个加油站，都要计算一次花费
2. 在一个加油站所需要加的油，就是能够支持它到达下一个油价比它低的加油站的量
3. 如果在当前加油站加满油都不能到下一个油价更低的，就加满油箱，前往能够到达的加油站中油价最低的
4. 如果加满油都不能到下一个加油站，说明无解
直接上原码：
```cpp
#include<bits/stdc++.h>
using namespace std;
double d1,c,d2,p,d[10],v[10],last,ans;	//last:剩下的油量 ans:最小费用 其他变量均为输出数据 
int n;
int main(){
	scanf("%lf%lf%lf%lf%d",&d1,&c,&d2,&p,&n);
	for(int i=1;i<=n;i++) scanf("%lf%lf",&d[i],&v[i]);
	d[n+1]=d1;v[0]=p;
	for(int i=1;i<=n;i++){		//油箱中的油能否跑完每一段加油站的距离 
		if(d[i+1]-d[i]>c*d2){
			printf("No Solution\n");	//不能打出 No Solution
			return 0;
		}
	}
	int j;
	for(int i=0;i<=n;i=j){
		for(j=i+1;j<=n+1;j++){
			if(d[j]-d[i]>c*d2){		//在c*d2的距离内的加油站中寻找最便宜的加油站
				j--;
				break;
			}
			if(v[j]<v[i]) break;	//找到更便宜的加油站就退出循环 
		}
		if(v[j]<v[i]){		//有找到更便宜的加油站
			ans+=((d[j]-d[i])/d2-last)*v[i];	//加刚好足够到达第j个加油站的油 
			last=0;
		}
		else {
			ans+=(c-last)*v[i];			//加满油箱
			last=c-(d[j]-d[i])/d2;
		}
	}
	printf("%.2lf\n",ans);
	return 0;
}
```
嘿嘿这道题当时也没写，所以来直接看题解代码啦
## P1036 选数 tags(dfs)
核心dfs代码
```cpp
void dfs(int size,int index,int val)
{
	if(size==k&&isPrime(val))
	{
		ans++;
		return;
	}
	for(int i=index;i<n;i++)
	{
		if(vis[i]==0)
		{
			vis[i]=1;
			dfs(size+1,i,val+a[i]);
			vis[i]=0;
		}
	}
}
//唯一需要关注的点就是index的意义，这里用index来表示当前的位置，所以for循环可以直接从当前的index开始
```
简单题，不多说
## P1091 合唱队形 tags(单调队列，dp)
题目的核心就是找到一个合适的同学，让他的前后都是递减序列
抽象一下，就是寻找以每个位置结尾的最长上升子序列和以其开头的最长下降子序列
```cpp
 // 计算以每个位置结尾的最长上升子序列（LIS）
    for(int i = 1; i <= n; i++){
        lis_dp[i] = 1; // 每个位置至少可以单独形成一个序列
        for(int j = 1; j < i; j++){
            if(heights[j] < heights[i]){
                lis_dp[i] = max(lis_dp[i], lis_dp[j] + 1);
            }//就看要不要第j个人
        }
    }
// 计算以每个位置开头的最长下降子序列（LDS）
    for(int i = n; i >= 1; i--){
        lds_dp[i] = 1; // 每个位置至少可以单独形成一个序列
        for(int j = n; j > i; j--){
            if(heights[j] < heights[i]){
                lds_dp[i] = max(lds_dp[i], lds_dp[j] + 1);
            }
        }
    }
```
这里是找LIS和LDS的模板，就是用个for循环，其中用一个if条件判断，再用max来更新dp数组
# 第八周习题
## P8306 字典树 tags(Trie)
这是个模板题，关于字典树的知识已经比较熟悉了，其中TrieNode的children可以用数组，也可以用unordered_map，注意如果用unordered_map的话要注意char和node*的顺序，想好是通过谁来找谁。这里代码就用哈希map来实现
```cpp
struct trienode
{
	int count = 0;//这里的count实际就是pass
	unordered_map<char, trienode*>children;
};
void insert(string s, trienode* root)//这里把虚拟根节点root直接作为参数传进来
{
	trienode* node = root;
	for (char ch : s)
	{
		if (node->children.count(ch) == 0)
		{
			node->children[ch] = new trienode();
		}
		node = node->children[ch];
		node->count++;
	}
}
int query(string s, trienode* root)//这里也同样传了root参数
{
	trienode* node = root;
	for (char ch : s)
	{
		auto it = node->children.find(ch);
		if (it == node->children.end())
		{
			return 0;
		}
		node = it->second;
	}
	return node->count;
}
```
简单板子题，不多说
## P1481 魔族密码 tags(dp，字典树，最长上升子序列)
这题有两种方法，一种是字典树：从根节点开始建树，最后记录一下每一个链上的end值的和。
另外一种方法是利用dp来做LIS，其中用了一个char的函数strncmp()，具体代码：
```cpp
for(int i = 1; i <= n; i++)
    {
        cin >> strings[i];
        for(int j = 1; j < i; j++)
        {
            if(strncmp(strings[i].c_str(), strings[j].c_str(), strings[j].length()) == 0)//该函数返回值为0表示两个字符串匹配
            {
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        ans = max(ans, dp[i]);
    }
```
## P2922 信息暗号 tags(字典树)
这也是一个比较模板的题，用字典树可以很快解决，但这里有几个关键点：
1. children可以用数组快速表示
2. 字典树节点需要pass和end两个成员变量
3. 需要分信息更长/暗号更长两种不同情况来讨论
具体代码：
```cpp
//涉及到pass和end的建树过程
void insert(trienode* root, vector<int>bits, int len)
{
	trienode* node = root;
	for (int i = 0; i < bits.size(); i++)
	{
		int x = bits[i];
		if (!node->children[x])
		{
			node->children[x] = new trienode();
		}
		node = node->children[x];
		node->pass++;
		if (i + 1 == len)//到end
		{
			node->end++;
		}
	}
}
//这里的treenode描述的是信息
int query(trienode* root, vector<int>vec, int len)//len是暗号的len
{
	trienode* node = root;
	int cas1 = 0, cas2_sum = 0, k = 0;
	for (int x:vec)
	{
		if (!node->children[x])
			break;//跳出循环，返回0
		node = node->children[x];
		k++;//记的是信息的长度
		if (k < len)
			cas2_sum += node->end;//暗号更长，所以要统计信息结束的总和
		else if (k == len)
		{
			cas1 = node->pass;//信息更长或一样长，此时就不要求信息要结尾，只要pass了就可以，因为暗号已经全部看完结束，所以可以直接break掉
			break;
		}
	}
	return cas1 + cas2_sum;//计算两个情况的总和
}
```
这里还是要理清字典树中文本串和查询目标串的关系，并且理解pass和end两个变量的意义。
## P1827 中序前序推后序 tags(递归，树的遍历)
很经典的题目了
几个核心点：
1. 把中序遍历的结果转化为map方便随时查询头节点的位置
2. 计算左右子树的大小并且划分好区间
3. 索引位置理清，start和end分别在哪儿
具体核心代码：
```cpp
//递归建树
void build(int pre_s, int pre_e, int in_s, int in_e, vector<char>preorder)
{
	if (pre_s > pre_e)return;
	//先找到根节点的数据和位置
	int root_val = preorder[pre_s];
	int root_index = map[root_val];
	//左子树的节点数是根的索引-起始索引
	int left_size = root_index - in_s;
	//先构建左子树，
	build(pre_s + 1, pre_s + left_size, in_s, root_index - 1, preorder);
	build(pre_s + left_size + 1, pre_e, root_index + 1, in_e, preorder);
	postorder.push_back(root_val);
}
//solution代码，先通过中序来建立对应的map表，再调用build函数
vector<char>buildPostorder(vector<char>preorder, vector<char>inorder)
{
	for (int i = 0; i < inorder.size(); i++)
	{
		map[inorder[i]] = i;
	}
	build(0, preorder.size() - 1, 0, inorder.size() - 1, preorder);
	return postorder;
}
```
简单题，不多说
# 第九周习题
## P8602 树的直径 tags(dfs)
这里有一个很巧妙的方法（具体需要数学证明其正确性，但是可以直接使用）
要得到dfs最远路径，可以先从任意节点dfs一遍得到最终节点，再从当前最终节点dfs一次得到的就是最远路径，也就是树的直径
还有一点点小细节，注意在dfs前要把vis数组设置一下，不然会有一丢丢的问题
```cpp
int cur = 0, i = 1;
visited[i] = 1;
dfs(i, cur);
visited[i] = 0;
dfs(target, cur);
```
## 扩展反思：
这里有个特别特别有意思的点，是cpp语法和csapp的梦幻联动：
这里的dfs传参，为什么直接传参和引用间接传参都是正确的呢？
首先考虑为什么引用传参能修改传入的数据，由csapp知识可以知道，传入的第一个参数实际是存在rdi中，如果是直接传参，在函数内部修改的是rdi的值，但是一旦函数返回后，rdi就会失效，寄存器的保护就会把它恢复到调用前的状态。如果是间接传参的话，编译器就会通过内存寻址，到内存中直接把对应位置上的值修改掉。
再来想想为什么这里直接传和引用传都可以正确解决问题。核心是因为这个cur在函数外部不起作用，它在函数内部就已经被正确地传入给了下一个dfs的递归函数，在这样一个一个接力的过程中被传下去了。在外部作用的是res全局变量，而我用max已经把cur的影响偷偷地放在了res上，所以cur是通过res在main函数中发挥作用的。
最后再来多想一步，这里和树形dp有什么联系区别呢？这里都是当前状态受其他状态影响，但是dfs这里的只跟前面一个(父节点)的状态有关，所以可以直接用一个cur变量来往下传。而与之相对应的是，树形dp的当前状态和父/子节点状态都有关系，所以要用一个dp数组(有时是二维来体现更多的信息)来存节点的信息。
**福至心灵的顿悟时刻**
## P1395 树的重心 tags(树上问题)
在我前面的blog中已经详细讲过这个点，可以返回看看~[树的重心](https://xhy777.asia/2025/04/23/my-new-post/)
## P1352 没有上司的舞会 tags(树形dp)
这是一道很经典的树形dp入门题，当前职员去或不去不仅受到上级影响，同时也会影响到下级。
用二维数组来存储信息，第一维表示第i个人，第二维表示去或不去，具体dfs代码：
```cpp
void dfs(int x)
{
	dp[x][0] = 0;//不来
	dp[x][1] = p[x];//来
	for (int i = 0; i < son[x].size(); i++)
	{
		int y = son[x][i];
		dfs(y);
        //核心dp递推公式
		dp[x][0] += max(dp[y][0], dp[y][1]);
        //不来的话，就去下级来或不来的max
		dp[x][1] += dp[y][0];
        //来的话，下级就不能来了
	}
}
```
小小反思：
感觉树形dp的核心就是创建好存储信息的数组，明确是几个维度、每个维度分别代表的是什么。当然递推公式也是很重要的。  
之前在bilibili上听左神讲课的时候，讲到用树形dp找最大的二叉搜索树，其中核心点是分析父树需要子树的哪些信息，把这些信息的全集定义为递归的返回值，在dfs主体中根据传入的信息来得到当前树的对应信息并且返回。
# 第十周
## P2015 二叉苹果树 tags(树形dp+背包问题)
问题抽象出来就是找一棵树中给定边数的子树中最大节点权重和
核心dfs代码：
```cpp
void dfs(int u, int fath)
{
	for (edge v1 : tree[u])
	{
		int next = v1.branch;
		int w = v1.v;
		if (next == fath)continue;
		else
		{
			dfs(next, u);//因为dfs在前，所以子节点的信息是先算的
			sum[u] += sum[next] + 1;//这里算总数
			for (int j = min(sum[u],k); j >= 0; j--)
			{
				for (int i = 0; i <= j - 1; i++)//尝试在当前子树选几个
				{
					dp[u][j] = max(dp[u][j], dp[u][j - i - 1] + dp[next][i] + w);//核心递推公式
				}
			}
		}
	}
}
```
几个关键点：
1. 这里二维数组dp[u][j]表示的是以u为根节点的子树中取j条边的最大节点权重和
2. 第一层for循环中要从最大的j开始递减的原因是要用更新前的dp而不能用更新后的（这里可能有点抽象难以理解），更新后的dp已经考虑了当前这个子树并且拿了一些树枝了。
3. 理解这个递推公式：dp[u][j] = max(dp[u][j], dp[u][j - i - 1] + dp[next][i] + w);
## P3379 最近公共祖先LCA模板 tags(倍增法)
这道题的解决方法有很多，之前的blog中有过截图总结，这里还是用倍增法来写。
```cpp
void dfs(int u, int fath)//预处理初始化的过程
{
	fa[u][0] = fath;
	depth[u] = depth[fath] + 1;
	for (int i = 1; i <= 16; i++)
	{
		fa[u][i] = fa[fa[u][i - 1]][i - 1];
	}
	for (int v : cowshed[u])
	{
		if (v != fath)
			dfs(v, u);
	}
}
//上面是用递归来写的，但是如果碰到链过于长的时候，系统栈就有可能爆掉，此时就可以用到这里的stack来写避免爆栈
void dfs(int root) {
    //把传入的参数用pair存进st中
    stack<pair<int, int>> st;
    st.push({root, 0});

    while (!st.empty()) {
        auto [x, fath] = st.top();
        st.pop();
        if (deep[x]) {
            continue; // 已经访问过
        }

        deep[x] = deep[fath] + 1;
        fa[x][0] = fath;
        for (int i = 1; i<=16; i++) {
            fa[x][i] = fa[fa[x][i - 1]][i - 1];
        }

        for (int to : edges[x]) {
            if (to != fath) {
                st.push({to, x});
            }
        }
    }
}
//上面都是预处理共做，接下来是找LCA
int getLCA(int x, int y)
{
	if (depth[x] < depth[y])swap(x, y);//此时x深度更深，这也是前面要存储depth的原因
	for (int i = 16; i >= 0; i--)
	{
		if (depth[fa[x][i]] >= depth[y])//开始从最大的步长尝试向上跳
			x = fa[x][i];
	}
	if (x == y)return x;
    //这里x和y在同一深度，开始一起往上跳
	for (int i = 16; i >= 0; i--)
	{
		if (fa[x][i] != fa[y][i])
		{
			x = fa[x][i];
			y = fa[y][i];
		}
	}
	return fa[x][0];
}
```
## P3128 树上点差分 tags(点差分)
核心代码和上道题的LCA很相似，但多了一个差分的核心
```cpp
for (int i = 0; i < k; i++)
{
	int x, y;
	cin >> x >> y;
	edge[x]++, edge[y]++;		
    int lca = getLCA(x, y);
	edge[lca]--, edge[fa[lca][0]]--;
}
void work(int u, int fath)
{
	for (int v : cowshed[u])
	{
		if (v == fath)continue;
		work(v, u);
		edge[u] += edge[v];
	}
	res = max(res, edge[u]);
}
```
注意理解点差分修改的四个点，为什么改的是这四个？为什么修改这四个就够？
# 写在最后
5周的习题量也不多，只有17题左右。验收考试会考5道题，3黄2绿，希望能考到我比较熟练熟悉的。有些题目会有改动，到时候还需要随机应变。
洛谷，真是一个让我又爱又恨恨恨恨恨恨恨的网站呢~






