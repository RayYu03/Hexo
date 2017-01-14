---
title: 在Ubuntu 14.04上部署Flask
date: 2016-12-31 15:46:33
tags: ['Flask', '部署']
---

最近正好遇上阿里云搞活动，9元体验半年云服务，于是就入手了一个阿里云`ESC`。

但是阿里云上的系统版本都较老，没有腾讯云那么新，就挑了一个最新版本的`Ubuntu`系统，Ubuntu 14.04 64位。

对，你没有看错，就是 14.04！！！！

<!-- more -->

# Flask 项目

阿里云上的系统虽然版本较老，但 git, Python2/3 都已经内置啦。

以我最近的课程项目[MovieRental](https://github.com/RayYu03/MovieRental)为例。（ps：这个项目很水，用了三个下午搞好的。所以不要在意太多细节啦。。）


首先从 gayhub 上 clone 该项目：

```
$ cd /home
$ git clone https://github.com/RayYu03/MovieRental
```

安装并激活虚拟环境：

```
$ cd MovieRental
$ python3 -m virtualenv
$ source venv/bin/activate
```


安装依赖：

```
(venv) $ pip3 install -r requirements
```

然后 deploy 该项目：（主要是数据库初始化）


```
(venv) $ python3 manage.py deploy
```


如果需要体验完整效果，则需要 fetch 数据到数据库：

```
(venv) $ python3 fetch.py
```
测试：

```
(venv) $ python3 manage.py runserver

* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
* Restarting with stat
* Debugger is active!
* Debugger pin code: 165-870-438

```


另开一个`shell`：

```Python
$ pip3 install httpie
....success!
$ http --json --auth : GET http://127.0.0.1:5000/api/v1/movies/1

HTTP/1.0 200 OK
Content-Length: 671
Content-Type: application/json
Date: Sat, 31 Dec 2016 08:09:44 GMT
Server: Werkzeug/0.11.11 Python/3.5.2

{
    "alt": "http://127.0.0.1:5000/movie/1",
    "api": "http://127.0.0.1:5000/api/v1/movies/1",
    "casts": [
        "蒂姆·罗宾斯",
        "摩根·弗里曼",
        "鲍勃·冈顿"
    ],
    "directors": [
        "弗兰克·德拉邦特"
    ],
    "douban_alt": "https://movie.douban.com/subject/1292052/",
    "genres": [
        "犯罪",
        "剧情"
    ],
    "images": "https://img3.doubanio.com/view/movie_poster_cover/lpst/public/p480747492.jpg",
    "original_title": "The Shawshank Redemption",
    "rating": 9.6,
    "title": "肖申克的救赎",
    "year": 1994
}
```

测试完毕，Flask也就配置好啦！


# 配置 uwsgi

```
(venv) $ touch config.ini
(venv) $ vim config.ini
```

将以下内容粘贴到里面：

```
[uwsgi]

socket = 127.0.0.1:8001

chdir = /home/MovieRental

wsgi-file = manage.py

callable = app

processes = 4

threads = 2

stats = 127.0.0.1:9191

```

启动 uwsgi：
```
(venv) $ uwsgi config.ini
```

# 配置 supervisor

安装：

```
(venv) $ apt-get install supervisor
```

配置`supervisor.conf`：
```
(venv) $ vim /etc/supervisor/conf.d/supervisor.conf
```

将以下内容粘贴到里面：

```
[program:MovieRental]

command=/home/MovieRental/venv/bin/uwsgi /home/MovieRental/config.ini

directory=/home/MovieRental

user=root

autostart=true

autorestart=true

stdout_logfile=/home/MovieRental/logs/uwsgi_supervisor.log
```

启动 supervisor：

```
(venv) $ service supervisor start
```

# 配置 nginx

安装：

```
(venv) $ apt-get install nginx
```

配置 default 文件：

```
(venv) $ vim /etc/nginx/sites-available/default
```

将以下内容粘贴到里面：

```
server {
      listen  xx; # 端口号
      server_name xx.xx.xx.xx; # 公网ip

      location / {
        include      uwsgi_params;
        uwsgi_pass   127.0.0.1:8001;

        uwsgi_param UWSGI_PYHOME /home/MovieRental/venv;

        uwsgi_param UWSGI_CHDIR  /home/MovieRental;
        uwsgi_param UWSGI_SCRIPT manage:app;
      }
    }
```

重启 nginx ：

```
(venv) $ service nginx restart
```

最后在浏览器地址栏输入公网ip地址和端口号就可以访问了！！！
