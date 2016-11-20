---
title: Python中多继承与super()用法
date: 2016-09-17 19:01:17
tags: "Python"
---

Python类分为两种，一种叫`经典类`，一种叫`新式类`。两种都支持`多继承`。

考虑一种情形，B继承于A，C继承于A和B, 但C需要调用父类的init()函数时，前者会导致父类A的init()函数被调用2次，这是不希望看到的。而且子类要显式地指定父类，不符合`DRY`原则。

<!-- more -->

``` python
# 经典类
class A():
    def __init__(self):
        print 'A'

class B(A):
    def __init__(self):
        A.__init__(self)
        print 'B'

class C(B, A):
    def __init__(self):
        A.__init__(self)
        B.__init__(self)
        print 'C'
```
采用新式类，要求最顶层的父类一定要继承于`object`，这样就可以利用`super()`函数来调用父类的`init()`等函数，每个父类都执行且执行一次，并不会出现重复调用的情况。而且在子类的实现中，不用到处写出所有的父类名字，符合`DRY`原则。

``` python
# 新式类
class A(object):
    def __init__(self):
        print 'A'

class B(A):
    def __init__(self):
        super(B, self).__init__()
        print 'B'

class C(B, A):
    def __init__(self):
        super(C, self).__init__()
        print 'C'
```

采用`super()`方式时，会自动找到第一个多继承中的第一个父类，但是如果还想`强制调用`其他父类的`init()`函数或两个父类的`同名函数`时，就要用老办法了。

``` python
class Person(object):
    name = ""
    sex = ""
    def __init__(self, name, sex='U'):
        print 'Person'
        self.name=name
        self.sex=sex


class Consumer(object):
    def __init__(self):
        print 'Consumer'

class Student(Person, Consumer):
    def __init__(self, score,name):
        print Student.__bases__
        super(Student, self).__init__(name, sex='F')
        Consumer.__init__(self)
        self.score=score

s1 = Student(90, 'abc')
print s1.name, s1.score, s1.sex
```
