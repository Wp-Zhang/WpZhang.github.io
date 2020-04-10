---
layout:     post
title:      pandas基础-数据结构/基本运算
subtitle:   ""
date:       2019-02-15 05:16:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Data Science
---
## pandas入门
###### 建议与Jupyter Notebook配合阅读
---


```python
import pandas as pd
```

### 数据结构

#### Series -- 一维数据表


```python
obj = pd.Series([1,3,5,7,9])
obj
```

左边第一列为index，默认为range(n)，可以自己设置


```python
obj = pd.Series([1,3,5,7,9], index=['a','b','c','d','e'])
obj
```

通过.values获得值，.index获得索引


```python
obj.values
obj.index
```

可以通过标签对元素进行索引


```python
obj = pd.Series([1,3,5,7,9], index=['a','b','c','d','e'])
obj['a']
obj[0]
```

**Series对象可以由字典生成**，数组操作与Numpy中相似


```python
dic = {'a':1, 'b':2, 'c':3}
obj = pd.Series(dic)
obj
```


```python
# 可以通过设置索引改变排列顺序
index = ['b','c','a']
obj = pd.Series(dic, index)
obj
```

使用isnull和notnull检查缺失数据


```python
dic = {'a':1, 'b':2, 'c':3}
obj = pd.Series(dic, index=['d','a','b','c','e'])
obj 
```


```python
pd.isnull(obj)
pd.notnull(obj)
```

为Series对象设置name属性


```python
dic = {'a':1, 'b':2, 'c':3}
obj = pd.Series(dic, index=['d','a','b','c','e'])
obj.name = "Seties_obj 1"  #整个对象的name
obj.index.name = "Letter"  #索引的name
obj
```

#### DataFrame -- 二维数据表

常用包含**等长度列表或NumPy数组**的字典形成


```python
dic = {
    'Name':['Angela','Bob','Cindy','David'],
    'Sex':['Female','Male','Female','Male'],
    'Age':[23,25,64,10]
      }
df = pd.DataFrame(dic)
df
```

通过.head()获取前5行数据，常用于大型数据

通过传递参数columns指定列顺序


```python
df = pd.DataFrame(dic, columns=['Sex','Name','Age'], index=list(range(1,len(dic['Name'])+1)) )
df
```

像字典那样检索Series


```python
df['Name'] #通用
df.Name    #列名应是有效的Python变量名
```

转置操作与NumPy类似


```python
df.T
```

获取index和values操作与Series相同


```python
df.index
df.values
```

pandas中的索引对象像数组也像**固定大小**的集合，所以有一些相关类似方法

- append &nbsp;&nbsp; 产生新索引
- difference &nbsp;&nbsp; 两索引的差集
- intersection &nbsp;&nbsp; 两索引交集
- union &nbsp;&nbsp; 两索引并集
- isin
- delete &nbsp;&nbsp; 删除位置i的元素
- drop &nbsp;&nbsp; 删除指定索引值
- is_monotonic &nbsp;&nbsp; 索引序列是否递增
- is_unique &nbsp;&nbsp; 索引是否有重复元素
- unique &nbsp;&nbsp; 索引去重

***注；以上方法均针对index对象***

### 基本功能

#### 重建索引

使用.reindex()方法


```python
frame = pd.DataFrame(np.arange(9).reshape((3,3)),
                    index = ['a','b','c'],
                    columns = ['Chengdu', 'Changsha', 'Beijing'])
frame
```

##### 修改索引


```python
frame2 = frame.reindex(['c','a','b'])
frame2
```

##### 修改列名


```python
frame3 = frame.reindex(columns=['Beijing','Chengdu','Changsha'])
frame3
```

##### 也可以使用.loc[]方法修改

#### 删除条目


```python
frame = pd.DataFrame(np.arange(9).reshape((3,3)),
                    columns = ['a','b','c'],
                    index = ['Chengdu', 'Changsha', 'Beijing'])
frame
```

##### 删除行


```python
frame.drop(['Chengdu','Beijing'])
```

##### 删除列 -- 设置axis参数


```python
frame.drop('b', axis=1)
frame.drop('b', axis="columns")
```

##### 直接修改原对象 -- 设置inplace参数


```python
frame.drop('Beijing', inplace=True)
frame
```

#### 索引、选择与过滤

##### 基本操作

###### Series


```python
s = pd.Series(np.arange(2,7), index=['a','b','c','d','e'])
s
```


```python
s[0]
s[[0,2,4]]
s[:3]
s['a':'d'] ## 注意：Series通过索引的切片包含尾部！
s[s>3]
```

###### DataFrame


```python
df = pd.DataFrame(np.arange(1,17).reshape((4,4)),
                 index=['Beijing','Shanghai','Guangdong','Shenzhen'],
                 columns = ['a','b','c','d'])
df
```


```python
## 注：DataFrame中不能像 df[1] 这样进行单独索引行
df[2:]  #索引行
df[['b','d']]  #索引列
df[df['d']>10]
```

##### 使用loc和iloc选择器

- **loc**为***轴标签***，即通过字符串索引
- **iloc**为***整数标签***，即通过整数索引


```python
df.loc['Guangdong']
df.loc['Guangdong','b']
df.loc[['Guangdong','Beijing'],['a','d']]
df.loc[:, 'a':'c'][df.c>7]
```


```python
df.iloc[2]
df.iloc[2, 1]
df.iloc[[2,0], [0,3]]
df.iloc[:, :3][df.c>7]
```

#### 排序

##### 对索引排序


```python
df = pd.DataFrame(np.arange(1,17).reshape((4,4)),
                 index=['d','c','b','a'],
                 columns=['Beijing','Shanghai','Guangdong','Shenzhen'])
df
```

对行排序


```python
df.sort_index()
```

对列排序


```python
# 要降序排列时设置参数ascending
df.sort_index(axis=1, ascending=False)
```

按值排序


```python
# 当排序对象为DataFrame时要设置参数by以设定排序的标准
df.sort_values(by="Shanghai")
df.sort_values(by=['Beijing','Shanghai']) #多行排序，优先级依照列表顺序
```

### 统计与计算

#### 基础操作


```python
df = pd.DataFrame(np.arange(1,17).reshape((4,4)),
                 index=['d','c','b','a'],
                 columns=['Beijing','Shanghai','Guangdong','Shenzhen'])
df
```

###### 求和

1. 对列求和


```python
df.sum()
```

2. 对行求和


```python
df.sum(axis=1)
```

###### 求平均值

求平均值时将自动跳过为NaN的元素(如果有的话)，行、列选择与求和相同


```python
df.mean()
df.mean(axis=1)
```

###### 求最值

将返回最值所在位置的索引


```python
df.idxmax()
df.idxmin()
```

###### 积累性方法 -- cumsum与cumprod


```python
df.cumsum()
df.cumprod()
```

###### 描述方法


```python
df.describe()
```

#### 相关性和协方差

###### Series

1. 协方差


```python
df['Beijing'].cov(df['Shenzhen'])
df.Beijing.cov(df.Shenzhen)
```

2. 相关性


```python
df['Beijing'].corr(df['Shenzhen'])
df.Beijing.corr(df.Shenzhen)
```

###### DataFrame

1. 协方差


```python
df.cov()
```

2. 相关性


```python
# 计算两两之间的相关性
df.corr()
```


```python
# 计算与一行/列的相关性
df.corrwith(df['Beijing'])
```