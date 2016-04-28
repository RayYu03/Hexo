---
title: Python模块之善变的UUID小公举
date: 2016-04-28 09:44:29
tags: "UUID"
---

**UUID是一种通用唯一识别码，是一种软件建构的标准。**
-	UUID的目的，是让分布式系统中的所有元素，都能有唯一的辨识信息，而不需要通过中央控制端来做辨识信息的指定。如此一来，每个人都可以创建不与其它人冲突的UUID。在这样的情况下，就不需考虑数据库创建时的名称重复问题。目前最广泛应用的UUID，是微软公司的全局唯一标识符（GUID），而其他重要的应用，则有Linux ext2/ext3文件系统、LUKS加密分区、GNOME、KDE、Mac OS X等等。另外我们也可以在e2fsprogs包中的UUID库找到实现。——[维基百科](https://zh.wikipedia.org/zh-cn/%E9%80%9A%E7%94%A8%E5%94%AF%E4%B8%80%E8%AF%86%E5%88%AB%E7%A0%81)

用通俗的话说就是，uuid是一种唯一标识，在许多领域作为标识用途。python的uuid模块就是用来生成它的。

**python提供的生成uuid的方法一共有4种：**
- `uuid1() 从硬件地址和当前时间生成`
- `uuid3() 从md5算法生成`
- `uuid4() 随机生成`
- `uuid5() 从SHA-1算法生成`

`下面是使用示例：`
```python
>>> #导入uuid模块
>>> import uuid

>>> # 从硬件地址和当前时间生成一个UUID
>>> uuid.uuid1()
UUID('a8098c1a-f86e-11da-bd1a-00112444be1e')

>>> # 从md5算法生成一个UUID，参数是uuid.NAMESPACE_DNS和一个任意字符串
>>> uuid.uuid3(uuid.NAMESPACE_DNS, 'python.org')
UUID('6fa459ea-ee8a-3ca4-894e-db77e160355e')

>>> # 随机生成一个UUID
>>> uuid.uuid4()
UUID('16fd2706-8baf-433b-82eb-8c7fada847da')

>>> # 从SHA-1算法生成一个UUID，参数是uuid.NAMESPACE_DNS和一个任意字符串
>>> uuid.uuid5(uuid.NAMESPACE_DNS, 'python.org')
UUID('886313e1-3b8a-5372-9b90-0c9aee199e5d')

>>> # 从一个十六进制字符串生成一个UUID(大括号和连字符被忽略)
>>> x = uuid.UUID('{00010203-0405-0607-0809-0a0b0c0d0e0f}')

>>> # 将一个UUID转换成一个标准格式的十六进制字符串
>>> str(x)
'00010203-0405-0607-0809-0a0b0c0d0e0f'

>>> # 得到的原UUID的字节码
>>> x.bytes
'\x00\x01\x02\x03\x04\x05\x06\x07\x08\t\n\x0b\x0c\r\x0e\x0f'

>>> # 从一个十六进制字节码生成一个UUID
>>> uuid.UUID(bytes=x.bytes)
UUID('00010203-0405-0607-0809-0a0b0c0d0e0f')

```
