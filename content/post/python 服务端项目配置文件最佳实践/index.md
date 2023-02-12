---
slug: ba974023
author: "昊色居士"
title: "Python 服务端项目配置文件最佳实践"
description: 
date: 2022-10-13T10:52:35+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Python", "后端"
]
categories: [
    "Python"
]
series:  [

]
---

# python 服务端项目配置文件最佳实践

## 前言

我们在编写服务端代码或搭建服务器框架的时候，经常会考虑配置文件该怎么写？写在哪里？什么格式的？json？ini？yaml？环境变量？写完之后该怎么取呢？今天这篇就以`sanic`为例给大家带来一个我认为的最佳实践

## 常见方案

翻开`sanic`的[官方文档](https://sanic.dev/zh/guide/deployment/configuration.html)，配置文件是那一章节，可以看到官方有很多配置文件的加载方案，包括：

* dict格式数据：{"a": 1}

* python 文件格式数据: a = 1

* 通过类的方式：

  ```python
  class MyConfig:
      A = 1
      B = 2
  ```

然后再通过：`app.update_config(...)`的方法加载不同的配置文件，在程序中使用的时候也是通过`app.config.xx`的方式来获取配置

对应不同的服务端框架也有各自不同的配置文件载入方式，但都存在常见的几个问题：

* 灵活。配置文件可以复杂，也可以简单，可以有一层，也可以多层
* 多种格式。可以选择使用ini格式、.env格式、json格式、yaml格式等多种格式
* 代码提示。可以在项目的任何地方方便的获取配置，无需在去配置中去查看某个配置项到底是咋拼的来着
* 环境变量。可以选择加载环境变量的值，且支持类型的转换

## pydantic配置

### 简单介绍

pydantic的顶顶大名大家应该都了解了，可以用来做参数校验、类型的Schema生成等，但是还要一个同样强大的功能就是做项目的配置管理

> 官方文档：[Settings management](https://pydantic-docs.helpmanual.io/usage/settings/)

它具有丰富的类型，如：

* 使用`BaseModel`进行声明嵌套的结构
* 使用`RedisDsn`载入redis的连接信息
* 使用`PostgresDsn`载入pgsql的连接信息
* 使用` PyObject`载入python的函数等
* 更多类型

支持以全大写的方式从环境变量中自动获取参数，也就是说，声明变量的时候可以是小写的，但是获取环境变量的时候确是大写的，也支持环境变量的嵌套结构，下面通过实例去演示一下：

```python
from pydantic import (
    BaseModel,
    BaseSettings,
    PyObject,
    RedisDsn,
    PostgresDsn,
    AmqpDsn,
    Field,
)


class SubModel(BaseModel):
    foo = 'bar'
    apple = 1


class Settings(BaseSettings):
    auth_key: str
    api_key: str = Field(..., env='my_api_key')
	
    # 各种连接信息
    redis_dsn: RedisDsn = 'redis://user:pass@localhost:6379/1'
    pg_dsn: PostgresDsn = 'postgres://user:pass@localhost:5432/foobar'
    amqp_dsn: AmqpDsn = 'amqp://user:pass@localhost:5672/'
	
    # 传递一个python对象
    special_function: PyObject = 'math.cos'

	# 数组结构的数据：["foo.com", "bar.com"]
    domains: set[str] = set()

    # 嵌套结构的数据：{"foo": "x", "apple": 1}
    more_settings: SubModel = SubModel()


print(Settings().dict())
"""
{
    'auth_key': 'xxx',
    'api_key': 'xxx',
    'redis_dsn': RedisDsn('redis://user:pass@localhost:6379/1', ),
    'pg_dsn': PostgresDsn('postgres://user:pass@localhost:5432/foobar', ),
    'amqp_dsn': AmqpDsn('amqp://user:pass@localhost:5672/', scheme='amqp',
user='user', password='pass', host='localhost', host_type='int_domain',
port='5672', path='/'),
    'special_function': <built-in function cos>,
    'domains': set(),
    'more_settings': {'foo': 'bar', 'apple': 1},
}
"""
```

### 多种配置读取方式

使用pydantic管理配置可以很灵活的使用不同的格式来管理配置，可以通过自定义来源的方式来使用，如：

```python
from pydantic import BaseSettings
from pydantic.env_settings import SettingsSourceCallable

class SettingsBase(BaseSettings):
    """
    项目设置的基类
    """

    class Config:
        env_file = getpath_by_root("../.env")
        env_file_encoding = "utf-8"
        env_nested_delimiter = "__"

        @classmethod
        def customise_sources(
            cls,
            init_settings: SettingsSourceCallable,
            env_settings: SettingsSourceCallable,
            file_secret_settings: SettingsSourceCallable,
        ) -> tuple[SettingsSourceCallable, ...]:
            ...
```

在你的设置类里面定义配置类`Config`，可以进行灵活的配置，其中类方法`customise_sources`就是加载不同来源的地方

#### 环境变量

定义了三个变量：

* env_file：定义了`Dotenv`格式的环境变量文件的存储位置
* env_file_encoding：定义环境变量文件的编码格式
* env_nested_delimiter：定义了嵌套的结构用环境变量表示的时候怎么进行分割

这样既可以通过直接设置系统环境变量的方式来加载配置，也可以通过`.env`环境变量文件的方式来加载配置

#### json 配置文件

我们可以使用常用的json格式来加载我们的配置，使用方法如下：

```python
class CustomSettingsSource(object):
    """
    自定义的配置文件来源基类
    """

    def __init__(self, path: Path):
        self.path = path

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}(path={self.path!r})"
    
class JsonSettingsSource(CustomSettingsSource):
    """
    json文件来源导入配置项
    """

    def __call__(self, settings: BaseSettings) -> dict[str, Any]:
        encoding = settings.__config__.env_file_encoding
        return json.loads(self.path.read_text(encoding))
    
	......
    
    # 默认的设置
    default_settings = {
        init_settings,
        env_settings,
        file_secret_settings,
    }

    # json 配置文件
    json_file = getpath_by_root("../settings.json")
    if json_file.exists():
        json_settings_source = JsonSettingsSource(json_file)
        default_settings.add(json_settings_source)
    
    ......
    
```

在`customise_sources`中我们先判断了json文件的位置，如果存在则使用`JsonSettingsSource`类把json文件内容加载进去

#### ini、yaml配置文件

这两种就和上面的json类似了，通过各自的类去把文件加载成字典的格式，然后如果路径存在则加载对应的文件。

## 示例代码

这里代码是一个配置的基类代码：

```python
import json
from configparser import ConfigParser
from pathlib import Path
from typing import Any

import yaml
from pydantic import BaseSettings
from pydantic.env_settings import SettingsSourceCallable

from server.util.path import getpath_by_root


class CustomSettingsSource(object):
    """
    自定义的配置文件来源基类
    """

    def __init__(self, path: Path):
        self.path = path

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}(path={self.path!r})"


class JsonSettingsSource(CustomSettingsSource):
    """
    json文件来源导入配置项
    """

    def __call__(self, settings: BaseSettings) -> dict[str, Any]:
        encoding = settings.__config__.env_file_encoding
        return json.loads(self.path.read_text(encoding))


class IniSettingsSource(CustomSettingsSource):
    """
    ini文件来源导入配置项
    """

    def __call__(self, settings: BaseSettings) -> dict[str, Any]:
        encoding = settings.__config__.env_file_encoding
        parser = ConfigParser()
        parser.read(self.path, encoding)
        return getattr(parser, "_sections", {}).get("settings")


class YamlSettingsSource(CustomSettingsSource):
    """
    ini文件来源导入配置项
    """

    def __call__(self, settings: BaseSettings) -> dict[str, Any]:
        encoding = settings.__config__.env_file_encoding
        return yaml.safe_load(self.path.read_text(encoding))


class SettingsBase(BaseSettings):
    """
    项目设置的基类
    """

    class Config:
        env_file = getpath_by_root("../.env")
        env_file_encoding = "utf-8"
        env_nested_delimiter = "__"

        @classmethod
        def customise_sources(
            cls,
            init_settings: SettingsSourceCallable,
            env_settings: SettingsSourceCallable,
            file_secret_settings: SettingsSourceCallable,
        ) -> tuple[SettingsSourceCallable, ...]:
            """
            自定义配置来源
            Args:
                init_settings: 初始化设置
                env_settings:环境变量设置
                file_secret_settings:加密文件设置

            Returns:

            """
            # 默认的设置
            default_settings = {
                init_settings,
                env_settings,
                file_secret_settings,
            }

            # json 配置文件
            json_file = getpath_by_root("../settings.json")
            if json_file.exists():
                json_settings_source = JsonSettingsSource(json_file)
                default_settings.add(json_settings_source)

            # ini配置文件
            ini_file = getpath_by_root("../settings.ini")
            if ini_file.exists():
                ini_settings_source = IniSettingsSource(ini_file)
                default_settings.add(ini_settings_source)

            # yaml配置文件
            yaml_file = getpath_by_root("../settings.yaml")
            if yaml_file.exists():
                yaml_settings_source = YamlSettingsSource(yaml_file)
                default_settings.add(yaml_settings_source)

            return tuple(default_settings)
```

里面分别配置了默认值、环境变量、加密文件、json文件、ini配置文件、yaml配置文件多种来源方式，并设置优先级

之后可以再其他文件中继承这个类来使用：

```python
from pydantic import Field, RedisDsn

from ..entity.enum import RunModeEnum
from .base import SettingsBase
from .db import PGDBConfig


class Settings(SettingsBase):
    # 监听的地址和端口
    host: str = Field(default="0.0.0.0")
    port: int = Field(default=8080)

    # 模式 dev 或 production
    mode: RunModeEnum = Field(default=RunModeEnum.DEV)

    # 是否是调试模式 生产模式强制为False
    debug: bool = Field(default=True)

    # 是否开启自动重载 生产模式强制为False
    auto_reload = Field(default=True)
    
    # 数据库配置
    db: dict[str, PGDBConfig] = Field()
```

db.py：

```python
from pydantic import BaseModel, Field


class PGDBConfig(BaseModel):
    # 数据库名字
    database: str = Field()

    # 数据库地址
    host: str = Field(default="127.0.0.1")

    # 数据库端口
    port: int = Field(default=5432)

    # 数据库用户名
    user: str = Field(default="postgres")

    # 数据库密码
    password = Field(default="")

    # 数据库连接池大小
    maxsize = Field(default=10, cast=int)

    # 配置
    server_settings = Field(default={"jit": "off"})
```

这样可以更灵活的加载配置

## 结语

这篇文章主要以sanic为基础介绍了再项目中加载配置文件的优雅的一种方式，仓促之下没有说的太详细，后续会再修改修改，尽量配合实例更通俗易懂一点，希望大家能够有所收获！！！

