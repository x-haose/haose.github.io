---
slug: 35db0656
author: "昊色居士"
title: "Python并行编程实践（下）"
description: 
date: 2022-08-01T22:43:20+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Python", "并行", "异步"
]
categories: [
    "Python", "异步"
]
series:  [
    "Python并行编程实践"
]
---

# Python并行编程实践（下）

## 协程与异步

这里主要使用的就是`asyncio`这个库，用来编写并发代码，使用 async/await 语法，至于协程和异步什么关系，可以参考官方文档的一段话：

> 协程是子例程的更一般形式。子例程可以在某一点进入并在另一点退出。 协程则可以在许多不同的点上进入、退出和恢复。 它们可通过 async def 语句来实现

### 运行程序

异步程序的运行在不同的 python 版本有不同的方法，如果你在网上看如下这种运行异步程序的方法：

```python
import asyncio

async def hello_world():
    print("Hello World!")

loop = asyncio.get_event_loop()
loop.run_until_complete(hello_world())
loop.close()
```

上面写的也是没错的，但此方法是 python3.7 之前运行异步程序的方法，在 3.7 之后有了更简便的写法，如下：

```python
import asyncio

async def main():
     print('hello')
     await asyncio.sleep(1)
     print('world')

asyncio.run(main())

```

也建议大家使用更高级的写法，因为谁不想少写点代码呢。

### 基本使用

在 python 中，如果想直接调用一个使用`async`装饰的异步函数，一点要在前面加`await`等待运行结束

### 创建任务

当你想创建一个后台任务，让它慢慢执行的时候，可以用这个：`asyncio.create_task(coro, *, name=None)`其中 name 形参是 3.8 加入进去的，意思是任务的名字，使用实例如下：

```python
import asyncio


async def task():
    await asyncio.sleep(3)
    print("任务执行完毕")


async def factorial(name, number):
    f = 1
    await asyncio.sleep(1)
    f *= number
    print(f"Task {name}: factorial({number}) = {f}")
    return f


async def main():
    asyncio.create_task(task())
    await factorial("A", 1)
    await factorial("B", 1)
    await factorial("C", 1)


if __name__ == '__main__':
    asyncio.run(main())

# 运行结果：
Task A: factorial(1) = 1
Task B: factorial(1) = 1
任务执行完毕
Task C: factorial(1) = 1
```

可以看到我们在开始就创建了一个后台任务`task`，该任务会在 3 秒后输出内容，随后又调用了三次函数`factorial`，该函数会等待 1 秒后返回，所以输出了以下内容，但如果你们尝试把函数`factorial`只运行 1 此或两次，会发现任务`task`没有执行完程序就结束了，因为我们只是把它当作一个后台任务来运行，并没有等待，这时有两种方案：

- 如果你的程序的服务端程序就不用管了，反正不会结束
- 如果你的代码就是一个脚本可以在最后等待后台任务的结束：

```
import asyncio


async def task():
    await asyncio.sleep(3)
    print("任务执行完毕")


async def factorial(name, number):
    f = 1
    await asyncio.sleep(1)
    f *= number
    print(f"Task {name}: factorial({number}) = {f}")
    return f


async def main():
    t = asyncio.create_task(task())
    await factorial("A", 1)
    await t

if __name__ == '__main__':
    asyncio.run(main())

# 运行结果
Task A: factorial(1) = 1
任务执行完毕
```

### 并发运行任务

看了上面你可能会说，我懂了，只要批量创建任务，它不就是并发了吗？也没错，但有更好的方法，就是使用`asyncio.gather(*aws, return_exceptions=False)`，有两个参数，第一个参数就是一堆协程，第二个是发生异常时是直接中断还是随着结果一起返回，默认是中断不返回，这个函数有一个返回值，那就是所有任务的运行结果，如下示例：

```python
import asyncio


async def factorial(name, number):
    f = 1
    await asyncio.sleep(1)
    f *= number
    print(f"Task {name}: factorial({number}) = {f}")
    return f


async def main():
    results = await asyncio.gather(
        factorial("A", 2),
        factorial("B", 3),
        factorial("C", 4),
    )
    print(results)


if __name__ == '__main__':
    asyncio.run(main())

# 运行结果
Task A: factorial(2) = 2
Task B: factorial(3) = 3
Task C: factorial(4) = 4
[2, 3, 4]
```

### 控制速度

到现在为止，已经完成了使用异步的并发操作，但这种并发是不可控的，比如前文的线程池和进程池，都有一个参数为`max_workers`，是控制速度用的，而在异步编程中也有控制并发速度的，叫做`信号量`，简单写一个示例：
```python
import asyncio
import string


async def factorial(name, number, sem):
    async with sem:
        f = 1
        await asyncio.sleep(1)
        f *= number
        print(f"Task {name}: factorial({number}) = {f}")
        return f


async def main():
    sem = asyncio.Semaphore(5)
    func_list = [factorial(char, index, sem) for index, char in enumerate(string.ascii_uppercase)]
    results = await asyncio.gather(*func_list)
    print(results)


if __name__ == '__main__':
    asyncio.run(main())

# 运行结果
Task A: factorial(0) = 0
Task C: factorial(2) = 2
Task E: factorial(4) = 4
Task B: factorial(1) = 1
Task D: factorial(3) = 3
Task F: factorial(5) = 5
Task H: factorial(7) = 7
Task J: factorial(9) = 9
Task G: factorial(6) = 6
Task I: factorial(8) = 8
Task K: factorial(10) = 10
Task M: factorial(12) = 12
Task O: factorial(14) = 14
Task L: factorial(11) = 11
Task N: factorial(13) = 13
Task P: factorial(15) = 15
Task R: factorial(17) = 17
Task T: factorial(19) = 19
Task Q: factorial(16) = 16
Task S: factorial(18) = 18
Task U: factorial(20) = 20
Task W: factorial(22) = 22
Task Y: factorial(24) = 24
Task V: factorial(21) = 21
Task X: factorial(23) = 23
Task Z: factorial(25) = 25
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25]
```
仔细观看输出的时候你会发现是5个5个一组的输出出来，这就是用到了信号量：`asyncio.Semaphore()`，他只有一个参数就是要限制的并发数量，然后把要限制的代码使用上下文管理器包裹起来就好，但要注意的是，实例化信号量的时候一定要在主事件循环中，也就是使用`async`装饰的函数里面才可以，否则会报错，大家可以尝试一下。

### 参考资料
* [官方文档--协程与任务](https://docs.python.org/zh-cn/3/library/asyncio-task.html)
* [官方文档--信号量](https://docs.python.org/zh-cn/3/library/asyncio-sync.html#asyncio.Semaphore)

## 结束了
其实标题上的“并行编程实践并不严谨”，因为线程池和异步的协程都是属于并发的，只有多进程的才是并行。然后呢，本篇文章也只是带大家入个门，知道咋用。至于为啥没写那么详细呢？我更希望看到的各位呢，也只是把它当作一个入门，然后通过自己去了解更深入的知识。