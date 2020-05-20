---
layout:     post
title:      通过竞赛从零入门CV
subtitle:   ""
date:       2020-05-20 21:26:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Deep Learning
    - Computer Vision
---

> 实践是学习知识最好的方法，当初学习机器学习的时候看了很多书，都不如最后参加一场数据挖掘比赛收获的多。这次，就从街景字符识别的赛题入手，学习入门CV~



## 1. 赛题理解

> 阶段学习目标：
>
> - 理解赛题背景和赛题数据
> - 完成赛题报名和数据下载，理解赛题的解题思路

**赛题地址：https://tianchi.aliyun.com/competition/entrance/531795/information**

### 1.1 赛题数据

赛题以街道字符为为赛题数据，训练集数据包括3W张照片，验证集数据包括1W张照片，每张照片包括颜色图像和对应的编码类别和具体位置，测试集A包括4W张照片，测试集B包括4W张照片

![检测](/img/in-post/2020-05-20-learning-CV/Task01/detect.png)

### 1.2 数据标签

对于训练数据每张图片将给出对于的编码标签，和具体的字符框的位置（训练集、验证集都给出字符位置），可用于模型训练：

| Field  | Description |
| ------ | ----------- |
| top    | 左上角坐标X |
| height | 字符高度    |
| left   | 左上角最表Y |
| width  | 字符宽度    |
| label  | 字符编码    |

![坐标](/img/in-post/2020-05-20-learning-CV/Task01/coordinate.png)

在比赛数据（训练集和验证集）中，同一张图片中可能包括一个或者多个字符，因此在比赛数据的JSON标注中，会有多个字符的边框信息：

| 图片                                                         | 标注信息                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![19](/img/in-post/2020-05-20-learning-CV/Task01/raw_image.png) | ![标注](/img/in-post/2020-05-20-learning-CV/Task01/label.png) |

### 1.3 评测指标

这道赛题的评价指标比较简单：
$$
Score = \frac{编码识别正确的数量}{测试集图片数量}
$$


了解完了赛题的一些基本信息，就可以开始探索数据了。

### 1.4 读取数据

读取JSON中的标签的代码如下：

```python
import json
train_json = json.load(open('../input/train.json'))

# 数据标注处理
def parse_json(d):
    arr = np.array([
        d['top'], d['height'], d['left'],  d['width'], d['label']
    ])
    arr = arr.astype(int)
    return arr

img = cv2.imread('../input/train/000000.png')
arr = parse_json(train_json['000000.png'])

plt.figure(figsize=(10, 10))
plt.subplot(1, arr.shape[1]+1, 1)
plt.imshow(img)
plt.xticks([]); plt.yticks([])

for idx in range(arr.shape[1]):
    plt.subplot(1, arr.shape[1]+1, idx+2)
    plt.imshow(img[arr[0, idx]:arr[0, idx]+arr[1, idx],arr[2, idx]:arr[2, idx]+arr[3, idx]])
    plt.title(arr[4, idx])
    plt.xticks([]); plt.yticks([])
```



### 1.5 解题思路

赛题本质是分类问题，需要对图片的字符进行识别。但赛题给定的数据图片中不同图片中包含的字符数量不等，因此本次赛题的难点是需要对不定长的字符进行识别，与传统的图像分类任务有所不同。

**思路一**：将赛题抽象为一个定长字符识别问题，在赛题数据集中大部分图像中字符个数为2-4个，最多的字符    个数为6个。因此可以对于所有的图像都抽象为6个字符的识别问题，字符23填充为23XXXX，字符231填充为231XXX。经过填充之后，原始的赛题可以简化了6个字符的分类问题。在每个字符的分类中会进行11个类别的分类，假如分类为填充字符，则表明该字符为空。

**思路二**：有特定的方法来解决此种不定长的字符识别问题，比较典型的有CRNN字符识别模型。

**思路三**：在赛题数据中已经给出了训练集、验证集中所有图片中字符的位置，因此可以首先将字符的位置进行识别，利用物体检测的思路完成。此种思路需要构建字符检测模型，对测试集中的字符进行识别，可以参考物体检测模型SSD或者YOLO来完成。