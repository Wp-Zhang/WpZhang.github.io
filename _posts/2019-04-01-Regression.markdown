---
layout:     post
title:      预测数值型数据：回归
subtitle:   ""
date:       2019-04-01 15:38:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Machine Learning
    - Data Science
---
## 用线性回归找到最佳拟合直线



### 理论



将预测目标表示为由特征组成的线性回归方程，每个特征对应一个系数。当有新数据输入时将新数据的特征乘上对应的系数求和得到最终的预测结果。



通过不断调整系数向量，使预测结果与实际值的误差尽量小，最终得到最优系数。



计算误差的常用方法为计算平方误差：<img src="http://latex.codecogs.com/gif.latex?%5Csum%5Em_%7Bi%3D1%7D%28y_i-x_i%5ETw%29%5E2">，可用矩阵表示为<img src="http://latex.codecogs.com/gif.latex?%28y-Xw%29%5ET%28y-Xw%29">。

当矩阵X可以求逆时，将均方差公式求导可以得出公式：<img src="http://latex.codecogs.com/gif.latex?%5Cwidehat%7Bw%7D%20%3D%20%28X%5ETX%29%5E%7B-1%7DX%5ETy">，可以根据公式估计w的最优解。这就是最小二乘法。



### 算法实现



#### 载入数据





```python

import numpy as np

```





```python

def loadDataSet(fileName):

    with open(fileName) as f:

        data = f.readlines()

    dataMat = np.mat(np.zeros((len(data), len(data[0].strip().split('\t'))-1)))

    labels = np.mat(np.zeros((1, len(data))))

    for i,line in enumerate(data):

        tmp = line.strip().split('\t')

        dataMat[i,:] = np.array(tmp[:-1],dtype='float')

        labels[0,i] = float(tmp[-1])

    return dataMat, labels

```



#### 计算最佳拟合直线





```python

def standRegres(xMat, yArr):

    yMat = yArr.T

    xTx = xMat.T @ xMat

    if np.linalg.det(xTx) == 0:  # 矩阵无法求逆

        print("This matrix is singular, cannot do inverse.")

        return

    w = xTx.I @ xMat.T @ yMat

    return w

```





```python

xMat, yArr = loadDataSet('data/ex0.txt')

```





```python

standRegres(xMat, yArr)

```









    matrix([[3.00774324],

            [1.69532264]])







#### 绘制拟合曲线





```python

import matplotlib.pyplot as plt

```





```python

def plotRegresLine(xMat, yMat, w):

    fig = plt.figure()

    ax = fig.add_subplot(111)

    ax.scatter(xMat[:,1].flatten().A[0], yMat.T[:,0].flatten().A[0])  # 将X和Y降维成一位数组后绘制散点图

    # 将X排序后绘制预测直线

    xCopy = xMat.copy()

    xCopy.sort(0)

    yHat = xCopy @ w

    ax.plot(xCopy[:, 1], yHat)

    plt.show()

```





```python

wHat = standRegres(xMat, yArr)

plotRegresLine(xMat, yArr, wHat)

```



![](/img/in-post/old/output_17_0_20190401234014801681.png)





可以通过预测结果和实际值的相关系数来判断模型的预测效果。





```python

yHat = xMat @ wHat

np.corrcoef(yHat.T, yArr)

```









    array([[1.        , 0.98647356],

           [0.98647356, 1.        ]])







可见在这个案例中预测值和实际值相关系数为0.987,模型表现效果较好。



## 局部加权线性回归



### 理论



线性回归的很容易出现欠拟合现象，因为它求的是具有最小均方误差的无偏估计。如果允许在估计时引入一些偏差，可能可以降低预测的均方误差。其中一个方法就是局部加权线性回归。



这个方法与KNN相似，每次预测时取出目标预测点附近的数据子集，给数据子集中的每个点赋予一定的权重然后基于最小均方差来进行普通的回归。



回归系数的计算公式：<img src="http://latex.codecogs.com/gif.latex?%5Cwidehat%7Bw%7D%20%3D%20%28X%5ETWX%29%5E%7B-1%7DX%5ETWy">，其中W是权重矩阵



局部加权线性回归使用“核”来对附近的点赋予更高的权重。常用高斯核：<img src="http://latex.codecogs.com/gif.latex?w%28i%2Ci%29%20%3D%20exp%5Cleft%28%5Cfrac%7B%5Cleft%7Cx%5E%7B%28i%29%7D-x%5Cright%7C%7D%7B-2k%5E2%7D%5Cright%29">根据公式可以看出：点x与点x(i)越近，w(i,i)将会越大。公式中的k需要自行设置，当k越小，能用于训练回归模型的数据量就越小。



### 算法实现



#### 利用加权线性回归预测单条数据





```python

def lwlr(testPoint, xMat, yArr, k=1.0):

    yMat = yArr.T

    m = xMat.shape[0]

    weights = np.mat(np.eye((m)))  # 初始化权重矩阵为单位矩阵

    # 使用高斯核计算权重矩阵

    for j in range(m):

        diffMat = testPoint - xMat[j,:]

        weights[j,j] = np.exp(diffMat*diffMat.T / (-2*k**2))

        #weights[j,j] = np.exp(np.linalg.det(diffMat) / (-2*k**2))

    xTx = xMat.T @ weights @ xMat

    # 矩阵不可逆

    if np.linalg.det(xTx) == 0:

        print("This matrix is singular, cannot do inverse.")

        return

    result = xTx.I @ xMat.T @ weights @ yMat

    return testPoint @ result

```



#### 利用加权线性回归预测数据集





```python

def lwlrTest(testArr, xMat, yArr, k=1.0):

    m = xMat.shape[0]

    yHat = np.zeros(m)

    for i in range(m):

        yHat[i] = lwlr(testArr[i], xMat, yArr, k)

    return yHat

```



#### 绘制拟合曲线





```python

def plotLine(k,fig,plot_position):

    xMat, yArr = loadDataSet('data/ex0.txt')

    yHat = lwlrTest(xMat, xMat, yArr, k)

    ax = fig.add_subplot(plot_position)

    ax.scatter(xMat[:,1].flatten().A[0], yArr.T[:,0].flatten().A[0], s=2)  # 将X和Y降维成一位数组后绘制散点图

    # 将X排序后绘制预测直线

    sortedIndex = xMat[:,1].argsort(0)

    xCopy = xMat.copy()[sortedIndex][:,0]

    ax.plot(xCopy[:, 1], yHat[sortedIndex])

```





```python

fig = plt.figure(figsize=(8,15))

plotLine(1, fig, 311)

plotLine(0.01, fig, 312)

plotLine(0.003, fig, 313)

```



![](/img/in-post/old/output_34_0_20190401234028717750.png)





可见当k过小时容易发生过拟合现象，原因是考虑了太多噪声。



## 示例：预测鲍鱼的年龄



### 读取数据





```python

abX, abY = loadDataSet('data/abalone.txt')

```



### 训练模型





```python

yHat01 = lwlrTest(abX[:99,:], abX[:99,:], abY[:,:99], 0.1)

yHat1 = lwlrTest(abX[:99,:], abX[:99,:], abY[:,:99], 1)

yHat10 = lwlrTest(abX[:99,:], abX[:99,:], abY[:,:99], 10)

```



### 计算误差





```python

def rssError(yArr, yHat):

    return ((yArr - yHat)**2).sum()

```



训练数据集误差计算





```python

rssError(abY[:,:99].flatten().A[0], yHat01)

rssError(abY[:,:99].flatten().A[0], yHat1)

rssError(abY[:,:99].flatten().A[0], yHat10)

```









    56.81549669032892



    429.8905618702056



    549.1181708826584







测试数据将误差计算





```python

yHat01 = lwlrTest(abX[100:199,:], abX[:99,:], abY[:,:99], 0.1)

yHat1 = lwlrTest(abX[100:199,:], abX[:99,:], abY[:,:99], 1)

yHat10 = lwlrTest(abX[100:199,:], abX[:99,:], abY[:,:99], 10)

```





```python

rssError(abY[:,100:199].flatten().A[0], yHat01)

rssError(abY[:,100:199].flatten().A[0], yHat1)

rssError(abY[:,100:199].flatten().A[0], yHat10)

```









    40659.27596541478



    573.5261441898057



    517.5711905382693







可见k越小在训练集上表现越好，但是过拟合严重，在测试集上表现很差



## 缩减系数来“理解”数据



当数据的特征比样本点还多时，用之前的最小二乘法求系数矩阵时由于X不是满秩矩阵，在求逆时会出现问题，因此引入了接下来的方法。



### 岭回归



#### 理论



简单来说，岭回归就是在矩阵$X^TX$上加入一个<img src="http://latex.codecogs.com/gif.latex?%24%5Clambda%20I%24">使得矩阵非奇异，从而可以进行求逆运算。矩阵I是一个mxm的单位矩阵，λ的数值由用户自行设置。



因此，回归系数计算公式变为：<img src="http://latex.codecogs.com/gif.latex?%5Cwidehat%7Bw%7D%20%3D%20%28X%5ETX&plus;%5Clambda%20I%29%5E%7B-1%7DX%5ETy">



岭回归最初用来处理特征数多余样本数的情况，现在也用于在估计中加入偏差从而获得更好的估计。引入的λ作为惩罚项能够减少不重要的参数。



与之前相同，通过预测误差最小化求得λ。



#### 算法实现



##### 求回归系数





```python

def ridgeRegress(xMat, yMat, lam=0.2):

    xTx = xMat.T @ xMat

    tmp = xTx + np.eye(xMat.shape[1])*lam

    if np.linalg.det(tmp) == 0:

        print("This matrix is singular, cannot do inverse")

        return

    result = tmp.I @ xMat.T @ yMat

    return result

```



##### 岭回归预测





```python

def ridgeTest(xMat, yArr):

    yMat = yArr.T

    # 数据标准化

    yMean = np.mean(yMat, 0)

    yMat = yMat - yMean

    xMeans = np.mean(xMat, 0)

    xVar = np.var(xMat, 0)

    xMat = (xMat - xMeans) / xVar

    

    numTestPts = 30  # 测试的λ个数

    wMat = np.zeros((numTestPts,xMat.shape[1]))  # 存储所有λ对应的回归系数

    for i in range(numTestPts):

        ws = ridgeRegress(xMat, yMat, np.exp(i-10))

        wMat[i,:] = ws.T

    

    return wMat

```





```python

abX, abY = loadDataSet('data/abalone.txt')

```





```python

ridgeWeights = ridgeTest(abX, abY)

```



##### 不同lambda模型效果比较





```python

plt.plot(ridgeWeights)

plt.xlabel("log(λ)")

plt.show()

```



![](/img/in-post/old/output_65_2_20190401234053872011.png)





### lasso



与岭回归类似，lasso也对回归系数做了限定：<img src="http://latex.codecogs.com/gif.latex?%5Csum%5En_%7Bk%3D1%7D%5Cleft%7Cw_k%5Cright%7C%5Cleqslant%5Clambda">



通过这个限制条件，当λ足够小的时候，部分系数会被缩减到0，缩减特征的同时帮助我们更好地理解数据。



虽然效果很好，但是极大地增加了计算复杂度。下面的方法更为简单。



### 前向逐步回归



#### 理论



前向逐步回归可以得到与lasso差不多的效果但更加简单。它属于一种贪心算法即每一步都尽可能减小误差。初始时所有权重为1，每一步所做的决策是对某个权重增加或减少一个很小的值



#### 算法实现





```python

def regularize(xMat):

    inMat = xMat.copy()

    inMeans = np.mean(inMat,0)

    inVar = np.var(inMat,0)

    inMat = (inMat - inMeans)/inVar

    return inMat

```





```python

def stageWise(xMat, yArr, eps=0.1, numIt=100):

    """

    eps: 权重每次改变的步长

    numIt: 迭代次数

    """

    yMat = yArr.T

    # 数据标准化

    yMean = np.mean(yMat,0)

    yMat = yMat - yMean

    xMat = regularize(xMat)

    # 前向逐步回归

    returnMat = np.zeros((numIt,xMat.shape[1]))

    ws = np.zeros((xMat.shape[1],1))

    for i in range(numIt):

        #print(ws.T)

        wsMax = ws.copy()

        lowestError = np.inf  # 初始化最小误差为无穷大

        # 逐步修改每个特征的权重

        for j in range(xMat.shape[1]):

            # 权重增或减

            for sign in [-1,1]:

                wsTest = ws.copy()

                wsTest[j] += eps * sign

                yTest = xMat @ wsTest

                # 计算误差

                rssE = rssError(yMat.A, yTest.A)

                if rssE < lowestError:

                    lowestError = rssE

                    wsMax = wsTest

        ws = wsMax.copy()

        returnMat[i,:] = ws.T

    return returnMat

```





```python

xMat, yArr = loadDataSet('data/abalone.txt')

```





```python

weights = stageWise(xMat, yArr, 0.005, 1000)

plt.plot(weights)

plt.show()

```



![](/img/in-post/old/output_77_1_20190402122318183132.png)





可见当迭代次数到达600后，权重趋近于一个稳定值。这样的图像充分体现了模型的好处。