---
title: "用Virt-Mnanger来配置qemu hostfwd端口转发"
description: Configure hostfwd via virt-manager
slug: qemu-hostfwd-config
date: 2023-03-30T12:20:52+08:00
image: 
math: false
comments: true
categories: 
    - Tech
tags: 
    - toss
---

尝试用qemu/kvm装一台Ubuntu 22.04来跑一点服务， 通过[这个](https://wiki.archlinux.org/title/Virt-Manager)配置了virt-manager的qemu-user-session

~~(反复reboot了几次才成功跑起来，都是因为我没有把manual看全就急着up and running)~~

这个virtualhost是要对外提供服务的嘛，因为qemu的user-network是通过NAT的，这里就需要一点端口转发的魔法

开始的思路是用macvtap或者bridged的network
但是我的nic是一块intel AX211 wirelesscard, 而macvtap和bridge这种bridged的网络模式都不太支援无线网路

创建bridge的话还要干掉主机的network-manager

那么就另寻出路吧。。

询问了一下`@ziyao`姐姐， 得到道具`hostfwd`

去DuckDuckGo Search了一下`qemu hostfwd`找到了[这个](https://unix.stackexchange.com/questions/124681/how-to-ssh-from-host-to-guest-using-qemu)
这个post有166k的访问量诶， 可见这个需求是十分常见的

但是这个使用qemu的cli来运行的virtualhost, virt-manager是用XML格式的配置文件来管理每个virtualmachine的config的

继续Search`how to port forward ssh in virt manager`，在StackOverFlow上面找到了[这个](https://unix.stackexchange.com/questions/350339/how-to-port-forward-ssh-in-virt-manager) 嗯， 就是我想要的东西

~~又是因为太着急了， 没有看全manual就去操作~~，导致浪费了一些时间来找virt-manager的config文件在什么地方

翻了几篇post，得知要用`virsh edit`来编辑，这时候回来再看这个指南， 发现怎样编辑virtualhost的配置已经在第一行写明了呜呜呜

```
virsh -c qemu:///session edit ubuntu18.04
```
这就是stackoverflow那篇post上面给出的编辑virtualhost配置的方法

ubuntu18.04是创建的virtualhost的名字

接下来先把原有的nic的长得像下面这样子的配置去掉

```
<interface type='user'>
  <mac address='52:54:00:52:35:ff'/>
  <model type='rtl8139'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```

在xml的根节点里加上qemu的命名空间:

配置文件里面原本是这样子的
```
<domain type='kvm'>
```
插入之后是酱紫

```
<domain type='kvm' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
```
把这段插入到domain的根节点之间：
```
<qemu:commandline>
  <qemu:arg value='-netdev'/>
  <qemu:arg value='user,id=ubuntu-net,net=10.0.10.0/24,dhcpstart=10.0.10.10,hostfwd=tcp::10080-:80,hostfwd=tcp::10443-:443,hostfwd=tcp::22222-:22'/>
  <qemu:arg value='-device'/>
  <qemu:arg value='virtio-net,netdev=ubuntu-net,addr=0x10'/>
</qemu:commandline>
```

`-netdev`定义了一个qemu虚拟网络, 然后再通过`-device`参数， 定义一个虚拟网卡接入到刚才定义的网络当中去

保存退出， 运行一下这个虚拟机

发现新添加的nic的pcie address和已有的虚拟vga设备冲突了

原来是没有指定在pcie总线上面的地址，因为这个nic是通过命名行参数添加的
自动分配到第一个pcie地址上面去了

而第一个地址又被xml里面定义的其他设备使用了

所以用`addr=`分配一个没有被使用的地址就可以了！

改好之后果然就可以fireup and running了， 但是`ip link`发现虚拟网卡是`LINK DOWN`的状态

再去StackOverFlow搜索一下，`sudo dhclient $netdevice`

网络就可以用了

想用`network-manager`来配置， 但是发现ubuntu server默认不是用的NM,而是netplan

编辑`/etc/netplan/00-installer-config.yaml`:

把网络设备名字改成当前的问题就解决了
