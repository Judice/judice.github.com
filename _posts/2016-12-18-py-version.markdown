---
layout:     post
title:      "pip 使用镜像源 virtualenv"
subtitle:   "镜像源"
date:       2016-12-18
author:     “Aaron”
header-img: "img/post-09.jpg"
tags:
    - python
    - pip
    - virtualenv
---

## 镜像源

网络访问某些国外网站，总是时不时的连接不上, pypi.python.org就是其中一个，所以，使用pip给Python安装软件时，经常出现错误，修改pip连接的软件库可以解决这个问题。

http://pypi.douban.com 是豆瓣提供一个镜像源，连接速度很好。

先在Mac系统用户名下创建pip文件夹，在终端中使用 **mikdir ~/.pip**

在pip文件夹中创建pip.conf文件， 使用 **touch pip.conf**

编辑 **pip.conf** 文件

```python
[global]
index-url = http://pypi.douban.com/simple
[install]
trusted-host = pypi.douban.com
```

## virtualenv

virtualenv 是一个创建隔绝的Python环境的工具。virtualenv创建一个包含所有必要的可执行文件的文件夹，用来使用Python工程所需的包。

* 基本使用

为一个工程创建一个虚拟环境：

```python
      ~ virtualenv venv
```

选择使用一个Python解释器：

```python
      ~ virtualenv -p /usr/bin/python2.7 venv     #使用python2.7解释器
      ~ virtualenv -p /usr/local/bin/python3.5 venv    #使用python3.5解释器
```

要开始使用虚拟环境，其需要被激活：

```python
      ~ source venv/bin/activate
```

从现在起，任何你使用pip安装的包将会放在 venv 文件夹中，与全局安装的Python隔绝开,例如：

```python
       ~ pip install -r requirements.txt
```

如果你在虚拟环境中暂时完成了工作，则可以停用它

```python
       ~ deactivate
```
