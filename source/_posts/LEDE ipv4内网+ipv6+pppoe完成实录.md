---
title: LEDE ipv4内网+ipv6+pppoe完成实录
date: 2017-10-31 11:40:03
tags:["文档","LEDE","教程"]
category:技术文档
---
搬到北区以后呢，意外的发现楼里的宽带线路改成电信的了。因此办了电信的宽带。但是这样会有另一个不太好的结果，那就是接上路由以后上不了内网了。本着搞事的原则，在柳老司机的怂恿下，买了一个小米mini的路由器，刷了LEDE来做内外网融合。

## 小米mini路由器刷LEDE

首先说明一下，不是所有路由器都能方便地刷掉系统的，也不是所有路由器都有第三方固件可以让你刷的。因此想刷openwrt固件的同学可以点下面的链接查询：[table of hardware-openwrt](https://wiki.openwrt.org/toh/start)  

至于为什么我不刷Openwrt而是LEDE，因为柳老司机说LEDE内核比较新。而且好像Openwrt有第三方固件的，LEDE也都有。  

切入正题，我买的是小米Mini路由器，现在官方已经不卖了，想买的同学可以去买其他店里的存货，我买来是120元，算是个参考价吧。  

以下部分内容主要参考以下几个作者的教程：  

1. [MIUI论坛-【小米路由器mini刷机】mini刷潘多拉固件教程（固件已更新)](http://www.miui.com/thread-2036705-1-1.html)
2. [miwifi-ssh工具](https://d.miwifi.com/rom/ssh) --进入此页面需要登陆小米账号
3. [4c8t's blog-小米路由器mini刷OpenWrt/PandoraBox/LEDE](https://www.4c8t.com/archives/11)


### 1.给小米mini路由器开启ssh功能  

**注意**：开启ssh功能的同时，路由器将**没有保修**。请想清楚了再开始下面的行动。  

1. 把路由器固件刷成开发版的。

	ROM下载地址及刷机方法：[miwifi下载页面](http://www1.miwifi.com/miwifi_download.html)

	下拉找到小米mini固件开发版并下载。然后可以去路由器后台上传ROM刷机，也可以用U盘刷。刷机官方教程:[小米路由器mini U盘刷机 救变砖、重刷系统](http://bbs.xiaomi.cn/t-11720354)

2. 刷入ssh工具

	**再次提醒**：开启路由器的SSH功能带来的风险：**丧失安全稳定性，失去保修资格等**。请慎重考虑后在决定是否开启。

	访问页面：[miwifi-ssh工具](https://d.miwifi.com/rom/ssh)

	登陆小米账号后可以看到root密码,记住root密码,等下会用到。

	页面会显示刷入ssh工具的教程，请按照上面的方法进行：

	1. 请将下载的工具包bin文件复制到U盘（FAT/FAT32格式）的根目录下，保证文件名为miwifi_ssh.bin；
	2. 断开小米路由器的电源，将U盘插入USB接口；
	3. 按住reset按钮之后重新接入电源，指示灯变为黄色闪烁状态即可松开reset键；
	4. 等待3-5秒后安装完成之后，小米路由器会自动重启，之后您就可以尽情折腾啦 ：）
	
	请注意：确保你的U盘格式是FAT/FAT32格式的，且**没有**文件名为 **miwifi.bin** 的文件（不然可能就变成刷系统了）。同时，请**不要**用PE盘刷ssh功能。不保证成功。


### 2.刷LEDE并配置：

1. 刷机

	下载winscp和putty

	下载LEDE17.01.4:([LEDE firmware downloads](https://lede-project.org/toh/views/toh_fwdownload))

	运行winscp

	将文件协议SFTP修改为SCP

	主机名:192.168.31.1 用户名:root 密码:”获取ssh步骤中得到的密码”

	将LEDE上传至/tmp（[LEDE 17.01.4](https://downloads.lede-project.org/releases/17.01.4/targets/ramips/mt7620/lede-17.01.4-ramips-mt7620-miwifi-mini-squashfs-sysupgrade.bin)）

	运行putty

	主机名:192.168.31.1 用户名:root 密码:”获取ssh步骤中得到的密码”

	输入:
	```  
	mtd -r write /tmp/XXX.bin OS1
	```  
	（原教程写的是`mtd -r write /tmp/XXX.bin firmware`，但是不成功，查了以后发现小米路由改了命令。）

	XXX.bin为你上传的文件名

	完成后路由器将重启

2. 配置

	默认用户名为root 密码为admin 后台地址为192.168.1.1

	进入后台以后改密码，不然可能继续会无法ssh访问。

	运行putty

	主机名:192.168.1.1 用户名:root 密码:admin

		opkg update
		opkg install luci-i18n-base-zh-cn

	打开路由管理页面 依次点开

	System/System/Language and Style/Language

	选择普通话(Chinese)

