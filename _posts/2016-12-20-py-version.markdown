---
layout:     post
title:      "值传递与引用传递 深拷贝与浅拷贝"
subtitle:   ""
date:       2016-11-19
author:     “Aaron”
header-img: "img/post-11.jpg"
tags:
    - 内存地址
    - 引用
---

## 值传递与引用传递

**1.概要**

在python中，若是不可变对象， 比如 *字符串*，*数字*，*元组*，则相当于 *值传递*，只传递值，对原对象没有影响

若是可变对象，比如 *字典*，*列表*，*set集合*，则相当于 *引用传递*，会修改原对象

**2.对象与变量**

在python中，类型属于对象，变量是没有类型的，所有的变量都可以理解是内存中一个对象的“引用”，或者变量可以理解为一个“指针”，指向内存中的对象。

如 a = 1, 变量a将int类型视为内存中的一个对象，变量a是对内存中int类型的引用，变量a指向int类型。

**3.可变与不可变**

```python
   a = 1
   a = 2

   list = [1]
   list[0] = 2
```

内存中原始的1对象因为不能改变，于是被“抛弃”，变量a指向一个新的int对象，其值为2（相当于值传递）

由于列表是可变的，List原来指向包含一个对象的数组，这个数组的对象内容发生改变，但是对于List而言，它依旧“指向”内存中的原数组，但是数组中第一个元素的值已经由 1 变为 2 了。

## Python深拷贝和浅拷贝

**1.对象赋值**

```python
will = ["Will", 28, ["Python", "C#", "JavaScript"]]
wilber = will
print id(will)
print will
print [id(i) for i in will]
print "--------------------"
print id(wilber)
print wilber
print [id(i) for i in wilber]
print "--------------------"
will[0] = "Wilber"
will[2].append("CSS")
print id(will)
print will
print [id(i) for i in will]
print "--------------------"
print id(wilber)
print wilber
print [id(i) for i in wilber]
```

输出结果为：

```python
4361951784
['Will', 28, ['Python', 'C#', 'JavaScript']]
[4361990192, 140285599304000, 4361906296]
--------------------
4361951784
['Will', 28, ['Python', 'C#', 'JavaScript']]
[4361990192, 140285599304000, 4361906296]
--------------------
4361951784
['Wilber', 28, ['Python', 'C#', 'JavaScript', 'CSS']]
[4361994128, 140285599304000, 4361906296]
--------------------
4361951784
['Wilber', 28, ['Python', 'C#', 'JavaScript', 'CSS']]
[4361994128, 140285599304000, 4361906296]
```
* 创建了一个名为will的变量，这个变量指向一个list对象，从第一部分中可以看到所有对象的地址
* 通过will变量对wilber变量进行赋值，那么wilber变量将指向will变量对应的对象（内存地址），也就是说"wilber is will"，"wilber[i] is    will[i]"
* 可以理解为，Python中，对象的赋值都是进行对象引用（内存地址）传递
* 第三部分中，由于will和wilber指向同一个对象，所以对will的任何修改都会体现在wilber上
* 这里需要注意的一点是，str是不可变类型，所以当修改的时候会替换旧的对象，产生一个新的地址4361994128

**2.浅拷贝**

```python
import copy

first = ["first", 28, ["Python", "C#", "JavaScript"]]
second = copy.copy(first)

print id(first)
print first
print [id(i) for i in first]
print "---------------------"
print id(second)
print second
print [id(i) for i in second]
print "---------------------"
first[0] = "second"
first[2].append("CSS")
print id(first)
print first
print [id(i) for i in first]
print "---------------------"
print id(second)
print second
print [id(i) for i in second]
```

输出结果：

```python
4564704936
['first', 28, ['Python', 'C#', 'JavaScript']]
[4564684160, 140469024607296, 4564705080]
---------------------
4564745320
['first', 28, ['Python', 'C#', 'JavaScript']]
[4564684160, 140469024607296, 4564705080]
---------------------
4564704936
['second', 28, ['Python', 'C#', 'JavaScript', 'CSS']]
[4564684736, 140469024607296, 4564705080]
---------------------
4564745320
['first', 28, ['Python', 'C#', 'JavaScript', 'CSS']]
[4564684160, 140469024607296, 4564705080]
```

* 使用一个first变量，指向一个list类型的对象
* 然后，通过copy模块里面的浅拷贝函数copy()，对first指向的对象进行浅拷贝，然后浅拷贝生成的新对象赋值给second变量
* 浅拷贝会创建一个新的对象，这个例子中"second is not first"
* 但是，对于对象中的元素，浅拷贝就只会使用原始元素的引用（内存地址），也就是说"first[i] is second[i]"
* 当对first进行修改的时候
* 由于list的第一个元素str是不可变类型，所以first对应的list的第一个元素会使用一个新的对象4564684736
* 但是list的第三个元素是一个可变类型，修改操作不会产生新的对象，所以first的修改结果会相应的反应到second上

**3.深拷贝**

```python
import copy

first = ["first", 28, ["Python", "C#", "JavaScript"]]
second = copy.deepcopy(first)

print id(first)
print first
print [id(i) for i in first]
print "---------------------"
print id(second)
print second
print [id(i) for i in second]
print "---------------------"
first[0] = "second"
first[2].append("CSS")
print id(first)
print first
print [id(i) for i in first]
print "---------------------"
print id(second)
print second
print [id(i) for i in second]
```

输出结果：

```python
4420382376
['first', 28, ['Python', 'C#', 'JavaScript']]
[4420361600, 140235628381168, 4420382520]
---------------------
4420422760
['first', 28, ['Python', 'C#', 'JavaScript']]
[4420361600, 140235628381168, 4420422832]
---------------------
4420382376
['second', 28, ['Python', 'C#', 'JavaScript', 'CSS']]
[4420362176, 140235628381168, 4420382520]
---------------------
4420422760
['first', 28, ['Python', 'C#', 'JavaScript']]
[4420361600, 140235628381168, 4420422832]
```

* 同样使用一个first变量，指向一个list类型的对象
* 通过copy模块里面的深拷贝函数deepcopy()，对first指向的对象进行深拷贝，然后深拷贝生成的新对象赋值给second变量
* 深拷贝也会创建一个新的对象，这个例子中"first is not second"
* 但是，对于对象中的元素，深拷贝都会重新生成一份（有特殊情况，下面会说明），而不是简单的使用原始元素的引用（内存地址）
* 例子中first的第三个元素指向4420382520，而second的第三个元素是一个全新的对象4420422832，也就是说，"wilber[2] is not will[2]"
* 当对first进行修改的时候
* 由于list的第一个元素是不可变类型，所以first对应的list的第一个元素会使用一个新的对象4420362176
* 但是list的第三个元素是一个可变类型，修改操作不会产生新的对象，但是由于"first[2] is not second[2]"，所以first的修改不会影响second

**4.拷贝的特殊情况**
* 对于非容器类型（如数字、字符串、和其他'原子'类型的对象）没有拷贝这一说
* 如果 *元祖变量* 只包含原子类型对象，则不能深拷贝

```python
import copy

first = ("Python", "C#", "JavaScript")
second = copy.deepcopy(first)
print first is second

first = ("Python", "C#", "JavaScript", [])
second = copy.deepcopy(second)
print first is second

True
False
```

## 总结
* Python中对象的赋值都是进行对象引用（内存地址）传递
* 使用copy.copy()，可以进行对象的浅拷贝，它复制了对象，但对于对象中的元素，依然使用原始的引用
* 如果需要复制一个容器对象，以及它里面的所有元素（包含元素的子元素），可以使用copy.deepcopy()进行深拷贝
* 对于非容器类型（如数字、字符串、和其他'原子'类型的对象）没有被拷贝一说
* 如果元祖变量只包含原子类型对象，则不能深拷贝
