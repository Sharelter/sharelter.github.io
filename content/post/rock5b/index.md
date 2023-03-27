---
title: "Rock5b试玩"
slug: rock5b
description: Rock and Roll!
date: 2023-03-14T15:45:24+08:00
image: rock5b.jpg
math: false
comments: true
categories: 
    - Tech
tags: 
    - toss
---

本文记录了 ROCK5B (RK3588) 的开箱记录，以及遇到的问题及解决方法

### 烧录系统到SD卡并启动

按照 <https://wiki.radxa.com/Rock5/5b/getting_started> 下载并烧录系统，这里先测试下 ubuntu(server 版本）（不建议一开始使用这个系统，bug多，先用debian或者 armbian（推荐） 或者 安卓， 安卓是最好上手的）。

这里系统盘制作好了会发现磁盘最后面有一段空闲分区，不用手动去扩展，ubuntu 系统第一次启动会自动扩容根目录到卡的最大空间。

可以使用串口模块查看日志，串口连接（波特率是 1500000 ，可能有些串口模块不支持这么高的波特率）

![seriel_1.jpg](seriel_1.jpg)

但是日志是不完全的，因为默认系统都关闭了日志打印，也不能看到什么消息

Ubuntu 第一次开机 会一直循环

```
    [    9.952843] BUG: spinlock bad magic on CPU#1, systemd-udevd/433
    [    9.957383]  lock: 0xffffffc012b26080, .magic: 00000000, .owner: <none>/-1, .owner_cpu: 0
    [   12.590411] reboot: Restarting system
    DDR Version V1.08 20220617
    LPDDR4X, 2112MHz
    channel[0] BW=16 Col=10 Bk=8 CS0 Row=17 CS1 Row=17 CS=2 Die BW=8 Size=4096MB
    channel[1] BW=16 Col=10 Bk=8 CS0 Row=17 CS1 Row=17 CS=2 Die BW=8 Size=4096MB
    channel[2] BW=16 Col=10 Bk=8 CS0 Row=17 CS1 Row=17 CS=2 Die BW=8 Size=4096MB
    channel[3] BW=16 Col=10 Bk=8 CS0 Row=17 CS1 Row=17 CS=2 Die BW=8 Size=4096MB
    Manufacturer ID:0x6 
    CH0 RX Vref:28.7%, TX Vref:24.8%,25.8%
    CH1 RX Vref:28.7%, TX Vref:24.8%,23.8%
    CH2 RX Vref:27.7%, TX Vref:24.8%,23.8%
    CH3 RX Vref:29.7%, TX Vref:24.8%,24.8%
    change to F1: 528MHz
    change to F2: 1068MHz
    change to F3: 1560MHz
    change to F0: 2112MHz
    out
    INFO:    Preloader serial: 2
    NOTICE:  BL31: v2.3():v2.3-405-gb52c2eadd:derrick.huang
    NOTICE:  BL31: Built : 11:23:47, Aug 15 2022
    INFO:    spec: 0x1
    INFO:    ext 32k is not valid
    INFO:    GICv3 without legacy support detected.
    INFO:    ARM GICv3 driver initialized in EL3
    INFO:    system boots from cpu-hwid-0
    INFO:    idle_st=0x21fff, pd_st=0x11fff9, repair_st=0xfff70001
    INFO:    dfs DDR fsp_params[0].freq_mhz= 2112MHz
    INFO:    dfs DDR fsp_params[1].freq_mhz= 528MHz
    INFO:    dfs DDR fsp_params[2].freq_mhz= 1068MHz
    INFO:    dfs DDR fsp_params[3].freq_mhz= 1560MHz
    INFO:    BL31: Initialising Exception Handling Framework
    INFO:    BL31: Initializing runtime services
    WARNING: No OPTEE provided by BL2 boot loader, Booting device without OPTEE initialization. SMC`s destined for OPTEE will return SMC_UNK
    ERROR:   Error initializing runtime service opteed_fast
    INFO:    BL31: Preparing for EL3 exit to normal world
    INFO:    Entry point address = 0x200000
    INFO:    SPSR = 0x3c9
    [   10.314850] BUG: spinlock bad magic on CPU#5, systemd-udevd/394
    [   10.316119]  lock: 0xffffffc012b05080, .magic: 00000000, .owner: <none>/-1, .owner_cpu: 0
```

用了诱骗器让充电器输出固定的12V,就能跑起来

```
    [   10.305132] BUG: spinlock bad magic on CPU#7, systemd-udevd/379
    [   10.305139]  lock: 0xffffffc012a25080, .magic: 00000000, .owner: <none>/-1, .owner_cpu: 0
```

HDMI 插上也没有反应，串口终端

```
    [  723.967060] rockchip-hdptx-phy-hdmi fed70000.hdmiphy: hdptx phy pll locked!
    [  723.967381] rockchip-hdptx-phy-hdmi fed70000.hdmiphy: hdptx phy lane locked!
    [  726.353844] Buffer I/O error on device mmcblk0p2, logical block 12289
    [  726.354406] Buffer I/O error on device mmcblk0p2, logical block 141311
    [  726.354819] Buffer I/O error on device mmcblk0p2, logical block 141312
    [  726.355469] Buffer I/O error on device mmcblk0p2, logical block 141318
    [  726.355885] Buffer I/O error on device mmcblk0p2, logical block 141319
    [  726.356330] Buffer I/O error on device mmcblk0p2, logical block 141323
    [  726.356738] Buffer I/O error on device mmcblk0p2, logical block 141324
    [  726.357180] Buffer I/O error on device mmcblk0p2, logical block 141326
    [  726.357586] Buffer I/O error on device mmcblk0p2, logical block 141327
    [  726.357992] Buffer I/O error on device mmcblk0p2, logical block 141328
    [  726.363448] Aborting journal on device mmcblk0p2-8.
    [  726.363592] JBD2: Error -5 detected when updating journal superblock for mmcblk0p2-8.
    [  726.364213] EXT4-fs (mmcblk0p2): I/O error while writing superblock
    [  726.364331] EXT4-fs error (device mmcblk0p2): ext4_journal_check_start:83: Detected aborted journal
    [  726.364484] EXT4-fs (mmcblk0p2): Remounting filesystem read-only
    [  726.364594] EXT4-fs (mmcblk0p2): ext4_writepages: jbd2_start: 9223372036854775779 pages, ino 11320; err -30
    [  726.418700] JBD2: Error while async write back metadata bh 814.
    [  726.428108] EXT4-fs error (device mmcblk0p2): __ext4_find_entry:1611: inode #4508: comm systemd-udevd: reading directory lblock 0
    [  726.428414] EXT4-fs (mmcblk0p2): I/O error while writing superblock
    [  726.428590] EXT4-fs error (device mmcblk0p2): __ext4_find_entry:1611: inode #548: comm systemd-udevd: reading directory lblock 0
    [  726.428767] EXT4-fs (mmcblk0p2): I/O error while writing superblock
    [  726.428883] EXT4-fs error (device mmcblk0p2): __ext4_find_entry:1611: inode #2119: comm systemd-udevd: reading directory lblock 0
    [  726.429062] EXT4-fs error (device mmcblk0p2): __ext4_find_entry:1611: inode #2: comm systemd-udevd: reading directory lblock 0
    [  726.429233] EXT4-fs error (device mmcblk0p2): __ext4_find_entry:1611: inode #4508: comm systemd-udevd: reading directory lblock 0
```

这读取文件系统错误，有可能TF卡松动了，裸板的设计，没有外壳的时候TF卡确实很容易松动， 断电插紧重启即可

运行一会儿后又出现了问题。。。

```
    [ 1147.255838] rk806 spi2.0: SPI transfer timed out
    [ 1147.263313] spi_master spi2: failed to transfer one message from queue
    [ 1147.270938] cpu cpu0: rockchip_cpufreq_set_volt: failed to set voltage (925000 925000 950000 uV): -11
    [ 1147.278907] cpufreq: __target_index: Failed to change cpu frequency: -110
    [ 1147.489206] rk806 spi2.0: SPI transfer timed out
```

rk806 貌似是电源管理 IC，看样子有可能是充电协议握手出现了啥问题，先不管，断电重启，之后再说。

更换国内源，比如中科大 `mirrors.ustc.edu.cn`，编辑 `/etc/apt/sources.list`

```
    deb http://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
    #deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
    deb http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
    #deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
    deb http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
    #deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
    deb http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
    #deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
```

`apt update`发现报错, 又报错了

按照通用做法

```
sudo gpg --keyserver keyserver.ubuntu.com --recv-keys 9B98116C9AA302C7
```

报错 gpg: keyserver receive failed: No data，于是去apt.radxa.com源看看

```
    wget -O - apt.radxa.com/focal-stable/public.key | sudo apt-key add -
```

### 电源

PMU 是通过 I2C 连接到芯片的，可以在系统看到当前协商的电压：

```
    sensors tcpm_source_psy_4_0022-i2c-4-22
```

以及可以看到 typec 的相关信息

```
    grep "" /sys/class/typec/port0/* 2>/dev/null
```

实际电压可以(需要除以~172.5)

```
    awk '{printf ("%0.2f\n",$1/172.5); }' </sys/devices/iio_sysfs_trigger/subsystem/devices/iio\:device0/in_voltage6_raw
    11.98
```

### CPU

```
    cat /proc/cpuinfo
    sudo apt-get install cpufrequtils
    sudo cpufreq-info
```

可以看到最大频率为 2.4GHz， 以及 4个小核 1.8GHz
用 7z 在没有风扇的情况下先跑个分看看情况，跑起来手摸cpu盖子还是很烫的

```
    $ apt install p7zip
    $ 7zr b
    7-Zip (a) [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
    p7zip Version 16.02 (locale=C.UTF-8,Utf16=on,HugeFiles=on,64 bits,8 CPUs LE)
    LE
    CPU Freq: 64000000 - - - - - - - -
    RAM size:   15722 MB,  # CPU hardware threads:   8
    RAM usage:   1765 MB,  # Benchmark threads:      8
                           Compressing  |                  Decompressing
    Dict     Speed Usage    R/U Rating  |      Speed Usage    R/U Rating
             KiB/s     %   MIPS   MIPS  |      KiB/s     %   MIPS   MIPS
    22:      13939   726   1867  13560  |     199330   682   2494  17002
    23:      13416   748   1828  13670  |     193846   684   2452  16775
    24:      12832   759   1818  13797  |     186688   681   2406  16385
    25:      12244   758   1846  13980  |     179663   680   2350  15989
    ----------------------------------  | ------------------------------
    Avr:             748   1840  13752  |              682   2426  16538
    Tot:             715   2133  15145
```

可以看到这里的 speed 和 rating 是树莓派4B 的至少 3 倍了

尝试看看温度，用 ssh 登录试试，串口执行

```
ifconfig
```

看到ip，然后`ssh root@ip`输入密码`root`，报错`Permission denied, please try again.`，在串口终端修改密码`passwd root`为`root`，还是不行，可能默认设置不允许 root 用户登录，修改`vim /etc/ssh/sshd_config` 添加`PermitRootLogin yes `后 重启服务`systemctl restart sshd`即可

```
    watch -n 0.1 cat /sys/class/thermal/thermal_zone0/temp
```

同时跑分，温度最高跑到了72℃，还是很凉快的。
同时，看一下频率是否跑满了

```
watch -n 0.1 cpufreq-info -f -c 4
```

> 这里 -c 指令核心 id， 可以看到跑分时频率为 2400000Hz 或者 1800000Hz

到此为止，你以为真的跑到 2.4GHz 了吗，99.99%的可能并没有，只是显示 2.4GHz，最开始这个开发板甚至内置了一个测试软件来抽奖看自己买到的 CPU 处于什么水平。。。
RK在rk35系列使用了` PVTM (Process-Voltage-Temperature Monitor) （PVT）`来监控系统状态，有一个 MCU 专门来控制频率CPU 频率，以及 NPU GPU

Link: [What is PVTM? Or why your Rockchip RK3588 CPU may not reach 2.4 GHz](https://www.cnx-software.com/2022/07/17/what-is-pvtm-or-why-your-rockchip-rk3588-cpu-may-not-reach-2-4-ghz/)

Link: [cpufreq-rockchip.txt](cpufreq-rockchip.txt)

```
    rock@rock-5b:~$ dmesg |grep pvtm
    [    8.110543] rockchip-pvtm fda40000.pvtm: pvtm@0 probed
    [    8.110598] rockchip-pvtm fda50000.pvtm: pvtm@1 probed
    [    8.110647] rockchip-pvtm fda60000.pvtm: pvtm@2 probed
    [    8.110693] rockchip-pvtm fdaf0000.pvtm: pvtm@3 probed
    [    8.110736] rockchip-pvtm fdb30000.pvtm: pvtm@4 probed
    [    9.051869] cpu cpu0: pvtm=1456
    [    9.051948] cpu cpu0: pvtm-volt-sel=2
    [    9.060591] cpu cpu4: pvtm=1705
    [    9.064527] cpu cpu4: pvtm-volt-sel=4
    [    9.073916] cpu cpu6: pvtm=1724
    [    9.077872] cpu cpu6: pvtm-volt-sel=5
    [    9.162917] mali fb000000.gpu: pvtm=877
    [    9.162950] mali fb000000.gpu: pvtm-volt-sel=3
    [    9.346530] RKNPU fdab0000.npu: pvtm=880
    [    9.351791] RKNPU fdab0000.npu: pvtm-volt-sel=3
```

看不太懂，不知道每个数值对应了什么级别的频率

上面的信息是由内核报告的 `Operating Performance Point (OPP)`, 但是实际跑的频率通过 `ThomasKaiser/SBC-Bench.sh` 可以获得（内置的armbian-config命令中集成了此工具的，可以直接用），比如我的

```
    Checking cpufreq OPP for cpu0-cpu3 (Cortex-A55):
    Cpufreq OPP: 1800    Measured: 1798 (1798.646/1798.490/1798.372)
    Cpufreq OPP: 1608    Measured: 1614 (1614.427/1614.033/1613.875)
    Cpufreq OPP: 1416    Measured: 1414 (1415.323/1414.869/1414.627)
    Cpufreq OPP: 1200    Measured: 1233 (1233.787/1233.729/1233.585)     (+2.8%)
    Cpufreq OPP: 1008    Measured: 1007 (1007.980/1007.956/1007.524)
    Cpufreq OPP:  816    Measured:  806    (806.096/806.077/806.039)     (-1.2%)
    Cpufreq OPP:  600    Measured:  589    (589.884/589.422/589.281)     (-1.8%)
    Cpufreq OPP:  408    Measured:  391    (391.542/391.489/391.356)     (-4.2%)
    Checking cpufreq OPP for cpu4-cpu5 (Cortex-A76):
    Cpufreq OPP: 2400    Measured: 2227 (2227.990/2227.846/2227.750)     (-7.2%)
    Cpufreq OPP: 2208    Measured: 2161 (2162.236/2161.558/2161.286)     (-2.1%)
    Cpufreq OPP: 2016    Measured: 1998 (1998.559/1998.559/1998.414)
    Cpufreq OPP: 1800    Measured: 1826 (1826.351/1826.351/1826.271)     (+1.4%)
    Cpufreq OPP: 1608    Measured: 1623 (1623.824/1623.824/1623.744)
    Cpufreq OPP: 1416    Measured: 1442 (1442.152/1442.089/1441.869)     (+1.8%)
    Cpufreq OPP: 1200    Measured: 1197 (1197.094/1197.094/1196.904)
    Cpufreq OPP: 1008    Measured: 1002 (1002.915/1002.132/1002.060)
    Cpufreq OPP:  816    Measured:  807    (808.040/806.980/806.826)     (-1.1%)
    Cpufreq OPP:  600    Measured:  592    (592.867/592.828/592.802)     (-1.3%)
    Cpufreq OPP:  408    Measured:  394    (394.914/394.905/394.842)     (-3.4%)
    Checking cpufreq OPP for cpu6-cpu7 (Cortex-A76):
    Cpufreq OPP: 2400    Measured: 2243 (2244.059/2243.815/2243.718)     (-6.5%)
    Cpufreq OPP: 2208    Measured: 2160 (2160.970/2160.925/2160.744)     (-2.2%)
    Cpufreq OPP: 2016    Measured: 1990 (1990.232/1990.040/1990.040)     (-1.3%)
    Cpufreq OPP: 1800    Measured: 1812 (1812.294/1812.174/1812.055)
    Cpufreq OPP: 1608    Measured: 1604 (1604.397/1604.319/1604.203)
    Cpufreq OPP: 1416    Measured: 1421 (1421.929/1421.837/1421.837)
    Cpufreq OPP: 1200    Measured: 1214 (1214.996/1214.968/1214.912)     (+1.2%)
    Cpufreq OPP: 1008    Measured: 1019 (1019.959/1019.934/1019.910)     (+1.1%)
    Cpufreq OPP:  816    Measured:  818    (818.552/818.433/818.394)
    Cpufreq OPP:  600    Measured:  592    (592.906/592.893/592.854)     (-1.3%)
    Cpufreq OPP:  408    Measured:  394    (394.950/394.905/394.887)     (-3.4%)
```

可以看到，实际只有2.2GHz，和2.4GHz足足差了200MHz, 6.5%，确实很糟糕了。。。

### 内存

查看频率：
```
    cat /sys/class/devfreq/dmc/cur_freq
    cat /sys/kernel/debug/clk/clk_summary | grep ddr
```

> 官方说明： The ram for ROCK 5B is LPDDR 4x, two 32bits LPDDR 4x chips make 64bits, data frequency is up to 4224Mhz. ROCK 5B offers 4GB, 8GB and 16GB ram size options.

可以看到实际频率 `1068000000， 1GHz`， 查看可用频率

```
    cat /sys/class/devfreq/dmc/available_frequencies
    528000000 1068000000 1560000000 2112000000
    # 设置 DDR 频率，例如，设置 1560MHz
    echo userspace > /sys/class/devfreq/dmc/governor
    echo 1560000000 > /sys/class/devfreq/dmc/userspace/set_freq
```

### HDMI输出

到此为止，HDMI 仍然没有输出，我的显示器是 2k(2560x1440)

```
    cat /sys/class/drm/card0-HDMI-A-1/modes
```

发现没有 2k 的分辨率

换个 1080p 的显示器接上 HDMI，立马亮了，终端界面tty1出现了，没有桌面。

另外，在使用 debian 系统后，可以直接输出 1080p和4k输出，但均没有 2k 输出可以直接使用， 不过有个最简单的方法就是设置为 4k60fps，然后设置缩放200%即可，因为性能足够， 4k 也不卡。

### Armbian
Armbian 适配得更好一点，推荐使用
因为 GPU 驱动大多需要在 wayland 模式下使用，可以设置默认登录到 wayland 窗口管理模式，这样就不用每次登录时选择一次了

```
    sudo vim /etc/gdm3/custom.conf
```

修改`WaylandEnable=true`

另外，官网提供的链接不是最新的，可以到github.com/armbian/build/releases 找最新的。
另外， github.com/amazingfate/armbian-rock5b-images 据说是内置了 GPU 相关的驱动，在官方还没完全支持 GPU 前可以尝试这个，具体看后面 GPU 使用部分。

### 从NVME启动

SD 卡会比较慢，据说太慢还会影响到快充握手，因为快充握手协议是在系统驱动里面的，所以最好用 eMMc 或者 nvme ssd 或者 USB 启动读写更快。

启动流程: 内部有个叫 maskrom 的固件，会优先启动 SPI nor Flash -> sd -> eMMC

默认不支持从 nvme 启动，所以需要额外的引导，将引导保存到 SPI nor Flash中，然后这个固件再引导nvme的系统启动。

于是需要先更新 Nor Flash 的固件，使用 rk 官方的 rkdevtool，注意不能直接就给 nor flash 下载固件，还需要先给 maskrom 传一个用来和 rkdevtool 通讯以及写 flash 等操作的固件，这里名字叫 `rk*_spl_loader.bin`，这个固件只会临时运行在内存中。

装 USB 驱动，然后打开 rkdevtool， 按住 maskroom 按钮，上电，软件会检测到设备。
`Write by address` 默认没勾上，注意勾上

![rkdevtool](rkdevtool.jpg)

然后下载系统，可以用官方的 debian 或者 armbian

armbian 页面有些提示比如带硬件加速的 kodi 使用

> PD is broken on most revisions that are in the wild and is causing boot loop. Workaround is to use stupid / fixed at 5 volts USB-C power supply.
> NVME boot can work with eMMC/SD boot or SPI. Check those instructions to install / update SPI boot loader.
> In order to enable 3D acceleration:
```


    sudo add-apt-repository ppa:liujianfeng1994/panfork-mesa
    sudo add-apt-repository ppa:liujianfeng1994/rockchip-multimedia
    sudo apt update
    sudo apt dist-upgrade
    sudo apt install kodi
```

没买 m.2 转 USB转接器，所以直接把 nvme ssd 装到板子上，然后用 SD 卡进 debian 系统

```
    rock@rock-5b:~$ lsblk
    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    mtdblock0    31:0    0    16M  0 disk
    mmcblk1     179:0    0  29.2G  0 disk
    ├─mmcblk1p1 179:1    0   512M  0 part /boot
    └─mmcblk1p2 179:2    0  28.6G  0 part /
    nvme0n1     259:0    0 931.5G  0 disk
```

可以看到 直接能读到 nvme0n1，直接操作这个固态即可，因为是初次测试，硬盘也是空的，直接

```
    sudo xzcat  Armbian_22.11.2_Rock-5b_jammy_legacy_5.10.110_gnome_desktop.img.xz | sudo dd of=/dev/nvme0n1 bs=1M status=progress
```

写入完成后

```
    nvme0n1     259:0    0 931.5G  0 disk
    ├─nvme0n1p1 259:1    0   256M  0 part
    └─nvme0n1p2 259:2    0   6.8G  0 part
```

然后断电，拔掉 SD 卡上电即可从 nvme ssd 中的系统启动

进系统后看看接口情况

```
    sudo dmidecode | grep --color "PCI"
```

speed test:
iozone, fio

```
    iozone -e -I -a -s 100M -r 4k -r 16k -r 512k -r 1024k -r 16384k -i 0 -i 1 -i 2
                                                                 random    random     bkwd    record    stride                                    
                  kB  reclen    write  rewrite    read    reread    read     write     read   rewrite      read   fwrite frewrite    fread  freread
              102400       4   131760   167822   191255   204110    49352   108890                                                                
              102400      16   400881   478178   340041   289103   164730   277126                                                                
              102400     512  1444041  1676845  1619100  1573208  1314692  1723524                                                                
              102400    1024  1797630  1854966  1684764  1714470  1559952  1803609                                                                
              102400   16384  2478047  2356570  2140251  2189357  2261106  2410056
```

可以看到 我的 SSD 是三星的 970 evo plus, 是支持 nvme3x4 的，1M 读写 分别是 1.68GB/s 1.79GB/s

虽然都是 PCIE3.0x4 接口，可以看到顺序读写都没有达到在电脑上 3.5GB/s 和 3.3GB/s 的水平，在板子上只能有 2.1GB/s 和 2.5GB/s，读写速度甚至没有写入速度高，随机 4k 读写也只能到 50MB/s 和 108MB/s，远远低于上面测评中的 >= 800MB/s，可见，开发板（armbian22）远远没有跑满 PCIE3.0x4 的速度，暂时不知是硬件还是软件的问题，以后发现了补上

另外，需要考虑散热问题，固态的发热量挺大的！可以通过`/dev/nvme0n1`看到固态温度

```
    sudo apt install nvme-cli
    sudo nvme smart-log /dev/nvme0 |grep -i "^Temperature"
```

### GPU 硬件解码

装好 armbian 发现自带的 mpv 播放器只能软解，跑个 4k 直接把 CPU 吃得差不多一卡一卡。

按照 armbian 下载页面的源装了 kodi，不能直接用只能放声音没有画面，是因为无法解码导致的。

参考这个帖子的步骤即可（可以直接重装系统，amazingfate 打包了带这些软件包的系统，也可以自己手动更新包，自己更新会麻烦一点）：https://forum.radxa.com/t/rk3588-kodi-rkmpp-accelerated-decoding-working-out-of-box/12785/2 和 https://forum.armbian.com/topic/24802-kodi-for-rk35xx-510-legacy-kernel/

> 以及 GPU 相关也可以参考玩 我的世界 的帖子： https://forum.radxa.com/t/guide-run-minecraft-at-full-speed-on-rock-5b/11937/8

```
    You have to create the following udev rules to enable mpp and rga hardware acceleration:
    KERNEL=="mpp_service", MODE="0660", GROUP="video"
    KERNEL=="rga", MODE="0660", GROUP="video"
    KERNEL=="system-dma32", MODE="0666", GROUP="video"
    KERNEL=="system-uncached-dma32", MODE="0666", GROUP="video" RUN+="/usr/bin/chmod a+rw /dev/dma_heap"
    to /etc/udev/rules.d/11-rockchip-multimedia.rules
    To enable libv4l-rkmpp for chromium, you have to install libv4l-rkmpp, v4l-utils and chromium-browser in this ppa:
    sudo apt install chromium-browser=$(apt-cache show chromium-browser|grep Version|grep rkmpp|cut -d " " -f2) chromium-codecs-ffmpeg-extra=$(apt-cache show chromium-browser|grep Version|grep rkmpp|cut -d " " -f2) libv4l-rkmpp, v4l-utils
    Then run the following commands:
    sudo ln -s /lib /usr/lib64
    cd /usr/lib64/ && sudo ln -s aarch64-linux-gnu/libv4l2.so.0.0.0 libv4l2.so
    And add the following lines to /etc/rc.local:
    echo dec > /dev/video-dec0
    chown root:video /dev/video-dec0
    chmod 0660 /dev/video-dec0
    echo enc > /dev/video-enc0
    chown root:video /dev/video-enc0
    chmod 0660 /dev/video-enc0
    Add chrome flags "--use-gl=egl" to /etc/chromium-browser/default:
    CHROMIUM_FLAGS="--use-gl=egl"
    ==================================================
    chromium related FYI: https://github.com/JeffyCN/libv4l-rkmpp
```

```
    sudo add-apt-repository ppa:liujianfeng1994/panfork-mesa
    sudo apt update
    wget https://github.com/JeffyCN/rockchip_mirrors/raw/libmali/firmware/g610/mali_csffw.bin
    sudo mv mali_csffw.bin /lib/firmware
```
