---
title: caffe-ssd安装教程 (ubuntu16.04+CUDA8.0+cuDNNv5.1+opencv3.2.0+contrib3.2.0+mkl+caffe)

date: 2017-10-28 23:01:46
tags: [caffe,文档,安装教程]
category: "技术文档"
---


## 1 ubuntu16.04安装  
1. 下载安装镜像，校内可从ustc镜像源下载：  

		mirrors.ustc.edu.cn

2. uefi安装可直接将镜像内文件复制至U盘，legency安装推荐使用UltraISO最新版刻制。  
3. 安装时推荐选择英文，以免后续使用出现编码等问题。  

4. 安装完成后，于System Settings中选择Software&Updates，选择Additional Drivers选项卡，出现选项后，在NVIDIA Corporation部分选择Using NVIDIA binary driver。  
5. 完成此部分

## 2.CUDA8.0 + cuDNNv5.1
	选择cuDNNv5.1是因为SSD源码使用的Caffe版本所限，使用更高版本的cuDNN会引起报错。

1. 下载CUDA8.0  

		https://developer.nvidia.com/cuda-downloads
![](http://i.imgur.com/8fmKb5t.png)
选项如上图所示。
2. 下载完成后，以管理员权限运行： 

		sudo ./cuda_8.0.61_375.26_linux.run
如出现无执行权限可先运行如下指令：

		sudo chmod a+x cuda_8.0.61_375.26_linux.run
再执行上述指令

3. 安装时可按空格加速阅读协议。请不要安装CUDA自带的显卡驱动，因为在上面已经安装了高版本驱动。其余均可选默认选项。
4. 安装完成后，将如下两行写入环境变量(~/.bashrc或/etc/profile)，两者均要用sudo权限更改。
		
		export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
		export PATH=/usr/local/cuda-8.0/bin:$PATH

5. 下载cuDNN  
下载cuDNN地址如下，需要注册NVIDIA开发者。
	
		https://developer.nvidia.com/cudnn
选择如下选项：（不要选择下方的Deb，那些是给Power8使用的）
![](http://i.imgur.com/NA05QR9.png)

6. 下载完成后解压，可以右键选择Extract here，也可以使用如下代码：
	
		tar -xzvf cudnn-8.0-linux-x64-v5.1.tgz

7. 解压后可以得到cuda文件夹，进入：

		cd cuda/include
		sudo cp cudnn.h /usr/local/cuda/include/
		cd ../lib64
		sudo cp lib* /usr/local/cuda/lib64/

8. 进入/usr/local/cuda/lib64目录，重新创建链接库：

		cd /usr/local/cuda/lib64/
		sudo rm -f libcudnn.so libcudnn.so.5
		sudo ln -s libcudnn.so.5.1.10 libcudnn.so.5
		sudo ln -s libcudnn.so.5 libcudnn.so

9. 完成

##3.MKL安装

	MKL	是intel的库，在使用intel处理器时会有更好的计算性能。

1. 下载英特尔® Parallel Studio XE：

		https://software.intel.com/zh-cn/qualify-for-free-software
选择对应选项，示例为学生。  
	1. 选择英特尔® Parallel Studio XE Cluster 版（包括 Fortran 和 C/C++），Linux版。  
![](http://i.imgur.com/X75Faqk.png)

	2. 在license界面选择所有选项后，选项accept。
	3. 在request界面填好email(注意要填学邮)，选好国家后submit
	![](http://i.imgur.com/jLG7hkV.png)
	4. 得到Serial number，下载离线包。
	![](http://i.imgur.com/JQn5umr.png)
2. 安装Parallel Studio XE：
	1. 解压安装包：

			tar -xzvf parallel parallel_studio_xe_2017_update4.tgz
	2. 进入文件夹，启动GUI安装（当然也可以选择非GUI安装，看个人喜好）
	 
			cd parallel_studio_xe_2017_update4
			sudo ./install_GUI.sh
	3. 按提示向后走，在licence步骤填入之前得到的serial number
	4. 继续完成安装，完成后结束。

## 4.OpenCV3.2.0 + contrib 3.2.0安装
1. 下载Opencv 3.2.0与 contrib 3.2.0

		wget https://github.com/opencv/opencv/archive/3.2.0.tar.gz
		wget https://github.com/opencv/opencv_contrib/archive/3.2.0.tar.gz
为防止混淆，建议将Opencv主文件夹命名为`opencv`，contrib命名为`opencv_contrib_3.2.0`

2. 安装依赖库：

		sudo apt-get install build-essential

		sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev cmake-gui

		sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev  

3. 在opencv目录下建文件夹build，进入后使用cmake-gui（如果对cmake指令熟悉也可直接用命令行使用cmake）
		
		mkdir build
		cd build
		cmake-gui .. 

4. 选择cmake的编译器后，确保CUDA、MKL、python、numpy等的路径正确，并在OPENCV_EXTRA_MODULES_PATH选项中填入上面contrib文件夹的model文件夹位置。如在示例中，位置为`/home/catlab/opencv_contrib_3.2.0/model`。并选择编译python接口（`BUILD_opencv_python2`当然此处是python2接口，python3接口也基本同理），选择`CMAKE_BUILD_TYPE`为Release。完成后点Configure按键，无报错后点Generate按键。  
如果出现`ippicv_linux_20151201.tgz`等文件无法加载，则下载同一文件放于终端所提示的路径。

5. 开始编译：

		make -j
		make all -j
		sudo make install

6. 完成

##5.Caffe+ssd安装
	本部分主要参考博客，http://blog.csdn.net/samylee/article/details/51822832
	以及caffe官方教程：http://caffe.berkeleyvision.org/installation.html

1. 安装Caffe依赖库

		sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev  libhdf5-serial-dev protobuf-compiler
		sudo apt-get install --no-install-recommends libboost-all-dev
		sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
		
2. 下载ssd版caffe
		
		git clone https://github.com/weiliu89/caffe.git
		cd caffe
		git checkout ssd

	使用时可能会出现git clone下载很慢的问题，可以直接下载zip，但是要注意下载时要选择branch为ssd，或者用此网址：

		https://github.com/weiliu89/caffe/tree/ssd
	
3. 编译caffe：  

		cd /home/**（您服务器的名字）/caffe
		cp Makefile.config.example Makefile.config
	修改Makefile.config中的内容，如加上cuDNN，将blas选为mkl等。

		mkdir build
		cd build
		mkdir cmake-gui ..
	同样记得处理cuDNN与blas相关的选项，完成后进行Configure与Generate。
	
		make all -j
		make install
		make runtest
		make pycaffe

4. 数据文件下载
	1.  VGGNet下载：
	2.  VOC2007,VOC2012下载：
	
5. 生成LMDB文件  
终端输入：  

		cd /home/**（您服务器的名字）/caffe
		./data/VOC0712/create_list.sh
		./data/VOC0712/create_data.sh
在运行第三步时如果出现no module named caffe或者是no module named caffe-proto，则在终端输入：

		export PYTHONPATH=$PYTHONPATH:/home/**（您服务器的名字）/caffe/Python

6. 训练模型
	1. 训练  
	打开caffe/examples/ssd/ssd_pascal.py这个文件，找到gpus=’0,1,2,3’这一行，如果您的服务器有一块显卡，则将123删去，如果有两个显卡，则删去23，以此类推。  
如果您服务器没有gpu支持，则注销以下几行，程序会以cpu形式训练。（这个是解决问题cudasuccess（10vs0）的方法）

			#Ifnum_gpus >0:
            # batch_size_per_device =int(math.ceil(float(batch_size) / num_gpus))
			#iter_size =int(math.ceil(float(accum_batch_size) / (batch_size_per_device * num_gpus)))
  			# solver_mode =P.Solver.GPU
  			# device_id =int(gpulist[0])

		保存后终端运行：

			cd  /home/**（您服务器的名字）/caffe
			python examples/ssd/ssd_pascal.py
		如果出现问题cudasuccess（2vs0）则说明您的显卡计算量有限，再次打开caffe/examples/ssd/ssd_pascal.py这个文件，找到batch_size =32这一行，修改数字32，可以修改为16，或者8，甚至为4（相信大家这个修改可以理解，我就不作说明了），保存后再次终端运行python examples/ssd/ssd_pascal.py
	
	2. 终端输入：
	
			python examples/ssd/score_ssd_pascal.py（演示detection的训练结果，数值在0.718左右）