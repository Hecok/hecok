---
title: Getting Started
author: Cotes Chung
date: 2019-08-09 20:55:00 +0800
categories: [Blogging, Tutorial]
tags: [getting started]
pin: true
---


# 数据模型-Models
## 1.定义models
每个模型使用python的一个类表示，并且是django.db.models.Model的子类：
```python
from django.db import models
class XXX(models.Model):
```
## 2.安装模型
目录：settings-INSTALLED_APPS
命令：
- 验证模型：`python manage.py check`
- 生成迁移：`python manage.py makemigrations`
- 执行迁移：`python manage.py migrate`
- 使用shell测试模型：`python manage.py shell`

## 3.model管理器
objects是Django模型用于执行数据库查询的对象，一个Django模型至少有一个管理器，同时可以自定义管理器，定制访问数据库的方式。自定义管理器可能处于两方面的原因：添加额外的管理器方法和修改管理器返回的QuerySet。
- **添加额外的管理器方法**：
```python
class BookManager(models.Manager):
    def title_count(self, keyword):
       return self.filter(title__icontains=keyword).count()
class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()
    num_pages = models.IntegerField(blank=True, null=True)
    objects = BookManager()

    def __str__(self):
        return self.title
# 调用
Book.objects.title_count(keyword)
```

- **修改管理器返回的QuerySet**：

&emsp;&emsp;&emsp;&emsp;注意：Django把它解释的第一个管理器定义为“默认”管理器，之后很多地方只使用那个管理器
```python
"""
Book.objects.all() 返回数据库中的所有图书，而Book.dahl_objects.all() 只返回Roald Dahl 写的书
"""
# 首先，定义 Manager 子类
class DahlBookManager(models.Manager):
    def get_queryset(self):
    # 超类 super(class_name, self).def_name(*args, **kwargs)，确保把对象保存到数据库中。如果忘记，默认的行为不会发生，根本不会触及数据库
       return super(DahlBookManager, self).get_queryset().filter(author='Roald Dahl')
# 然后，放入 Book 模型
class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)
    # ...
    objects = models.Manager() # 默认的管理器
    dahl_objects = DahlBookManager() # 专门查询 Dahl 的管理器

# 调用
Book.dah1_objects.all()
Book.dah1_objects.filter()
Book.dah1_objects.count()
Book.objects.all()
```
## 4.模型方法
管理器的作用是执行数据表层的操作，而模型方法处理的是具体的模型实例。包括：
- **自带模型方法**：

 *__str__()*：返回对象的 Unicode 表示形式  在交互式控制台或管理后台中显示对象调用的都是这个方法。

*get_absolute_url()*：这个方法告诉 Django 如何计算一个对象的 URL。Django 在管理后台和需要生成
对象的 URL 时调用这个方法。具有唯一标识的 URL 的对象都要定义这个方法。

- **自定义模型方法**：在所设计的模型类中加入自定义函数方法即可
```python
import datetime
class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birth_date = models.DateField()
    def baby_boomer_status(self):
        # 返回一个人的出生日期与婴儿潮的关系
        if self.birth_date < datetime.date(1945, 8, 1):
            return "Pre-boomer"
        elif self.birth_date < datetime.date(1965, 1, 1):
            return "Baby boomer"
        else:
            return "Post-boomer"
    def _get_full_name(self):
        # 返回一个人的全名
        return '%s %s' % (self.first_name, self.last_name)
    full_name = property(_get_full_name)
```
- **覆盖预定义的模型方法**：
```python
class Blog(models.Model):
     name = models.CharField(max_length=100)
     tagline = models.TextField()
     def save(self, *args, **kwargs):
        do_something()
        super(Blog, self).save(*args, **kwargs) # 调用“真正的”save () 方法
        do_something_else()
```
## 5.数据操作
参考：[QuerySet API 参考¶][1]
- 检索数据：get()、all()...
- 过滤数据：filter()、exclude()...
- 排序数据：order_by("字段",) 常常包含在 class Meta:
- 链式操作：objects.filter(**kwargs).order_by()/delete()/save()...
- 数据切片：order_by()[0:5] 不支持负数...


  [1]: https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/#
