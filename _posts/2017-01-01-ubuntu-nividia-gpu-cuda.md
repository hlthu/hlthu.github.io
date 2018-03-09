---
layout: post
title: ubuntu16.04上配置nvidia驱动、cuda和cudnn
date: 2017-01-01 22:27:30 +08:00
category: "cuda"
tags: [nvidia, cuda]
---

本文主要介绍在ubuntu16.04上配置nvidia驱动、cuda和cudnn，我的操作系统是ubuntu 16.04.1 LTS Desktop 64bit，GPU型号是Tesla C2075（一款相对较老的GPU）。本文的组织结构如下：


# 1.准备工作
由于在安装显卡驱动的过程中可能会出现重启后在登录界面反复重复的问题，因此后面可能会使用命令行的方式进行进一步设置。建议通过网络ssh的方式访问机器，记下机器的ip，同时安装openssh-server。

    sudo apt install openssh-server

同时需要将Intel自带的显卡功能禁用，这部分主要依靠在启动时修改BIOS，具体方法请google。然后检查显卡是否安装正常，可以将显示器接在显卡的显示接口上，检查是否正常显示。

# 2.安装显卡驱动

## 2.1 查看显卡型号
建议先到[参考3](http://www.nvidia.cn/Download/index.aspx?lang=cn)网站确定驱动的型号，但是请不要下载。

## 2.2 安装驱动
安装前先更新系统的软件包。

    sudo apt-get update
    sudo apt-get upgrade

然后查询NVIDIA显卡驱动。

    sudo apt-cache search nvidia*

从中选择一个自己需要的我安装的是nvidia-375。

    sudo apt-get install nvidia-375

使配置生效，并重启。

    sudo nvidia-xconfig
    sudo reboot

## 2.3 解决循环登录问题
重启后等我输完密码回车，过两秒又回到了登录界面，尴尬，似乎和显卡的驱动程序有关，我的解决方法主要参考了[参考1](http://www.jianshu.com/p/34236a9c4a2f?winzoom=1)。

由于进不去桌面，所以只能选择命令行模式或者通过网络访问。可以按`Ctrl+Alt+F1`进入命令行界面，如果不可用，可以通过ssh访问机器。z在命令行再依次执行下面的命令。

```
sudo apt-get remove --purge nvidia-*
sudo apt-get install ubuntu-desktop
sudo apt-get install linux-headers-generic
sudo apt-get update
sudo apt-get upgrade
sudo apt-cache search nvidia*
sudo apt-get install nvidia-367
sudo nvidia-xconfig
sudo reboot
```

重启之后就可以发现可以正常进入系统了，只是图像和字体比较大，但是无所谓，因为正常情况下我们很少使用图形界面。

## 2.4  检验驱动安装是否成功
分别执行以下两条命令。

    nvidia-smi
    nvcc -V

命令行的执行结果如下。

![检测驱动](https://raw.githubusercontent.com/hlthu/hlthu.github.io/master/images/posts/cuda/cuda-1.jpeg)

# 3. 安装cuda

## 3.1 下载cuda
到[参考4](https://developer.nvidia.com/cuda-downloads)所在的网址下载cuda8.0，根据自己的系统信息选择相应的版本，请选择runfile(local)类型，比较大。

![下载cuda8.0](https://raw.githubusercontent.com/hlthu/hlthu.github.io/master/images/posts/cuda/cuda-2.jpg)

## 3.2 安装CUDA
进入到刚才下载的cuda8.0目录，执行

    sudo sh cuda_8.0.44_linux.run

执行后会有一系列提示让你确认，但是注意，有个让你选择是否安装nvidia361驱动时，一定要选择否：

    Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 361.62?

因为前面我们已经安装了更加新的nvidia驱动，所以这里不要选择安装，其余的都直接默认或者选择是即可。 

## 3.3 环境变量配置
以下命令以普通用户执行即可。先编辑`~/.bashrc`。

    vim ~/.bashrc

然后在结尾加上3句。

    export PATH="/usr/local/cuda-8.0/bin:$PATH"
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"
    export CUDA_HOME=/usr/local/cuda


然后要使更改生效。

    source .bashrc

## 3.4 CUDA Sammples
下面可以测试一下cuda是否正常安装。进入cuda samples的安装目录，然后依次执行

    cd Path_To_Samples/1_Utilities/deviceQuery 
    make
    ./deviceQuery

如果显示一些关于GPU的信息，则说明安装成功。

# 4. 安装cudnn
cudnn是GPU加速计算深层神经网络的库，cudnn只支持computing capacity在3.0(含)以上的GPU上运行，所以，在安装cudnn之前请检查你的GPU版本是否支持cudnn。可以在NVIDIA官网检测，也可以参考[链接5](http://blog.csdn.net/JiaJunLee/article/details/52067962)。

在确定支持cudnn后可以到[NVIIDA官网](https://developer.nvidia.com/rdp/cudnn-download)下载。

下载完成后解压并将其拷贝至cuda的安装目录。

    tar xvzf cudnn-8.0-linux-x64-v5.1-ga.tgz
    sudo cp -P cuda/include/cudnn.h /usr/local/cuda/include
    sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda/lib64
    sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

至此，cuda和cudnn已经安装完成。



# 参考

 1. [Ubuntu 16.04安装NVIDIA驱动后循环登录问题](http://www.jianshu.com/p/34236a9c4a2f?winzoom=1)
 2. [Ubuntu16.04+CUDA8.0+caffe配置](http://blog.csdn.net/xuzhongxiong/article/details/52717285)
 3. [NVIDIA驱动下载](http://www.nvidia.cn/Download/index.aspx?lang=cn)
 4. [CUDA8.0 Downloads](https://developer.nvidia.com/cuda-downloads)
 5. [NVIDIA GPU的Compute Capability一览](http://blog.csdn.net/JiaJunLee/article/details/52067962)



