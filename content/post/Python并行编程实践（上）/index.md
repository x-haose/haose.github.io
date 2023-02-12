---
slug: bec1ae1d
author: "昊色居士"
title: "Python并行编程实践（上）"
description: 
date: 2022-07-31T20:59:51+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Python", "并行", "线程池", "进程池"
]
categories: [
    "Python"
]
series:  [
    "Python并行编程实践"
]
---

# Python并行编程实践（上）

## 前言

在`python`中关于并行相关的也不少了，比如：进程、线程、协程、异步等等，资料也不少。我今天也来浅浅的说一下这个。

## 进程和线程

这里准备用到的多进程和多线程均会使用`concurrent.futures`库来实现，是 3.2 版本的出现的功能，但我相信不会人还使用着小于 3.2 版本的 python 吧，不会吧，不会吧

### 导入

在`concurrent.futures`中线程池和进程池在使用上面是很像很像的，看起来只是引用的类不同，线程池是：`ThreadPoolExecutor`，进程池是`ProcessPoolExecutor`。

### 实例化线程池/进程池

一般有两种方式，一个是使用`with`上下文管理器的方式创建，一个是直接创建对象的方式。如果是用的地方逻辑比较少，几行代码就写完了，那就直接使用`with`上下文管理器，方便快捷不用自己关闭，如果涉及的逻辑比较复杂，用到的地方比较多就可以自己创建对象自己管理，根据需要自己选择

```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor


def test(index):
    time.sleep(index)


def main():
    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        pass

    # 手动创建和关闭
    pool = ThreadPoolExecutor()
    pool.shutdown()


if __name__ == '__main__':
    main()
```

再看看参数，大部分时候需要关注的就一个`max_workers`，代表最大工作线程/进程的数量，这个数量可以自己指定，不指定的话也会有一个默认值

- ThreadPoolExecutor 的默认值为 cpu 的数量加 4，但不会大于 32
- ProcessPoolExecutor 的默认值为 cpu 的数量，但在 Windows 平台上面最大不会超过 61 个

### submit 提交任务

`submit`方法是单次提交任务，下面是函数签名：

```python
def submit(self, fn, /, *args, **kwargs):
  pass
```

fn 参数是要提交的函数名字，不用加括号，`*args, **kwargs`是函数的参数，直接写进去就行，下面看示例

```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor


def test(index, index_2=None):
    print(f"索引：{index} {index_2}")
    time.sleep(2)


def main():
    numbers = [1, 2, 3, 4, 5, 6]

    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        for i in numbers:
            pool.submit(test, i, index_2=i)


if __name__ == '__main__':
    main()

# 运行结果
索引：1 1
索引：2 2
索引：3 3
索引：4 4
索引：5 5
索引：6 6
```

### map 提交任务

上面是一个一个的提交任务，这个是批量提交任务，下面是函数签名：

```python
def map(self, fn, *iterables, timeout=None, chunksize=1):
  pass
```

大部分时候看第一个、第二个参数就行，第一个参数是要调用的函数名字，第二个是迭代器就是`list、set、tuple`之类的，但这次这个函数默认只能穿一个参数就是迭代是数据，如下：

```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor


def test(index):
    print(f"索引：{index}")
    time.sleep(2)


def main():
    numbers = [1, 2, 3, 4, 5, 6]
    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        pool.map(test, numbers)


if __name__ == '__main__':
    main()
```

但如果你的函数真的有好几个参数，你还非要用 map，你可以使用偏函数`functools.partial`重新构造一个函数，看例子就懂了：

```python
import time
import functools
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor


def test(index, arg_2):
    print(f"索引：{index} {arg_2}")
    time.sleep(2)


def main():

    numbers = [1, 2, 3, 4, 5, 6]
    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        pool.map(functools.partial(test, arg_2=2), numbers)


if __name__ == '__main__':
    main()

# 结果
索引：1 2
索引：2 2
索引：3 2
索引：4 2
索引：5 2
索引：6 2
```

### map 获取结果

上面只是提交任务，提交之后任务会自动执行，先说说`map`获取结果的方式，map 会`“直接返回结果的列表”`，严格来说不是这样的，但可以这么理解，就是这样：

```python
import time
import functools
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor


def test(index, arg_2):
    print(f"索引：{index} {arg_2}")
    time.sleep(2)
    return index


def main():
    numbers = [1, 2, 3, 4, 5, 6]
    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        results = pool.map(functools.partial(test, arg_2=2), numbers)
        for result in results:
            print(f"结果：{result}")


if __name__ == '__main__':
    main()

# 结果
索引：1 2
索引：2 2
索引：3 2
索引：4 2
索引：5 2
索引：6 2
结果：1
结果：2
结果：3
结果：4
结果：5
结果：6
```

### submit 获取结果

下面在看看 submit 获取结果的方法，使用上面略有不同，稍微麻烦一点点，需要用到`as_completed`来等待完成，直接看例子就懂了：

```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed


def test(index, arg_2):
    print(f"索引：{index} {arg_2}")
    time.sleep(2)
    return index


def main():
    numbers = [1, 2, 3, 4, 5, 6]
    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        results = [pool.submit(test, index, 2) for index in numbers]
        for result in as_completed(results):
            print(f"结果：{result.result()}")


if __name__ == '__main__':
    main()
```

获取结果的时候需要使用`.result()`方法

### 捕获异常

线程池或进程池中的函数如果发生了异常，程序是不会中断的，也不会直接捕获到，比如下面：

```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed


def test(index, arg_2):
    raise ValueError("报错了")
    print(f"索引：{index} {arg_2}")
    time.sleep(2)
    return index


def main():
    numbers = [1, 2, 3, 4, 5, 6]
    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        pool.map(functools.partial(test, arg_2=2), numbers)
        for index in numbers:
            pool.submit(test, index, 2)


if __name__ == '__main__':
    main()
```

你会发现程序什么也没输出也没有被中止，但你如果尝试着使用捕获返回值的方式运行程序，就是发现程序抛出了异常被中止了，所以捕获异常就是在这个时候去捕获:
map 捕获异常

```python
import time
import functools
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor


def test(index, arg_2):
    raise ValueError("报错了")
    print(f"索引：{index} {arg_2}")
    time.sleep(2)
    return index


def main():
    numbers = [1, 2, 3, 4, 5, 6]
    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        results = pool.map(functools.partial(test, arg_2=2), numbers)
        try:
            for result in results:
                print(f"结果：{result}")
        except Exception as e:
            print(e)


if __name__ == '__main__':
    main()
```

submit 捕获异常

```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed


def test(index, arg_2):
    raise ValueError("报错了")
    print(f"索引：{index} {arg_2}")
    time.sleep(2)
    return index


def main():
    numbers = [1, 2, 3, 4, 5, 6]
    # 上下文管理器，会自动关闭
    with ProcessPoolExecutor() as pool:
        results = [pool.submit(test, index, 2) for index in numbers]
        for result in as_completed(results):
            try:
                print(f"结果：{result.result()}")
            except Exception as e:
                print(e)


if __name__ == '__main__':
    main()
```

可以看出不一样的是，map 捕获异常只能在循环的外面，但 submit 在里面和外面都可以。

### 差不多了

如题，基本的使用差不多了就这样了，当然这个库的知识还是很多的，我只是带大家入个门，更深入的可以去看相关的文档去了解：

- [官方文档](https://docs.python.org/zh-cn/3/library/concurrent.futures.html)
- [Python Cookbook](https://python3-cookbook.readthedocs.io/zh_CN/latest/c12/p07_creating_thread_pool.html)
- [python-parallel-programmning-cookbook](https://python-parallel-programmning-cookbook.readthedocs.io/zh_CN/latest/chapter4/02_Using_the_concurrent.futures_Python_modules.html#)

## 下节继续

这节就先到这里了，下节来说剩下的内容。