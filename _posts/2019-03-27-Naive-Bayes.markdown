---
layout:     post
title:      朴素贝叶斯
subtitle:   ""
date:       2019-03-27 09:19:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Machine Learning
    - Data Science
---
## 理论部分

### 基于贝叶斯决策理论的分类方法

**优点**：在数据量较少的情况下仍然有效，可以处理多类别问题

**缺点**：对于输入数据的准备方式较为敏感

**适用数据类型**：标称型数据

### 概率公式

条件概率公式：<img src="http://latex.codecogs.com/gif.latex?p%28A%7CB%29%20%3D%20%5Cfrac%7Bp%28AB%29%7D%7Bp%28B%29%7D">，计算事件B发生的前提下事件A发生的概率

贝叶斯准则：<img src="http://latex.codecogs.com/gif.latex?p%28c%7Cx%29%20%3D%20%5Cfrac%7Bp%28x%7Cc%29%20p%28c%29%7D%7Bp%28x%29%7D">，在已知<img src="http://latex.codecogs.com/gif.latex?p%28x%7Cc%29">的情况下计算<img src="http://latex.codecogs.com/gif.latex?p%28c%7Cx%29">

### 使用条件概率进行分类

假设要分类的点是(x, y)，现有两个分类<img src="http://latex.codecogs.com/gif.latex?c_1">和<img src="http://latex.codecogs.com/gif.latex?c_2">，通过条件概率对这个点进行分类即比较该点可能为每一个类别的概率，出现概率最大的即为该点所属分类，即比较<img src="http://latex.codecogs.com/gif.latex?p%28c_1%7Cx%2C%20y%29">和<img src="http://latex.codecogs.com/gif.latex?p%28c_2%7Cx%2C%20y%29">。

可以通过贝叶斯准则计算：<img src="http://latex.codecogs.com/gif.latex?p%28c_i%7Cx%2Cy%29%20%3D%20%5Cfrac%7Bp%28x%2Cy%7Cc_i%29%20p%28c_i%29%7D%7Bp%28x%2Cy%29%7D">

最终若<img src="http://latex.codecogs.com/gif.latex?p%28c_1%7Cx%2Cy%29%20%3E%20p%28c_2%7Cx%2Cy%29">，则(x,y)属于<img src="http://latex.codecogs.com/gif.latex?c_1">, 若<img src="http://latex.codecogs.com/gif.latex?p%28c_1%7Cx%2Cy%29%20%3C%20p%28c_2%7Cx%2Cy%29">，(x,y)属于<img src="http://latex.codecogs.com/gif.latex?c_2">

### 使用朴素贝叶斯进行文档分类

朴素贝叶斯之所以称为“朴素”，是因为其假设所有特征之间在统计学上是互相独立的（即每个特征的出现与其他特征无关）。虽然这个假设在现实生活中的大多数情况中看起来都不是很合适，但是朴素贝叶斯在实际中的表现却很好。

## 使用Python进行侮辱性文本分类

### 准备数据：从文档中构建词向量--词集模型

文本分类过程中使用的特征来自于**词条**（字符的任意组合），这里我们吧文本看成单词向量或词条向量（把句子转换为向量）。

具体步骤为：
1. 考虑出现在所有文档中的所有单词，将一些词纳入词汇表/词汇集合；
2. 将每一篇文章转换为词汇表上的向量


```python
import numpy as np
```


```python
def loadDataSet():
    postingList=[['my', 'dog', 'has', 'flea', 'problems', 'help', 'please'],
                 ['maybe', 'not', 'take', 'him', 'to', 'dog', 'park', 'stupid'],
                 ['my', 'dalmation', 'is', 'so', 'cute', 'I', 'love', 'him'],
                 ['stop', 'posting', 'stupid', 'worthless', 'garbage'],
                 ['mr', 'licks', 'ate', 'my', 'steak', 'how', 'to', 'stop', 'him'],
                 ['quit', 'buying', 'worthless', 'dog', 'food', 'stupid']]
    classVec = [0,1,0,1,0,1]    #1 is abusive, 0 not
    return postingList,classVec

def createVocabList(dataSet):
    vocabSet = set([])  # 创建一个空词汇集合
    for document in dataSet:
        vocabSet = vocabSet | set(document)  # 将文档中每一行的词汇集合与当前词汇集合作并集
    return list(vocabSet)

def words2Vec_set(vocabList, inputSet):
    result = np.zeros_like(vocabList, dtype='int')  # 创建一个等长于词汇集合的由0组成的表
    for word in inputSet:
        if word in vocabList:
            result[vocabList.index(word)] = 1  # 若单词出现，则在列表中对应位置将值置1
        else:
            print("the word %s is not in vocabulary" % word)
    return result
```

loadDataSet函数创建了一个数据集合用于测试，createVocabList函数根据数据集创建并返回了一个词汇集合，word2Vec函数根据已有的词汇集合以及输入集合，由单词是否出现构建0-1列表并返回

### 训练算法：从词向量计算概率

下面是朴素贝叶斯分类器训练函数

根据我们已有的数据修改贝叶斯准则：将x,y替换为**w**:  <img src="http://latex.codecogs.com/gif.latex?p%28c_i%7Cw%29%20%3D%20%5Cfrac%7Bp%28w%7Cc_i%29%20p%28c_i%29%7D%7Bp%28w%29%7D">

这里的**w**指的是一个向量，由多个数值组成。由朴素贝叶斯假设可知，<img src="http://latex.codecogs.com/gif.latex?%24p%28w%7Cc_i%29%20%3D%20%5Cprod%5En_%7Bj%3D1%7Dp%28w_j%7Cc_i%29%24">


```python
def trainNB(trainMatrix, trainCategory):
    """
    trainMatrix: 文档矩阵
    trainCategory: 文档类别标签
    """
    numTrainDocs = trainMatrix.shape[0]
    numWords = trainMatrix.shape[1]
    p_c = np.sum(trainCategory) / numTrainDocs  # 计算侮辱性文档在总文档中占比
    p_1 = np.ones(numWords)  # 统计侮辱性文档中单词出现次数
    p_0 = np.ones(numWords)  # 统计非侮辱性文档中单词出现次数
    
    for i in range(numTrainDocs):
        if trainCategory[i] == 1:
            p_1 += trainMatrix[i]
        else:
            p_0 += trainMatrix[i]
    
    # 计算p(wj|ci)
    p_1_vec = np.log(p_1 / (np.sum(p_1)+1))  # 计算侮辱性文档中单词出现的占比
    p_0_vec = np.log(p_0 / (np.sum(p_0)+1))  # 计算非侮辱性文档中单词出现的占比
    
    return p_0_vec, p_1_vec, p_c    
```

trainNB函数返回文档列表中：非侮辱性文档中各单词出现占比，侮辱性文档中各单词出现占比，侮辱性文档在总文档中占比

**注意：**
1. 在初始化单词出现次数的时候不是生成全0列表，而是全1列表的原因是：避免在之后将所有<img src="http://latex.codecogs.com/gif.latex?p%28w_j%7Cc_i%29">相乘时由于单一出现概率为0导致最终结果为0的情况
2. 在最终计算<img src="http://latex.codecogs.com/gif.latex?p%28w_j%7Cc_i%29">时在分母上+1是为了避免总出现次数为0而导致分母为0无法计算除法的情况
3. 在最终计算<img src="http://latex.codecogs.com/gif.latex?p%28w_j%7Cc_i%29">时将整体结果进行对数计算是为了防止多个小概率相乘导致计算结果不精确

至此，分类器就已经训练完成了，下面开始测试分类。


```python
def classifyNB(vec_to_classify, p_0_vec, p_1_vec, p_class_1):
    p1 = np.sum(vec_to_classify * p_1_vec) / p_class_1
    p0 = np.sum(vec_to_classify * p_0_vec) / p_class_1
    return 1 if p1 > p0 else 0
```

classifyNB函数根据贝叶斯准则计算文档属于各分类的概率，进行比较后返回比较结果。

函数中仅计算了<img src="http://latex.codecogs.com/gif.latex?p%28w%7Cc_i%29p%28c_i%29">而没有计算<img src="http://latex.codecogs.com/gif.latex?%5Cfrac%7Bp%28w%7Cc_i%29p%28c_i%29%7D%7Bp%28w%29%7D">的原因是要比较的概率都有相同的<img src="http://latex.codecogs.com/gif.latex?p%28w%29">，此时只用比较分子即可。


```python
def testingNB():
    posts, class_list = loadDataSet()  # 获取数据集
    vocab_list = createVocabList(posts)  # 获取词汇集合
    # 根据词汇集合中的单词是否在文档中出现，将文档列表转化为0-1矩阵
    trainMat = np.zeros((len(posts), len(vocab_list)))
    for index,post in enumerate(posts):
        trainMat[index] = words2Vec_set(vocab_list, post)
    # 获取训练集中的信息
    p0_vec, p1_vec, pAbusive = trainNB(trainMat, np.array(class_list))
    
    # 准备测试集1
    test = ['love', 'my', 'dalmation']
    test_doc = np.array(words2Vec_set(vocab_list, test))
    # 分类
    print("'"+' '.join(test)+"'","classified as :", classifyNB(test_doc, p0_vec, p1_vec, pAbusive))
    # 准备测试集2
    test = ['stupid','garbage']
    test_doc = np.array(words2Vec_set(vocab_list, test))
    # 分类
    print("'"+' '.join(test)+"'","classified as :", classifyNB(test_doc, p0_vec, p1_vec, pAbusive))
```


```python
testingNB()
```

    'love my dalmation' classified as : 0
    'stupid garbage' classified as : 1
    

testingNB函数中封装了一次完整的文档分类流程：准备训练集，训练分类器，准备测试集，进行分类

### 准备数据：文档词袋模型

之前使用的**词集模型**将每个词的出现与否作为一个特征，如果某个词在文档中出现不止一次，则意味着该词出现的次数可能包含了一些该词出现与否所没有的隐含信息，这就是词袋模型。


```python
def words2Vec_bag(vocabList, inputSet):
    result = np.zeros_like(len(vocabList))
    for word in inputSet:
        if word in vocabList:
            result[vocabList.index(word)] += 1
    return result
```

words2Vec_bag函数中将原来的```result[vocabList.index(word)] = 1```改为了```result[vocabList.index(word)] += 1```

## 使用朴素贝叶斯过滤垃圾邮件

### 文本解析


```python
def textParse(text):
    import re
    result = re.split(r'\W*', text)
    return [_.lower() for _ in result if len(_)>2]
```

textParse函数将一个字符串文本拆分为单词列表，并将所有大写字母转换为小写，为了删除一些没有什么作用的单词，只返回拆分结果中长度大于2的单词。

### 垃圾邮件测试


```python
def test_garbage_email():
    doc_list = []
    class_list = []
    full_text = []
    # 读取文件
    import os
    ## 读取正常邮件
    base_dir_spam = 'data/email/spam'
    for file in os.listdir(base_dir_spam):
        word_list = textParse(open(base_dir_spam+'/'+file,'r').read())  # 拆分文档内容
        doc_list.append(word_list)
        full_text.extend(word_list)
        class_list.append(0)
    ## 读取垃圾邮件
    base_dir_ham = 'data/email/ham'
    for file in os.listdir(base_dir_ham):
        """fileName = base_dir_ham+'/'+file
        print(fileName)
        with open(fileName, '') as f:
            content = f.read()"""
        word_list = textParse(open(base_dir_ham+'/'+file,'r').read())  # 查分文档内容
        doc_list.append(word_list)
        full_text.extend(word_list)
        class_list.append(1)
    
    # 训练分类器
    vocab_list = createVocabList(doc_list)
    train_test_ratio = 0.8  # 训练测试集比例
    training_set = []  # 训练集
    train_classes = [] # 训练集标签
    #train_range = range(len(doc))
    ## 随机构建训练集
    for i in range(int(train_test_ratio*len(doc_list))):
        rand_index = np.random.randint(0, len(doc_list))
        training_set.append(doc_list.pop(rand_index))
        train_classes.append(class_list.pop(rand_index))
    ## 构建测试集
    test_set = []  # 测试集
    test_classes = []  # 测试集标签
    test_set = doc_list.copy()
    test_classes = class_list.copy()
    ## 将文档数据转化为矩阵
    train_mat = []
    for content in training_set:
        train_mat.append(words2Vec_set(vocab_list, content))
    ## 训练分类器
    p0_vec, p1_vec, pSpam = trainNB(np.array(train_mat), np.array(train_classes))
    
    # 测试分类器
    error_count = 0
    for index, content in enumerate(test_set):
        word_vec = words2Vec_set(vocab_list, content)
        result = classifyNB(np.array(word_vec), p0_vec, p1_vec, pSpam)
        if result != test_classes[index]:
            error_count += 1
    print("the error rate is %f" % (error_count/len(test_set)))
```


```python
test_garbage_email()
```
    the error rate is 0.000000


## 使用朴素贝叶斯分类器从个人广告中获取区域倾向


```python
def calcMostFreq(vocabList, fullText):
    """
    统计单词出现次数
    vocabList: 词汇集合
    fullText: 所有文本
    """
    freqDict = {}
    for word in vocabList:
        freqDict[word] = fullText.count(word)
    freqDict = sorted(freqDict.items(), key=lambda x: x[1], reverse=True)  # 将单词出现次数按从大到小排列
    return freqDict[:30]  # 返回出现次数最多的30个单词
```


```python
def localWords(feed1,feed0):
    import feedparser
    docList=[]; classList = []; fullText =[]
    minLen = min(len(feed1['entries']),len(feed0['entries']))  # 获取数据集长度
    # 数据处理
    for i in range(minLen):
        wordList = textParse(feed1['entries'][i]['summary'])
        docList.append(wordList)
        fullText.extend(wordList)
        classList.append(1)  # 将纽约设置为类别1
        wordList = textParse(feed0['entries'][i]['summary'])
        docList.append(wordList)
        fullText.extend(wordList)
        classList.append(0)  # 将旧金山设置为类别0
    vocabList = createVocabList(docList)
    
    # 去除出现次数最多的30个词语
    top30Words = calcMostFreq(vocabList,fullText)
    for pairW in top30Words:
        if pairW[0] in vocabList: vocabList.remove(pairW[0])
    
    # 构建测试集
    trainingSet = range(2*minLen); testSet=[]
    for i in range(20):
        randIndex = np.random.randint(0,len(trainingSet))
        testSet.append(trainingSet[randIndex])
        del(trainingSet[randIndex])
    # 构建训练集
    trainMat=[]; trainClasses = []
    for docIndex in trainingSet:
        trainMat.append(words2Vec_bag(vocabList, docList[docIndex]))
        trainClasses.append(classList[docIndex])
    p0V,p1V,pSpam = trainNB(np.array(trainMat),np.array(trainClasses))
    
    # 分类测试
    errorCount = 0
    for docIndex in testSet:
        wordVector = words2Vec_bag(vocabList, docList[docIndex])
        if classifyNB(np.array(wordVector),p0V,p1V,pSpam) != classList[docIndex]:
            errorCount += 1
    print('the error rate is: ', errorCount/len(testSet))
    return vocabList,p0V,p1V
```