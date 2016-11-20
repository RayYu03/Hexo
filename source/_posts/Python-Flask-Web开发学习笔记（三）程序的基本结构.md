---
title: Python Flask Web开发学习笔记（三）程序的基本结构
date: 2016-05-16 11:33:32
tags: ["Flask","Python"]
---
## 2.5　请求-响应循环
现在你已经开发了一个简单的 Flask 程序，或许希望进一步了解 Flask 的工作方式。下面将介绍这个框架的一些设计理念。
####  2.5.1　程序和请求上下文
Flask 从客户端收到请求时，要让视图函数能访问一些对象，这样才能处理请求。`请求对象`就是一个很好的例子，它封装了客户端发送的 HTTP 请求。

要想让视图函数能够访问请求对象，一个显而易见的方式是将其作为参数传入视图函数，不过这会导致程序中的每个视图函数都增加一个参数。

除了访问请求对象，如果视图函数在处理请求时还要访问其他对象，情况会变得更糟。
<!-- more -->
为了避免大量可有可无的参数把视图函数弄得一团糟，Flask 使用上下文临时把某些对象变为全局可访问。有了上下文，就可以写出下面的视图函数：

```python
from flask import request

@app.route('/')
def index():
	user_agent = request.headers.get('User-Agent')
	return '<p>Your browser is %s</p>' % user_agent
```
注意在这个视图函数中我们如何把 request 当作全局变量使用。事实上， `request` 不可能是全局变量。

试想，在多线程服务器中，多个线程同时处理不同客户端发送的不同请求时，每个线程看到的 request 对象必然不同。`Falsk` 使用上下文让特定的变量在一个线程中全局可访问，与此同时却不会干扰其他线程。

在 Flask 中有两种上下文：`程序上下文`和`请求上下文`。

#### 表 2-1 列出了这两种上下文提供的变量。

![](http://ww3.sinaimg.cn/large/647dc635jw1f3x32qe0n0j20he03amxo.jpg)



Flask 在分发请求之前激活（或推送）程序和请求上下文，请求处理完成后再将其删除。程序上下文被推送后，就可以在线程中使用 current_app 和 g 变量。类似地，请求上下文被推送后，就可以使用 request 和session 变量。如果使用这些变量时我们没有激活程序上下文或请求上下文，就会导致错误。

**下面这个 Python shell 会话演示了程序上下文的使用方法**
```python
>>> from hello import app
>>> from flask import current_app
>>> current_app.name
Traceback (most recent call last):
...
RuntimeError: working outside of application context
>>> app_ctx = app.app_context()
>>> app_ctx.push()
>>> current_app.name
'hello'
>>> app_ctx.pop()
```

在这个例子中，没激活程序上下文之前就调用 `current_app.name` 会导致错误，但推送完上下文之后就可以调用了。注意，在程序实例上调用 `app.app_context() `可获得一个程序上下文。

#### 2.5.2　请求调度
程序收到客户端发来的请求时，要找到处理该请求的视图函数。为了完成这个任务，`Flask`会在程序的 `URL` 映射中查找请求的 URL。URL 映射是 URL 和视图函数之间的对应关系。`Flask` 使用 `app.route` 修饰器或者非修饰器形式的` app.add_url_rule() `生成映射。

要想查看 Flask 程序中的 URL 映射是什么样子，我们可以在 Python shell 中检查为 hello.py生成的映射。测试之前，请确保你激活了虚拟环境：	
```python
(venv) $ python
>>> from hello import app
>>> app.url_map
Map([<Rule '/' (HEAD, OPTIONS, GET) -> index>,
<Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
<Rule '/user/<name>' (HEAD, OPTIONS, GET) -> user>])
```
`/ `和 `/user/<name>`路由在程序中使用 `app.route `修饰器定义。` /static/<filename>` 路由是`Flask` 添加的特殊路由，用于访问静态文件。第 3 章会详细介绍`静态文件`。

URL 映射中的 `HEAD` 、 `Options` 、 `GET` 是请求方法，由路由进行处理。`Flask `为每个路由都指定了请求方法，这样不同的请求方法发送到相同的 `URL` 上时，会使用不同的视图函数进行处理。 `HEAD` 和 `OPTIONS` 方法由 Flask 自动处理，因此可以这么说，在这个程序中，URL映射中的 3 个路由都使用 `GET 方法`。第 4 章会介绍如何为路由指定不同的请求方法。
#### 2.5.3　请求钩子
有时在处理请求之前或之后执行代码会很有用。例如，在请求开始时，我们可能需要创建数据库连接或者认证发起请求的用户。为了避免在每个视图函数中都使用重复的代码，Flask 提供了注册通用函数的功能，注册的函数可在请求被分发到视图函数之前或之后调用。

请求钩子使用修饰器实现。Flask 支持以下 4 种钩子。
- before_first_request：注册一个函数，在处理第一个请求之前运行。
- before_request ：注册一个函数，在每次请求之前运行。
- after_request ：注册一个函数，如果没有未处理的异常抛出，在每次请求之后运行。
- teardown_request ：注册一个函数，即使有未处理的异常抛出，也在每次请求之后运行。

在`请求钩子函数`和`视图函数`之间共享数据一般使用上下文`全局变量 g `。例如，` before_request` 处理程序可以从数据库中加载已登录用户，并将其保存到 `g.user `中。随后调用视图函数时，视图函数再使用`g.user` 获取用户。
请求钩子的用法会在后续章中介绍，如果你现在不太理解，也不用担心。

#### 2.5.4　响应

Flask 调用视图函数后，会将其返回值作为响应的内容。大多数情况下，响应就是一个简单的字符串，作为 HTML 页面回送客户端。

但 HTTP 协议需要的不仅是作为请求响应的字符串。HTTP 响应中一个很重要的部分是状态码，Flask 默认设为 200，这个代码表明请求已经被成功处理。

如果视图函数返回的响应需要使用不同的状态码，那么可以把数字代码作为第二个返回值，添加到响应文本之后。例如，下述视图函数返回一个 400 状态码，表示请求无效：

```python
@app.route('/')
def index():
	return '<h1>Bad Request</h1>', 400
```
视图函数返回的响应还可接受第三个参数，这是一个由首部（header）组成的字典，可以添加到 HTTP 响应中。一般情况下并不需要这么做，不过你会在第 14 章看到一个例子。

如果不想返回由 1 个、2 个或 3 个值组成的元组，Flask 视图函数还可以返回 Response 对象。 make_response() 函数可接受 1 个、2 个或 3 个参数（和视图函数的返回值一样），并返回一个 Response 对象。有时我们需要在视图函数中进行这种转换，然后在响应对象上调用各种方法，进一步设置响应。下例创建了一个响应对象，然后设置了 cookie：

```python
from flask import make_response
@app.route('/')
def index():
	response = make_response('<h1>This document carries a cookie!</h1>')
	response.set_cookie('answer', '42')
	return response
```
有一种名为重定向的特殊响应类型。这种响应没有页面文档，只告诉浏览器一个新地址用以加载新页面。重定向经常在 Web 表单中使用，第 4 章会进行介绍。重定向经常使用 302 状态码表示，指向的地址由 Location 首部提供。重定向响应可以使用3 个值形式的返回值生成，也可在 Response 对象中设定。不过，由于使用频繁，Flask 提供了 redirect() 辅助函数，用于生成这种响应：
```python
from flask import redirect
@app.route('/')
def index():
	return redirect('http://www.example.com')
```
还有一种特殊的响应由 abort 函数生成，用于处理错误。在下面这个例子中，如果 URL 中动态参数 id 对应的用户不存在，就返回状态码 404：
```python
from flask import abort
@app.route('/user/<id>')
def get_user(id):
	user = load_user(id)
	if not user:
		abort(404)
	return '<h1>Hello, %s</h1>' % user.name
```
注意， abort 不会把控制权交还给调用它的函数，而是抛出异常把控制权交给 Web 服务器。

## 2.6　Flask扩展
Flask 被设计为可扩展形式，故而没有提供一些重要的功能，例如数据库和用户认证，所以开发者可以自由选择最适合程序的包，或者按需求自行开发。

社区成员开发了大量不同用途的扩展，如果这还不能满足需求，你还可使用所有 Python 标准包或代码库。为了让你知道如何把扩展整合到程序中，接下来我们将在 `hello.py` 中添加一个扩展，使用命令行参数增强程序的功能。

#### 使用Flask-Script支持命令行选项
Flask 的开发 Web 服务器支持很多启动设置选项，但只能在脚本中作为参数传给` app.run()`函数。这种方式并不十分方便，传递设置选项的理想方式是使用命令行参数。
`Flask-Script` 是一个 `Flask `扩展，为 `Flask `程序添加了一个命令行解析器。`Flask-Script` 自带了一组常用选项，而且还支持自定义命令。
`Flask-Script `扩展使用` pip` 安装：

```
	(venv) $ pip install flask-script
```
示例 2-3 显示了把命令行解析功能添加到 hello.py 程序中时需要修改的地方。
**`示例 2-3　hello.py：使用 Flask-Script`**
```python
from flask.ext.script import Manager
manager = Manager(app)
# ...
if __name__ == '__main__':
	manager.run()
```
专为 Flask 开发的扩展都暴漏在 `flask.ext` 命名空间下。Flask-Script 输出了一个名为`Manager` 的类，可从 `flask.ext.script `中引入。

这个扩展的初始化方法也适用于其他很多扩展：把程序实例作为参数传给构造函数，初始化主类的实例。创建的对象可以在各个扩展中使用。在这里，服务器由 `manager.run() `启动，启动后就能解析命令行了。

这样修改之后，程序可以使用一组基本命令行选项。现在运行 hello.py，会显示一个用法消息：

```
$ python hello.py
usage: hello.py [-h] {shell,runserver} ...
positional arguments:
	{shell,runserver}
	  shell 在 Flask 应用上下文中运行 Python shell
	  runserver 运行 Flask 开发服务器：app.run()
optional arguments:
	-h, --help 显示帮助信息并退出
```
shell 命令用于在程序的上下文中启动 Python shell 会话。你可以使用这个会话中运行维护任务或测试，还可调试异常。顾名思义， runserver 命令用来启动 Web 服务器。运行 python hello.py runserver 将以调试模式启动 Web 服务器，但是我们还有很多选项可用：


```python
(venv) $ python hello.py runserver --help
usage: hello.py runserver [-h] [-t HOST] [-p PORT] [--threaded]
				[--processes PROCESSES] [--passthrough-errors] [-d][-r]
							
运行 Flask 开发服务器：app.run()

optional arguments:
	-h, --help 显示帮助信息并退出
	-t HOST, --host HOST
	-p PORT, --port PORT
	--threaded
	--processes PROCESSES
	--passthrough-errors
	-d, --no-debug
	-r, --no-reload
``` 
--host参数是个很有用的选项，它告诉 Web 服务器在哪个网络接口上监听来自客户端的连接。默认情况下，Flask 开发 Web 服务器监听 localhost 上的连接，所以只接受来自服务器所在计算机发起的连接。下述命令让 Web 服务器监听公共网络接口上的连接，允许同网中的其他计算机连接服务器：
```python
(venv) $ python hello.py runserver --host 0.0.0.0
 * Running on http://0.0.0.0:5000/
 * Restarting with reloader
```
现在，Web 服务器可使用 http://a.b.c.d:5000/ 网络中的任一台电脑进行访问，其中“a.b.c.d”是服务器所在计算机的外网 IP 地址。
本章介绍了请求响应的概念，不过响应的知识还有很多。对于使用模板生成响应，Flask提供了良好支持，这是个很重要的话题，下一章我们还要专门介绍模板。