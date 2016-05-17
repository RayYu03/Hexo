---
title: Python Flask Web开发学习笔记（一）程序的基本结构
date: 2016-05-01 09:50:28
tags: "Flask"
---
**首先还是要提一下，需要安装的工具：**

- `Python 2.7 `or `Python 3.5`（不保证能用）

**虚拟环境` virtualenv`：**

- 大多数 Linux 发行版都提供了 virtualenv 包。例如， Ubuntu 用户可以使用下述命令安装它：

`$ sudo apt-get install python-virtualenv`

- 如果你的电脑是 Mac OS X 系统，就可以使用 easy_install 安装 virtualenv ：

`$ sudo easy_install virtualenv`

- 如果你使用微软的 Windows 系统，可以使用pip安装。

`$ pip install virtualenv`

<!-- more -->

现在你要新建一个文件夹，用来保存示例代码（示例代码可从 GitHub 库中获取）。获取示例代码最简便的方式是使用 Git 客户端直接从 GitHub 下载。下述命令从 GitHub 下载示例代码，并把程序文件夹切换到“`1a`”版
本，即程序的初始版本：
```night eighties
$ git clone https://github.com/miguelgrinberg/flasky.git
$ cd flasky
$ git checkout 1a
```

**下一步是使用 virtualenv 命令在 flasky 文件夹中创建 Python 虚拟环境。**
这个命令只有一个必需的参数，即虚拟环境的名字。创建虚拟环境后，当前文件夹中会出现一个子文件夹，名字就是上述命令中指定的参数，与虚拟环境相关的文件都保存在这个子文件夹中。
按照惯例，一般虚拟环境会被命名为` venv `：
```night eighties
$ virtualenv venv
New python executable in venv/bin/python2.7
Also creating executable in venv/bin/python
Installing setuptools............done.
Installing pip...............done.
```
现在，flasky 文件夹中就有了一个名为 venv 的子文件夹，它保存一个全新的虚拟环境，其中有一个私有的 Python 解释器。在使用这个虚拟环境之前，你需要先将其**激活**。

- 如果你使用 bash 命令行（Linux 和 Mac OS X 用户），可以通过下面的命令激活这个虚拟环境：

`$ source venv/bin/activate`

- 如果使用微软 Windows 系统，激活命令是：

`$ venv\Scripts\activate`

虚拟环境被激活后，其中 Python 解释器的路径就被添加进 PATH 中，但这种改变不是永久
性的，它只会影响当前的命令行会话。为了提醒你已经激活了虚拟环境，激活虚拟环境的
命令会修改命令行提示符，加入环境名：

`(venv) $`

当虚拟环境中的工作完成后，如果你想回到全局 Python 解释器中，可以在命令行提示符下输入 `deactivate `。

**在虚拟环境中安装` Flask`：**

 `(venv) $ pip install flask`

执行上述命令，你就在虚拟环境中安装 Flask 及其依赖了。要想验证 Flask 是否正确安装，
你可以启动 Python 解释器，尝试导入 Flask：
```night eighties
(venv) $ python
>>> import flask
>>>
```
如果没有看到错误提醒，那恭喜你——你已经可以开始学习下面的内容，了解如何开发第一个 `Web` 程序了。