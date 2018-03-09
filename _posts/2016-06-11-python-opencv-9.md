---
layout: post
title: Python OpenCV获取视频并保存
date: 2016-06-11 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---


本节实现的是使用内建摄像头捕获视频，并保存视频。

- 创建摄像头对象
- 逐帧显示实现视频播放
- 保存摄像头的每一帧图像



# 实现过程

## 引用
不再赘述，代码如下。

``` python
import cv2  
import numpy
```

## 创建摄像头对象
使用opencv自带的VideoCapture()函数定义摄像头对象，其参数0表示第一个摄像头，一般就是笔记本的内建摄像头。

``` python
cap = cv2.VideoCapture(0)
```

## 定义写入视频格式和写入对象
利用cv2.cv.FOURCC(*'XVID')定义视频格式，然后创建视频写入对象。

``` python
# Define the codec and create VideoWriter object
fourcc = cv2.cv.FOURCC(*'XVID')
out = cv2.VideoWriter('output.avi', fourcc, 20, (640, 480))
```

## 逐帧显示视频并写入
在while循环中，利用视频对象的read()函数读取视频的某帧，并显示，并写入视频。然后等待1个单位时间，如果期间检测到了键盘输入q，则退出，即关闭窗口。

``` python
while(1):
	# get a frame
	ret, frame = cap.read()
	# save a frame
	out.write(frame)
	# show a frame
	cv2.imshow("capture", frame)
	if cv2.waitKey(1) & 0xFF == ord('q'):
		break
```

## 释放摄像头对象和窗口
调用release()释放摄像头，调用destroyAllWindows()关闭所有图像窗口。

``` python
cap.release()
out.release()
cv2.destroyAllWindows()  
```
# 源代码
整个程序的源代码如下：

``` python
# created by Huang Lu
# 27/08/2016 17:05:45 
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

cap = cv2.VideoCapture(0)

# Define the codec and create VideoWriter object
fourcc = cv2.cv.FOURCC(*'XVID')
out = cv2.VideoWriter('output.avi', fourcc, 20, (640, 480))

while(1):
	# get a frame
	ret, frame = cap.read()
	# save a frame
	out.write(frame)
	# show a frame
	cv2.imshow("capture", frame)
	if cv2.waitKey(1) & 0xFF == ord('q'):
		break
cap.release()
out.release()
cv2.destroyAllWindows() 
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Save_Vedio/)。

# 运行结果
本次输出的视频为：https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Save_Vedio/output.avi


在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![使用摄像头捕获视频并保存](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Save_Video/Screenshot.png)


# 参考
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Save_Vedio/
