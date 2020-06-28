---
title: Python 对 list 的处理
date: 2017-06-25T08:00:00+08:00
draft: false
toc:
comments: true
---


list 是一种有序的数据集合，索引从 0 开始，类似 C 语言中的数组。比如定义一个 list ：

    >>> array=[2,3,4]
    >>> print array
    [2, 3, 4]
    >>> print array[0]
    2

## 1. 基本操作    

获取 list 元素的个数可以用 `len()` 函数：

    >>> len(array)
    3
    
所以，最后一个元素的索引是 `len()-1` ，还可以用 -1 做索引，直接获取最后一个元素，以此类推，可以获取倒数第二个，倒数第三个元素：

    >>> array[-1]
    4
    >>> array[-2]
    3

追加一个元素:

    >>> array.append(5)
    >>> array
    [2, 3, 4, 5]
    
在指定位置插入一个元素：

    >>> array.insert(1,9)
    >>> array
    [2, 9, 3, 4, 5]

删除某个元素可以用 `pop()` 函数，参数是元素的索引，不带参数的话，默认删除末尾的元素，返回值是删除的元素的内容：

    >>> array.pop()
    5
    >>> array
    [2, 9, 3, 4]
    >>> array.pop(1)
    9
    >>> array
    [2, 3, 4]
    
获取某个元素的下表,如果有重复的元素，只返回第一个元素的下标：

    >>> a=[1,2,3,4,2]
    >>> a.index(1)
    0
    >>> a.index(2)
    1
    
## 2. 遍历

Pytion 提供了一个内置函数 `enumerate()` ，可用于遍历数据：

    enumerate(iterable[, start])
    
第一个参数是一个 list ，第二个可选参数允许我们自定义一个计数器，例如 list ：

    >>> array=['a','b','c','d']
    >>> for c,v in enumerate(array):
    ...     print c,v
    ...
    0 a
    1 b
    2 c
    3 d
    >>> for c,v in enumerate(array,1):
    ...     print c,v
    ...
    1 a
    2 b
    3 c
    4 d
    
如果要从指定索引开始遍历到结尾，可以利用 `range()` 函数：
    
    >>> array=['a','b','c','d']
    >>> for i in range(1,len(array)):
    ...     print i,array[i]
    ...
    1 b
    2 c
    3 d

## 3. 切片

有时候，我们想要精确的选取 list 中的一部分，Python 提供了优雅的切片语法：

    alist[begin:end:step]
    
* begin 表示切片的起始位置，缺省是 list 的开始位置
* end 表示切片的结束位置，缺省是 list 的结束位置
* step 表示步长，确实是正序的 1 ，逆序用负数表示

以字符串 `hello` 为例，切片语法的坐标如下：

    0   1   2   3   4   5
    +---+---+---+---+---+
    | h | e | l | l | o |
    +---+---+---+---+---+
    -6  -5  -4  -3  -2  -1
    
如果要取中间的 `el` :

    >>> a='hello'
    >>> a[1:3:1]
    'el'
    >>> a[1:3]
    'el'
    
将字符串逆序：

    >>> a[::-1]
    'olleh'
    >>> a[-1:-6:-1]
    'olleh'

取头尾两个字符：
    
    >>> a[::4]
    'ho'
