---
layout:     post
title:      读书笔记—— Urllib库(2)
subtitle:   "《精通Python网络爬虫》"
date:       2018-10-21 14:46:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Crawler
---
### 一、利用Urllib爬取网页

##### 8.HTTP请求



&emsp;&emsp;HTTP协议请求主要分为6类：GET，POST，DELETE，PUT， HEAD， OPTIONS



###### 1). GET 请求



在淘宝上搜索Python后观察可以发现URL由

‘http://www.taobao.com’

;变为了

‘https://s.taobao.com/search?q=Python&imgfile=&commend=all&ssid=s5-e&search_type=item&sourceId=tb.index&spm=a21bo.2017.201856-taobao-item.1&ie=utf8&initiative_id=tbindexz_20170306’，其中所搜索的‘Pyhton’在‘search?/q=’后。由此推测，如果 URL= 'htttp://s.taobao.com/search?q=' + 关键词 

那么我们就可以直接爬取我们想要在淘宝上搜索的商品的网页信息。因此可以如下操作：

```python

url = "http://s.taobao.com/search?q="

keyword = "Python"

url += keyword 

request = urllib.request.Request(url)

data = urllib.request.urlopen(request).read()

file = open("D:/Pyhton/2.html", 'wb')

file.write(data)

file.close()

```

这样就保存了淘宝搜索关键词为“Python”的网页信息。



如果搜索关键词含中文或其他一些特殊字符，应先将关键词编码：

```python

url = "http://s.taobao.com/search?q="

keyword = "Python爬虫"

keyword_code = urllib.request.quote(keyword)

url += keyword_code

```





###### 2).POST请求



&emsp;&emsp;实现思路：



1. 设置URL网址；



2. 构建表单数据，使用urllib.parse.urlencode对数据进行编码；



3. 创建Request对象，参数含URL、要传递的数据；



4. 使用urllib.request.urlopen()打开Request对象；



5. 后续处理。



**重点：构建表单数据并对其编码：**



&emsp;&emsp;查看网页源代码，找到对应的form表单部分，找到输入框属性值，将其包含于构造的数据中，格式为字典，属性值在字典中对应的值为要传递的值。此处用“http://www.iqianyue.com/mypost/”作为例子，构建表单部分如下：

```python

url = "http://www.iqianyue.com/mypost/"

postdata = urllib.parse.urlencode({

"name":"12345678@163.com",

"password":"Aa12345"

}).encode('utf-8') #将数据处理后设置为utf-8编码

request = urllib.request.Request(url, postdata)

```



完整代码如下：





```python

import urllib.parse

import urllib.request

url = "http://www.iqianyue.com/mypost/"

postdata = urllib.parse.urlencode({

"name":"123456@163.com",

"password":"Aa12345"

}).encode('utf-8')

request = urllib.request.Request(url, postdata)

request.add_header('User-Agent', 'Mozilla/5.0')

data = urllib.request.urlopen(request).read()

file = open("D:/3.html",'wb')

file.write(data)

file.close()

```

##### 9.设置代理服务器



1. 使用  urllib.request.ProxyHandler({'http':Proxy_addr})  设置代理服务器信息，其中Proxy_addr为“网址：端口号”；



2. 创建一个opener对象，以代理服务器信息和urllib.request.HTTPHandler为参数 ；



3. 创建全局默认的opener对象；



4. 后续操作；



代码如下：

```python

import urllib.request



proxy = urllib.request.ProxyHandler({'http':'116.55.77.81:61201'})  #a

urllib.request.build_opener(proxy, opener = urllib.request.HTTPHandler)  #b

urllib.request.install_opener( opener)  #c

data = urllib.request.urlopen("http://www.baidu.com").read().decode('utf-8')  #d

```





##### 10.URLError



这里主要使用两个类：URLError类 及其子类 HTTPError类



使用代码：

```python

import urllib.request

import urllib.error



try:

    urllib.request.urlopen("http://www.baidu.com")

except urllib.error.URLError as e:

    print(e.code)

    print(e.reason)

```

产生URLError的可能有：



&emsp;&emsp;a.连不上服务器；



&emsp;&emsp;b.远程URL不存在;



&emsp;&emsp;c.无网络；



&emsp;&emsp;d.触发 HTTPError



&emsp;&emsp;若产生 HTTPError 可用 HTTPError 替换 except 中的 URLError，但是要提前进行判断错误类型。若触发前三种错误时使用 except urllib.error.HTTPError as e 将会产生错误，原因是无法打印 e.reason。因此可以将代码写成：

```python

import urllib.request

import urllib.error



try:

    urllib.request.urlopen("http://www.baidu.com")

except urllib.error.HTTPError as e:

    print(e.code)

    print(e.reason)

except urllib.error.URLError as e: #先用子类处理，若无法处理再使用父类处理

    print (e.reason)

（状态码及含义见《精通Python网络爬虫》P48）

```