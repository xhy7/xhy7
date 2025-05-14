---
title: 学习日记 | CSAPP信号
date: 2025-05-13
tags:
  -学习
  -CSAPP
---

首先要清楚，什么是信号？**信号**是一种更高层次的软件形式的异常，它允许进程和内核中断其他进程。
每种信号都对应于某种系统事件。信号大概可以分为两类：
1. 对应底层的硬件异常，通常由内核异常处理程序处理，对用户进程不可见。
2. 对应于内核或其他用户进程中较高层的软件事件。
## 信号术语
1. 发送信号：内核通过更新目的进程上下文的某个状态，发送一个信号给目的进程
2. 接收信号：进程可以忽略、终止或通过执行信号处理程序的用户层函数来捕获这个信号
![信号处理](/img/errorsolve.png)
3. 待处理信号：待处理信号是指发出但没有被接收的信号。进程可以有选择地阻塞接收某种信号。当一种信号被阻塞，它仍可以发送，但是产生的待处理信号不会被接收。一个待处理信号最多只能被接收一次。内核为每个进程在 pending 位向量中维护着待处理信号的集合，在 blocked 位向量中维护着被阻塞的信号集合。
## 发送信号
Unix 系统提供了大量向进程发送信号的机制。这些机制都是基于进程组（process group）的概念。
### 进程组
每个进程都只属于一个进程组，进程组由一个正整数进程组ID来标识。
可以使用 getpgrp 函数获取当前进程的进程组 iD，可以使用 setpgid 函数改变自己或其他进程的进程组。
```c
#include<unistd.h>
pid_t getpgrp(void);   // 返回调用进程的进程组 ID
int setpgid(pid_t pid, pid_t pgid);     // 将进程 pid 的进程组改为 pgid。若成功返回 0，错误返回 -1。
        // 如果 pid=0，就表示使用当前进程的 pid，如果 pgid=0，就表示要将使用第一个参数 pid 作为进程组 ID。        
```
### /bin/kill程序发送信号
可以用kill程序向另外的进程发送任意的信号，正的PID表示发送到对应进程，负的PID表示发送到每个进程组的每个进程
### 用键盘发送信号
shell用**作业**这个抽象概念来表示为对一条命令行求值而创建的进程。任何时刻，至多只有一个前台作业，后台作业可以有多个。
shell 为每个作业创建一个独立的进程组，进程组 ID 通常取作业中父进程中的一个。
![前台和后台进程组](/img/前后台.png)
键盘上输入Ctrl+C 会导致内核发送一个SIGINT信号到从前台进程组中的每个进程，会终止前台作业
输入 Ctrl+Z 会发送一个SIGTSTP信号到前台进程组中的每个进程，会停止（挂起）前台作业
### kill函数发送信号
进程可以通过调用kill函数发送信号给其他进程（包括自己）
```c
#include<sys/types.h>
#include<signal.h>
int kill(pid_t pid, int sig);  //若成功则返回 0，若错误则返回 -1
```
1. pid>0，kill函数发送信号号码sig给进程pid
2. pid=0，kill 函数发送信号 sig 给调用进程所在进程组中的每个进程
3. pid<0，kill 函数发送信号 sig 给进程组 | pid | (pid 的绝对值)中的每个进程。 
### 用alarm函数发送信号
进程可以通过调用alarm函数向自己发送SIGALRM信号
```c
#include<unistd.h>
unsigned int alarm(unsigned int secs);  //返回前一次闹钟剩余的描述，如果以前没有设定闹钟，就返回 0。
```
alarm函数安排内核在secs秒后发送一个SIGALRM信号给调用进程。如果secs=0，则不会调度安排新的闹钟。
在任何情况下，对 alarm 的调用都将取消任何待处理的闹钟，并返回任何待处理的闹钟在被发送前还剩下的秒数。如果没有待处理的闹钟，就返回 0。
### 总结
1. 内核给进程/进程组发送信号
2. 使用bin/kill程序
3. 调用kill函数
4. 进程调用alarm函数给自己发送SIGALRM信号
5. 键盘按键发送信号
## 接收信号
内核把进程p从内核模式切换到用户模式时（从系统调用返回），会检查进程p的未被阻塞得待处理信号的集合。
如果集合为空，内核就将控制传递到 p 的逻辑控制流中的下一条指令；如果集合非空，内核就选择集合中的某个信号（通常是最小的 k），并强制 p 接收信号 k。
每个信号类型都有一个预定义得默认行为：
1. 进程终止
2. 进程终止并且转储内存
3. 进程停止（被挂起）直到SIGCONT信号重启
4. 进程忽略该信号
进程可以通过signal函数修改和信号相关联的默认行为，其中SIGSTOP和SIGKILL默认行为不能被修改
signal函数让程序捕捉到信号，并指定当收到特定信号时要执行的操作。
```c
#include<signal.h>  
typedef void (*sighandler_t)(int); 
sighandler_t signal(int signum, sighandler_t handler); //若成功返回指向前次处理程序的指针，若出错则返回 SIG_ERR（不设置 errno）。
```
signal函数接受两个参数：
1. signum，要处理的信号类型。如SIGINT,SIGTERM,SIGKILL,SUFAKRM
2. handler，函数指针，指向要执行信号的处理函数
### 改变和signum关联的行为
1. 如果 handler 是 SIG_IGN，那么忽略类型为 signum 的信号。SIG_IGN 是 signal.h 中定义的一个宏。
2. 如果 handler 是 SIG_DFL，那么类型为 signum 的信号行为恢复为默认行为。SIG_DEF 是 signal.h 中定义的一个宏。
3. 如果 hanlder 是用户定义的函数的地址，这个函数就被称为信号处理程序。只要进程接收到一个类型为 signum 的信号，就会调用这个程序。
### 信号处理程序中断
信号处理程序可以被其他信号处理程序中断
![信号处理程序的中断](/img/kill.png)
### 信号处理程序代码
```c
#include "csapp"
void sigint_handler(int sig)  //定义了一个信号处理程序
{
    printf("Caught SIGINT!\n");
    exit(0);
}
int main()
{
    /* Install the SIGINT handler */
    if(signal(SIGINT, sigint_handler)) == SIGERR)
        unix_error("signal error");
    pause(); // wait for the receipt of signal
    return 0;    
}
```
### 代码细节分析
这里的pause函数是暂停当前进程的执行，直到进程接收到一个信号。这里和signal函数配合使用，使得程序在pause处暂停执行，不会立刻结束而是等待信号的到来。
如果没有pause函数，程序会执行完signal函数后直接继续执行main函数下一条return 0语句结束。这样会导致程序在结束之前没有足够时间让信号发生并被处理。pause函数的存在阻塞了进程，让程序处于等待状态，以便能够接受并且处理SIGINT信号。
## 阻塞和解除阻塞信号
有两种阻塞信号的机制
1. 隐式阻塞机制。内核默认阻塞任何当前处理程序正在处理信号类型的待处理信号。
2. 显式阻塞机制。程序可以使用sigprocmask函数和辅助函数明确地阻塞和解除阻塞选定的信号。
### sigprocmask函数
可以改变当前阻塞的信号集合（blocked位向量），具体和how值有关：
1. SIG_BLOCK：把set中的信号增添到blocked中：
blocked=blocked|set
2. SIG_UNBLOCK：从blocked中删除set中的信号：
blocked=blocked&~set
3. SIG_SETMASK：
block=set
如果oldset非空，blocked位向量之前的值保存在oldset中
### 辅助函数
辅助函数用来对 set 信号集合进行操作：
1. sigemptyset 初始化 set 为空集合；
2. sigfillset 把每个信号都添加到 set 中；
3. sigaddset 把信号 signum 添加到 set 中；
4. sigdelset 把信号 signum 从 set 中删除。如果 signum 是 set 的成员返回 1，不是返回 0。
### 临时阻塞SIGINT信号的代码例
```c
sigset_t mask, prev_mask;
Sigemptyset(&mask);   
Sigaddset(&mask, SIGINT);  //将 SIGINT 信号添加到 set 集合中

Sigprocmask(SIG_BLOCK, &mask, &prev_mask);  //阻塞 SIGINT 信号，并把之前的阻塞集合保存到 prev_mask 中。
...  //这部分的代码不会被 SIGINT 信号所中断
Sigprocmask(SIG_SETMASK, &prev_mask, NULL); //恢复之前的阻塞信号，取消对 SIGINT 的阻塞
```
## 编写信号处理程序
信号处理是Linux系统编程最棘手的问题，复杂属性在:
1. 处理程序与主程序和其他信号处理程序并发运行，共享同样的全局变量，可能和主程序与其他处理程序互相干扰。
2. 如何接收信号及何时接收信号的规则常常有违人的直觉。
3. 不同的系统有不同的信号处理语义。
### 具体代码
```c
#include "csapp"
void sigint_handler(int sig)  //定义了一个信号处理程序
{
    printf("Caught SIGINT!\n");
    exit(0);
}
```
### SIGCHLD/注意事项
要注意：如果存在一个未处理的信号 k，那表明至少有一个 k 信号到达了，实际上可能不止一个，以下是忽略了这一点的错误示例：
**SIGCHLD：当子进程停止或终止的时候，会给父进程发送此信号，默认处理方式是忽略。**
先来看代码：
```c
void handler1(int sig)
{
    int olderrno = errno;  //保存 errno
    
    if((waitpid(-1, NULL, 0)) < 0)   //回收子进程
        sio_error("waidpid error");
    Sio_puts("Handler reaped child\n");
    Sleep(1);  // Sleep(1) 用来比喻回收子进程后要做的其他清理工作
    errno = olderrno;  //恢复 errno
}

int main()
{
    int i, n;
    char buf[MAXBUF];
    
    //将 handler1 函数设置为信号 SIGCHLD 的处理程序
    if(signal(SIGCHLD, handler1) == SIG_ERR)   //如果 signal 函数失败会返回 SIG_ERR
        unix_error("signal error");
        
    //创建子进程
    for(int i=0; i<3; i++)
    {
        if(Fork() == 0){                       //Fork() 是封装了错误处理的 fork() 函数
            printf("Hello from child %d\n", (int)getpid());
            exit(0);
        }
    }
    
    //父进程等待并处理终端输入————这部分就是父进程在回收子进程时想做的其他事情。
    if((n = read(STDIN_FILENO, buf, sizeof(buf))) < 0)
        unix_error("read");
    printf("Parent processing input\n");
    while(1);  // while(1) 用来比喻父进程一直在处理输入。
    
    exit(0);    
}
```
### 函数分析与理解
**理解**
1. 信号处理程序不需要显式调用，当该进程收到了相关信息，会在自动调用信号处理程序。
2. 信号处理程序与主程序共享同样的全局数据，所以应该是线程级并发？
**错误分析**
这里三个子进程，每个终止时都会给父进程发送SIGCHLD信号，父进程收到后就会调用一次信号处理程序，使用waitpid来回收僵尸进程。但是因为信号不会排队，如果在信号处理程序处理第一个僵尸子进程的时候，后两个子进程也终止了，那么第二个终止的子进程发给父进程的 SIGCHLD 信号会放到父进程的待处理信号集中，而第三个终止的子进程发送的 SIGCHLD 信号则会被丢弃。这样父进程就只能检测到两次SIGCHLD 信号，这样就只能回收两个子进程了。
**问题修正**
**问题分析**：问题在于信号不会排队，即父进程检测到一个 SIGCHLD 信号时，可能终止的子进程不止一个。
**解决方式**：修改信号处理程序，让它在回收子进程时不要只回收一个，而是一次性回收所有僵死的子进程。
```c
void handler2(int sig)
{
    int olderrno = errno;
    
    while(waitpid(-1, NULL, 0) > 0){         //循环回收所有僵死子进程。
        Sio_puts("Handler reaped child\n");
    }
    if(errno != ECHILD)  //当循环结束时，waitpid 的返回值为 -1。如果是因为已经没有了子进程而返回 -1，这时 errno 就被设置为了 ECHILD，这是正确的结果。如果不是 ECHILD，则说明出错了。
        Sio_error("waitpid error");
    Sleep(1);
    errno = olderrno;    
}
```
### waitpid函数
这里对这个函数理解还不够，插入来讲讲它。
作用是等待一个或多个子进程的状态改变，并获取其退出状态等信息。
pid_t waitpid(pid_t pid, int *status, int options);
这里的status是一个指向整数的指针，用于存储子进程的退出状态信息。通过WIFEXITED、WEXITSTATUS等宏可以从这个整数中提取出具体的退出状态信息。如果不关心子进程的退出状态，可以将status设置为NULL。
这里的options提供了一些额外的选项，常用的有WNOHANG，指定该选项后waitpid不会阻塞父进程，无论子进程是否结束，它都会立即返回。如果没有子进程退出，它会返回0；如果有子进程退出，它会返回子进程ID。
来看一段综合代码来理解理解：
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid;
    int status;

    pid = fork();
    if (pid < 0) {
        perror("fork");
        exit(EXIT_FAILURE);
    } else if (pid == 0) {
        // 子进程
        printf("子进程开始执行，PID: %d\n", getpid());
        sleep(5);
        exit(0);
    } else {
        // 父进程
        printf("父进程等待子进程结束，子进程PID: %d\n", pid);
        // 等待子进程结束，阻塞方式
        pid_t wpid = waitpid(pid, &status, 0);
        if (wpid == -1) {
            perror("waitpid");
            exit(EXIT_FAILURE);
        }
        if (WIFEXITED(status)) {
            printf("子进程正常结束，退出状态码: %d\n", WEXITSTATUS(status));
        }
    }

    return 0;
}
```
**分析**：父进程fork创建了一个子进程，用waitpid等子进程结束。在子进程中，它会休眠 5 秒后正常退出。父进程在waitpid返回后，通过WIFEXITED和WEXITSTATUS宏来判断子进程是否正常结束，并获取其退出状态码。
## 同步流以避免讨厌的并发错误
如何编写读写相同存储位置的并发流程序是一个难题。
**解决方式**：以某种方式同步并发流，从而得到最大的可行的交错集合，每个可行的交错都能得到正确的结果。
### 具有细微同步错误的shell程序
父进程在一个全局作业列表中记录着它的当前子进程，每个作业一个条目。当父进程创建一个子进程后，就把这个子进程添加到作业列表中。当父进程在 SIGCHLD 处理程序中回收一个僵死子进程时，它就从作业列表删除这个子进程。
```c
void handler(int sig)
{
    int olderrno = errno;
    sigset_t mask_all, prev_all; //信号集合
    pid_t pid;
    
    Sigfillset(&mask_all); //把每个信号都添加到信号集合中
    while((pid = waitpid(-1, NULL, 0)) > 0)//while循环处理多个子进程结束的情况
    {
        Sigprocmask(SIG_BLOCK, &mask_all, &prev_all); //删除作业前先阻塞所有信号，保证执行的时候不会被其他信号所打断
        deletejob(pid);
        Sigprocmask(SIG_SETMASK, &prev_all, NULL); //删除完后恢复之前的阻塞信号集
    }
    if(errno != ECHILD)
        Sio_error("waitpid error");
    errno = olderrno;    
}

int main(int argc, char **argv)
{
    int pid;
    sigset_t mask_all, prev_all;
    
    Sigfillset(&mask_all);
    Signal(SIGCHLD, handler);
    initjobs();   //初始化作业列表
    
    while(1){
        if((pid = Fork()) == 0){//子进程中fork返回值为0
            Execve("/bin/date", argv, NULL);
        }
        Sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
        addjob(pid);
        Sigprocmask(SIG_SETMASK, &prev_all, NULL);
    }
    exit(0);
}
```
### 事件序列
1. 父进程执行fork，内核调度创建的子进程运行
2. 父进程运行前，子进程终止，变成一个僵尸进程，内核传递一个SIGCHLD信号给父进程
3. 父进程变为可运行状态，正式执行之前，内核注意到有未处理的SIGCHLD，通过在父进程中运行信号处理程序来接受这个信号
4. 信号处理程序回收终止的子进程，并调用 deletejob，但实际上 deletejob 函数什么也不会做，因为父进程还没有把这个子进程添加到作业列表中。
5. 信号处理程序运行完毕后，内核运行父进程，父进程从 fork 返回，调用 addjob 函数错误地把这个已经终止并被回收掉的子进程添加到作业列表中。
在这个序列中，main和信号处理流之间的错误交错，让addjob前调用了deletejob。
### 经典同步错误：竞争
main 函数中调用 addjob 和信号处理程序中调用 deletejob 之间存在竞争，如果 addjob 赢得竞争，结果就是正确的，反之，结果就是错误的。
### 消除竞争
通过在调用fork前，阻塞SIGCHLD信号，在调用addjob后取消阻塞，保证了在子进程被添加到作业列表后才可能回收该子进程
**注意：**子进程会继承父进程的被阻塞集合，所以必须在调用 execve 之前，解除子进程中阻塞的 SIGCHLD 信号。
```c
void handler(int sig)
{
    int olderrno = errno;
    sigset_t mask_all, prev_all;
    pid_t pid;
    
    Sigfillset(&mask_all);
    while((pid = waitpid(-1, NULL, 0)) > 0){
        Sigprocmask(SIG_BLOCK, mask_all, prev_all);
        deletejob(pid);
        Sigprocmask(SIG_SETMASK, prev_all, NULL);
    }
    if(errno != ECHILD)
        Sio_error("waitpid error");
    errno = olderrno;
}
int main(int argc, char **argv)
{
    int pid;
    sigset_all mask_all, mask_one, prev_one;
    
    Sigfillset(&mask_all);
    Sigemptyset(&mask_one);
    Sigaddset(&mask_one, SIG_CHLD);
    Signal(SIG_CHLD, handler);
    initjobs();
    
    while(1) {
        Sigprocmask(SIG_BLOCK, &mask_one, &prev_one);
        if((pid = Fork()) == 0){
            Sigprocmask(SIG_SETMASK, &prev_one, NULL);
            Execve('/bin/date', argv, NULL);
        }
        Sigprocmask(SIG_BLOCK, &mask_all, NULL);
        addjob(pid);
        Sigprocmask(SIG_SETMASK, &prev_one, NULL);
    }
    exit(0);
}
```
## 显式地等待信号
主程序需要显式地等待某个信号处理程序运行。shell在创建一个前台作业的时候，接收到下一条命令之前，它必须等待作业终止，被SIGCHLD处理程序回收。
```c
#include "csapp.h"

volatile sig_atomic_t pid;

void sigchld_handler(int s)//信号处理函数
{
    int olderrno = errno;
    pid = waitpid(-1, NULL, 0); //-1表示等待任意子进程结束，SIG_CHLD 的信号处理程序修改了 pid 的值。
    errno = olderrno;
}

void sigint_handler(int s) {}

int main(int argc, char **argv)
{
    sigset_t mask, prev;
    //mask用域存储要阻塞的信号，prev用于保存之前的信号屏蔽状态
    Signal(SIGCHLD, sigchld_handler);
    Signal(SIGINT, sigint_handler);
    Sigemptyset(&mask);
    Sigaddset(&mask, SIGCHLD);
    
    while(1){
        Sigprocmask(SIG_BLOCK, &mask, &prev);  //阻塞 SIGCHLD 信号
        if(Fork() == 0)
            exit(0);
        //子进程中直接结束
        
        pid = 0; //在阻塞 SIG_CHLD 信号期间，将 pid 置为 0。
        Sigprocmask(SIG_SETMASK, &prev, NULL); //取消阻塞 SIGCHLD 信号
        
        while(!pid) ;  //这个无限循环就是在显式地等待作业终止并被信号 SIGCHLD 的处理程序回收。这里的等待过程一直在占用 CPU，实际上是很浪费资源的。
        printf(".")        
    }
    exit(0);
}
```
### 避免while(!pid)等待过程浪费资源
1. 在循环体内插入pause
while(!pid) pause();  
//有问题，有严重的竞争条件：如果在 while 测试后和 pause 之前收到 SIGCHLD 信号，pause 会永远暂停。
2. 在循环体内插入sleep
while(!pid) sleep(1);
//有问题，太慢了，每次循环检查后需要等很长时间才会再次检查循环条件。
**正确解决**
用sigsuspend函数
它等价于下面代码地原子版本，暂时用mask替换当前的阻塞集合，然后挂起该进程，直到收到一个信号
```c
sigprocmask(SIG_SETMASK, &mask, &prev);
pause();
sigprocmask(SIG_SETMASK, &prev, NULL);
```
使用sigsuspend函数来修改后的正确代码：
```c
int main(int argc, char **argv)
{
    sigset_t mask, prev;
    
    Signal(SIGCHLD, sigchld_handler);
    Signal(SIGINT, sigint_handler);
    Sigemptyset(&mask);
    Sigaddset(&mask, SIGCHLD);
    
    while(1){
        Sigprocmask(SIG_BLOCK, &mask, &prev);  //阻塞 SIGCHLD 信号
        if(Fork() == 0)
            exit(0);
        pid = 0;
        while(!pid)
            sigsuspend(&prev);  //在 sigsuspend 中取消阻塞 SIGCHLD 信号并等待作业终止，等作业终止后恢复之前的阻塞集合。
        Sigpromask(SIG_SETMASK, &prev, NULL);  //取消阻塞 SIGCHLD 信号。
        printf(".");        
    }
    exit(0);
}
```
## shell程序构成
一个极简的 shell 程序包括以下几个函数：main 函数、eval 函数、parseline 函数、buildin 函数，它们的各自的主要职责如下：
1. main：shell 程序的入口点，职责：循环从标准输入读取命令行字符串并调用 eval 函数解析并执行命令行字符串。
2. eval：解析并执行命令行字符串。职责：首先调用 parseline 函数解析命令行字符串，然后使用buildin 函数检查是否为内置命令，不是的话要生成一个进程（作业）来完成此命令，还要根据情况回收相应进程。
3. parseline 函数：解析命令行字符串。职责：根据空格拆分命令行字符串，构造 argv 向量。
4. buildin 函数：检查命令是否为内置命令，如果是的话直接调用相应函数，不是的话返回交给 eval 函数负责。
## 写在最后
感觉这是我写的最长的一篇学习blog了，这还只是csapp一门科目的一章内容的一部分，就要花这么这么多的时间和精力来学习。第8章异常控制流中还有8.1异常、8.2进程，8.3错误处理和8.4进程控制，这些都还不太熟悉。啊啊啊太难了。
任重而道远！