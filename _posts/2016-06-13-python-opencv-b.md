---
layout: post
title: Python OpenCV双击鼠标绘制圆
date: 2016-06-13 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---


本节实现的是使用OpenCV里自带的函数，在双击图片时，以其为圆心绘制圆。

- 回调函数
- 捕捉鼠标事件


# 实现过程

## 引用与创建空图
不再赘述，代码如下。

``` python
import cv2  
import numpy

# empty image
img = np.zeros((512, 512, 3), np.uint8)
```

## 设置回调函数
检测鼠标事件，如果左击鼠标则绘制圆。

```python
# call back function
def draw_circle(event, x, y, flags, param):
	if event == cv2.EVENT_LBUTTONDBLCLK:
		cv2.circle(img,(x,y),100,(255,0,0),1)
```

## 回调上述函数

```python
cv2.namedWindow('circle')
cv2.setMouseCallback('circle', draw_circle)
```

## 显示图像
循环显示图像，如果检测到键盘输入q则退出。

```python
while(1):
	cv2.imshow('circle', img)
	if cv2.waitKey(10) & 0xFF == ord('q'):
		break

cv2.destroyAllWindows()
```


# 源代码
整个程序的源代码如下：

``` python
# created by Huang Lu
# 27/08/2016 19:33:52  
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

# empty image
img = np.zeros((512, 512, 3), np.uint8)

# call back function
def draw_circle(event, x, y, flags, param):
	if event == cv2.EVENT_LBUTTONDBLCLK:
		cv2.circle(img,(x,y),100,(255,0,0),1)

cv2.namedWindow('circle')
cv2.setMouseCallback('circle', draw_circle)

while(1):
	cv2.imshow('circle', img)
	if cv2.waitKey(10) & 0xFF == ord('q'):
		break

cv2.destroyAllWindows()
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Draw_Mouse/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![使用摄像头捕获视频并保存](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Draw_Mouse/Screenshot.png)


# 参考
- http://docs.opencv.org/2.4/modules/core/doc/drawing_functions.html
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Draw_Mouse/
