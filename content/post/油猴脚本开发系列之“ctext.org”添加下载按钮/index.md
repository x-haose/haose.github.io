---
slug: 4b34a1be
author: "昊色居士"
title: "油猴脚本开发系列之“ctext.org”添加下载按钮"
description: 
date: 2022-08-13T23:24:25+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "油猴脚本", "javascript", "逆向"
]
categories: [
    "油猴脚本", "逆向"
]
series:  [
    "油猴脚本"
]
---

# 油猴脚本开发系列之“ctext.org”添加下载按钮

## 声明

> 本人所有逆向、破解及爬虫相关教程均是以纯技术的角度来探讨研究和学习，严禁使用教程中提到的技术去破解、滥用、伤害其他公司及个人的利益，以及将以下内容用于商业或者非法用途。

## 前言

`ctext.org`网站名字叫做：“中国哲学书电子化计划”，此网站有很多咱们中国的文化书籍，但是网站却没有下载按钮，如果要下载的话只能手动复制，今天咱们就来给他加一个下载按钮

## 内容获取

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6592b28fc1e4ad28b51aaa548169fe9~tplv-k3u1fbpfcp-watermark.image?)
就拿这篇文章为例，我们要想下载内容，肯定要先把想要下载的内容获取到，咱们来分析一下。

### 标题

下载一篇完整的文章，一定要有个标题，我们先来把标题取到，直接`F12`打开调试模式看看

![](https://files.mdnice.com/user/29990/86e9bf6a-17cf-42dc-9879-903a7e7b1998.png)

可以很明显的看到标题文本元素的位置，但元素下面还有子节点，子节点下面还存在着文本。所以获取节点文本的时候不能直接用`textContent`，可以使用如下方法：

```javascript
document.querySelector('.wikisectiontitle').firstChild.nodeValue
```

![](https://files.mdnice.com/user/29990/04087ee5-de5d-4200-91d9-f9f32e4a331f.png)
可以看出标题已经成功获取出来了

### 内容

下面就是获取文章的内容了，同样先看看内容所在的元素，

![](https://files.mdnice.com/user/29990/be4dd83a-7a32-403c-8f2d-73d3a84adeed.png)
可以看到这里的内容就是在`tbody`下面的每一个`tr`下面，所以我们只要获取所有的`tr`就可以获取所有的文字内容了，所以根据特征可以写出 css 表达式为：

```javascript
document.querySelectorAll('table[style="width: 100%;"]>tbody>tr')
```

![](https://files.mdnice.com/user/29990/18b88130-9098-442c-b3f4-2fc29cea5034.png)
下面就是获取每个 tr 里面的内容，但为注意的是，`tr`里面可能是标题，也可能是内容，所以要更加特征去区分一下

![](https://files.mdnice.com/user/29990/77f58ab5-5b86-429f-8e1d-6b22cb8b0b65.png)
可以看到标题和文本虽然都在`tr`下面，但是其 css 样式还是不一样的，只要在遍历的时候区分出来就好，如下代码：

```javascript
    let texts = []
    // 查询所有的tr
    let all_text_el = document.querySelectorAll('table[style="width: 100%;"]>tbody>tr')
    // 依次遍历所有的tr
    all_text_el.forEach(el => {
        // 子标题的元素
        let title_el = el.querySelector('.wikisubsectiontitle')
        // 文本的元素
        let text_el = el.querySelector('td:nth-child(2)')
        // 有标题元素就把标题文本加入进去，否则就加入文本
        if (title_el) {
            texts.push(title_el.textContent)
        } else {
            texts.push(text_el.textContent)
         }
    })
    // 换行符分割，拼成字符串
    let text = texts.join('\n')
```

## 下载按钮

下载的方法找到了，就要有一个处罚的地方，总不能进来直接就下载啊，体验多不好，我们研究研究网页，看看啥地方合适。

![](https://files.mdnice.com/user/29990/d535ecc2-ee0b-4e83-bf29-d258d93b4354.png)

这里刚好有个地方，可以看看怎么加进去，先看看其他几个咋加的

![](https://files.mdnice.com/user/29990/c6872b9f-b59b-4f95-923c-d98b5dc71a47.png)
咱们也~~抄~~仿照一下，在找一个下载的按钮图标，也同样加在后面就可以了，
可以这样写：
```javascript
    let a = document.createElement('a')
    a.innerHTML = '<img src="https://cdn.icon-icons.com/icons2/7/PNG/128/arrow_torrent_download_889.png" border="0" width="44" height="44" alt="" title="">';
    document.querySelector('div[class="noprint opt"]').appendChild(a);
```
如此就把下载的按钮加到网页上面去了，就像这样：

![](https://files.mdnice.com/user/29990/bee11cc4-1c6f-4593-9814-703eb97091ab.png)
然后就是点击下载按钮的事件了

## 下载吧
下载的操作可以使用`URL.createObjectURL`的方式去下载，如一下示例代码：
```javascript
    let blob = new Blob([text])
    let filename = `${title}.txt`
    a.download = filename
    a.href = URL.createObjectURL(blob)
    URL.revokeObjectURL(blob)
```
这样就完成了，点击的时候会弹出下载的弹窗

![](https://files.mdnice.com/user/29990/2b4a777e-7b7a-428a-8241-2ff66fab2a43.png)

## 整合进油猴脚本
可以把脚本放到油猴脚本或脚本猫中，都可以，下面附上全部代码：
```javascript
// ==UserScript==
// @name         中国哲学书电子化计划下载
// @namespace    https://bbs.tampermonkey.net.cn/
// @version      0.1.0
// @description  中国哲学书电子化计划添加下载按钮
// @author       昊色居士
// @match        *://ctext.org/wiki.pl?if=*&chapter=*
// ==/UserScript==

(function () {
    'use strict';

    // 标题的元素
    let title = document.querySelector('.wikisectiontitle').firstChild.nodeValue

    let texts = []
    // 查询所有的tr
    let all_text_el = document.querySelectorAll('table[style="width: 100%;"]>tbody>tr')
    // 依次遍历所有的tr
    all_text_el.forEach(el => {
        // 子标题的元素
        let title_el = el.querySelector('.wikisubsectiontitle')
        // 文本的元素
        let text_el = el.querySelector('td:nth-child(2)')
        // 有标题元素就把标题文本加入进去，否则就加入文本
        if (title_el) {
            texts.push(title_el.textContent)
        } else {
            texts.push(text_el.textContent)
         }
    })
    // 换行符分割，拼成字符串
    let text = texts.join('\n')

    // 添加下载按钮
    let a = document.createElement('a')
    a.innerHTML = '<img src="https://cdn.icon-icons.com/icons2/7/PNG/128/arrow_torrent_download_889.png" border="0" width="44" height="44" alt="" title="">';
    document.querySelector('div[class="noprint opt"]').appendChild(a)
    
    // 增加下载事件
    let blob = new Blob([text])
    let filename = `${title}.txt`
    a.download = filename
    a.href = URL.createObjectURL(blob)
    URL.revokeObjectURL(blob)
    
})();

```
## 结语
一篇小油猴脚本的创造过程就是这些了，不能的地方呢，我感觉就是还是不能全集下载和文档格式如果是markdown的就更好了，我在研究研究，弄好了分享给大家