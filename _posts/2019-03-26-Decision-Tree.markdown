---
layout:     post
title:      决策树
subtitle:   ""
date:       2019-03-26 04:18:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Machine Learning
    - Data Science
---
## 决策树的构造

> 将原始数据集分为若干个数据子集，它们分布在第一个决策点的所有分支上。若某分支下的数据属于同一类型，则当前数据集已经分类完成，无需对其进行进一步分割；否则重复划分数据子集直到每个数据子集仅包含一个类型的数据。

### 信息增益

> 划分数据集的大原则是：将无序的数据变得更加有序。再划分数据集前后信息发生的变化称为信息增益， 集合信息的度量方式称为香农熵或熵。

#### 基本概念

&nbsp; &nbsp; &nbsp; &nbsp; 熵为信息的期望值，而“信息”的定义如下：

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 若待分类的事务可能划分在多个分类中，则符号 <img src="http://latex.codecogs.com/gif.latex?%24x_%7Bi%7D%24"> 的信息为：<img src="http://latex.codecogs.com/gif.latex?l%28x_%7Bi%7D%29%20%3D%20-%5Clog_%7B2%7Dp%28x_%7Bi%7D%29"> , 其中 <img src="http://latex.codecogs.com/gif.latex?p%28x_%7Bi%7D%29"> 是选择该分类的概率。

&nbsp; &nbsp; &nbsp; &nbsp; 为了计算信息熵，我们需要计算所有类别所有可能值包含的信息期望值：

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <img src="http://latex.codecogs.com/gif.latex?H%20%3D%20-%20%5Csum%5E%7Bn%7D_%7Bi%3D1%7Dp%28x_%7Bi%7D%29%5Clog_%7B2%7Dp%28x_%7Bi%7D%29"></img，>， 其中n为分类数目。

#### 计算香农熵

下面是计算给定数据集的熵的代码


```python
from math import log

def calcEnt(dataSet):
    """
    dataSet: pandas.DataFrame，最后一列为label
    """
    labelCounts = {}  # 存储所有分类的字典
    # 将所有可能分类加入字典
    for label in dataSet[dataSet.columns[-1]]:
        if label in labelCounts.keys():
            labelCounts[label] += 1
        else:
            labelCounts[label] = 1
    # 计算熵
    ent = 0
    for label in labelCounts:
        prob = labelCounts[label] / dataSet.shape[0]  # 计算该分类出现的概率
        ent -= prob * log(prob, 2)  # 计算香农熵
    return ent
```

可以自己创建数据集进行测试，会发现熵越高，混合的数据越多。

### 划分数据集

splitDataSet函数抽取数据集中column对应的列的值等于value的所有数据


```python
def splitDataSet(dataSet, column, value):
    """
    dataSet: pandas.DataFrame，最后一列为label
    column: 列名
    values: 列名对应列的值
    """
    result = dataSet[dataSet[column]==value]  # column列的值等于value的数据集
    result.drop(column, axis=1,inplace=True)  # 去掉column列
    return result
```

接下来遍历整个数据集，循环计算香农熵和执行splitDataSet()函数，找到最好的特征划分方式（根据之前提到的香农熵的数值特点得出）


```python
def chooseBestFeatureToSplit(dataSet):
    baseEntropy = calcEnt(dataSet)  # 初始香农熵
    bestInfoGain = 0  # 存储最佳信息增益
    bestFeature = None  # 存储最佳特征
    
    for i in dataSet.columns[:-1]:
        featList = dataSet[i]
        uniqueVals = set(featList)  # 不同的值
        newEntropy = 0
        # 计算每种划分方式的信息熵
        for value in uniqueVals:
            subDataSet = splitDataSet(dataSet, i, value)
            prob = subDataSet.shape[0] / dataSet.shape[0]
            newEntropy += prob * calcEnt(subDataSet)
        infoGain = baseEntropy - newEntropy  # 信息增益
        # 计算最好的信息增益
        if (infoGain > bestInfoGain):
            bestInfoGain = infoGain
            bestFeature = i
    return bestFeature
```

### 递归构建决策树

递归结束的条件是：程序遍历完所有划分数据集的属性，或每个分支下的所有实例都具有相同的分类。如果所有实例具有相同的分类，则得到一个叶子节点或终止块。因此任何达到叶子节点的数据必然属于叶子节点的分类。

如果数据集已经处理了所有属性，但是类标签依然不是唯一的，此时我们需要决定如何定义该叶子节点，在这种情况下，我们通常会采用多数表决的方法决定该叶子节点的分类。


```python
def majorityCnt(classList):
    classCount = {}
    for vote in classList:
        if vote not in classCount.keys(): classCount[vote] = 0
        classCount[vote] += 1
    sortedClassCount = sorted(classCount.items(), key=lambda x: x[1], reverse=True)
    return sortedClassCount[0][0]
```

创建决策树


```python
def createTree(dataSet):
    classList = dataSet[dataSet.columns[-1]]
    # 如果类别完全相同则停止继续划分
    if len(set(classList)) == 1:
        return classList.tolist()[0]
    # 遍历完所有特征是返回出现次数最多的特征
    if dataSet.shape[1] == 1:
        return majorityCnt(classList)
    
    bestFeat = chooseBestFeatureToSplit(dataSet)
    myTree = {bestFeat:{}}
    # 得到列表包含的所有属性值
    featValues = dataSet[bestFeat]
    uniqueVals = set(featValues)
    for value in uniqueVals:
        myTree[bestFeat][value] = createTree(splitDataSet(dataSet, bestFeat, value))
    return myTree
```

## 利用matplotlib注解绘制树形图


```python
import matplotlib.pyplot as plt
```


```python
decisionNode = dict(boxstyle="sawtooth", fc="0.8")
leafNode = dict(boxstyle="round4", fc="0.8")
arrow_args = dict(arrowstyle="<-")
```

### 绘制树节点


```python
def plotNode(nodeTxt, centerPt, parentPt, nodeType):
    createPlot.ax1.annotate(nodeTxt, xy=parentPt,  xycoords='axes fraction',
             xytext=centerPt, textcoords='axes fraction',
             va="center", ha="center", bbox=nodeType, arrowprops=arrow_args )
```

获取叶节点的数目和树的层数


```python
def getNumLeafs(myTree):
    numLeafs = 0
    firstStr = list(myTree.keys())[0]
    secondDict = myTree[firstStr]
    for key in secondDict.keys():
        if isinstance(secondDict[key],dict):
            numLeafs += getNumLeafs(secondDict[key])
        else:
            numLeafs += 1
    return numLeafs

def getTreeDepth(myTree):
    maxDepth = 0
    firstStr = list(myTree.keys())[0]
    secondDict = myTree[firstStr]
    for key in secondDict.keys():
        if isinstance(secondDict[key],dict):
            thisDepth = 1 + getTreeDepth(secondDict[key])
        else:
            thisDepth = 1
        if thisDepth > maxDepth: maxDepth = thisDepth
    return maxDepth
```


### 绘制决策树


```python
def plotMidText(cntrPt, parentPt, txtString):
    xMid = (parentPt[0]-cntrPt[0])/2.0 + cntrPt[0]
    yMid = (parentPt[1]-cntrPt[1])/2.0 + cntrPt[1]
    createPlot.ax1.text(xMid, yMid, txtString, va="center", ha="center", rotation=30)

def plotTree(myTree, parentPt, nodeTxt):#if the first key tells you what feat was split on
    numLeafs = getNumLeafs(myTree)  #this determines the x width of this tree
    depth = getTreeDepth(myTree)
    firstStr = list(myTree.keys())[0]     #the text label for this node should be this
    cntrPt = (plotTree.xOff + (1.0 + float(numLeafs))/2.0/plotTree.totalW, plotTree.yOff)
    plotMidText(cntrPt, parentPt, nodeTxt)
    plotNode(firstStr, cntrPt, parentPt, decisionNode)
    secondDict = myTree[firstStr]
    plotTree.yOff = plotTree.yOff - 1.0/plotTree.totalD
    for key in secondDict.keys():
        if type(secondDict[key]).__name__=='dict':#test to see if the nodes are dictonaires, if not they are leaf nodes   
            plotTree(secondDict[key],cntrPt,str(key))        #recursion
        else:   #it's a leaf node print the leaf node
            plotTree.xOff = plotTree.xOff + 1.0/plotTree.totalW
            plotNode(secondDict[key], (plotTree.xOff, plotTree.yOff), cntrPt, leafNode)
            plotMidText((plotTree.xOff, plotTree.yOff), cntrPt, str(key))
    plotTree.yOff = plotTree.yOff + 1.0/plotTree.totalD
#if you do get a dictonary you know it's a tree, and the first element will be another dict

def createPlot(inTree):
    fig = plt.figure(1, facecolor='white')
    fig.clf()
    axprops = dict(xticks=[], yticks=[])
    createPlot.ax1 = plt.subplot(111, frameon=False, **axprops)    #no ticks
    #createPlot.ax1 = plt.subplot(111, frameon=False) #ticks for demo puropses 
    plotTree.totalW = float(getNumLeafs(inTree))
    plotTree.totalD = float(getTreeDepth(inTree))
    plotTree.xOff = -0.5/plotTree.totalW; plotTree.yOff = 1.0;
    plotTree(inTree, (0.5,1.0), '')
    plt.show()
```

## 测试和存储分类器

### 使用决策树执行分类


```python
def classify(inputTree, featLabels, testVec):
    firstStr = list(inputTree.keys())[0]
    secondDict = inputTree[firstStr]
    featIndex = featLabels.index(firstStr)
    for key in secondDict.keys():
        if testVec[featIndex] == key:
            if isinstance(secondDict[key], dict):
                classLabel = classify(secondDict[key], featLabels, testVec)
            else:
                classLabel = secondDict[key]
    return classLabel
```

### 存储决策树


```python
def storeTree(inputTree, filename):
    import pickle
    with open(filename, 'wb') as f:
        pickle.dump(inputTree, f)

def grabTree(filename):
    import pickle
    with open(filename, 'rb') as f:
        return pickle.load(f)
```

## 使用决策树预测隐形眼镜类型

### 读取文件


```python
import pandas as pd
```


```python
def txt2csv(fileName, columns):
    # 打开文件按行读取
    with open(fileName, 'r') as f:
        data = f.readlines()
    # 将每一行数据间隔方式由'\t'改为','
    for index, line in enumerate(data):
        data[index] = line.replace('\t', ',')
    # 将列名插入数据头部
    content = ','.join(columns) + '\n'
    content += ''.join(data)
    # 存储修改后的数据为.csv文件
    with open(fileName.split('.')[0]+'.csv', 'w', encoding='utf-8') as f:
        f.write(content)
```


```python
txt2csv('data/lenses.txt', ['age','prescript','astigmatic','tearRate','label'])
```


```python
df = pd.read_csv('data/lenses.csv')
```


```python
lensesTree = createTree(df)
```

```python
plt.figure(figsize=(10,10))
createPlot(lensesTree)
```


![](/media/editor/output_44_1_20190326122306690649.png)