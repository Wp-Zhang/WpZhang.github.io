---
layout:     post
title:      读书笔记—— Urllib库(1)
subtitle:   "《精通Python网络爬虫》读书笔记"
date:       2018-10-21 14:46:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Crawler
---
### 一、利用Urllib爬取网页

##### 1.导入对应模块：



&emsp;&emsp;因为Python 3.*版本中将urllib和urllib2合并为urllib，所以直接

```python

import urllib.request

```

##### 2.使用 urllib.request.urlopen(url)打开并爬取一个网页



&emsp;&emsp;这里将百度作为例子

```python

file = urllib.request.urlopen("http://www.baidu.com")

```

##### 3.将对应网页的内容读取出来



&emsp;&emsp;读取内容的常见方式有3种：

1. file.read() 读取文件全部内容，把读取到的内容赋给一个字符串变量；

2. file.readlines() 读取文件全部内容， 把读取到的内容赋给一个列表变量，若要读取全部内容，推荐这种方式；

3. file.readline() 读取文件的一行内容。



```python

data = file.read()

```

```python

dataline = file.readline()

```

&emsp;&emsp;这里分别将爬取的网页的全部内容和一行内容分别赋给变量data, dataline



&emsp;&emsp;若想输出爬取到的内容，可以直接使用print() :

```python

print (data)

print (dataline)

```





##### 4.保存爬取的网页内容



 步骤：a. 爬取一个网站读取出来并赋给一个变量；



           b.以写入的方式打开一个本地文件，使用 .html后缀；



           c.将a中的变量的值写入文件；



           d.关闭文件



request = urllib.request.urlopen("http://www.baidu.com")

data = request.read()

file = open("D:/Python/test.html", 'wb')

file.write(data)

file.close()

此外，还有另一种方法直接写入文件：



filename = urllib.request.urlretrieve(url, filename = "D:/Python/test.html")





    可以使用用 urllib.request.cleanup()清空使用urlretrieve产生的缓存。







##### 5.其他



1). 使用 request.getcode() 可以返回爬取网页的状态码，若为200则爬取正常;



2). 使用 request.geturl()返回爬取网页的url;



3). 一般来说，URL标准中只会允许一部分ASCⅡ字符，若要在其中使用汉字等其他字符，需对url进行编码（这里以淘宝为例）：

```python

urllib.request.quote("http://www.taobao.com")

'http%3A//www.taobao.com'

```



&emsp;&emsp;相应地，对url进行解码诗可以使用如下操作：





```python

urllib.request.unquote("http%3A//www.taobao.com")

'http://www.taobao.com'

```





##### 6.模拟浏览器



&emsp;&emsp;用爬虫直接爬取网页时，request中的header会显示其为爬虫，某些网页为了防止爬虫采集信息会进行反爬虫设置。此时为了成功爬取该网页，需以浏览器的身份进行爬取。因此需要修改Headers属性——将键‘user-agent’的值修改为‘Mozilla/5.0’



###### 方法一： 使用 build_opner()修改报头

```python

opener = urllib.request.build_opener()

opener.adddheaders = [ ('User-Agent','Mozilla/5.0') ]

data = opener.open(url).read()

```

###### 方法二： 使用 add_headers()添加报头



首先使用urllib.request.Request(url)创建一个Request对象并赋给request，然后使用add_header()添加对应的报头信息

```python

url = "http://www.baidu.com"

request = urllib.request.Request(url)

request.add_header('User-Agent', 'Mozilla/5.0')

data = urllib.request.urlopen(request).read()

```





##### 7.超时设置



&emsp;&emsp;有时访问一个网页，其长时间未反应，系统会判断访问超时，我们可以自己设置限制的超时访问时间

```python

request = urllib.request.urlopen("htttp://www.baidu.com", timeout = 30)

```

&emsp;&emsp;其中，timeout的单位为秒。



&emsp;&emsp;例中若向网页发送请求后服务器在30s内未响应，则会引发异常。