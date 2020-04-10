---
layout:     post
title:      Django 定制后台和修改模型
subtitle:   ""
date:       2018-11-09 04:54:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Web
    - Django
---
### 1.定制admin后台

##### 1).进入models.py
在数据模型类中定义函数：
```python
    def __str__(self):
        return self.title
```
这样就在后台管理Article模型时将显示文章的标题：

![](/media/editor/20181109125151777_20190219134145350035.png)

#### 2).进入admin.py
**a. 定义一个数据模型的专属类继承** admin.ModelAdmin类，其中设置将要显示的详细信息组成的元组/列表(由于后期无需修改，设置为元组比较好)
```python
class ArticleAdmin(admin.ModelAdmin):
    list_display = ('id', 'title', 'created_date', 'update_date')
```
**b. 在注册数据模型时添加之前建立的类为参数** 

方法一：在类之后添加如下语句
```python
admin.site.register(Article, ArticleAdmin)
```

方法二：在类之前添加修饰器：
```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    ...
```


完成之后后台显示如下：

![](/media/editor/20181109125333222_20190219134208857651.png)

发现目前排序方式为以ID的倒序排序，可以在admin.py中修改：
```python
class ArticleAdmin(admin.ModelAdmin):
    list_display = ('id', 'title', 'created_date', 'update_date')
    ordering = ('id',)  #根据id顺序排列
    #ordering = ('-id',) 根据id倒序排列
```
更改结果如下：

![](/media/editor/20181109125419747_20190219134239273757.png)