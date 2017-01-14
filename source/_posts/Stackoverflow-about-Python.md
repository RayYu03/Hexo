---
title: Stackoverflow about Python
date: 2016-10-14 21:26:03
tags: ['Stackoverflow','Python']
---
- [用Python如何一个判断文件是否存在?](#用Python如何判断一个文件是否存在)
- [在Python中有三元运算符吗?](#在Python中有三元运算符吗)
- [在Python里如何用枚举类型?](#在Python里如何用枚举类型)
- [如何在一个表达式里合并两个字典?](#如何在一个表达式里合并两个字典)

<!-- more -->
# 用Python如何判断一个文件是否存在
```Python
import os.path
os.path.isfile(fname)
```
# 在Python中有三元运算符吗
有,在2.5版本中加入.对于python初学者可能有点难以理解,所以要记住了.

语法如下:
```Python
a if test else b
```
根据test的布尔值来判断返回的是a还是b;如果test为真则返回a,反之则返回b.

来个大栗子:
```Python
>>> 'true' if True else 'false'
'true'
>>> 'true' if False else 'false'
'false'
```
**官方文档:**

- [Conditional expressions](https://docs.python.org/3/reference/expressions.html#conditional-expressions)
- [Is there an equivalent of C’s ”?:” ternary operator?](https://docs.python.org/3.3/faq/programming.html#is-there-an-equivalent-of-c-s-ternary-operator)

# 在Python里如何用枚举类型
PEP435标准里已经把枚举添加到Python3.4版本,在Pypi中也可以向后支持3.3, 3.2, 3.1, 2.7, 2.6, 2.5, 和 2.4版本.

如果想向后兼容$ pip install enum34,如果下载enum(没有数字)将会是另一个版本.
```Python
from enum import Enum
Animal = Enum('Animal', 'ant bee cat dog')
```
或者等价于:
```Python
class Animals(Enum):
    ant = 1
    bee = 2
    cat = 3
    dog = 4
```
在更早的版本,下面这种方法来完成枚举:
```Python
def enum(**enums):
    return type('Enum', (), enums)
像这样来用:

>>> Numbers = enum(ONE=1, TWO=2, THREE='three')
>>> Numbers.ONE
1
>>> Numbers.TWO
2
>>> Numbers.THREE
'three'
```
也很容易支持自动计数,像下面这样:
```Python
def enum(*sequential, **named):
    enums = dict(zip(sequential, range(len(sequential))), **named)
    return type('Enum', (), enums)
```
这样用:
```Python
>>> Numbers = enum('ZERO', 'ONE', 'TWO')
>>> Numbers.ZERO
0
>>> Numbers.ONE
1
```
如果要把值转换为名字可以加入下面的方法:
```Python
def enum(*sequential, **named):
    enums = dict(zip(sequential, range(len(sequential))), **named)
    reverse = dict((value, key) for key, value in enums.iteritems())
    enums['reverse_mapping'] = reverse
    return type('Enum', (), enums)
```
这样会覆盖名字下的所有东西,但是对于枚举的输出很有用.如果转换的值不存在就会抛出KeyError异常.用前面的例子:
```Python
>>> Numbers.reverse_mapping['three']
'THREE'
```
# 如何在一个表达式里合并两个字典?
我有两个Python字典,我想写一个表达式来返回两个字典的合并.update()方法返回的是空值而不是返回合并后的对象.
```Python

>>> x = {'a':1, 'b': 2}
>>> y = {'b':10, 'c': 11}
>>> z = x.update(y)
>>> print z
None
>>> x
{'a': 1, 'b': 10, 'c': 11}
```
怎么样才能最终让值保存在z而不是x?

可以用下面的方法:
```Python

z = dict(x.items() + y.items())
```
最后就是你想要的最终结果保存在字典z中,而键b的值会被第二个字典的值覆盖.
```Python

>>> x = {'a':1, 'b': 2}
>>> y = {'b':10, 'c': 11}
>>> z = dict(x.items() + y.items())
>>> z
{'a': 1, 'c': 11, 'b': 10}
```
如果你用的是Python3的话稍微有点麻烦:
```Python

>>> z = dict(list(x.items()) + list(y.items()))
>>> z
{'a': 1, 'c': 11, 'b': 10}
```
还可以这样:
```Python

z = x.copy()
z.update(y)
```
