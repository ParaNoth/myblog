---
title: ffmpeg发送rtsp流
date: 2017-12-05 00:39:41
tags: ["rtsp","ffmpeg"]
category: "项目记录"
---

之前记录过用ffmpeg做rtsp流的发送和接收，这里也记录一下以备后面还要用。

## 发送
用ffmpeg发送rtsp流要用ffserver。使用ffserver首先，要设置好config:

最简单的一版：
```
Port 9999
RTSPPort 9990
BindAddress 0.0.0.0
MaxClients 1000
MaxBandwidth 100000
CustomLog –
#只需要指定待播放的文件的路径以及格式信息即可
<Stream test.flv> #此处名称可以随便取，为rtsp地址后面的内容。如此处就写为rtsp://ip:9990/test.flv
    File "/tmp/test.flv"
    Format flv
</Stream>
```

然后再在终端里输入ffserver –f ffserver.conf