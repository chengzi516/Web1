---
title: 【debug笔记】记录初次使用qt做项目的一些tip
date: 2023-06-16 09:56:58
tags:
- qt
categories:
- debug
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/debug.jfif
ai: true
---

>前言：这几天学校实训，第一次接触qt的项目，也是遇到了许多的bug，在这里写一下我个人的解决方案，仅供参考。在修改时注意备份，以免造成损失。


# :umbrella: 复制以及创建项目

这个问题还是蛮常见的，当你copy一份别人的代码想在自己的项目上运行时，可能会无法运行。在拷贝他人工程时，如果代码里有后缀为`.pro.user`的文件，需要将此文件`删除`即可。此文件用于保存之前的编译环境信息。
在Qt项目中，会生成一个.pro.user文件，这个文件是用于记录用户的Qt Creator IDE设置和配置的。这些设置包括编译器、构建目录、自定义构建步骤等等。因为这些设置是`与用户相关`的，所以`不应该被包含在项目文件`中，否则可能会影响其他用户的设置和配置。
当你复制别人的Qt项目时，如果你想在自己的电脑上使用该项目，应该删除项目目录中的.pro.user文件。这是因为你的电脑上的Qt Creator IDE设置和配置可能与原项目的设置和配置不同，如果你不删除.pro.user文件，则可能会导致编译错误、构建错误等问题。
当你删除.pro.user文件后，Qt Creator会在下一次打开项目时重新生成一个新的.pro.user文件，并使用默认的设置和配置。这样可以确保你的项目在不同的电脑上都能够正确地编译和构建。
并且，如果你在开发过程中修改了Qt Creator的设置和配置，并且想将这些设置和配置保存在项目文件中，可以使用`.pro.shared`文件来实现。.pro.shared文件是用于记录项目的`共享设置和配置`的。这些设置包括编译器、库路径等等。使用.pro.shared文件可以确保项目在不同的电脑上使用相同的设置和配置。
也需要注意的是，在创建工程时，最好`避免中文，使用英文`。
`创建的路径禁止使用中文路径。创建的源文件名字禁止使用中文。编译代码的路径不能有中文`。

# :sunny: ui文件复制

这个很简单，在复制前先将自己的进行备份，直接拷贝别人的即可，注意更改objectname

# :cloud: makefile执行出错
Makefile 执行过程中出错：make: *** No rule to make target ‘ all ‘，needed by ‘all’.Stop
警戒
在编写Makefile文件时，不要随意的留空行，空格。虽然编写此文件频率很少，但仍然请注意。

出现如上的报错信息基本上都是属于找不到`所依赖的文件`所导致的，所以首先应该确保依赖的文件，是否真实存在。
但是很常见的一个现象就是，在此处，写了`多余的空格或空行`时，误以为依赖文件，导致找不到该文件，然后报错。
解决办法就很简单，去掉多余的空格即可。

# :snowflake: 插入图片后运行不显示
这个问题很常见，解决方法也很多，我在网上看到了很多种解决方案，在这里贴几种常见且靠谱的办法。
方案1. 去Qt的项目中构建设置中，`Shandow build`勾选项去掉。
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/qt1.png'>
方案2. 如果是QWidget类不显示，需要重新实现`paintEvent()`函数。
```c++
void Widget::paintEvent(QPaintEvent *)
{
    QStyleOption opt;
    opt.init(this);
    QPainter p(this);
    style()->drawPrimitive(QStyle::PE_Widget, &opt, &p, this);
} 
```
注意引入头文件 `#include<qpainter.h>`，不引入则会报错：qt error: variable `QPainter painter' has initializer but incomplete type

一般来说是不建议在QWidget类中添加图片的。不过还有方案3
方案3. 在QWidegt中再添加一个QWidegt在这个QWidegt中添加背景图就能直接显示出来。
不过方案1，2就能解决大部分的情况。

# :snowman: 更改ui后运行不变
当你改变ui界面后，再次debug或者release，可能会发现界面并未更改。
导致的原因大概率是你复制的源码或ui,导致"ui_xxx.h"头文件不能`及时更新`导致的.
只需要将工程的编译目录下的`ui_xxx.h文件`全部删除,再重新编译即可.
如果这样还不行，可以尝试删除掉整个realse或者debug文件。反正这两个文件夹都是会自己生成的。具体的路径可以去构建目录看，也就是上文的那张图片。
不过某些情况，直接取消`Shandow build`的勾选或者`重新进行勾选`也可以解决问题。
本质上，造成这种现象的原因就是工程目录下的文件`没有及时更新`。所以只要可以进入构造目录更新ui_xx.h即可。


# :foggy: 字体设置失败

当使用构造函数设置字体时，对某些控件无效，这个一般是因为该控件已经在`designer界面`手动设置了其他字体，需要去掉。

# :ocean: 找不到文件路径
非常熟悉的报错信息了，No such file or directory，hhhhh。
文件可能压根就`不存在`或者`路径`出现问题。
可以考虑用`Everything/FSearch`搜索一下是否有这个文件。文件存在则大概是路径的问题。
在Windows下的Qt，就找目标文件所在的路径，添加到.pro文件里的`INCLUDEPATH += 绝对路径 / 相对路径 $$PWD/xxxxxx`。

# :cat:file not recognized: File truncated
一般是编译数据被截断或则编译过程中发生异常退出系统导致数据丢失。
直接`clean - qmake - 重新构建(rebuild)`。

# :dog: "qualified-id in declaration before (’ token"
这个报错也是非常常见。一旦出现就可能会出现一堆。要从`第一个开始报错`的地方往前找。问题就是前面漏了小括号或者中括号。

# :mouse: Invalid parameter passed to C runtime function
空指针，或者下标越界？这个就很宽泛了，需要dbug好好调一下。

# :frog: Qt ui界面存在控件，而代码编写无提示甚至报错
在ui界面添加的物件，在代码内操作时，ui->没提示，手打也显示错误，这种情况一般`运行一下`就可以了。
如果构建项目，也提示报错。那么可以考虑将项目的构建目录与项目不再置于同一个目录下，修改了构建目录，也能解决。

以上是我这几天遇到的一些报错，之后有了新的也会持续更新。