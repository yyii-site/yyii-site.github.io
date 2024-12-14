---
title: Ubuntu-RSP1-GNURadio
date: 2024-05-05 15:14:43
tags: SDR
---

MSI2500 MSI001 RSP1 SoapySDR libmirisdr gr-osmosdr GNURadio Ubuntu

## 1.概述

国内版克隆版RSP1在RTL-433和GNU Radio中的使用

### windows:

使用 [SDRuno](https://www.sdrplay.com/softwarehome/) 软件安装简单易上手.  (建议关闭或卸载流氓杀毒软件)

### Ubuntu: 

无法使用 SDRconnec. 

按照 [Rev Space](https://revspace.nl/Msi2500SDR) 教程，可将其识别为miri，且能正常采样。

使用方式一： [rtl_433](https://github.com/merbanan/rtl_433)

使用方式二： [GNU Radio](https://www.gnuradio.org/)

两种方式下面均有使用教程。

对于ubuntu下整个系统框架我是这样理解的：

![Ubuntu_RSP1](Ubuntu_RSP1.png)


## 2.Hardware 硬件

BG7YZF 黄淮创客空间

a chinese clone of the SDRPlay RSP1.

![taobao](taobao.jpg)

![screenshot_rsp1](Screenshot_rsp1.jpg)

MS001+MSI2500

10k~2GHz ADC:12bit Bandwidth:10MHz TCXO:0.5ppm

![msi001_msi2500](msi001_msi2500.jpg)

## 3.API 驱动

SoapySDR 是另一个开源的SDR硬件抽象层，旨在提供一种通用的API和中间件，用于与不同的SDR硬件设备进行交互。它提供了一个统一的接口，使应用程序能够以一致的方式访问和控制各种SDR设备。SoapySDR 支持多种平台和操作系统，并提供了C++、Python和其他编程语言的绑定。

### 禁用和卸载内核驱动

remove the modules manually

```sh
sudo modprobe -r msi2500 msi001
```

create a file /etc/modprobe.d/blacklist-msi.conf

```
blacklist msi001
blacklist msi2500
```

### 安装SoapySDR

```sh
sudo apt install soapysdr-module-mirisdr
```

### 查找设备

将设备连接到USB接口

```sh
 SoapySDRUtil --find
 ```

回显 result(for example)
 
```sh
######################################################
##     Soapy SDR -- the SDR abstraction library     ##
######################################################

[INFO] [UHD] linux; GNU C++ version 13.1.0; Boost_107400; UHD_4.4.0.0+ds1-4
Found device 0
  default_input = True
  default_output = True
  device_id = 0
  driver = audio
  label = Built-in Audio

Found device 1
  driver = miri
  label = Mirics MSi2500 default (e.g. VTX3D card)
  miri = 0
 ```

Print detailed information  打开并打印测试信息

```sh
SoapySDRUtil  --probe="driver=miri"
```
回显 result(for example)

```sh
######################################################
##     Soapy SDR -- the SDR abstraction library     ##
######################################################

Probe device driver=miri
Using device #0: Mirics MSi2500 default (e.g. VTX3D card)

----------------------------------------------------
-- Device identification
----------------------------------------------------
  driver=miri
  hardware=miri

----------------------------------------------------
-- Peripheral summary
----------------------------------------------------
  Channels: 1 Rx, 0 Tx
  Timestamps: NO

----------------------------------------------------
-- RX Channel 0
----------------------------------------------------
  Full-duplex: NO
  Supports AGC: NO
  Stream formats: CF32
  Native format: CF32 [full-scale=1]
  Antennas: RX
  Full gain range: [-1, 49, 1] dB
    LNA gain range: [-1, 49, 1] dB
  Full freq range: [0.15, 30], [64, 108], [162, 240], [470, 960], [1450, 1675] MHz
    RF freq range: [0.15, 30], [64, 108], [162, 240], [470, 960], [1450, 1675] MHz
    CORR freq range:  MHz
  Tune args:
     * LO Offset - Tune the LO with an offset and compensate with the baseband CORDIC.
       [key=OFFSET, units=Hz, default=0.0, type=float]
     * CORR - Specify a specific value for this component or IGNORE to skip tuning it.
       [key=CORR, units=Hz, default=DEFAULT, type=float, options=(DEFAULT, IGNORE)]
  Sample rates: 8 MSps

_mirisdr_alloc_async_buffers
Lost samples!
82 01 00 00 00 10 00 00 fb fd bc b4 0e f6 fd fe 
Lost samples!
Lost samples!
OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
```

## 4.rtl_433

rtl_433 是一个开源的软件项目，用于接收和解码使用软件定义无线电（SDR）接收器（如RTL-SDR）捕获的无线电信号。

rtl_433 项目的目标是支持解码多种不同的无线电设备和协议，包括气象传感器、汽车遥控器、无线温度传感器等。它提供了一个命令行工具，可以直接与RTL-SDR设备进行交互，并将捕获的无线电信号解码为可读的文本输出。

### 安装

```sh
sudo apt install rtl-433
```

### 查看设备信息

```sh
rtl_433 -d driver=miri -v
```

### 解码测试

```sh
rtl_433 -d "miri"
```

回显 result(for example)

```sh
rtl_433 -d "miri"
rtl_433 version 22.11 (2022-11-19) inputs file rtl_tcp RTL-SDR SoapySDR
Use -h for usage help and see https://triq.org/ for documentation.
Trying conf file at "rtl_433.conf"...
Trying conf file at "/home/yourname/.config/rtl_433/rtl_433.conf"...
Trying conf file at "/usr/local/etc/rtl_433/rtl_433.conf"...
Trying conf file at "/etc/rtl_433/rtl_433.conf"...
Registered 191 out of 223 device decoding protocols [ 1-4 8 11-12 15-17 19-23 25-26 29-36 38-60 63 67-71 73-100 102-105 108-116 119 121 124-128 130-149 151-161 163-168 170-175 177-197 199 201-215 217-223 ]
[INFO] [UHD] linux; GNU C++ version 13.1.0; Boost_107400; UHD_4.4.0.0+ds1-4
[INFO] Using format CS16.
Sample rate set to 250000 S/s.
Tuner set to automatic gain.
Tuner gain set to Auto.
Tuned to 433.920MHz.
RtAudio pulse: _NOT_ running realtime scheduling
baseband_demod_FM_cs16: low pass filter for 250000 Hz at cutoff 25000 Hz, 40.0 us
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
time      : 2024-05-05 14:15:39
model     : Acurite-986  id        : 4096
channel   : 1R           Battery   : 1             temperature: 1.000000 F   status    : 4             Integrity : CRC
```

##  5.libmirisdr-5

GNU Radio未能通过SoapySDR打开设备，故使用[libmirisdr-5](https://github.com/ericek111/libmirisdr-5)(forked from f4exb/libmirisdr-4)

libmirisdr 提供了与 Mirics SDR 接收器进行交互的底层驱动程序和API。它允许开发人员通过编程语言（如C/C++）访问和控制 Mirics SDR 接收器的功能。

```sh
git clone git@github.com:ericek111/libmirisdr-5.git
```

build and install

```sh
cd libmirisdr-5
mkdir build
cd build
cmake ..
make -j4
sudo make install
sudo ldconfig
```

## 6.gr-osmosdr

gr-osmosdr 是一个与 GNU Radio 软件无线电平台相关的开源项目，用于与各种软件定义无线电（SDR）硬件设备进行交互。

使用 gr-osmosdr，你可以将不同厂商的SDR硬件设备（如RTL-SDR、HackRF、LimeSDR等）与GNU Radio进行集成。它提供了一组模块和函数，用于配置和控制SDR硬件设备的参数，如频率、增益、采样率等。通过这些功能，你可以将SDR硬件设备接收到的无线电信号导入到GNU Radio的流程图中进行进一步的信号处理和分析。

[gr-osmosdr](https://osmocom.org/projects/gr-osmosdr/wiki/GrOsmoSDR)

```sh
git clone https://git.osmocom.org/sdr/gr-osmosdr.git
```

build and install

```sh
cd gr-osmosdr
mkdir build
cd build
cmake ..
make -j4
sudo make install
sudo ldconfig
```


## 7.GNU Radio

GNU Radio 是一个流程图驱动的开源软件工具包，用于构建软件无线电系统和信号处理应用程序。gr-osmosdr 是 GNU Radio 的一个模块，它提供了与各种SDR硬件设备进行通信的功能。

### 安装

```sh
sudo apt install gnuradio
```

open gnuradio

### 流图1

功能：从sdr读取数据，并现实到界面。如下：

![gnu_project1_grc](gnu_project1_grc.png)

Execute the flow graph

![gnu_project1_run](gnu_project1_run.png)

### 流图2 FM

[YouTube](https://youtu.be/tj_9p_rXULM?si=qtQ6HvAgtN4TNqyG)

功能：从sdr读取数据，使用FM解码，最后从声卡播放。如下：

![gnu_project2_grc](gnu_project2_grc.png)

Execute the flow graph

![gnu_project2_run](gnu_project2_run.png)