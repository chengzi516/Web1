---
title: 【c】简单的进度条程序
date: 2023-07-11 16:36:42
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/c.png
tags:
  - c语言
  - linux
categories: 
- c/c++
ai: ture
---

>实现一个简单的进度条程序。主要的思路就是利用一个`字符数组`，不断的改变其内容实现进度条的动画效果。

<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/processbar1.png">

# :green_heart:processbar.h

```c
#pragma once
#include <stdio.h>
#define NUM 102
#define TOP 100
#define BODY '='
#define RIGHT '>'
void processbar(int rate);
```
以上则是进度条的头文件，有如下说明：
1. TOP是字符数组的容量大小，NUM是TOP+2，为何这样设置在源文件处解释。
2. BODY与RIGHT是组成进度条的两个字符。
3. processbar则是主要的函数，用于接收目前的读取或下载进度（int，大小0-100）。
4. <unistd.h>头文件是POSIX标准定义的头文件，主要用于`UNIX/Linux`系统中。如果你在Windows系统上使用Visual Studio编译C程序，可以使用Windows API提供的`Sleep()函数`来实现类似于sleep()函数的功能。Sleep()函数的头文件是<windows.h>，需要在程序开头使用#include <windows.h>语句引入该头文件。Sleep()函数的使用方法和sleep()函数类似，不同之处在于它的参数是以毫秒为单位的时间长度，例如Sleep(5000)可以使程序挂起5秒钟。




# :blue_heart:processbar.c

```c
#include "processBar.h"
#include <string.h>
#include <unistd.h>  //sleep和usleep的头文件，作用是暂时休眠程序

const char *lable="|/-\\";  //一个字符数组指针，用来实现读取进度条时搭配百分比数字的动画效果
char bar[NUM];  //定义的字符数组

#define GREEN "\033[0;32;32m"   //自定义设置部分元素颜色
#define NONE "\033[m"


void processbar(int rate)
{
    if(rate < 0 || rate > 100) return;

    int len = strlen(lable);
    printf(GREEN"[%-100s]"NONE"[%d%%][%c]\r", bar, rate, lable[rate%len]); 
    fflush(stdout);
    bar[rate++] = BODY;
    if(rate < 100) bar[rate] = RIGHT;
}


```

核心代码就是这一行：

```c
 printf(GREEN"[%-100s]"NONE"[%d%%][%c]\r", bar, rate, lable[rate%len]); 
```
进度条的展示共分成三部分，第一部分是bar数组的=====>,第二部分是rate，也就是百分比数字展示，第三部分则是一个简单的动态转圈效果，为了防止访问越界，采用了取模的操作来保证所取到的字符只有|,/,- , \\。
要注意一点，因为是进度条程序，所以不可以使用\n来换行，整个程序的实现都`必须在一行上`完成。所以要使用`\r`,\r是回车符（carriage return）的转义字符，也称为`光标复位符`。在文本文件中，回车符通常用于将光标移动到当前行的开头位置，以便在同一行上输出新的内容。
在计算机中，标准输出通常是通过`缓冲区`实现的。缓冲区是一种临时存储区域，用于存储输出数据，直到`缓冲区被填满或手动刷新`为止。
在没有\n的情况下，printf函数`不会自动刷新`缓冲区，因为它无法确定什么时候应该刷新缓冲区。因此，输出数据将留在缓冲区中，直到缓冲区被自动刷新或手动刷新为止。
为了手动刷新缓冲区，我们可以使用`fflush函数`。fflush函数可以刷新指定的输出流，并将缓冲区中的数据输出到相应的设备中。在上面的代码中，使用fflush(stdout)来刷新标准输出缓冲区，确保输出数据立即显示在屏幕上。
但频繁地调用fflush函数可能会影响程序的性能。因此，只有在必要时才应该手动刷新缓冲区。在大多数情况下，当程序正常退出时，缓冲区中的数据将自动刷新到屏幕上。
所以要是`不用\n也不用fflush(stdout)`，就会导致程序运行后`等待接近一分钟`才会进行打印(这里我进行测试的设备是linux云服务器)，显然是不行的。


# :yellow_heart:main.c

```c
#include "processBar.h"
#include <unistd.h>


// 模拟一种安装或者下载
void downLoad()
{
    int total = 1000; // 1000MB
    int curr = 0;     // 0MB
    while(curr <= total)
    {
        usleep(50000);  // 模拟下载花费的时间
        int rate = curr*100/total; // 更新进度
        cb(rate); // 通过回调，展示进度
        curr += 10;     // 循环下载了一部分
    }
    printf("\n");
}

int main()
{
    printf("donwnload 1: \n");
    downLoad();
    printf("donwnload 2: \n");
    downLoad();
    printf("donwnload 3: \n");
    downLoad();
    return 0;
}

```