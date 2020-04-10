---
layout:     post
title:      Logistic回归
subtitle:   ""
date:       2019-03-28 14:17:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Machine Learning
    - Data Science
---
## 基本理论



- **优点**：计算代价不高，易于理解和实现

- **缺点**：容易欠拟合，分类精度可能不高

- **适用数据类型**：数值型和标称型数据



预测结果得出后要将其进行分类。原始的函数有跃阶函数（即当值小于某个临界点时得0，大于时得1），但这样的瞬间跳跃有些时候不便于处理，于是这里使用另一个函数——sigmoid函数



sigmoid函数：<img src="http://latex.codecogs.com/gif.latex?%5Csigma%20%28z%29%20%3D%20%5Cfrac%7B1%7D%7B1&plus;e%5E%7B-z%7D%7D">，作图可知sigmoid函数的图像是一条平滑的曲线，这让之后求梯度的步骤变得十分容易且分类效果也要好很多。



sigmoid函数的输入为z，最简单的情况下<img src="http://latex.codecogs.com/gif.latex?z%20%3D%20ax">，但是当有多个特征时，公式变为<img src="http://latex.codecogs.com/gif.latex?z%20%3D%20w_1x_1%20&plus;%20w_2x_2%20&plus;%20...%20&plus;%20w_nx_n">，为了便于数据的操作，引入矩阵运算：令<img src="http://latex.codecogs.com/gif.latex?w%20%3D%20%5Cbegin%7Bbmatrix%7Dw_1%20%26%20w_2%20%26%20...%20%26%20w_n%5Cend%7Bbmatrix%7D">，则有<img src="http://latex.codecogs.com/gif.latex?z%20%3D%20w%5ETx">



### 梯度上升/下降法



函数f在点(x, y)的梯度可以表示为：<img src="http://latex.codecogs.com/gif.latex?%5Ctriangledown%20f%28x%2Cy%29%20%3D%20%5Cbegin%7Bpmatrix%7D%5Cfrac%7B%5Cpartial%20f%28x%2Cy%29%7D%7B%5Cpartial%20x%7D%5C%5C%5Cfrac%7B%5Cpartial%20f%28x%2Cy%29%7D%7B%5Cpartial%20y%7D%20%5Cend%7Bpmatrix%7D">，即f(x,y)分别对x，y的偏导



于是可以得出梯度上升/下降法中w的迭代公式：<img src="http://latex.codecogs.com/gif.latex?w%20%3D%20w%20&plus;%20%5Calpha%20%5Ctriangledown_w%20f%28w%29">(梯度上升发为+，梯度下降法为-，二者的区别是前者求最大值，后者求最小值)，其中<img src="http://latex.codecogs.com/gif.latex?%5Calpha">为学习率，其值越大，学习速率越快，但也越容易导致迭代不收敛。



设置好学习率和初始的w矩阵以及最大的迭代次数后，就可以开始迭代。当到达最大迭代次数或某个停止条件时迭代停止，此时的w就是经过学习后的参数矩阵。



## 训练算法：使用梯度上升找到最佳参数



### 载入数据





```python

import pandas as pd

import numpy as np

```





```python

def loadData():

    """

    从目录读取测试数据，将数据切分为数据矩阵和标签矩阵，并以np.array的格式返回

    """

    with open('data/testSet.txt') as f:

        data = f.readlines()

    for index, line in enumerate(data):

        data[index] = [float(_) for _ in line.strip().split()]

    data = np.array(data)

    dataMat = data[:,:2]

    labelMat = data[:,-1].astype('int')

    

    return dataMat, labelMat

    

```



### 梯度上升算法



#### sigmoid函数





```python

def sigmoid(inX):

    return 1 / (1 + np.exp(-inX))

```



#### 梯度上升算法





```python

def gradAscent(dataMat, classLabels):

    dataMatrix = np.ones((dataMat.shape[0], 3))

    dataMatrix[:, 1:] = dataMat

    classLabels = classLabels.T.reshape(classLabels.shape[0],1)  # 将标签数据矩阵转置

    

    alpha = 0.01  # 学习率

    maxCycles = 500  # 最大迭代次数

    weights = np.ones((dataMatrix.shape[1],1))  # 初始化系数矩阵

    # 开始迭代

    for i in range(maxCycles):

        z = sigmoid(dataMatrix@weights)

        error = classLabels - z  # 计算错误量

        weights = weights + alpha * dataMatrix.T @ error  # 更新系数



    return weights

```





```python

dataMat, labelMat = loadData()

gradAscent(dataMat, labelMat)

```



## 分析数据：画出决策边界



### 画出拟合直线





```python

def plotBestFit(weight):

    import matplotlib.pyplot as plt

    

    dataMat, labelMat = loadData()

    x1 = []; y1 = []

    x2 = []; y2 = []

    for i in range(dataMat.shape[0]):

        if labelMat[i] == 1:

            x1.append(dataMat[i][0])

            y1.append(dataMat[i][1])

        else:

            x2.append(dataMat[i][0])

            y2.append(dataMat[i][1])

    fig = plt.figure(figsize=(12,10)).add_subplot(111)

    fig.scatter(x1, y1, s=30, c='red')

    fig.scatter(x2, y2, s=30, c='c')

    

    x = np.arange(-3, 3, 0.1)

    y = (-weight[0]-weight[1]*x) / weight[2]

    fig.plot(x, y)

    plt.show()

```





```python

plotBestFit(gradAscent(dataMat, labelMat))

```



![](/img/in-post/old/output_20_0_20190328222420960806.png)





## 训练算法：随机梯度上升?



之前的梯度上升算法每次更新系数都要遍历一次数据集，当数据集很庞大时将极为耗时。





```python

def stocGradAscent(dataMat, classLabels):

    dataMatrix = np.ones((dataMat.shape[0], dataMat.shape[1]+1))

    dataMatrix[:, 1:] = dataMat

    classLabels = classLabels.T.reshape(classLabels.shape[0],1)  # 将标签数据矩阵转置

    

    alpha = 0.01  # 学习率

    weights = np.ones(dataMatrix.shape[1])  # 初始化系数矩阵

    # 开始迭代

    for i in range(dataMatrix.shape[0]):

        z = sigmoid(sum(dataMatrix@weights))

        error = classLabels[i] - z  # 计算错误量

        weights = weights + alpha * error * dataMatrix[i]  # 更新系数

    return weights

```





```python

plotBestFit(stocGradAscent(dataMat, labelMat))

```



![](/img/in-post/old/output_24_0_20190328222348320357.png)







```python

def stocGradAscent1(dataMatrix, classLabels, numIter=150):

    classLabels = classLabels.T.reshape(classLabels.shape[0],1)  # 将标签数据矩阵转置

    

    alpha = 0.01  # 学习率

    weights = np.ones(dataMatrix.shape[1])  # 初始化系数矩阵

    # 开始迭代

    import random

    for j in range(numIter):

        randIndex = list(range(dataMatrix.shape[0]))

        random.shuffle(randIndex)

        for i in range(dataMatrix.shape[0]):

            alpha = 4 / (1 + i + j) + 0.01

            index = randIndex.pop(0)

            z = sigmoid(sum(dataMatrix[index]*weights))

            error = classLabels[index] - z  # 计算错误量

            weights = weights + alpha * error * dataMatrix[index]  # 更新系数

    return weights

```



## 示例：从疝气病症预测病马的死亡率



### 准备数据：处理缺失值



由于有缺失值时NumPy数组无法进行运算，同时也会影响分类，于是需要对其进行填补。常用的缺失值填补方法有多种，但是这里用0填补。因为根据系数的迭代公式可以知道当系数为0时，该系数将在跌倒过程中保持不变，即不会对前期更新的系数产生负面影响。



### 测试算法：用Logistic回归进行分类





```python

def classifyVector(inX, weights):

    prob = sigmoid(inX@weights)

    return 1 if prob > 0.5 else 0

```





```python

def colicTest():

    # 准备数据

    with open('data/horseColicTest.txt') as f:

        strTrain = f.readlines()

    for index, line in enumerate(strTrain):

        strTrain[index] = [float(_) for _ in line.strip().split()]

    trainingData = np.array(strTrain)

    

    # 训练分类器

    trainSet = trainingData[:, :-1]  # 训练数据

    trainLabel = trainingData[:, -1]  # 训练标签

    trainWeights = stocGradAscent1(trainSet, trainLabel, 500)

    

    # 测试分类器

    errorCount = 0

    with open('data/horseColicTraining.txt') as f:

        testData = f.readlines()

    numTestVec = len(testData)

    for line in testData:

        currLine = line.strip().split('\t')

        currLine = np.array(currLine, dtype='float')

        #print(trainWeights.shape)

        #print(currLine.shape)

        if int(classifyVector(currLine[:-1], trainWeights)) != int(currLine[-1]):

            errorCount += 1

    errorRate = errorCount / numTestVec

    print("the error rate of this test is: %f" % errorRate)

    return errorRate

```





```python

def multiTest():

    numTests = 10

    errorSum = 0

    for k in range(numTests):

        errorSum += colicTest()

    print("after %d iterations the average error rate is: %f" % (numTests, errorSum/numTests))

```





```python

multiTest()

```



    the error rate of this test is: 0.314381

    the error rate of this test is: 0.334448

    the error rate of this test is: 0.371237

    the error rate of this test is: 0.327759

    the error rate of this test is: 0.314381

    the error rate of this test is: 0.317726

    the error rate of this test is: 0.374582

    the error rate of this test is: 0.364548

    the error rate of this test is: 0.311037

    the error rate of this test is: 0.324415

    after 10 iterations the average error rate is: 0.335452

    



multiTest函数运行了10次分类函数，计算了错误率的平均值。