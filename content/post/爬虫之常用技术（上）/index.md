---
slug: 0a3373f9
author: "昊色居士"
title: "爬虫之常用技术（上）"
description: 
date: 2022-08-06T14:24:55+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "爬虫", "Python"
]
categories: [
    "爬虫", "Python"
]
series:  [

]
---

# 爬虫之常用技术（上）

## 前言

写了几篇关于逆向相关了知识，今天再来唠唠`爬`相关的，也就是代码该怎么写，用要用到什么库，什么库好用等，这里用的主要语言还是`python`

## 静态网页信息提取

静态的网页，就是 html 类型文本，提取方式一般有`css`提取、`xpath`提取、`正则表达式`提取，而针对标准的`html`网页来说前两种用的居多，正则表达式常用于使用`css`和`xpath`已经提取了，但还是不满足要求则需使用`正则表达式`来提取

### 使用的工具

在提取中，大部分的教程和大部分人喜欢使用`lxml`库或`BeautifulSoup`库来提取数据，下面是看下这两个库的介绍：

> - BeautifulSoup 是 Python 程序员中非常流行的 Web 抓取库，它基于 HTML 代码的结构构造 Python 对象，并且还合理地处理坏标记，但它有一个缺点：它很慢。
> - lxml 是一个 XML 解析库（也可以解析 HTML），它使用基于 `ElementTree`的 pythonic API 。（lxml 不是 Python 标准库的一部分)。

可以看的出来，都很流行，但是我呢不推荐这两个，因为用起来都太麻烦，所以就推荐用使用库：`parsel`，看看介绍：

> parsel 是一个独立的 Web 抓取库，可以在没有 Scrapy 的情况下使用。它使用了 lxml 库，并在 lxml API 之上实现了一个简单的 API。这意味着 Scrapy 选择器的速度和解析精度与 lxml 非常相似

最主要的就是`简单的API`，程序嘛，懒，怕麻烦才是王道

### xpath 和 css 语法

这个东西写起来可多可少，就看想不想水。想水就查资料抄全一点，不想水就挑几个常用介绍介绍。恰好我是怕麻烦的，还是挑几个说说吧

![](https://files.mdnice.com/user/29990/c8d8e190-40bc-4dea-ba4d-314f2f756e5d.png)
就拿上面空色框框，框住的地方来说，在使用的场景里，可能会遇到获取文本的、获得`role`属性的

- 使用`xpath`获取文本写法：`//cite[@class="iUh30 qLRx3b tjvcx"]/text()`
- 使用`xpath`获取属性写法：//cite[@class="iUh30 qLRx3b tjvcx"]/@role`
- 使用`css`获取文本写法：cite[class="iUh30 qLRx3b tjvcx"]::text
- 使用`css`获取属写法：cite[class="iUh30 qLRx3b tjvcx"]::attr(role)

  其中//cite 是匹配所有的 cite 元素，[]里面是按照属性来匹配，@后面跟着的就是属性名字，然后跟着的就是符号和值，使用`!=`、`>`、`<`等也是可以的，后面的`/`就是路径，`text()`是一个函数（同样还有其他的很多函数），就是去下面的文本，取属性则和`[]`里面的情况是一样的，同样是@+属性名

  css 选择器就是使用标准的 css 获取元素加扩展的语法获取文本和属性，引用文档中的一句话：

  > CSS 选择器扩展
  > 根据 W3C 标准， CSS selectors 不支持选择文本节点或属性值。但是在 Web 抓取环境中选择这些是非常重要的，Scrapy（parsel）实现了一些 非标准的伪元素:
  >
  > - 要选择文本节点，请使用 ::text
  > - 选择属性值，用 ::attr(name) name 是你想要处理的值的属性的名称
  >
  > 这些伪元素是 Scrapy / Parsel 特有的。它们很可能不适用于其他库，如 lxml 或 PyQuery 等

### 正则表达式

正则表达式的写法有很多，但在爬虫里面我感觉最常用的和实用的就是取两者之间的字符串，写法如: `a(.+?)b`就是取 a 和 b 直接的字符串，用的库也还是可以直接使用`parsel`的`.re()`方法，剩下的查资料吧（反正我也懒）

### 获取多个和获取一个
在使用`Scrapy / Parsel`获取一个数据和多个数据时，常推荐的api是：`extract`和`extract_first`，但是你用的时候会发现，你在IDE里面拼这几个单词，他一个提示都没有，全都要手敲，所以这里推荐使用：`get()`和`getall()`，效果是一样的，而且还好记。

## 暂时结束
先到这吧，下节继续，把官方文档附上：[https://scrapy-docs.readthedocs.io/zh/latest/topics/selectors.html](https://scrapy-docs.readthedocs.io/zh/latest/topics/selectors.html)