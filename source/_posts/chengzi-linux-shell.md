---
title: 【linux】简单模拟shell的实现
date: 2023-10-29 19:47:55
tags:
- 进程
- linux
categories:
- linux
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/linux.jpg
ai: true
---

# 什么是Shell
在Linux系统中，Shell是一种`命令行界面`，用于与Linux操作系统进行交互和控制。它是用户与操作系统内核通信的接口，允许`用户`输入命令，然后操作系统执行这些命令。Shell也可以执行脚本，这些脚本是一系列命令的集合，可以自动化执行多个任务。
常见的Linux Shell包括：Bash（Bourne Again Shell），Zsh，Fish，Dash和Ksh（Korn Shell）。
其中最为常用的就是Bash。
不同的Shell提供不同的功能和语法，用户可以根据自己的需求和偏好选择适合的Shell。无论使用哪种Shell，它们都允许用户执行文件操作、管理进程、配置系统和执行各种系统任务。
接下来就是一个简单实现shell的代码。

# Shell的实现

参照普通的linux命令行：
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/shell%E6%A8%A1%E6%8B%9F1.png'>
先来给出一个类似的格式：

```c
#define LEFT "[" // 左括号
#define RIGHT "]" // 右括号
#define LABLE "#" // 井号
#define LINE_SIZE 1024 // 设定命令行的最大长度
char pwd[LINE_SIZE]; // 全局变量，存储当前工作目录
// 获取当前用户名
const char *getusername()
{
    return getenv("USER");
}

// 获取主机名
const char *gethostname()
{
    return getenv("HOSTNAME");
}
// 获取当前工作目录
void getpwd()
{
    getcwd(pwd, sizeof(pwd));
}
getpwd();
//打印格式
printf(LEFT "%s@%s %s" RIGHT "" LABLE " ", getusername(), gethostname(), pwd);
```
拿getusername举例，getenv("USER")调用了一个名为getenv的标准库函数，该函数用于`获取环境变量的值`。getenv函数接受一个字符串参数，该参数是要查询的环境变量的名称，然后返回该环境变量的值。
`getenv`函数返回一个`指向字符串的指针`。它返回的是字符串在内存中的`地址`，而不是字符串本身的副本，这有助于减少内存开销。
并且，getenv返回的指针指向的字符串是`只读`的，通常应该避免修改它，以免引发问题。如果需要修改环境变量的值，应该使用其他函数来设置环境变量的值，而不是直接修改getenv返回的指针指向的字符串。

>那么什么是环境变量？

在Linux系统中，环境变量是一种在操作系统上存储的`键值对`信息，它们包含了系统运行时需要用到的配置信息和数据。这些变量可以影响系统和用户进程的行为。而它们又可以被粗略分为以下几类：
1. 系统环境变量：
系统环境变量是在Linux系统启动时由操作系统设置的，`对所有用户和进程都可见`。
一些常见的系统环境变量包括：
PATH：定义了系统在哪些目录中查找可执行文件。
HOME：指定用户的主目录路径。
USER：当前用户的用户名。
LANG：定义了系统的默认语言。
TERM：定义了终端类型。
2. 用户环境变量：
用户环境变量是由`用户自定义`的，它们通常用于存储个性化的配置信息，只对`当前用户`可见。用户可以根据需要设置自己的环境变量，例如自定义的路径、编辑器、别名等。
3. 设置环境变量：
在Linux系统中，可以使用`export命令`来设置环境变量。例如，要将MY_VARIABLE设置为hello，可以使用以下命令：
```
export MY_VARIABLE=hello
```
4. 查看环境变量：
可以使用`echo命令`来查看环境变量的值。例如要查看MY_VARIABLE的值，可以使用以下命令：
```
echo $MY_VARIABLE
```
5. 永久性环境变量：
用户可以将环境变量的设置添加到.bashrc或.bash_profile等启动脚本文件中，使其在每次用户登录时自动加载。这样设置的环境变量将在用户的会话期间一直有效。
6. 使用环境变量：
程序可以使用标准库中的`getenv函数`来获取环境变量的值。例如，C语言中的getenv("MY_VARIABLE")将返回MY_VARIABLE环境变量的值。

那么hostname也是同理，通过getenv函数就能取到存在系统中的指定环境变量。
`getcwd`是一个标准C库函数，用于`获取当前工作目录的绝对路径`，并将结果存储在`提供的缓冲区`中。
pwd 是一个缓冲区用于存储当前工作目录的路径。该缓冲区必须在函数调用之前分配足够的内存以容纳路径，通常通过char pwd[SIZE]的方式来声明。
sizeof(pwd) 用于获取缓冲区 pwd 的大小，以确保 getcwd 函数不会超出缓冲区的边界。

运行结果如下：
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/shell%E6%A8%A1%E6%8B%9F2.png'>

当我们使用linux的命令行时，只有当我们主动退出，shell才会终止工作，所以我们可以认为它一直在进行类似于`while(1)`也就是死循环的操作。那么按照这个想法封装一下代码：
```cpp
// 与用户进行交互，获取命令行输入
void interact(char *cline, int size)
{
    getpwd();
    printf(LEFT "%s@%s %s" RIGHT "" LABLE " ", getusername(), gethostname(), pwd);
    char *s = fgets(cline, size, stdin);
    assert(s);
    (void)s;
    // 移除输入中的换行符
    cline[strlen(cline) - 1] = '\0';
}
char commandline[LINE_SIZE]; // 存储用户输入的命令行
int quit = 0; // 退出标志
int main()
{
    while (!quit) {
        // 1. 与用户交互，获取命令行
        interact(commandline, sizeof(commandline));
    }
    return 0;
}
```
那么此时就有了完整的交互函数。
fgets函数用于从`标准输入`（stdin）读取用户输入的命令行，并将其存储在 `cline 字符数组`中。size 参数指定了 cline 可以存储的最大字符数，以避免缓冲区溢出。
读取的文本包括`换行符`，因此需要在后面的代码中将其移除。
fgets函数会将读取的整行文本存储到指定的字符数组 cline 中，直到遇到`换行符（'\n'）或者达到了指定的最大字符数 size - 1`。如果在输入中存在换行符，它也会被读取并存储在数组中。所以需要移除掉换行符，否则在打印出我们需要的命令行格式后还会额外进行换行。

现在有了命令行的样式，就要实现命令行的执行。在这里就需要引入`进程替换`的概念与相关的几个函数。

>进程替换

用fork创建子进程后执行的是和父进程相同的程序代码,所以子进程往往要调用一种`exec函数`以执行另一个程序。当进程调用一种exec函数时,该进程的用户空间代码和数据`完全被新程序替换`。调用exec并`不创建新进程`,所以调用exec前后该进程的id并未改变。

![图源网络，侵权请联系删除](https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/shell%E6%A8%A1%E6%8B%9F3.png)

> 替换函数

在Linux中，有几个与进程替换相关的函数和系统调用，其中最常见的是exec()系列函数。

```
#include <unistd.h>
int execl(const char *path, const char *arg, ...);
int execlp(const char *file, const char *arg, ...);
int execle(const char *path, const char *arg, ...,char *const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execve(const char *path, char *const argv[], char *const envp[]);
```

execve()是最常用的exec()函数之一，它允许一个新的程序替代当前进程。execve()接受三个参数：`可执行程序的路径、命令行参数数组和环境变量数组`。它加载并执行指定的可执行程序，替代当前进程的映像。
exec()系列函数如果调用成功则加载新的程序从启动代码开始执行,不再返回。如果调用出错则返回-1,
所以exec函数`只有出错的返回值`而没有成功的返回值。


>记忆技巧(源自网络)
l(list) : 表示参数采用列表
v(vector) : 参数用数组
p(path) : 有p自动搜索环境变量PATH
e(env) : 表示自己维护环境变量

举例：
```c
int main() {
    char *const argv[] = {"ls", "-l", NULL};
    char *const envp[] = {"PATH=/bin:/usr/bin", "TERM=console", NULL};

    execl("/bin/ls", "ls", "-l", NULL);  // 使用绝对路径替代进程
    execlp("ls", "ls", "-l", NULL);      // 使用PATH环境变量中的路径替代进程
    execle("ls", "ls", "-l", NULL, envp);  // 使用自定义环境变量替代进程

    // 使用数组传递参数和环境变量
    execv("/bin/ls", argv);              // 使用绝对路径和参数数组
    execvp("ls", argv);                  // 使用PATH环境变量中的路径和参数数组
    execve("/bin/ls", argv, envp);       // 使用绝对路径、参数数组和自定义环境变量数组

    exit(0);
}

```



# 完整代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define LEFT "[" // 左括号
#define RIGHT "]" // 右括号
#define LABLE "#" // 井号
#define DELIM " \t" // 分隔符，包括空格和制表符
#define LINE_SIZE 1024 // 设定命令行的最大长度
#define ARGC_SIZE 12 // 设定命令参数的最大数量
#define EXIT_CODE 1 // 退出码

int lastcode = 0; // 最后一个命令的退出码
int quit = 0; // 退出标志
extern char **environ; // 外部环境变量表
char commandline[LINE_SIZE]; // 存储用户输入的命令行
char *argv[ARGC_SIZE]; // 存储命令参数
char pwd[LINE_SIZE]; // 存储当前工作目录

// 自定义环境变量表
char myenv[LINE_SIZE];

// 自定义本地变量表

// 获取当前用户名
const char *getusername()
{
    return getenv("USER");
}

// 获取主机名
const char *gethostname()
{
    return getenv("HOSTNAME");
}

// 获取当前工作目录
void getpwd()
{
    getcwd(pwd, sizeof(pwd));
}

// 与用户进行交互，获取命令行输入
void interact(char *cline, int size)
{
    getpwd();
    printf(LEFT "%s@%s %s" RIGHT "" LABLE " ", getusername(), gethostname(), pwd);
    char *s = fgets(cline, size, stdin);
    assert(s);
    (void)s;
    // 移除输入中的换行符
    cline[strlen(cline) - 1] = '\0';
}

// 将命令行按分隔符拆分成参数数组
int splitstring(char cline[], char *_argv[])
{
    int i = 0;
    argv[i++] = strtok(cline, DELIM);
    while (_argv[i++] = strtok(NULL, DELIM)); // 故意写的=
    return i - 1;
}

// 执行普通命令
void NormalExcute(char *_argv[])
{
    pid_t id = fork();
    if (id < 0) {
        perror("fork");
        return;
    } else if (id == 0) {
        // 让子进程执行命令
        execvp(_argv[0], _argv);
        exit(EXIT_CODE);
    } else {
        int status = 0;
        pid_t rid = waitpid(id, &status, 0);
        if (rid == id) 
        {
            lastcode = WEXITSTATUS(status);
        }
    }
}

// 处理内建命令
int buildCommand(char *_argv[], int _argc)
{
    if (_argc == 2 && strcmp(_argv[0], "cd") == 0) {
        chdir(argv[1]); // 改变当前工作目录
        getpwd(); // 更新当前工作目录
        sprintf(getenv("PWD"), "%s", pwd); // 更新环境变量中的PWD
        return 1;
    } else if (_argc == 2 && strcmp(_argv[0], "export") == 0) {
        strcpy(myenv, _argv[1]); // 复制自定义环境变量
        putenv(myenv); // 将自定义环境变量加入环境变量表
        return 1;
    } else if (_argc == 2 && strcmp(_argv[0], "echo") == 0) {
        if (strcmp(_argv[1], "$?") == 0) {
            printf("%d\n", lastcode); // 打印上一个命令的退出码
            lastcode = 0;
        } else if (*_argv[1] == '$') {
            char *val = getenv(_argv[1] + 1); // 获取环境变量的值
            if (val) printf("%s\n", val);
        } else {
            printf("%s\n", _argv[1]); // 直接打印参数
        }
        return 1;
    }

    // 特殊处理一下ls命令
    if (strcmp(_argv[0], "ls") == 0) {
        _argv[_argc++] = "--color"; // 添加ls命令的颜色选项
        _argv[_argc] = NULL; // 设置参数数组结尾为NULL
    }
    return 0;
}

int main()
{
    while (!quit) {
        // 1. 与用户交互，获取命令行
        interact(commandline, sizeof(commandline));

        // 2. 分割命令行为参数数组
        int argc = splitstring(commandline, argv);
        if (argc == 0) continue;

        // 3. 处理内建命令或执行普通命令
        int n = buildCommand(argv, argc);

        // 4. 执行普通命令
        if (!n) NormalExcute(argv);
    }
    return 0;
}

```
