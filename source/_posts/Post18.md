---
title: 复习笔记 | 数据结构
date: 2025-06-15
tags:
  -学习
  -数据结构
---
**xhy数据结构期末考加油！！！**
<!-- more -->
# 重要代码具体实现
```cpp
//栈和队列的具体实现（用数组和栈）
//用数组实现难度不大，注意用索引加减的时候最好对capacity取个余数 %
//关键是用链表的实现，指针引用时要时刻注意空指针，push和pop的时候要用.empty函数先检验，最后要++/-- size
void push(const T& value) {
        Node* newNode = new Node(value);
        newNode->next = topNode;
        topNode = newNode;
        ++stackSize;
    }

    void pop() {
        if (empty()) {
            throw std::underflow_error("Stack underflow");
        }
        Node* temp = topNode;
        topNode = topNode->next;
        delete temp;
        --stackSize;
    }
//这里是链表实现栈的关键，一个是确定链表是从栈顶向栈底，栈顶一定是head头指针，另外在pop的时候要注意用temp暂存再delete

//用栈进行表达式的求值
//首先是把中缀表达式转化为后缀
string infixToPostfix(string infix) {
    stack<char> s;
    string postfix;
    unordered_map<char, int> precedence = {{'+', 1}, {'-', 1}, {'*', 2}, {'/', 2}};
    
    for (char c : infix) {
        if (isdigit(c)) {
            postfix += c;
        } else if (c == '(') {
            s.push(c);
        } else if (c == ')') {
            while (s.top() != '(') {
                postfix += s.top();
                s.pop();
            }
            s.pop(); // 弹出 '('
        } else { // 运算符
            while (!s.empty() && s.top() != '(' && precedence[s.top()] >= precedence[c]) {
                postfix += s.top();
                s.pop();
            }
            s.push(c);
        }
    }
    
    // 弹出剩余运算符
    while (!s.empty()) {
        postfix += s.top();
        s.pop();
    }
    return postfix;
}
//后缀表达式求值
int evaluatePostfix(string postfix) {
    stack<int> s;
    
    for (char c : postfix) {
        if (isdigit(c)) {
            s.push(c - '0');
        } else {
            int b = s.top(); s.pop();
            int a = s.top(); s.pop();
            
            switch(c) {
                case '+': s.push(a + b); break;
                case '-': s.push(a - b); break;
                case '*': s.push(a * b); break;
                case '/': s.push(a / b); break;
            }
        }
    }
    return s.top();
}

//双端队列实现滑动窗口最大值
#include <vector>
#include <deque>
using namespace std;

vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    vector<int> res;
    deque<int> q; // 存储下标，按值降序排列
    
    for (int i = 0; i < nums.size(); i++) {
        // 移除窗口外的元素
        if (!q.empty() && q.front() == i - k) q.pop_front();
        
        // 维护队列单调性：移除队尾比当前值小的元素
        while (!q.empty() && nums[q.back()] < nums[i]) q.pop_back();
        
        // 当前元素入队
        q.push_back(i);
        
        // 窗口形成后记录最大值
        if (i >= k - 1) res.push_back(nums[q.front()]);
    }
    return res;
}

// kmp算法及其应用
// 计算next数组（以-1起始）
vector<int> getNext(const string& pattern) {
    int m = pattern.size();
    vector<int> next(m, -1);
    int i = 0, j = -1;
    while (i < m - 1) {
        if (j == -1 || pattern[i] == pattern[j]) {
            i++;
            j++;
            next[i] = j;
        } else {
            j = next[j];
        }
    }
    return next;
}
int kmpSearch(const string& text, const string& pattern) {
    int n = text.size();
    int m = pattern.size();
    if (m == 0) return 0;  // 空模式匹配开头
    
    vector<int> next = getNext(pattern);
    int i = 0, j = 0;
    while (i < n) {
        if (j == -1 || text[i] == pattern[j]) {
            i++;
            j++;
        }
        if (j == m) {  // 匹配成功
            return i - j;
        } else if (i < n && text[i] != pattern[j]) {
            j = next[j];  // 回退j
        }
    }
    return -1;  // 未找到匹配
}

//建树可能的几种输入方法：
//1.前序遍历序列带有空节点标记
TreeNode* buildTree(vector<string>& nodes, int& idx) {
    if (idx >= nodes.size() || nodes[idx] == "#") {
        idx++;
        return nullptr;
    }
    TreeNode* node = new TreeNode(stoi(nodes[idx++]));
    node->left = buildTree(nodes, idx);
    node->right = buildTree(nodes, idx);
    return node;
}
//2.前序遍历+中序遍历
TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
    unordered_map<int, int> inMap;
    for (int i = 0; i < inorder.size(); i++) {
        inMap[inorder[i]] = i;
    }
    return build(preorder, 0, preorder.size()-1, inorder, 0, inorder.size()-1, inMap);
}

TreeNode* build(vector<int>& preorder, int preStart, int preEnd, 
                vector<int>& inorder, int inStart, int inEnd, 
                unordered_map<int, int>& inMap) {
    if (preStart > preEnd || inStart > inEnd) return nullptr;
    TreeNode* root = new TreeNode(preorder[preStart]);
    int inRoot = inMap[root->val];
    int numsLeft = inRoot - inStart;
    root->left = build(preorder, preStart+1, preStart+numsLeft, 
                      inorder, inStart, inRoot-1, inMap);
    root->right = build(preorder, preStart+numsLeft+1, preEnd, 
                       inorder, inRoot+1, inEnd, inMap);
    return root;
}
//3.括号法建立
TreeNode* buildTree(const string& s, int& i) {
    if (i >= s.length() || s[i] == ')') return nullptr;
    
    TreeNode* node = new TreeNode(s[i++]);
    
    if (i < s.length() && s[i] == '(') {
        i++; //跳过 (
        node->left = buildTree(s, i);
        i++; //跳过 )

        if (i < s.length() && s[i] == ',') {
            i++;  // 跳过 ','
            node->right = buildTree(s, i);
            i++;  // 跳过 ')'
        }
    }
    
    return node;
}
//4.层序遍历法建立
TreeNode* buildTree(vector<string>& nodes) {
    if (nodes.empty() || nodes[0] == "#") return nullptr;
    TreeNode* root = new TreeNode(stoi(nodes[0]));
    queue<TreeNode*> q;
    q.push(root);
    int i = 1;
    while (!q.empty() && i < nodes.size()) {
        TreeNode* curr = q.front();
        q.pop();
        // 处理左子节点
        if (nodes[i] != "#") {
            curr->left = new TreeNode(stoi(nodes[i]));
            q.push(curr->left);
        }
        i++;
        // 处理右子节点,++之后就要判断是否<size了
        if (i < nodes.size() && nodes[i] != "#") {
            curr->right = new TreeNode(stoi(nodes[i]));
            q.push(curr->right);
        }
        i++;
    }
    return root;
}

//区分最小生成树和最短路径问题，最小生成树中某顶点到其余各顶点的路径不一定具有最短路径的性质。

// Prim算法求最小生成树
    int primMST() {
        vector<bool> inMST(V, false); // 是否在MST中
        vector<int> key(V, INT_MAX); // 每个顶点的最小权值
        vector<int> parent(V, -1); // 父节点

        // 优先队列用于选择最小权值的边
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;

        // 从顶点0开始
        int src = 0;
        pq.push({0, src});
        key[src] = 0;

        while (!pq.empty()) {
            int u = pq.top().second;
            pq.pop();

            if (inMST[u]) continue;

            inMST[u] = true;

            // 遍历所有邻接顶点
            for (auto& edge : adj[u]) {
                int v = edge.first;
                int weight = edge.second;

                if (!inMST[v] && weight < key[v]) {
                    key[v] = weight;
                    parent[v] = u;
                    pq.push({key[v], v});
                }
            }
        }
    }
 // Kruskal算法求最小生成树
    int kruskalMST() {
        vector<Edge> edges;
        // 收集所有边
        for (int u = 0; u < V; u++) {
            for (auto& edge : adj[u]) {
                int v = edge.first;
                int weight = edge.second;
                if (u < v) { // 避免重复添加
                    edges.push_back({u, v, weight});
                }
            }
        }

        // 按权值排序
        sort(edges.begin(), edges.end());

        // 并查集初始化
        vector<int> parent(V);
        for (int i = 0; i < V; i++) {
            parent[i] = i;
        }

        // 查找函数
        function<int(int)> find = [&](int x) {
            if (parent[x] != x) {
                parent[x] = find(parent[x]);
            }
            return parent[x];
        };

        int totalWeight = 0;
        int edgeCount = 0;

        // 遍历所有边
        for (auto& edge : edges) {
            int u = edge.u;
            int v = edge.v;
            int weight = edge.weight;

            int setU = find(u);
            int setV = find(v);

            if (setU != setV) {
                totalWeight += weight;
                edgeCount++;
                parent[setU] = setV; // 合并集合

                if (edgeCount == V - 1) break; // MST的边数为V-1
            }
        }

        return totalWeight;
    }

   // Dijkstra算法求最短路径
    vector<int> dijkstra(int src) {
        vector<int> dist(V, INT_MAX); // 距离数组
        dist[src] = 0;

        // 优先队列用于选择最小距离的顶点
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
        pq.push({0, src});

        while (!pq.empty()) {
            int u = pq.top().second;
            int d = pq.top().first;
            pq.pop();

            if (d > dist[u]) continue;

            // 遍历所有邻接顶点
            for (auto& edge : adj[u]) {
                int v = edge.first;
                int weight = edge.second;

                if (dist[u] + weight < dist[v]) {
                    dist[v] = dist[u] + weight;
                    pq.push({dist[v], v});
                }
            }
        }

        return dist;
    }
// Floyd-Warshall算法实现
void floydWarshall(vector<vector<int>>& dist) {
    int n = dist.size();
    
    // 中间节点k从0到n-1
    for (int k = 0; k < n; ++k) {
        // 源节点i从0到n-1
        for (int i = 0; i < n; ++i) {
            // 目标节点j从0到n-1
            for (int j = 0; j < n; ++j) {
                // 如果经过中间节点k的路径更短，则更新距离
                if (dist[i][k] != INF && dist[k][j] != INF && 
                    dist[i][j] > dist[i][k] + dist[k][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
}

// 1. 冒泡排序 - 重复交换相邻元素，将最大元素"冒泡"到末尾
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {//注意这里是<n-1，因为把前n-1个排好后，最后一个自然是对的
        bool swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break; // 优化：如果未发生交换，说明数组已有序
    }
}

// 2. 选择排序 - 每次从未排序部分选择最小元素，放到已排序部分末尾
void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int minIndex = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        if (minIndex != i) {
            swap(arr[i], arr[minIndex]);
        }
    }
}

// 3. 插入排序 - 将未排序数据插入到已排序序列的适当位置
void insertionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        // 将比key大的元素后移
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key; // 插入key
    }
}

// 4. 希尔排序 - 分组的插入排序，逐步缩小增量
void shellSort(vector<int>& arr) {
    int n = arr.size();
    // 初始增量为数组长度的一半，逐步减半
    for (int gap = n / 2; gap > 0; gap /= 2) {
        // 对每个增量进行插入排序
        for (int i = gap; i < n; i++) {
            int temp = arr[i];
            int j;
            // 直接插入排序，步长为gap
            for (j = i; j >= gap && arr[j - gap] > temp; j -= gap) {
                arr[j] = arr[j - gap];
            }
            arr[j] = temp;
        }
    }
}

// 5. 归并排序 - 分治法，递归拆分并合并有序数组
void merge(vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    vector<int> L(n1), R(n2);

    // 复制数据到临时数组
    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    // 合并临时数组回原数组
    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k] = L[i];
            i++;
        } else {
            arr[k] = R[j];
            j++;
        }
        k++;
    }

    // 复制剩余元素
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void mergeSort(vector<int>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        // 递归拆分
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        // 合并
        merge(arr, left, mid, right);
    }
}

// 6. 快速排序 - 分治法，选择基准值并分区
int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high]; // 选择最后一个元素作为基准
    int i = low - 1;

    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return i + 1; // 返回基准值的正确位置
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        // 递归排序左右两部分
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

// 7. 堆排序 - 利用最大堆特性进行排序
void heapify(vector<int>& arr, int n, int i) {
    int largest = i; // 初始化根节点
    int left = 2 * i + 1;
    int right = 2 * i + 2;

    // 如果左子节点比根大，则更新最大值
    if (left < n && arr[left] > arr[largest])
        largest = left;

    // 如果右子节点比根大，则更新最大值
    if (right < n && arr[right] > arr[largest])
        largest = right;

    // 如果最大值不是根节点，则交换并继续调整
    if (largest != i) {
        swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}
void heapSort(vector<int>& arr) {
    int n = arr.size();

    // 构建最大堆
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    // 逐个提取元素
    for (int i = n - 1; i > 0; i--) {
        // 将当前根节点（最大值）移到末尾
        swap(arr[0], arr[i]);
        // 在缩小的堆上调整
        heapify(arr, i, 0);
    }
}
```
