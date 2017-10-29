---
title: Hexo建站记录(VPS+git+apache2)
date: 2017-10-29 13:28:29
tags: [Hexo, 文档, 教程]
category: "技术文档"
---

&emsp;&emsp;因为种种原因买了一个VPS，但是觉得钱都花了我总不能只用来做一件事，然后就决定弄个博客。但是，因为没钱买高配的VPS，只有512M内存，觉得如果用来跑wordpress可能负荷太重，决定弄个轻量级的，百度了几次以后，决定就用Hexo。一来是觉得它好看，二来是推荐的人也多。  

&emsp;&emsp;但是我还是被坑了，因为网上大量大量大量的教程全是在教你用github+Hexo去建站。关于用VPS建的还是比较少的，不过还是有的，贴一下我找到的一些资源和教程吧：  

<small>
1. Hexo官网：[Hexo中文](https://hexo.io/zh-cn/)  
2. NexT主题：[NexT主题](http://theme-next.iissnan.com/getting-started.html)  
3. SegmentFault教程：[SegmentFault](https://segmentfault.com/3.a/1190000008091729)  
4. Dacader Luo的教程：[Dacader Luo的博客](http://decader.cn/2017/03/24/%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%83%A8%E7%BD%B2Hexo%E5%8D%9A%E5%AE%A2/)  
5. 回风的教程：[Deploy Hexo sites to VPS](http://huifeng.me/2015/08/16/deploy-hexo-to-vps/)  
6. NexT github:[hexo-theme-next](https://github.com/iissnan/hexo-theme-next)
</small>

&emsp;&emsp;在开始，我要向和我一样从小白开始折腾这些的同学们说一句，Hexo的轻量级是体现在它是一个**静态**的框架。换句话说（我不确定我说得对不对），实际上它只需要在服务端布署生成好的网页就OK了。当然你也可以选择在VPS上跑服务然后反向代理过去。但这样Hexo的优势就体现不出来了。  


&emsp;&emsp;在这里总的框架是，在本地（自己的PC）上布署Hexo，然后在VPS上布署git仓库，用git来提交页面给VPS上的仓库，用Hooks来实现每次git push后会自动把页面转交给apache2。

![结构示意图](https://i.imgur.com/sqHuPsK.png)

## 1. 在本地安装Hexo  

在这部分里，主要参考Hexo官网([Hexo中文](https://hexo.io/zh-cn/))的内容。  

按照Hexo官网内容来就好了，但是，国内的用户在运行到npm指令的时候可能会遇到一些麻烦，你会发现npm指令运行得特别特别慢。原因很简单：npm因为墙的存在变得很慢，有时候还会失败。因此，推荐在国内使用时可以用cnpm来代替npm指令。什么是cnpm可以参考[淘宝npm镜像](http://npm.taobao.org/)。  

	npm install -g cnpm --registry=https://registry.npm.taobao.org
	cnpm install [name]
在完成安装cnpm以后，就可以用cnpm完成除了npm publish以外的几乎所有功能。

## 2.  NexT主题的安装   

这部分内容主要参考NexT的官网与SegmentFault上的教程。  
　　[NexT主题](http://theme-next.iissnan.com/getting-started.html)   
　　[SegmentFault](https://segmentfault.com/3.a/1190000008091729)    

从github上克隆最新版。  

	$ cd <folder>
	$ git clone https://github.com/iissnan/hexo-theme-next themes/next
将网站配置文件_config.yml中的theme关键字成next。 
 
	theme: next 
先用`hexo clean`来清除缓存。  

运行`hexo s --debug`来启动本地站点，开启调试模式。  

启动后可以在访问`http://localhost:4000`来看本地的页面。  

其他主题定制可参考NexT官网说明，在此不赘述。

## 3.  VPS上配置：
本部分主要参考：  
<small>
1. [Dacader Luo的博客](http://decader.cn/2017/03/24/%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%83%A8%E7%BD%B2Hexo%E5%8D%9A%E5%AE%A2/)  
2. [回风.me:Deploy Hexo sites to VPS](http://huifeng.me/2015/08/16/deploy-hexo-to-vps/)
</small>

如何购买VPS及配置VPS在此我不赘述了，相关教程有很多。  
在开始之前，首先确保的你的VPS上有git以及Apache或者Nginx

### 创建专用的用户
1. 添加用户  

		useradd -m git

2. sudo权限  
		
		vim /etc/sudoers
	找到如下片段并添加git的内容：  

		## Allowroot to run any comment anywhere
		root    ALL=(ALL)    ALL
		git     ALL=(ALL)    ALL

### 创建部署静态文件的Git仓库

切换到git用户`su git`，然后初始化git用户环境：
	
	cd ~
	mkdir .ssh && cd .SSH
	touch authorized_keys
	vim authorized_keys

---
  
**这部分要在本地进行** 

- 在本地上产生公钥：  

	在shell上输入（windows用户请用git bash）:
		
		ssh-keygen -t rsa -C "blog"
	然后一路回车。  

	然后进入.ssh文件夹：
		cd ~/.ssh
		ls
	可以看到`id_rsa`,`id_rsa.pub`，`id_rsa.pub`就是**公钥**。

---
然后在`authorized_keys`填入之前得到的公钥里的内容，并保存。  

在git的主目录下(/home/git)创建一个目录：  

	mkdir /home/git/blog.git

初始化git仓库：
	
	cd /home/git/blog.git 
	git init --bare

---
**这部分要在本地进行**  

- 修改hexo配置文件中的url内容：

		url: http://服务器ip
- 修改hexo配置文件中的deploy选项：

		deploy:
  			type: git
  			repo: git@12.34.56.78:blog.git,master

	这里的git 12.34.56.78是你的VPS的IP，也可以填写你VPS的域名。后面跟的blog.git就相当于绝对路径：/home/git/blog.git/  

- 运行`hexo g` `hexo d`，如果正常的话，静态文件可以被push到blog仓库。 

		如果这里出现了报错，
		$ hexo d 
		ERROR Deployer not found: git
		那么在要在本地安装hexo的git插件：
		npm install --save hexo-deployer-git

---

## 配置Apache2(Ubuntu)
其他版本的Linux发行版请注意包管理和服务管理指令的不同。

	sudo apt-get install apache2  

创建新的网站根目录/var/www/hexo

	sudo mkdir /var/www/hexo

修改默认根目录为/var/www/hexo

	sudo vim /etc/apache2/sites-available/000-default.conf

找到如下字段

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www

改为：

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/hexo #修改的地方

重启Apache生效：

	sudo service apache2 restart

## 创建Git hooks
本部分内容主要参考Decader Luo的教程：[Dacader Luo的博客](http://decader.cn/2017/03/24/%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%83%A8%E7%BD%B2Hexo%E5%8D%9A%E5%AE%A2/)

在服务器上的裸仓库 blog.git 创建一个钩子，在满足特定条件时将静态 HTML 文件传送到 Web 服务器的目录下，即 /var/www/hexo。在自动生成的 hooks 目录下创建一个新的钩子文件：

	touch /home/git/blog.git/hooks/post-receive
	vim /home/git/blog.git/hooks/post-receive

当中添加两行：

	#!/bin/bash
	git --work-tree=/var/www/hexo --git-dir=/home/git/blog.git checkout -f

保存并退出文件，并让该文件变为可执行文件

	chmod +x /home/git/blog.git checkout -f

同时，要确保git用户有能更改/var/www目录下内容的能力。

	chown git:git -R /var/www

到此应该基本就可以完成了~