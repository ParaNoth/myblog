---
title: LEDE+ipv4内网+ipv6+pppoe完成实录
date: 2017-10-31 11:40:03
tags: ["文档","LEDE","教程"]
category: 技术文档
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

		mtd -r write /tmp/XXX.bin OS1
 
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


## ipv4内网+pppoe融合

此部分内容主要参考：[复旦pt-关于南区路由器连内网](http://pt.vm.fudan.edu.cn/index.php?topic=96649.msg1024380;highlight=%E8%B7%AF%E7%94%B1+%E8%A1%A8)。**此部分内容只有pt用户才能访问**
### 1.ipv4与ipv6地址获取
复旦北区公寓的电信线路上的内网地址是可以被DHCP分配的。而且ipv6的分配方式颇为诡异……

ipv4:将WAN口的接口之一设为DHCP客户端。  

![](https://i.imgur.com/oVJyilN.png)

ipv6:将WAN口的另一个接口设为DHCPv6客户端，并设参数为force,禁止：
![](https://i.imgur.com/ot0sVup.png)

### 2.新建接口pppoe拨号：
进入网络-接口界面，点击新建接口，随便取个名字，协议选择pppoe，包括接口选上网所用的接口，即你的ipv4与ipv6线的接口。
![](https://i.imgur.com/48Yajml.png)

在基本设置中填入拔号帐号密码。在防火墙设置中选wan。
![](https://i.imgur.com/VW1JCPB.png)

最终的接口界面如图：
![](https://i.imgur.com/wTRZ1JL.png)

### 3.设置路由表

**注意**，作为示例，我要说明一下，在我的图片中：内网ipv4=WAN_FDU，pppoe=WAN，ipv6=WAN6。如果参考我的方法请自行做好对应。

进入网络-静态路由表界面，填入下图的路由表：

![](https://i.imgur.com/UZ8ACeK.png)

接口请选ipv4的接口，此处就是wan_fdu。ipv4网关请自行对应自己的网关。

然后重新连接各接口。

## ipv6+nat设置
本部分主要参考内容：
[Padavan /Openwrt /LEDE下实现ipv6 nat /napt66](http://www.jianshu.com/p/eb07eaac6167)
[CSDN-在OpenWrt上配置原生IPv6 NAT](http://blog.csdn.net/cod1ng/article/details/45421025)
[TS的自留地-在OpenWrt上配置原生IPv6 NAT](http://tang.su/2017/03/openwrt-ipv6-nat/)

听各路大神说IPV6 NAT是一个很邪恶的东西，因为IPV6就是为了消灭NAT。但总之在路由器下面用这招才能上得去ipv6……

1. 安装`ip6tables`和`kmod-ipt-nat6`。

		opkg update
		opkg install ip6tables
		opkg install kmod-ipt-nat6
2. 使用WinSCP更改`/etc/config/network`文件内容,在`config interface 'lan'`下添加一行：

		option ip6addr 'fc00:100:100:1::1/64'

3. 更改`/etc/config/dhcp`文件,将`config dhcp 'lan'`那一栏改为以下内容:
```
config dhcp 'lan'
    option interface 'lan'
    option start '100'
    option limit '150'
    option leasetime '12h'
    option dhcpv6 'server'
    option ra 'server'
    option ra_management '1'
    option ra_default '1'
```
	
4. 更改`/etc/firewall.user`，假设WAN对应的接口为eth0.2，则添加以下内容：

		ip6tables -t nat -A POSTROUTING -o eth0.2 -j MASQUERADE

5. 在路由器下面的设备中用`tracepath6 -b tv.byr.cn`(linux)，`tracert tv.byr.cn`
(windows)命令，获取目前网络的IPv6网关地址(找最上面的IPv6地址)，假定是`2001:1234:1234:1234::1`。
	这一步，如果出现不能连接外网的情况，请在路由器上输入：
	```
uci set network.wan6.sourcefilter=0
uci commit network
ifup wan6
```

6. 使用`route -A inet6 add default gw 2001:1234:1234:1234::1`命令，为路由器添加默认网关。这一步非常重要，不进行的话是上不了IPv6的。。。完成之后，连接到路由器的计算机应该可以访问IPv6网站了。

	重启之后，需要重新添加网关，如果要做到路由器开机自动添加该网关,可以在/etc/hotplug.d/iface/下新建一个文件90-ipv6,给予可执行权限,内容为(注意替换为自己的网关地址)

``` bash
#!/bin/sh
[ "$ACTION" = ifup ] || exit 0
route -A inet6 add default gw 2001:1234:1234:1234::1

```
