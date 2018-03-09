---
layout: post
title: Python OpenCV绘制简单图形
date: 2016-06-12 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---

本节实现的是使用OpenCV里自带的函数，绘制直线、长方形、圆形和椭圆。

- 绘制直线
- 绘制长方形
- 绘制圆形
- 绘制椭圆
- 添加文字


# 实现过程

## 引用与创建空图
不再赘述，代码如下。

``` python
import cv2  
import numpy

# empty image
img = np.zeros((512, 512, 3), np.uint8)
```

## 绘制直线
使用opencv自带的line()函数绘制一条对角线，其声明如下：

``` python
cv2.line(img, pt1, pt2, color[, thickness[, lineType[, shift]]])
```

其中各种参数的意义如下：

- **img** – Image.
- **pt1** – First point of the line segment.
- **pt2** – Second point of the line segment.
- **color** – Line color.
- **thickness** – Line thickness.
- **lineType** – Type of the line:

| 参数 | 意义 |
|--------|--------|
|8 (or omitted) | 8-connected line|
|4 | 4-connected line|
|CV_AA |antialiased line|

- **shift** – Number of fractional bits in the point coordinates.


在本文中，我绘制的直线从(0,0)到(511,511)，线的颜色为蓝色(255,0,0)，粗细为10，其他参数省略。

``` python
# draw a line
cv2.line(img, (0,0), (511,511), (255, 0, 0), 10)
```

## 绘制矩形
利用opencv自带的rectangle()函数可以绘制一个长方形，其声明如下：

```python
cv2.rectangle(img, pt1, pt2, color[, thickness[, lineType[, shift]]]) 
```

其中各种参数的意义如下：

- **img** – Image.
- **pt1** – Vertex of the rectangle.
- **pt2** – Vertex of the rectangle opposite to pt1 .
- **rec** – Alternative specification of the drawn rectangle.
- **color** – Rectangle color or brightness (grayscale image).
- **thickness** – Thickness of lines that make up the rectangle. Negative values, like CV_FILLED , mean that the function has to draw a filled rectangle.
- **lineType** – Type of the line. See the line() description.
- **shift** – Number of fractional bits in the point coordinates.

本文中绘制一个长方形从(0,0)到(255,255)，设置线的颜色为绿色(0,255,0)，线的粗细为5，其他参数默认，如下：

``` python
# draw a rectangle
cv2.rectangle(img, (0,0), (255,255), (0, 255, 0), 5)
```

## 绘制圆
opencv里自带的circle()函数可以绘制以一个点为圆心特定半径的圆，其函数的声明如下：

```python
cv2.circle(img, center, radius, color[, thickness[, lineType[, shift]]])
```

其中各个参数的意义如下：

- **img** – Image where the circle is drawn.
- **center** – Center of the circle.
- **radius** – Radius of the circle.
- **color** – Circle color.
- **thickness** – Thickness of the circle outline, if positive. Negative thickness means that a filled circle is to be drawn.
- **lineType** – Type of the circle boundary. See the line() description.
- **shift** – Number of fractional bits in the coordinates of the center and in the radius value.

本文中绘制的圆的圆心为(255,255)，半径为127，线的颜色为红色，线的粗细为8，如下：

``` python
# draw a circle
cv2.circle(img, (255, 255), 127, (0, 0, 255), 8)
```

## 绘制椭圆
调用ellipse()函数绘制椭圆，其声明为：

```python
cv2.ellipse(img, center, axes, angle, startAngle, endAngle, color[, thickness[, lineType[, shift]]]) 
```

其中各个参数的意义如下：

- **img** – Image.
- **center** – Center of the ellipse.
- **axes** – Half of the size of the ellipse main axes.
- **angle** – Ellipse rotation angle in degrees.
- **startAngle** – Starting angle of the elliptic arc in degrees.
- **endAngle** – Ending angle of the elliptic arc in degrees.
- **box** – Alternative ellipse representation via RotatedRect or CvBox2D. This means that the function draws an ellipse inscribed in the rotated rectangle.
- **color** – Ellipse color.
- **thickness** – Thickness of the ellipse arc outline, if positive. Otherwise, this indicates that a filled ellipse sector is to be drawn.
- **lineType** – Type of the ellipse boundary. See the line() description.
- **shift** – Number of fractional bits in the coordinates of the center and values of axes.

本文绘制了一个完整的椭圆，中心在(255,255)，长轴150，短轴75，旋转0°，起始角度0°，终止角度360°，线的颜色为黄色(0,255,255)，粗细为1。

``` python
# draw a ellipse
cv2.ellipse(img, (255, 255), (150, 75), 0, 0, 360, (0, 255, 255), 1) 
```

## 添加文字
opencv里的自带函数putText()可以在图像上加上文字，其声明如下：

```python
cv2.putText(img, text, org, fontFace, fontScale, color[, thickness[, lineType[, bottomLeftOrigin]]])
```

各个参数的意义如下：

- **img** – Image.
- **text** – Text string to be drawn.
- **org** – Bottom-left corner of the text string in the image.
- **font** – CvFont structure initialized using InitFont().
- **fontFace** – Font type. One of FONT_HERSHEY_SIMPLEX, FONT_HERSHEY_PLAIN, FONT_HERSHEY_DUPLEX, FONT_HERSHEY_COMPLEX, FONT_HERSHEY_TRIPLEX, FONT_HERSHEY_COMPLEX_SMALL, FONT_HERSHEY_SCRIPT_SIMPLEX, or FONT_HERSHEY_SCRIPT_COMPLEX, where each of the font ID’s can be combined with FONT_ITALIC to get the slanted letters.
- **fontScale** – Font scale factor that is multiplied by the font-specific base size.
- **color** – Text color.
- **thickness** – Thickness of the lines used to draw a text.
- **lineType** – Line type. See the line for details.
- **bottomLeftOrigin** – When true, the image data origin is at the bottom-left corner. Otherwise, it is at the top-left corner.

本文在图像上显示OpenCV这几个字，如下：

```Python
font=cv2.FONT_HERSHEY_SIMPLEX
cv2.putText(img,'OpenCV',(10,500), font, 4,(255,255,255),2)
```

## 显示图像
显示最终绘制完的图像：

```python
# show image
cv2.imshow("shape", img)

cv2.waitKey(0)
cv2.destroyAllWindows()
```


# 源代码
整个程序的源代码如下：

``` python
# created by Huang Lu
# 27/08/2016 17:59:31  
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

# empty image
img = np.zeros((512, 512, 3), np.uint8)

# draw a line
cv2.line(img, (0,0), (511,511), (255, 0, 0), 10)

# draw a rectangle
cv2.rectangle(img, (0,0), (255,255), (0, 255, 0), 5)

# draw a circle
cv2.circle(img, (255, 255), 127, (0, 0, 255), 8)

# draw a ellipse
cv2.ellipse(img, (255, 255), (150, 75), 0, 0, 360, (0, 255, 255), 1)

# add text
font=cv2.FONT_HERSHEY_SIMPLEX
cv2.putText(img,'OpenCV',(10,500), font, 4,(255,255,255),2)

# show image
cv2.imshow("shape", img)

cv2.waitKey(0)
cv2.destroyAllWindows()
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python/tree/master/Python_OpenCV/Draw_Shape/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![使用摄像头捕获视频并保存](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Draw_Shape/Screenshot.png)


# 参考
- http://docs.opencv.org/2.4/modules/core/doc/drawing_functions.html
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python/tree/master/Python_OpenCV/Draw_Shape/
- http://hlthu.github.io/blogs/PythonOpenCV/09-Draw_Shape.html