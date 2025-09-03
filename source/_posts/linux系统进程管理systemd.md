---
title: linux系统进程管理systemd
date: 2025-07-27 11:27:52
tags: linux
---

视频 https://youtu.be/y5E98mjcZfc?si=i5-pACE_PS9Nq1jM

## 介绍

systemd 作为 init 进程，其他进程是他的子进程

systemctl 是systemd 的管理工具

## 常用命令

`htop` 按 t 键可以树状视图查看进程列表

`pstree -p` 可以树状视图查看列表

`systemctl list-unit` 分类查看当前系统定义的系统unit (device mount path service socket timer)

要具体查看其中某一个unit 的状态，例如：`systemctl status sshd.service` 例如：systemctl status dnf-makecache.timer

停止 `systemctl stop chronyd.service`

启动 `systemctl start chronyd.service`

开机自启 `systemctl enable chronyd`

禁用自启 `systemctl disable chronyd`

## systemd unit 配置文件

查看当前系统中存在的配置文件 systemctl list-unit-files

查看文件路径和内容 systemctl cat chronyd.service

修改配置文件 systemctl edit chronyd.service 退出的时候如果有错误会有提示，也可以通过vim nano

/etc/systemd/system         自定义的

/lib/systemd/system          系统

/usr/lib/systemd/system   用户安装软件产生的

### service

编写配置文件 man 5 systemd.service 内部还有 examples

包含三个部分 [Unit] [Install] [Service]

`vim /etc/systemd/system/hello-world.service`

```bash
[Unit]
Description=Hello Wrold

[Service]
ExecStart=/usr/bin/python3 /root/hello-world.py

[Install]
WantedBy=multi-user.target
```

`cat hello-world.py`

```python
import logging
import time

logging.basicConfig(format='Date-Time ： %(asctime)s : Line No. : %(lineno)d - %(message)s', \
                   Level = logging.DEBUG)
while True:
    logging.debug("A Debug Logging Message")
    time.sleep(10)
```

`systemctl status hello-world`  如果没有找到服务 需要使用 `systemctl daemon-reload`

`systemctl start hello-world` 开始执行 status 查看状态

systemctl 可以查看的日志不多，可以使用 `journalctl -u hello-world.service`

### timer

`systemctl list-timers`

`man 5 systemd.timer`

包含三个部分 [Unit] [Install] [Timer]

cat hello-world.timer

```bash
[Unit]
Description=My Service Timer

[Timer]
OnUnitActiveSec=1min
Unit=hello-world.service

[Install]
WantedBy=timers.target
```

每隔一分钟执行一下 hello-world.service 的内容

hello-world.py 取消循环

```bash
import logging
import time

logging.basicConfig(format='Date-Time ： %(asctime)s : Line No. : %(lineno)d - %(message)s', \
                   Level = logging.DEBUG)
logging.debug("A Debug Logging Message")
```

`journalctl -u hello-world.service` 查看日志核每隔一分钟运行一次。