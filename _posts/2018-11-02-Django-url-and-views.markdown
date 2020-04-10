---
layout:     post
title:      Django——url和views
subtitle:   ""
date:       2018-11-02 06:36:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Web
    - Django
---
### Views
不同的request请求由不同的views.py中的函数响应

格式如下：

```python
def 名称(request):
    return render(request, 调用的html静态文件, {传入html文件的参数})
```
编写完成后如下：
```python
def index(request):
    #显示简单的“Hello World”,一般很少用
    #return HttpResponse("Hello World")
    
    #返回html静态文件
    return render(request, 'index.html', {})
```

### index/urls
在app目录下的urls.py文件中设置访问路径及该路径对应的views


```python
from django.urls import path
from index import views

urlpatterns = [
    
    #这里将主界面与index/views.py 中的index函数关联
    path('', views.index, name="MainPage"),
]
```

### 根目录/urls.py
在Django项目名称的文件夹中的urls.py中调用app中的urls.py文件（将url分散到各个app下的urls.py文件中再由根目录的urls.py调用可以有效地分类管理）

格式为：

```python
urlpatterns = [
    path('路径'), include('调用的app的urls文件'),
]
```
这里设置如下：
```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('index.urls')),
]
```