---
title: 《Effective Python》之用Pythonic的方式来思考(一)
date: 2016-05-10 20:41:17
tags: "Pythonic"
---
## Part 1 ： 遵循PEP8风格指南

《Python Enhancement Proposal #8》[英文版](https://www.python.org/dev/peps/pep-0008/) [中文版](https://lizhe2004.gitbooks.io/code-style-guideline-cn/content/python/python-pep8.html)，又叫PEP8，它是针对Python代码格式而修订的风格指南。尽管可以在保证语法正确的前提下随意编写Python代码，但是，采用一致的风格来写代码，可以使项目更利于多人协作。

PEP8列出了许多编码细节，以描述如何撰写清晰的Python代码。
<!-- more -->

下面列出几条我们需要遵守的规则。

###  **代码布局**
#### **缩进**
- 每一级缩进使用4个空格
- 续行应该将换行的元素垂直对齐，可以使用Python的圆括号、方括号、花括号中的隐式的连接符，也可以使用悬挂式缩进
- 当采用悬挂式缩进方式时，应该符合下列考虑条件：第一行不应该有参数，而且应该能够清楚无误地将后面采用缩进的行作为后续行从其他行中区分出来

**Right Example :**
```python
# Aligned with opening delimiter.
# 使用分隔符来对齐
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# More indentation included to distinguish this from the rest.
# 多使用一些缩进来与其他行进行区分
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)

# Hanging indents should add a level.
# 悬挂式缩进应该增加一个级别。
foo = long_function_name(
    var_one, var_two,
    var_three, var_four)
```

**Wrong Example :**
```python
# Arguments on first line forbidden when not using vertical alignment.
# 当不使用垂直对齐方式时，第一行不能有参数
foo = long_function_name(var_one, var_two,
    var_three, var_four)

# Further indentation required as indentation is not distinguishable.
# 需要增加额外的缩进，因为现在的续行的缩进不够明显，不能将其与其他行的缩进区分开来
def long_function_name(
    var_one, var_two, var_three,
    var_four):
    print(var_one)
```

**The 4-space rule is optional for continuation lines. 对于续行而言，4个空格的规定是可选的。**

```python
# Hanging indents *may* be indented to other than 4 spaces.
# 悬挂式缩进的空格个数*可以*不为4。
foo = long_function_name(
  var_one, var_two,
  var_three, var_four)
```
**当if语句由于条件判断部分太长而需要跨多行时，值得注意的是，这一两个字母的关键字（比如if）加上一个空格，再加上一个左括号，就使得需要在这一多行的条件语句的后面添加4个空格的缩进，这是很自然的事情。 这可能会与if语句所嵌套的那套分支代码产生视觉上的冲突，因为很自然地，分支执行代码也是缩进4个空格的。对于如何（或者是否要）将条件判断语句与if语句中的分支代码明确地区分出来，本PEP文档没有明确的立场。对于这种情况，下面的选项都是可以接受的，但是又不限于此：**
```python
# No extra indentation.
# 没有多余缩进
if (this_is_one_thing and
    that_is_another_thing):
    do_something()

# Add a comment, which will provide some distinction in editors
# supporting syntax highlighting.
# 添加一条注释来在编辑器中提供一些不同之处
# 支持与语法高亮
if (this_is_one_thing and
    that_is_another_thing):
    # Since both conditions are true, we can frobnicate.
    # 当两种条件都为true时，我们可以执行操作
    do_something()

# Add some extra indentation on the conditional continuation line.
# 在条件判断语句所在行添加一些额外的缩进
if (this_is_one_thing
        and that_is_another_thing):
    do_something()
```
**在多行的代码结构中，右花括号/中括号/小括号可以排在列出的元素的最后一行的第一个非空格字符的下面，比如：**
```python
my_list = [
    1, 2, 3,
    4, 5, 6,
    ]
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
    )
```
**也可以排在该多行代码结构的起始行的第一个字符下面。**
```python
my_list = [
    1, 2, 3,
    4, 5, 6,
]
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
)
```