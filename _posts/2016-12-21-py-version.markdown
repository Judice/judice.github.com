---
layout:     post
title:      "Flask的配置处理"
subtitle:   "Flask config"
date:       2016-12-21
author:     “Aaron”
header-img: "img/post-12.jpg"
tags:
    - Flask
    - config
---

## 概述

应用会需要某种配置，需要根据应用环境更改不同的设置，比如切换调试模式、设置密钥、或是别的设定环境的东西，Flask 对象的 config 属性，是 Flask 自己放置特定配置值的地方，也是扩展可以存储配置值的地方，也可以把自己的配置保存到这个对象里。

## 配置基础

config继承于字典，并且可以像修改字典一样修改config

```python
    app = Flask(__name__)
    app.config['DEBUG'] = True
```

## 从文件配置

一般而言会在独立的文件里存储配置，理想情况是存储在当前应用包之外

一般而言有两种常见方法加载config参数：

可以从 config file 中加载：

```python
   app.config.from_pyfile('yourconfig.cfg')
```

或者定义一个config module,调用方法from_object()：

```python
   app = Flask(__name__)
   app.config.from_object('yourapplication.default_settings')
   app.config.from_envvar('YOURAPPLICATION_SETTINGS')
```

首先从 yourapplication.default_settings 模块加载配置，然后用 YOURAPPLICATION_SETTINGS 环境变量指向的文件的内容覆盖其值

## from_object（self, obj）方法：

```python
   def from_object(self, obj):
     if isinstance(obj, string_types):
            obj = import_string(obj)
     for key in dir(obj):
            if key.isupper():
                self[key] = getattr(obj, key)
```
* 先判断obj是否是字符串，可以使用werkzeug.import_string找到相应的config.py

* 或者遍历obj字典，运用 isupper()方法，只有大写名称的值才会被存储到配置对象中

* odj 可以是 python模块 例如：

```python
   app.config.from_object('config') # 直接使用 config.py
```

* obj 可以是字典，字典元素是python的类 例如：

```python
   from config import config  # 从config.py传入config字典
   app.config.from_object(config[name]) # config[name]是一个类
```

# 配置中使用类和继承

```python
class Config(object):
    DEBUG = False
    TESTING = False
    DATABASE_URI = 'sqlite://:memory:'

class ProductionConfig(Config):
    DATABASE_URI = 'mysql://user@localhost/foo'

class DevelopmentConfig(Config):
    DEBUG = True

class TestingConfig(Config):
    TESTING = True
```

启用这样的配置你需要调用 from_object()

```python
  app.config.from_object('configmodule.ProductionConfig')
```
