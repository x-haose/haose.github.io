---
slug: 32a0c6bd
author: "昊色居士"
title: "图片隐私之虚拟定位"
description: 
date: 2022-08-22T23:27:43+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Python"
]
categories: [
    "Python"
]
series:  [

]
---

# 图片隐私之虚拟定位（一）

## 前言

大家都知道，在我们拍照的时候有时候会附带我们的位置信息，那么这些位置信息是存放的什么地方的呢？就是图片的`EXIF`信息，里面会带着图片的位置、照相机、图片、作者等信息，而存在这些信息的图片就会对我们的隐私进行泄露，我们今天就来处理一下

## 目标

我们的目标是保护图片的隐私，也就是不暴露真实的位置信息，这样一来就有了两个方法：

- 删除位置信息
- 填充假的位置信息

这里我们采用假的位置信息这个方案，因为没有信息也是信息，在一个人认真找对方信息的时候，如果发生此路不通会立刻想其他的办法，但如果是告诉有路，但只不过这条路是错的，那就有意思了。

## 使用到的工具

> 工欲善其事，必先利其器。

### `EXIF`信息库

完成这个功能是要改变图片的`EXIF`信息，而`EXIF`存在于图片的二进制信息当中的某一块，我们可以根据理论去手写一个这样的功能，当然最简单就是寻找已经完成的库去使用，我们这次使用到的库是`piexif`，这个库是 python 中专门修改和查询`EXIF`信息的

> piexif 官方文档：[https://piexif.readthedocs.io/en/latest/about.html](https://piexif.readthedocs.io/en/latest/about.html)

### 假位置库

我们的目标既然是填充的假的位置，那么我们就要生成假的位置信息，这个假位置信息有几种方案：

- 根据经纬度规则来随机生成经纬度
- 使用第三方库来生成虚拟经纬度
- 根据范围地址生成虚拟经纬度

这里我们先采用第二种方案，也就是第三方库的方式，这里采用的库是号称`“最假的库”`，也就是`Faker`，这个库可以生成很多虚假的信息，其中就包括虚拟的经纬度、地址等信息。大家可以自行研究一下。

> Faker 官方文档：[https://faker.readthedocs.io/en/master/](https://faker.readthedocs.io/en/master/)

## 编码

代码方面分为初始化方法、生成原始经纬度方法、生成 EXIF 格式经纬度方法、写入 EXIF 方法，以及供外部调用的设置随机经纬度方法

### 构造函数

```python
def __init__(self, img_path=None):
    self.img_path = img_path
    self._faker = Faker()

    # 原始经纬度信息
    self.src_lat = None
    self.src_lng = None

    # Exif格式经纬度信息
    self.exif_lat = None
    self.exif_lng = None
    self.exif_lat_ref = None
    self.exif_lng_ref = None
```

传递了一个参数，为图片的路径，然后生成假信息库的对象，最后定义了经度、维度的原始变量和 exif 变量，带`ref`的变量是说明方向的

### 小数的经纬度转度、分、秒格式

由于我们生成出来的经纬度是小数格式的，但我们`Exif`中使用的格式却是度、分、秒的，且都为**整形**，所以需要做一定的转换

> 转换方法：如小数的经纬度为：10.121806，转换为度分秒为 10°7′18.501611351966858″，具体步骤如下
>
> - 度数(°)---10°；
> - 分数(′)---0.121806×60=7.308360189199448，取整数部分为 7，做为分数；
> - 秒数(″)---0.30836018919944763×60=18.501611351966858，做为秒数。
>   代码如下：

```python
    def transition_deg(self, latlng_deg):
        """
        把小数的度数格式转换成度、分、秒的格式
        Args:
            latlng_deg: 小数的经纬度

        Returns:
            返回度、分、秒的格式的元组
        """
        deg = int(math.modf(latlng_deg)[1])
        sec, minute = math.modf((latlng_deg - deg).copy_abs() * 60)
        minute = int(minute)
        sec *= 60 * 1000
        return (abs(deg), 1), (minute, 1), (int(sec), 1000), deg > 0
```

上面代码就是转换的过程，返回值是有4个，前三个的度、分、秒的元组，元组第一个是度分秒的值，且为正整形，第二个的倍数，也就是说真正的值是元组[0]/元组[1]，第四个值是度是是正的还是负的，我们把度数使用的绝对值，就要告诉上一层是正还是负，才好分别东西南北

### 设置exif经纬度
```python
    def set_exif_latlng(self, latlng: tuple):
        """
        设置exif经纬度
        Args:
            latlng: 经纬度的元组

        Returns:

        """
        self.src_lat, self.src_lng = latlng

        # 获取exif经纬度
        self.exif_lng = self.transition_deg(self.src_lng)
        self.exif_lat = self.transition_deg(self.src_lat)

        # 东经正数，西经为负数
        self.exif_lng_ref = "E" if self.exif_lng[3] > 0 else "W"
        self.exif_lng = self.exif_lng[:3]
        # 北纬为正数，南纬为负数
        self.exif_lat_ref = "N" if self.exif_lat[3] > 0 else "S"
        self.exif_lat = self.exif_lat[:3]
```
传来的参数是经纬度的元组，然后分别调用上面提到的转换方便得到exif的经纬度，在更加第四个参数设置东经、西经、南纬、北纬

### 设置图片Exif信息
这一步就是调用`piexif`的`insert`方法插入并覆盖gps的信息，更详细的值和使用方法可以自行查阅文档，示例代码:
```python
    def set_exif(self):
        """
        设置图片Exif信息
        Returns:

        """
        gps_ifd = {
            piexif.GPSIFD.GPSVersionID: (2, 0, 0),
            piexif.GPSIFD.GPSLongitude: self.exif_lng,
            piexif.GPSIFD.GPSLatitude: self.exif_lat,
            piexif.GPSIFD.GPSLongitudeRef: self.exif_lng_ref,
            piexif.GPSIFD.GPSLatitudeRef: self.exif_lat_ref,
        }
        exif_dict = {"GPS": gps_ifd}
        exif_data = piexif.dump(exif_dict)
        piexif.insert(exif_data, self.img_path)
```

### 生成随机的GPS定位信息
```python
    def set_random_gps(self):
        """
        生成随机的GPS定位信息
        Returns:

        """
        latlng = self._faker.latlng()
        self.set_exif_latlng(latlng)
        self.set_exif()
        print(f"已设置照片随机经纬度：（{self.src_lng}，{self.src_lat}）")
```
这里的代码就是调用之前的各个方法，大家可以看到我这里把调用`Faker`库的代码放到了这里，而不是`set_exif_latlng`方法里面，目的是达到`set_exif_latlng`方法的通用性，可以支持后面的其他获取经纬度并生成`Exif`信息。

## 效果图
运行结果：
```
已设置照片随机经纬度：（-178.114658，-65.4817155）
```
设置之前：
![](https://files.mdnice.com/user/29990/92e8bf46-c597-4043-b7a1-4aaca68d8248.png)

设置之后：
![](https://files.mdnice.com/user/29990/0b9635d8-9ade-4953-80ed-6c90cb4e4e9f.png)

网页查询信息：

![](https://files.mdnice.com/user/29990/21f79c0e-4bc6-4074-b688-415b72110c0a.png)

## 结语
这节根据图片Exif信息，写了个简单的脚本，没错。目前也只能称得上一个脚本，后面我们在继续完善，变成一个完成的程序，敬请期待！！！