---
layout:     post
title:      Numpy基础
subtitle:   ""
date:       2019-02-15 05:09:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Data Science
---
## Numpy基础
###### 建议与Jupyter Notebook配合阅读
---


```python
import numpy as np
```

### 概述

NumPy特点：
- 内部数据存储在连续的内存块上
- 算法库使用C语言编写
- 针对全量数组进行复杂运算，**不用使用循环**
- NumPy方法比Python方法 **快** 10~100倍，使用 *更少* 的内存

### 多维数组对象

#### 生成ndarray

###### np.array -- 由列表生成数组


```python
data1 = [1,2,3,4,5]
a = np.array(data1)
a
# 自动推断生成的数据数据类型
a.dtype
```

###### np.zeros -- 生成全0数组


```python
# 设置参数dtype可以设定数据类型
np.zeros(10, dtype=np.int64)

np.zeros((3, 4))

np.zeros((2, 4, 3))
```

###### np.arange -- 类似于range


```python
np.arange(20)
```

###### np.ones -- 生成全1数组


```python
np.ones(10)
```

###### np.eye -- 生成N×N特征矩阵


```python
np.eye(8)
```

#### 数据类型

- int, uint
- float
- complex
- bool
- object
- string_
- unicode_

###### astype -- 转换数据类型


```python
arr = np.arange(20)
arr_ = arr.astype(np.float16)

arr.dtype
arr_.dtype
```

#### 数组算数 -- 向量化


```python
arr = np.array([range(5), range(3,8), range(10,15)])
arr
```


```python
arr * arr
```


```python
1 / arr
```


```python
arr < arr*arr
```

#### 基础索引与切片

基本操作与Python内建的list相同，多维数组略有区别


```python
arr = np.array([range(1,4), range(2,5), range(3,6)])
arr
```


```python
arr[0][1]
```


```python
arr[0, 1]
```


```python
arr[0, :2]
```


```python
arr[1:, :2]
```

#### 布尔索引


```python
words = np.array(["Apple", "Bat", "Cake", "Dog", "Bat"])
condition = words == "Bat"

words
condition
```


```python
arr2 = np.array([range(4,7), range(5,8), range(6,9), range(7,10), range(8,11)])
arr2
arr2[condition]
```

#### 神奇索引 -- 按指定顺序索引


```python
arr = np.array([range(4,7), range(5,8), range(6,9)])
arr
arr[[2,1,0]]
```

***神奇索引将数据复制到新数组中***

#### 数组转置


```python
arr = np.array([range(4,7), range(6,9), range(8,11)])
arr
arr.T
```

### 通用函数 -- 逐元素数组函数

#### 一元通用函数

- abs, fbs &nbsp;&nbsp; 绝对值
- sqrt &nbsp;&nbsp; 开方
- square &nbsp;&nbsp; 平方
- exp &nbsp;&nbsp; 自然指数值e^x
- log log2 log10 log1p &nbsp;&nbsp; log1p:log(1+x)
- ...

#### 二元通用函数

- add &nbsp;&nbsp; 数组相加
- multiply &nbsp;&nbsp; 元素对应相乘
- divide &nbsp;&nbsp; 除
- ...

### 面向数组编程

#### 将条件逻辑作为数组操作


```python
arr = np.random.randn(5,5)
#将数组中大于0的元素置1，其余置0
result = np.where(arr>0, 1, 0) #np.where(condition, if condition, else)

arr
result
```

#### 数学和统计方法

- sum &nbsp;&nbsp; 求和
- mean &nbsp;&nbsp; 求平均值
- std, var &nbsp;&nbsp; 求标准差，方差
- min, max &nbsp;&nbsp; 最小值，最大值
- argmin, argmax &nbsp;&nbsp; 最小值，最大值的位置
- cumsum &nbsp;&nbsp; 从0开始元素累积和
- cumprod &nbsp;&nbsp; 从1开始元素累积积


```python
# cumsum
arr = np.arange(1,11)
arr
```


```python
np.cumsum(arr)
```


```python
# cumprod
np.cumprod(arr)
```

#### 集合逻辑

- unique(x) &nbsp;&nbsp; 对x去重并排序
- intersect1d(x,y) &nbsp;&nbsp; 计算x和y交集并排序
- union1d(x,y) &nbsp;&nbsp; 计算x和y并集并排序
- in1d(x,y) &nbsp;&nbsp; 判断x中元素是否在y中，返回布尔数组
- setdiff1d(x,y) &nbsp;&nbsp; 差集
- setxor1d(x,y) &nbsp;&nbsp; 异或集

### 文件输入输出

#### 存储


```python
arr = np.arange(10)
np.save('file_name', arr)
```

文件类型为.npy

#### 读取


```python
arr = np.load('file_name.npy')
```

### 线性代数


```python
x = np.array([range(1,4), range(2,5)])
y = np.array([range(3,5), range(4,6), range(5,7)])
```

#### 矩阵点乘


```python
# 以下三个操作等价
np.dot(x,y)
x.dot(y)
x @ y
```

#### 其他操作


```python
import numpy.linalg
```