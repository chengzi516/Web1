---
title: 【linux】进程间如何实现通信
date: 2023-11-18 16:19:01
tags:
- 进程
- linux
- 通信
categories:
- linux
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/linux.jpg
ai: true
---

# 概述

## 什么是通信

进程通信是指不同进程之间在操作系统环境下`交换数据、共享信息或者进行协作`的过程。多个进程间可能需要相互协作来完成复杂的任务。

>其本质可以简单的理解为让多个进程看到同一份"资源"。

这个资源一般指的是`特定形式的内存空间`，其属于操作系统管辖。这个资源`不属于`任一个进程，否则这会破坏进程的独立性。访问资源也就等同于访问操作系统。所以在通信过程中不过避免的会使用到`系统调用接口`。
数进程通信允许进程之间传递数据，可以是简单的消息、文件、共享内存或者其他形式的信息,也涉及到进程之间的同步，确保它们按照预期的顺序执行，并协同工作以完成特定的任务。

## 通信目的
其目的可以总结为三点：
资源共享： 允许进程`共享`信息和资源，如文件、内存等，以便彼此访问和使用这些资源。
协同工作： 多个进程可能需要合作完成一个大型任务，进程通信允许它们之间进行有效的`协调和合作`。
并发控制： 控制并发进程的访问，确保数据的一致性和完整性，避免竞态条件和数据损坏。


## 通信方式


>管道
1. 匿名管道pipe
2. 命名管道

>System V IPC
1. System V 消息队列
2. System V 共享内存
3. System V 信号量

>POSIX IPC

1. 消息队列
2. 共享内存
3. 信号量
4. 互斥量
5. 条件变量
6. 读写锁

本篇博客主要讲解以上三种通信方式。

# 管道

## 概念

在操作系统中，管道是一种用于进程间通信的机制，也是unix中最古老的通信方式，其`允许一个进程的输出成为另一个进程的输入`，从而实现单向的通信。

```
  +------------------+       +------------------+
  |   Process 1     | -----> |   Process 2     |
  +------------------+     / +------------------+
           |              /
         (output)      (input)
           |            /        
          \|/          /
  +------------------+
  |      Pipe        |
  +------------------+
         

```

1. 单向通信：管道是单向的，即数据只能在一个方向上流动。
2. 进程关系：通常，管道是在具有`父子关系`的两个相关进程之间创建的。

管道的创建：
```bash
# 使用命令行创建管道
$ command1 | command2
```

command1 的输出将作为 command2 的输入。


## 原理

在进程运行时，会建立对应的task_struct，也就是PCB，其中有一个指针，指向一个file_struct，其本质是一个数组，通过访问数组下标，就能取得对应的文件，且默认会打开三个输入输出流，stdin，stdout，stderr，分别占据下标0，1，2。新建或要打开文件则从下标3开始，这在之前的博客里就已经详细介绍过了。

有了下标，操作系统就能取得相应的文件，就能获取inode，读写方法，以及缓冲区等等。
例如我们打开一个文件，其下标按顺序为3。

<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A11.png'>

当该进程进行fork操作，创建一个子进程，相应的file_struct也会被复制一份给子进程，且指向的仍然是`父进程的四个文件`。
想到开篇的那句话了吗？通信的前提是让多个进程看到同一份文件。在管道操作中，就需要借助fork来实现这一特性，也就是`二者必须有关系`！

且读写的struct是分开的。
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A12.png'>
更详细的拆分一下：
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A13.png'>

之所以要分开进行读写，是因为：


## pipe接口

原型
```c
#include <unistd.h>
int pipe(int pipefd[2]);
```

pipe() 函数创建一个管道，并将两个文件描述符存储在传入的数组`pipefd`中。pipefd[0] 用于`读取`数据，pipefd[1] 用于`写入`数据。

管道是单向的，只能支持单向数据流动。通常用于父子进程或者兄弟进程之间的通信。
管道在内存中创建一个缓冲区，数据暂存于其中。缓冲区的大小有限，这会对读写造成限制。

>遵循先进先出（FIFO）原则： 写入管道的数据会按照先进先出的顺序被读取。

如何利用管道进行进程间通信？
创建管道： 使用 pipe() 函数创建管道，获取用于读和写的文件描述符。
fork() 创建子进程： 在父进程中创建子进程，子进程继承了父进程的文件描述符。
关闭不需要的文件描述符： 在父子进程中关闭不需要的文件描述符。例如，父进程关闭管道的读端（pipefd[0]），而子进程关闭管道的写端（pipefd[1]）。
使用 read() 和 write() 进行通信： 父子进程分别使用 read() 和 write() 来进行数据的读取和写入，这也和前面所说的通信是操作系统在调控，所以得用系统调用接口来做相吻合。
关闭管道： 当通信完成后，关闭管道的文件描述符来释放资源。

>请注意：
pipe() 可能失败，返回 -1。此时，可以通过检查 errno 来获取具体的错误信息。
管道的容量是有限的，如果写入速度快于读取速度，可能导致写入进程阻塞（等待缓冲区有空间）。
管道一旦关闭，对端不再可写或可读，读取端会收到一个EOF（文件结束符）。
 
示例代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    int pipefd[2];
    pid_t pid;
    char message[] = "Hello, Pipe!";

    if (pipe(pipefd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    pid = fork();

    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid > 0) { // Parent process
        close(pipefd[0]); // Close the read end of the pipe

        write(pipefd[1], message, sizeof(message));
        close(pipefd[1]); // Close the write end of the pipe
    } else { // Child process
        close(pipefd[1]); // Close the write end of the pipe

        char buffer[100];
        int nbytes = read(pipefd[0], buffer, sizeof(buffer));
        printf("Received message in child: %.*s\n", nbytes, buffer);

        close(pipefd[0]); // Close the read end of the pipe
    }

    return 0;
}
```

从上面的代码还能看出，管道是面向`字节流`的！
管道中会出现四种情况：
1. 读写正常，而管道为空，则读阻塞。
2. 读写正常，而管道已满，则写阻塞。
3. 读正常，而写关闭，则读到0，也就是文件结尾。
4. 写正常，而读关闭，操作系统会杀死写进程。

