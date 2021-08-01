---
title: Ubuntu20.04下安装，使用cJSON
date: 2021-08-01 12:59:56
tags: -LinuxC
---

cJSON是一个给C语言提供的用于创建和解析JSON的函数库

[JSON是什么？](JSON是什么？.md)

***然后最开始的，是cjson的下载***

下载cJSON有两种途径

* [在sourceforge下载](https://sourceforge.net/projects/cjson/)

* 用git命令直接clone

```sh
git clone https://github.com/DaveGamble/cJSON
```

使用方法也有两种

* 将下载下来的文件中的cJSON.h和cJSON.c直接放入项目文件目录中，增加`#include"cJSON.h"`并且在编译时带上cJSON.c文件

* （仅限使用git命令clone的情况）

```sh
cd <克隆下来的仓库>
mkdir build 
cd build
cmake  ..
make
make install
# 安装后 使用时需要增加 #include<cjson/cJSON.h> 并且在编译时增加 -lcjson 参数，链接动态库
```

***安装完毕之后,就是使用cJSON***

* 如何用cJSON解析JSON字符串？

    1. 将JSON字符串转化为链表结构  
    cJSON储存JSON字符串的基本单位是一个cJSON结构体，cJSON会通过调用`cJSON* cJSON_Parse(const char* value)`函数将一个JSON字符串转化为一个链表结构存储在内存中并返回链表的头节点的指针，链表其中的每一个节点存储着这个JSON中的每个元素（如果这个元素又是一个JSON字符串的话这个节点就会指向由这个JSON字符串生成的链表）。
    2. 使用`cJSON_Get`系函数来得到链表中想得到的元素的节点  
    在执行过第一步后，我们得到了一个指向链表结构的头指针，将头指针放入`cJSON* cJSON_GetObjectItem(const cJSON* const object, const char* const string)`函数的第一个参数，将我们想得到的元素的名字放入第二个参数，随后函数就会返回我们需要的元素的节点的指针，这个时候我们还没有取到元素的内容，所以还需要继续下一步。  
    3. 使用cJSON_GetNumberValue或者cJSON_GetStringValue得到我们节点中存储的元素的内容  
    在执行完第二步后，我们得到了指向我们目标元素的节点的指针，这时只需要把指针放入`double cJSON_GetNumberValue(const cJSON* item)`或者`char* cJSON_GetStringValue(const cJSON* item)`的参数中，函数就会给我们返回这个节点中元素的内容，然后我们就成功地将JSON字符串通过cJSON函数解析成为我们需要的数据  

```c
/* 解析JSON字符串的demo */
/* 这里用的是第二种安装方法的使用方法 */

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <cjson/cJSON.h>

int main(void)
{
    /* 一个JSON字符串 */
    char buf_JSON[] = 
    "{                                              \
                        \"name\" : \"BBAslayer\",   \
                        \"age\" : 19,               \
                        \"sex\" : \"male\",         \
                        \"favorite\" : \"surf\"     \
    }";

    cJSON* cjson, *cjson_name, *cjson_age, *cjson_sex, *cjson_favorite;
    char* name, *sex, *favorite;
    int age;

    /* 函数将JSON字符串转化为链表结构存储在内存并返回头结点的指针 */
    cjson = cJSON_Parse(buf_JSON);

    /* 获得指向需要元素的节点的指针 */
    /* 如果有嵌套JSON，多次调用此函数就好 */
    cjson_name = cJSON_GetObjectItem(cjson, "name");
    cjson_age = cJSON_GetObjectItem(cjson, "age");
    cjson_sex = cJSON_GetObjectItem(cjson, "sex");
    cjson_favorite = cJSON_GetObjectItem(cjson, "favorite");

    /* 从节点中取得元素内容 */
    name = cJSON_GetStringValue(cjson_name);
    age = cJSON_GetNumberValue(cjson_age);
    sex = cJSON_GetStringValue(cjson_sex);
    favorite = cJSON_GetStringValue(cjson_favorite);

    printf("%s %d %s %s\n", name, age, sex, favorite);

    /* 输出结果为 */

    /* BBAslayer 19 male surf */

    cJSON_Delete(cjson);

    return 0;
}
```

* 如何用cJSON创建JSON字符串？  
    1. 使用`cJSON* cJSON_CreateObject()`创建一个cJSON头结点，返回指向头结点的指针
    2. 使用`cJSON_Add`系函数向指针指向的结构添加元素
    3. 使用`char* cJSON_Print(const cJSON* item)`将cJSON链表转化为JSON字符串

```c
/* 创建JSON字符串的demo */

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <cjson/cJSON.h>

int main(void)
{
    cJSON* cjson;
    char* buf;

    /* 创建cJSON头结点 */
    cjson = cJSON_CreateObject();

    /* 向指针指向的结构添加元素 */
    cJSON_AddStringToObject(cjson, "name", "BBAslayer");
    cJSON_AddNumberToObject(cjson, "age", 19);
    cJSON_AddStringToObject(cjson, "sex", "male");
    cJSON_AddStringToObject(cjson, "favorite", "surf");

    /* 将指针指向的cJSON链表转化为JSON字符串 */
    buf = cJSON_Print(cjson);

    printf("%s\n", buf);

    /* 输出结果为 */

    /*
        {
        "name": "BBAslayer",
        "age":  19,
        "sex":  "male",
        "favorite":     "surf"
        }
    */

    cJSON_free(buf);
    cJSON_Delete(cjson);

    return 0;
}
```

* 对JSON中数组的创建和解析  
数组在cJSON中的创建和解析略特殊，需要用到专有的函数，直接上demo

```c
/* 解析JSON数组元素的demo */
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <cjson/cJSON.h>

int main(void)
{
    char buf_JSON[] =                     \
    "{      \"name\" : \"BBAslayer\",   \
            \"array\" : [1, 2, 3, 4, 5],\
            \"weapon\" : \"claymore\"   \
    }";

    cJSON* cjson, *cjson_array, *cjson_name, *cjson_weapon, *cjson_item;
    int array_size, array_item, i;
    char* name, *weapon; 

    /* 这些就不说了 */
    cjson = cJSON_Parse(buf_JSON);

    cjson_name = cJSON_GetObjectItem(cjson, "name");
    cjson_weapon = cJSON_GetObjectItem(cjson, "weapon");

    name = cJSON_GetStringValue(cjson_name);
    weapon = cJSON_GetStringValue(cjson_weapon);

    printf("%s %s\n", name, weapon);

    /* 以下是解析cJSON链表结构中数组的流程 */

    /* 获得指向cJSON结构链表中名为array数组的指针 */
    cjson_array = cJSON_GetObjectItem(cjson, "array");

    /* 将上文获得的指针放入下方函数中可得到数组长度 */
    array_size = cJSON_GetArraySize(cjson_array);

    printf("以下为数组中的内容\n");

    /* 然后就可以便历数组元素了 */
    for(i=0; i<array_size; i++)
    {
        /* 获得指向array数组中指定索引的元素的指针 */
        cjson_item = cJSON_GetArrayItem(cjson_array, i);

        /* 从指针指向的cJSON结构中取数据 */
        array_item = cJSON_GetNumberValue(cjson_item);

        printf(" %d ", array_item);
    }

    printf("\n");

    /* 运行结果为 */

    /*
        BBAslayer claymore
        以下为数组中的内容
         1  2  3  4  5
    */

    cJSON_Delete(cjson);

    return 0;
}
```

```c
/* 创建JSON数组元素的demo */
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <cjson/cJSON.h>

int main(void)
{
    /* 想创建JSON中的数组，首先要创建cJSON结构链表，并且要创建特殊的数组结构链表 */

    cJSON* cjson, *cjson_array;
    char* buf;

    cjson = cJSON_CreateObject();
    cJSON_AddStringToObject(cjson, "name", "BBAslayer");

    /*  */
    cjson_array = cJSON_CreateArray();

    /* 此函数向数组结构中增加节点 */
    //cJSON_AddItemToArray(cjson_array, cjson_item);
    /* cJSON_Create系函数也可以通过这种方式直接创建节点 */

    cJSON_AddItemToArray(cjson_array, cJSON_CreateNumber(1));
    cJSON_AddItemToArray(cjson_array, cJSON_CreateNumber(2));
    cJSON_AddItemToArray(cjson_array, cJSON_CreateNumber(3));
    cJSON_AddItemToArray(cjson_array, cJSON_CreateNumber(4));
    cJSON_AddItemToArray(cjson_array, cJSON_CreateNumber(5));

    /* 可以直接输出数组 */

    buf = cJSON_Print(cjson_array);

    printf("%s\n", buf);

    cJSON_free(buf);

    /* 也可以将这个数组加入其他的cJSON结构中 */

    cJSON_AddItemToObject(cjson, "array", cjson_array);

    buf = cJSON_Print(cjson);

    printf("%s\n", buf);

    /* 输出结果为 */

    /*
        [1, 2, 3, 4, 5]
        {
                "name": "BBAslayer",
                "array":        [1, 2, 3, 4, 5]
        }
    */

    cJSON_free(buf);
    cJSON_Delete(cjson);

    return 0;
}
```

* cJSON结构的内存释放  
cJSON对于内存释放有两个函数，`void cJSON_free(void* object)` 和 `void cJSON_Delete(cJSON* item)`。  
第一个函数释放由cJSON结构解析出的字符串（其实你用 `free()` 也可以）  
第二个函数递归释放cJSON链表结构
