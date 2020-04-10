---
layout:     post
title:      Django——数据库配置和数据模型创建
subtitle:   ""
date:       2018-11-01 16:33:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Web
    - Django
---
### 数据库配置
###### settings.py

```python
DATABASES = {
    #第一个数据库
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'blog',
        'USER': 'root',
        'PASSWORD': 'Zwp0816...',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    },
    
    #第二个数据库
    'MyDjango': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'sqlite3'),
    },
}
```

### 数据模型创建
Django的数据模型以类包装，具体格式为：

```python
from django.db import models

#class 模型类名称(models.Model)
class ModelName(models.Model)
    
    #数据模型 = 数据类型
    title = models.CharField(max_length=20)
    author = models.CharField(max_length=20, null=True)
    created_date = models.DateField(auto_now_add=True)
    update_date = models.DateField(auto_now=True)
    content = models.TextField()
    is_showed = models.BooleanField()
```

###### index/models.py

```python
from django.db import models

# Create your models here.
class Article(models.Model):
    title = models.CharField("标题",max_length=20)
    author = models.CharField("作者", max_length=20, null=True)
    created_date = models.DateField("创建日期", auto_now_add=True)
    update_date = models.DateField("修改日期", auto_now=True)
    content = models.TextField("内容")
    is_showed = models.BooleanField("私密设置")

    #定义数据库中存储数据模型的表名称
    class Meta:
        db_table = "article"

    def __str__(self):
        return self.title
```

### 激活模型
1. 在 settings.py 中的INSTALLED_APPS列表中添加app
2. 运行命令
```python
python manage.py makemigrations index
```
其中index是app名称
3. 运行命令
```python
python manage.py migrate
```

### 注册模型
为了便于管理数据模型，需要将数据注册到admin中。
如果没有创建admin账户，首先在终端运行指令：
```python
python manage.py createsuperuser
```
然后根据提示输入相关信息即可创建一个超级用户
创建完成后进入app目录下的 admin.py
1. 导入想要注册的数据模型
    
```python
from .models import Article
```
2. 进行注册操作
```python
admin.site.register(Article)
```
至此注册完成，可以在后台对该数据模型进行管理
![](/media/editor/20181109125018773_20190219134334745039.png)