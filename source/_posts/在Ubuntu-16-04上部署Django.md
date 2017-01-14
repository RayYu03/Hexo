---
title: 在Ubuntu 16.04上部署Django
date: 2016-12-13 12:46:08
tags: ['Django', '部署']
---
为了提高 Django 应用的安全和性能，在生产环境非常有必要使用 Apache 或 Nginx，即使它本身就可以启动运行。

本帖介绍怎么在 Python 虚拟环境中安装配置 Django，并和 Apache 集成。

Django 集成到 Apache 有两种方式：python_mod 和 wsgi，后者相对于前者更加稳定，这里我们通过wsgi的方式来进行集成 mod_wsgi 的 Apache 的一个模块，Apache 可以通过 WSGI 接口和 Django 沟通。

<!-- more -->

## 安装配置Django

以[ONE](https://github.com/RayYu03/ONE)这个项目为例, 本文适用于 Python.5。

首先 clone 该项目：
```
$ git clone https://github.com/RayYu03/ONE
$ cd ONE
```

安装Python虚拟环境：
```Python
$ sudo pip3 install virtualenv
```

创建虚拟环境：
```Python
$ virtualenv venv
```

激活新虚拟环境：
```Python
$ source venv/bin/activate
```

安装 requirements:

```Python
(venv)$ pip3 install -r requirements.txt
```

初始化并数据库：
```Python
(venv)$ python manage.py makemigrations
(venv)$ python manage.py migrate
```

为项目创建管理员用户：
```Python
(venv)$ python manage.py createsuperuser
```

配置静态文件目录，在 ONE/settings.py 文件尾修改`STATIC_ROOT`：
```
STATIC_ROOT = 'ONE/static'
```
可以自己选择目录，按照路径创建文件夹

把静态文件放到static目录：
```Python
(venv)$ python manage.py collectstatic
```
打开8000端口：
```
$ sudo ufw allow 8000
```

测试Django项目，启动server：
```Python
(venv)$ python manage.py runserver 0.0.0.0:8000
```

使用浏览器访问：http://server_domain_or_IP:8000，你应该能看到如下页面：
![](https://ws3.sinaimg.cn/large/647dc635jw1fap2xkn0suj20pt07qgm3.jpg)

使用浏览器访问：http://server_domain_or_IP:8000/admin，进入管理员登录界面：
![](https://ws3.sinaimg.cn/large/647dc635jw1fap2yv9jnsj20p00az0sz.jpg)

Ctrl+C 终止 Django 应用，然后退出 Python 虚拟环境。
```Python
(venv)$ deactivate
```

## 配置Apache

现在你应该有了一个可以正常工作的 Django 项目了，接下来配置 Apache 做为它的前端。

首先安装 `apache2` 和 `mod_wsgi`：
```
$ sudo apt-get install Python-pip apache2 libapache2-mod-wsgi-py3
```

编辑默认 Virtual Host 配置文件：
```
$ sudo vim /etc/apache2/sites-available/000-default.conf
```

所有 static 的请求映射到 Django 项目的 /static 目录，在 VirtualHost 块中添加：
```
    Alias /static /home/rayyu/myproject/static
    <Directory /home/rayyu/myproject/static>
        Require all granted
    </Directory>
```

配置 apache 有访问项目目录中 wsgi.py 的权限：

```
    <Directory /home/rayyu/myproject/myproject>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
```

Django 建议使用 daemon 模式运行 WSGI 进程，配置 WSGIDaemonProcess：
```
WSGIDaemonProcess myproject python-home=/home/rayyu/mynewenv python-path=/home/rayyu/myproject
    WSGIProcessGroup myproject
    WSGIScriptAlias / /home/rayyu/myproject/myproject/wsgi.py
```
注意替换上面的Python虚拟环境路径和项目路径。

完整示例：
```
    Alias /static /home/ubuntu/ONE/static
    <Directory /home/ubuntu/ONE/static>
        Require all granted
    </Directory>

    <Directory /home/ubuntu/ONE/ONE>
        <Files wsgi.py>
         Require all granted
        </Files>
    </Directory>

    WSGIDaemonProcess ONE python-home=/home/ubuntu/ONE/venv python-path=/home/ubuntu/ONE
    WSGIProcessGroup ONE
    WSGIScriptAlias / /home/ubuntu/ONE/ONE/wsgi.py
```

修正一些目录和文件权限：
```
$ chmod 664 ~/myproject/db.sqlite3
$ sudo chown :www-data ~/myproject/db.sqlite3
$ sudo chown :www-data ~/myproject
```
如果你配置了防火墙，开启80、443端口：

```
$ sudo ufw delete allow 8000
$ sudo ufw allow 'Apache Full'
```

检查Apache配置文件是否有语法错误：
```
$ sudo apache2ctl configtest
```

如果没有语法错误，重启Apache：
```
$ sudo systemctl restart apache2
```

现在使用浏览器访问 http://your_doamin_or_IP/admin/ 测试 Django 部署是否成功。

到此，完成Django-Apache的部署。
