---
slug: 311e4ef3
author: "昊色居士"
title: "Python元类浅谈之单例模式"
description: 
date: 2022-10-16T23:37:04+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Python", "元类"
]
categories: [
    "Python"
]
series:  [

]
---

# Python元类浅谈之单例模式

## 前言

今天来带大家以单例模式为案例，给大家简单的讲解一下Python元类相关的知识。

## 啥是元类

### 类的创建

说元类之前先说一下普通类的创建过程，众所周知，类的创建的通过`class`关键字来进行定义一个类的，然后可以选择继承自其他的类，比如：

```python
class A(object):
    pass

```

但也有另外一种创建类的方式，就是使用`type()`函数，`type()`函数有两种用法，我们来看看官方的函数签名：

```python
    def __init__(cls, what, bases=None, dict=None): # known special case of type.__init__
        """
        type(object_or_name, bases, dict)
        type(object) -> the object's type
        type(name, bases, dict) -> a new type
        # (copied from class doc)
        """
        pass
```

上面明确的指出了如果只传递一个参数，就是走的` type(object)`方法，作用是反正对象的类型，如果传递了三个函数则是使用`type(name, bases, dict)`，作用就是创建一个新的类型，也就是类的创建，我们来看看三个参数：

* name：新类型的名字
* bases：新类型要继承的类，类型的元组
* dict：新类型的属性

我们来看一个案例：

```python
def main():
    A = type('A', (object,), {})
    a = A()
    b = A()
    print(a)
    print(b)


if __name__ == '__main__':
    main()

# 输出
<__main__.A object at 0x0000019C84A2EFA0>
<__main__.A object at 0x0000019C84A2EF70>
```

我们使用type创建了一个类，类型是A，继承自object类，里面没有属性，然后使用这个A创建了a、b两个对象

### 元类

由上面可以看到，我们的类A是通过type创建出来的，所以这个`type`就是类`A`的元类，也是所有类的默认元类，如果我们要想使用自定义的元类，就使用自己的类替换type就好，先看一下单例模式的例子：

```python

class SingletonMeta(type):
    _instances: Dict = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]
```

类`SingletonMeta`继承自`type`，就表示可以作为一个元类来使用，然后`__call__`方法这里是定义在了元类里面，意思就是每次执行`A()`的时候会调用的方法，看第一个参数`cls`，就是使用这个元类的类，也就是`A`，然后这个类如果不存在元类的类属性`_instances`中，就保存进去，然后返回这个类的实例

### `__call__`和`__init__`区别

看到上面大家可能会有疑问`A()`不是创建一个对象吗？这个时候不是应该执行`__init__`方法吗？那哪个会先执行呢？一般情况下大家看到的`__call__`是定义在普通的类里面的，也就是先实例化一个对象，然后调用对象的方式如：

```python
a = A()
a()
```

这样的情况下一定是先执行的`__init__`方法，然后再每次执行`a()`的时候再去调用`__call__`方法，但是这里的`__call__`方法是定义再元类上面的，上面说了元类的控制类的创建，所以`__call__`方法就是再类创建成功之后，每次执行`A()`的时候调用，这个时候类执行`A() `的时候`__init__`方法，只会在第一次的`__call__`方法执行结束之后调用一次，其他的时候都会只执行`__call__`方法

### `__new__`和`__init__`区别

这里面还有一个很常见的问题：`__init__和__new__`的区别是啥，`__init__`是类创建对象的时候去调用的方法，__new__`则是创建类的时候调用的，就拿手动创建类来说：

```python
def main():
    A = SingletonMeta('A', (object,), {})
    a = A()
    b = A()
    print(a)
    print(b)


if __name__ == '__main__':
    main()
```

`__new__`方法是第二行调用的，而`__init__`方法则是第三行调用的。

### 运行结果

```python
from typing import Dict


class SingletonMeta(type):
    _instances: Dict = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


def main():
    A = SingletonMeta('A', (object,), {})
    a = A()
    b = A()
    print(a)
    print(b)


if __name__ == '__main__':
    main()

 # 结果
<__main__.A object at 0x00000181D1FA8340>
<__main__.A object at 0x00000181D1FA8340>
```

```python
from typing import Dict


class SingletonMeta(type):
    _instances: Dict = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class A(object, metaclass=SingletonMeta):
    pass
    
def main():
    a = A()
    b = A()
    print(a)
    print(b)


if __name__ == '__main__':
    main()

 # 结果
<__main__.A object at 0x00000181D1FA8340>
<__main__.A object at 0x00000181D1FA8340>
```

上面的两种用法是相同的

## 结语

其实元类并没有那么恐怖，理解意思和过程就很清楚了，先浅谈到这里，希望大家有所收获！！！
