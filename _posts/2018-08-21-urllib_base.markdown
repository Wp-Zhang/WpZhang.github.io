---
layout:     post
title:      基本库的使用——urllib库
subtitle:   ""
date:       2018-08-21 02:10:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Crawler
---
# urllib库
### 1.导入对应模块：
因为Python 3.*版本中将urllib和urllib2合并为urllib，所以直接

```python
import urllib.request
```

### 2.使用 urllib.request.urlopen(url)打开并爬取一个网页
这里将百度作为例子

```python
file = urllib.request.urlopen("http://www.baidu.com")
```

### 3.将对应网页的内容读取出来
读取内容的常见方式有3种：
1. a. file.read() 读取文件全部内容，把读取到的内容赋给一个字符串变量；
1. b. file.readlines() 读取文件全部内容，把读取到的内容赋给一个列表变量，若要读取全部内容，推荐这种方式；
1. c. file.readline() 读取文件的一行内容。

```python
data = file.read()
dataline = file.readline()
```

这里分别将爬取的网页的全部内容和一行内容分别赋给变量data, dataline
若想输出爬取到的内容，可以直接使用print() :

```python
print (data)
print (dataline)
```

 
### 4.保存爬取的网页内容
 步骤：
1. 爬取一个网站读取出来并赋给一个变量；
1. 以写入的方式打开一个本地文件，使用 .html后缀；
1. 将a中的变量的值写入文件；
1. 关闭文件

```python
request = urllib.request.urlopen("http://www.baidu.com")
data = request.read()
file = open("D:/Python/test.html", 'wb')
file.write(data)
file.close()
```
或

```python
with open("D:/Python/test.html", 'wb') as file:
    request = urllib.request.urlopen("http://www.baidu.com")
    data = request.read()
    file.write(data)
```


此外，还有另一种方法直接写入文件：

```python
filename = urllib.request.urlretrieve(url, filename = "D:/Python/test.html")
```


可以使用用 urllib.request.cleanup()清空使用urlretrieve产生的缓存。
 
### 5.其他
1. 使用 request.getcode() 可以返回爬取网页的状态码，若为200则爬取正常;
1. 使用 request.geturl()返回爬取网页的url;
1. 一般来说，URL标准中只会允许一部分ASCⅡ字符，若要在其中使用汉字等其他字符，需对url进行编码

    这里以淘宝为例

    ```python
    urllib.request.quote("http://www.taobao.com")
    'http%3A//www.taobao.com'
    ```
    
    相应地，对url进行解码诗可以使用如下操作：
    

 

    urllib.request.unquote("http%3A//www.taobao.com")
    'http://www.taobao.com'




 
### 6.模拟浏览器
用爬虫直接爬取网页时，request中的header会显示其为爬虫，某些网页为了防止爬虫采集信息会进行反爬虫设置。此时为了成功爬取该网页，需以浏览器的身份进行爬取。因此需要修改Headers属性——将键‘user-agent’的值修改为‘Mozilla/5.0XXXXXXXX’
#####     方法一： 使用 build_opner()修改报头

```python
url = "http://www.baidu.com"
opener = urllib.request.build_opener()
opener.adddheaders = [ ('User-Agent','Mozilla/5.0') ]
data = opener.open(url).read()
```

#####     方法二： 使用 add_headers()添加报头
首先使用urllib.request.Request(url)创建一个Request对象并赋给request，然后使用add_header()添加对应的报头信息

```python
url = "http://www.baidu.com"
request = urllib.request.Request(url)
request.add_header('User-Agent', 'Mozilla/5.0')
data = urllib.request.urlopen(request).read()
```
#####   方法三： 使用headers参数
首先构造报头信息

```python
url = "http://www.baidu.com"
headers = {
    'user-agent':'Mozilla/5.0'
}
request = urllib.request.urlopen(url, headers=headers)
```

 
### 7.超时设置
有时访问一个网页，其长时间未反应，系统会判断访问超时，可以自己设置限制的超时访问时间

```python
request = urllib.request.urlopen("htttp://www.baidu.com", timeout = 30)
```

其中，timeout的单位为秒。
例中若向网页发送请求后服务器在30s内未响应，则会引发异常。


```python
import socket
import urllib.request
import urllib.error

try:
    response = urllib.request.urlopen('http://www.baidu.com', timeout=20)
except urllib.error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```


### 8.HTTP请求
HTTP协议请求主要分为6类：GET，POST，DELETE，PUT， HEAD， OPTIONS
#### 1). GET 请求
在淘宝上搜索Python后观察可以发现URL由

```python
http://www.taobao.com
```

变为了

```python
https://s.taobao.com/search?q=Python&imgfile=&commend=all&ssid=s5-e&search_type=item&sourceId=tb.index&spm=a21bo.2017.201856-taobao-item.1&ie=utf8&initiative_id=tbindexz_20170306
```

其中所搜索的‘Pyhton’在‘search?/q=’后。由此推测，如果
```python
URL= 'htttp://s.taobao.com/search?q=' + 关键词
```
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

 
 
#### 2).POST请求
实现思路：
1. 设置URL网址；
1. 构建表单数据，使用urllib.parse.urlencode对数据进行编码；
1. 创建Request对象，参数含URL、要传递的数据；
1. 使用urllib.request.urlopen()打开Request对象；
1. 后续处理。

重点：构建表单数据并对其编码：
        查看网页源代码，找到对应的form表单部分，找到输入框属性值，将其包含于构造的数据中，格式为字典，属性值在字典中对应的值为要传递的值。
        
此处用 http://www.iqianyue.com/mypost/ 作为例子，构建表单部分如下：

```python
url = "http://www.iqianyue.com/mypost/"
postdata = urllib.parse.urlencode(
{
    "name":"12345678@163.com",
    "password":"Aa12345"
}).encode('utf-8') #将数据处理后设置为utf-8编码
request = urllib.request.Request(url, postdata)
#或
postdata = bytes(postdata)
request = urllib.request.urlopen(url, data=postdata)
```

 
完整代码如下：

```python
import urllib.parse
import urllib.request
url = "http://www.iqianyue.com/mypost/"
postdata = urllib.parse.urlencode(
{
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

### 9.设置代理服务器
a.使用  ```urllib.request.ProxyHandler({'http':Proxy_addr})```
设置代理服务器信息，其中Proxy_addr为“网址：端口号”；

b.创建一个opener对象，以代理服务器信息和urllib.request.HTTPHandler为参数 ；

c.创建全局默认的opener对象；

d.后续操作；

代码如下：

```python
import urllib.request
 
proxy = urllib.request.ProxyHandler({'http':'116.55.77.81:61201'})  #a
urllib.request.build_opener(proxy, opener = urllib.request.HTTPHandler)  #b
urllib.request.install_opener( opener)  #c
data = urllib.request.urlopen("http://www.baidu.com").read().decode('utf-8')  #d
```

 
### 10.URLError
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
 
 a.连不上服务器;
 
 b.远程URL不存在;
 
 c.无网络;
 
 d.触发 HTTPError
 
若产生 HTTPError 可用 HTTPError 替换 except 中的 URLError，但是要提前进行判断错误类型。若触发前三种错误时使用 except urllib.error.HTTPError as e 将会产生错误，原因是无法打印 e.reason。因此可以将代码写成：

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
```

### 11.使用Cookie
获取Cookie

```python
import urllib.request, http.cookiejar

cookie = cookiejar.CookieJar()
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
```

保存Cookie到本地

```python
filename = 'cookie.txt'
cookie = http.cookiejar.MozillaCookieJar(filename) #此处将CookieJar替换为MozillaCookieJar
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open("http://www.baidu.com")
cookie.save(ignore_discard=True, ignore_expires=True)
```

读取本地Cookie

```python
cookie = http.cookiejar.MozillaCookieJar()
cookie.load('cookie.txt', ignore_discard=True, ignore_expires=True)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
```