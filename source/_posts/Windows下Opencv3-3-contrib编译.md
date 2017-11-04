---
title: Windows下Opencv3.3+contrib编译
date: 2017-11-02 10:41:17
tags: ["教程","文档","Opencv"]
category: 技术文档
---
本文主要参考：[Install OpenCV 3 on Windows](https://www.learnopencv.com/install-opencv3-on-windows/)

使用软件：cmake-gui, visual studio 2015

OpenCV版本：opencv 3.3

python版本：python 2.7

首先在这里要说明的是，OpenCV3开始，将一些部分的功能转移到了contrib中，不再放在原来整个OpenCV的源码里，因此想要使用这部分功能，必须自己编译OpenCV和contrib。以及如果要使用opencv的python接口，请先装好numpy库。
## 安装Visual Studio 2015
此部分不多讲，请大家自行搜索安装VS2015，但请**注意**，Visual Studio 2015的默认安装是**不带编译器的**！！所以请自定义安装并选译编译器（在编程语言中选译visual c++）

## 安装CMake

1. CMake 下载：[CMake.org](https://cmake.org/download/)

2. 开始安装，并选择将CMake加入到路径中：

![](https://i.imgur.com/HbcBeDy.png)

3. 根据引导，完成安装。

## 下载OpenCV3.3 与OpenCV_contrib3.3

下载地址：
1. [OpenCV release](https://github.com/opencv/opencv/releases)
2. [OpenCV contrib release](https://github.com/opencv/opencv_contrib/releases)

**特别提醒：**OpenCV版本与contrib**版本要一致**，不然会有编译错误。

### 分别下载OpenCV与contrib，并解压。

## 用CMake生成Visual Studio工程：

### 打开Make并进行默认配置：

1. 打开CMake-GUI
2. 在where is source code栏填入OpenCV的位置：
3. 在where to build binaries栏填入编译完成内容的位置：

	![](https://i.imgur.com/DFOiWcD.png)

4. 点击Configure，选择配置：这里我们选择VS14 2015 Win64(此处可根据自己的系统与编译器选择)
	![](https://i.imgur.com/lJwNZfj.png)

5. 等待CMake读取完成（这一步可能会比较久，因为CMake会下载一些库）
	如果出现下载失败，参考此文：[编译OpenCV时，FFmpeg或ippicv下载不成功的解决方案](http://blog.csdn.net/yiyuehuan/article/details/52951574)

6. 完成:
 
	![](https://i.imgur.com/Dxf7fbN.png)

### 用Cmake选择编译选项：

1. 勾上“INSTALL_C_EXAMPLES” 和 “INSTALL_PYTHON_EXAMPLES”
2. 在“OPENCV_EXTRA_MODULES_PATH”填入contrib库中module文件夹的路径。在我这里是：E:/OpenSourceCode/OpenCV/opencv_contrib-3.3.1/modules

	![](https://i.imgur.com/4rqKpPQ.png)

3. 如要编译Python接口，请记得检查是否有并勾上了BUILD_opencv_python2的选项（python 3同理）
4. 配置完成后点Generate

## 编译OpenCV

### 打开VS工程：点击Cmake中的Open_Project按钮并选VS。

![](https://i.imgur.com/uuQnJwy.png)
### 编译OpenCV debug版：

1. 在上方选择Debug 和 x64。

	![](https://i.imgur.com/j8hcciG.png)

2. 在右边找到CmakeTarget下的INSTALL，右键选BUILD（中文版就是生成）。
3. 漫长的等待。
4. 完成。

### 编译OpenCV release版：

1. 在上方选择Release 和 x64。

	![](https://i.imgur.com/oPEUyre.png)

2. 在右边找到CmakeTarget下的INSTALL，右键选BUILD（中文版就是生成）。
3. 漫长的等待。
4. 完成。

## 测试：
### 将编译目录放PATH：
在PATH中添加：OPENCV_buildPATH\install\x64\vc14\bin。**请不要直接填入OPENCV_PATH，这里的意思是你的OPENCV路径**，如我的是：
E:\OpenSourceCode\OpenCV\opencv_build\install\x64\vc14\bin

### C++:
待更新

### Python:
打开Python,输入

```python
import cv2
```

