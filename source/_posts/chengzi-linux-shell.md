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
为了完成指令的执行，还需要将输入的指令进行分割。比如输入ls -l，就得提取出ls与-l两个操作。
```c
#define DELIM " \t" // 分隔符，包括空格和制表符
// 将命令行按分隔符拆分成参数数组
int splitstring(char cline[], char *_argv[])
{
    int i = 0;
    argv[i++] = strtok(cline, DELIM);
    while (_argv[i++] = strtok(NULL, DELIM)); // 是=而非==，解释在下文
    return i - 1;
}
```

>strtok（字符串分割函数）是C标准库中的一个函数，用于将字符串拆分成多个子字符串，其中每个子字符串都是由特定的分隔符字符分隔的。

```
char *strtok(char *str, const char *delim);
```

参数：
str：要分割的字符串，通常是第一次调用时传递字符串的指针，后续调用传递NULL来继续分割相同的字符串。
delim：包含分隔符字符的字符串，用于指定拆分字符串的分隔符。
返回值：
strtok函数返回指向分隔后的子字符串的指针，如果没有找到更多的子字符串，就返回NULL。
工作原理：
第一次调用strtok时，传递要分割的字符串（通常是命令行字符串）和分隔符字符串。
strtok会在字符串中查找分隔符字符，并将第一个子字符串的指针返回。
在后续调用中，你可以传递`NULL`作为第一个参数，以继续分割相同的字符串。strtok会继续从上一次返回的位置继续查找分隔符字符，并返回下一个子字符串的指针。
注意，strtok函数会修改原始字符串，将分隔符替换为NULL字符。
如果在一个字符串中连续出现多个分隔符字符，strtok会将它们视为一个单一的分隔符。
如果没有找到更多的子字符串，strtok会返回NULL，这是结束分割的标志。

>在这个代码片段中，while 循环中故意写 = 而不是 == 是因为它的目的是为了终止循环。

strtok(NULL, DELIM)：在第一次循环迭代之后，strtok 函数会继续查找命令行字符串中的下一个分隔符，并返回下一个子字符串的指针。当`没有`更多子字符串时，strtok 返回 `NULL`。
while (_argv[i++] = strtok(NULL, DELIM))：这是一个 while 循环，它的条件部分是一个赋值表达式。在每次循环迭代中，strtok(NULL, DELIM) 会被执行，返回下一个子字符串的指针。这个指针被赋值给 _argv[i++]，然后 i 会自增。
循环条件部分 _argv[i++] = strtok(NULL, DELIM) 的结果是被赋值的指针值，如果这个指针不为 NULL，它被视为真（true），导致循环继续执行。
当 strtok 找不到更多子字符串时，它会返回 NULL。此时，_argv[i++] 会被赋值为 NULL，同时 i 会自增一次。因此，条件部分 _argv[i++] = strtok(NULL, DELIM) 的结果为 NULL，导致循环条件为假（false），从而终止循环。

拿到了需要处理的指令，就需要使用exec系列函数将其交给`子进程`来处理。

```c
int lastcode = 0; // 最后一个命令的退出码
#define EXIT_CODE 1 // 退出码
 pid_t id = fork();
    if (id < 0) {
        perror("fork");
        return;
    } else if (id == 0) {
        // 让子进程执行命令
        execvp(_argv[0], _argv);
        exit(EXIT_CODE);
    } else {
        //父进程执行部分
        int status = 0;
        pid_t rid = waitpid(id, &status, 0); //使用 waitpid 函数来等待指定的子进程（由 id 指定）。等待子进程执行完毕，同时将子进程的状态信息存储在 status 变量中。
        if (rid == id) 
        { //使用 WEXITSTATUS 宏来提取子进程的退出码。WEXITSTATUS(status) 会从 status 中提取子进程的退出码，将其存储在 lastcode 变量中，
            lastcode = WEXITSTATUS(status);
        }
    }
```

>为什么父进程要拿到子进程的退出码？

答案是以便在父进程中了解子进程的`执行结果`。
子进程的退出码：在UNIX和类UNIX操作系统中，每个进程在退出时都可以返回一个整数值，通常称为退出码（Exit Code）。这个退出码用来表示进程执行的结果，通常为0表示成功，非零值表示出现了错误或异常。
status 变量：在父进程中，waitpid 函数返回一个整数值 status，其中包含有关子进程的状态信息，包括子进程的退出码。
WEXITSTATUS(status)：这是一个宏，用于从 status 中提取子进程的退出码。WEXITSTATUS(status) 会将 status 中的`退出码部分`提取出来，使其可以存储在 lastcode 变量中。
lastcode 变量：lastcode 是用来存储子进程的退出码的整数变量。一旦 WEXITSTATUS(status) 获取了退出码，它将被赋值给 lastcode，这样就能在父进程中访问子进程的退出码了。
WEXITSTATUS(status) 宏是用来从 status 中提取子进程的退出码的，status 带有各种`子进程状态信息`的位字段，其中包括退出码。WEXITSTATUS 宏通过运算来获取退出码的部分。

status 变量的布局通常如下所示：
```
  15     7        0
  +-------+-------+
  | Exit  |   0   |
  | Code  | Signal|
  +-------+-------+

```
Exit Code（退出码）：低八位（bits 0-7）用来存储子进程的退出码。
Signal：在非正常退出时，高八位（bits 8-15）用来存储终止子进程的信号。
WEXITSTATUS(status) 宏会对 status 进行右移8位，将 Exit Code 部分移动到最低的八位，然后返回这个值，从而提取出子进程的退出码。

通过获取子进程的退出码，父进程可以根据这个值来判断子进程的执行结果，例如，是否成功执行了命令，或者是否出现了错误。这使得父进程能够根据子进程的状态采取进一步的操作。

> execvp(_argv[0], _argv)这里第一个参数为什么传入的是_argv[0],而不是路径？

在实际使用中，它可以是程序的绝对路径（如/usr/bin/ls）或相对路径（如./my_program），也可以是一个在系统的 PATH 环境变量中可以找到的可执行文件的名称（如ls）。
当你提供一个在系统的 PATH 环境变量中可以找到的可执行文件的名称（如 ls），系统会根据`PATH 环境变量中定义的路径`来查找可执行文件。系统会按照 PATH 中定义的路径顺序`逐个查找`，直到找到匹配的可执行文件或者查找失败。
```
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

这意味着系统会首先在 /usr/local/bin 目录下查找是否存在 ls 可执行文件，如果找到了，就执行这个文件。如果在这个目录下找不到，系统会继续查找 /usr/bin，然后是 /bin，以此类推，直到找到匹配的可执行文件或者在所有路径中都没有找到。

所以，当运行 ls 命令时，系统会根据 PATH 环境变量中定义的路径，找到与 ls 匹配的可执行文件，然后执行它。路径的查找顺序是从左到右，所以位于 PATH 环境变量最左边的路径会首先被搜索。

但这段代码存在缺陷，比如你输入cd命令，pwd命令，就得不到想要的结果，因为不管是什么命令，你都一律选择`创建子进程`来运行，而诸如cd，pwd这样的命令是内建命令，需要在父进程中运行。执行pwd是想要父进程的当前目录，执行cd是要进入父进程目录的相关条目。所以需要让普通命令和内建命令分别执行才行。

下面这段代码处理了cd、export 和 echo三个内建命令，如果需要更多的内建命令，就直接在循环里挨着添加else if即可，在Linux底层也是这样做的。
```c
// 自定义环境变量表
char myenv[LINE_SIZE];
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
