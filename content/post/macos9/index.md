---
title: "安装MacOS9.2.2中文版"
description: macos9-install
date: 2022-12-29T19:38:57+08:00
image: check.jpg
math: false
license: 
hidden: false
comments: true
categories:
    - Tech
tags: 
    - Retro
    - toss
---

因为我手边只有一台在laptop上面拆下来的DVD Writer, 而且这货所使用的接口并不是标准的SATA接口，
而是一种缩短了电源部分，只留下5V的东西。

苦于缺少各种条件， 刻录MacOS9 光盘的操作等了好多天才得以完成\
~~我不会说我跑去借了一台x86计算机来做~~

这个iso文件直接写进USB存储器里面是没有办法正常进行安装的\
去Google了一下， 似乎是MacOS9会检测安装介质是否可写

~~如果发现可写，就会立即终止启动进程~~

启动的时候按住Option键， 在CD光盘上面启动，如果顺利的话， 大概会是下图这个样子
![start.jpg](start.jpg)

经过一番加载和SuperDrive的轰鸣声(这东西真的好吵)之后，顺利进入到了MacOS9的LiveCD System
![home](home.jpg)

系统里面出现的文字几乎都是简体中文字符(zh-cn), 不过编码方式和现在流行的不一样？

直接执行MacOS安装就可以开始把MacOS9 安装进磁盘里面
![check](check.jpg)

一路选择下一步，就可以开始安装作业系统啦
![install](install.jpg)

大约十分钟后， 会提示作业系统安装好了， 可以进行重新启动了

这里进入的还是MacOS X 10.4， 选中刚刚装好的MacOS9分区之后,\
是可以成功开起Classic环境的
![classic](classic.jpg)

不过。。。这个怎么看起来像是在MacOS X 里面又运行了一个MacOS 9 的虚拟机呀。
