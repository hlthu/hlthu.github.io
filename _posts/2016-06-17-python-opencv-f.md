---
layout: post
title: Python OpenCV简单几何变换
date: 2016-06-17 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---




本节实现的是使用OpenCV里自带的函数，对图像进行简单的几何变换。

- 放大
- 缩小
- 平移
- 旋转


# 实现过程

## 读取原图并显示
不再赘述。

``` python
import cv2
import numpy as np

# read the original
img = cv2.imread('../test2.jpg')
cv2.imshow('original', img)
```

## 放大
利用OpenCV自带的resize()函数实现放大与缩小。其声明为：

```python
cv2.resize(src, dsize[, dst[, fx[, fy[, interpolation]]]]) → dst
```
其中各个参数的意义如下：

- **src** – input image.
- **dst** – output image; it has the size dsize (when it is non-zero) or the size computed from src.size(), fx, and fy; the type of dst is the same as of src.
- **dsize** –output image size; if it equals zero, it is computed as:
dsize = Size(round(fx × src.cols), round(fy × src.rows))
- **fx** –scale factor along the horizontal axis; when it equals 0, it is computed as
- **fy** –scale factor along the vertical axis; when it equals 0, it is computed as
- **interpolation** –interpolation method:

| 参数 | 意义 |
|--------|--------|
|INTER_NEAREST | a nearest-neighbor interpolation|
|INTER_LINEAR | a bilinear interpolation (used by default)|
|INTER_AREA | resampling using pixel area relation. It may be a preferred method for image decimation, as it gives moire’-free results. But when the image is zoomed, it is similar to the INTER_NEAREST method.|
|INTER_CUBIC | a bicubic interpolation over 4x4 pixel neighborhood|
|INTER_LANCZOS4 | a Lanczos interpolation over 8x8 pixel neighborhood|

本文将原图放大至原来的2倍。

```python
# expand
rows, cols, channels = img.shape
img_ex = cv2.resize(img, (2*cols, 2*rows), interpolation=cv2.INTER_CUBIC)
cv2.imshow('expand', img_ex)
```

## 缩小
这里将原图缩小为原来的一半。

```python
# zoom
img_zo = cv2.resize(img, (cols/2, rows/2), interpolation=cv2.INTER_AREA)
cv2.imshow('zoom', img_zo)
```

## 平移
平移可以由平移矩阵描述：

$$\begin{bmatrix}
   1 & 0 & t_x \\
   0 & 1 & t_y 
  \end{bmatrix} \tag{1}
$$

其中$$t_x$$和$$t_y$$分别为向右和向下平移的距离。这里我们利用np.array()创建这个矩阵，然后调用warpAffine来实现这个变换，并保持图像的大小不变。

```python
# trans
M = np.array([[1, 0, 50],[0, 1, 50]], np.float32)
img_tr =cv2.warpAffine(img, M, img.shape[:2])
cv2.imshow('trans', img_tr)
```

其中warpAffine()的声明如下：

```python
cv2.warpAffine(src, M, dsize[, dst[, flags[, borderMode[, borderValue]]]]) → dst
```

各个参数的意义如下：

- **src** – input image.
- **dst** – output image that has the size dsize and the same type as src .
- **M** – 2 × 3 transformation matrix.
- **dsize** – size of the output image.
- **flags** – combination of interpolation methods (see resize() ) and the optional flag WARP_INVERSE_MAP that means that M is the inverse transformation.
- **borderMode** – pixel extrapolation method (see borderInterpolate()); when borderMode=BORDER_TRANSPARENT , it means that the pixels in the destination image corresponding to the “outliers” in the source image are not modified by the function.
- **borderValue** – value used in case of a constant border; by default, it is 0.

## 旋转
利用getRotationMatrix2D()获得旋转矩阵，其声明为

```python
cv2.getRotationMatrix2D(center, angle, scale) → retval
```
各个参数的意义：

- **center** – Center of the rotation in the source image.
- **angle** – Rotation angle in degrees. Positive values mean counter-clockwise rotation (the coordinate origin is assumed to be the top-left corner).
- **scale** – Isotropic scale factor.
- **retval** – The output affine transformation, 2x3 floating-point matrix.

然后再利用warpAffine()函数进行变换。

```python
# Rotation
M=cv2.getRotationMatrix2D((cols/2,rows/2), 45, 1)
img_ro =cv2.warpAffine(img, M, img.shape[:2])
cv2.imshow('rotation', img_ro)
```


# 源代码
程序的源代码如下：

```python
# created by Huang Lu
# 2016/8/26 17:35
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

# read the original
img = cv2.imread('../test2.jpg')
cv2.imshow('original', img)

# expand
rows, cols, channels = img.shape
img_ex = cv2.resize(img, (2*cols, 2*rows), interpolation=cv2.INTER_CUBIC)
cv2.imshow('expand', img_ex)

# zoom
img_zo = cv2.resize(img, (cols/2, rows/2), interpolation=cv2.INTER_AREA)
cv2.imshow('zoom', img_zo)

# trans
M = np.array([[1, 0, 50],[0, 1, 50]], np.float32)
img_tr =cv2.warpAffine(img, M, img.shape[:2])
cv2.imshow('trans', img_tr)

# Rotation
M=cv2.getRotationMatrix2D((cols/2,rows/2), 45, 1)
img_ro =cv2.warpAffine(img, M, img.shape[:2])
cv2.imshow('rotation', img_ro)

# wait the key and close windows
cv2.waitKey(0)
cv2.destroyAllWindows()
```

也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV/Geometric_Trans/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Geometric_Trans/Screenshot.png)

# 参考
- http://docs.opencv.org/2.4/modules/imgproc/doc/geometric_transformations.html
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Python_OpenCV/Geometric_Trans/
- http://hlthu.github.io/blogs/PythonOpenCV/16-Geometric_Trans.html