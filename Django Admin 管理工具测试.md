# Django Admin 管理工具 | 测试--readthedocs

---

### <center>目录</center>



---

[Source](http://www.runoob.com/django/django-admin-manage-tool.html "Permalink to Django Admin 管理工具 | 菜鸟教程")



`Django` 提供了基于 `web` 的管理工具。

`Django`自动管理工具是`django.contrib`的一部分。你可以在项目的 `settings.py` 中的`INSTALLED_APPS`看到它：
    
    
    
    INSTALLED_APPS = (
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
    )
    

`django.contrib`是一套庞大的功能集，它是`Django`基本代码的组成部分。

* * *

I.激活管理工具
===


通常我们在生成项目时会在`urls.py`中自动设置好，我们只需去掉注释即可。

配置项如下所示：
    
    
    
    from django.contrib import admin
    admin.autodiscover()
    
    # And include this URLpattern...
    urlpatterns = patterns('',
        # ...
        (r'^admin/', include(admin.site.urls)),
        # ...
    )
    
i.实际`urls.py`
---


```python 
[root@master HelloWorld]# cat urls.py
"""HelloWorld URL Configuration

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/1.10/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  url(r'^$', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  url(r'^$', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.conf.urls import url, include
    2. Add a URL to urlpatterns:  url(r'^blog/', include('blog.urls'))
"""
# Origin code
# from django.conf.urls import url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
 ]

# from django.conf.urls import *
from HelloWorld.view import hello
from django.conf.urls import url
from HelloWorld.testdb import testdb
from HelloWorld import search
from HelloWorld import search2


urlpatterns = [
    url('^hello/$',hello),
    url('^testdb/$',testdb),
    url(r'^search-form/$',search.search_form),
    url(r'^search/$',search.search),
    url(r'^search-post/$',search2.search_post),
]

```

ii.报错`NameError: name 'url' is not defined`
---


运行服务器

```python
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
Performing system checks...

Unhandled exception in thread started by <function wrapper at 0x23d4de8>
Traceback (most recent call last):
  File "/usr/lib/python2.7/site-packages/django/utils/autoreload.py", line 226, in wrapper
    fn(*args, **kwargs)
  File "/usr/lib/python2.7/site-packages/django/core/management/commands/runserver.py", line 121, in inner_run
    self.check(display_num_errors=True)
  File "/usr/lib/python2.7/site-packages/django/core/management/base.py", line 374, in check
    include_deployment_checks=include_deployment_checks,
  File "/usr/lib/python2.7/site-packages/django/core/management/base.py", line 361, in _run_checks
    return checks.run_checks(**kwargs)
  File "/usr/lib/python2.7/site-packages/django/core/checks/registry.py", line 81, in run_checks
    new_errors = check(app_configs=app_configs)
  File "/usr/lib/python2.7/site-packages/django/core/checks/urls.py", line 14, in check_url_config
    return check_resolver(resolver)
  File "/usr/lib/python2.7/site-packages/django/core/checks/urls.py", line 24, in check_resolver
    for pattern in resolver.url_patterns:
  File "/usr/lib/python2.7/site-packages/django/utils/functional.py", line 35, in __get__
    res = instance.__dict__[self.name] = self.func(instance)
  File "/usr/lib/python2.7/site-packages/django/urls/resolvers.py", line 313, in url_patterns
    patterns = getattr(self.urlconf_module, "urlpatterns", self.urlconf_module)
  File "/usr/lib/python2.7/site-packages/django/utils/functional.py", line 35, in __get__
    res = instance.__dict__[self.name] = self.func(instance)
  File "/usr/lib/python2.7/site-packages/django/urls/resolvers.py", line 306, in urlconf_module
    return import_module(self.urlconf_name)
  File "/usr/lib64/python2.7/importlib/__init__.py", line 37, in import_module
    __import__(name)
  File "/root/python_project/HelloWorld/HelloWorld/urls.py", line 21, in <module>
    url(r'^admin/', admin.site.urls),
NameError: name 'url' is not defined
```

[**`django.conf.urls`**官网](https://docs.djangoproject.com/en/1.10/ref/urls/)

重新检查了一下`urls.py`，发现这个问题可能是和位置有关系，因为代码是从上往下执行的。这个时候还有导入`url`模块。即`from django.conf.urls import url`。要么把导入模块放到上面，要么把需要`url`的这段代码放到下面。

iii.解决`NameError: name 'url' is not defined`
---


更改`urls.py`

```python
[root@master HelloWorld]# cat urls.py
"""HelloWorld URL Configuration

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/1.10/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  url(r'^$', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  url(r'^$', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.conf.urls import url, include
    2. Add a URL to urlpatterns:  url(r'^blog/', include('blog.urls'))
"""
# Origin code
# from django.conf.urls import url
# from django.contrib import admin

# urlpatterns = [
#    url(r'^admin/', admin.site.urls),
# ]

# from django.conf.urls import *
from HelloWorld.view import hello
from django.conf.urls import url
from HelloWorld.testdb import testdb
from HelloWorld import search
from HelloWorld import search2
from django.contrib import admin

urlpatterns = [
    url('^hello/$',hello),
    url('^testdb/$',testdb),
    url(r'^search-form/$',search.search_form),
    url(r'^search/$',search.search),
    url(r'^search-post/$',search2.search_post),
]

urlpatterns = [
    url(r'^admin/', admin.site.urls),
 ]
```

更改完成后服务器变化

```python
  File "/root/python_project/HelloWorld/HelloWorld/urls.py", line 21, in <module>
    url(r'^admin/', admin.site.urls),
NameError: name 'url' is not defined
Performing system checks...

System check identified no issues (0 silenced).
November 26, 2016 - 12:07:36
Django version 1.10.3, using settings 'HelloWorld.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```



当这一切都配置好后，`Django`管理工具就可以运行了。

* * *

II.使用管理工具
===


启动开发服务器，然后在浏览器中访问：`http://yoursite:8000/admin/`，得到如下界面：


访问后服务器的变化

```python
[26/Nov/2016 12:11:09] "GET /admin/ HTTP/1.1" 302 0
[26/Nov/2016 12:11:09] "GET /admin/login/?next=/admin/ HTTP/1.1" 200 1653
[26/Nov/2016 12:11:09] "GET /static/admin/css/login.css HTTP/1.1" 200 1203
[26/Nov/2016 12:11:09] "GET /static/admin/css/base.css HTTP/1.1" 200 15937
[26/Nov/2016 12:11:09] "GET /static/admin/css/fonts.css HTTP/1.1" 200 423
[26/Nov/2016 12:11:09] "GET /static/admin/fonts/Roboto-Regular-webfont.woff HTTP/1.1" 200 80304
[26/Nov/2016 12:11:09] "GET /static/admin/fonts/Roboto-Light-webfont.woff HTTP/1.1" 200 81348
```

![1.10登录界面|center](http://i.imgur.com/KBWYABp.jpg)

![1.10登录界面大图|center](http://i.imgur.com/5tEgLr3.jpg)

![源文档的图|center][1]

你可以通过命令 `python manage.py createsuperuser` 来创建超级用户，如下所示：
    
    
    
    # python manage.py createsuperuser
    Username (leave blank to use 'root'): admin
    Email address: admin@w3cschool.cc
    Password:
    Password (again):
    Superuser created successfully.
    [root@solar HelloWorld]#


i.实际测试创建用户
---


    
```python
[root@master HelloWorld]# python manage.py createsuperuser
Username (leave blank to use 'root'): jason
Email address: 512380548@qq.com
Password: 
Password (again): 
This password is too short. It must contain at least 8 characters.
This password is too common.
This password is entirely numeric.
Password: 
Password (again): 
Superuser created successfully.
```

之后输入用户名密码登录，界面如下：

![原文档][2]

实际测试图

![实际测试图](http://i.imgur.com/cWX6Vjk.jpg)

![](http://i.imgur.com/whWXFth.jpg)

为了让`admin`界面管理某个数据模型，我们需要先注册该数据模型到`admin`。比如，我们之前在 `TestModel` 中已经创建了模型 `Test` 。修改 `TestModel/admin.py`:
    
    
    
    from django.contrib import admin
    from TestModel.models import Test
    
    # Register your models here.
    admin.site.register(Test)
    
实际`TestModel/admin.py`

```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test

# Register your models here.
admin.site.register(Test)
```


刷新后即可看到 `Testmodel` 数据表:

![原文档|center][3]

实际测试图

![实际测试|center](http://i.imgur.com/lKE175Q.jpg)

ii.插入数据
---


表中插入数据，点击`ADD TEST`

![](http://i.imgur.com/hiItZYP.jpg)

插入`Wilson`，并保存

![](http://i.imgur.com/g94vI8A.jpg)

插入成功

![|center](http://i.imgur.com/azYNhXL.jpg)

在`MySQL`数据库中查看

```bash
(root@localhost)(08:57:02) [test]> select * from TestModel_test;
+----+------------+
| id | name       |
+----+------------+
|  1 | JasonBourn |
|  5 | Wilson     |
+----+------------+
2 rows in set (0.00 sec)
```

因为之前测试过，没有具体观察这个`id`的规律。

iii.查看`id`变化规律
---

又插入数据到表中，观察这个`id`变化规律

```bash
(root@localhost)(08:59:45) [test]> select * from TestModel_test;
+----+------------+
| id | name       |
+----+------------+
|  1 | JasonBourn |
|  5 | Wilson     |
|  7 | testid     |
|  9 | the more   |
+----+------------+
4 rows in set (0.00 sec)
```

<font color=red>现在有问题，就是如何实现对所有数据库和表的增删改查的操作？</font>
当前仅仅是针对一个表进程操作。


* * *

III.复杂模型
===


管理页面的功能强大，完全有能力处理更加复杂的数据模型。

先在 `TestModel/models.py` 中增加一个更复杂的数据模型：    
    
    from django.db import models
    
    # Create your models here.
    class Contact(models.Model):
        name   = models.CharField(max_length=200)
        age    = models.IntegerField(default=0)
        email  = models.EmailField()
        def __unicode__(self):
            return self.name
    
    class Tag(models.Model):
        contact = models.ForeignKey(Contact)
        name    = models.CharField(max_length=50)
        def __unicode__(self):
            return self.name

i.实际`models.py`
---

    
```python
[root@master TestModel]# cat models.py
# from __future__ import unicode_literals

# from django.db import models

# Create your models here.

from django.db import models

class Test(models.Model):
    name = models.CharField(max_length=20)

class Contact(models.Model):
    name = models.CharField(max_length=200)
    age = models.IntegerField(default=0)
    email = models.EmailField()
    def __unicode__(self):
        return self.name

class Tag(models.Model):
    contact = models.ForeignKey(Contact)
    name = models.CharField(max_length=50)
    def __unicode__(self):
        return self.name
```

这里有两个表。`Tag`以`Contact`为外部键。一个`Contact`可以对应多个`Tag`。

我们还可以看到许多在之前没有见过的属性类型，比如`IntegerField`用于存储整数。

![][4]

在 `TestModel/admin.py` 注册多个模型并显示：
    
    
    
    from django.contrib import admin
    from TestModel.models import Test,Contact,Tag
    
    # Register your models here.
    admin.site.register([Test, Contact, Tag])
    
ii.实际`TestModel/admin.py` 
---


```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test

# Register your models here.
admin.site.register[(Test, Contact, Tag])
```

服务器的提示

```python
  File "/root/python_project/HelloWorld/TestModel/admin.py", line 5
    admin.site.register[(Test, Contact, Tag])
                                           ^
SyntaxError: invalid syntax
```

iii.发现错误`admin.py` `SyntaxError: invalid syntax`
---


```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test,Contact,Tag

# Register your models here.
admin.site.register[(Test, Contact, Tag])
```

服务器的提示

```python
  File "/root/python_project/HelloWorld/TestModel/admin.py", line 5
    admin.site.register[(Test, Contact, Tag])
                                           ^
SyntaxError: invalid syntax
```

iv.解决`admin.py` `SyntaxError: invalid syntax`问题
---


更改`admin.py`

仔细看了看发现是符号的位置有问题，其实自己没有仔细看错误。当然了到这里发现的是自己有两个错误，一个是导入不全面。另一个就是符号的位置。上面的提示已经非常明确的，就是你的`[]`有问题。

```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test,Contact,Tag

# Register your models here.
admin.site.register([Test, Contact, Tag])
```

服务器的提示变成正常

```python
  File "/root/python_project/HelloWorld/TestModel/admin.py", line 5
    admin.site.register[(Test, Contact, Tag])
                                           ^
SyntaxError: invalid syntax
Performing system checks...

System check identified no issues (0 silenced).
November 26, 2016 - 13:47:28
Django version 1.10.3, using settings 'HelloWorld.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```

刷新管理页面，显示结果如下：

![原文档|center][5]

v.实际测试
---



![实际测试|center](http://i.imgur.com/9WWy9uz.jpg)


![](http://i.imgur.com/4NlQObF.jpg)

```python
 File "/usr/lib64/python2.7/site-packages/MySQLdb/connections.py", line 36, in defaulterrorhandler
    raise errorclass, errorvalue
ProgrammingError: (1146, "Table 'test.TestModel_tag' doesn't exist")
[26/Nov/2016 13:57:32] "GET /admin/TestModel/tag/ HTTP/1.1" 500 164692
```
估计还是需要同步一下数据库。那就没有其他的同步数据库的方式了吗？

### 1.同步数据库


```python
[root@master HelloWorld]# ls
db.sqlite3  HelloWorld  manage.py  templates  TestModel
[root@master HelloWorld]# python manage.py makemigrations
Migrations for 'TestModel':
  TestModel/migrations/0002_contact_tag.py:
    - Create model Contact
    - Create model Tag
[root@master HelloWorld]# python manage.py migrate
Operations to perform:
  Apply all migrations: TestModel, admin, auth, contenttypes, sessions
Running migrations:
  Applying TestModel.0002_contact_tag... OK
```

### 2.访问`contact`


>访问后的界面

![contact|center](http://i.imgur.com/8l1Itj0.jpg)

![](http://i.imgur.com/Mv0BelV.jpg)

>服务器端的提示

```python
[26/Nov/2016 14:18:33] "GET /admin/TestModel/contact/ HTTP/1.1" 200 3059
[26/Nov/2016 14:18:33] "GET /admin/jsi18n/ HTTP/1.1" 200 3217
```

### 3.`contact`插入数据


>插入数据界面

![](http://i.imgur.com/8OaZMhg.jpg)

>从`MySQL`数据库中查看

```bash
(root@localhost)(10:25:57) [test]> show tables;
+----------------------------+
| Tables_in_test             |
+----------------------------+
| TestModel_contact          |
| TestModel_tag              |
| TestModel_test             |
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
+----------------------------+
13 rows in set (0.00 sec)

(root@localhost)(10:25:58) [test]> select * from TestModel_contact;
+----+------+-----+---------------+
| id | name | age | email         |
+----+------+-----+---------------+
|  1 | John |  18 | test@gmai.com |
+----+------+-----+---------------+
1 row in set (0.00 sec)
```

从上面的表中可以看到

- `TestModel_contact`          
- `TestModel_tag` 

是新建立的。不过并没有测试这个是在执行了`python manage.py migrate`之前，还是之后的。我的假设是执行了这个命令之后产生的。因为执行前，在访问的时候，报的错误是没有这个表的。


### 4.访问`tag`


访问`tag`界面

![|center](http://i.imgur.com/go0BvFH.jpg)

```python
[26/Nov/2016 14:21:29] "GET /admin/TestModel/tag/ HTTP/1.1" 200 3031
[26/Nov/2016 14:21:29] "GET /admin/jsi18n/ HTTP/1.1" 200 3217
```


### 5.`tag` 插入数据测试


>`tag` 插入数据界面

![](http://i.imgur.com/5ZVupb7.jpg)

>从`MySQL`中查看

```bash
(root@localhost)(10:29:37) [test]> select * from TestModel_tag;
+----+------+------------+
| id | name | contact_id |
+----+------+------------+
|  1 | SRE  |          1 |
+----+------+------------+
1 row in set (0.00 sec)
```

>服务器的提示

```python
[26/Nov/2016 14:29:48] "POST /admin/TestModel/tag/add/ HTTP/1.1" 302 0
```

在以上管理工具我们就能进行复杂模型操作。

* * *

IV.自定义表单
===


我们可以自定义管理页面，来取代默认的页面。比如上面的"add"页面。我们想只显示`name`和`email`部分。修改 `TestModel/admin.py`:
        
    
    from django.contrib import admin
    from TestModel.models import Test,Contact,Tag
    
    # Register your models here.
    class ContactAdmin(admin.ModelAdmin):
        fields = ('name', 'email')
    
    admin.site.register(Contact, ContactAdmin)
    admin.site.register([Test, Tag])
    
i.实际代码`admin.py`
---

```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test,Contact,Tag

# Register your models here.
# admin.site.register([Test, Contact, Tag])
class ContactAdmin(admin.ModelAdmin):
    fields = ('name','email')

admin.site.register(Contact,ContactAdmin)
admin.site.register([Test,Tag])
```

```python
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
Performing system checks...

System check identified no issues (0 silenced).
November 27, 2016 - 00:16:22
Django version 1.10.3, using settings 'HelloWorld.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
Performing system checks...

System check identified no issues (0 silenced).
November 27, 2016 - 00:16:54
Django version 1.10.3, using settings 'HelloWorld.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```
服务器端检测没有问题


以上代码定义了一个`ContactAdmin`类，用以说明管理页面的显示格式。

里面的`fields`属性定义了要显示的字段。

由于该类对应的是`Contact`数据模型，我们在注册的时候，需要将它们一起注册。显示效果如下：

![源文档][6]

ii.访问`contact`测试
---


![实际测试](http://i.imgur.com/TyTgB5J.jpg)

iii.`contact`插入数据测试
---


![](http://i.imgur.com/J1Nc3uT.jpg)

>插入数据成功

![|center](http://i.imgur.com/wyyQpam.jpg)

### 查看`MySQL`


```bash
(root@localhost)(08:29:01) [test]> select * from TestModel_contact;
+----+------+-----+----------------+
| id | name | age | email          |
+----+------+-----+----------------+
|  1 | John |  18 | test@gmail.com |
|  3 | rose |   0 | rose@gmail.com |
+----+------+-----+----------------+
2 rows in set (0.00 sec)
```

我们还可以将输入栏分块，每个栏也可以定义自己的格式。修改`TestModel/admin.py`为：
    
    
    
    from django.contrib import admin
    from TestModel.models import Test,Contact,Tag
    
    # Register your models here.
    class ContactAdmin(admin.ModelAdmin):
        fieldsets = (
            ['Main',{
                'fields':('name','email'),
            }],
            ['Advance',{
                'classes': ('collapse',), # CSS
                'fields': ('age',),
            }]
        )
    
    admin.site.register(Contact, ContactAdmin)
    admin.site.register([Test, Tag])
    
iv.输入栏分块--实际代码`admin.py`
---


```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test,Contact,Tag

# Register your models here.
# admin.site.register([Test, Contact, Tag])

# class ContactAdmin(admin.ModelAdmin):
#    fields = ('name','email')

class ContactAdmin(admin.ModelAdmin):
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',),
            'fields': ('age',),
        }]
    )

admin.site.register(Contact,ContactAdmin)
admin.site.register([Test,Tag])
```


上面的栏目分为了`Main`和`Advance`两部分。`classes`说明它所在的部分的`CSS`格式。这里让`Advance`部分隐藏：

![原文档][7]

v.输入栏分块实际测试--`show`
---


![|center](http://i.imgur.com/u1nTiMn.jpg)


`Advance`部分旁边有一个 `Show` 按钮，用于展开，展开后可点击 `Hide` 将其隐藏，如下图所示：

![原文档][8]

vi.输入栏分块实际测试--`hide`
---


![实际测试|center](http://i.imgur.com/h4x4tRB.jpg)

### 1.插入数据测试


![|center](http://i.imgur.com/UqrBJ4x.jpg)

![|center](http://i.imgur.com/TnUjfBd.jpg)

### 2.查看`MySQL`


```python
(root@localhost)(08:29:10) [test]> select * from TestModel_contact;
+----+--------------+-----+------------------+
| id | name         | age | email            |
+----+--------------+-----+------------------+
|  1 | John         |  18 | test@gmail.com   |
|  3 | rose         |   0 | rose@gmail.com   |
|  5 | blockSeprate |  20 | blocak@gmail.com |
+----+--------------+-----+------------------+
3 rows in set (0.01 sec)
```

* * *

V.内联(`Inline`)显示
===


上面的`Contact`是`Tag`的外部键，所以有外部参考的关系。

而在默认的页面显示中，将两者分离开来，无法体现出两者的从属关系。我们可以使用内联显示，让`Tag`附加在`Contact`的编辑页面上显示。

修改`TestModel/admin.py`：
    
    
    
    from django.contrib import admin
    from TestModel.models import Test,Contact,Tag
    
    # Register your models here.
    class TagInline(admin.TabularInline):
        model = Tag
    
    class ContactAdmin(admin.ModelAdmin):
        inlines = [TagInline]  # Inline
        fieldsets = (
            ['Main',{
                'fields':('name','email'),
            }],
            ['Advance',{
                'classes': ('collapse',),
                'fields': ('age',),
            }]
    
        )
    
    admin.site.register(Contact, ContactAdmin)
    admin.site.register([Test])

i.实际代码`admin.py`
---


```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test,Contact,Tag

# Register your models here.
# admin.site.register([Test, Contact, Tag])

# class ContactAdmin(admin.ModelAdmin):
#    fields = ('name','email')

# custome form
# class ContactAdmin(admin.ModelAdmin):
#    fieldsets = (
#        ['Main',{
#            'fields':('name','email'),
#        }],
#        ['Advance',{
#            'classes': ('collapse',),
#            'fields': ('age',),
#        }]
#    )

class TagInline(admin.TabularInline):
    model = Tag

class ConatactAdmin(admin.ModelAdmin):
    inlines = [TagInline]
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',),
            'fields': ('age',),
        }]

    )

admin.site.register(Contact,ContactAdmin)
admin.site.register([Test,Tag])
```
    
服务器提示

```python
  File "/root/python_project/HelloWorld/TestModel/admin.py", line 38, in <module>
    admin.site.register(Contact,ContactAdmin)
NameError: name 'ContactAdmin' is not defined
```

更改`admin.py`

```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test,Contact,Tag

# Register your models here.
# admin.site.register([Test, Contact, Tag])

# class ContactAdmin(admin.ModelAdmin):
#    fields = ('name','email')

# custome form
# class ContactAdmin(admin.ModelAdmin):
#    fieldsets = (
#        ['Main',{
#            'fields':('name','email'),
#        }],
#        ['Advance',{
#            'classes': ('collapse',),
#            'fields': ('age',),
#        }]
#    )

class TagInline(admin.TabularInline):
    model = Tag

class ContactAdmin(admin.ModelAdmin):
    inlines = [TagInline]
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',),
            'fields': ('age',),
        }]

    )

admin.site.register(Contact,ContactAdmin)
admin.site.register([Test,Tag])
```
是在定义`ContactAdmin`这个类的时候，拼写出现错误。

显示效果如下：

![原文档][9]

![小图|center](http://i.imgur.com/nXGLthZ.jpg)

![](http://i.imgur.com/z4hAr28.jpg)

服务器端提示

```python
[27/Nov/2016 01:03:14] "GET /static/admin/js/inlines.js HTTP/1.1" 200 13404
```

ii.使用内联插入数据
---


![](http://i.imgur.com/QqIlHNz.jpg)

内联插入数据成功

![](http://i.imgur.com/cVTUMQT.jpg)

iii.从`MySQL`中查看插入的数据
---


```bash
(root@localhost)(08:45:03) [test]> select * from TestModel_contact;
+----+--------------+-----+------------------+
| id | name         | age | email            |
+----+--------------+-----+------------------+
|  1 | John         |  18 | test@gmail.com   |
|  3 | rose         |   0 | rose@gmail.com   |
|  5 | blockSeprate |  20 | blocak@gmail.com |
|  7 | Jack         |  21 | jack@163.com     |
+----+--------------+-----+------------------+
4 rows in set (0.01 sec)
```


```bash
(root@localhost)(09:08:50) [test]> select * from TestModel_tag;
+----+-------+------------+
| id | name  | contact_id |
+----+-------+------------+
|  1 | SRE   |          1 |
|  3 | jack  |          7 |
|  5 | email |          7 |
|  7 | age   |          7 |
+----+-------+------------+
4 rows in set (0.00 sec)
```


* * *

VI.列表页的显示
===


在`Contact`输入数条记录后，`Contact`的列表页看起来如下:

![][10]

我们也可以自定义该页面的显示，比如在列表中显示更多的栏目，只需要在`ContactAdmin`中增加`list_display`属性:
    
    
    
    from django.contrib import admin
    from TestModel.models import Test,Contact,Tag
    
    # Register your models here.
    class ContactAdmin(admin.ModelAdmin):
        list_display = ('name','age', 'email') # list
    
    admin.site.register(Contact, ContactAdmin)
    admin.site.register([Test, Tag])
    
i.列表显示实际代码`admin.py`
---


```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test,Contact,Tag

# Register your models here.
# admin.site.register([Test, Contact, Tag])

# class ContactAdmin(admin.ModelAdmin):
#    fields = ('name','email')

# custome form
# class ContactAdmin(admin.ModelAdmin):
#    fieldsets = (
#        ['Main',{
#            'fields':('name','email'),
#        }],
#        ['Advance',{
#            'classes': ('collapse',),
#            'fields': ('age',),
#        }]
#    )

class TagInline(admin.TabularInline):
    model = Tag

class ContactAdmin(admin.ModelAdmin):
    list_display = ('name','age','email')
    inlines = [TagInline]
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',),
            'fields': ('age',),
        }]

    )

admin.site.register(Contact,ContactAdmin)
admin.site.register([Test,Tag])
```

刷新页面显示效果如下：

![原文档][11]

ii.列表显示实际测试
---


![实际测试](http://i.imgur.com/pVSRusc.jpg)


搜索功能在管理大量记录时非常有，我们可以使用`search_fields`为该列表页增加搜索栏：
    
    
    
    from django.contrib import admin
    from TestModel.models import Test,Contact,Tag
    
    # Register your models here.
    class ContactAdmin(admin.ModelAdmin):
        list_display = ('name','age', 'email') 
        search_fields = ('name',)
    
    admin.site.register(Contact, ContactAdmin)
    admin.site.register([Test])
    
iii.搜索栏实际代码`admin.py`
---



```python
[root@master TestModel]# cat admin.py
from django.contrib import admin
from TestModel.models import Test,Contact,Tag

# Register your models here.
# admin.site.register([Test, Contact, Tag])

# class ContactAdmin(admin.ModelAdmin):
#    fields = ('name','email')

# custome form
# class ContactAdmin(admin.ModelAdmin):
#    fieldsets = (
#        ['Main',{
#            'fields':('name','email'),
#        }],
#        ['Advance',{
#            'classes': ('collapse',),
#            'fields': ('age',),
#        }]
#    )

class TagInline(admin.TabularInline):
    model = Tag

class ContactAdmin(admin.ModelAdmin):
    list_display = ('name','age','email')
    search_fields = ('name',)
    inlines = [TagInline]
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',),
            'fields': ('age',),
        }]

    )

admin.site.register(Contact,ContactAdmin)
admin.site.register([Test,Tag])
```


在本实例中我们搜索了 `name` 为 `w3cschool.cc`(本站域名) 的记录，显示结果如下：

![原文档][12]

iv.搜索栏实际测试
---


![](http://i.imgur.com/3JJok9Y.jpg)

>刷新动作带来的变化

```python
[27/Nov/2016 01:17:07] "GET /static/admin/img/search.svg HTTP/1.1" 304 0
[27/Nov/2016 01:17:07] "GET /admin/jsi18n/ HTTP/1.1" 200 3217
```

### 搜索`rose`


![](http://i.imgur.com/FglxNhd.jpg)

`Django Admin` 管理工具还有非常多实用的功能，感兴趣的同学可以深入研究下。



[1]: http://www.runoob.com/wp-content/uploads/2015/01/admin1.png
[2]: http://www.runoob.com/wp-content/uploads/2015/01/admin2.jpg
[3]: http://www.runoob.com/wp-content/uploads/2015/01/admin3.jpg
[4]: http://www.runoob.com/wp-content/uploads/2015/01/admin4.png
[5]: http://www.runoob.com/wp-content/uploads/2015/01/admin5.jpg
[6]: http://www.runoob.com/wp-content/uploads/2015/01/admin6.jpg
[7]: http://www.runoob.com/wp-content/uploads/2015/01/admin7.jpg
[8]: http://www.runoob.com/wp-content/uploads/2015/01/admin8.jpg
[9]: http://www.runoob.com/wp-content/uploads/2015/01/admin9.jpg
[10]: http://www.runoob.com/wp-content/uploads/2015/01/admin10.jpg
[11]: http://www.runoob.com/wp-content/uploads/2015/01/admin11.jpg
[12]: http://www.runoob.com/wp-content/uploads/2015/01/admin12.jpg

  



---

