---
layout:     post
title:      几行代码在Django搭建的网站中显示markdown中的latex数学公式
subtitle:   ""
date:       2019-04-14 12:26:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Web
    - Django
---
&nbsp; &nbsp; &nbsp; &nbsp; 在自己用Django搭的博客网站上写博客的时候一直很难受，因为基本的markdown模块好像并不支持latex。网上虽然能找到方法但是我很懒，不想再为这个修改很多代码了。之前为了在博客里面显示latex公式一直都是在这个网站[codecogs](http://latex.codecogs.com/)上把公式输进去再复制产生的图片链接添加到博客里。当公式很多的时候是一个很浪费时间的事情，几乎就要放弃个人博客转战简书了。



&nbsp; &nbsp; &nbsp; &nbsp; 今天突然想到一种曲线救国的方式：可以用正则表达式搜索文章中的latex公式，然后自动将其替换为相应的图片链接！



代码如下：



```python

import re

from urllib.parse import urlencode

pattern = re.compile(r'(\$\$.*?\$\$)', re.S)

latex1 = re.sub(pattern, lambda m: '<div align=center><img src="http://latex.codecogs.com/gif.latex?' + urlencode({'':m.group(0).replace('$$','')})[1:]+'"></div>', article.content, 0)

pattern2 = re.compile(r'(\$.*?\$)', re.S)

content = re.sub(pattern2, lambda m: '<img src="'+ 'http://latex.codecogs.com/gif.latex?' +urlencode({'':m.group(0).replace('$','')})[1:]+'">', latex1, 0)

content = content.replace('+',' ')    # urlencode会把空格替换为+号，这里把它替换回来

```



&nbsp; &nbsp; &nbsp; &nbsp; 这里的pattern匹配的是单行公式，pattern2匹配的是行内公式，将匹配到的结果的首尾的公式标识符'$'去掉以后使用urllib中的parse模块将公式替换为相应的图片链接，然后将其包装为富文本格式（用来在网页上显示图片并调整排版），最终用结果替换原来的latex公式。



&nbsp; &nbsp; &nbsp; &nbsp; 将这几行代码复制到django项目中单个文章的views函数中，在从数据库获得文章内容后将其处理，把处理后的文本再返回给请求对象。



在我的django项目中是这样的：



```python

def article_page(request, article_id):

    change_info(request)

    article = models.Article.objects.get(pk=article_id)



    article.increase_views()



    ############### 新增加的代码   article.content是我的文章的model中对应的文章内容

    import re

    from urllib.parse import urlencode

    pattern = re.compile(r'(\$\$.*?\$\$)', re.S)

    latex1 = re.sub(pattern, lambda m: '<div align=center><img src="http://latex.codecogs.com/gif.latex?' + urlencode({'':m.group(0).replace('$$','')})[1:]+'"></div>', article.content, 0)

    pattern2 = re.compile(r'(\$.*?\$)', re.S)

    content = re.sub(pattern2, lambda m: '<img src="'+ 'http://latex.codecogs.com/gif.latex?' +urlencode({'':m.group(0).replace('$','')})[1:]+'">', latex1, 0)

    ##############

        

    content = markdown.markdown(content,

                                extensions=[

                                    'markdown.extensions.extra',

                                    'markdown.extensions.codehilite',

                                    'markdown.extensions.toc',

                                ])

    comment_list = article.comment_set.all().order_by("-created_time")



    return render(request, 'blog/newPost.html', {'article': article, 'content': content, 'comments': comme

nt_list})

```



&nbsp; &nbsp; &nbsp; &nbsp; 这样更改之后，编辑博客内容的时候就可以像简书中那样正常编辑了～唯一的不足是编辑文章的时候预览界面仍然是latex格式的公式，不能预览。