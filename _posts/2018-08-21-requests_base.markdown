---
layout:     post
title:      基本库的使用——requests
subtitle:   ""
date:       2018-08-21 09:38:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Crawler
---
# requests库
### 1) GET请求
##### 基本操作
```python
import requests
url = 'https://www.baidu.com'
response = requests.get(url)
```
##### 添加参数，头信息

```python
params = {
    'kw':'Python',
    'page':'1'
}
headers = {
    'user-agent':'Mozilla/5.0'
}
url = 'https://www.baidu.com'

response = requests.get(url, params = params, headers = headers)

#此时url就被构造为了'https://www.baidu.com/?kw=Python&page=1', 且更改了头信息
```

```response.text```
为str类型数据，如果是json格式储存的，可以使用```response.json()```将其转化为字典类型（如在Ajax分析中的应用）

##### 抓取数据
对于普通的网页源码信息，可以直接使用正则表达式等对```response.text```进行过滤抓取

对于图片、视频等以二进制格式储存的文件，可以使用```response.content```直接提取内容后写入，例：

```python
import requests

url = 'https://tse3-mm.cn.bing.net/th?id=OIP.ylRuhRHnozVzZh7MP4EUIwHaEo&w=300&h=187&c=7&o=5&pid=1.7'

response = requests.get(url)
with open('MyImg.jpg', 'wb') as file:
    file.write(response.content)
```
### 2) POST请求
#### 基本操作
```python
import requets
data = {
    'name': 'Ramond',
    'age': 21,
    'weight': 35 
}
url = 'http://www.personalInfo.com'
response = requests.post(url, data = data)
```
#### 上传文件
```python
files = {
    'file': open('upload.dat', 'rb')
}
url = 'http://www.uploadfile.com'
response = requests.post(url, files = files)
```

### 3) 响应

信息 | 代码
---|---
状态码 | ```response.status_code```
头信息 | ```response.headers```
Cookies | ```response.cookies```
URL | ```response.url```
请求历史 | ```response.history```


### 4) Cookies
#### 获取Cookies
```python
import requests
response = requests.get('http://www.baidu.com')
cookies = response.cookies
```
#### 使用Cookies
###### 方法一：在头文件中设置Cookie信息
```python
#使用cookies登录豆瓣
import requests
headers = {
    'user-agent': 'Mozilla/5.0',
    'Cookie': 'll="118318"; bid=BHMEaMiJwCE; _vwo_uuid_v2=DECC5EA8684CB406A84D872F80340BDF6|72a3f16f036e918fe233f47599bf881a; __yadk_uid=saRZoxU38xPpSTuPI6OCW2FQPCdRLRnq; gr_user_id=3d628e0d-cd54-4985-9f06-809b68c0d917; viewed="6021440_6709783"; ps=y;  ',
    'host': 'www.douban.com'
}
response = requests.get('https://www.douban.com', headers = headers)
```
###### 方法二：构造RequetsCookieJar对象

```python
import requests

headers = {
    'user-agent': 'Mozilla/5.0',
    'host': 'www.douban.com'
}
cookies = 'll="118318"; bid=BHMEaMiJwCE; _vwo_uuid_v2=DECC5EA8684CB406A84D872F80340BDF6|72a3f16f036e918fe233f47599bf881a; __yadk_uid=saRZoxU38xPpSTuPI6OCW2FQPCdRLRnq; gr_user_id=3d628e0d-cd54-4985-9f06-809b68c0d917; viewed="6021440_6709783"; ps=y;  '
jar = requests.cookies.RequestsCookieJar()

for single_cookie in cookies.split(';'):
    key, value = single_cookie.split('=', 1)
    jar.set(key, value)

url = 'https://www.douban.com'
response = requests.get(url, headers = headers, cookies = jar)
```

### 5) 会话维持

相当于在不关闭浏览器的情况下打开新标签页，可用于保持post()后的登录状态

```python
import requests
s = requests.Session()
response = s.get('https://www.douban.com')
#除了构造Session对象外，与直接操作没有太大区别
```

### 6) 代理设置

```python
import requests

proxies = {
    'http': 'http://1.1.1.1:8888',
}
response = requests.get('http://www.baidu.com', proxies = proxies)
```

### 7) 超时设置

```python
import requests

url = "https://www.baidu.com"
response = requests.get(url, timeout = 1)
#当超时时抛出异常
```
timeout时间为连接和读取的时间总和，可以分别设置
```
response = requests.get(url, timeout = (1,5,10))
```

### 8) 身份认证
```python
import requests
url = 'http://www.baidu.com'
response = requests.get(url, auth=('username', 'password'))
```

### 9) Prepared Request
可以将请求进行封装后以对象的形式提交

```python
from requests import Request,Session

url = 'http://www.baidu.com'
headers = {
    'user-agent':'Mozilla/5.0',
}
data = {
    'kw': 'keyword',
}

s = Session()
request = Request('POST', url, data=data, headers=headers)
prepared = s.prepare_request(request)

response = s.send(prepared)
```