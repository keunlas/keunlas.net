---
title: 安装 Vmware 遇到的网络问题
tags: 
  - archlinux
  - vmware
  - network
comments: true
abbrlink: a5941e9c
date: 2025-01-04 17:53:32
categories: 折腾
---


最近在Archlinux上安装vmware虚拟机的时候, 虚拟机一直连不上网络. 经过网络搜索, 问题已解决. 安装和解决全过程如下.

# 通过aur安装vmware-workstation

```shell
yay -S vmware-workstation

# 注意依赖中的linux-header一定要安装
pacman -S linux-header
```

# 确保必要的模块和服务已启动

```shell
# 启动服务
# 用于虚拟机网络访问
systemctl enable --now vmware-networks.service
# 用于将 USB 设备连接到虚拟机
systemctl enable --now vmware-usbarbitrator.service

# 检查模块
modprobe -a vmw_vmci vmmon vmnet vmxnet3 vmw_vsock_vmci_tansport

# 如果有必要可以使用下面的命令重新编译模块
vmware-modconfig --console --install-all
```

# 解决没有网络的问题

当时安装到这一步时, 已经可以打开vmware, 并且也可以安装不同的系统在虚拟机里面了, 但是没有网络. 经过网络上的搜索, 找到了解决方法. 

通过下面这行命令启动虚拟机的网卡.

```shell
# 启动虚拟机的网卡
vmware-networks --start
```

最后成功解决虚拟机的网络问题!

