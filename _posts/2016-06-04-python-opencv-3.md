---
layout: post
title: Python OpenCV提取彩色图像三通道
date: 2016-06-04 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---

本节实现的是提取出彩色图像的三个通道。

- 打开一个彩色图片
- 利用系统函数分离三通道
- 自行分离三通道
- 显示显示三通道图片


# 实现过程

## 引用与打开图片
不再赘述，代码如下。

``` python
import cv2  
import numpy

img = cv2.imread("test.png")
```

## 系统函数分离三通道
opencv里自带了分离三通道的函数split()，返回值依次是蓝色、绿色和红色通道的灰度图，代码如下：

``` python
b, g, r = cv2.split(img)
cv2.imshow("Blue 1", b)
cv2.imshow("Green 1", g)
cv2.imshow("Red 1", r)
```


## 自行分离三通道函数
定义了三个函数来获取三个通道的子矩阵。

获取红色通道：

``` python
def get_red(img):
	redImg = img[:,:,2]
	return redImg
```

获取绿色通道：

``` python
def get_green(img):
	greenImg = img[:,:,1]
	return greenImg
``` 

获取蓝色通道

``` python
def get_blue(img):
	blueImg = img[:,:,0]
	return blueImg
```

## 调用函数分离通道并显示
调用上述三个函数，显示分离结果，并与之前的比较。

``` python
b = get_blue(img)
g = get_green(img)
r = get_red(img)
cv2.imshow("Blue 2", b)
cv2.imshow("Green 2", g)
cv2.imshow("Red 2", r)
```

## 等待键盘输入并关闭所有窗口
这里cv2.waitKey里的参数0表示等待输入任何按键，当用户输入任何一个按键后即调用destroyAllWindows()关闭所有图像窗口。

``` python
cv2.waitKey (0)  
cv2.destroyAllWindows() 
```

# 源代码
整个程序的源代码如下：

``` python
# created by Huang Lu
# 2016/8/22 23:02
# Depart. of EE, Tsinghua Univ.

import cv2
import numpy

def get_red(img):
	redImg = img[:,:,2]
	return redImg
	
def get_green(img):
	greenImg = img[:,:,1]
	return greenImg
	
def get_blue(img):
	blueImg = img[:,:,0]
	return blueImg
	
if __name__ == '__main__':
	img = cv2.imread("test.png")
	b, g, r = cv2.split(img)
	cv2.imshow("Blue 1", b)
	cv2.imshow("Green 1", g)
	cv2.imshow("Red 1", r)
	b = get_blue(img)
	g = get_green(img)
	r = get_red(img)
	cv2.imshow("Blue 2", b)
	cv2.imshow("Green 2", g)
	cv2.imshow("Red 2", r)
	cv2.waitKey(0)
	cv2.destroyAllWindows()
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/RGB_Extract/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果，结果如下：
![运行结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/RGB_Extract/Screenshot.png)

# 参考
- http://blog.csdn.net/sunny2038/article/details/9080047
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV
