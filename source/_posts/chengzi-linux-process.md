---
title: 【linux】初识进程
date: 2023-07-15 12:19:45
tags:
- linux
- 进程
categories:
- linux
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/linux.jpg
ai: true 


---

# 操作系统
在了解进程前，还得介绍一下操作系统。

## 概念
操作系统是计算机系统中的一种重要软件，它是计算机`硬件和软件`之间的桥梁，负责管理计算机系统的各种资源，如CPU、内存、输入输出设备等。操作系统可以被看作是计算机系统的管理者，它控制和协调计算机系统中各个部件的工作，使得应用程序能够正确地运行。

操作系统通常包括以下几个组成部分：

1. 内存管理：负责管理计算机系统的内存资源，包括内存的分配、释放和保护等。
2. `进程管理`：负责管理计算机系统中的进程，包括进程的创建、调度、同步和通信等。
3. 文件系统：负责管理计算机系统中的文件和目录，包括文件的读写、创建、删除和保护等。
4. 输入输出管理：负责管理计算机系统中的输入输出设备，包括输入输出的缓存、设备的分配和释放等。

维基百科这样总结操作系统：`操作系统`（Operating System，缩写：OS）是一组`主管并控制`计算机操作、运用和运行硬件、软件资源和提供公共服务来组织用户交互的相互关联的系统软件程序，同时也是计算机系统的`内核`与基石。操作系统需要处理如管理与配置内存、决定系统资源供需的优先次序、控制输入与输出设备、操作网络与管理文件系统等基本事务。操作系统也提供一个让用户与系统交互的操作界面。
操作系统的类型非常多样，不同机器安装的操作系统可从简单到复杂，可从移动电话的嵌入式系统到超级电脑的大型操作系统。许多操作系统制造者对它涵盖范畴的定义也不尽一致，例如有些操作系统集成了`图形用户界面`，而有些仅使用`命令行界面`，将图形用户界面视为一种非必要的应用程序。
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84.png'>
那么总的概括下来呢，计算机系统中都包含且存在的一个基本的`程序集合`，就被称为操作系统。其根本目的就在于为用户`提供一个相对方便的操作环境，管理计算机的软硬件资源`。

## 发展

在20世纪40年代初期，计算机系统还处于非常初级的阶段，计算机的应用也非常有限。直到1945年，约翰·冯·诺伊曼提出了`冯洛伊曼体系`，这一体系彻底改变了计算机系统的结构和设计，开创了计算机技术的新时代。

随着计算机技术的不断发展，计算机系统的规模和复杂度也不断增加，操作系统的概念也随之出现。20世纪60年代初期，IBM公司发布了第一款商用操作系统——OS/360，这标志着操作系统开始进入商用化阶段。

从此以后，操作系统和冯洛伊曼体系的发展就开始了新的篇章。随着计算机技术的不断进步，操作系统和冯洛伊曼体系也不断更新和演化，逐步适应了现代计算机系统的各种应用场景和技术需求。
<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E5%86%AF%E8%AF%BA%E4%BE%9D%E6%9B%BC%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84.png">[冯诺依曼体系结构设计概念]

## 内存
我们如今所能使用的`大部分计算机`，都遵循这冯诺依曼体系。但冯诺依曼结构并非这一节的重点，我们只需要知道，memory指的是存储器，也就是常谈的`内存`！

>所有程序都是必须放到内存中去进行的！

这是因为计算机的`CPU只能直接访问内存中的数据和指令`，而无法直接访问硬盘等外部存储设备中的数据和指令。因此，为了使程序能够被CPU执行，必须先将程序加载到内存中。
当程序被加载到内存中后，CPU可以通过`内存地址`来访问程序的指令和数据。而内存的访问速度比硬盘等外部存储设备的访问速度要快很多，所以将程序加载到内存中可以提高程序的运行效率。
程序加载到内存中运行是计算机系统必须遵循的`基本原则`，也是现代计算机系统高效运行的重要保障。这样做也可以提高程序间的并发能力，也提高了程序的安全性与稳定性。

>而内存的管理，又正好是操作系统的工作。

而要谈的主题——进程，简单的说就是`运行的程序`，也就是加载到内存中的程序，这是需要操作系统进行协调控制的。
计算机作为`管理者`，驱动程序就是`执行者`，各种软硬件资源就是`被管理者`。管理者和被管理者并不需要见面，就像学校里你不用与校长见面，在公司也大概率不会见老板，你们之间的协调是通过辅导员或组长来完成的，而在计算机系统中，这个过程就交由程序作为执行者来完成。
`系统调用`则作为操作系统与程序之间的`接口`，它将操作系统的底层功能暴露给程序，使得程序可以直接调用操作系统提供的服务和资源。系统调用是程序实现系统级别功能的重要手段之一。

操作系统在进程方面的作用可以简单的概括为：`先描述进程，再组织进程`。

# 进程

## 概念
何为描述？又何为组织？
>进程是指正在运行的程序实例，它包含了程序代码、数据和执行状态等信息。操作系统需要对进程进行管理，包括创建、调度、终止、通信等操作。为了对进程进行管理，操作系统需要`先对进程进行描述`，即确定进程的`属性和状态`，如进程ID、优先级、状态等。进程描述通常由`进程控制块（Process Control Block，PCB）`来完成。

在进程描述的基础上，操作系统需要对进程进行组织，以便`进行管理和调度`。进程可以组织成多种形式，如进程队列、进程树等。进程队列是指将同类进程组织到一起，如就绪队列、等待队列等。进程树是指将进程按照父子关系组织起来，形成树形结构。
举个例子，当一个进程需要被调度时，操作系统可以根据进程的属性和状态，选择合适的调度算法将进程调度到CPU上执行。同时，操作系统还可以通过进程间通信等机制，实现进程之间的数据共享和协作。

>一个进程等于`PCB`加上自己的`数据与代码`。

当一个进程需要被加载到内存中去时，首先会创建一个描述进程的结构体对象，也就是PCB。

PCB通常包含以下信息：

1. 进程状态：表示进程当前的状态，如就绪、运行、等待等。
2. 进程标识：标识进程的唯一标识符，如`进程ID、父进程ID`等。
3. 寄存器值：保存进程在执行过程中各个寄存器的值，如程序计数器、堆栈指针等。
4. 进程优先级：表示进程的优先级，用于调度器进行进程调度。
5. 进程资源：表示进程所占用的资源，如打开的文件、分配的内存等。
6. 进程调度信息：包含了进程的调度信息，如进程的调度时间片、已执行的CPU时间等。

Linux操作系统下的PCB叫做`task_struct`。
task_struct是Linux内核的一种数据结构，它会被装载到RAM(内存)里且包含着进程的信息。

在task_struct里常有以下的内容：
标示符: 描述本进程的唯一标示符，用来区别其他进程。
状态: 任务状态。
优先级: 相对于其他进程的优先级。
程序计数器: 程序中即将被执行的下一条指令的地址。
内存指针: 包括程序代码和进程相关数据的指针，还有和其他进程共享的内存块的指针

除此之外还有上下文数据，记账信息等等。

可以通过proc指令查看进程,或者ps，top等用户级工具。
<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B3.png">

也可以通过系统调用的方法来获取进程的`pid（进程id）`以及`ppid（父进程id）`。

```c
#include <stdio.h>
#include <unistd.h>
int main()
{
 printf("pid: %d\n", getpid());
 printf("ppid: %d\n", getppid());
 return 0;
}
```

运行此程序得到如下结果：
<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B4.png">

在操作系统中，父进程和子进程指的是进程之间的关系。

>父进程是生成其他进程的进程，而子进程是由父进程生成的进程。

当一个进程创建一个新的进程的时候，创建者进程就被叫做父进程，而新创建的进程被称为子进程。父进程会通过`系统调用fork()`来创建子进程。在创建子进程后，父进程和子进程就会拥有相同的代码段、数据段和堆栈等资源，但是它们具备`各自独立`的执行空间。

子进程会从父进程那里继承部分的属性，包括进程ID,文件描述符等。子进程可以通过系统调用exec()来加载新的程序代码，从而替换掉原有的代码段，从而开始执行新的程序。子进程还可以通过系统调用exit()来终止自己的执行。

父进程和子进程的关系是一种层次结构，可以形成进程树。父进程可以创建多个子进程，而子进程也可以再次创建新的子进程，形成`多层级的进程树结构`。

接下来就认识一下如何用fork创建进程。

## fork

fork()是一个系统调用，作用是创建一个新的进程。它会`复制当前进程，创建一个新的子进程`。在调用fork()时，操作系统会为子进程创建一个新的进程控制块（PCB），并将父进程的PCB的副本复制给子进程。

头文件为：unistd.h
函数原型：pid_t fork(void);

>fork()函数`返回两次`，一次在父进程中返回子进程的进程ID（PID），一次在子进程中返回0。这样可以通过`返回值的不同`来区分父进程和子进程的执行路径。

在fork()函数执行后，父进程和子进程会继续执行fork()调用之后的代码。父进程和子进程会拥有相同的代码段、数据段和堆栈等资源，但是它们有各自独立的执行空间。

子进程会继承父进程的很多属性,但同时子进程会有一些自己的特点，例如它的父进程ID会是父进程的进程ID，而子进程的进程ID会是一个新的唯一值。

如何做到返回两次？
>fork()函数可以创建一个新的进程，该新进程是原始进程的一个副本。通过fork()函数，原始进程就会被复制一份，包括代码、数据、堆栈等，并且在两个进程中返回不同的值。
具体的讲,fork()函数被调用时，操作系统会创建一个新的进程（称为子进程），并将子进程的副本返回给父进程。`父进程会收到子进程的进程ID（PID），而子进程会收到0`。这样就实现了fork()函数返回两次的效果。
在fork()函数被调用后，父进程和子进程会在`fork()函数调用的位置`继续执行。它们会完全`独立`地运行，并且有各自的进程ID（PID）。子进程会继承父进程的文件描述符、用户ID等属性，但它们的执行环境是相互独立的。
通过fork()函数的返回值，父进程和子进程可以根据不同的返回值来执行不同的代码逻辑。且它们的执行顺序是不确定的，取决于操作系统如何调度。

总结一下fork：
1. fork有两个返回值。
2. 父子进程`代码共享`，`数据各自开辟空间`，私有一份（采用写时拷贝）。

使用fork时常用if进行分流。

```c
#include <stdio.h>
#include <unistd.h>
int main()
{
 int ret = fork();
 if(ret < 0){
 perror("fork");
 return 1;
 }
 else if(ret == 0){ 
 printf("I am child : %d!, ret: %d\n", getpid(), ret);
 }else{ 
 printf("I am father : %d!, ret: %d\n", getpid(), ret);
 }
 sleep(1);
 return 0;
}

```
结果：
<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B5.png">

fork()函数的作用是创建一个新的进程，使得父进程和子进程可以并发执行、进行进程间通信、共享和隔离资源，并形成进程树结构，实现进程的管理和控制。具体的使用会在后面介绍。

在Linux环境下，父进程创建子进程的常用方法是使用fork()函数，但除了fork()函数，还有其他的方式可以创建子进程(了解)。

vfork()函数：与fork()函数类似，但是vfork()函数创建的子进程会与父进程共享相同的地址空间，子进程在调用exec()函数或者exit()函数之前必须要调用exec()函数或者_exit()函数来替换或退出，否则可能会导致数据不一致的问题。

clone()函数：clone()函数是Linux特有的系统调用，可以创建一个新的进程，并允许在创建时指定各种选项，例如共享地址空间、共享文件描述符等。

pthread_create()函数：该函数用于在一个进程内创建一个新的线程，这个线程可以看作是一个轻量级的子进程。线程共享父进程的地址空间和资源，但是有独立的栈空间和线程ID。

还有一些特殊情况下可以创建子进程，例如通过system()函数调用shell命令，或者通过exec()函数在当前进程内启动一个新的程序。但是这些方式实际上都是`通过fork()或者clone()等底层机制`来实现的。




## 进程状态

在Linux中，进程有主要存在以下几个状态：

1. 运行（Running）：进程正在执行或等待执行。
2. 就绪（Ready）：进程已经准备好执行，但还没有得到CPU的执行时间。
3. 等待（Waiting）：进程正在等待某个事件的发生，如IO操作的完成、信号的到达等。
4. 停止（Stopped）：进程被暂停执行，通常是由于接收到一个信号或被调试器所控制。
5. 僵尸（Zombie）：进程已经执行结束，但其父进程还没有调用wait()或waitpid()来获取其状态信息，所以进程的退出状态还没有被收集。

>Linux中的进程状态是动态变化的，进程可以在不同的状态之间切换。

查看进程状态可以使用`ps aux`指令。

<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B6.png">

而在其中的僵尸状态是很特殊的一个。
>在Linux中，僵尸（Zombie）状态是指一个已经退出的进程，但父进程还`没有调用wait()或waitpid()`来获取其退出状态信息的进程。当一个进程结束时，它的退出状态信息会保存在内核的`进程表`中，直到其父进程调用wait()或waitpid()来获取这些信息。如果父进程没有及时处理，那么该子进程就会成为僵尸进程。

僵尸进程因为已经结束所以不会占用系统资源，只有一个进程表的槽位和一个进程ID。但是，如果系统中存在大量的僵尸进程，也会占用过多的进程表资源，从而导致系统性能下降。

僵尸进程的产生通常有以下几种情况：

1. 父进程没有及时调用wait()或waitpid()来获取子进程的退出状态信息。
2. 父进程已经终止，但是没有正常处理子进程的退出状态信息。
3. 父进程忽略了SIGCHLD信号，该信号是子进程终止时向父进程发送的信号。

简单的说，父进程交给了子进程工作，那么子进程就需要一个途径来向父进程反馈任务的完成情况，可父进程如果一直不读取其状态信息，那子进程就就一直处于Z状态。
维护退出状态本身就是要用数据维护，也属于进程基本信息，所以保存在task_struct(PCB)中，换句话
说，Z状态如果一直不退出，PCB一直都要维护。
那一个父进程创建了很多子进程，但是不进行回收，就会造成内存的浪费，甚至更严重的后果——内存泄漏。




进程切换是操作系统进行多任务处理的基本机制之一。
当我们在使用电脑时，不会只使用一个软件，同时会有大量的进程在进行交替执行！这种现象我们称为`并发`。尽管在宏观下我们发现软件是在同时运行，比如听歌打游戏可以被同时进行，但微观上是由处理器快速的切换进程并交替执行所完成的。

当有多个进程同时进行,若系统是单CPU,那么它只能把CPU运行时间划分成若干个`时间片`,再将时间段分配给各个进程执行,在一个时间段的进程代码运行时,其它线程处于挂起状态.这种方式称为并发(Concurrent)。

当系统有多个CPU时，进程互不抢占CPU资源,可以同时进行,这种方式称为并行(Parallel)。

下面是一个简化的图示来解释Linux下进程的切换过程：
```
+-------------------------+                   +-------------------------+
|        进程A            |                   |        进程B            |
|-------------------------|                   |-------------------------|
|       寄存器状态A        |                   |       寄存器状态B        |
|                         |                   |                         |
|          代码           |                   |          代码           |
|          数据           |                   |          数据           |
|          堆栈           |                   |          堆栈           |
|                         |                   |                         |
|-------------------------|                   |-------------------------|
|        进程控制块A       |                   |       进程控制块B        |
+-------------------------+                   +-------------------------+


```
当进程A正在执行时，它的寄存器状态A（包括程序计数器、栈指针等）和相关数据存储在进程A的内存空间中。

当`操作系统调度器`决定要切换到进程B时，它会保存进程A的寄存器状态A和其他重要的信息（例如堆栈指针等），并将这些信息存储在进程A的进程控制块中。

操作系统会将进程B的进程控制块B中存储的寄存器状态B和其他信息加载到对应的寄存器中，来恢复进程B的执行环境。

进程切换完成后，操作系统会将CPU的控制权转移到进程B，让进程B继续执行。

在进程切换过程中，操作系统会保存和恢复进程的寄存器状态、堆栈指针等重要信息，以确保进程的执行环境得以保存和恢复。这样，操作系统能够在不同进程之间快速切换，实现多任务处理和资源共享。


## 僵尸进程与孤儿进程

### 模拟

僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵尸进程。

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
 
 
   int main()
   {
       printf("%d\n",getpid());
       pid_t pid = fork();//此时就创建了子进程
      if(pid < 0)
    {
          printf("error\n");
         return -1;
      }
      else if(pid == 0) //子进程
      {
          
          printf("I am child——%d\n",getpid());
          sleep(5);
          exit(0);                                                                                                                                                                        
      }
      else if(pid > 0) //父进程
      {
          
          printf("I am father——%d\n",getpid());
      }
     while(1)
      {
          printf("%d\n",getpid());
          sleep(1);
      }
  
     return 0;
  }

```

得到结果：

```
14110
I am father——14110
14110
I am child——14111
14110
14110
14110
14110
```

<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B8.png">



孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。如果是前台进程的话，我们在前台将无法输入命令，但是如果变成孤儿进程的话，就会变成`后台的进程`，依然可以运行。



```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
   
   int main()
   {
       printf("%d\n",getpid());
       pid_t pid = fork();
       if(pid > 0)
      {
          sleep(5);
          exit(0);                                                                                                                                                                        
  
      }
      while(1)
     {
          printf("%d\n",getppid());
          sleep(1);
      }
  
      return 0;
 }

```

执行结果：
<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B9.png">



此时孤儿进程将被init进程(进程号为1)所收养。只有通过kill才能杀死它。



### 一些细节

>题1.下面有关孤儿进程和僵尸进程的描述，说法错误的是？
A.孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。
B.僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵尸进程。
C.孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。
D.孤儿进程和僵尸进程都可能使系统不能产生新的进程，都应该避免。


如果进程不调用wait或者waitpid，那么保留信息就不会被释放，其进程号就会一直被占用，如果大量的产生僵尸进程，会产生内存浪费的现象，应当避免。
而孤儿进程是没有父进程的进程，孤儿进程的管理这个重任就落到了init进程身上 ，init进程专门负责处理孤儿进程的善后。每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为init，当一个孤儿进程结束其生命周期时，init进程就会去处理它， 因此孤儿进程并不会有什么危害。

>题2.关于僵尸进程，以下描述正确的有？
A.僵尸进程必须使用waitpid/wait接口进行等待
B.僵尸进程最终会自动退出
C.僵尸进程可以被kill命令杀死
D.僵尸进程是因为父进程先于子进程退出而产生的

A选项正确，僵尸进程会造成资源泄露，必须使用wait/waitpid接口进行等待处理。
B选项错误，僵尸进程不会完全释放资源退出。
C选项错误，僵尸进程是已经退出运行的进程，无法被杀死。
D选项错误，僵尸进程是子进程先于父进程退出。

>题3.以下关于孤儿进程的描述正确的有
A.父进程先于子进程退出，则子进程成为孤儿进程
B.孤儿进程会产生资源泄漏
C.孤儿进程运行在系统后台
D.孤儿进程没有父进程

A选项正确：父进程先于子进程退出，子进程就会成为孤儿进程。
B选项错误：孤儿进程退出不会成为僵尸进程，因此也不会资源泄露。
C选项正确：孤儿进程是运行在后台的。
D选项错误：孤儿进程也有父进程，父进程是1号进程。

## 进程优先级

既然进程可以切换，那么到底谁在前面谁又在后面呢？操作系统的调度器是以什么标准来判断：何时一个进程该被执行，何时一个进程该被阻塞呢？

>cpu`资源分配`的先后顺序，就是指进程的优先权（priority）。优先权高的进程有优先执行权利。

可以通过`ps -l`指令获取优先级。
<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E8%BF%9B%E7%A8%8B7.png">

1. UID : 执行者
2. PID : 这个进程的代号
3. PPID ：父进程的代号
4. PRI ：代表这个进程可被执行的优先级，其值越`小`越早被执行
5. NI ：代表这个进程的nice值

PRI是进程的优先级，就是程序被CPU执行的先后顺序，此值越小，进程的优先级别越高。
NI，也就是nice值，其表示进程可被执行的优先级的`修正数值`，PRI值越小越快被执行，那么加入nice值后，PRI则变为`PRI(new)=PRI(old)+nice`。当nice值为负值的时候，那么该程序将会优先级值将变小，即其优先级会变高，则其越快被执行。调整进程优先级，在Linux下，就是`调整进程nice值`。nice值通过top指令来调整。





