---
title: Ubuntu-RSP1-RTL_433
date: 2024-04-21 22:07:47
tags: SDR
---

RSP1 MSI2500 MSI001 SoapySDR RTL_433 Ubuntu

## 建议查看参考链接

[Link !!!](https://revspace.nl/Msi2500SDR)

### 3.1.1. Linux kernel driver

Not sure what the Linux kernel driver is actually any good for anyway. 

remove the modules manually

```sh
sudo modprobe -r msi2500 msi001
```

create a file /etc/modprobe.d/blacklist-msi.conf

```
blacklist msi001
blacklist msi2500
```

### 3.1.2. Soapy SDR

install the soapy 'miri' driver:

```sh
sudo apt install soapysdr-module-mirisdr
```

try to find it with SoapySDRUtil

```sh
SoapySDRUtil --find
```

result (for example): 

```sh
Found device 1
  driver = miri
  label = Mirics MSi2500 default (e.g. VTX3D card)
  miri = 0
```

try to open with SoapySDRUtil

```sh
 SoapySDRUtil  --probe="driver=miri"
 ```
 or
 ```sh
 SoapySDRUtil  --probe="driver=soapyMiri"
 ```

### 3.1.3. rtl_433

[link](https://github.com/merbanan/rtl_433)

install
```sh
sudo apt install rtl-433
```

run
```sh
rtl_433 -d ""
```