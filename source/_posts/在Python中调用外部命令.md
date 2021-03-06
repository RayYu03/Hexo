---
title: 在Python中调用外部命令?
date: 2016-10-13 22:51:43
tags: 'Python'
---

## 怎么在Python脚本里调用外部命令?(就好像直接输入在Unix shell中或者windows的命令行里)
1 `os.system`("命令加参数")把命令和参数传递给你系统的`shell`中.用这个命令的好处在于你可以一次运行好多命令还可以设置管道来进行重定向。

举个栗子:
- `os.system`("命令 < 出入文件 | 另一个命令 > 输出文件")尽管它非常方便,但是你还是不得不手动输入像空格这样的shell字符.从另一方面讲,对于运行简单的shell命令而不去调用外部程序来说的话还是非常好用的。
<!-- more -->
2 `stream = os.popen`("命令和参数")这个命令和`os.system`差不多,但是它提供了一个连接标准输入/输出的管道。还有其他3个popen可以调用。
如果你传递一个字符串,你的命令会把它传递给shell,如果你传递的是一个列表,那么就不用担心溢出字符了(`escaping characters`)。

3 `subprocess`模块的管道`Popen`

这个Popen是打算用来替代os.popen的方法,它有点复杂:
```Python
print subprocess.Popen("echo Hello World", shell=True,stdout=PIPE).stdout.read()
```
  而用`os.popen`:
```Python
print os.popen("echo Hello World").read()
```
它最大的优点就是一个类里代替了原来的4个不同的`popen`。

4 `subprocess`的`call`方法.它的基本用法和上面的`Popen`类参数一致,但是它会等待命令结束后才会返回程序.来个大狸子:
```Python
return_code = subprocess.call("echo Hello World", shell=True)
```
5 os模块里也有C语言里`fork/exec/spawn`方法,但是我不建议你直接用它们。

`subprocess`模块可能更适合你。

最后请注意在你传递到`shell`的命令一定要注意参数的安全性。

给你个提示,看下面代码:
```Python
print subprocess.Popen("echo %s " % user_input, stdout=PIPE).stdout.read()
```
想象一下如果哪个SB输入 `my mama didnt love me && rm -rf /`
