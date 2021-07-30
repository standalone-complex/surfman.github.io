---
title: ubuntu20.04 mysql和其相关c接口的安装
date: 2021-07-17 17:47:24
tags: -MYSQL
---

在最新ubuntu下安装mysql非常简单，输入

```sh
sudo apt install mysql-server
```

即可

为了使用方便，可以再安装mysql的命令行界面mycli，如下

```sh
sudo apt install mycli
```

mycli相比mysql增加语法高亮和命令补全功能，增加了舒适度和效率

然后是安装mysql的工具包，其中有一些函数接口，在开发程序的时候会用到

```sh
shdo apt install libmysql++-dev
```

以上
