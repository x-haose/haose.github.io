---
slug: fc681a01
author: "昊色居士"
title: "让WebSocket变得优雅之添加路由"
description: 
date: 2022-10-14T23:35:37+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Python", "WebSocket"
]
categories: [
    "Python", "WebSocket"
]
series:  [

]
---

# 让WebSocket变得优雅之添加路由

> 优雅永不过时

## 前言

在Python里面会经常用到websockets库，但对于一个后端程序员来说，用的时候会有点懵，路由呢？我该写到哪？咋只有连接的地方？咋区分不同的业务呢？实际上websockets库本身的没有路由的这一说法的，只有路径的这一说法，我们就可以根据这个路径，来为它手动的实现一个路由机制。

## 实现

依赖于第三方库：[routes](https://github.com/bbangert/routes)，用于生成url路由映射用

### 路由路径（RoutedPath）

重新封装了路由的路径相关，有处理器和路由参数两个属性，具体代码如下：

```python
class RoutedPath(str):
    """
    路由路径类
    """
    handler: typing.Any
    params: typing.Mapping[str, str]

    @classmethod
    def create(cls, raw_path, handler, params) -> "RoutedPath":
        path = cls(raw_path)
        path.handler = handler
        path.params = params
        return path
```

### 路由类 （WSRouter）

实现一个简单的类似蓝图的路由类。由一个路由前缀、路由列表和一个装饰器方法组成。具体代码如下：

```python
class WSRouter(object):
    routes: typing.List = []

    def __init__(self, path_prefix: str):
        """
        路由类，相当于简单的蓝图

        Args:
            path_prefix: 路径前缀
        """
        self.path_prefix = path_prefix

    def route(self, path: str, *, name: typing.Optional[str] = None):
        """
        装饰器方法
        Args:
            path:
            name:

        Returns:

        """

        def decorator(endpoint: typing.Callable[[], typing.Any]):
            self.routes.append(Route(name, path, __handle=endpoint))
            return endpoint

        return decorator

```

装饰器的作用是把路径、名字、处理器封装成`Route`后保存在列表中

### 异常类（NotFoundRoute）

定义了自定义异常，找不到路由时会触发：

```python
class NotFoundRoute(Exception):
    """
    找不到路由的异常
    """

    def __init__(self, path):
        self.path = path
```

### 处理器（Handler）

上面我们实现了一个类似蓝图的路由类，这里我们添加一个方法，把若干个蓝图添加进去：

```python
class Handler(object):
    """
    处理器
    """

    def __init__(self):
        self._mapper = Mapper()

    def add_route(self, route: WSRouter = None, **kwargs):
        """
        把子路由添加到看路由表中去
        Args:
            route:

        Returns:

        """
        if route:
            self._mapper.extend(route.routes, route.path_prefix)
        else:
            name = kwargs.get('name')
            path = kwargs.get('path')
            endpoint = kwargs.get('endpoint')
            self._mapper.connect(name, path, __handle=endpoint)
```

先创建了一个成员变量`_mapper`，用来保存所有的路由表，然后通过`add_route`方法来添加路由，这里的route属性就是蓝图，但后面还有一个`**kwargs`参数，这里是用来添加但是路由的，来实现既可以注册蓝图也可以注册普通的url

我们已经把路由表保存下来了，然后下一步就要根据路径来匹配对应的`RoutedPath`，具体代码如下：

```python
    def __init__(self, default_handler: typing.Callable[[], typing.Any] = None):
        self._mapper = Mapper()
        self._default_handler = default_handler
        
    def match(self, path: str) -> typing.Optional[RoutedPath]:
            """
            匹配路由，匹配不到调用默认处理函数，没有默认处理函数则引发NotFoundRoute异常
            Args:
                path: 路径

            Returns:
                返回RoutedPath的实例
            """
            params = self._mapper.match(path)
            if not params:
                if self._default_handler:
                    return RoutedPath.create(path, self._default_handler, {})
                raise NotFoundRoute(path=path)

            handler = params.pop("__handle")
            return RoutedPath.create(path, handler, params)
```

我们添加了一个新的初始化可选属性：`default_handler`，用作默认的处理器函数，也就是无法匹配到路由的时候用

在`match`方法中只有一个参数就是路径，然后使用路由表的匹配方法：`Mapper.match(...)`来获取对应的路由参数，在查找不到的时候，如果没有默认的处理器则会触发`NotFoundRoute`异常，我们在路由类（WSRouter）和添加路由（add_route）方法中，把处理器保存在`__handle`中，这里就要把处理器取出来，然后创建RoutedPath类

调用处理器：

由于websockets的监听方法，传递的是一个方法，我们的处理器对象是一个对象，是无法直接作为参数传递进去的，但我们可以实现`__call__`方法，让对象可以向方法一样可以调用，代码如下：

```python
    async def __call__(self, ws, path):
        """
        可直接调用这个对象
        Args:
            ws:
            path:

        Returns:

        """
        try:
            if not isinstance(path, RoutedPath):
                path = self.match(path)
            if path.params is None:
                await ws.close(4040)
            await path.handler(ws, path)
        except NotFoundRoute as e:
            logger.error(f"找不到路由：{e.path}")
        except Exception as e:
            logger.error(f"发生异常: [{e.__class__.__name__}] {e}")

```

这里就是由`websockets`调用的默认处理函数，来调用匹配路由和处理器的地方，并且做了异常的处理

### 完整代码

```python
import typing

from loguru import logger
from routes import Mapper
from routes.route import Route


class RoutedPath(str):
    """
    路由路径类
    """
    handler: typing.Any
    params: typing.Mapping[str, str]

    @classmethod
    def create(cls, raw_path, handler, params) -> "RoutedPath":
        path = cls(raw_path)
        path.handler = handler
        path.params = params
        return path


class WSRouter(object):
    routes: typing.List = []

    def __init__(self, path_prefix: str):
        """
        路由类，相当于简单的蓝图

        Args:
            path_prefix: 路径前缀
        """
        self.path_prefix = path_prefix

    def route(self, path: str, *, name: typing.Optional[str] = None):
        """
        装饰器方法
        Args:
            path:
            name:

        Returns:

        """

        def decorator(endpoint: typing.Callable[[], typing.Any]):
            self.routes.append(Route(name, path, __handle=endpoint))
            return endpoint

        return decorator


class NotFoundRoute(Exception):
    """
    找不到路由的异常
    """

    def __init__(self, path):
        self.path = path


class Handler(object):
    """
    处理器
    """

    def __init__(self, default_handler: typing.Callable[[], typing.Any] = None):
        self._mapper = Mapper()
        self._default_handler = default_handler

    async def __call__(self, ws, path):
        """
        可直接调用这个对象
        Args:
            ws:
            path:

        Returns:

        """
        try:
            if not isinstance(path, RoutedPath):
                path = self.match(path)
            if path.params is None:
                await ws.close(4040)
            await path.handler(ws, path)
        except NotFoundRoute as e:
            logger.error(f"找不到路由：{e.path}")
        except Exception as e:
            logger.error(f"发生异常: [{e.__class__.__name__}] {e}")

    def add_route(self, route: WSRouter = None, **kwargs):
        """
        把子路由添加到看路由表中去
        Args:
            route:

        Returns:

        """
        if route:
            self._mapper.extend(route.routes, route.path_prefix)
        else:
            name = kwargs.get('name')
            path = kwargs.get('path')
            endpoint = kwargs.get('endpoint')
            self._mapper.connect(name, path, __handle=endpoint)

    def match(self, path: str) -> typing.Optional[RoutedPath]:
        """
        匹配路由，匹配不到调用默认处理函数，没有默认处理函数则引发NotFoundRoute异常
        Args:
            path: 路径

        Returns:
            返回RoutedPath的实例
        """
        params = self._mapper.match(path)
        if not params:
            if self._default_handler:
                return RoutedPath.create(path, self._default_handler, {})
            raise NotFoundRoute(path=path)

        handler = params.pop("__handle")
        return RoutedPath.create(path, handler, params)

```



## 使用

server.py

```python
import asyncio

import websockets

from wss.routers import Handler
from wss.test_ws import test_router


async def root_handler(websocket, _):
    """
    根路径的处理函数，直接回传发来的信息
    Args:
        websocket:
        _

    Returns:

    """
    async for message in websocket:
        await websocket.send(message)


async def main():
    handler = Handler()

    handler.add_route(path='/', endpoint=root_handler)
    handler.add_route(test_router)

    async with websockets.serve(handler, "", 5989):
        await asyncio.Future()


if __name__ == '__main__':
    asyncio.run(main())

```

test_ws.py

```python
from websockets import WebSocketServerProtocol

from wss.routers import WSRouter, RoutedPath

test_router = WSRouter('/test')


@test_router.route('/{uid}')
async def test(websocket: WebSocketServerProtocol, path: RoutedPath):
    uid = path.params.get('uid')
    await websocket.send(f"新连接: {uid}")
    async for message in websocket:
        await websocket.send(message)

```

如上所示，我们在`server.py`的main函数里面注册了处理器，并添加了一个根路由及一个`test_router`蓝图，分别处理各自的业务，这样一来我们就可以根据不能的业务来声明不同的蓝图来处理

## 结语

一个简单的websocket路由编写代码就结束了，这个代码也不完全是我本人写的，是看到一个github的库实现了功能，然后它的功能只有装饰器的形式，只能添加跟路由，无法添加路由组及蓝图，这个方式的弊端就是只能写到一个文件中，如果业务和服务分开多个文件，就很容易产生循环引用，所以我就对这个库的代码就是一些修改，并且分享了出来。至于这次为啥没有写出源库到底是哪个，纯粹是因为是我忘了！！！
