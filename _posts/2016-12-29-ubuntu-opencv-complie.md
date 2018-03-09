---
layout: post
title: 在ubuntu下编译安装opencv
date: 2016-12-29 22:27:30 +08:00
category: "opencv"
tags: [ubuntu, opencv]
---

本文介绍了如何在linux(本文是ubunut 16.04系统)下编译安装opencv(本文采用的opencv版本是3.1.0)，包括安装流程和可能遇到的问题，这些大部分都是基于我个人的经验总结和opencv官网。



# 1.安装过程

## 1.1 解决基本依赖
首先必须确保你的系统中安装了opencv编译过程中的基本软件包，如果没有请用管理员权限或者请你的服务器的管理员帮忙执行以下命令。

```
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

上述软件包在安装的过程中可能会有部分不存在，如果不存在，建议使用`apt-cache search <关键字>`进行搜索，如使用`apt-cache search tbb`来寻找有关libtbb2的软件包，找到合适的以后自行选择安装。

如果你想使用opencv的python接口，则需要安装numpy等。如果你想使用操作系统自带的python，那么需要使用管理员权限执行

```
sudo apt-get install python-dev python-numpy
```

如果你没有管理员权限，建议自己安装自己的python，安装miniconda或者anaconda，可以从[清华大学TUNA协会](https://tuna.moe/)的开源站点下载，anaconda所在网址为[https://mirrors.tuna.tsinghua.edu.cn/anaconda/](https://mirrors.tuna.tsinghua.edu.cn/anaconda/)。安装时请确保你的anaconda或miniconda的路径写在了`~/.bashrc`下，并执行`source ~/.bashrc`使更改生效。


下载安装后建议把conda的安装源改为清华的站点，即执行下面两条命令。

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```

接下来是使用conda来安装numpy等。

```
conda install numpy scipy
```

## 1.2 下载opencv3.1.0
可以选择从[opencv官网](http://opencv.org/downloads.html)下载，也可以选择使用`wget`获取。

```
wget https://codeload.github.com/opencv/opencv/zip/3.1.0
```


下载后解压。

```
unzip opencv-3.1.0.zip
```

## 1.3 cmake：进一步解决依赖
进入opencv源码的目录。

```
cd opencv-3.1.0
```

新建一个文件夹build(或者其他任何名字)，并进入。

```
mkdir build && cd build/
```

先cmake一下看还需要解决哪些依赖。

```
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/home/huanglu/usr/ ..
```

上面选择的是release模式，并告诉我要把opencv安装在/home/huanglu/usr/目录下，当然你也可以选择/usr/local等目录（这个计算机如果只有你一个人使用的话，建议这么做，如果你没有管理员权限而且这台机器有多人使用，建议安装在自己的home目录下）。

在cmake的过程中，你会发现有很多库没有找到，这个时候你就需要自己使用`apt-cache search <关键字>`来搜索这些库并安装，对于其他一些warning或者缺少的东西，可以选择使用google逐一解决。

某一次我cmake时发现缺少软件包libgphoto2。然后我使用

```
apt-cache search libgphoto2
```

来查找软件包，发现下面的libgphoto2-dev应该是我所需要的。然后我使用管理员权限或者请服务器管理员帮我安装。

```
sudo apt-get install libgphoto2-dev
```


这样，你可以基本上解决绝大部分依赖库，有极个别不是必须的，可以忽略，至于matlab接口等，只看个人需要进一步解决
。本文不再阐述，有意者可以继续google。

## 1.4 make：编译你的源代码
在编译源代码之前，建议在运行一次

```
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/home/huanglu/usr/ ..
```

然后根据自己的服务器CPU数量，可以选择合适的线程数，这里，我使用的是32线程。

```
make -j32
```

make的过程中可能会出错，而且错误百出，需要自行Google。不过我有时胡比较顺利，没有出错。


## 1.5 make install：安装
在make完需要执行

```
make install
```

来把opencv安装到刚才制定的目录，这里是`/home/huanglu/usr/`。

## 1.6 设置环境
如果你的opencv不是安装在`/usr/local/`下，而是在自己的home目录下的话，还需要设置环境变量。

打开`~/.bashrc`文件，在最后加两行

```
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:XXXX/lib/pkgconfig
export OpenCV_INCLUDE_DIRS=XXXX/inlcude:$OpenCV_INCLUDE_DIRS
```

在这里，`XXXX`代表opencv的安装路径，在本文中是`/home/huanglu/usr/`。即

```
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/huanglu/usr/lib/pkgconfig
export OpenCV_INCLUDE_DIRS=/home/huanglu/usr/inlcude:$OpenCV_INCLUDE_DIRS
```

然后使用下面的命令使更改生效。

```
source ~/.bashrc
```

# 2.测试例程
在build目录下，进入`../samples/cpp/example_cmake`。然后里面有三个文件： CMakeLists.txt、example.cpp、 Makefile。因此，直接执行下面三步即可。

```
cmake .
make
./opencv_example
```

# 3.常见错误

## 3.1 下载ippicv失败
在cmake的过程中，若出现 

```
– ICV: Downloading ippicv_linux_20141027.tgz… 
```

然后失败的问题。则自行google并下载 ippicv\_linux\_20141027.tgz，替换掉 `opencv-3.1.0/3rdparty/ippicv/downloads/linux-8b449a536a2157bcad08a2b9f266828b`下的同名文件即可。

## 3.2 没有cv2.so
安装完以后，发现在python中使用opencv时执行

```
import cv2
```

时出错，则可以把`opencv-3.1.0/build/lib`目录下的cv2.so拷贝到python的`site-packages/`中。

如果是anaconda或者miniconda，请拷贝至`~/miniconda2/lib/python2.7/site-packages`中；如果是系统的python，可以拷贝至`/usr/local/lib/python2.7/site-packages`中。

# 参考

1. [Introduction to OpenCV](http://docs.opencv.org/2.4/doc/tutorials/introduction/table_of_content_introduction/table_of_content_introduction.html)
2. [Installation in Linux](http://docs.opencv.org/2.4/doc/tutorials/introduction/linux_install/linux_install.html#linux-installation)
