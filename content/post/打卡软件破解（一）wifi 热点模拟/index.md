---
slug: e21238d0
author: "昊色居士"
title: "打卡软件破解（一）wifi 热点模拟"
description: 
date: 2022-08-16T23:33:41+08:00
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

# 打卡软件破解 -- （一）wifi 热点模拟

## 声明

> 本人所有逆向、破解及爬虫相关教程均是以纯技术的角度来探讨研究和学习，严禁使用教程中提到的技术去破解、滥用、伤害其他公司及个人的利益，以及将以下内容用于商业或者非法用途。

## 原理

上一篇文章介绍了打卡软件破解的思路，和一些方法。这些就来专门说说WIFI的的破解思路。wifi打卡的判断方法根据打卡者是否连接对应的wifi来判断，那么一个WiFi都有啥特征呢？
* wifi名字
* wifi密码
* IP地址
* mac地址

其中wifi名字是可以随便设置的，IP地址是和所在路由的网关配置有关，也可以相当于是随便设置的，那么mac地址呢，这个是所在网卡的地址是不可轻易改动的。然后我们猜测的是这种，可能也是很大的，在通过网络上查询看看：

![](https://files.mdnice.com/user/29990/c99f3b3f-dd39-4172-a5a2-aada4951eb78.png)

![](https://files.mdnice.com/user/29990/6a481433-938c-4318-8829-8b67ad413f93.png)

![](https://files.mdnice.com/user/29990/731a3e13-3dea-4db5-b8da-afd1ecbdcce1.png)

和我们猜测的差不多，因为一个WiFi的特征就这么多，只要挨个模拟一下就好，下面我们就来使用一个自家的笔记本电脑来模拟一下`“公司的WIFI”`，环境是带无线网卡的WIN10或WIN11系统电脑。

## WIFI名称
这个不用说，直接在设置里面修改成一样的就可以，谨慎的可以连密码都修改成一样的

## IP地址
如果真的判断了IP地址，那么判断的就一样的网关地址，所以在正常情况下连接正常的wifi，查询到ip地址的网段，比如：`192.168.105.1`，然后把笔记本电脑的移动热点网段也修改成相同的，但大家会发现开启之后网段会自动变成：`192.168.137.1`，下面就看看如果来修改：
### 一、常见方法
在这里有一个网络上普遍流传的版本，但是在我电脑上没有用，我的系统是`WIN11`的，但供大家参考，自行尝试：
打开之后，直接在路径上面输入：`HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\SharedAccess\Parameters`

![](https://files.mdnice.com/user/29990/99cd0091-9d39-48d7-87ac-f54273a302ae.png)
修改之后重新电脑，重新打开热点后查看网段是否修改成功。

### 二、我成功的方法
下面说说我成功的方法，既然上面的方法修改不成功，但确实是在注册表中找到了移动热点的默认网段：`192.168.137.1`，那么我们就按照如此思路在搜一下看看：

![](https://files.mdnice.com/user/29990/4e86f0a9-8857-4761-b660-94a644a3d12a.png)

可以看到这里还有一个值为`192.168.137.1`的项，我们把它也修改一下，但也会发现报错了：

![](https://files.mdnice.com/user/29990/c2faec6b-438c-41ec-b4f1-517e20fbfb79.png)

既然说了是`无法编辑`和`写入时出错`，我们就从权限这里入手。

![](https://files.mdnice.com/user/29990/b8e37a91-3310-457f-808d-45f1f2e06d07.png)
所在文件夹点击右键权限，这里就和文件夹操作很类似了。先看所在用户组有没有权限，能不能编辑权限，不能就点击高级，看看所有者是谁，如果不是当前用户，就改为当前用户。

![](https://files.mdnice.com/user/29990/b1f95892-fa21-4ad4-b29d-b59a89b327fe.png)

![](https://files.mdnice.com/user/29990/06d03cc3-bab4-43da-bd84-15bb8b4d5509.png)

![](https://files.mdnice.com/user/29990/341311c3-f52a-4879-9fd7-cb34fc4c0d73.png)

![](https://files.mdnice.com/user/29990/3e7f5200-214f-42b9-88f0-725d3fd2e1b7.png)

点击更改->高级->立即查找->找到自己的用户名->确定->应用，然后在修改权限。都完成后就可以把`192.168.137.1`修改成自定义的网段了，比如: `192.168.99.1`

![](https://files.mdnice.com/user/29990/5824bd88-222c-445d-96ac-22cbdaba16c7.png)

## 修改MAC地址
### 查看当前地址
然后在进行修改网卡地址，先查看当前的网卡地址，控制面板中有这个界面

![](https://files.mdnice.com/user/29990/be4f8c0e-e5a3-4403-b965-d41831de6323.png)

### 查看目标地址
使用软件也好查询也好，抓包也好，查资料也好，就不多说了（有软件可以看到的，快去查百度谷歌），这里假定你已经知道了所连接网络的MAC地址，如："88:DC:96:6D:4B:99"

### 修改
打开电脑的设备管理器，找到你的无线网卡，点击属性，高级，查看属性里面是否有网络地址的选项

![](https://files.mdnice.com/user/29990/5c63b1cb-f0ad-42c8-9915-5000eb48a5de.png)

没有的话，打开注册表，路径为：`\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}`

![](https://files.mdnice.com/user/29990/b333fb89-5876-4ae2-b1ae-c76dd2e6ae37.png)

如上图所示，在子文件夹中找到：`DriverDesc`为你的 **`有线网卡`** 的文件夹，然后定位到，下面的子文件夹："\Ndi\Params\NetworkAddress"中，然后右键点击导出.如下图：

![](https://files.mdnice.com/user/29990/1fa80b60-38b3-47f6-8f51-66696c9ee2f4.png)

![](https://files.mdnice.com/user/29990/0ade36ea-eb81-4d4b-9ac4-60915ebf5cbe.png)

导出保存文件之后，打开文件，使用如上方法把里面的数字更改成无效网卡的数字

![](https://files.mdnice.com/user/29990/4a46b8ef-9bfb-4e46-9413-e5fe901a9399.png)
然后双击导入即可

### 修改地址
成功导入之后，即可发现有网络地址这一项了
![](https://files.mdnice.com/user/29990/bcc553ac-a068-4a0f-9f2b-47813ffe0eee.png)

这里要注意一下：修改的网卡的主wifi网卡设备，但win10或win11自带开热点的是虚拟网卡，虚拟网卡地址不能直接设置，他的规则是在主wifi网卡mac地址的第二位减去二，比如虚拟网卡前两位要设置88，wifi网卡则设置8a，咱们这次目标的地址是："88:DC:96:6D:4B:99"，所以我们要设置网络地址为："8ADC966D4B99"，改好之后直接确定即可，然后我们在重新开启移动热点看看地址修改成功了吗

![](https://files.mdnice.com/user/29990/bdf43a8a-3e38-4b2a-9065-5faed58ebcaf.png)

已经成功了！！！

## 结语
完结撒花！！！再次声明一下下：严禁用于非法用途和作弊，仅限于技术学习