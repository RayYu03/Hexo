---
title: 在Python中有三元运算符吗?
date: 2016-09-21 22:15:57
tags: 'Python'
---

如果没有,可以像其他语言用的简单方法来实现吗?

在python 2.5版本中加入了三元运算.对于python初学者可能有点难以理解,所以要记住了。
<!-- more -->
语法如下:

```python
a if test else b
```
根据test的布尔值来判断返回的是a还是b;如果test为真则返回a,反之则返回b.

来个大栗子:
```python
>>> 'true' if True else 'false'
'true'
>>> 'true' if False else 'false'
'false'
```
官方文档:

- [Conditional expressions](https://docs.python.org/3/reference/expressions.html#conditional-expressions)
- [Is there an equivalent of C’s ”?:” ternary operator?](https://docs.python.org/3.3/faq/programming.html#is-there-an-equivalent-of-c-s-ternary-operator)
