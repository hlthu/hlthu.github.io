---
layout: post
title: Python OpenCV读取并显示图像
date: 2016-06-01 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---

Python作为一门极其易学的语言，在科学计算等领域存在较大的应用，同样，著名的OpenCV库也发布了支持Python的库，本节主不再介绍如何在Ubuntu上配置Python OpenCV，有需要的同学可以参考[这里](http://www.pyimagesearch.com/2015/06/22/install-opencv-3-0-and-python-2-7-on-ubuntu/)。

本节将利用Python OpenCV做一个简单的测试，即

- 打开一个图片并显示
- 创建一个空图并显示
- 将彩色图像转成灰度图像并显示


# 实现过程

## 引用
这里我们需要引用cv2和numpy，cv2不言而喻，而NumPy是Python语言的一个扩充程序库。支持高级大量的维度数组与矩阵运算，此外也针对数组运算提供大量的数学函数库，这里在引用时把numpy重命名为np。

```python
import cv2  
import numpy as np
```


## 打开图片并显示
cv2库里的函数imread()用来读取图片，imshow()可用于显示图片，此外imwrite可以用来写图片，即保存图片。这里把显示图片的窗口指定为“Image”。

```python
img = cv2.imread("./cat.png") 
cv2.imshow("Image", img) 
```

## 创建一个空图
numpy里有函数np.zeros用来创建零值矩阵，该函数的第一个参数为矩阵大小，这里是刚才读入图片的大小，即img.shape，第二个参数为数据类型，为uint8类型。指定显示该图片的窗口为“emptyImage”。

``` python
emptyImage = np.zeros(img.shape, np.uint8)
cv2.imshow("EmptyImage", emptyImage) 
```


##将彩色图像转成灰度图像
利用opencv自带的cvtColor可以将彩色图像转成灰度图像，这里主要是参数cv2.COLOR_BGR2GRAY进行说明的，指定显示该灰度图片的窗口为“emptyImage2”。

``` python
emptyImage2 = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY) 
cv2.imshow("EmptyImage2", emptyImage2) 
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
import numpy as np  
  
img = cv2.imread("./cat.png")  
emptyImage = np.zeros(img.shape, np.uint8)  
emptyImage2=cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)   
  
cv2.imshow("EmptyImage", emptyImage)  
cv2.imshow("Image", img)  
cv2.imshow("EmptyImage2", emptyImage2)   
 
cv2.waitKey (0)  
cv2.destroyAllWindows() 
```

也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/OpenCV_test)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果，结果如下：
![运行结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/OpenCV_test/screen.png)

# 参考
- http://blog.csdn.net/sunny2038/article/details/9057415
- http://www.pyimagesearch.com/2015/06/22/install-opencv-3-0-and-python-2-7-on-ubuntu/
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV