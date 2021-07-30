---
title: Linux下隐式输入的解决
date: 2021-07-16 15:24:29
tags: -LinuxC
---

有些输入并不合适直白地出现在屏幕上，所以需要在输入时做不回显处理

getch函数有这种功能，它会读取一个输入的字符，但是不显示在屏幕上。

但是在linux环境下并不方便使用getch，它所属的头文件conio.h并不在标准c库中

在linux环境下可以使用**stty**命令和`system()`函数结合的方法达到getch的效果

- stty

    **stty命令会修改终端命令行的相关设置**
    有屏蔽显示的作用

    ```sh
    stty echo #打开回显
    stty -echo #禁止回显
    ```

具体用法

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(void)
{
    char buf[101];
    system("stty -echo");
    scanf("%s", buf);
    system("stty echo");
    printf("%s", buf);
    return 0;
}
```

如此以来就可以在linux环境下实现不回显输入的功能

更多stty用法 -> <https://ipcmen.com/stty>

7.17更新

好像还有几种方法可以解决不回显输入的问题

- getch函数在linux下不在头文件conio.h而在另外一个中，
- 还有一种函数可以让命令行输入不回显，是个名字很长的系统调用
