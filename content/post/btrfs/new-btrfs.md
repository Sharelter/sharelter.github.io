---
title: "让ArchLinux运行在Btrfs上！"
slug: Install archlinux with Btrfs
date: 2023-03-12T09:21:48Z
image: btrfs-logo.webp
math: false
license: CC-BY-NA-SA
hidden: false
comments: true
categories: 
    - Tech
draft: false
---

## 缘起

最近主力laptop上面的Arch要爆根目录了
`/`仅剩的900+MB 已经不能完成Arch的每日一滚了

因为我在安装这个Arch的时候使用了固定的分区方式，
而且根目录只分配了20GB, 使用桌面作业系统的话
很轻松就可以吃光这点空间

我在使用的磁盘里面没有留下空余的未分配空间，
比较好的方法只有重装整个OS

那么方法确定了， 那就开搞吧

之前的分区方式大概如下表
```
/dev/nvme0n1---
             |
             --- partation1 - /root(ext4, 20G)
             |
             --- partation2 - /home(f2fs, 460G)
```

在求教c10s之后，她扔出了如下表情
![recommend-btrfs](btrfs-recommend.webp)

嗯。。被强烈安利了btrfs

之前也有听过btrfs这个东东，但是~~因为懒~~还没有用过

上Archwiki, 找到btrfs的[wikipage](https://wiki.archlinux.org/title/Btrfs)
发现自带很多高级的功能 例如逻辑卷管理、快照、RAID ...etc.

看起来很好耶，上车！

## 创建文件系统

先在磁盘上面创建一个btrfs的文件系统
` sudo mkfs.btrfs -L mylabel /dev/nvme0n1p2 `

btrfs也可以以霸占整个磁盘的方式进行使用， 此时这个磁盘的分区表就由btrfs接管了

当然，因为我还需要ESP partation来启动， 就不可以使用这种方式

这里我的磁盘是分好了区的， 一个2GB的FAT32分区来实现UEFI启动 剩下的所有空间给btrfs

操作到这一步， 已经可以继续下一步往这些文件系统上面安装系统了

但是， 这样使用btrfs不能用上它的许多高级功能
此时如果直接挂载这个分区的话， 可以享受到btrfs的CoW(写时复制) 特性来减少闪存设备的损耗，还可以启用压缩

继续操作， 需要使用用户空间的btrfs工具`btrfs-progs`
在ArchLinux上面要安装它，只需
```sudo pacman -S btrfs-progs```

## btrfs子卷

创建好btrfs文件系统之后， 就拥有了一个`ID=5`的顶级子卷

创建一个空闲的目录(例如`/btrfs`) 把文件系统的顶级子卷挂上，
就可以用 `btrfs subvolume create /path/to/subvolume`来创建子卷

这里， 我为`/root, /home, /swap`这三个目录在顶级子卷下面创建了三个子卷
```
btrfs subvoleme create /btrfs/subvol_root
btrfs subvolume create /btrfs/subvol_home

```

Archwiki推荐为swap文件单独创建一个子卷直属于顶级子卷的子卷来挂载:
`btrfs subvolume create /btrfs/@swap`

btrfs-progs 6.1版本之后，提供了一个直接创建swap文件的命令:
`btrfs filesystem mkswapfile --size 8g /btrfs/@swap/swapfile`

可以通过`--size`来定义创建的缓存文件的大小，这样创建出来的swap文件就可以直接挂载了～

通过`btrfs subvolume list -p /mnt` 可以列出文件系统下面的子卷

```
ID 256 gen 10069 parent 5 top level 5 path subvol_root
ID 257 gen 10069 parent 5 top level 5 path subvol_home
ID 258 gen 9195 parent 5 top level 5 path @swap
ID 259 gen 17 parent 256 top level 256 path subvol_root/var/lib/portables
ID 260 gen 18 parent 256 top level 256 path subvol_root/var/lib/machines
```
上面是我的子卷列表



## 挂载

在挂载的时候加上`subvol=/path/to/subvolume`或者`subvolid=?`，btrfs子卷就可以像文件系统分区一样被挂载

推荐创建一个子卷来挂载根目录，这样会为以后更改子卷布局和快照带来非常大的便利

我添加了`autodefrap`自动除碎片， `compress=zstd`启用zstd压缩方式，`discard=async`启用SSD-TRIM 三个参数

因为是在安装过程中操作， 挂载的选项都会被`genfstab`自动插到`/etc/fstab`当中

我的`/etc/fstab`如下:

```
# /dev/nvme0n1p2 LABEL=Arch
UUID=efeb8313-4210-4499-a805-d0f97e57c549	/         	btrfs     	rw,relatime,compress=zstd:3,ssd,discard=async,space_cache=v2,autodefrag,subvolid=256,subvol=/subvol_root	0 0

# /dev/nvme0n1p1
UUID=45BB-F7D1      	/boot     	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro	0 2

# /dev/nvme0n1p2 LABEL=Arch
UUID=efeb8313-4210-4499-a805-d0f97e57c549	/home     	btrfs     	rw,relatime,compress=zstd:3,ssd,discard=async,space_cache=v2,autodefrag,subvolid=257,subvol=/subvol_home	0 0

# /dev/nvme0n1p2 LABEL=Arch
UUID=efeb8313-4210-4499-a805-d0f97e57c549	/swap     	btrfs     	rw,relatime,compress=zstd:3,ssd,discard=async,space_cache=v2,autodefrag,subvolid=258,subvol=/@swap	0 0

/swap/swapfile      	none      	swap      	defaults  	0 0
```

