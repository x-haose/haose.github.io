---
slug: 40877897
author: "昊色居士"
title: "打卡软件破解（三）逆向实践"
description: 
date: 2022-08-31T23:40:37+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "逆向", "破解"
]
categories: [
    "逆向"
]
series:  [
    "打卡软件破解"
]
---

## 前言

我们上一节简单的介绍了安卓逆向的相关知识，以及分析的我们要修改的代码，这一节就来实际操作一下！

## 声明

> 本人所有逆向、破解及爬虫相关教程均是以纯技术的角度来探讨研究和学习，严禁使用教程中提到的技术去破解、滥用、伤害其他公司及个人的利益，以及将以下内容用于商业或者非法用途。

## 修改代码

### 工具下载

这里我们要用到另一个工具，也是比较常用的，叫做: apktool，可以解压和打包apk文件，下载地址及安装方法请按照文档：[https://ibotpeaches.github.io/Apktool/install/](https://ibotpeaches.github.io/Apktool/install/)上面说的去做	

下载好之后，我们打开终端，cd到工具的目录下面输入：`.\apktool.bat d "D:\Android\apk\xx.apk"`即可解压当前apk

### 修改smail文件

下载之后我们打开这个文件夹，我这里使用的是`VS CODE `打开的，打开后界面如下

![](https://images.haose.pro/2022/8/31/1661960888353_23%3A48%3A08_h252ne4wfp.png)

其中框选的地方就是反编译的`smali`代码，我们修改的也是这部分代码，然后我们就要找到即将要修改的文件位置，我们回到`jadx`里面，看文件的开头，就会定义这个文件的路径，如下图：

![](https://images.haose.pro/2022/8/31/1661961026937_23%3A50%3A26_xobj62bjt3.png)

我们也可以直接使用`jadx`的smali视图，直接观看代码

![](https://images.haose.pro/2022/9/1/image-20220831235455784_08%3A51%3A59_4a4b01fhle.png)

然后在`vode`中搜索关键子，直接找到相关的文件，在搜索之后会找到这个文件

![](https://images.haose.pro/2022/9/1/1661961739222_00%3A02%3A19_6qwcach2tr.png)

这个代码就是安卓的`smali`代码，`smali`代码的语法是很复杂，这里不做过多的展开，大家要知道的是前三行就是定义的`@JavascriptInterface`后面是逻辑代码，各种逻辑判断、调用函数、赋值变量等，但我们不需要知道那么多，我们要做的只是把一个固定的字符串赋值给一个变量然后返回即可，查查资料然后照葫芦画瓢之后，可以这么写：

```smali
.method public final getLatitude()Ljava/lang/String;
    .locals 2
    .annotation runtime Landroid/webkit/JavascriptInterface;
    .end annotation

    .line 1
    const-string v0, "30.630451"
    return-object v0
.end method

.method public final getLongitude()Ljava/lang/String;
    .locals 2
    .annotation runtime Landroid/webkit/JavascriptInterface;
    .end annotation

    const-string v0, "104.043425"
    return-object v0
.end method

```

这里我们只选择修改了经纬度的方法，在修改之后就可以打包了，打包依旧使用`apktool`

## 打包

### 打包APK

打包使用的是`apktool b xxx`命令，其中xxx是文件夹的路径，我们来打包执行一下试试：

![](https://images.haose.pro/2022/9/1/1661962337409_00%3A12%3A17_1ziuhjksdo.png)

执行完毕后会在`xxx/dict`目录下生成打包好的apk包，这时我们可以再次把apk拖进`jadx`中看看修改之后的结果：

![](https://images.haose.pro/2022/9/1/1661963039981_00%3A24%3A00_tpdohpokwl.png)

可以看到已经修改成功了

### 签名

现在的这种状态的包是安装不上当，因为没有经过签名，所以我们接下来要对apk进行签名，还是会使用到一个工具：**[uber-apk-signer](https://github.com/patrickfav/uber-apk-signer)**，这个工具会自动会apk进行v1、v2、v3签名

下载好后，我们使用如下命令对apk进行签名：

```
java -jar uber-apk-signer.jar -a xx.apk
```

不出意外的你会看到如下的结果：

![](https://images.haose.pro/2022/9/1/1661963426767_00%3A30%3A26_9mg9f6pqlk.png)

然后就像安装测试即可

## 这回结束了

一次安卓逆向实践就此结束了，没有说的太细，但把整体的流程呈现给了大家，大家如果真的感兴趣的话是可以单独对某一个技术点单独学习的，比如smli的语法，apk包的具体结构、签名的流程等，毕竟只是入门嘛，祝大家学有所成！！！