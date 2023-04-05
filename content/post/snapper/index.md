---
title: "用Snapper自动管理btrfs快照"
slug: snapper
description: 合理利用btrfs快照喵
date: 2023-04-05T16:23:08+08:00
hidden: false
comments: true
categories: 
    - Tech
tags: 
    - toss
---

前天因为在疲惫的情况下操作很多东西， 一不小心把要dd进USB的ArchLinux LiveCD给dd到了系统盘里面。。。
命令执行的时候速度达到了1.8GB/s， 我看了一眼dd的of参数，心中一阵凉意

> 我的记忆就这样被清除了吗。。。。

好在我还有备份， 不至于太糟， 只不过需要耗费时间来reinstall了

我尝试把作业系统完全还原到之前的状态，花费时间3hour

但是装回来之后， 我感到这个UI过于熟悉， 动起了翻新系统的念头

因为不久之前我在折腾飞腾的时候配置了Fedora， 配好之后还蛮好看的。。

真的心动了。。。

于是就真的开始动手了。。

把刚装好的Arch又完全抹掉，一手拿着[Installation Guide](https://wiki.archlinux.org/title/Installation_guide) 操作Livecd, 从头开始安装了一遍Arch, 花费时间2hour

~~ 才不会因为忘记安装的时候忘记安装链接Internet的包而需要重新进入Livecd装包呢 ~~

~~ 才不会因为贪心有线网络的一点加速而耗费更多时间来让有线链接正常工作呢 ~~

这样操作一次真的耗时好多的。。。 不知道我在玩Rock5B的时候怎么会又耐心来来回回这样操作几十次的。。

在另一台设备上面打开[General Recommendations](https://wiki.archlinux.org/title/General_recommendations)， 给黑漆漆的TTY注入灵魂

## Btrfs子卷规划

要构建一个可以持久使用的作业系统的话， 需要有一个良好布局的文件系统来承载它

snapper推荐的文件系统布局如下
```
subvolid=5
  |
  ├── @ -|
  |     contained directories:
  |       ├── /usr
  |       ├── /bin
  |       ├── /.snapshots
  |       ├── ...
  |
  ├── @home
  ├── @snapshots
  ├── @var_log
```

`@`是根目录的子卷 `@home`是家目录的子卷， `@var_log`挂载在`/var/log`

`@snapshot`是专用于快照的子卷， 挂载在`/.snapshot`

上面所有的子卷前面都有一个`@`符号表示这些子卷都直属于subvolid=5的顶层子卷

## Snapper

```
# pacman -S snapper 
```

snapper是一个辅助管理btrfs子卷的工具

可以自动化地定期对配置的子卷进行快照， 而且还可以清除过时的子卷

如果要把某个子卷交给`snapper`进行快照，可以像下面这样写：
```
# snapper -c root create-config /
# snapper -c home create-config /home
```

子卷的路径写子卷的挂载点就好

我一开始操作的时候还以为要把子卷的顶层目录挂载上， 使用由顶层子卷索引的目录才可以让`snapper`进行快照操作

默认情况下， `snapper`将会在根目录子卷下面创建一个`/.snapshot`的子卷来存放所有快照

这个子卷的路径如下图所示

```
subvolid=5
|
|
 ---- @  # 挂载在/
      |
	  |
	  .snapshots # 挂载在/.snapshots
```
这样存放快照有一个弊端， 如果后续需要回滚的话， 回滚之后会丢失掉所有的快照

要解决这个问题的话 可以为创建快照单独创建一个直属于顶层子卷的子卷`@snapshot`

在一个正在运行的作业系统上面创建一个子卷的话， 可以先把顶层子卷找一个地方挂载上去再创建子卷

```
# mkdir /btrfs
# btrfs subvolume create /@snapshot
# mount -o subvol=@snapshot /dev/nvme1n1p2 /.snapshots
```

### 通过定时器来实现自动化操作
snapper可以通过`cron`或者`systemd timer`来定时触发快照
但是如果同时启用了`cron`和`systemd timer`的定时服务， 那么会重复创建两次快照

我用了systemd timer来触发， 同时为了避免后面装了cron的实现之后出锅
把snapper使用cron的定时文件删除掉

在`/etc/pacman.conf`里面添加一下内容：
```
NoExtract = etc/cron.daily/snapper etc/cron.hourly/snapper
```


## snap-pac

```
# pacman -S snap-pac
```

snap-pac实现了通过`pacman`包管理器的hook函数来进行系统更新前后的自动化快照

它的配置文件在`/etc/snap-pac.ini`
是一份注释良好的文件

里面所有的内容初始状态下都是被注释的 如果没有任何有效的配置内容， `snap-pac`将以默认配置运行

默认状况下 这个小工具将会对`snapper`的root配置进行snapshot 

如果你的snapper里面有这个名为root的配置的话 不需要任何其他配置，`snap-pac`就可以正常工作了

配置文件里面详细写了每个参数的作用和它的默认值， 如果想要设定的值和默认值一样， 这项配置留空就可以

下面放一下我的`/etc/snap-pac.ini`文件

```
# snap-pac example configuration file
# see snap-pac(8) for more details

# Each section corresponds with a snapper configuration. Add additional sections to add
# other configurations to be snapshotted. By default, only the root configuration is snapshotted.
# Create a section named [DEFAULT] to have a setting apply for all snapper configurations

## Uncomment to set parameters for snapper configuration named root
[root]

## How many characters to limit the description for snapper.
## Default is 72
desc_limit = 72

## Whether or not to take snapshots of this snapper configuration
## Default is True for root configuration and False for all other configurations
snapshot = True

## What snapper cleanup algorithm to use
## Default is number
cleanup_algorithm = number

## Pre snapshot description.
## Default is the pacman command that triggered the hook
#pre_description = pacman pre snapshot

## Post snapshot description.
## Default is the list of packages involved in the pacman transaction
#post_description = pacman post snapshot

## Uncomment to add "important=yes" to userdata for snapshots referring to these packages
## Default is []
important_packages = ["linux"]

## Uncomment to add "important=yes" to userdata for snapshots that were created with the following commands
## Default is []
important_commands = ["pacman -Syu"]

## Add custom userdata. Each key-value pair should be an item in the list
## Default is []
#userdata = ["key=value","foo=bar"]

## Example for another snapper configuration named "home"
# [home]
## Default is False
# snapshot = True
```

配置好之后， 每次使用`pacman`进行包的操作， snapper都会对设定的子卷进行快照
