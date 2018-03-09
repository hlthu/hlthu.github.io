---
layout: post
title: Python OpenCV对像素进行操作
date: 2016-06-03 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---

本节实现的是在图片上模拟机上椒盐噪声，主要过程如下。

- 打开一个图片
- 产生随机坐标
- 加上“椒盐”
- 显示加噪图片


# 实现过程

## 引用
这里我们需要引用cv2和numpy，cv2不言而喻，而NumPy是Python语言的一个扩充程序库。支持高级大量的维度数组与矩阵运算，此外也针对数组运算提供大量的数学函数库。

``` python
import cv2  
import numpy
```

## 打开图片
cv2库里的函数imread()用来读取图片，imshow()可用于显示图片，此外imwrite可以用来写图片，即保存图片。这里把显示图片的窗口指定为“Image”。

``` python
img = cv2.imread("test.png")
```

## 加椒盐噪声
numpy里有函数可以随机生成0~1之间的随机数，将此随机值乘以图像长宽即可得到随机坐标，然后，如果是灰度图像，则只需将像素值改为255；若是彩色，则需将三个通道均改为255。这里我们定义了一个函数，输入的img几位图像矩阵，n为加的椒盐噪声数量。

``` python
def salt(img, n):
	for k in range(n):
		i = int(numpy.random.random() * img.shape[1])
		j = int(numpy.random.random() * img.shape[0])
		if img.ndim == 2:
			img[j, i] = 255
		elif img.ndim == 3:
			img[j, i, 0] = 255
			img[j, i, 1] = 255
			img[j, i, 2] = 255
	return img
```


## 主函数调用并显示图像
调用上述函数，让其生成1000个噪声点，并显示加噪图像。

``` python
saltImage = salt(img, 1000)
cv2.imshow("salt", saltImage)
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
import cv2
import numpy

# add n salt noise to an image
def salt(img, n):
	for k in range(n):
		i = int(numpy.random.random() * img.shape[1])
		j = int(numpy.random.random() * img.shape[0])
		if img.ndim == 2:
			img[j, i] = 255
		elif img.ndim == 3:
			img[j, i, 0] = 255
			img[j, i, 1] = 255
			img[j, i, 2] = 255
	return img

if __name__ == '__main__':
	img = cv2.imread("test.png")
	saltImage = salt(img, 1000)
	cv2.imshow("salt", saltImage)
	cv2.waitKey(0)
	cv2.destroyAllWindows()
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV/Image_Pixels/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果，结果如下：
![运行结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master//Image_Pixels/Screenshot.png)

# 参考
- http://blog.csdn.net/sunny2038/article/details/9080047
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV