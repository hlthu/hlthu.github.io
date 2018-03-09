---
layout: post
title: Python OpenCV两张图片融合
date: 2016-06-10 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---



本节实现的是使用OpenCV里自带的函数，将两幅图片按照特定的比例融合


# 实现过程

## 引用与读取图片
不再赘述，代码如下。

```python
import cv2  
import numpy

img1 = cv2.imread('test1.png')
img2 = cv2.imread('test2.png')
```

## 融合图片
利用addWeighted()函数，将图片1的比例设置为0.6，图片2的0.4，如下：

```python
mg_mix = cv2.addWeighted(img1, 0.6, img2, 0.4, 0)
```

## 显示图片
分别显示两幅原图和融合后的图片：

```python
cv2.imshow('img1', img1)
cv2.imshow('img2', img2)
cv2.imshow('img_mix', img_mix)

cv2.waitKey(0)
cv2.destroyAllWindows()
```


# 源代码
整个程序的源代码如下：

```python
# created by Huang Lu
# 28/08/2016 13:44:54   
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

img1 = cv2.imread('test1.png')
img2 = cv2.imread('test2.png')

img_mix = cv2.addWeighted(img1, 0.6, img2, 0.4, 0)

cv2.imshow('img1', img1)
cv2.imshow('img2', img2)
cv2.imshow('img_mix', img_mix)

cv2.waitKey(0)
cv2.destroyAllWindows()
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV/Image_Mix/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![融和图片](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Image_Mix/Screenshot.png)


# 参考
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV/Image_Mix/
- http://hlthu.github.io/blogs/PythonOpenCV/12-Image_Mix.html