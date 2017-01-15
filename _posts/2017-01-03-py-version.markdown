---
layout:     post
title:      "蛇形矩阵、蛇形填数、蛇形遍历问题"
subtitle:   ""
date:       2017-01-03
author:     “Aaron”
header-img: "img/post-13.jpg"
tags:
    - 蛇形矩阵
    - 蛇形填数
    - 蛇形遍历
---

## 蛇形矩阵问题

```python
while True:  
    try:
        a = input()
        l = [[0]*a for i in range(a)]
        s = 1
        for i in range(a):
            col = i
            while col >= 0:
                row = i - col
                l[row][col] = s
                col -= 1
                s += 1
        for i in l:
            for j in i:
                if j !=0:
                   print j,
            print
    except:
        break
```

```python
输入 3
1 2 4
3 5
6
```

```python
while True:  
    try:
        row = 0
        col = 0
        s = 1
        a = input()
        bool = True
        l = [[0]*a for i in range(a)]
        for i in range(a):
            row = i
            while row >= 0:
                col = i -row
                if bool:
                    l[row][col] = s
                else:
                    l[col][row] = s
                s += 1
                row -= 1
            bool = not bool
        for i in l:
            for j in i:
                if j != 0:
                    print j,
            print
    except:
        break
```

```python
输入 3
1 2 6
3 5
4
```

```python
while True:
    try:
        s = 1
        a = input()
        l = [[0]*a for i in range(a)]
        bool = True
        for i in range(2*a-1):
            col = i
            while col >= (0 if i < a else i-a+1):
                if col > a - 1:
                    col = a - 1
                row = i - col
                if bool:
                    l[row][col] = s
                else:
                    l[col][row] = s
                s += 1
                col -= 1
            bool = not bool
        for i in l:
            for j in i:
                    print j,
            print
    except:
        break
```

```python
输入 3
1 3 4
2 5 8
6 7 9
```

## 蛇形填数问题

```python
# coding=utf-8
while True:
    try:
        cols, rows=input()
        matrix = [[0 for col in range(cols)] for row in range(rows)]
        i = j = 0
        n = 1
        matrix[i][j] = 1
        while n < rows*cols: # 不取=
             while j+1 < cols and matrix[i][j+1] == 0: # 考虑索引问题取j+1
                 j += 1
                 n += 1
                 matrix[i][j] = n
             while i+1 < rows and matrix[i+1][j] == 0:
                 i += 1
                 n += 1
                 matrix[i][j] = n
             while j > 0 and matrix[i][j-1] == 0: # 不取=
                 j -= 1
                 n += 1
                 matrix[i][j] = n
             while i > 0 and matrix[i-1][j] == 0:
                 i -= 1
                 n += 1
                 matrix[i][j] = n
        for i in matrix:
            for j in i:
                print j,
            print
    except:
        break
```

```python
输入 3 、4
1   2   3
10  11  4
9   12  5
8   7   6
```

## 蛇形遍历

```python
# coding=utf-8
def snake_list(origin_list):
    biggest_len = max(len(i) for i in origin_list) # 取最大长度

    result = []
    for sub_index in range(biggest_len):
        for sub_list in origin_list:
            if sub_index >= len(sub_list):
                continue
            result.append(sub_list[sub_index])
        origin_list.reverse()  # 反转二维矩阵保证蛇形遍历
    return result
```
