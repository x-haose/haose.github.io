---
slug: 9fbf6831
author: "昊色居士"
title: "油猴脚本开发之 CSDN（下）"
description: 
date: 2022-08-04T23:54:16+08:00
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

# 油猴脚本开发之 CSDN（下）

## 声明

> 本人所有逆向、破解及爬虫相关教程均是以纯技术的角度来探讨研究和学习，严禁使用教程中提到的技术去破解、滥用、伤害其他公司及个人的利益，以及将以下内容用于商业或者非法用途。

## 关注展开全文

![](https://files.mdnice.com/user/29990/d05a3c03-c845-4e27-bcd4-d30fed90e34a.png)
大家对于这个界面应该不陌生，在网上搜一个问题，然后随手打开一个网页，发现答案还挺匹配的，看了一半，哎，看不了了。这个界面就出来了。我们同样以`纯技术角度`来研究研究怎么看到剩下的文章。

### 方向

同样先找到方向，回想一下上一篇文章是根据禁用 js 脚本来判断使用的方法是`js`还是`css`。我们可以使用相同的办法来试一下，禁用 js 来看看内容是否加载全了。然后会发现全部加载出来了，到此为止如果只是想看的话已经可以看到了。但这不优雅，我们来继续。

### 关键代码

既然是 js 控制的，我们就要找到控制的部分，先按关键字符串找一下试试，全局搜一下：`关注博主即可阅读`试试

![](https://files.mdnice.com/user/29990/f0a9eda7-699d-4ccb-8ef4-9ba1ec1dfa27.png)
可以看到只有一条记录就是在网页本身里面，直接搜没有搜索到关键代码，也不要紧。就要考虑一下他的实现方案，即使把相关的字符串都直接写到了`html`，那么他要想控制这个`字符串`的显隐，则只能是通过元素的控制，既然是控制元素，就一定要查找到这个元素，这就很清晰了，只需要观察这个字符串所在的上下元素分别有什么特征，如：`classs`、`id`等。

![](https://files.mdnice.com/user/29990/cd615da0-9c4e-4285-8863-af15039b136a.png)
可以看到这几个元素都是比较可疑，我们来依次进行全局搜索一下

![](https://files.mdnice.com/user/29990/c1c2406e-4eb5-4b54-aaa7-c1a7d21cc6ee.png)
可以看到在搜：`hide-article-box`的时候已经在 js 中搜索到了相关的代码，随便选择一行点进去，然后进行格式化。

### 寻找控制代码

![](https://files.mdnice.com/user/29990/74e126bf-4053-4af9-8b05-da3fee68903c.png)
既然是搜索`hide-article-box`出现的结果，我们就看看这个元素是干嘛的，可以看到这个元素就是控制的根元素，而且经常搜索后，可以看到有`$(".hide-article-box").hide()`和`$(".hide-article-box").show()`的逻辑，很明显就是控制显示和隐藏的，可以在控制台直接调用可以看看，正如我们所料，就是这个元素控制的。但是内容还没有出来，还要继续看，既然这段代码中是控制隐藏的，那么也一定会有显示的逻辑，我们来自己看看这段代码

![](https://files.mdnice.com/user/29990/527a5b6b-9a55-4065-965e-8c3ca999076d.png)
仔细看代码，可以发现真正控制元素的只有画圈的这几个，其他的就是一些判断而已

![](https://files.mdnice.com/user/29990/bae146c7-189d-47f4-b9c3-54b560335b90.png)
可以看到这个是自动点击关注的逻辑肯定不是。

![](https://files.mdnice.com/user/29990/6b8c068e-7c80-4c78-8b0a-887db48edb70.png)

注意看这里，在搜索到这个的时候网页上面很多区域都被选中了，而且还有样式是一个设置高度，另一个字面意思是隐藏什么相关的，而且代码中也写的是删除样式。所以这个的可能性很高，同样复制到控制台来看看。哎，成了，就是这个！所以这部分代码就两行：
```
$(".hide-article-box").hide()
$("div.article_content").removeAttr("style")
```

## 油猴脚本
油猴脚本是一个浏览器的扩展，可以在特定网页上面运行自定义的脚本，可以使用扩展：`Tampermonkey`或`脚本猫`，前者是官方的，后者是兼容前者的基础上加了一些自定义的功能。

![](https://files.mdnice.com/user/29990/143dc745-9d21-4952-8b1a-d752c6d1edeb.png)

上面是配置，下面是代码，配置中主要注意`match`这个是匹配哪些网址，可以写多条。最后附上所有代码：
```javascript
// ==UserScript==
// @name         CSDN清除复制
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  
// @author       You
// @match        *.csdn.net/*
// @icon         https://www.google.com/s2/favicons?domain=csdn.net
// @grant        none
// ==/UserScript==

(function () {
    // 允许不登录复制
    for (const e of document.querySelectorAll('#content_views pre')) {
        e.style.setProperty('user-select', 'text', 'important')
    }
    for (const e of document.querySelectorAll('#content_views pre code')) {
        e.style.setProperty('user-select', 'text', 'important')
    }

    // 复制不带版权
    var copy_element = document.querySelector('main .blog-content-box')
    csdn.copyright.init(copy_element, '')

    // 登录后复制按钮 data-title 改成随便复制并删除点击事件
    document.querySelectorAll('.hljs-button').forEach(
        doc => {
            doc.setAttribute("data-title", "随便复制")
            doc.setAttribute("style", "pointer-events: none;!important")
        }
    )
    
    // 不关注博主即可看全文
    $(".hide-article-box").hide()
    $("div.article_content").removeAttr("style")
})();
```

## 结束
好这次的油猴脚本开发结束了，代码上来说不难，主要是寻找问题的思路要清晰。一步一步递进。愿大家学有所成。