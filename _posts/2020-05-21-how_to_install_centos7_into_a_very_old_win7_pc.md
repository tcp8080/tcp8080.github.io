---
layout: post
title:  "[centos]How to install centos7 into a very old win7 computer"
date:   2020-05-21 now
categories: centos
---

最近做了个小服务，需要在linux下频繁登陆网站发布消息，决定在家里老电脑上装个linux来运行服务。
选中了Centos，并启用samba服务。在macos上开发，发送到centos上布署，现在稳定运行了，记录一下步骤。

### Minimal安装：
1. prepare a Centos7 iso USB or CD（ 可以使用minimal iso，那样就可以用小容量u盘or CD刻录）
2. 进入Win7系统，找到磁盘管理，清空除了系统盘之外的其他盘，删除卷，得到一个块大区域的未使用空间
3. 进入BIOS，设置usb启动。（如果是光盘，那不需要此步）
4. 重启电脑，进入centos7安装
5. 如果是minimal安装，安装完成是命令行，可以在命令行做其他软件or桌面系统安装。

### 在装完Minimal之后：
1. 找一根网线，将电脑有线连到路由器（否则系统无法联网）
2. 理论上插上网线之后就联网了，ping www.baidu.com试一下
3. 修改/etc/sysconfig/network-scripts/ifcfg-网卡名 文件，ONBOOT=yes （此步也可等之后进入桌面系统再改）
如果ping不通，重启一下服务（命令行输入：service network restart，或者service stop network再service start network）
确定能ping通了，开始安装软件。

### 软件安装：
1. yum install net-tools  //网络工具包, 装完才可以使用dig, nslookup, ipconfig等命令
2. yum install wget 
3. 安装图形化界面
   yum groupinstall "X Window System"
   yum groupinstall "GNOME Desktop" //装完图形化界面之后，输入startx，既可以进入桌面系统
   systemctl set-default graphical.target //设置重启默认使用桌面系统

4. yum grouplist 查看一下，继续安装一些其他软件，也可以不装，等需要再装。
   我安装了 Development Tools/Compatibility Libraries/Legacy UNIX Campatibility
   
### 需要使用WIFI：
拔掉网线，命令行输入 systemctl start NetworkManager，可以在桌面右上角看到无线图标
如果有问题，restart一下，或者stop之后重新start一下，ping www.baidu.com理论上没问题。

### Samba配置：
目的：为了开放一个区域可以让家里多台电脑直接访问存取。

#### 1. 设置静态IP
在 /etc/sysconfig/network-scripts/ifcfg-网卡名 和etc/sysconfig/network-scripts/ifcfg-wifi名
这两个文件里同时加上以下几行，并删除BOOTPROTO=dhcp，其中的ip地址等可以在局域网其他电脑上获取
```
BOOTPROTO=static
IPADDR=192.168.20.200   # IP地址
GATEWAY=192.168.0.1     #网关
NETMASK=255.255.255.0    # 子网掩码
DNS1=114.114.114.114    # DNS服务器
DNS2=8.8.8.8         # 备用DNS服务器
```
重启网络，systemctl stop NetworkManager/ systemctl start NetworkManager
重启之后如果发现无法上网，可以关掉network试试：service network stop
确定可以上网之后，输入ifconfig，获取ifcfg-wifi名文件里对应的静态IP
这个静态ip地址之后其他电脑使用smb访问时需要输入

#### 2. 安装samba并启动，设置开机自动
yum install samba samba-client
systemctl start smb.service 
systemctl enable smb.service

#### 3. 在centos里设置需要共享的文件夹
mkdir /home/xxx/smb
chmod 777 /home/xxx/smb

#### 4. 临时关闭SELINUX，并设置防火墙
setenforce 0
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload

#### 5. 修改配置文件/etc/samba/smb.conf，增加这段
```
[share]
    path = /home/xxx/smb
    available = yes
    browseable = yes
    public = yes
    writable = yes
```

#### 6. 给centos系统用户开放samba权限
smbpasswd -a username1
然后会提示你输入两次密码再确定

### 从Macos对Centos系统里的samba文件夹读写
在Finder或者桌面，输入cmd+k
在弹出框里输入之前设置好的smb://静态IP，输入centos里设置的samba用户名和密码，
就可以在macos里挂载/home/xxx/smb，自由读写了。



