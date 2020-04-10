---
layout:     post
title:      Scrapy框架基础
subtitle:   ""
date:       2018-08-21 15:11:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Crawler
---
### cmd操作指令



commands | meaning

---|---

bench | Run quick benchmark test

fetch  | Fetch a URL using the Scrapy downloader

  genspider  |   Generate new spider using pre-defined templates

  runspider  |   Run a self-contained spider (without creating a project)

  settings   |   Get settings values

  shell      |   Interactive scraping console

  startproject | Create new project

  version   |    Print Scrapy version

  view      |    Open URL in browser, as seen by Scrapy





#### fetch: 

```python

scrapy fetch http://baidu.com -nolog

#不显示爬取日志，，直接显示爬取结果

```

#### runspider：

spider.py:

```python

from scrapy.spiders import Spider

class FirstSpider(Spider):

    #设置爬虫名字

    name = "first"

    #控制允许爬取的域名

    allowed_domains = ["baiud.com"]

    

    #起始域名

    start_urls = ["http://www.baidu.com",]

    

    #回调方法--运行后调用该方法

    def parse(self,response):   #response为爬取后的响应

        pass



scrapy runspider spider.py

```

#### shell(不启动scrapy爬虫的情况下测试网站):

```scrapy shell http://www.baiud.com```



#### startproject:

```scrapy startproject 爬虫项目名```



#### version:

```scrapy version```



#### view(下载并用浏览器查看网页)：

```scrapy view http://www.baiud.com```



#### bench(测试本地硬件性能):

创建一个本地服务器，测试最大爬取网页速度

```scrapy bentch```



![](/img/in-post/old/20180821230930526_20190219140226739797.png)



#### genspider:

```scrapy genspider -l #展示爬虫模板```



![](/img/in-post/old/20180821230950856_20190219140249288966.png)



![](/img/in-post/old/20180821231009325_20190219140312386204.png)



My_First.py：



```python

#-*-coding: utf-8-*-

import scrapy

class MyFirstSpider(scrapy.Spider):

    name = 'My First'

    allowed domains = ['baidu.com']

    start-urls = ['http://baidu.com/']

    

    def parse(self, response):

        pass

```



#### check(测试爬虫)：

```scrapy check 爬虫名```



#### crawl(运行爬虫)：

```scrapy crawl 爬虫名 控制参数(--nolog等)```



#### list(展示当前可以使用的爬虫文件)：

```scrapy list```



#### parse(获取指定url并进行相应处理及分析)：

```scrapy parse http://www.baidu.com```





### 项目结构



![](/img/in-post/old/20180821231030102_20190219140337401655.png)



middlewares.py 为中间件



### Xpath表达式

通过标签(<tag></tag>)提取信息→更适用于爬虫



/ : 选择标签



text()：定位标签内容       例：

```html/a/b/text() ```



```

<html>

<a>

<b>abc</b>

</a>

</html>

结果：abc

```python

@定位标签属性：标签[@属性=属性值]



//定位所有标签:```//li@[class='hidden-xs']/a/@href```

```

<li class=''hidden-xs>

<a href="www.baidu,com"></a>

</li>

结果：www.baidu.com

```



### 搭建项目步骤



1.scrapy startproject 项目名



2.scrapy genspider -t 模板类型 爬虫名 目标域名



3.进入item.py设置爬取结果容器,容器名 = scrapy.Field()



4.进入爬虫ming.py



5.from 项目名.items import item类名



6.进入parse方法

实例化item类：item = item类名() --------等号左边item名字自取

爬虫操作



7.进入settings.py 设置ITEM_PIPELINES



8.进入pipelines.py设置爬取信息处理动作