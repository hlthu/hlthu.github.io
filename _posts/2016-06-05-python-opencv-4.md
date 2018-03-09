---
layout: post
title: Python OpenCV显示直方图
date: 2016-06-05 22:27:30 +08:00
category: "opencv"
tags: [Python, OpenCV]
---


本节实现的是提取出灰度图像和彩色图像的直方图。

- 显示灰度图像的灰度直方图
- 显示彩色图像各个通道的灰度直方图
- 在一幅图上显示三个通道的灰度直方图


# 实现过程

## 引用与打开图片
不再赘述，代码如下。

``` python
import cv2  
import numpy

img1 = cv2.imread("test1.jpg", 0)	#灰度图像
img2 = cv2.imread("test2.jpg")	   #彩色图像
```

## 灰度图像直方图
opencv里自带了calcHist()函数，可以计算一幅图像中各个像素值出现的次数，其函数的各个参数如下：

``` python
hist = cv2.calcHist([image],#图像
    				[0], #使用的通道
   				 None, #没有使用mask
    				[256], #HistSize
    				[0.0,255.0]) #直方图柱的范围
```

但是这个函数的返回值只是从0~255这256个像素值出现的次数，我们还需要将其转化为可以显示的直方”图“。为此，我们生成了一个255x255的彩色图像，在横轴对应的列上显示一条与灰度出现次数成正比的直线，这里利用了cv库里的line()函数，并指定线条的颜色为输入的color参数。这部分被我封装成了一个函数，代码如下：

``` python
def HistGraphGray(image, color):    
    hist= cv2.calcHist([image], [0], None, [256], [0.0,255.0])       
    histGraph = np.zeros([256,256,3], np.uint8)
    m = max(hist)
    hist = hist * 220 / m
    for h in range(256): 
       	n = int(hist[h])
        cv2.line(histGraph,(h,255), (h,255-n), color)        
    return histGraph; 
```

下面对读入的灰度图像显示其直方图，显示线条的颜色为白色，代码如下：

``` python
color = [255, 255, 255]
histGraph1 = HistGraphGray(img1, color)
cv2.imshow("Hist Gray", histGraph1)
```


## 分离彩色图像的三通道并显示各通道直方图
利用split()函数分理出三个通道，分别调用上面的函数，显示颜色分别为红色、绿色和蓝色，代码如下：

``` python
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
```

## 在一幅图中绘制彩色图像的灰度直方图
实现起来与分离三通道相似，但是需要注意的是把三幅树状图绘制在一张图上会彼此掩盖，导致现实的结果比较混乱，因此我采用的是显示包括，即连点成线：将每一个通道的当前像素值出现的次数与前一个像素值出现的次数连接起来，三幅图用三个颜色，最后形成是三个通道的直方图。这部分也被封装成了一个函数，代码如下：

``` python
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
```

然后调用这个函数，绘制彩色图像的灰度直方图：

``` python
histGraph2 = HistGraphColor(img2)
cv2.imshow("Hist Color", histGraph2)
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
	# test for a gray image
	img1 = cv2.imread("test1.jpg", 0)
	color = [255, 255, 255]
	histGraph1 = HistGraphGray(img1, color)
	cv2.imshow("Hist Gray", histGraph1)

	# test for a color image
	img2 = cv2.imread("test2.jpg")
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
	cv2.waitKey(0)    
	cv2.destroyAllWindows() 
```
也可以参考我的GitHub上的，点击[这里](https://github.com/hlthu/Python/tree/master/Python_OpenCV/Hist_Display/)。

# 运行结果
在命令行进入该源程序所在目录后，运行`python main.py`后即可显示结果，灰度图像的直方图结果如下：
![运行结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Hist_Display/gray.png)

彩色图像的直方图结果如下：
![运行结果](https://raw.githubusercontent.com/hlthu/Python-OpenCV-Learn/master/Hist_Display/color.png)

# 参考
- http://blog.csdn.net/sunny2038/article/details/9097989
- https://github.com/hlthu/Python/tree/master/Python_OpenCV/Hist_Display