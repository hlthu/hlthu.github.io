---
layout: post
title: Python OpenCV使用滚动条自定义颜色
date: 2016-06-12 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---



本节实现的是使用OpenCV里自带的有关滚动条的函数，可以调节R、G、B三个数值，并显示颜色。

- 回调函数
- 滚动条设置
- 显示设置的颜色


# 实现过程

## 引用与创建空图
不再赘述，代码如下。

``` python
import cv2  
import numpy

# empty image
img = np.zeros((512, 512, 3), np.uint8)
```

## 设置空的回调函数
作为滚动条值变化时的回调函数，这里不需要做任何设置，设置为pass：

```python
# callbacks
def nothing(x):
	pass
```

## 创建四个滚动条
其中三个分别为R、G和B，其值范围为0~255，还有一个控制是否显示颜色，值只能取0或1。

```python
# creat track bars
cv2.createTrackbar('R', 'Track Bar', 0, 255, nothing)
cv2.createTrackbar('G', 'Track Bar', 0, 255, nothing)
cv2.createTrackbar('B', 'Track Bar', 0, 255, nothing)
cv2.createTrackbar('1:ON\n0:OFF', 'Track Bar', 0, 1, nothing)
```

## 获取滚动条数值并创建颜色
获取R、G、B滚动条的数值，如果此时控制颜色的为1，则显示该颜色，否则显示颜色为黑色。直到键盘输入q退出。

```python
while(1):
		R = cv2.getTrackbarPos('R', 'Track Bar')
		G = cv2.getTrackbarPos('G', 'Track Bar')
		B = cv2.getTrackbarPos('B', 'Track Bar')
		F = cv2.getTrackbarPos('1:ON\n0:OFF', 'Track Bar')		
		if F == 1:
			img[:]=[B,G,R]
		else:
			img[:]=[0,0,0]
		cv2.imshow('Track Bar', img)
		if cv2.waitKey(1) & 0xFF == ord('q'):
			break
	
	cv2.destroyAllWindows()
```


# 源代码
整个程序的源代码如下：

```python
# created by Huang Lu
# 27/08/2016 19:54:11   
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

# callbacks
def nothing(x):
	pass

if __name__ == '__main__':
	# empty image
	img = np.zeros((512, 512, 3), np.uint8)
	cv2.namedWindow('Track Bar')

	# creat track bars
	cv2.createTrackbar('R', 'Track Bar', 0, 255, nothing)
	cv2.createTrackbar('G', 'Track Bar', 0, 255, nothing)
	cv2.createTrackbar('B', 'Track Bar', 0, 255, nothing)
	cv2.createTrackbar('1:ON\n0:OFF', 'Track Bar', 0, 1, nothing)

	# while loop
	while(1):
		R = cv2.getTrackbarPos('R', 'Track Bar')
		G = cv2.getTrackbarPos('G', 'Track Bar')
		B = cv2.getTrackbarPos('B', 'Track Bar')
		F = cv2.getTrackbarPos('1:ON\n0:OFF', 'Track Bar')
		
		if F == 1:
			img[:]=[B,G,R]
		else:
			img[:]=[0,0,0]
		cv2.imshow('Track Bar', img)
		if cv2.waitKey(1) & 0xFF == ord('q'):
			break
	
	cv2.destroyAllWindows()
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Track_Bar/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![使用摄像头捕获视频并保存](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Track_Bar/Screenshot.png)


# 参考
- http://docs.opencv.org/2.4/modules/core/doc/drawing_functions.html
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Track_Bar/