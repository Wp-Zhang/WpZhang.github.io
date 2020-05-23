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



## 2.数据读取与数据扩增

> 阶段学习目标：
>
> - 学习Python和Pytorch中的图像读取
> - 学会基本的图像扩增方法，使用Pytorch读取赛题数据

### 2.1 图像读取

Python中有很多库可以完成图像读取的操作，比较常见的有Pillow和OpenCV。

1. **Pillow**

```python
from PIL import Image
im = Image.open('test.jpg')
```

2. **OpenCV**

```python
import cv2 as cv
im = cv.imread('test.jpg')
```

### 2.2 数据扩增方法

在深度学习中数据扩增（Data Augmentation）非常重要，数据扩增可以增加训练集的样本，同时也可以有效缓解模型过拟合的情况，也可以给模型带来的更强的泛化能力。

在常见的数据扩增方法中，一般会从图像颜色、尺寸、形态、空间和像素等角度进行变换。当然不同的数据扩增方法可以自由进行组合，得到更加丰富的数据扩增方法。         

**以`torchvision`为例，常见的数据扩增方法包括**：

- `transforms.CenterCrop`      对图片中心进行裁剪      
- `transforms.ColorJitter `     对图像颜色的对比度、饱和度和零度进行变换      
- `transforms.FiveCrop`     对图像四个角和中心进行裁剪得到五分图像     
- `transforms.Grayscale`      对图像进行灰度变换    
- `transforms.Pad`        使用固定值进行像素填充     
- `transforms.RandomAffine`      随机仿射变换    
- `transforms.RandomCrop`      随机区域裁剪     
- `transforms.RandomHorizontalFlip`      随机水平翻转     
- `transforms.RandomRotation`     随机旋转     
- `transforms.RandomVerticalFlip`     随机垂直翻转

![](\img\in-post\2020-05-20-learning-CV\Task02\data_augmentation.png)

**常用的数据扩增库:**

- `torchvision`

  https://github.com/pytorch/vision      

  pytorch官方提供的数据扩增库，提供了基本的数据数据扩增方法，可以无缝与torch进行集成；但数据扩增方法种类较少，且速度中等；       

- `imgaug`

  https://github.com/aleju/imgaug

  是常用的第三方数据扩增库，提供了多样的数据扩增方法，且组合起来非常方便，速度较快；      

- `albumentations`

  https://albumentations.readthedocs.io     

  是常用的第三方数据扩增库，提供了多样的数据扩增方法，对图像分类、语义分割、物体检测和关键点检测都支持，速度较快。      

### 2.3 Pytorch读取数据  

在Pytorch中数据是通过Dataset进行封装，并通过DataLoder进行并行读取。所以我们只需要重载一下数据读取的逻辑就可以完成数据的读取。

```python
import os, sys, glob, shutil, json
import cv2

from PIL import Image
import numpy as np

import torch
from torch.utils.data.dataset import Dataset
import torchvision.transforms as transforms

class SVHNDataset(Dataset):
    def __init__(self, img_path, img_label, transform=None):
        self.img_path = img_path
        self.img_label = img_label 
        if transform is not None:
            self.transform = transform
        else:
            self.transform = None

    def __getitem__(self, index):
        img = Image.open(self.img_path[index]).convert('RGB')

        if self.transform is not None:
            img = self.transform(img)
        
        # 原始SVHN中类别10为数字0
        lbl = np.array(self.img_label[index], dtype=np.int)
        lbl = list(lbl)  + (5 - len(lbl)) * [10]
        
        return img, torch.from_numpy(np.array(lbl[:5]))

    def __len__(self):
        return len(self.img_path)

train_path = glob.glob('../input/train/*.png')
train_path.sort()
train_json = json.load(open('../input/train.json'))
train_label = [train_json[x]['label'] for x in train_json]

data = SVHNDataset(train_path, train_label,
          transforms.Compose([
              # 缩放到固定尺寸
              transforms.Resize((64, 128)),

              # 随机颜色变换
              transforms.ColorJitter(0.2, 0.2, 0.2),

              # 加入随机旋转
              transforms.RandomRotation(5),

              # 将图片转换为pytorch 的tesntor
              # transforms.ToTensor(),

              # 对图像像素进行归一化
              # transforms.Normalize([0.485,0.456,0.406],[0.229,0.224,0.225])
            ]))
```

通过上述代码，可以将赛题的图像数据和对应标签进行读取，在读取过程中的进行数据扩增，效果如下所示：       

| 1                                                            | 2                                                            | 3                                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![IMG](\img\in-post\2020-05-20-learning-CV\Task02\23.png)    | ![IMG](\img\in-post\2020-05-20-learning-CV\Task02\23_1.png)  | ![IMG](\img\in-post\2020-05-20-learning-CV\Task02\23_2.png)  |
| ![IMG](\img\in-post\2020-05-20-learning-CV\Task02\144_1.png) | ![IMG](\img\in-post\2020-05-20-learning-CV\Task02\144_2.png) | ![IMG](\img\in-post\2020-05-20-learning-CV\Task02\144_3.png) |

接下来我们将在定义好的Dataset基础上构建DataLoder，你可以会问有了Dataset为什么还要有DataLoder？其实这两个是两个不同的概念，是为了实现不同的功能。                 

- Dataset：对数据集的封装，提供索引方式的对数据样本进行读取      
- DataLoder：对Dataset进行封装，提供批量读取的迭代读取    
       

 加入DataLoder后，数据读取代码改为如下：      

```python
import os, sys, glob, shutil, json
import cv2

from PIL import Image
import numpy as np

import torch
from torch.utils.data.dataset import Dataset
import torchvision.transforms as transforms

class SVHNDataset(Dataset):
    def __init__(self, img_path, img_label, transform=None):
        self.img_path = img_path
        self.img_label = img_label 
        if transform is not None:
            self.transform = transform
        else:
            self.transform = None

    def __getitem__(self, index):
        img = Image.open(self.img_path[index]).convert('RGB')

        if self.transform is not None:
            img = self.transform(img)
        
        # 原始SVHN中类别10为数字0
        lbl = np.array(self.img_label[index], dtype=np.int)
        lbl = list(lbl)  + (5 - len(lbl)) * [10]
        
        return img, torch.from_numpy(np.array(lbl[:5]))

    def __len__(self):
        return len(self.img_path)

train_path = glob.glob('../input/train/*.png')
train_path.sort()
train_json = json.load(open('../input/train.json'))
train_label = [train_json[x]['label'] for x in train_json]

train_loader = torch.utils.data.DataLoader(
        SVHNDataset(train_path, train_label,
                   transforms.Compose([
                       transforms.Resize((64, 128)),
                       transforms.ColorJitter(0.3, 0.3, 0.2),
                       transforms.RandomRotation(5),
                       transforms.ToTensor(),
                       transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225])
            ])), 
    batch_size=10, # 每批样本个数
    shuffle=False, # 是否打乱顺序
    num_workers=10, # 读取的线程个数
)

for data in train_loader:
    break
```

在加入DataLoder后，数据按照批次获取，每批次调用Dataset读取单个样本进行拼接。此时data的格式为：

``` torch.Size([10, 3, 64, 128]), torch.Size([10, 6]) ```        

前者为图像文件，为batchsize * chanel * height * width次序；后者为字符标签。

