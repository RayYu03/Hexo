---
title: Python Flask Web开发学习笔记（二）程序的基本结构
date: 2016-05-01 11:13:52
tags: "Flask"
---
## 2.1　初始化
所有 Flask 程序都必须创建一个程序实例。Web 服务器使用一种名为 Web 服务器网关接口（Web Server Gateway Interface，WSGI）的协议，把接收自客户端的所有请求都转交给这个对象处理。程序实例是 Flask 类的对象，经常使用下述代码创建：
```python
     from flask import Flask
     app = Flask(__name__)
```
Flask 类的构造函数只有一个必须指定的参数，即程序主模块或包的名字。在大多数程序中，Python 的 `__name__ `变量就是所需的值。 
<!-- more -->
## 2.2　路由和视图函数
`客户端`（例如Web浏览器）把请求发送给`Web服务器`，Web服务器再把请求发送给Flask程序实例。程序实例需要知道对每个URL请求运行哪些代码，所以保存了一个URL到Python的映射关系。

处理URL和函数之间关系的程序称为`路由`。

在 Flask 程序中定义路由的最简便方式，是使用程序实例提供的 `app.route` [装饰器](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014318435599930270c0381a3b44db991cd6d858064ac0000)，把修饰的函数注册为`路由`。下面的例子说明了如何使用这个修饰器声明路由：
```python
@app.route('/')
def index():
return '<h1>Hello World!</h1>'
```
> **修饰器是 Python 语言的标准特性，可以使用不同的方式修改函数的行为。惯常用法是使用修饰器把函数注册为事件的处理程序。**

前例把 `index() `函数注册为程序根地址的处理程序。如果部署程序的服务器域名为 `www.example.com`，在浏览器中访问`http://www.example.com` 后，会触发服务器执行 `index() `函数。这个函数的返回值称为`响应`，是客户端接收到的内容。如果客户端是` Web 浏览器`，响应就是显示给用户查看的文档。

像 `index() `这样的函数称为`视图函数（view function）`。视图函数返回的`响应`可以是包含HTML 的简单字符串，也可以是复杂的表单，后文会介绍。

如果你仔细观察日常所用服务的某些URL格式，会发现很多地址中都包含可变部分。例如， 你的Facebook资 料页面的地址是	` http://www.facebook.com/<your-name>`， `用户名（your-name）`是地址的一部分。Flask 支持这种形式的 URL，只需在 `route 修饰器`中使用特殊的句法即可。下例定义的路由中就有一部分是动态名字：
```python
     @app.route('/user/<name>')
     def user(name):
     return '<h1>Hello, %s!</h1>' % name
```
尖括号中的内容就是动态部分，任何能匹配静态部分的 URL 都会映射到这个路由上。调用视图函数时，Flask 会将动态部分作为参数传入函数。在这个视图函数中，参数用于生成针对个人的欢迎消息。

路由中的动态部分默认使用字符串，不过也可使用类型定义。例如，路由`/user/<int:id>`只会匹配动态片段 id 为整数的 URL。Flask 支持在路由中使用 `int` 、 `float `和`path `类型。path 类型也是字符串，但不把斜线视作分隔符，而将其当作动态片段的一部分。

## 2.3　启动服务器

程序实例用 run 方法启动 Flask 集成的开发 Web 服务器：
```python
if __name__ == '__main__':
app.run(debug=True)
```
 **`__name__== ' __main__ '` 是 Python 的惯常用法，在这里确保直接执行这个脚本时才启动开发Web 服务器。如果这个脚本由其他脚本引入，程序假定父级脚本会启动不同的服务器，因此不会执行` app.run()` 。**

服务器启动后，会进入`轮询`，等待并处理请求。轮询会一直运行，直到程序停止，比如按`Ctrl-C `键。

有一些选项参数可被` app.run() `函数接受用于设置 Web 服务器的操作模式。在开发过程中启用调试模式会带来一些便利，比如说激活调试器和重载程序。要想启用调试模式，我们可以把 `debug` 参数设为 `True` 。

## 2.4　一个完整的程序

前几节介绍了 Flask Web 程序的不同组成部分，现在是时候开发一个程序了。整个 hello.py程序脚本就是把前面介绍的三部分合并到一个文件中。程序代码如示例 2-1 所示。

**示例 2-1**　`hello.py`：一个完整的 Flask 程序
```python
from flask import Flask
#创建程序实例
app = Flask(__name__)

#定义路由和视图函数
@app.route('/')
def index():
     return '<h1>Hello World!</h1>'

#启动服务器
if __name__ == '__main__':
     app.run(debug=True)
```

> **如果你已经从 `GitHub` 上克隆了这个程序的 Git 仓库，那么可以执行 `git checkout 2a `签出程序的这个版本。**

**要想运行这个程序，请确保激活了你之前创建的虚拟环境，并在其中安装了 Flask。现在打开 Web 浏览器，在地址栏中输入` http://127.0.0.1:5000/`。图 2-1 是浏览器连接到程序后的示意图。**
![](http://ww4.sinaimg.cn/large/647dc635jw1f3fps5guwxj20kk08ajrp.jpg)

**`图 2-1　hello.py Flask 程序`**

**然后使用下述命令启动程序：**
```python
(venv) $ python hello.py
* Running on http://127.0.0.1:5000/
* Restarting with reloader
```
> **如果你输入其他地址，程序将不知道如何处理，因此会向浏览器返回错误代码 `404`。访问不存在的网页时，你也会经常看到这个熟悉的错误。**

**示例 2-2** 是这个程序的增强版，添加了一个动态路由。访问这个地址时，你会看到一则针对个人的欢迎消息。

**示例 2-2**　`hello.py`：包含动态路由的 Flask 程序
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
     return '<h1>Hello World!</h1>'

@app.route('/user/<name>')
def user(name):
     return '<h1>Hello, %s!</h1>' % name

if __name__ == '__main__':
     app.run(debug=True)
```

> 如果你已经从 GitHub 上克隆了这个程序的 Git 仓库，那么可以执行 git checkout 2b 签出程序的这个版本。测试动态路由前，你要确保服务器正在运行中，然后访问 `http://localhost:5000/user/Dave`。程序会显示一个使用 name 动态参数生成的欢迎消息。请尝试使用不同的名字，可以看到视图函数总是使用指定的名字生成响应。

图 2-2 展示了一个示例。

![](http://ww1.sinaimg.cn/large/647dc635jw1f3fpu6b51uj20kf08kjrt.jpg)
**`图 2-2　hello.py Flask 程序`**

