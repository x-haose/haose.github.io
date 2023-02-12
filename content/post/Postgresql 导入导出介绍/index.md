---
slug: 43f67b6e
author: "昊色居士"
title: "Postgresql 导入导出介绍"
description:
date: 2022-10-21T20:26:23+08:00
image: rand
math:
license:
hidden: false
comments: true
draft: false
tags: ["数据库", "PostgrSQL"]
categories: ["数据库"]
---


# Postgresql 导入导出介绍

## 简介

在我们开发过程中，经常会遇到把生产环境或测试环境的数据库导入至本地进行测试。今天这篇给大家分享一下，如果使用 Postgresql 的导出工具及导入工具完成这项工作

## 命令及工具

### 导出工具

导出工具用的是`pg_dump` ，在 Windows 平台下这个工具叫做`pg_dump.exe`，谁用来导出指定数据库的工具，常用参数有：

- -h 指定运行服务器的主机名，就是要导出数据所在的地址
- -p 对应的端口号
- -U 有权限的一个用户名
- -d 要导出的数据库名字
- -f 保存的文件名
- -w 不发出请输入密码的问题，如果无法自动连接就会连接失败，对于我们编写自动脚本很有帮助
- -F 设置导出的文件格式，可以选择：
  - -Fp 纯文本 SQL 脚本文件(缺省), db.sql
  - -Fc 适合输入到 pg_restore 里的自定义格式归档，有压缩效果 db.dump
  - -Ft 适合输入到 pg_restore 里的`tar`归档文件。没有压缩效果 db.tar

### 导入工具

导入工具有两种可以选择，分别是 psql，和 pg_restore，但是 psql 只能导入纯文本 sql 格式的数据，而 pg_restore 则可以导入任何格式的数据，在这里我们会选择使用 pg_restore 来进行导入，常用参数有：

- -h 指定运行服务器的主机名，就是要导出数据所在的地址
- -p 对应的端口号
- -U 有权限的一个用户名
- -d 要导入进去的数据库名字
- -j 并行导入，后面数值可以是 cpu 的核心数量
- -F 导入的文件格式，和导出的时候相同就好
- 要导入数据库文件

要注意的是导入时，要导入一个空的数据库中，且要手动去创建这个目标数据库，虽然文档中存在`-C -c`参数，但经过测试是没有效果的：

> -c
> --clean
>
> 创建数据库对象前先清理(删除)它们。（如果任一对象不在目标数据库中， 这可能会产生一些无害的错误消息。）
>
> -C
> --create
>
> 在恢复数据库之前先创建它。如果也声明了`--clean`， 那么在连接到数据库之前删除并重建目标数据库。
>
> 如果出现了这个选项，和`-d`在一起的数据库名只是用于发出最初的 `DROP DATABASE`和`CREATE DATABASE`命令。 所有数据都恢复到名字出现在归档中的数据库中去。

## 密码文件

由于我们是想要通过脚本去导入导出，所以采用的也是免密连接的方式，在 pgsql 中，可以通过设置密码文件：`.pgpass`来达到免密的目的。

下面是.pgpass 文件的格式：

```
IP地址:端口号:数据库(可以用*代表所有数据库):用户名:密码
ip:port:db:user:pass
```

## 实例测试

本地数据库：

- ip: 127.0.0.1
- port: 5432
- user：postgres
- passwd：123456

目标数据库：

- ip: 192.168.1.5
- port: 15432
- user：user11
- passwd：abc123+
- db: video

切换用户

```shell
# 登陆到postgres用户
su postgres

# 切换至～目录
cd ~

```

配置免密文件

把本地数据库和目标数据库的连接信息配置进`.pgpass`文件中：

```shell
# 本地的连接信息
echo '127.0.0.1:5432:*:postgres:123456' >> .pgpass
# 目标的连接信息
echo '192.168.1.5:15432:*:user11:abc123+' >> .pgpass
```

进行导出：

```shell
# 导出数据库
pg_dump -h 192.168.1.5 -p 15432 -d video -U user11 -w -Fc -f db.dump
```

命令执行成功之后会导出至当前文件夹下，然后进行导入

```shell
# 使用template0模板创建一个纯净的数据库
psql -c 'CREATE DATABASE video WITH TEMPLATE template0;'

pg_restore -Fc -j 6 -w -d video db.dump
```

会注意到我们在导入的时候并没有指定 IP 地址端口号用户名之类的信息，因为我们使用的就是默认的信息，在默认的情况下，ip 为 127.0.0.1，端口号为 543，用户则默认是当前系统登陆的用户

## 结语

这里主要介绍了`Postgresql`数据库的导出工具，导入工具的使用，和认证文件`.pgpass`的介绍，希望大家有所收获！！！
