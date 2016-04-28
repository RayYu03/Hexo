---
title: Python之神秘的加密模块cryptography（一）
date: 2016-04-28 22:14:28
tags: "cryptography"
---
**Cryptography的目标是建立一个标准Python加密库。**
-	如果你曾经在工作中使用过Python加密技术，那么你很可能知道一些库，例如M2Crypto、PyCrypto或者PyOpenSSL。

**Cryptography库想要解决已有库中存在的一些问题：**
	
-	缺少PyPy和Python 3支持
-	缺少维护
-	使用了差评的算法实现（例如旁路攻击side-channel attacks）
-	缺少高级（易于使用）的APIs
-	缺少AES-GCM和HKDF等算法
-	经不住测试错误百出的APIs

**Fernet（对称加密）**
-	Fernet保证了使用它进行加密的信息不能随意被操纵或者无秘钥解析。Fernet是一种对称加密的方式，同时也可以通过使用MultiFernet实现秘钥轮换的支持。

**需要用到的类和方法：**
-  class cryptography.fernet.Fernet(key)
	`该类提供加密和解密的工具`
- classmethod generate_key()
`生成秘钥`
- encrypt(data)
`通过秘钥对信息进行加密`
- decrypt(token, ttl=None)
`通过秘钥对信息进行解密`

`下面是使用示例：`
```python
>>> from cryptography.fernet import Fernet
>>> key = Fernet.generate_key()
>>> f = Fernet(key)
>>> token = f.encrypt(b"my deep dark secret")
>>> token
'gAAAAABXIhRmkk8oAyHB39s9JZJ_1ssqsMtoMiHinwht40hhjNJx2SXJbDfjAwmTGg_6FFV0i447lmssao9iXOnSOsbvgFrBZFki0iKd2eXM0158-IHt_bE='
>>> f.decrypt(token)
'my deep dark secret'
```