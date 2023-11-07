---
title: 【linux】关于文件，你可能需要知道这些
date: 2023-11-04 20:46:07
tags:
- 文件
categories:
- linux
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/linux.jpg
ai: true
---

# 回顾c语言的文件操作

## 相关接口
简单的回顾下在c语言中，我们是如何使用相关的文件接口的。
C语言提供了一套强大而灵活的文件接口，使得程序能够在磁盘上读取和写入数据。这个文件接口是操作系统提供的API（应用程序编程接口）的一部分，这里则是简单介绍一下C语言中常用的文件接口，包括文件的打开、读取、写入、关闭等操作。

>打开文件

在C语言中，要对一个文件进行操作，首先需要`将其打开`。这可以通过使用fopen函数来实现。

```c
FILE *fptr;  // 声明一个文件指针
fptr = fopen("example.txt", "r");  // 打开名为example.txt的文件以供读取
```

声明了一个文件指针fptr，使用fopen函数将名为example.txt的文件以`只读`模式打开，函数返回一个指向该文件的指针。

>读取文件

一旦文件打开成功，我们可以使用fread函数来读取文件内容。

```c
char buffer[100];  // 声明一个用于存储数据的缓冲区
fread(buffer, sizeof(char), 100, fptr);  // 从文件中读取100个字符到缓冲区中
```

上使用fread函数从打开的文件中读取了100个字符，然后将其存储到名为buffer的字符数组中。

>写入文件

如果需要将数据写入文件，可以使用fwrite函数。

```c
char data[] = "Hello, World!";
fwrite(data, sizeof(char), strlen(data), fptr);  // 将数据写入文件
```

将字符串"Hello, World!"写入到已经打开的文件中。

>关闭文件

在完成文件操作后，应该`及时关闭`文件，以释放资源。

```c
fclose(fptr);  // 关闭文件
```

>错误处理

在实际使用中，我们需要考虑可能发生的错误情况。可以通过检查函数的`返回值`来判断文件是否成功打开或操作是否成功。

```c
FILE *fptr;
fptr = fopen("example.txt", "r");
if (fptr == NULL) {
    printf("无法打开文件\n");
    return 1;
}
```
## c语言的输入输出流

```c
int main()
{
 const char *str = "hello world\n";
 fwrite(str, strlen(str), 1, stdout);
 printf("hello linux\n");
 fprintf(stdout, "hello world\n");
 return 0;
}
```

fwrite函数将字符串 "hello world\n" 写入`标准输出`。fwrite 函数的参数依次为字符串的指针（str），字符串的长度（strlen(str)），写入次数（1），和目标输出（stdout）。
rintf 函数来打印 "hello linux"。printf 函数将格式化字符串写入标准输出。
fprintf 函数来打印 "hello world"。fprintf 函数的参数依次为目标输出（stdout）和格式化字符串。

上面三个函数都提到了`stdout`，那么stdout到底是什么？
>在C语言中，stdout是`指向标准输出的文件指针`。标准输出是一个特殊的`文件流`，通常用于将程序的输出`打印`到屏幕上。
在这个函数中，使用了不同的输出函数来将字符串打印到标准输出上。fwrite、printf和fprintf都可以接受`文件指针作为参数`，用于指定输出的目标。
使用stdout可以方便地将输出打印到屏幕上，而不需要指定具体的文件或设备。这样，程序的输出就可以在控制台上可见，并且可以通过重定向等方式将输出保存到文件中。

C语言默认会打开`三个`输入输出流，分别是`stdin, stdout, stderr`。
stdout已经在上文解释过，当涉及到`输入和错误输出`时，C语言提供了两个额外的标准流：stdin和stderr。

stdin是指向`标准输入`的文件指针。标准输入用于`接收来自用户的输入`，通常是通过`键盘`输入。例如，使用scanf函数可以从标准输入中读取用户的输入。

stderr是指向`标准错误`的文件指针。标准错误用于`输出程序的错误消息或其他诊断信息`。与标准输出不同，标准错误的输出通常被发送到屏幕上的错误流中，而不会被重定向到文件。例如，使用fprintf(stderr, ...)函数可以将错误消息输出到标准错误流。



# linux下的文件操作

##  系统文件io
除了上面提到的C语言来进行文件操作，同样的，我们可以通过`系统调用`来对文件进行读或者写等等操作。
当使用系统调用进行文件I/O时，有这么几个常用的函数：

1. open()：用于打开文件。它接受文件路径和一些标志作为参数，并返回一个文件描述符（file descriptor），表示打开的文件。
2. read()：用于从文件中读取数据。它接受文件描述符、数据缓冲区和读取字节数作为参数，并返回实际读取的字节数。
3. write()：用于向文件中写入数据。它接受文件描述符、数据缓冲区和写入字节数作为参数，并返回实际写入的字节数。
4. close()：用于关闭文件。它接受文件描述符作为参数，并在操作完成后关闭文件。

>打开一个名为 "example.txt" 的文件（如果不存在则创建），然后写入字符串 "Hello, world!"。最后关闭文件。

```c
#include <fcntl.h>
#include <unistd.h>

int main() {
    int fd = open("example.txt", O_WRONLY | O_CREAT, S_IRUSR | S_IWUSR);
    if (fd == -1) {
        // 处理打开文件失败的情况
        return 1;
    }

    const char *data = "Hello, world!";
    ssize_t bytes_written = write(fd, data, strlen(data));
    if (bytes_written == -1) {
        // 处理写入文件失败的情况
        return 1;
    }

    close(fd);

    return 0;
}
```


## 四个常用接口介绍

>open

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```

pathname: 要打开或创建的目标文件
flags: 打开文件时，可以传入多个参数选项，用下面的一个或者多个常量进行“或”运算，构成flags。
参数:
O_RDONLY: 只读打开。
O_WRONLY: 只写打开。
O_RDWR : 读，写打开。
这三个常量，必须指定一个且只能指定一个。
O_CREAT : 若文件不存在，则创建它。
O_APPEND: 追加写。
返回值：
成功：新打开的文件描述符（什么是文件描述符会在下文讲）。
失败：-1

剩下的三个也都大差不差：
>close
```c
int close(int fd);
```

这个函数用于关闭文件。它接受文件描述符（fd）作为参数，在操作完成后关闭文件。 
参数：
fd：要关闭的文件描述符。
返回值：
成功：0。
失败：-1。

>write

```c
ssize_t write(int fd, const void *buf, size_t count);
```

这个函数用于向文件中写入数据。它接受文件描述符（fd），数据缓冲区指针（buf）和要写入的字节数（count）作为参数。 
参数：
fd：要写入的文件描述符。
buf：指向要写入的数据的缓冲区的指针。
count：要写入的字节数。
返回值：
成功：实际写入的字节数。
失败：-1。

>read

```c
ssize_t read(int fd, void *buf, size_t count);
```

这个函数用于从文件中读取数据。它接受文件描述符（fd），数据缓冲区指针（buf）和要读取的最大字节数（count）作为参数。

参数：
fd：要读取的文件描述符。
buf：指向存储读取数据的缓冲区的指针。
count：要读取的最大字节数。
返回值：
成功：实际读取的字节数。
失败：-1。

>接口演示

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    int fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        // 处理文件打开失败的情况
        return 1;
    }

    char buffer[100];
    ssize_t bytes_read = read(fd, buffer, sizeof(buffer));
    if (bytes_read == -1) {
        // 处理读取文件失败的情况
        close(fd);
        return 1;
    }

    close(fd);

    // 打印读取的数据
    write(STDOUT_FILENO, buffer, bytes_read);

    return 0;
}
```

可以看到上文很多地方都出现了fd和文件描述符的概念（二者是一个东西），那么什么是文件描述符呢？

## 文件描述符

在文章开篇时，我提到了c语言会打开三个输入输出流，那我们是如何得知这三个流被打开了呢？

文件描述符是一个`非负整数`，用于`唯一标识`一个打开的文件或I/O流。它是一个抽象的概念，可以是文件、管道、套接字等。

标准文件描述符
>Linux系统通常会为每个进程自动分配三个标准文件描述符：
0（stdin）：标准输入，通常用于从键盘或其他输入设备读取数据。
1（stdout）：标准输出，通常用于向终端或其他输出设备输出数据。
2（stderr）：标准错误，通常用于输出错误消息。
0,1,2对应的物理设备一般是：`键盘，显示器，显示器`。

文件描述符主要用于进行文件和I/O操作，通常通过以下系统调用来操作文件描述符：

>open()：打开一个文件并返回一个文件描述符。
close()：关闭一个文件描述符，释放相关资源。
read()：从文件描述符中读取数据。
write()：向文件描述符中写入数据。
lseek()：移动文件描述符的读/写位置。
dup()和dup2()：复制文件描述符，创建一个新的文件描述符与之关联。

也就是说，可以这样输出：
```c
int main()
{
 char buffer[1024];
 ssize_t s = read(0, buffer, sizeof(buffer));
 if(s > 0){
 buffer[s] = 0;
 write(1, buffer, strlen(buffer));
 write(2, buffer, strlen(buffer));
 }
 return 0;
}
```
上面的代码实现了从标准输入读取数据，并将其写入标准输出和标准错误。
那么现在就可以理解，文件描述符就是从`0`开始的整数。当打开文件时，操作系统在内存中要创建相应的数据结构来
`描述目标文件`，也就是file结构体来表示一个已经打开的文件对象。
例如，当进程执行open系统调用时，进程和文件必须有所关联。每个进程都有一个指针`*files`, 指向一张表`files_struct`,该表有一个指针数组，每个元素都是一个指向打开文件的指针。所以，本质上，`文件描述符就是该数组的下标`。所以，只要有文件描述符，就可以找到对应的文件。

```
+-------------------+         +------------------------+
|                   |         |                        |
|   Process (进程)  |         |   File Struct (file)   |
|                   |         |                        |
|   +------------+  |         |   +----------------+   |
|   |  *files    |  |         |   | File*  0       |   |
|   |  (指针)    |----------> |   | File*  1       |   |
|   +------------+  |         |   | File*  2       |   |
|                   |         |   +----------------+   |                  
+-------------------+         +------------------------+
  
0,1,2分别是这个指针数组的下标，从上往下分别指向stdin，stdout，stderr所代表的三个已打开的文件。
```

>注意，上图的File*列表仅存储打开的文件，也就是说，如果此时新建一个文件，那么分配给他的下标就会是3，如果在创建此文件前关闭了1，那么此文件的文件描述符就会被分配为1。这就是Linux系统下文件描述符基本的分配规则。并且File Struct这张表是唯一的，哪怕你在进程中打开了一个子进程，这张表也不会被拷贝，而是以共享的形式存在。

## 重定向

当我们关闭掉1:

```c
int main()
{
 close(1);
 int fd = open("file", O_WRONLY|O_CREAT, 00644);
 
 printf("fd: %d\n", fd);
 
 close(fd);
 exit(0);
}

```
按照我们才说的，那么屏幕就不会打印我们想要输出的内容，而是将其输入到file文件中。
事实确实如此，因为printf底层封装的仍然是输出到下标1，哪怕stdout被关闭，系统也不清楚，换句话说，系统只认识1，他要做的就是将需要打印的内容输出到1代表的这个文件。这也是我认为的linux环境下文件管理的一个显著特征，层层封装，再由系统统一调用，有点多态的意思在里面。

>将原本该打印到屏幕的内容打印到file里，这就叫重定向。

文件描述符的重定向允许将一个文件描述符与另一个文件或设备相关联。例如，可以使用>将命令的输出重定向到文件，或使用<将文件内容作为输入。
```c
//输出重定向：将命令的标准输出保存到文件。
//将 "Hello, World!" 写入到文件 output.txt
echo "Hello, World!" > output.txt
//输入重定向：从文件中读取数据作为命令的标准输入。
//从文件 input.txt 中读取数据并将其作为命令的输入
cat < input.txt
//追加：将命令的标准输出追加到文件末尾。
//追加 "Hello again!" 到文件 output.txt
echo "Hello again!" >> output.txt
//错误输出重定向：将命令的标准错误输出保存到文件。
//将命令的标准错误输出保存到 error.log 文件
ls non_existent_directory 2> error.log  
//标准输出和标准错误重定向到同一文件：
//将标准输出和标准错误输出都重定向到同一个文件
ls non_existent_directory > output_and_error.log 2>&1
//使用管道：将一个命令的输出传递给另一个命令的输入。
//列出当前目录下的文件，并将结果通过管道传递给 grep 命令以筛选文件名中包含 "example" 的文件
ls | grep "example"
```

但在本质上，其更改的是文件描述符所指向的内容。在上面我画了一张关于File Struct的图，当执行了close(1)操作，再执行新建file文件，那么此时文件描述符为1的坑位就指向了file而不是stdout。
可以这么说，重定向的魅力在于`操作文件描述符`，将它们连接到不同的位置，从而`改变了命令的输入和输出源`，使得命令行操作更加灵活多变。
举一个例子：
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    // 打开或创建文件 "output.txt"，并获取文件描述符
    int file_fd = open("output.txt", O_WRONLY|O_CREAT|O_TRUNC, 0666);
    
    if (file_fd == -1) {
        perror("open");
        exit(1);
    }
    
    // 备份标准输出文件描述符
    int stdout_backup = dup(1);
    
    if (stdout_backup == -1) {
        perror("dup");
        exit(1);
    }
    
    // 使用 dup2 将文件描述符 "file_fd" 复制到标准输出文件描述符 "1"
    if (dup2(file_fd, 1) == -1) {
        perror("dup2");
        exit(1);
    }
    
    // 现在标准输出已经被重定向到 "output.txt"
    printf("This will be written to output.txt\n");
    
    // 恢复标准输出
    if (dup2(stdout_backup, 1) == -1) {
        perror("dup2");
        exit(1);
    }
    
    // 关闭文件描述符
    close(file_fd);
    close(stdout_backup);
    
    // 现在标准输出已经恢复，继续输出到屏幕
    printf("This will be shown on the screen\n");
    
    return 0;
}

```
小tip:因为IO相关函数与系统调用接口对应，并且库函数封装系统调用，所以本质上，访问文件都是通过`fd`访问的。所以C库当中的FILE结构体内部，`必定封装了fd`。

## 缓冲区

```c

int main()
{
 const char *p1="fwrite\n";
 const char *p2="write\n";
 printf("printf");
 fwrite(p1, strlen(p1), 1, stdout);
 write(1, p2, strlen(p2));
 fork();
 return 0;
}
```
执行三个输出，将信息打印到屏幕上，再执行fork新建一个子进程，这就是目前代码所执行的逻辑。
但当你将此文件重定向到一个普通文件中时，输出会变成这样：
```c
write
printf
fwrite
printf
fwrite
```
看起来write仍然只执行一次，而printf和fwrite被执行了两次。

首先，需要明白一个概念:printf和fwrite库函数有自带的缓冲区，而write作为系统调用则没有缓冲区。
这些缓冲区都是用户级缓冲区。
printf和fwrite都是库函数，这个缓冲区由C的标准库来提供。而write是系统调用，库函数则是在系统调用的“上层”， 是对系统调用的“封装”，所以write没有这个缓冲区也不足为奇。
一般C库函数写入文件时是`全缓冲`(进程结束统一刷新)的，而写入显示器是`行缓冲`(遇到\n刷新)。当发生重定向到普通文件时，数据的缓冲方式由行缓冲变成了全缓冲。那么缓冲区中的数据就不会被立即刷新，甚至fork之后也不会刷新。当`进程退出`之后，就会被统一刷新，再写入文件当中。
当进行fork时，父子数据会发生写时拷贝，缓冲区也是不被共享的，但数据会被拷贝，那么就有了两个缓冲区，分别在等待进程结束时进行刷新，将数据刷新至内核级别的缓冲区。
目前来说可以这么理解，当数据被刷新到了内核级别的缓冲区，就可以认为数据到了硬件层面。

#  linux如何管理文件

在Linux系统中，文件系统以一种层次化的`树状结构`组织和描述所有的文件和目录。以`根目录`为起点，所有的文件和目录都从这里开始。
也许你听过一句话，`在Linux系统中，一切皆为文件`。这包括了普通文件、目录、设备文件、链接等等，并且不同类型的文件具有不同的属性和用途。相信在读到这里，你对这句话有了一个更深入的理解。
可以简单的总结，Linux将`硬件底层封装为文件`，并允许进程通过`文件指针`来进行调用和访问。
在Linux中，硬件设备通常由设备文件来表示，这些设备文件位于/dev目录下。每个硬件设备都有一个相应的设备文件，例如硬盘设备可以表示为/dev/sda，串口可以表示为/dev/ttyS0等。
同时，每个进程都有一个文件描述符表，它是一个索引到文件的整数数组。文件描述符是进程用来访问文件的句柄。通常，标准输入、标准输出和标准错误分别对应文件描述符0、1和2。
进程可以通过系统调用来操作文件。例如，open系统调用用于打开一个文件，read和write用于读取和写入文件数据，close用于关闭文件。进程通过这些系统调用来请求操作文件或设备。
当进程打开一个文件时，操作系统维护一个文件指针（或文件偏移量），它指示文件中下一个读取或写入操作的位置。文件指针可以通过系统调用来移动，如lseek。
借用一张linux的概念图：
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E6%96%87%E4%BB%B61.png'>
>可以这么认为，f#系列的函数，都是对系统调用的封装，方便二次开发。

这么一说，对Linux一切皆文件是不是又清楚了一些呢？

把话题转回来，在Linux中，每个文件都有一个`唯一的路径`，用于描述其在文件系统中的位置。路径可以是绝对路径（从根目录开始的完整路径）或相对路径（相对于当前工作目录的路径）。每个文件和目录都有权限设置，这确定了谁可以对其进行读、写和执行操作。此外，每个文件都有一个`所有者和所属的用户组`。
Linux支持多种文件系统类型，包括ext4、XFS、btrfs等。每个文件系统类型都具有不同的特性和性能。
Linux系统通过`系统调用`提供了一组API，用于`管理文件和目录`。这些系统调用包括打开、读取、写入、关闭、创建和删除文件，以及更改文件属性等。
以下几点作为了解：
链接和挂载： Linux支持硬链接和符号链接，允许多个文件名引用同一个文件。此外，Linux还支持文件系统挂载，使不同的文件系统可以被组合到同一个目录结构中。
特殊文件： Linux系统还包括特殊文件，如设备文件（用于与硬件设备通信）、套接字文件（用于进程间通信）和管道文件（用于进程间数据传输）。
文件系统维护： Linux系统中有一系列工具用于文件系统维护，如fsck用于文件系统检查和修复，du用于查看磁盘使用情况，df用于查看磁盘空间等。

