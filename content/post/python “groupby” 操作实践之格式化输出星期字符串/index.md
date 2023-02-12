---
slug: 51fa43e9
author: "昊色居士"
title: "Python “Groupby” 操作实践之格式化输出星期字符串"
description: 
date: 2022-08-12T18:15:12+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Python", "groupby"
]
categories: [
    "Python"
]
series:  [

]
---

# python “groupby” 操作实践之格式化输出星期字符串

## 前言

最近遇到一个需求，是在一周内选择几天，并按格式输出出来，来说下完成这个功能的过程，如何从 0 到 1。

## 需求目标

一个星期几的多选字段，可以选择星期一到星期日，如果选择了星期一、星期二、星期三、星期五、星期六，则输出，星期一至星期三、星期五至星期六，如果选择了星期一、星期二、星期四、星期五、星期日，则输出，星期一至星期二、星期四至星期五、星期日

## 敲代码

### 敲之前

需求上是输出星期几的字符串，如果把星期直接换成数据，就变成了，连续数字分组的问题，比如：[1, 2, 4, 5, 7]，分成[1, 2]、[4, 5]、[7]，我们就是看这类问题在 python 中如何处理

### 不会啊，咋办

知道了是连续数字的分组又如何，还是不会啊，不会就百度啊

![](https://files.mdnice.com/user/29990/1f94e03a-f153-4fd6-8a64-7f470b6dbc3f.png)
去掉前面几个不靠谱的，可以看到，最后一个还是靠点谱的，点进去看看

![](https://files.mdnice.com/user/29990/93eeb196-a639-4d0a-bfe3-84955206fefd.png)
嗯，和我们的需求类似，看看下面有没有什么好的方案

![](https://files.mdnice.com/user/29990/2b5bc2f2-bf25-435e-b06a-6dc2170cdd0c.png)
这个看着是可以的，代码又少又简洁，还特意强调了`python3`可用。

### 分析

我们先写一个案例模拟一下，看看可不可以用：

```python
from itertools import groupby
from operator import itemgetter


def main():
    data = [1, 2, 4, 5, 7]
    ranges = []
    for k, g in groupby(enumerate(data), lambda x: x[0] - x[1]):
        group = list(map(itemgetter(1), g))
        ranges.append((group[0], group[-1]))

    print(ranges)


if __name__ == '__main__':
    main()

# 运行结果
[(1, 2), (4, 5), (7, 7)]
```

结果和预期的差不多，我们来分析一下，他是怎么实现的这个功能，先看下上面用到的知识点：

- groupby：对序列进行分组操作
- enumerate：把序列变成索引加值的方式，并返回迭代器
- lambda： 兰不达表达式，匿名函数
- map：对迭代中的每一项执行相同的方法，方法有一个参数就是序列的值
- itemgetter：返回一个使用`[]`的 **getitem**() 方法从`[]`中获取 item 的可调用对象（就是一个函数，参数是列表或字典）

上面简单说了下代码中使用到的一些函数，我们先从第一步看`enumerate`，上面说了这个方法是把现有列表的上面加上索引，先看一下。

```python
data = [1, 2, 4, 5, 7]
enumerate(data)
Out[4]: <enumerate at 0x1d942fb1140>
list(enumerate(data))
Out[5]: [(0, 1), (1, 2), (2, 4), (3, 5), (4, 7)]
```

可以很明显的看到，把原有的一维列表变成了由`(索引，值)`组成的二维列表，下一步就看看怎么用的这个二维列表，是当作了`groupby`的参数，`groupby`是用来分组的，第一个参数就是一个序列，第二个是一个`key`，就是分组的依据，再这个里面使用的是一个匿名函数，函数的参数肯定就是序列的每一个值，然后操作是用第 0 个元素减去第一个元素，把结果作为分组的依据，虽然到这里意思懂了，但还不知道为啥这样做，先看下分组的结果：

```python
data = [1, 2, 4, 5, 7]
ranges = {}
for k, g in groupby(enumerate(data), lambda x: x[0] - x[1]):
    ranges[k] = list(g)

ranges
Out[12]: {-1: [(0, 1), (1, 2)], -2: [(2, 4), (3, 5)], -3: [(4, 7)]}
```

我们利用上面的代码输出了一下`groupby`分组后的结果，就可以看出规律来了，再连续数字分组的问题中，重点就是如何判断数字的连续性，这里使用的方法就是用`索引减去数字的值`，只要是连续的数字，`索引减去数字的值`的结果一定是相等的。

咱们再往下看，后面对结果进行处理的时候使用了`group = list(map(itemgetter(1), g))`，相当于就是对`g`里面的每一项都执行了方法`itemgetter(1)`，而`itemgetter`的意思，上面也说了就是代替操作符`[]`，如下：

```python
l = (0, 1)
f = itemgetter(1)
f(l)
Out[15]: 1
```

所以这里的代码意思就是，取二维数组的第一个元素，也就是把`enumerate`加进去的索引再删掉。

### 带入业务

技术方向已经分析好了，接下来就是带入业务逻辑了，已知现有的数据结构是这样的：

```python
  # 周一
  monday = fields.Boolean(string='周一')
  # 周二
  tuesday = fields.Boolean(string='周二')
  # 周三
  wednesday = fields.Boolean(string='周三')
  # 周四
  thursday = fields.Boolean(string='周四')
  # 周五
  friday = fields.Boolean(string='周五')
  # 周六
  saturday = fields.Boolean(string='周六')
  # 周天
  sunday = fields.Boolean(string='周天')
```

获取字段的时候可以通过`data.monday`的方法来获取属性，也可以通过`data['monday']`的方式获取属性，所以第一步就是把 7 个`Boolean`变成一个`list`类型的数据，这时就可以动态的去判断属性，然后吧对应的`索引+1`，存入新的列表中，这时就成功的把数据结构转换成了我们想要的

```python
    week_attr_names = ['monday', 'tuesday', 'wednesday', 'thursday', 'friday', 'saturday', 'sunday']
    week_attr_names = [
        index + 1
        for index, week_attr in enumerate(week_attr_names)
        if rule_data[week_attr]
    ]
```

下面就可以带入上面的方法，处理字符串了：

```python
    week_dict = {1: '一', 2: '二', 3: '三', 4: '四', 5: '五', 6: '六', 7: '日'}
    week_list = []
    for k, g in groupby(enumerate(week_attr_names), lambda x: x[0] - x[1]):
        group = list(map(itemgetter(1), g))
        week_list.append('至'.join(map(lambda s: f"周{week_dict[s]}", {group[0], group[-1]})))
```

不知道大家有没有发现，有一个不同的地方，就是把`(group[0], group[-1])`变成了`{group[0], group[-1]}`，这其中的区别就是后者的类型是`set`类型也就是会自动去重的，否则会出现`星期日至星期日`的情况。

## 结语

这节主要说了 python 中`itertools`中的几个方法，和分析需求到，查询资料，到分析资料中的代码的过程。所以遇到问题，可以查资料，但是查了资料后一定要懂，为啥这么写，好还是不好，有没有啥改善的地方。就这么吧，下篇更精彩！！！