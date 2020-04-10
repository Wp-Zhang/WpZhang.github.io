---
layout:     post
title:      寻找集合的所有子集
subtitle:   "算法思考"
date:       2019-03-09 14:57:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Algorithm
---
### 题目：找出一个集合的所有子集

---

#### 方法一

&nbsp; &nbsp; &nbsp; &nbsp; 仿照人工寻找子集的方式：设集合为 **[a, b, c, d]** 则寻找顺序为：

>[a, b, c, d]<br>

>[ab, ac, ad], [bc, bd], [cd]<br>

>[abc, abd, acd], [bcd]<br>

>[abcd]



##### Pyhton代码

```python

def getSubset1(l):

    set = [l]

    for i in range(2, len(l)+1):

        tmpSet = []

        for j in range(len(set[-1])):

            tmpSet2 = []

            for k in range(j+1, len(set[-1])):

                for element in set[-1][k]:

                    tmpSet2.append(l[j] + element)

            if tmpSet2:

                tmpSet.append(tmpSet2)

        set.append(tmpSet)

    return set

```



#### 方法二

&nbsp; &nbsp; &nbsp; &nbsp; 采用如下的寻找子集的方式：



1. 初始化空嵌套列表 `[[]]`

2. 遍历集合，将当前元素与当前子集列表中每一个元素分别相加，将产生的新子集列表添加至就的子集列表中



&nbsp; &nbsp; &nbsp; &nbsp; 仍以 **[a, b, c, d]** 为例：

>[ ] # 初始情况<br>

>[ ], [ a ] # a<br>

>[ ], [ a ], [ b ], [ ab ] # b<br>

>[ ], [ a ], [ b ], [ ab ], [ c ], [ ac ], [ bc ], [ abc ]# c<br>

>[ ], [ a ], [ b ], [ ab ], [ c ], [ ac ], [ bc ], [ abc ], [ d ], [ ad ], [ bd ], [ abd ], [ cd ], [ acd ], [ bcd ], [ abcd ] # d



##### Python代码

```python

def getSubset2(l):

    result = [[]]

    for x in l:

        result.extend([subset + [x] for subset in result])

    return result

```

&nbsp; &nbsp; &nbsp; &nbsp; **显然这个算法较第一种简洁很多且速度快了很多**



###### 直观比较如下



![](/img/in-post/old/compare1_20190310174443735630.png)



#### 方法三

&nbsp; &nbsp; &nbsp; &nbsp; 利用二进制，***速度奇快*** 。易知长度为N的集合有 (2^N-1) 个子集，1~2^N-1的二进制表示分别对应了1种情况的元素的索引值，例：**“1”**对应的二进制数可以表示为**“0001”**，则对应的是**"a"**;**“3”**对应的二进制数可以表示为**“0011”**，则对应的是**"ab"**

```python

def getSubset3(l):

    N = len(l)

    for i in range(2**N):

        combo = []

        for j in range(N):

            if(i >> j ) % 2 == 1:

                combo.append(l[j])

        yield combo

```

&nbsp; &nbsp; &nbsp; &nbsp; 前几种方法当集合长度为20+时速度就明显下降，25+以后要等很长时间，但是这个算法...



![](/img/in-post/old/quick_20190310174913020197.png)