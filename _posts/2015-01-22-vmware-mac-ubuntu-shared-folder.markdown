---
layout: post
title: "mac vmware ubuntu shared folder"
date: Thu Jan 22 13:06:07 2015
categories: study update
tags: vmware ubuntu shared floder
---

mac上在VMware Fusion虚拟机里面装了个ubuntu,/mnt/hgfs怎么尝试都看不到mac共享的目录，网上找了各种方法，重启了N次。

### 解决方案：

	apt-get install make gcc
	vmware-config-tools (除了hgfs选择yes,其他默认即可)
	
	
	
### 重新安装VMware tools

	mkdir /mnt/cdrom
	mount /dev/cdrom /mnt/cdrom
	cd /mnt/cdrom
	tar xzvf VMwareTools-8.8.3-682996.tar.gz -C /tmp
	cd /tmp/VMwareTools-8.8.3-682996
	./vmware-install.pl
	reboot
	ls /mnt/hgfs
	

### VMware Fusion 小技巧
	
	ctrl + command mac和虚拟机之间切换
	alt + 方向键  多个命令行界面切换
	
###  Linux 小tips

	sudo passwd root  #初次设置root密码
	apt-get install openssh-server #默认安装了openssh-client
	ps aux | grep ssh  #查看sshd进程
	vim /etc/ssh/sshd_config  #ssh server 配置
	service ssh restart #重启ssh服务
	PermitRootLogin yes #开启root登录 
	AuthorizedKeysFile #取消注释 开启ssh key文件登录
	apt-get -d openjdk-7-jre-lib #下载不安装 下载目录 /var/cache/apt/archives
	vim /etc/hostname #修改主机名
	
	







