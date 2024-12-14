---
title: jetson刷机救砖
date: 2024-07-03 22:11:07
tags: Jetson
---

## 1.烧录EMMC引导

如果开机不显示Nvidia的绿色图片，说明引导固件损坏。虽然sd卡或U盘中有系统，但硬件并不知道要去何处加载。现象为开机指示绿灯亮，但HDMI无显示，USB接口无供电（鼠标不亮）。通过TTL串口可以看到机器不停的重启。

[亚博教程](https://www.yahboom.com/public/upload/upload-html/1710831903/JetsonNano%E7%83%A7%E5%BD%95%E7%B3%BB%E7%BB%9F.html) 中的刷机的指令我在jetson nano上尝试失败。

[NVIDIA SDK Manager](https://docs.nvidia.com/sdk-manager/system-requirements/index.html) 打开链接查看刷写jetson需要的宿主机系统要求，如 Jetson nano 需要 Ubuntu 1804.

下载 [SDK Manager](http://docs.nvidia.com/sdk-manager/download-run-sdkm/index.html) 此软件可下载需要刷机的工具和软件包。

建议使用物理机，虚拟机Oracle VM VirtualBox出现将USB分配给虚拟机，但通过界面写入的时候会出现USB短线的问题。

也可以在界面刷写失败后尝试使用命令行的方式，需要在SDK Manager下载软件包中执行，可参考上面的亚博教程。

https://forums.developer.nvidia.com/t/recover-bootloader-in-nano-4g-b01/277975/4

```bash
sudo ./flash.sh jetson-nano-qspi mmcblk1p1
```

将跳线帽连接到FC REC和GND引脚， 接入电源，最后和插入microUSB数据线。

上电开机后会自动进入REC刷机模式。 

使用Ubuntu刷机


## 2.烧录系统镜像

下载[img系统镜像](https://developer.nvidia.com/embedded/downloads#?tx=$product,jetson_nano) ，选择链接中的 SD Card Image。

烧录过程参考上一小节的亚博教程



## 3.SD卡扩容

```bash
sudo apt install gparted
```

在图形界面环境下操作。



## 4.SSH远程登录

[参考链接](https://linuxize.com/post/how-to-enable-ssh-on-ubuntu-18-04)



## 5.VNC远程桌面

[亚博智能](https://www.yahboom.com/build.html?id=6189&cid=586)



## 6.Jtop的安装和使用

[参考](https://jetsonhacks.com/2023/02/07/jtop-the-ultimate-tool-for-monitoring-nvidia-jetson-devices)


