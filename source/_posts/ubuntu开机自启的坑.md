---
title: ubuntu开机自启的坑
date: 2017-12-04 21:32:55
tags: ["开机自启","ubuntu使用"]
category: "项目记录"
---

昨天甲方发现老康神给的自启脚本要登陆了以后才会启动，但是现在使用TX1，开机以后不加载GUI是不会自动登录的，这样就会带来问题。昨天查了一下，发现是因为，自启脚本放在profile.d下面了。

profile.d的加载是在登陆以后才会加载的，因此如果在开机就加载，只能把指令写在rc.local里面。

但是rc.local里面的指令会以root权限执行。之前也是这个原因不使用rc.local。因此，找了一个办法：

先贴流程：

### 1.移动profile.d中的test.sh移动到主目录下：

```bash
sudo mv /etc/profile.d/test.sh /home/nvidia/test.sh
```
### 2. 给test.sh开放权限：

```bash
sudo chmod 777 /home/nvidia/test.sh
```
### 3.在/etc目录下放rc.local文件：

```bash
sudo vim /etc/rc.local
```

内容写如下内容：

```bash
#!/bin/sh -e

/bin/su - nvidia -c "bash /home/nvidia/test.sh>1.log 2>&1 &"
exit 0
```

### 4.给rc.local可执行权限

``` bash
sudo chmod +x /etc/rc.local
```

### 5. 重启测试
```bash
su - 用户名 -c "指令内容"
```

上面的指令是用于以特定用户权限执行指令。这样就不会因为权限不同导致一些问题。

同时，有时候在/etc下面会没有rc.local文件，但在/etc/init.d下面会有，实际上，那个rc.local会因为/etc下面存在rc.local而不执行里面的内容，同时，里面默认是case语句，所以，在末尾加上内容是无效的，不会被执行。

