---
layout: post
title: ubuntu 16.04离线安装kaldi
date: 2017-02-06 23:44:30 +08:00
category: "kaldi"
tags: [kaldi, asr]
---

在很多科研单位和企业研发部门，服务器是极少被允许连外网的，一般情况下只可以通过内网里的机器访问服务器。由于自己的经历，我曾经接触过这样一台服务器，由于有8张Tesla K80的GPU，我很想在上面安装一个kladi。因此，这里的离线指的是服务器不能访问外网，或者服务器压根不能上网。我下面的操作是在和服务器在同一内网下的Windows机器上进行的，读者也可以直接在服务器上操作（不过一般很难得到许可）。

在阅读本文之前，建议先阅读一下我之前写的文章：[ubuntu 16.04在线安装kaldi](http://hlthu.github.io/2017/01/01/ubuntu-install-kaldi-with-network/)，以熟悉kaldi安装的大概过程。

# 1. 下载kaldi并上传

从kaldi在github的代码仓库（[https://github.com/kaldi-asr/kaldi](https://github.com/kaldi-asr/kaldi)）下载源码。然后上传到服务器上并解压。



# 2. 解决依赖环境

先检查环境是否齐全。

```
cd KALDI-ROOT-PATH/tools/
./extras/check_dependencies.sh
```

如果OK，那就可以，否则就按提示解决依赖。需要提醒的是虽然服务器不能访问外网，但是大多数服务器都配备了自己的软件包source，因此使用`sudo apt-get install XXX`进行软件安装是行得通的。由于还可能需要以下软件包，再执行：

```
sudo apt-get install sox
```

# 3.下载并上传kaldi的tools

之前在[ubuntu 16.04在线安装kaldi](http://hlthu.github.io/2017/01/01/ubuntu-install-kaldi-with-network/)一文中，我们可以直接在tools目录下make，以实现工具的编译，但是需要的工具的源代码是从互联网获取的。由于这里服务器不能访问外网，因此必须把tools工具提前下载好，读者可以从[我的网站下载](http://hlthu.github.io/downloads/kaldi/tools.zip)。

下载完成后解压，并上传到tools目录下。

# 4.编译tools

编译的过程和文章[ubuntu 16.04在线安装kaldi](http://hlthu.github.io/2017/01/01/ubuntu-install-kaldi-with-network/)中编译tools一样，只不过在此之前需要修改一下Makefile文件，把该文件中的下面几行注释掉。

```
distclean:
	rm -rf openfst-$(OPENFST_VERSION)/
	rm -rf sctk-2.4.10/
	rm -rf sctk
	rm -rf ATLAS/
	rm -rf sph2pipe_v2.5/
	rm -rf sph2pipe_v2.5.tar.gz
	rm -rf atlas3.8.3.tar.gz
	rm -rf sctk-2.4.10-20151007-1312Z.tar.bz2
	rm -rf openfst-$(OPENFST_VERSION).tar.gz
	rm -f openfst
	rm -rf libsndfile-1.0.25{,.tar.gz} BeamformIt-3.51{,.tgz}
```

这部分只要是清除之前的软件包，由于我们刚上传了，而不是从网络获取，因此千万不能清除。之后直接执行make即可。

```
	make -j n
```

# 5.编译src
和文章[ubuntu 16.04在线安装kaldi](http://hlthu.github.io/2017/01/01/ubuntu-install-kaldi-with-network/)中的一样。

```
	cd ../src/
	cat INSTALL
	./configure --shared
	make depend -j8
	make -j8

```


# Reference
* [ubuntu 16.04在线安装kaldi](http://hlthu.github.io/2017/01/01/ubuntu-install-kaldi-with-network/)