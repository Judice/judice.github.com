---
layout:     post
title:      "二叉树问题"
subtitle:   "遍历、反转"
date:       2017-01-15
author:     “Aaron”
header-img: "img/post-01.jpg"
tags:
    - 二叉树
    - 遍历
    - 反转
---

## 翻转二叉树

```python
# coding:utf-8
class TreeNode:
    def __init__(self, x):
       self.val = x
       self.left = None
       self.right = None

    def __str__(self):
        return str(self.val)

class Solution:
    def Mirror(self, root):
        if not root:
           return
        root.right, root.left = root.left, root.right
        self.Mirror(root.right)
        self.Mirror(root.left)
```

## 前序遍历二叉树

```python
# coding:utf-8
class Solution():  # 递归方法
   def preorder(self,root):
      if not root:
        return
      print root.val
      self.preorder(root.left)
      self.preorder(root.right)
```

```python
# coding:utf-8
class Solution(): # 非递归方法
   def preorder(self,root):
     if not root:
       return
     stack = []
     stack.append(root)
     while stack:
       node = stack[-1]
       stack.pop()
       print node.val
       if node.right:
         stack.append(node.right)
       if node.left:
         stack.append(node.left)
```
