---
title: VsCode中使用PlatformIO
date: 2024-10-19 11:43:25
tags: PlatformIO
---

## 安装 Visual Studio Code

在 [官网](https://code.visualstudio.com/download) 下载 .deb 安装包

使用下面的指令安装（注意修改文件名）

```bash
sudo dpkg -i code_1.94.2-1728494015_amd64.deb
```

## 从开始菜单打开 Visual Studio Code

点击插件 (Ctrl + Shift + X)，搜索 PlatformIO 点击安装

如果PlatformIO弹窗提示没有找到python，我的系统已经安装了python3.11

此时需要安装 python3-venv 参考[链接](https://community.platformio.org/t/ubuntu-vscode-pio-extension-install-platformio-can-not-find-working-python-3-6-interpreter/27853/3)

```bash
https://community.platformio.org/t/ubuntu-vscode-pio-extension-install-platformio-can-not-find-working-python-3-6-interpreter/27853/3
```

继续安装

## 串口权限

通过串口下载程序，提示没有权限。此时需要根据提示配置udev

参考链接 [99-platformio-udev.rules](https://docs.platformio.org/en/stable/core/installation/udev-rules.htmlhttps://docs.platformio.org/en/stable/core/installation/udev-rules.html)

下载并将文件复制到 `/etc/udev/rules.d/` 文件夹中

重启udev服务 `sudo service udev restart`

查看串口所属组名，如下面的 `dialout`

```bash
ls -l /dev/ttyACM0
# crw-rw---- 1 root dialout 166, 0 juil. 10 13:43 /dev/ttyACM0
```

将当前用户加入到 `dialout` 组

```bash
sudo usermod -a -G dialout $USERNAME
```

注销后重新登陆