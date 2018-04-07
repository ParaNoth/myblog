---
title: Pycharm远程调试使用服务器资源
date: 2018-03-19 20:28:51
tags: ["Pycharm", "远程调试"]
category: 技术文档
---

 

由于实验室现在只有两台高配的显卡加速电脑，但最近显然想用的人越来越多了，现在僧多粥少，那就用ssh吧，但是vim又不是大家都用得下来，不过大家似乎现在都在用Python，那么就可以非常高兴的让大家都用上Pycharm，然后就可以用远程调试来进行编程，这样可以一方面保留IDE界面，另一方面又用上显卡资源。

下面开始正式的教程。

本教程主要参考机器之心的教程：一步步从零开始：[使用PyCharm和SSH搭建远程TensorFlow开发环境](https://www.jiqizhixin.com/articles/2017-03-18)

## 第一步 安装Pycharm Professional

下载地址：[PyCharm下载中心](https://www.jetbrains.com/pycharm/download/)

安装方法及激活方法略。

## 第二步 新建项目并配置

### 新建新项目

File>New Project。然后选好Location即可，解释器由于后面要换成远程的所以不用管。

### 改变解释器

打开 Preferences > Project >Project Interpreter, 点击右上角的设置符号，点击Add remote。

![IM截图2018031922040](Pycharm远程调试使用服务器资源\TIM截图20180319220408.png)

![IM截图2018031922045](Pycharm远程调试使用服务器资源\TIM截图20180319220459.png)

![IM截图2018031922052](Pycharm远程调试使用服务器资源\TIM截图20180319220528.png)

点击 SSH Credentials 按钮然后输入你的信息。填好host，帐号，密码，选好python解释器位置。

![IM截图2018031922055](Pycharm远程调试使用服务器资源\TIM截图20180319220553.png)

点击OK > Apply。

### 开始部署

由于远程调试实际上是将本地的文件和脚本上传到远程端运行的，所以需要进行配置。【当然，这些过程是自动进行的，所以看起来和在本地跑一样。】

1. 点击 File > Setting 选择 Build, Execution,Deployment > Deployment > Options, 确保勾选了Create empty directories。这样创建文件夹，Pycharm就会自动同步

   ![IM截图2018032009525](Pycharm远程调试使用服务器资源\TIM截图20180320095255.png)

2. 点击Build, Execution, Deployment > Deployment，然后点+按钮，选择SFTP，然后进行命名，点击OK：

   ![IM截图2018032010060](Pycharm远程调试使用服务器资源\TIM截图20180320100602.png)

3. 然后在Connection页面中输入远程机器的IP，帐号和密码。然后点击Test SFTP connection，如果成功，再点击Autodetect自动定位用户根目录。

   ![IM截图2018032010121](Pycharm远程调试使用服务器资源\TIM截图20180320101216.png)

4. 选择Mapping选项卡，设置远程的文件夹位置，这个路径是相对于前面的Root Path的。也可以点击后边的...按钮进行手动浏览。

   ![IM截图2018032010163](Pycharm远程调试使用服务器资源\TIM截图20180320101636.png)

5. 最后设定完全以后Apply > OK，然后点击 Tools > Deployment > Automatic Upload，确认其被勾选：

   ![截图](Pycharm远程调试使用服务器资源\TIM截图20180320102120.png)

6. 设置完成后可以用Tools > Deployment > Sync with Deployed to xx进行同步。xx是你的远SFTP host的名称。

### 设置控制台

1. 打开File > Setting。然后选择Build, Execution, Deployment > Console > Python console 然后点击Environment variables的...按钮：

   ![IM截图2018032013240](Pycharm远程调试使用服务器资源\TIM截图20180320132403.png)

   输入如下内容：

   ```shell
   DISPLAY=localhost:10.0
   LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
   CUDA_HOME=/usr/local/cuda
   ```

   ​

   ​

2. 选择Build, Execution, Deployment > Console > Python console 然后选择 Always show the debug console。这在我们调试的时候非常有用：

   ![IM截图2018032013441](Pycharm远程调试使用服务器资源\TIM截图20180320134414.png)

   ​

   

## 第三步 配置脚本

对于每个py文件，需要为其配置相应的运行配置。

点击Run > Edit Configurations...，直接修改对应脚本的配置。主要还是Environment Variables的配置，加上上面的内容：

```shell
DISPLAY=localhost:10.0
LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
CUDA_HOME=/usr/local/cuda
```

![IM截图2018032014074](Pycharm远程调试使用服务器资源\TIM截图20180320140743.png)

![IM截图2018032014121](Pycharm远程调试使用服务器资源\TIM截图20180320141210.png)

## 第四步 打开带X转发的SSH信道（非必要）

如果想要使用带有GUI的脚本，比如opencv的imshow函数等，需要打开带有X转发的SSH信道，即要先连接上一个SSH信道。

windows下，本人使用的是MobaXterm。这是一个个人版免费的windows下带X转发的SSH客户端。

下载地址：[MobaXterm](https://mobaxterm.mobatek.net/download.html)

这样，就可以用MobaXterm转发GUI。





