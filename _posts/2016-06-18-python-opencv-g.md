---
layout: post
title: Python OpenCV程序执行时间
date: 2016-06-18 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---



本节实现的是使用OpenCV里自带的函数，计算程序的执行时间。

- 获取系统时钟数
- 获取系统时钟频率


# 实现过程

## 初始时间
不再赘述，代码如下。

``` python
# time start
t1 = cv2.getTickCount()
```

## 执行代码
我这里执行的是之前绘制直方图的代码，请参考[我的博客](http://blog.csdn.net/huanglu_thu13/article/details/52332716)和[GitHub](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Hist_Display)。


## 结束时间
获取程序结束时间。

```python
# time end
t2 = cv2.getTickCount()
```

## 计算执行秒数
利用getTickFrequency()获取时钟频率。

```python
t = (t2-t1)/cv2.getTickFrequency()
print t
```


# 源代码
整个程序的源代码如下：

```python
# created by Huang Lu
# 2016/8/26 17:35
# Department of EE, Tsinghua Univ.

import cv2
import numpy as np

# get the hist graph of a gray image
def HistGraphGray(image, color):    
    hist= cv2.calcHist([image], [0], None, [256], [0.0,255.0])       
    histGraph = np.zeros([256,256,3], np.uint8)
    m = max(hist)
    hist = hist * 220 / m
    for h in range(256): 
       	n = int(hist[h])
        cv2.line(histGraph,(h,255), (h,255-n), color)        
    return histGraph; 
  
# get the hist graph of a color image
def HistGraphColor(image):
	histGraph = np.zeros([256,256,3], np.uint8)
	colorBlue = [255, 0, 0]
	colorGreen = [0, 255, 0]
	colorRed = [0, 0, 255]
	b, g, r = cv2.split(image)
	bhist = cv2.calcHist([b], [0], None, [256], [0.0,255.0])
	ghist = cv2.calcHist([g], [0], None, [256], [0.0,255.0]) 
	rhist = cv2.calcHist([r], [0], None, [256], [0.0,255.0])
	bm = max(bhist)
	gm = max(ghist)
	rm = max(rhist)
	bhist = bhist * 220 / bm
	rhist = rhist * 220 / rm
	ghist = ghist * 220 / gm
	for h in range(256):
		bn = int(bhist[h])
		gn = int(ghist[h])
		rn = int(rhist[h])
		if h != 0:
			cv2.line(histGraph,(h-1,255-bStart), (h,255-bn), colorBlue)
			cv2.line(histGraph,(h-1,255-gStart), (h,255-gn), colorGreen)
			cv2.line(histGraph,(h-1,255-rStart), (h,255-rn), colorRed)
		bStart = bn
		gStart = gn
		rStart = rn
	return histGraph

# main fuction
if __name__ == '__main__':
	# time start
	t1 = cv2.getTickCount()
	
	# test for a gray image
	img1 = cv2.imread("../test1.jpg", 0)
	color = [255, 255, 255]
	histGraph1 = HistGraphGray(img1, color)
	cv2.imshow("Hist Gray", histGraph1)

	# test for a color image
	img2 = cv2.imread("../test2.jpg")
	# first tset for three channels
	colorRed = [0, 0, 255]
	colorGreen = [0, 255, 0]
	colorBlue = [255, 0, 0]
	b, g, r = cv2.split(img2)
	# blue channel
	bhistGraph = HistGraphGray(b, colorBlue)
	cv2.imshow("Hist Blue", bhistGraph)
	# green channel
	ghistGraph = HistGraphGray(g, colorGreen)
	cv2.imshow("Hist Green", ghistGraph)
	# red channel
	rhistGraph = HistGraphGray(r, colorRed)
	cv2.imshow("Hist Red", rhistGraph)
	# get three channels together
	histGraph2 = HistGraphColor(img2)
	cv2.imshow("Hist Color", histGraph2)
	
	# time end
	t2 = cv2.getTickCount()
	t = (t2-t1)/cv2.getTickFrequency()
	print t
	
	cv2.waitKey(0)    
	cv2.destroyAllWindows()
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Run_Time/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果。显示结果如下：

![结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Run_Time/Screenshot.png)

根据上图，程序用时0.103647207s。

# 参考
- OpenCV-Python-Toturial-中文版.pdf
- https://github.com/hlthu/Python-OpenCV-Learn/tree/master/Run_Time/
