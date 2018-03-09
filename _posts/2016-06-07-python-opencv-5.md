---
layout: post
title: Python OpenCV使用Matplotlib显示图像
date: 2016-06-07 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---

本节实现的是同时使用opencv和matplotlib两种方式来显示图像，以比较二者之间的差别。

- 测试灰度图像
- 测试彩色图像
- 分析彩色图像出现差异的原因


# 实现过程

## 引用
不再赘述，代码如下。

``` python
import cv2  
import numpy
import matplotlib.pyplot as plot
```

## 测试灰度图像
打开灰度图像，先用opencv显示，再用matplotlib显示，代码如下：

``` python
# test for a gray image
img1 = cv2.imread("test1.jpg")
# using opencv
cv2.imshow("Gray(opencv)", img1)
# using matplotlib
plot.imshow(img1)
plot.show()
```


## 测试彩色图像
打开彩色图像，并创建一个它的反转图像，即R、G、B依次变成了B、G、R。然后先用opencv显示这两幅彩色图像，再用matplotlib显示，代码如下：

``` python
# test for a color image
img2 = cv2.imread("test2.jpg")
b, g, r = cv2.split(img2)
img2_c = cv2.merge([r, g, b])
# using opencv
cv2.imshow("Color(opencv, img2)", img2)
cv2.imshow("Color(opencv, img2_c)", img2_c)
# using matplotlib
plot.subplot(121);plot.imshow(img2)
plot.subplot(122);plot.imshow(img2_c)
plot.show()
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
# 27/08/2016 16:05:20
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np
import matplotlib.pyplot as plot

# test for a gray image
img1 = cv2.imread("test1.jpg")
# using opencv
cv2.imshow("Gray(opencv)", img1)
# using matplotlib
plot.imshow(img1)
plot.show()

# test for a color image
img2 = cv2.imread("test2.jpg")
b, g, r = cv2.split(img2)
img2_c = cv2.merge([r, g, b])
# using opencv
cv2.imshow("Color(opencv, img2)", img2)
cv2.imshow("Color(opencv, img2_c)", img2_c)
# using matplotlib
plot.subplot(121);plot.imshow(img2)
plot.subplot(122);plot.imshow(img2_c)
plot.show()

cv2.waitKey(0)    
cv2.destroyAllWindows() 
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python/tree/master/Python_OpenCV/Matplotlib_Display/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。

## 灰度图像
灰度图像的结果二者相同。灰度图像opencv显示结果：
![灰度图像opencv显示结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Matplotlib_Display/Screenshot2.png)

灰度图像matplotlib显示结果：
![灰度图像matplotlib显示结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Matplotlib_Display/Screenshot1.png)

## 彩色图像
彩色图像二者显示的结果却出现了反转，即本应正常的图像用matplotlib显示却是其“反转”图像。彩色图像opencv显示结果：
![彩色图像opencv显示结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Matplotlib_Display/Screenshot3.png)

彩色图像matplotlib显示结果：
![彩色图像matplotlib显示结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Matplotlib_Display/Screenshot4.png)

# 参考
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python/tree/master/Python_OpenCV/Matplotlib_Display/
- http://stackoverflow.com/questions/15072736/extracting-a-region-from-an-image-using-slicing-in-python-opencv/15074748#15074748