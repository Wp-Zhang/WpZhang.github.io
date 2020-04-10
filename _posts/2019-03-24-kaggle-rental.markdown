---
layout:     post
title:      kaggle-Rental Listing Inquiries-数据探索
subtitle:   ""
date:       2019-03-24 13:33:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Data Science
    - Visualization
---
### 导入相关库


```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
```


```python
%matplotlib inline
```

### 读取数据


```python
base_dir = 'data/'
train_df = pd.read_json(base_dir+'train.json')
test_df = pd.read_json(base_dir+'test.json')
```

### 数据探索

#### 查看基本信息

#####  看一下表格


```python
train_df.head()
```
略



#####  查看列名，也就是基本特征


```python
train_df.columns
test_df.columns
```
    Index(['bathrooms', 'bedrooms', 'building_id', 'created', 'description',
           'display_address', 'features', 'latitude', 'listing_id', 'longitude',
           'manager_id', 'photos', 'price', 'street_address', 'interest_level'],
          dtype='object')

    Index(['bathrooms', 'bedrooms', 'building_id', 'created', 'description',
           'display_address', 'features', 'latitude', 'listing_id', 'longitude',
           'manager_id', 'photos', 'price', 'street_address'],
          dtype='object')



##### 查看数据集的基本信息，观察每一列的数据类型


```python
train_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 49352 entries, 10 to 99994
    Data columns (total 15 columns):
    bathrooms          49352 non-null float64
    bedrooms           49352 non-null int64
    building_id        49352 non-null object
    created            49352 non-null object
    description        49352 non-null object
    display_address    49352 non-null object
    features           49352 non-null object
    latitude           49352 non-null float64
    listing_id         49352 non-null int64
    longitude          49352 non-null float64
    manager_id         49352 non-null object
    photos             49352 non-null object
    price              49352 non-null int64
    street_address     49352 non-null object
    interest_level     49352 non-null object
    dtypes: float64(3), int64(3), object(9)
    memory usage: 6.0+ MB
    

##### 查看缺失值


```python
train_df.isna().sum()
```
    bathrooms          0
    bedrooms           0
    building_id        0
    created            0
    description        0
    display_address    0
    features           0
    latitude           0
    listing_id         0
    longitude          0
    manager_id         0
    photos             0
    price              0
    street_address     0
    interest_level     0
    dtype: int64



看来没有列含有缺失值，就不用进行缺失值填补了。

####  观察特征分布及特征之间的关系

##### 首先观察预测目标：'interest_level' 的分布


```python
sns.countplot(train_df['interest_level'], order=['low', 'medium', 'high'])
```

    <matplotlib.axes._subplots.AxesSubplot at 0x1c98fd5c978>

![](/media/editor/output_18_1_20190324213620668391.png)


可见客户对大多数房屋的兴趣由低到高递减，这也符合常识，毕竟优质房源只占少数。

然后对分类型数据进行编码便于之后的预测工作


```python
# 方案一
#train_df['interest'] = 0
#train_df['interest'][train_df['interest_level']=='medium'] = 1
#train_df['interest'][train_df['interest_level']=='high'] = 2

# 方案二
transfer_map = {'high':2, 'medium':1, 'low':0}
parse = lambda label:transfer_map[label]
train_df['interest'] = train_df['interest_level'].apply(parse)
```

**然后根据顺序依次分析特征**

#####  卫生间数量


```python
sns.countplot(train_df['bathrooms'])
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c98fdbb208>




![](/media/editor/output_24_1_20190324213636426138.png)


为什么会有小数数量的卫生间呢？？？？后面做特征工程的时候可以考虑处理一下。

查看卫生间数量和客户兴趣直接的关系


```python
order = ['low','medium','high']
sns.stripplot(train_df['interest_level'], train_df['bathrooms'], jitter=True,order=order)
```
    <matplotlib.axes._subplots.AxesSubplot at 0x1c9902c0a58>

![](/media/editor/output_27_1_20190324213822403513.png)


可见：
1. 客户更偏爱卫生间数在1-4之间的房屋；
2. 很少人喜欢没有卫生的房屋；
3. 卫生间数量大于4的房屋很少

于是再统计超出4个卫生间的房屋数


```python
(train_df['bathrooms']>4).sum()
```




    61



只有61个，而测试数据总量为上万个，可以将其视为离群点，其很有可能是噪声并且干扰之后的预测，于是对其进行处理。


```python
train_df['bathrooms'][train_df['bathrooms']>4] = 4
```

    D:\Anaconda3\lib\site-packages\ipykernel_launcher.py:1: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      """Entry point for launching an IPython kernel.
    

##### 卧室数量


```python
sns.countplot(train_df['bedrooms'])
```

    <matplotlib.axes._subplots.AxesSubplot at 0x1c990305240>
![](/media/editor/output_33_1_20190324213844539591.png)

可见大部分卧室数量在0-3之间

再观察其与客户兴趣的关系


```python
order = ['low', 'medium', 'high']
sns.stripplot(train_df['interest_level'], train_df['bedrooms'], jitter=True, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c990376860>




![](/media/editor/output_35_1_20190324213933598126.png)



```python
sns.countplot(x='bedrooms', hue='interest_level', data=train_df)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c9903c6240>

![](/media/editor/output_36_1_20190324213958872530.png)


观察发现客户更偏爱0-2个卧室的房屋（没有卧室你租房子干嘛？？？），但是客户偏好受卧室数量的影响很小。

##### 价格  

首先查看一下价格分布


```python
plt.plot(range(train_df.shape[0]), train_df['price'], 'co')
plt.ylabel="Price"
plt.show()
```




    [<matplotlib.lines.Line2D at 0x1c9905e6710>]

![](/media/editor/output_40_1_20190324214018688403.png)


明显有些离群点，进行之前同样的处理方法。


```python
split = np.percentile(train_df['price'], (99), interpolation='midpoint')
train_df['price'][train_df['price']>split] = split
```

    D:\Anaconda3\lib\site-packages\ipykernel_launcher.py:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      
    

现在再来看一下价格的分布


```python
sns.distplot(train_df['price'], bins=50, kde=True)
```

    D:\Anaconda3\lib\site-packages\matplotlib\axes\_axes.py:6462: UserWarning: The 'normed' kwarg is deprecated, and has been replaced by the 'density' kwarg.
      warnings.warn("The 'normed' kwarg is deprecated, and has been "
    




    <matplotlib.axes._subplots.AxesSubplot at 0x1c990633f28>

![](/media/editor/output_44_2_20190324214037152449.png)


可见分布有点偏离，使用log来处理一下


```python
sns.distplot(np.log1p(train_df['price']))
```

    D:\Anaconda3\lib\site-packages\matplotlib\axes\_axes.py:6462: UserWarning: The 'normed' kwarg is deprecated, and has been replaced by the 'density' kwarg.
      warnings.warn("The 'normed' kwarg is deprecated, and has been "
    




    <matplotlib.axes._subplots.AxesSubplot at 0x1c9906bc828>

![](/media/editor/output_46_2_20190324214053648426.png)


再看价格和客户兴趣直接的关系


```python
order = ['low', 'medium', 'high']
sns.stripplot(train_df['interest_level'], train_df['price'], jitter=True, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c99061bf28>

![](/media/editor/output_48_1_20190324214116906482.png)


很明显，大家都喜欢价格低一点的房子，但是medium和high分布不是很均匀，于是用小提琴图观察更加清晰


```python
order = ['low', 'medium', 'high']
sns.violinplot(x='interest_level', y='price', data=train_df, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c990800550>

![](/media/editor/output_50_1_20190324214140257942.png)


##### listing_id

观察其与客户兴趣的关系


```python
order = ['low', 'medium', 'high']
sns.stripplot(train_df['interest_level'], train_df['listing_id'], jitter=True, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c99048f860>

![](/media/editor/output_53_1_20190324214210359561.png)


##### 地理位置

根据经纬度来观察客户偏好的房屋的分布


```python
sns.lmplot(
    x='longitude',
    y='latitude',
    fit_reg=False,
    hue='interest_level',
    hue_order=['low', 'medium', 'high'],
    size=9,
    scatter_kws={'alpha':0.3, 's':30},
    data = train_df
)
```




    <seaborn.axisgrid.FacetGrid at 0x1c9904c0160>

![](/media/editor/output_56_1_20190324214232829533.png)


离群点影响了观察，去除后重新作图


```python
split1 = np.percentile(train_df['longitude'], (0.5, 99.5), interpolation='midpoint')
split2 = np.percentile(train_df['latitude'], (0.5, 99.5), interpolation='midpoint')
data = train_df[
    (train_df['longitude']>split1[0]) &
    (train_df['longitude']<split1[1]) &
    (train_df['latitude']>split2[0]) &
    (train_df['latitude']<split2[1])
]
sns.lmplot(
    x='longitude',
    y='latitude',
    fit_reg=False,
    hue='interest_level',
    hue_order=['low', 'medium', 'high'],
    size=9,
    scatter_kws={'alpha':0.3, 's':30},
    data = data
)
```




    <seaborn.axisgrid.FacetGrid at 0x1c9907ef0f0>

![](/media/editor/output_58_1_20190324214343441531.png)


下面的代码将坐标信息保存为.gpx文件，使用Google Earth打开可以查看实景地图

import gpxpy as gpx
import gpxpy.gpx

gpx = gpxpy.gpx.GPX()

for index, row in train_df.iterrows():
    #print (row['latitude'], row['longitude'])

    if row['interest_level'] == 'high': #opting for all nominals results in poor performance of Google Earth
        gps_waypoint = gpxpy.gpx.GPXWaypoint(row['latitude'],row['longitude'],elevation=10)
        gpx.waypoints.append(gps_waypoint)
        
filename = "GoogleEarth.gpx"
FILE = open(filename,"w")
FILE.writelines(gpx.to_xml())
FILE.close()

##### Display_address

统计一下各个地理位置出现的次数


```python
count = train_df.groupby('display_address')['display_address'].count()
plt.hist(count.values, bins=100, log=True, alpha=0.9)
plt.show()
#sns.distplot(cnt_srs.values, bins=100)
```




    (array([7.496e+03, 4.120e+02, 2.470e+02, 1.330e+02, 8.800e+01, 8.000e+01,
            4.100e+01, 3.800e+01, 4.000e+01, 2.400e+01, 2.700e+01, 1.400e+01,
            1.400e+01, 1.900e+01, 1.500e+01, 1.600e+01, 1.200e+01, 1.100e+01,
            1.000e+01, 5.000e+00, 1.000e+01, 5.000e+00, 4.000e+00, 6.000e+00,
            6.000e+00, 3.000e+00, 4.000e+00, 2.000e+00, 0.000e+00, 1.000e+00,
            3.000e+00, 3.000e+00, 5.000e+00, 2.000e+00, 1.000e+00, 1.000e+00,
            0.000e+00, 4.000e+00, 1.000e+00, 1.000e+00, 4.000e+00, 1.000e+00,
            0.000e+00, 3.000e+00, 1.000e+00, 1.000e+00, 0.000e+00, 0.000e+00,
            2.000e+00, 0.000e+00, 0.000e+00, 1.000e+00, 0.000e+00, 0.000e+00,
            2.000e+00, 1.000e+00, 0.000e+00, 0.000e+00, 1.000e+00, 0.000e+00,
            0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 1.000e+00,
            0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00,
            0.000e+00, 0.000e+00, 0.000e+00, 1.000e+00, 0.000e+00, 0.000e+00,
            0.000e+00, 1.000e+00, 0.000e+00, 1.000e+00, 0.000e+00, 0.000e+00,
            0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00,
            0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00, 0.000e+00,
            0.000e+00, 0.000e+00, 0.000e+00, 1.000e+00]),
     array([  1.  ,   5.37,   9.74,  14.11,  18.48,  22.85,  27.22,  31.59,
             35.96,  40.33,  44.7 ,  49.07,  53.44,  57.81,  62.18,  66.55,
             70.92,  75.29,  79.66,  84.03,  88.4 ,  92.77,  97.14, 101.51,
            105.88, 110.25, 114.62, 118.99, 123.36, 127.73, 132.1 , 136.47,
            140.84, 145.21, 149.58, 153.95, 158.32, 162.69, 167.06, 171.43,
            175.8 , 180.17, 184.54, 188.91, 193.28, 197.65, 202.02, 206.39,
            210.76, 215.13, 219.5 , 223.87, 228.24, 232.61, 236.98, 241.35,
            245.72, 250.09, 254.46, 258.83, 263.2 , 267.57, 271.94, 276.31,
            280.68, 285.05, 289.42, 293.79, 298.16, 302.53, 306.9 , 311.27,
            315.64, 320.01, 324.38, 328.75, 333.12, 337.49, 341.86, 346.23,
            350.6 , 354.97, 359.34, 363.71, 368.08, 372.45, 376.82, 381.19,
            385.56, 389.93, 394.3 , 398.67, 403.04, 407.41, 411.78, 416.15,
            420.52, 424.89, 429.26, 433.63, 438.  ]),
     <a list of 100 Patch objects>)

![](/media/editor/output_63_1_20190324215005730249.png)


观察地理位置出现次数与客户偏好的关系


```python
counter =  lambda address: count[address]
tmp = train_df[['display_address', 'interest_level']]
tmp['display_address'] = tmp['display_address'].apply(counter)
```

    D:\Anaconda3\lib\site-packages\ipykernel_launcher.py:3: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      This is separate from the ipykernel package so we can avoid doing imports until
    


```python
order = ['low', 'medium', 'high']
#sns.stripplot(tmp['interest_level'], tmp['display_address'], jitter=True, order = order)
sns.violinplot('interest_level', 'display_address', data=tmp, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c99057d828>

![](/media/editor/output_66_1_20190324214430934020.png)


#####  照片数量


```python
train_df['num_photos'] = train_df['photos'].apply(len)
split = np.percentile(train_df['num_photos'], (99))
train_df['num_photos'][train_df['num_photos']>split] = split

sns.countplot(train_df['num_photos'])
```

    D:\Anaconda3\lib\site-packages\ipykernel_launcher.py:3: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      This is separate from the ipykernel package so we can avoid doing imports until
    




    <matplotlib.axes._subplots.AxesSubplot at 0x1c99103ccf8>

![](/media/editor/output_68_2_20190324214514308895.png)



```python
order = ['low', 'medium', 'high']
sns.violinplot('num_photos', 'interest_level', data=train_df, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c99057ed68>

![](/media/editor/output_69_1_20190324214548897829.png)


##### 特点（features）

统计房屋列出的特点的条数与客户兴趣之间的关系


```python
train_df['len_features'] = train_df['features'].apply(len)
sns.countplot(train_df['len_features'])
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c990add518>

![](/media/editor/output_72_1_20190324214622422846.png)



```python
order = ['low', 'medium', 'high']
sns.violinplot('len_features', 'interest_level', data=train_df, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c990c76c18>

![](/media/editor/output_73_1_20190324214650475354.png)


##### 描述

首先统计描述长度，并观察它与客户兴趣之间的关系


```python
train_df['len_description'] = train_df['description'].apply(len)
```


```python
order = ['low', 'medium', 'high']
sns.stripplot(train_df['interest_level'], train_df['len_description'], jitter=True, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c99103f198>

![](/media/editor/output_77_1_20190324214705403563.png)



```python
order = ['low', 'medium', 'high']
sns.violinplot('len_description', 'interest_level', data=train_df, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c990abf208>

![](/media/editor/output_78_1_20190324214718985928.png)


然后统计描述单词数与客户兴趣之间的关系


```python
count =  lambda description: len(description.split())
train_df['num_description_words'] = train_df['description'].apply(count)
```


```python
order = ['low', 'medium', 'high']
sns.stripplot(train_df['interest_level'], train_df['num_description_words'], jitter=True, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c990b46c88>

![](/media/editor/output_81_1_20190324214736268655.png)



```python
order = ['low', 'medium', 'high']
sns.violinplot('num_description_words', 'interest_level', data=train_df, order=order)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c990b50898>

![](/media/editor/output_82_1_20190324214751401692.png)


查看出现次数前15的用词


```python
words = train_df['description'].apply(lambda x: x.split())
```


```python
l = []
for word_list in words:
    l += word_list
word_dict = {}
for word in l:
    if word_dict.get(word):
        word_dict[word] += 1
    else:
        word_dict[word] = 1
```


```python
sorted(word_dict.items(), key=lambda x:x[1])[-20:]
```

    [('kitchen', 15995),
     ('from', 16065),
     ('building', 16269),
     ('/><p><a', 16589),
     ('The', 17098),
     ('this', 18843),
     ('on', 19408),
     ('or', 22454),
     ('apartment', 24377),
     ('for', 26662),
     ('website_redacted', 35409),
     ('is', 42832),
     ('of', 53380),
     ('in', 57049),
     ('with', 59600),
     ('to', 63061),
     ('a', 83595),
     ('the', 85583),
     ('/><br', 115420),
     ('and', 135368)]