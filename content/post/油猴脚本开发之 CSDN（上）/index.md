---
slug: c66d5f14
author: "昊色居士"
title: "油猴脚本开发之 CSDN（上）"
description: 
date: 2022-08-03T21:54:16+08:00
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

# 油猴脚本开发之 CSDN（上）

## 声明

> 本人所有逆向、破解及爬虫相关教程均是以纯技术的角度来探讨研究和学习，严禁使用教程中提到的技术去破解、滥用、伤害其他公司及个人的利益，以及将以下内容用于商业或者非法用途。

## 前言

CSDN，大家都清楚，但大家在阅读的时候，想要借鉴一下代码，可能会出现不太方便的情况，比如登录复制啦、关注查看剩余内容啦，今天就来看看怎么进行改造一下来`方便自己阅读`。所使用到的工具是油猴脚本，大家可以自行了解一下这个。

## CSDN 不登录即可复制

![](https://files.mdnice.com/user/29990/556f55ab-cc0e-4307-a4f1-90f6877d23ba.png)
使用过程中有时会看到这种情况，这个时候是没有办法进行直接复制的。如果以`纯技术`的角度开始该怎么弄破解这个呢？我们来首先思考是不是由于 js 达成的效果，要想确定这个可以使用插件：`Toggle JavaScript`如图所示
![](https://files.mdnice.com/user/29990/84510cb6-9691-4360-bf54-7bf3ecd2bdbf.png)只需在当前页面点一下，即可关闭/开启当前页面的 js 脚本加载功能。我们来关闭当前页面的 js 脚本加载功能，看看还能不能复制了，结果可以看出，依旧不能复制。这个时候就要思考了，是怎么达成的这个效果，大家都知道一个网页是由 html 元素、css 样式、js 脚本组成的。html 元素肯定没有这个效果，我们又禁止加载了 js，所以就只剩下了 css 样式。就是看看 css 样式如何才能达成这个效果，我们来百度一下

![](https://files.mdnice.com/user/29990/9a6e87e5-2907-43f5-b1e1-d424ad5c16aa.png)

百度了之你就会看到其实是和一个叫做`user-select`样式相关，设置为`none`就不可复制，我们在副本附近的元素找找，有没有这个样式

![](https://files.mdnice.com/user/29990/a3b095a1-9204-4e86-b88a-88eeb1b31d85.png)

![](https://files.mdnice.com/user/29990/f7189fb2-86f5-4eaa-afe0-c53dd4191adf.png)
细心的去找了之后，就会分别在两个样式下面都存在`user-select: none`，既然发现了我们就可以来验证一下猜想，把前面的勾勾取消掉再看看对应的文本可不可以复制，发现可以了，接下来就是编写代码了，首先`user-select`样式是分别存在两个类下面的，我们的思路就是找到所有相关的元素后把`user-select`样式设置为`text`，表示可复制，代码如下：

```javascript
    for (const e of document.querySelectorAll('#content_views pre')) {
        e.style.setProperty('user-select', 'text', 'important')
    }
    for (const e of document.querySelectorAll('#content_views pre code')) {
        e.style.setProperty('user-select', 'text', 'important')
    }
```

下面我们重新刷新一下网页，然后直接把以上代码复制进控制台看是否生效，发现已经生效了

## 文字修改及事件删除

发生虽然可以复制文字了，但右边的按钮依旧可以点击并弹出登录的对话框。正常思路是既然这里一个点击事件，那我就把点击事件删除不就好了，这次就来一个不一样的方案，也是使用`css`，就是把不需要点击的元素上面设置`pointer-events: none`就好了，然后再顺便把文字改一下，示例代码如下：

```
    document.querySelectorAll('.hljs-button').forEach(
        doc => {
            doc.setAttribute("data-title", "随便复制")
            doc.setAttribute("style", "pointer-events: none;!important")
        }
    )
```

## 复制不带版权

> 再次声明：此功能仅适用于已知版权信息内容，并且确保不会违反的情况下使用。如不满足上述情况，严禁使用，请尊重作者知识产权。

大家在复制一段代码用做借鉴的时候往往后面会出现版权相关的信息，而且如果是直接粘贴在 IDE 中，还是出现格式错乱的情况。先看下版权的内容：

```
————————————————
版权声明：本文为CSDN博主「x」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/xx/article/details/xxx
```

我们先全局查找一下，看看能不能直接在代码中找到这样的字符串。

![](https://files.mdnice.com/user/29990/0c4f793b-f6de-4bd5-911c-fc2c361682cd.png)
嗯，运气很好，直接就找到了，观察了下代码后会发现，主要的代码是这一句: `csdn.copyright.init(e, o)`，`o`就是版权文本的内容，`e`就是元素`blog-content-box`，原理找到了，自然就有了解决方案，再次进行初始化就好了，同时`o`的内容保持为空，可以试一下，示例代码如下：

```javascript
    var copy_element = document.querySelector('main .blog-content-box')
    csdn.copyright.init(copy_element, '')
```

## 先到这里
先暂时到这里，还有下节，本篇涉及到的代码总量不多，但思路讲的很细致，遇到问题怎么思考的，怎么一步一步去完成。希望大家能学到东西，下节在说说怎么出来关注展开全文和把代码加到油猴脚本中。