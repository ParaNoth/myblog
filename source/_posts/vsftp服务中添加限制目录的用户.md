---
title: 在Ubuntu的vsftp服务中添加限制目录的用户
date: 2018-03-14 00:21:36
tags: ["文档","Opencv","Ubuntu"]
category: 网管日志
---



接了达达学长的网管，结果交表第一天就出现了任务：老师想要在ftp上开一个用户，重新上传和修改ftp上的文件。本来以为这是非常轻松的事，以为加个用户加个权限就行了，结果忙活了一晚上才弄出来。现在也算做个记录，免得以后再踩进坑里。

## ftp服务

一开始学长说ftp是在samba下面的，我查了以后觉得非常奇怪，samba的访问方法是`smb://`，ftp的访问方法是`ftp://`，这两个好像完全不一样。再查到现在很多ftp用的都是vsftpd服务进行搭建，所以开始在`/etc`下面找vsftpd文件夹，结果没找到，一时以为没有这个服务。

但是，运行如下指令以后给了我答复，让我百思不得其解：

```shell
$ sudo service vsftpd status
```

然后发现了如下的重点：

---

### Caution:

**在Ubuntu系统下，vsftpd的config等配置文件是在`/etc`文件夹下的，没有单独的文件夹。**

因为实验室的系统是Ubuntu不是Centos的，所以很多网上的教程都不是能直接弄过来的。

前置任务：ftp服务的搭建，这一步网上教程很多，在此不赘述了。参考教程：[ubuntu安装ftp服务器（一般配置）](http://blog.csdn.net/nation_chen/article/details/7066277)。这篇教程里面除了有的地方把两个路径：`/srv/ftp`和`/home/ftp`混淆了，看的时候注意一下就OK。

---

## 正篇：给ftp添加特定目录下有权限的用户

###情景设定：

1. 假设你的ftp目录是：`/srv/ftp`，下面有两个目录`/srv/ftp/upload`和`/srv/fpt/download`。
2. 你希望新建一个用户，假定名称为`ftpuser`，权限为能仅能在`/srv/ftp/upload`目录下上传文件。
3. 你的config文件位置是`/etc/vsftpd.conf`。

### 步骤

#### 1.修改vsftpd.conf文件

主要是如下几行：

```bash
# 是否能够使用本地用户
local_enable=YES
# 是否用户用户上传文件
write_enable=YES
# 是否能够切换到根目录之外(YES则不能随意切换到根目录以外)
chroot_local_user=YES
# 只有目录中的账户才能切换
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
pam_service_name=vsftpd
```

创建文件`/etc/vsftpd.chroot_list`而这个文件中的用户则是一人一行，文件中的用户可以访问到任何目录。在我们的环境下，我们不能将ftpuser加入这个文件中。

#### 2.创建特定的用户  

由上面的设置我们可以知道，本地用户在ftp登陆时将只能访问到自己的根目录，所以要创建用户并将用户的根目录设在`/srv/ftp/upload`。

---

##### Caution:

注意在`/etc/pam.d/vsftpd`文件中，可以看到如下内容：

```bash
# Standard behaviour for ftpd(8).
auth	required	pam_listfile.so item=user sense=deny file=/etc/ftpusers onerr=succeed

# Note: vsftpd handles anonymous logins on its own. Do not enable pam_ftp.so.

# Standard pam includes
@include common-account
@include common-session
@include common-auth
auth	required	pam_shells.so
```

注意三个细节：

```bash
sense=deny 
file=/etc/ftpusers 
auth	required	pam_shells.so
```

第一行说明，在第二行所在文件中的用户都会被禁止访问ftp服务。第三行则说明，登陆用户需要有shell。

不然连接ftp时会报错：530 Login incorrect.

---

由此可知，我们需要创建一个**有shell，根目录为`/srv/ftp/upload`**的用户。

运行指令创建用户并设置好ftpuser的密码：

```bash
sudo useradd -d /home/nation/ftp/upload -s /bin/bash -M ftpuser
sudo passwd ftpuser
```

---

##### Caution:

不要听从网络上的教程将`/etc/vsftpd.conf`里面的`pam_service_name=vsftpd`改成其他值或者删除`/etc/pam.d/vsftpd`。这会导致安全性问题，即你无法将root等高级权限帐号排除在ftp服务外，这将导致可以使用root登陆ftp，引起一些不必要的麻烦。

---

#### 3.登陆ftp并测试

登陆ftp并上传文件即可。

