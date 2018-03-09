---
layout: post
title: ubuntu 16.04在线安装Kaldi
tags: [ASR, kaldi]
---

Kaldi是一个语音识别工具，使用C++开发，基于Apache 许可证，目的是为语音识别研究者提供。本文将介绍在线安装kaldi，为之后的离线安装kaldi做一个准备和基础。


# 1. 下载kaldi
目前kaldi是开源的，在github上可以clone；clone以后进入该目录，然后查看安装方法。

    git clone https://github.com/kaldi-asr/kaldi.git
    cd kaldi/
    cat INSTALL 

INSTALL文件指示如下：

    This is the official Kaldi INSTALL. Look also at INSTALL.md for the git mirror installation.
    [for native Windows install, see windows/INSTALL]
    (1) go to tools/  and follow INSTALL instructions there.
    (2) go to src/ and follow INSTALL instructions there.
    
INSTALL文件指示我们先去tools目录下，follow那里的INSTALL文件；然后再到src目录下，follow那里的INSTALL文件。

# 2. 编译tools

## 2.1 查看安装方法
进入tools目录，查看INSTALL文件。

    cd tools/
    cat INSTALL

INSTALL文件内容如下：

    To install the most important prerequisites for Kaldi:
     first do
      extras/check_dependencies.sh
    to see if there are any system-level installations or modifications you need to do.
    Check the output carefully: there are some things that will make your life a lot
    easier if you fix them at this stage.
    
    Then run
      make
    If you have multiple CPUs and want to speed things up, you can do a parallel
    build by supplying the "-j" option to make, e.g. to use 4 CPUs:
      make -j 4
    
    By default, Kaldi builds against OpenFst-1.3.4. If you want to build against
    OpenFst-1.4, edit the Makefile in this folder. Note that this change requires
    a relatively new compiler with C++11 support, e.g. gcc >= 4.6, clang >= 3.0.
    
    In extras/, there are also various scripts to install extra bits and pieces that
    are used by individual example scripts.  If an example script needs you to run
    one of those scripts, it will tell you what to do.


INSTALL文件指示我们应该：
 1. 检查依赖安装情况：`extras/check_dependencies.sh`
 2. 编译基本工具：`make -jn`
 3. 安装其他工具：`./extras/install_XXXXX.sh`

## 2.2 检查依赖情况
执行：

    ./extras/check_dependencies.sh

 结果提示我缺少`libtool`、`subversion`，还需要执行

    sudo ln -s -f bash /bin/sh

于是我安装库并执行上述命令：

    sudo apt-get install libtool subversion
    sudo apt-get install sox
    sudo ln -s -f bash /bin/sh

上面第2条命令是因为在跑swbd代码的时候需要。执行完之后再运行

    ./extras/check_dependencies.sh

 显示OK！
 
 
 
## 2.3 编译基本工具
执行

    make -j8

最终显示编译成功。


## 2.4 编译其他工具
先应该让tools/和tools/extras/目录下的.sh文件具有执行权限：

    chmod u+x *.sh extras/*.sh

然后依次执行

    ./install_portaudio.sh 
    ./install_pfile_utils.sh 
    ./install_speex.sh 
    ./install_srilm.sh 


最后一句先会提示下面的错误：需要自己到提示的网站下载srilm.tgz，放到tools/目录下。


提示没有安装`gawk`，执行下面命令后重新安装。

    sudo apt install gawk



# 3. 编译src
进入src/目录，查看安装方法：

    cd ../src/
    cat INSTALL 

INSTALL文件内容如下：

    These instructions are valid for UNIX-like systems (these steps have
    been run on various Linux distributions; Darwin; Cygwin).  For native Windows
    compilation, see ../windows/INSTALL.
    
    You must first have completed the installation steps in ../tools/INSTALL
    (compiling OpenFst; getting ATLAS and CLAPACK headers).
    
    The installation instructions are:
    ./configure --shared
    make depend
    make
    
    Note that "make" takes a long time; you can speed it up by running make
    in parallel if you have multiple CPUs, for instance
     make depend -j 8
     make -j 8
    For more information, see documentation at http://kaldi-asr.org/doc/
    and click on "The build process (how Kaldi is compiled)".

因此接下来只需要执行以下三条命令，一般情况下不会出错的。

    ./configure --shared
    make depend -j8
    make -j8

显示done即完成。


# 4. 运行yseno例子
执行

    cd ../egs/yesno/s5/
    ./run.sh

结果表明字错误率(WER)为0%，即正确率为100%。

