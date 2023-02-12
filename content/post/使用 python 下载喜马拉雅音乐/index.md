---
slug: 913fa4bc
author: "昊色居士"
title: "使用 Python 下载喜马拉雅音乐"
description: 
date: 2022-08-19T23:25:51+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "爬虫", "Python", "逆向"
]
categories: [
    "Python", "逆向"
]
series:  [

]
---

# 使用 python 下载喜马拉雅音乐

## 声明

> 本人所有逆向、破解及爬虫相关教程均是以纯技术的角度来探讨研究和学习，严禁使用教程中提到的技术去破解、滥用、伤害其他公司及个人的利益，以及将以下内容用于商业或者非法用途。

## 前言

喜马拉雅音乐是默认不能直接下载的，需要下载手机 App 后才能下载，不想下载手机 App 又想下载音乐咋整呢？用 python 搞定！

## 找到下载地址及信息

我们先随便打开一个音乐，然后打开浏览器的控制台调到 network 选项卡，在选卡 XHR 过滤。然后刷新一下网页，network 会出现打开这个网页过程中的所有请求，XHR 是指发起的 AJAX 请求，是一种用于创建快速动态网页的技术，很多网站都会使用到这个。

![](https://files.mdnice.com/user/29990/ed5d9de1-6867-4e0d-a29b-e7a16a0418d3.png)

我们看一下这些请求，点开然后切换至`Preview`选项卡，这个是请求的响应预览界面,会对响应结果进行格式化，`Response`是响应是直接结果，不会有格式化。
通过观察我们可以发现，有两个主要的请求来加载音乐：

- https://www.ximalaya.com/revision/seo/getTdk ：获取音乐的详细信息
- https://www.ximalaya.com/revision/play/v1/audio ：获取音乐的播放地址

![](https://files.mdnice.com/user/29990/3091fb68-eb81-4778-898b-042692a0059c.png)

![](https://files.mdnice.com/user/29990/cd167d02-51f2-41e1-a829-66160997b73e.png)

知道了这两个请求后我们就可以根据这两个请求来下载音乐及获取信息

## 开始编码

### 首先来编写获取音乐信息的方法：

url：https://www.ximalaya.com/revision/seo/getTdk
方法：get
参数：{"typeName: "TRACK": "uri": "音乐播放地址的路径"}
响应：音乐的基本信息及相关推荐，json 类型
使用`requtst`库来编写代码

```python
def get_music_name(self):
        """
        获取音乐名字
        :return:
        """
        url = "https://www.ximalaya.com/revision/seo/getTdk"
        data = {
            "typeName": "TRACK",
            "uri": urlparse(self.web_url).path
        }
        headers = {"User-Agent": self.ua.random}
        try:
            response = requests.get(url, params=data, headers=headers)
        except Exception as e:
            print(e)
        else:
            self.music_name = response.json()['data']['tdkMeta']['keywords']
```

### 获取音乐下载地址的方法

url：https://www.ximalaya.com/revision/play/v1/audio
方法：get
参数：{"ptype: 1: "id": "音乐播放地址的最后一段"}
响应：音乐的基本信息及相关推荐，json 类型

```python
def get_music_url(self):
    """
    获取音乐下载地址
    :return:
    """
    url = "https://www.ximalaya.com/revision/play/v1/audio"
    data = {
        "ptype": 1,
        "id": self.web_url.split('/')[-1]
    }
    headers = {"User-Agent": self.ua.random}
    try:
        response = requests.get(url, params=data, headers=headers)
    except Exception as e:
        print(e)
    else:
        self.music_url = response.json()['data']['src']
```

这两段代码都是简单的调用 get 方法来获取响应。需要注意的是请求的同时需要带上 User-Agent，这个字段标记了使用什么来访问的 url，一般都会做这个验证，不是浏览器或允许的客户端禁止访问，这里是使用的`fake_useragent`库来随机获取 User-Agent

```python
import os
import requests
from fake_useragent import UserAgent
from urllib.parse import urlparse


class DownloadXmla(object):
    music_url = ""
    music_name = ""

    def __init__(self, url: str, save_dir: str):
        self.web_url = url
        self.ua = UserAgent()
        self.get_music_name()
        self.get_music_url()
        self.save_dir = save_dir
```

### 下载音乐

地址获取到了，下载就很简单了，有很多方法，比如：request.get()获取二进制获取写入文件、wget 命令、其他的第三方库及命令等
这里是简单的 request.get()获取二进制获取写入文件的方法

```python
def download(self):
    """
    下载文件的方法
    只是一个简单的写入文件，可以用更多的方法来下载
    :return:
    """
    path = os.path.join(self.save_dir, self.music_name + ".m4a")
    base_dir = os.path.dirname(path)
    # 文件夹不存在创建文件夹
    if not os.path.exists(base_dir):
        os.makedirs(base_dir)

    # 文件存在删除文件
    if os.path.exists(path):
        os.remove(path)
    try:
        response = requests.get(self.music_url)
        with open(path, 'wb') as f:
            f.write(response.content)
    except Exception as e:
        print(e)
```

### 运行试试

```python
def main():
    xmla = DownloadXmla("https://www.ximalaya.com/sound/59636711", '.')
    xmla.download()


if __name__ == '__main__':
    main()

```

## 编写结束

到此为止已经可以简单的爬取喜马拉雅上面的音乐了，应该说是下载，不应该说是爬取，因为只是单个的，爬取的话只是把流程自动化、批量化了而已。
代码本身不难，值得注意的就是：

- 能够利用浏览器找到合适的接口
- User-Agent 这里，因为不加这个也会成功，返回 200.但是没有返回数据
- 文件的保存，写入文件的时候不要忘记判断文件夹是否存在，不存在则需要创建
- 下载好的音乐的 m4a 文件，可以根据需要转成 mp3

## 附上代码：
```python
import os
import requests
from fake_useragent import UserAgent
from urllib.parse import urlparse


class DownloadXmla(object):
    music_url = ""
    music_name = ""

    def __init__(self, url: str, save_dir: str):
        self.web_url = url
        self.ua = UserAgent()
        self.get_music_name()
        self.get_music_url()
        self.save_dir = save_dir

    def get_music_name(self):
        """
        获取音乐名字
        :return:
        """
        url = "https://www.ximalaya.com/revision/seo/getTdk"
        data = {
            "typeName": "TRACK",
            "uri": urlparse(self.web_url).path
        }
        headers = {"User-Agent": self.ua.random}
        try:
            response = requests.get(url, params=data, headers=headers)
        except Exception as e:
            print(e)
        else:
            self.music_name = response.json()['data']['tdkMeta']['keywords']

    def get_music_url(self):
        """
        获取音乐下载地址
        :return:
        """
        url = "https://www.ximalaya.com/revision/play/v1/audio"
        data = {
            "ptype": 1,
            "id": self.web_url.split('/')[-1]
        }
        headers = {"User-Agent": self.ua.random}
        try:
            response = requests.get(url, params=data, headers=headers)
        except Exception as e:
            print(e)
        else:
            self.music_url = response.json()['data']['src']

    def download(self):
        """
        下载文件的方法
        只是一个简单的写入文件，可以用更多的方法来下载
        :return:
        """
        path = os.path.join(self.save_dir, self.music_name + ".m4a")
        base_dir = os.path.dirname(path)
        # 文件夹不存在创建文件夹
        if not os.path.exists(base_dir):
            os.makedirs(base_dir)

        # 文件存在删除文件
        if os.path.exists(path):
            os.remove(path)
        try:
            response = requests.get(self.music_url)
            with open(path, 'wb') as f:
                f.write(response.content)
        except Exception as e:
            print(e)


def main():
    xmla = DownloadXmla("https://www.ximalaya.com/sound/59636711", '.')
    xmla.download()


if __name__ == '__main__':
    main()

```