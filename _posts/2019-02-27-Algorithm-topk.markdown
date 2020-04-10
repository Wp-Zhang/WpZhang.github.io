---
layout:     post
title:      topk问题堆排序、快速排序比较
subtitle:   "算法思考"
date:       2019-02-27 13:51:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Algorithm
---
### 题目：分析求解topk(big)问题时 *堆排序* 和 *快速排序* 的使用场景

---

#### 快速排序求解

&#160; &#160; &#160; &#160;这种算法利用了快速排序的主要思想：每次调用快速排序的partition函数后将会得出一个数n的最终位置，比较n及其右边所有数的总个数与目标个数k



	若大于k，则对数n右边的数构成的数组调用partition函数；

	若等于k，则说明n及其右边的数就是我们想要得到的那k个数，排序结束；

	若小于k，这说明最终的k个数中还包含了n左边的数，对n左边的数构成的数组调用partition函数。



Pyhton代码：

```python

import numpy as np



def qselect(A, k):

    if len(A) < k:

        return A

    pivot = A[-1]

    tmp = A[:-1][A[:-1] >= [pivot]]

    right = np.insert(tmp, 0, pivot)

    try:

        rlen = len(right)

    except:

        rlen = 1

    if rlen == k:

        return right

    if rlen > k:

        return qselect(right, k)

    else:

        left = [x for x in A[:-1] if x < pivot]

        return np.append(qselect(left, k-rlen), right)



L = np.random.randint(0, 10000, size=10000)

k = 100

qselect(L, k)

```



&#160; &#160; &#160; &#160;利用快速排序解决topk问题时,速度非常快，但是缺点它在数组中直接交换数据，所以要用一个数组来存储所有的数据，当数据量十分庞大时将占用惊人的内存，属于用空间换时间的算法。



#### 堆排序

&#160; &#160; &#160; &#160;这种算法的大致思路为：首先初始化一个大小为k的小根堆，然后遍历数组，每遍历一个数就将其与堆的根节点相比较，若比根节点大，则将其与根节点交换变更新堆结构。当数组遍历完成时堆中存储的就是我们想要的结果。



Pyhton代码：

```python

import numpy as np



def sift(l, low, hight):

    # 调整堆结构

    tmp = l[low]

    i = low

    j = 2 * i + 1

    while j <= hight:  # 情况2：i已经是最后一层

        if j + 1 <= hight and l[j + 1] < l[j]:  # 右孩子存在并且小于左孩子

            j += 1

        if tmp > l[j]:

            l[i] = l[j]

            i = j

            j = 2 * i + 1

        else:

            break  # 情况1：j位置比tmp小

    l[i] = tmp



def top_k(l, k):

    heap = l[:k]

    # 建堆

    for i in range(k // 2 - 1, -1, -1):

        sift(heap, i, k - 1)

    for i in range(k, len(l)):

        if l[i] > heap[0]:

            heap[0] = l[i]

            sift(heap, 0, k - 1)



    for i in range(k - 1, -1, -1):

        heap[0], heap[i] = heap[i], heap[0]

        sift(heap, 0, i - 1)

    return heap



L = np.random.randint(0, 10000, size=10000)

k = 100

top_k(L, k)

```



&#160; &#160; &#160; &#160;利用堆排序解决topk问题时，速度是所有算法中除了快速排序之外最快的，它的突出优点就是不论数据量有多大，它的空间复杂度总是常数级的，在处理大数据时常将数据分块打包后分别利用堆排序进行筛选，再将筛选结果进行比较，选出我们想要的最终结果。



#### 两种算法直观比较：

![](/img/in-post/old/topk_20190227222216653630.png)

>纵坐标为耗时，横坐标为初始序列长度，数值取值范围为[0,999999999]，k(即最终筛选出的数值个数)为100，