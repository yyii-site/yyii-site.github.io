---
title: Jetson安装USB转RS232串口驱动
date: 2024-07-03 22:53:38
tags: Jetson
---

USB转串口芯片使用 **pl2303**

## 1.环境

硬件:

* jetson nano
* USB转RS232

串口调试软件：

带ui类：cutecom、putty

命令行：minicom、screen

先用调试软件测试，系统自带的驱动有问题再进行下面的操作。

## 2.我遇到的问题

先用的usb转RS232，芯片试CH340，出现的问题是：波特率115200，windows中收到的数据正常。Jetson收到的数量差不多，但数据全部错误。收发如下：

![jetson_ch340_error](jetson_ch340_error.jpeg)

发送'1'接收0x11。随后再WCH沁恒官网下驱动，编译替换。调试助手测试正常，但接到实际设备上使用数据还是乱码，遂放弃使用。

换PL2303，windows需要等一会才能自动安装驱动。而jetson不能正常挂载到ttyUSBx。`dmesg`查看只能到usb 1-2.1: SerialNumber: CMCVe12CJ06，重新编译和替换驱动后，显示attached到ttyUSB0，通过putty查看设备数据，接收正常。

```bash
[ 1640.374193] usb 1-2.1: new full-speed USB device number 8 using tegra-xusb
[ 1640.398877] usb 1-2.1: New USB device found, idVendor=067b, idProduct=23a3
[ 1640.398894] usb 1-2.1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 1640.398905] usb 1-2.1: Product: USB-Serial Controller
[ 1640.398914] usb 1-2.1: Manufacturer: Prolific Technology Inc.
[ 1640.398923] usb 1-2.1: SerialNumber: CMCVe12CJ06
[ 3483.947952] usbserial: USB Serial deregistering driver pl2303
[ 3483.948103] usbcore: deregistering interface driver pl2303
[ 3500.329161] pl2303: loading out-of-tree module taints kernel.
[ 3500.335716] usbcore: registered new interface driver pl2303
[ 3500.335756] usbserial: USB Serial support registered for pl2303
[ 3500.335830] pl2303 1-2.1:1.0: pl2303 converter detected
[ 3500.336221] usb 1-2.1: pl2303 converter now attached to ttyUSB0
```

## 3.基础

`dmesg` 先插拔USB，再使用指令查看系统变化信息

`lsusb` 查看USB接口硬件列表

`ls /dev` 查看硬件设备列表

`sudo lsusb -v -d 067b:23a3` 查看某个具体设备的详细信息

`lsmod`  显示已载入系统的模块 

`sudo rmmod pl2303` 卸载模块

`insmod` 一次只能加载特定的一个设备驱动，且需要驱动的具体地址。写法为：

```bash
insmod drv.ko
```

`modprobe` 则可以一次将有依赖关系的驱动全部加载到内核。不加驱动的具体地址，但需要在安装文件系统时是按照make modues_install的方式安装驱动模块的。驱动被安装在/lib/modules/$(uname -r)/...下。写法为：

```bash
modprob drv
```

modprobe 和insmod一样都是用来加载内核module的不过modprobe比较智能，它可以根据module的依赖性来自动为你加载；而insmod就做不到这点



## 4.下载驱动

从 [UGREEN绿联](https://www.lulian.cn/download/tag/pl2303qudong) 网下载驱动，并解压。内容如下：

![pl2303_driver](pl2303_driver.png)

## 5.查看jetson内核版本

```bash
uname -a
```

可以看到 4.9.337

```bash
Linux booster 4.9.337-tegra #1 SMP PREEMPT Thu Jun 8 21:19:14 PDT 2023 aarch64 aarch64 aarch64 GNU/Linux
```

## 6.将驱动上传到Jetson

从PL2303G_Linux_Driver_v1.0.6中可以看到有两个4.9版的，选一个（失败了再试另一个）使用MobaXterm或其他软件上传到jetson。

## 7.编译安装

```bash
cd 4.9_ok
make
```

可以看到成功编译出pl2303.ko文件

接下来卸载旧的，再安装新编译的。

```bash
sudo rmmod pl2303
sudo insmod pl2303.ko
```

dmesg | tail 查看是否attach到ttyUSBx

`usb 1-2.1: pl2303 converter now attached to ttyUSB0`

## 8.测试

```bash
sudo putty
```

设置波特率，刚开始可以将RS232的Tx和Rx引脚短接，键盘按什么屏幕显示什么说明通信正常。

替换自带驱动

```bash
sudo cp pl2303.ko /lib/modules/4.9.337-tegra/kernel/drivers/usb/serial/
```



## 9.修改权限免ROOT

参考[github](https://github.com/Slamtec/rplidar_ros/blob/ros2/scripts/create_udev_rules.sh)

以rplidar.rules为模板，修改idVendor和idProduct的值，如下：

```bash
KERNEL=="ttyUSB*", ATTRS{idVendor}=="067b", ATTRS{idProduct}=="23a3", MODE:="0777", SYMLINK+="mygps"
```

启用规则

```bash
sudo service udev reload
sudo service udev restart
sudo udevadm control --reload && sudo udevadm trigger
```

查看规则是否应用成功

```bash
ll /dev
lrwxrwxrwx   1 root root           7 7月   2 14:06 mygps -> ttyUSB0
```

测试

```bash
putty
```

重启，如果不能正常挂载，需要使用下面的指令：

```bash
modprobe
```
