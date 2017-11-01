---
title: LEDE+PPTP穿透
date: 2017-11-01 10:50:30
tags: ["文档","教程","LEDE"]
category: "技术文档"
---
表示使用了LEDE以后，做好校园网+宽带融合却发现布置在实验室的pptp VPN上不去了，这样就在寝室不能看文献了，就很烦。查了一下发现是因为LEDE和Openwrt在某版本之后就把pptp穿透功能去掉了。因此，表现就是在连PPTP的时候，能通过验证，但无法连上，最后会报619错误，也就是无法连接，该端口已被关闭。

## 解决方法:
以下内容主要参考：[Openwrt.pro-OpenWRT PPTP 穿透](http://www.openwrt.pro/post-328.html)

ssh连接路由器，输入如下两条指令，安装完成即可连接PPTP VPN。

	opkg update
	opkg install kmod-nf-nathelper-extra

