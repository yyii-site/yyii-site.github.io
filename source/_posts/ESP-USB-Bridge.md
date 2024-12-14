---
title: ESP-USB-Bridge
date: 2022-05-02 15:55:32
tags: ESP32
---

[https://github.com/espressif/esp-usb-bridge](https://github.com/espressif/esp-usb-bridge)

## 1.Clone

``` sh
git clone --depth 1 https://github.com/espressif/esp-usb-bridge.git
```

## 2.VSCode

<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> `ESP-IDF: Open ESP-IDF Terminal`

1. Config

``` sh
idf.py menuconfig
```

down: <kbd>J</kbd>   up:<kbd>K</kbd> 

settings are in the "Bridge Configuration" sub-menu, swap  TDO/TDI target pin num.

2. Build

``` sh
idf.py build
```

3. Flash

``` sh
idf.py -p PORT flash monitor
```

PORT is the serial port created by an USB-to-UART chip

Tips: set boot load mode

![Step](set-boot-step.png)

4. Reboot

## 3.Install Driver

Windows Device manager

![device manager 0](device-manager-0.png)

install driver

[UsbDriverTool](https://visualgdb.com/UsbDriverTool/)

![install driver.png](install-driver.png)

install success

![device manager 1](device-manager-1.png)

## 4.Connect Target

1. connect target mcu

![concept](concept.png)

2. set target mcu download mode

![target mcu download mode](target-mcu-download-mode.png)

3. OpenOCD connect ESP32-WROVER-E

<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> ESP-IDF: Open ESP-IDF Terminal

``` sh
openocd.exe -f interface/esp_usb_bridge.cfg -f target/esp32.cfg
```

![connect esp32 wrover](connect-esp32-wrover.png)

3. use jtag flash

![use jtag flash](use-jtag-flash.png)

4. use serial

   * Select Port  (COMx)

   * ESP-IDF Monitor device

   * reboot target mcu

![serial test](serial-test.png)


