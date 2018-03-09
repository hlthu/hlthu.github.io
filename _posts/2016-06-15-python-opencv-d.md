---
layout: post
title: Python OpenCV图像按位操作
date: 2016-06-15 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---



本节实现的是使用OpenCV里自带的函数，将一幅logo加到一张图片上去。

- 提取mask
- 利用mask生成带logo图片


# 实现过程

## 引用与读取图片
不再赘述，代码如下。

``` python
import cv2
import numpy as np

img = cv2.imread('test.png')
logo = cv2.imread('logo.jpg')
cv2.imshow("Img_Original", img)
```

## 获取mask
先将logo转成黑白，然后设置合适的阈值二值化，使得有内容的部分为黑（0），无内容的部分为白（255），这里使用的阈值为205。

```python
logo_gray = cv2.cvtColor(logo, cv2.COLOR_BGR2GRAY)
# binary & mask
ret, mask = cv2.threshold(logo_gray, 205, 255, cv2.THRESH_BINARY)
```

## 根据logo大小提取感兴趣区域
获取logo大小，在图像的左上角提取同样大小的ROI。

```python
rows, cols, channels = logo.shape
roi = img[0:rows, 0:cols]
```

## 将logo加到感兴趣区域
如果mask部分为黑，则将ROI的这部分用logo的内容替换，否则保留其原先内容。

```python
# dst
dst = roi
for r in range(rows):
	for c in range(cols):
		if mask[r, c] == 0:
			dst[r, c, :] = logo[r, c, :]
# add the dst to the img
img[0:rows, 0:cols] = dst
```

### 显示图片

```python
cv2.imshow("Color Logo", logo)
cv2.imshow("Gray Logo", logo_gray)
cv2.imshow("Mask", mask)
cv2.imshow("Dst", dst)
cv2.imshow("Img", img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```


# 源代码
整个程序的源代码如下：

```python
# created by Huang Lu
# 28/08/2016 13:55:37    
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

img = cv2.imread('test.png')
logo = cv2.imread('logo.jpg')
cv2.imshow("Img_Original", img)

logo_gray = cv2.cvtColor(logo, cv2.COLOR_BGR2GRAY)
rows, cols, channels = logo.shape
roi = img[0:rows, 0:cols]
# binary & mask
ret, mask = cv2.threshold(logo_gray, 205, 255, cv2.THRESH_BINARY)
# dst
dst = roi
for r in range(rows):
	for c in range(cols):
		if mask[r, c] == 0:
			dst[r, c, :] = logo[r, c, :]
# add the dst to the img
img[0:rows, 0:cols] = dst


cv2.imshow("Color Logo", logo)
cv2.imshow("Gray Logo", logo_gray)
cv2.imshow("Mask", mask)
cv2.imshow("Dst", dst)
cv2.imshow("Img", img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV/Bit_Operation/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![融和图片](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Bit_Operation/Screenshot.png)


# 参考
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV/Bit_Operation/
- http://hlthu.github.io/blogs/PythonOpenCV/13-Bit_Operation.html