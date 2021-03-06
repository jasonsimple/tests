﻿# <center>Django 创建第一个项目--测试</center>

---

### <center>目录</center>

[toc]

---

[**`Source`**](http://www.runoob.com/django/django-first-app.html "Permalink to Django 创建第一个项目 | 菜鸟教程")


本章我们将介绍`Django` 管理工具及如何使用 `Django` 来创建项目，第一个项目我们以 `HelloWorld` 来命令项目。

* * *

I.`Django`管理工具
===
[**回目录**](#目录)

安装 `Django` 之后，您现在应该已经有了可用的管理工具 `django-admin.py`。我们可以使用 `django-admin.py` 来创建一个项目:

<font color=red>注意</font>:这里第一眼看上去，还以为这里的`django-admin.py`是一个脚本，但事实上这里它是一个命令

我们可以来看下`django-admin.py`的命令介绍:
    
```bash
[root@master python_project]# ls
HelloWorld  HeloWorld  settings.py
```
可以看到当前目录下面并没有`django-admin.py`这个脚本，但是当你敲这个命令的时候是可以自动补齐的。

```bash
[root@master python_project]# django-admin.py 

Type 'django-admin.py help <subcommand>' for help on a specific subcommand.

Available subcommands:

[django]
    check
    compilemessages
    createcachetable
    dbshell
    diffsettings
    dumpdata
    flush
    inspectdb
    loaddata
    makemessages
    makemigrations
    migrate
    runserver
    sendtestemail
    shell
    showmigrations
    sqlflush
    sqlmigrate
    sqlsequencereset
    squashmigrations
    startapp
    startproject
    test
    testserver
Note that only Django core commands are listed as settings are not properly configured (error: Requested setting INSTALLED_APPS, but settings are not configured. You must either define the environment variable DJANGO_SETTINGS_MODULE or call settings.configure() before accessing settings.).
```

这个命令并不是只有在这一个目录下才有，而是可以作为全局变量来调用。

i.使用`which`查看`django-admin.py`命令
---
[**回目录**](#目录)

```bash
[root@master python_project]# which django-admin.py
/usr/bin/django-admin.py
```

ii.查看`django-admin.py`
---
[**回目录**](#目录)

```bash
[root@master python_project]# cat /usr/bin/django-admin.py
#!/usr/bin/python
from django.core import management

if __name__ == "__main__":
    management.execute_from_command_line()
```


---

II.创建第一个项目
===
[**回目录**](#目录)

i.创建项目
---
[**回目录**](#目录)

使用 `django-admin.py` 来创建 `HelloWorld` 项目：
    
    
    
    django-admin.py startproject HelloWorld
    
### 1.创建`HelloWorld` 项目之前
[**回目录**](#目录)

```bash
[root@master python_project]# ls
settings.py
```

### 2.创建`HelloWorld` 项目
[**回目录**](#目录)

```bash
[root@master python_project]# django-admin.py startproject HelloWorld
```

### 3.创建`HelloWorld` 项目之后
[**回目录**](#目录)

```bash
[root@master python_project]# ls
HelloWorld  settings.py
[root@master python_project]# tree HelloWorld/
HelloWorld/
├── HelloWorld
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

1 directory, 5 files
```

<font color=red>注意</font>：受过去影响，就是“所有代码都是自己写出来的"这种观念的影响。一开始还以为目录以及目录里面内容都需要自己来写。而操作之后发现，当你执行了
`django-admin.py startproject HelloWorld`这个命令之后，就会自动生成`HelloWorld`目录以及下面的内容。


ii.目录说明
---
[**回目录**](#目录)

- **`HelloWorld`:** 项目的容器。
- **`manage.py`:** 一个实用的命令行工具，可让你以各种方式与该 `Django` 项目进行交互。 
- **`HelloWorld/__init__.py`:** 一个空文件，告诉 `Python` 该目录是一个 `Python` 包。
- **`HelloWorld/settings.py`:** 该 `Django` 项目的设置/配置。
- **`HelloWorld/urls.py`:** 该 `Django` 项目的 `URL` 声明; 一份由 `Django` 驱动的网站"目录"。
- **`HelloWorld/wsgi.py`:** 一个 `WSGI` 兼容的 `Web` 服务器的入口，以便运行你的项目。


iii.查看目录中的代码具体内容
---
[**回目录**](#目录)

### 1.查看`manage.py`
[**回目录**](#目录)

```python
[root@master HelloWorld]# cat manage.py 
#!/usr/bin/env python
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "HelloWorld.settings")
    try:
        from django.core.management import execute_from_command_line
    except ImportError:
        # The above import may fail for some other reason. Ensure that the
        # issue is really that Django is missing to avoid masking other
        # exceptions on Python 2.
        try:
            import django
        except ImportError:
            raise ImportError(
                "Couldn't import Django. Are you sure it's installed and "
                "available on your PYTHONPATH environment variable? Did you "
                "forget to activate a virtual environment?"
            )
        raise
    execute_from_command_line(sys.argv)
```

```bash
[root@master HelloWorld]# cat __init__.py 
[root@master HelloWorld]# 
```

### 2.查看`settings.py`
[**回目录**](#目录)

```python
[root@master HelloWorld]# cat settings.py 
"""
Django settings for HelloWorld project.

Generated by 'django-admin startproject' using Django 1.10.3.

For more information on this file, see
https://docs.djangoproject.com/en/1.10/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/1.10/ref/settings/
"""

import os

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/1.10/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'f9onn^p_ngdj62*ja-yd7-wp7ec_q&w3&x00y-g@3qru034ocu'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = []


# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'HelloWorld.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'HelloWorld.wsgi.application'


# Database
# https://docs.djangoproject.com/en/1.10/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}


# Password validation
# https://docs.djangoproject.com/en/1.10/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/1.10/topics/i18n/

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'UTC'

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.10/howto/static-files/

STATIC_URL = '/static/'
```

### 3.查看`urls.py`
[**回目录**](#目录)

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
from django.conf.urls import url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
]
```

### 4.查看`wsgi.py`
[**回目录**](#目录)

```python
[root@master HelloWorld]# cat wsgi.py 
"""
WSGI config for HelloWorld project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/1.10/howto/deployment/wsgi/
"""

import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "HelloWorld.settings")

application = get_wsgi_application()
```
    
iv.启动服务器
---
[**回目录**](#目录)

接下来我们进入 `HelloWorld` 目录输入以下命令，启动服务器：
            
    python manage.py runserver 0.0.0.0:8000
    

0.0.0.0让其它电脑可连接到开发服务器，8000为端口号。如果不说明，那么端口号默认为8000。

在浏览器输入你服务器的ip及端口号，如果正常启动，输出结果如下：

![python][1]

### 1.实际测试
[**回目录**](#目录)

```bash
[root@master HelloWorld]# ls
HelloWorld  manage.py
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
Performing system checks...

System check identified no issues (0 silenced).

You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

November 25, 2016 - 08:33:27
Django version 1.10.3, using settings 'HelloWorld.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```

访问服务器报错提示

![](http://i.imgur.com/GxrxTMc.jpg)

```python
DisallowedHost at /
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
Request Method:	GET
Request URL:	http://192.168.142.32:8000/
Django Version:	1.10.3
Exception Type:	DisallowedHost
Exception Value:	
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
Exception Location:	/usr/lib/python2.7/site-packages/django/http/request.py in get_host, line 113
Python Executable:	/usr/bin/python
Python Version:	2.7.5
Python Path:	
['/root/python_project/HelloWorld',
 '/usr/lib64/python27.zip',
 '/usr/lib64/python2.7',
 '/usr/lib64/python2.7/plat-linux2',
 '/usr/lib64/python2.7/lib-tk',
 '/usr/lib64/python2.7/lib-old',
 '/usr/lib64/python2.7/lib-dynload',
 '/usr/lib64/python2.7/site-packages',
 '/usr/lib64/python2.7/site-packages/gtk-2.0',
 '/usr/lib/python2.7/site-packages']
Server time:	Fri, 25 Nov 2016 08:34:42 +0000
```

访问了两次之后在`CentOS7`上面新增加的内容

```python
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
[25/Nov/2016 08:34:42] "GET / HTTP/1.1" 400 59517
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
[25/Nov/2016 08:34:42] "GET /favicon.ico HTTP/1.1" 400 60254
```

### 2.解决问题
[**回目录**](#目录)

忽略了一个地方，就是启动之后

```python
You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
```

### 2.1解决`migrate`问题
[**回目录**](#目录)

这里已经提示你了解决办法

	python manage.py migrate

```python
[root@master HelloWorld]# python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK
```

再次启动

```python
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
Performing system checks...

System check identified no issues (0 silenced).
November 25, 2016 - 08:51:20
Django version 1.10.3, using settings 'HelloWorld.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
[25/Nov/2016 08:51:28] "GET / HTTP/1.1" 400 60582
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
[25/Nov/2016 08:51:28] "GET /favicon.ico HTTP/1.1" 400 60535
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
[25/Nov/2016 08:52:17] "GET / HTTP/1.1" 400 60389
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
[25/Nov/2016 08:52:18] "GET /favicon.ico HTTP/1.1" 400 60080
Invalid HTTP_HOST header: '192.168.142.32:8000'. You may need to add u'192.168.142.32' to ALLOWED_HOSTS.
[25/Nov/2016 08:52:18] "GET /favicon.ico HTTP/1.1" 400 60154
```
这个时候没有了上面的`migrate`提示，但是依旧是无法访问网页。

### 2.2解决`ALLOWED_HOSTS`
[**回目录**](#目录)

解决办法，搜素引擎帮助我找到了官网。
[`ALLOWED_HOSTS`官网解释](https://docs.djangoproject.com/en/dev/ref/settings/#allowed-hosts)

>然后从官网上看到
Changed in Django Development version:
In older versions, ALLOWED_HOSTS wasn’t checked when running tests.
In older versions, ALLOWED_HOSTS wasn’t checked if DEBUG=True. This was also changed in Django `1.10.3`, 1.9.11, and 1.8.16 to prevent a DNS rebinding attack.

<br/>

>然后自己在报错的页面使用`Ctrl+F`搜索`ALLOWED_HOSTS`

搜索结果图

![](http://i.imgur.com/DkSeQWx.jpg)


中间改了一次但是自己没有在地址两边加`''`，即让它成为字符串，会报这个错误

```python
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
Traceback (most recent call last):
  File "manage.py", line 22, in <module>
    execute_from_command_line(sys.argv)
  File "/usr/lib/python2.7/site-packages/django/core/management/__init__.py", line 367, in execute_from_command_line
    utility.execute()
  File "/usr/lib/python2.7/site-packages/django/core/management/__init__.py", line 316, in execute
    settings.INSTALLED_APPS
  File "/usr/lib/python2.7/site-packages/django/conf/__init__.py", line 53, in __getattr__
    self._setup(name)
  File "/usr/lib/python2.7/site-packages/django/conf/__init__.py", line 41, in _setup
    self._wrapped = Settings(settings_module)
  File "/usr/lib/python2.7/site-packages/django/conf/__init__.py", line 97, in __init__
    mod = importlib.import_module(self.SETTINGS_MODULE)
  File "/usr/lib64/python2.7/importlib/__init__.py", line 37, in import_module
    __import__(name)
  File "/root/python_project/HelloWorld/HelloWorld/settings.py", line 28
    ALLOWED_HOSTS = [192.168.142.32]
                               ^
SyntaxError: invalid syntax
```

更改之后

```python
[root@master HelloWorld]# grep ALLOWED_HOSTS settings.py
ALLOWED_HOSTS = ['192.168.142.32']
```

再次访问

![](http://i.imgur.com/Zxx7PTv.jpg)

查看服务

```python
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
Performing system checks...

System check identified no issues (0 silenced).
November 25, 2016 - 09:01:21
Django version 1.10.3, using settings 'HelloWorld.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
[25/Nov/2016 09:01:24] "GET / HTTP/1.1" 200 1767
Not Found: /favicon.ico
[25/Nov/2016 09:01:25] "GET /favicon.ico HTTP/1.1" 404 1945
[25/Nov/2016 09:13:53] "GET / HTTP/1.1" 200 1767
[25/Nov/2016 09:14:11] "GET / HTTP/1.1" 200 1767
[25/Nov/2016 09:14:27] "GET / HTTP/1.1" 200 1767
```


抓包


![抓包|center](http://i.imgur.com/sm5KrGs.jpg)

### 3.测试
[**回目录**](#目录)

`DEBUG` = `False`
```bash
[root@master HelloWorld]# grep DEBUG settings.py
DEBUG = False
```

```bash
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
CommandError: You must set settings.ALLOWED_HOSTS if DEBUG is False.
```
不开启`DEBUG`会提示需要设置`settings.ALLOWED_HOSTS`。

注释`DEBUG`

```bash
[root@master HelloWorld]# grep DEBUG settings.py
# DEBUG = True
```

```bash
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
CommandError: You must set settings.ALLOWED_HOSTS if DEBUG is False.
```
和上面的提示是一样的。

注释掉，但是写地址

```python
# DEBUG = True

ALLOWED_HOSTS = ['192.168.142.32']
```

```python
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
Performing system checks...

System check identified no issues (0 silenced).
November 25, 2016 - 09:22:50
Django version 1.10.3, using settings 'HelloWorld.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
[25/Nov/2016 09:22:55] "GET / HTTP/1.1" 404 74
```

![](http://i.imgur.com/scmEoJr.jpg)

这个时候的报错是`404`

---


III.视图和`URL` 配置
===
[**回目录**](#目录)

i.原文档
---
[**回目录**](#目录)

在先前创建的 `HelloWorld` 目录下的 `HelloWorld` 目录新建一个 `view.py` 文件，并输入代码：
    
    
    
    from django.http import HttpResponse
    
    def hello(request):
    	return HttpResponse("Hello world ! ")
    

接着，绑定 `URL` 与视图函数。打开 `urls.py` 文件，删除原来代码，将以下代码复制粘贴到 `urls.py` 文件中：
    
    
    
    from django.conf.urls import *
    from HelloWorld.view import hello
    
    urlpatterns = patterns("",
    	('^hello/$', hello),
    )
 
 整个目录结构如下：
    
    
    
    [root@solar HelloWorld]# tree
    .
    |-- HelloWorld
    |   |-- __init__.py
    |   |-- __init__.pyc
    |   |-- settings.py
    |   |-- settings.pyc
    |   |-- urls.py              # url 配置
    |   |-- urls.pyc
    |   |-- view.py              # 添加的视图文件
    |   |-- view.pyc             # 编译后的视图文件
    |   |-- wsgi.py
    |   `-- wsgi.pyc
    `-- manage.py
    

完成后，启动 Django 开发服务器，并在浏览器访问打开浏览器并访问：

![python-helloworld][2]

**注意：**项目中如果代码有改动，服务器会自动监测代码的改动并自动重新载入，所以如果你已经启动了服务器则不需手动重启。


ii.实际测试
---

实际按着这个测试是不成功的会提示：

```bash
[root@master HelloWorld]# python manage.py runserver 0.0.0.0:8000
Performing system checks...

Unhandled exception in thread started by <function wrapper at 0x1c45aa0>
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
  File "/root/python_project/HelloWorld/HelloWorld/urls.py", line 27, in <module>
    urlpatterns = patterns("",
NameError: name 'patterns' is not defined
```

也是将`NameError: name 'patterns' is not defined`作为关键词作为检索

![](http://i.imgur.com/CqFQBZZ.jpg)


[**`1.10`**官方文档](https://docs.djangoproject.com/ja/1.10/releases/1.8/)

![](http://i.imgur.com/Dx7o58X.jpg)

其实在上面的文档是要删除，而在具体的实践过程中是注释掉了。在出现问题的时候也是看了一下原来的代码。原来的代码中`urlpatterns`是列表，文档更改之后是元组。不是元组，应该是方法，按照文档中的建议，应该是`*`中包含了`patterns`方法。但是实际操作中发现这个是没有的。

### 1.最后实现的代码
[**回目录**](#目录)

`view.py`

```python
[root@master HelloWorld]# cat view.py
from django.http import HttpResponse

def hello(request):
	return HttpResponse("Hello world ! ")
```

`urls.py`

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
#from django.conf.urls import url
# from django.contrib import admin

# urlpatterns = [
#    url(r'^admin/', admin.site.urls),
# ]

# from django.conf.urls import *
from HelloWorld.view import hello
from django.conf.urls import url

urlpatterns = [
    url('^hello/$',hello),
]
```
访问结果

![](http://i.imgur.com/IeWyRvk.jpg)

![|center](http://i.imgur.com/NUknHPy.jpg)

### 2.更改网页显示内容
[**回目录**](#目录)

```python
[root@master HelloWorld]# cat view.py
from django.http import HttpResponse

def hello(request):
	return HttpResponse("Complete the first project! That's Awesome! ")
```

![|center](http://i.imgur.com/yRhj5l8.jpg)



[1]: http://www.runoob.com/wp-content/uploads/2015/01/python.jpg
[2]: http://www.runoob.com/wp-content/uploads/2015/01/python-helloworld.jpg

  </subcommand>


[**回目录**](#目录)


---

