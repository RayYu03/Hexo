---
title: Python每日小程序 Vol.0000
date: 2016-04-24 12:09:36
tags: "python"
---

>题目： 将你的 QQ 头像（或者微博头像）右上角加上红色的数字，类似于微信未读信息数量那种提示效果。

`代码`
```python
#!/usr/bin/env python
# encoding: utf-8
#导入Python图像处理库中的模块
import Image
import ImageDraw
import ImageFont
#打开图片
im = Image.open("/home/rayyu/python_day/jindong.jpg")
draw = ImageDraw.Draw(im)
#导入字体，设置字体大小
fo = ImageFont.truetype("/home/rayyu/python_day/ARIALN.TTF", 300)
#在打开的图片右上角画图
draw.text((500,1), text = "2", font=fo, fill="#ff0000")
#显示修改后的图片
im.show()
#将图片保存到当前文件夹
im.save('result.jpg')
```