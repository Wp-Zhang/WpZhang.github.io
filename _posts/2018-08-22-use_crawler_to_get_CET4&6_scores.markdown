---
layout:     post
title:      Python爬虫——查询英语四、六级成绩
subtitle:   ""
date:       2018-08-22 14:27:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Crawler
---
今天出六级成绩，很多人在成绩出来之后的一段时间都查询不到自己的成绩。晚上有空就写了一个爬虫。



---



首先进入查询四、六级成绩的网页，这里使用的是“http://cet.neea.edu.cn/cet”



进入之后发现不能右键查看源代码，不过也没有关系，直接打开开发者工具

输入前两个信息后点击验证码输入框，发现出现两个新的GET请求。显然这是获取验证码的相关请求。点击第一个请求，可以发现验证码图片链接就在其中



![](/img/in-post/old/20180822222423984_20190219135833801350.png)



后期可以使用正则表达式直接提取。再看请求信息。



![](/img/in-post/old/20180822222445166_20190219135854997045.png)



可见此GET请求的url的参数包含3个参数：



1. 考试类别。这里是查询英语四六级成绩，所以可以设置为固定的'CET' 

1. 准考证号

1. 一个小数



第三个参数初步猜测是一个随机数，但是不确定。可以看见该网页的JavaScript源码



![](/img/in-post/old/20180822222457373_20190219135915933286.png)



可以找到网站构造请求的方法。其中的```Math.random()```说明之前的猜测是正确的。



然后在请求返回的内容中提取图片地址：```img_url = re.compile('"(.*?)"').findall(response.text)[0]```



获取验证码图片并保存到本地的代码如下（因为有验证码的相关操作，所以涉及到Cookies，为了方便使用会话Session）：



```python

#得到相关考生信息

def get_info():

    id_num = input("输入准考证号：")

    name = input("输入姓名：")

    return id_num, name



#获取图片

def get_img(Session, id_numm):

    try:

        headers = {

            'Connection': 'keep - alive',

            'Host': 'cache.neea.edu.cn',

            'Referer': 'http://cet.neea.edu.cn/cet',

            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3486.0 Safari/537.36',

        }

        Session.headers = headers

        get_url = 'http://cache.neea.edu.cn/Imgs.do?'

        params = {

            'c': 'CET',

            'ik': id_numm,

            't': random.random()

        }

        response = Session.get(get_url, params=params)

        img_url = re.compile('"(.*?)"').findall(response.text)[0]

        img = requests.get(img_url, timeout=None)

        with open('D:/img.png', 'wb') as f:

            f.write(img.content)

    except Exception as e:

        print("Imgae_Error:", e.args)

```



本来想使用tesserocr来进行图片识别，结果发现如果不对图片进行较多的预处理，不能直接识别出图片中的验证码，于是直接简单粗暴的让用户输入：）



```python

 def get_score(Session, id_num, name, level):

    Image.open('D:/img.png').show()

    capcha = input('请打开图片输入验证码:')

```



到这里，可以开始构造查询成绩的请求了。再分析请求信息：在输入正确信息后观察



![](/img/in-post/old/20180822222514966_20190219135943152996.png)



找到查询成绩的POST请求，查看其信息：



![](/img/in-post/old/20180822222526854_20190219140005301705.png)



发现传递的数据中有两项：

1. 包含三个参数：考试代码，准考证考，考生姓名

2. 验证码



再看访问网页时产生的请求，发现data.js中有相关信息：



![](/img/in-post/old/20180822222545643_20190219140027183709.png)



可见tab对应的value就是考试代码。而上网查询可知准考证考中的第九位是判断考试类别的。至此，POST请求分析结束，开始写代码：





```python

def get_info():

    id_num = input("输入准考证号：")

    name = input("输入姓名：")

    level = id_num[9]

    return id_num, name, level



def get_score(Session, id_num, name, level):

    Image.open('D:/img.png').show()

    capcha = input('请打开图片输入验证码:')

    

    headers = {

            'Connection': 'keep - alive',

            'Host': 'cache.neea.edu.cn',

            'Origin': 'http://cet.neea.edu.cn',

            'Referer': 'http://cet.neea.edu.cn/cet',

            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3486.0 Safari/537.36',

        }

    

        query_url = "http://cache.neea.edu.cn/cet/query"

    

        test = {

            '1': 'CET4_181_DANGCI',

            '2': 'CET6_181_DANGCI',

        }

        data = {

            'data': test.get(level) + ',' + id_num + ',' + name,

            'v': capcha

        }

        data = urlencode(data)

        response = Session.post(query_url, data=data, headers=headers)

```



再观察query这个POST请求的Response部分（图片上面有），可以发现信息存储的规律，同样使用正则表达式提取并显示出来：





```python

id_num = re.compile("z:'(.*?)'").findall(response.text)[0]

name = re.compile("n:'(.*?)'").findall(response.text)[0]

school = re.compile("x:'(.*?)'").findall(response.text)[0]

score = re.compile("s:(.*?),").findall(response.text)[0]

listening = re.compile("l:(.*?),").findall(response.text)[0]

reading = re.compile("r:(.*?),").findall(response.text)[0]

writing = re.compile("w:(.*?),").findall(response.text)[0]

rank = re.compile("kys:'(.*?)'").findall(response.text)[0]

if level=='1':

    print("\n====================\n\n四级笔试成绩：")

elif level=='2':

    print("\n====================\n\n六级笔试成绩：")

print("准考证号：" + str(id_num))

print("姓名：" + str(name))

print("学校：" + str(school))

print("总分：" + str(score))

print("听力：" + str(listening))

print("阅读：" + str(reading))

print("写作与翻译：" + str(writing))

if level=='1':

    print("\n====================\n\n四级口试成绩：")

elif level=='2':

    print("\n====================\n\n六级口试成绩：")

print("等级：" + str(rank))

```



至此，主要函数就构造完成。写主函数：



```python

def main():

    id_num, name ,level = get_info()

    s = requests.Session()

    get_img(s, id_num)

    get_score(s, id_num, name, level)

    #后期打包成.exe时有用，如果不打包，可以删掉

    #end = input("输入任意键退出...") 

```

四六级成绩查询的爬虫就写好了：



![](/img/in-post/old/20180822222633742_20190219140052839777.png)





完整代码如下：





```python

import requests

import re

import random

from PIL import Image

from urllib.parse import urlencode



def get_info():

    id_num = input("输入准考证号：")

    name = input("输入姓名：")

    level = id_num[9]

    return id_num, name, level



def get_img(Session, id_numm):

    try:

        headers = {

            'Connection': 'keep - alive',

            'Host': 'cache.neea.edu.cn',

            'Referer': 'http://cet.neea.edu.cn/cet',

            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3486.0 Safari/537.36',

        }

        Session.headers = headers

        get_url = 'http://cache.neea.edu.cn/Imgs.do?'

        params = {

            'c': 'CET',

            'ik': id_numm,

            't': random.random()

        }

        response = Session.get(get_url, params=params)

        img_url = re.compile('"(.*?)"').findall(response.text)[0]

        img = requests.get(img_url, timeout=None)

        with open('D:/img.png', 'wb') as f:

            f.write(img.content)

        Image.open('D:/img.png').show()

    except Exception as e:

        print("Imgae_Error:", e.args)



def get_score(Session, id_num, name, level):

    capcha = input('请打开图片输入验证码:')



    headers = {

        'Connection': 'keep - alive',

        'Host': 'cache.neea.edu.cn',

        'Origin': 'http://cet.neea.edu.cn',

        'Referer': 'http://cet.neea.edu.cn/cet',

        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3486.0 Safari/537.36',

    }



    query_url = "http://cache.neea.edu.cn/cet/query"



    test = {

        '1': 'CET4_181_DANGCI',

        '2': 'CET6_181_DANGCI',

    }

    data = {

        'data': test.get(level) + ',' + id_num + ',' + name,

        'v': capcha

    }

    data = urlencode(data)

    response = Session.post(query_url, data=data, headers=headers)

    if 'error' in response.text:

        e = re.compile("'error':'(.*?)'|error:'(.*?)'").findall(response.text)[0]

        if e is not None:

            #print(e)

            if '验证码错误' in e[1]:

                print("验证码输入错误！")

                get_img(Session, id_num)

                get_score(Session, id_num, name)

            else:

                print(e[0])

    else:

        id_num = re.compile("z:'(.*?)'").findall(response.text)[0]

        name = re.compile("n:'(.*?)'").findall(response.text)[0]

        school = re.compile("x:'(.*?)'").findall(response.text)[0]

        score = re.compile("s:(.*?),").findall(response.text)[0]

        listening = re.compile("l:(.*?),").findall(response.text)[0]

        reading = re.compile("r:(.*?),").findall(response.text)[0]

        writing = re.compile("w:(.*?),").findall(response.text)[0]

        rank = re.compile("kys:'(.*?)'").findall(response.text)[0]

        if level=='1':

            print("\n====================\n\n四级笔试成绩：")

        elif level=='2':

            print("\n====================\n\n六级笔试成绩：")

        print("准考证号：" + str(id_num))

        print("姓名：" + str(name))

        print("学校：" + str(school))

        print("总分：" + str(score))

        print("听力：" + str(listening))

        print("阅读：" + str(reading))

        print("写作与翻译：" + str(writing))

        if level=='1':

            print("\n====================\n\n四级口试成绩：")

        elif level=='2':

            print("\n====================\n\n六级口试成绩：")

        print("等级：" + str(rank))

def main():

    id_num, name ,level = get_info()

    s = requests.Session()

    get_img(s, id_num)

    get_score(s, id_num, name, level)

    end = input("输入任意键退出...")



if __name__ == '__main__':

    main()









```