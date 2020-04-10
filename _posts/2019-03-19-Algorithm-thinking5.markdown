---
layout:     post
title:      求乱序序列的中位数
subtitle:   "算法思考"
date:       2019-03-19 16:36:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Algorithm
---
### 题目：求乱序序列的中位数

---

#### 方法一

&nbsp; &nbsp; &nbsp; &nbsp; 最容易想到的解法，先将序列排序，再求出中位数



##### Python代码：

```python

def get_median_1(l):

	l = quick_sort(l)	# 这里没有使用自带的sorted函数是为了之后直观地比较两种算法的区别

	if len(l)%2 == 0:

        return (l[int(len(l)/2)] + l[int(len(l)/2)-1]) / 2

    else:

        return l[int(len(l)/2)]

```



#### 方法二

&nbsp; &nbsp; &nbsp; &nbsp; 运用快速排序的思想，算法和topK问题的基本相同（之前也有写过相关的博文，可以看一下）。



>设序列长度为n



>若序列长度为奇数：



>1. 找出序列中最大的 **int(n/2)+1** 个数，形成新序列subset；



>2. 找出subset中最小的数即为中位数





>若序列长度为偶数：



>1. 找出序列中最大的 **n/2+1** 个数，形成新序列subset；



>2. 找出subset中最小的两个数，求得的平均值即中位数



##### Python代码

```python

import numpy as np



def getK(l, k, direction='top'):

	# l为numpy的array类型

    if len(l) <= k:

        return l

    pivot = l[-1]

    if direction == 'top':

        tmp = l[:-1][l[:-1] > [pivot]]

        subset = np.insert(tmp, 0, pivot)

        

    if direction == 'min':

        tmp = l[:-1][l[:-1] < [pivot]]

        subset = np.insert(tmp, 0, pivot)

        

    length = len(subset)

    if length == k:

        return subset

    if length > k: 

        return getK(subset, k, direction)

    else:

        subset2 = l[:-1][l[:-1] <= [pivot]] if direction == 'top' else l[:-1][l[:-1] >= [pivot]]

        return np.append(getK(subset2, k-length, direction), subset)



def get_median_2(l):

    if len(l)%2 != 0:

        result = getK(l, int(len(l)/2+1), 'top')

        result = getK(result, 1, 'min')[0]

    else:

        result = getK(l, int(len(l)/2)+1, 'top')

        result = sum(getK(result, 2, 'min'))/2

    return result

```



#### 两种算法直观比较：



![](/img/in-post/old/plot_20190320014027694428.png)