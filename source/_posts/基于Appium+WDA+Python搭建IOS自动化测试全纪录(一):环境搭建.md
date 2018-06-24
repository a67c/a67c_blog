---
layout: 基于Appium+WDA+Python搭建IOS自动化测试全纪录(一):环境搭建
title: 基于Appium+WDA+Python搭建IOS自动化测试全纪录(一):环境搭建
date: 2018.01.10 20:40
tags:
    - Appium环境搭建
---

#### 写在前面
计划将整个自动化搭建过程全部纪录一下，从环境搭建到模拟器跑demo，到真机跑demo  

当前测试及环境跑通日期为2018.1月

**本文要点:**
* appium ios环境搭建  python环境搭建
* Mac下 appium 1.7.2  python 2.7

#### appium环境搭建
* 安装Xcode，安装xcode-command-line-tools
```
终端中输入以下命令：xcode-select --install 
```
* 安装brew

    [官网地址](https://brew.sh/)

    终端中输入
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
* 安装libimobiledevice
```
brew install libimobiledevice --HEAD 
```
* 安装carthage
```
brew install carthage
```
* 安装node [官方的地址](https://nodejs.org/en/download/) 下载.pkg文件安装

* 安装cnpm

安装cnpm(由于某种原因,直接用npm下载安装会有好多网络问题，安装淘宝的cnpm要比npm好用)https://npm.taobao.org/

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
* 安装ios-deploy
```
cnpm install -g ios-deploy
```
* 安装xcpretty
```
gem install xcpretty
```
* 安装java环境

(1)下载JAVA安装包：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

(2)配置JAVA_HOME环境：

在~/.bash_profile

修改如下：
```

export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home   

export PATH=$JAVA_HOME/bin:$PATH 

export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
* 安装appium，appium-doctor

如果cnpm也卡了，想办法连vpn~
```
cnpm install -g appium
cnpm install -g appium-doctor
```
这之后执行appium-doctor
如果ios部分都变绿了就ok了。其中Android_Home,adb之类的变红是安卓的环境，对于ios没有什么影响。如果报错缺少了哪里的安装，直接搜安装方法就好。
![image.png](http://upload-images.jianshu.io/upload_images/1094385-fec7f2f94bbfc472.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Python环境搭建

由于采用Python来写自动化测试脚本，因此需要搭建Python的相关环境

采用了Python 2.7

使用pip方式安装
```
brew install pip
pip install lxml
```
安装Python-client
```
pip install Appium-Python-Client
```
搭建或者运行过程中出现xxx not defined  之类的，可能是模块缺失，查一下是哪些模块  补一下就好，由于是在整个流程跑通之后回头做的纪录，所以可能有些地方纪录不完善，错误之处欢迎提出。

那么环境搭建之后，通过启动app跑脚本即可达到流程跑通，所以接下来说如何跑脚本。

----
[Tbc]








 


