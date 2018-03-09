---
layout: post
title: Python OpenCV检测特定颜色
date: 2016-06-16 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---


本节实现的是使用OpenCV里自带的函数，检测出视频里图像中的蓝色和天蓝色、青色，比如我的手机背景、我衣服上的蓝色格子，墙砖的蓝色和学生证照片上的蓝色背景。

- 设置“蓝色”HSV范围
- BGR转HSV
- 捕获摄像头视频
- 获取蓝色部分mask
- 图像按位与操作
- 显示蓝色部分图像


# 实现过程

## 定义视频对象
视频对象用于捕获摄像头视频流。

``` python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)
```

## 设置HSV中蓝色、天蓝色范围
这里主要参考了[这个博客](http://blog.csdn.net/taily_duan/article/details/51506776)，我设置的颜色范围如下。

```python
# set blue thresh
lower_blue=np.array([78,43,46])
upper_blue=np.array([110,255,255])
```


## 获取视频帧并转成HSV格式
利用cvtColor()将BGR格式转成HSV格式，参数为cv2.COLOR_BGR2HSV。

```python
# get a frame and show
ret, frame = cap.read()
cv2.imshow('Capture', frame)
# change to hsv model
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
```

## 获取mask
利用inRange()函数和HSV模型中蓝色范围的上下界获取mask，mask中原视频中的蓝色部分会被弄成白色，其他部分黑色。


```python
# get mask
mask = cv2.inRange(hsv, lower_blue, upper_blue)
cv2.imshow('Mask', mask)
```

## 图像按位操作
将mask于原视频帧进行按位与操作，则会把mask中的白色用真实的图像替换：

```python
# detect blue
	res = cv2.bitwise_and(frame, frame, mask=mask)
	cv2.imshow('Result', res)
```

# 源代码
程序的源代码如下：

```python
# created by Huang Lu
# 28/08/2016 14:46:31 
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

cap = cv2.VideoCapture(0)

# set blue thresh
lower_blue=np.array([78,43,46])
upper_blue=np.array([110,255,255])

while(1):
	# get a frame and show
	ret, frame = cap.read()
	cv2.imshow('Capture', frame)
	
	# change to hsv model
	hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
	
	# get mask
	mask = cv2.inRange(hsv, lower_blue, upper_blue)
	cv2.imshow('Mask', mask)
	
	# detect blue
	res = cv2.bitwise_and(frame, frame, mask=mask)
	cv2.imshow('Result', res)
	
	if cv2.waitKey(1) & 0xFF == ord('q'):
		break

cap.release()
cv2.destroyAllWindows()

```

也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Color_Track/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Color_Track/Screenshot.png)

可以发现，我的手机背景、我衣服上的蓝色格子，墙砖的蓝色和学生证照片上的蓝色背景都被正确的识别出来了，但是还存在一些噪声，有待进一步改进。

# 参考
- http://blog.csdn.net/taily_duan/article/details/51506776
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Color_Track/
