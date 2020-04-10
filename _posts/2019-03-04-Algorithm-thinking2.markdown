---
layout:     post
title:      GooglePageRank算法
subtitle:   "算法思考"
date:       2019-03-04 12:09:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Algorithm
---
### 题目：学习Google Page Rank 算法
---
**算法核心思想：**
>- 如果一个网页被其他网页链接的次数较多，则说明该网页相较于其他网页更为重要（PageRank值更高）
- 如果一个PageRank值较高的网页链接了其他网页，则被链接的网页的PageRank值也会相应因此提高

#### 具体原理
&nbsp; &nbsp; &nbsp; &nbsp;预先给每个网页一个PageRank值，一般为1/N，（N为网页总数）。然后通过算法不断迭代直到PageRank达到平稳分布。
#### 情况1

![](/media/editor/1_20190304221812771706.png)

&nbsp; &nbsp; &nbsp; &nbsp;如图，当打开网页B时，其打开网页A和网页C的概率是一样的，所以

<div align=center>
<img src="http://latex.codecogs.com/gif.latex?PR%28A%29%20%3D%20%5Cfrac%7BPR%28B%29%7D%7B2%7D%20&plus;%20%5Cfrac%7BPR%28C%29%7D%7B1%7D" style="cursor: zoom-in;">
</div>

#### 情况2

![](/media/editor/2_20190304222001450844.png)

&nbsp; &nbsp; &nbsp; &nbsp;如图，网页C没有链接任何网站，于是认为其链接了所有网页(包括它自己)，于是有：

<div align=center>
<img src="http://latex.codecogs.com/gif.latex?PR%28A%29%20%3D%20%5Cfrac%7BPR%28B%29%7D%7B2%7D%20&plus;%20%5Cfrac%7BPR%28C%29%7D%7B3%7D" style="cursor: zoom-in;">
</div>

#### 情况3

![](/media/editor/3_20190304222141469059.png)

&nbsp; &nbsp; &nbsp; &nbsp;如图，网页B和网页C只链接了对方，在迭代过程中，它们的PR值将一直增加。为了解决该问题引入一个概率，当在每个页面将要访问下一个页面时，有一定概率会输入网址直接跳转到一个已有的随机网页，于是有：

<div align=center>
<img src="http://latex.codecogs.com/gif.latex?PR%28A%29%20%3D%20%5Cfrac%7B1-%5Calpha%20%7D%7B3%7D%20%5Ctimes%203" style="cursor: zoom-in;">
</div>

##### 总结
&nbsp; &nbsp; &nbsp; &nbsp;一般情况下，PageRank值的计算公式为：
<div align=center>
<img src="http://latex.codecogs.com/gif.latex?PR%28A%29%20%3D%20%5Calpha%20%5Csum_%7BY_%7Bi%7D%5Cepsilon%20S%28A%29%7D%5E%7B%20%7D%20%5Cfrac%7BPR%28Y_%7Bi%7D%29%7D%7Bn_%7Bi%7D%7D%20&plus;%20%281%20-%20%5Calpha%29" style="cursor: zoom-in;">
</div>

&nbsp; &nbsp; &nbsp; &nbsp;其中S(A)表示指向网页A的所有网页的集合，<img src="http://latex.codecogs.com/gif.latex?%7Bn_%7Bi%7D%7D">表示网页<img src="http://latex.codecogs.com/gif.latex?%7BY_%7Bi%7D%7D">链接的其他网页的数量。

&nbsp; &nbsp; &nbsp; &nbsp;还有另一个常用的公式：

<div align=center>
<img src="http://latex.codecogs.com/gif.latex?PR%28A%29%20%3D%20%5Calpha%20%5Csum_%7BY_%7Bi%7D%5Cepsilon%20S%28A%29%7D%5E%7B%20%7D%20%5Cfrac%7BPR%28Y_%7Bi%7D%29%7D%7Bn_%7Bi%7D%7D%20&plus;%20%5Cfrac%7B1%20-%20%5Calpha%7D%7BN%7D" style="cursor: zoom-in;">
</div>

&nbsp; &nbsp; &nbsp; &nbsp;N表示网页总数。

&nbsp; &nbsp; &nbsp; &nbsp;之后的代码实现用第一个公式。

### 代码实现（Python）
##### 以情况3中的图为例
```python
import numpy as np
import pandas as pd

alpha = 0.85    # α值

M = np.array([
    [0, 0, 0],
    [1, 0, 1],
    [1, 1, 0]
])

pr = np.around(np.array([1/3, 1/3, 1/3]), decimals=6)
iteration = pd.DataFrame([pr.T], columns=["PR(A)", "PR(B)", "PR(C)"])

while True:
    pre = pr.copy()
    pr = np.around(alpha * M @ pr + 1 - alpha, decimals=6)
    if (pr == pre).all():
        break
    tmp = pd.DataFrame([pr.T], columns=["PR(A)", "PR(B)", "PR(C)"])
    iteration = iteration.append(tmp, ignore_index=True)

print(iteration)
```
##### 输出结果
```python
       PR(A)     PR(B)     PR(C)
0   0.333333  0.333333  0.333333
1   0.150000  0.716666  0.716666
2   0.150000  0.886666  0.886666
3   0.150000  1.031166  1.031166
4   0.150000  1.153991  1.153991
5   0.150000  1.258392  1.258392
6   0.150000  1.347133  1.347133
7   0.150000  1.422563  1.422563
8   0.150000  1.486679  1.486679
9   0.150000  1.541177  1.541177
10  0.150000  1.587500  1.587500
11  0.150000  1.626875  1.626875
12  0.150000  1.660344  1.660344
13  0.150000  1.688792  1.688792
14  0.150000  1.712973  1.712973
15  0.150000  1.733527  1.733527
16  0.150000  1.750998  1.750998
17  0.150000  1.765848  1.765848
18  0.150000  1.778471  1.778471
19  0.150000  1.789200  1.789200
20  0.150000  1.798320  1.798320
21  0.150000  1.806072  1.806072
22  0.150000  1.812661  1.812661
23  0.150000  1.818262  1.818262
24  0.150000  1.823023  1.823023
25  0.150000  1.827070  1.827070
26  0.150000  1.830509  1.830509
27  0.150000  1.833433  1.833433
28  0.150000  1.835918  1.835918
29  0.150000  1.838030  1.838030
..       ...       ...       ...
51  0.150000  1.849664  1.849664
52  0.150000  1.849714  1.849714
53  0.150000  1.849757  1.849757
54  0.150000  1.849793  1.849793
55  0.150000  1.849824  1.849824
56  0.150000  1.849850  1.849850
57  0.150000  1.849872  1.849872
58  0.150000  1.849891  1.849891
59  0.150000  1.849907  1.849907
60  0.150000  1.849921  1.849921
61  0.150000  1.849933  1.849933
62  0.150000  1.849943  1.849943
63  0.150000  1.849952  1.849952
64  0.150000  1.849959  1.849959
65  0.150000  1.849965  1.849965
66  0.150000  1.849970  1.849970
67  0.150000  1.849974  1.849974
68  0.150000  1.849978  1.849978
69  0.150000  1.849981  1.849981
70  0.150000  1.849984  1.849984
71  0.150000  1.849986  1.849986
72  0.150000  1.849988  1.849988
73  0.150000  1.849990  1.849990
74  0.150000  1.849991  1.849991
75  0.150000  1.849992  1.849992
76  0.150000  1.849993  1.849993
77  0.150000  1.849994  1.849994
78  0.150000  1.849995  1.849995
79  0.150000  1.849996  1.849996
80  0.150000  1.849997  1.849997

[81 rows x 3 columns]
```