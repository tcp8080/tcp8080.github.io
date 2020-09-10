---
layout: post
title:  "[centos]How to config XiaoMi box working with kodi"
date:   2020-09-10 now
categories: centos
---

最近回归了美剧的怀抱，但是线上平台美剧资源太少了，电脑下载看了又不爽，于是想折腾一下小米盒子，让我能方便的在电脑下载剧集，在电视上观看。

调查了一下，通用做法是在小米盒子上装一个Kodi，然后通过网络协议去读取电脑的视频。

> **家里现有资源**：
小米盒子4，Mac，Centos 各一台

> **计划**：
在Centos上开smb共享，从Mac上下载电影到centos共享文件夹，最后小米盒子4通过Kodi去读取共享文件夹。


经过半天的设置，it works like a charm

记录一下配置流程，以备后续查阅。

**First of All, make sure all the machines works in the same wifi.**

### 1. 在Centos上安装smb
参见[config smb](https://tcp8080.github.io/centos/2020/05/21/how_to_install_centos7_into_a_very_old_win7_pc.html)


### 2. 在Macos上远程连接到Centos
#### 2.1 Mac通过ssh连接Centos
```
ssh centosUser@xx.xx.xx.xxx  //centos局域网IP
```

#### 2.2 Mac通过桌面连接Centos的smb共享目录

**打开“访达”，press cmd+k，输入centos上设置上smb账号**

有问题可以check这些：
- 共享文件夹权限
- systemctl status smb //smb服务的状态
- /var/log/messages
- /var/log/samba/log.smbd

> 在Mac连接Centos的smb共享目录的时候经常出现这个问题：
change_to_user_internal: chdir_current_service() failed!
add "force user = xxx" into /etc/samba/smb.conf
```
[Shared] 
   comment = Allow all users to read/write 
   path = /home/andrius/Shared 
   public = yes guest ok = yes 
   writable = yes 
   force user = smbusername
```
any other issue, you can check /var/log/samba/log.smbd, and google the error, all solution is provided.
Just remember, when you can't connect smb from Macos, check if you have finished your last connection.

### 3. 从小米盒子4连接到Centos的smb共享目录看电影
首先安装Kodi，小米盒子4上装个当贝市场，在市场里搜kodi直接安装。
打开Kodi，进入System-Settings-Appearance，将Fonts 从 Skin default 改为 Arial Based, 否则中文会变成方块乱码
然后在菜单中间的enter files section，进入，add movies，然后选择smb服务浏览即可。
