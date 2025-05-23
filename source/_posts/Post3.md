---
title: 学习日记 | 日记是我与你重逢的唯一方式
date: 2025-04-26
tags:
  -日记
---
**日记也是那些被时光掩埋的回忆重新被唤醒的地方**
<!-- more -->  
# csapp学习
**异常控制流：异常与进程**
## 一、异常控制流（ECF）的核心概念
1. 为什么需要ECF?
传统的控制流（分支/函数调用）无法处理外部事件（硬件中断、系统请求）；
而ECF让程序能够响应系统状态变化（键盘输入、网络数据到达、程序错误）
2. ECF的层级
硬件级（异常中断陷阱）、操作系统级、应用级（非本地跳转）。
## 二、异常详解
1. 本质是系统事件触发的控制流强制跳转（硬件+操作系统协作实现）
2. 异常分类表
中断：由外部设备触发，会返回下一条指令。eg--定时器中断、键盘输入
陷阱：程序主动请求，会返回下一条指令。eg--系统调用（open，fork）
故障：可恢复错误，会重试或终止。eg--缺页异常（page fault）
中止：不可恢复错误，会直接终止程序。eg--内存校验错误、非法指令
3. 关键机制
**异常表**  事件编号->处理函数映射表
**系统调用** 通过陷阱实现（syscall指令）
## 三、进程（process）
1. 进程是什么？
**定义** 程序运行的实例，其中程序是静态代码而进程是动态执行。
**两大抽象**
逻辑控制流：每个进程独占CPU的假象（时间片轮转实现）
私有地址空间：每个进程独占内存的假象（虚拟内存实现）
2. 进程状态机
运行态--阻塞态（等待I/O）
运行态--停止态（被暂停，如Ctrl-Z）
终止态（需父进程回收资源，否则僵尸进程）
## 四、进程控制的关键操作
1. 进程创建：fork()
**一次调用，两次返回** 父进程返回子进程PID，子进程返回0
**子进程复制父进程完整状态** 包括代码、数据、堆栈、文件描述符
**代码示例**
```c
int x = 1;
pid_t pid = fork();
if (pid == 0) {  // 子进程
    printf("Child: x=%d\n", ++x); 
} else {         // 父进程
    printf("Parent: x=%d\n", --x);
}
```
// 可能输出：Parent: x=0  Child: x=2
2. 进程终结：exit()
**主函数return**
**显式调用exit()**
**收到终止信号**（如SIGSEGV）
3. 进程回收：wait()
父进程阻塞等待子进程结束，避免僵尸进程
```c
int status;
pid_t wpid = wait(&status);
if (WIFEXITED(status)) {
    printf("Child %d exited with code %d\n", wpid, WEXITSTATUS(status));
}
```
4. 程序加载：execve()
**特点** 
替换当前进程的代码段（保留PID和文件描述符）  
调用后不返回
```c
char *argv[] = {"/bin/ls", "-l", NULL};
char *envp[] = {"PATH=/usr/bin", "USER=alice", NULL};
execve(argv[0], argv, envp);
```
## 五、多进程执行模型
1. 并发执行
单核：通过时间片轮转模型并行  
多核：真实并行执行
2. 上下文切换流程
保存当前进程寄存器状态-- 内存  
调度器选择下一个进程  
加载新进程的寄存器状态+切换地址空间
## 六、难点与经典问题
1. fork()双返回本质
子进程从fork()返回出继续执行而不会从头开始  
通过写时复制优化内存复制开销
2. 僵尸进程vs孤儿进程
**僵尸** 子进程已经终止但没有被父进程回收，也就是wait()未调用
**孤儿** 父进程先于子进程终止
3. 并发执行的不确定性
父子进程执行顺序不可预测（依赖调度器）  
需要同步机制（如信号量、管道）解决竞态条件

# 数据结构学习
## 并查集
**直接上代码**
```cpp
class DSU {
private:
    vector<int> parent;
    vector<int> rank; // 秩（树的高度）
public:
    DSU(int n) : parent(n), rank(n, 1) {
        for (int i = 0; i < n; ++i) 
            parent[i] = i;
    }

    // 查找 + 路径压缩
    int find(int x) {
        return parent[x] == x ? x : parent[x] = find(parent[x]);
    }

    // 按秩合并
    void unite(int x, int y) {
        x = find(x), y = find(y);
        if (x == y) return; // 已在同一集合

        if (rank[x] < rank[y]) {
            parent[x] = y; // 将矮树合并到高树下
        } else {
            parent[y] = x;
            if (rank[x] == rank[y]) 
                rank[x]++; // 高度相同则合并后高度+1
        }
    }
};
```
## 图
**概念**
关注几组概念：连通图和非连通图，有向图和无向图，入度和出度
关键点在两种存储结构：
1. 邻接矩阵存储  
2. 邻接表存储  
感觉临界表有难度：是一种顺序分配和链式分配组合的存储方式。**！！！**
# 时间管理大师
对于今天晚上能在一个时间段同时做很多件事情感到很满意，江城战记任务做了，带朋友在武大逛了（并且坐了我最最最爱的校巴），凌波门音乐会听了，晚上辩论讨论准备了，凌晨金铲铲吃了。  
嘿嘿，充实的一晚上~
# 辩论备赛







