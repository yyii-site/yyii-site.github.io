---
title: 局域网内Raspberry上报IP地址脚本编写及部署
date: 2023-08-06 23:03:14
tags: RaspberryPi
---

IP UDP Python Shell

## 1.安装python包

```bash
sudo pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple netifaces
```

## 2.将脚本复制到指定的目录

/usr/local/bin/send_ipaddr_broadcast.py

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import socket
import netifaces
import signal
import sys
import time

def send_broadcast_message(sock):
    # 获取树莓派的IP地址
    interfaces = netifaces.interfaces()
    for iface in interfaces:
        addrs = netifaces.ifaddresses(iface)
        if netifaces.AF_INET in addrs:
            ip = addrs[netifaces.AF_INET][0]['addr']
            if ip.find('127.0.0.1') >= 0:
                continue

            # 发送UDP广播消息
            broadcast_ip = '255.255.255.255'  # 广播地址
            port = 7893  # 自定义端口号

            message = "Raspberry IP address is {}".format(ip)

            try:
                sock.sendto(message.encode(), (broadcast_ip, port))
            except socket.error as e:
                print("Failed to send broadcast message:", str(e))

def sigterm_handler(signal, frame):
    # 执行清理操作，例如关闭套接字等
    if sock:
        sock.close()
    sys.exit(0)

# 创建非阻塞式套接字
sock = None
try:
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
    sock.setblocking(0)  # 设置为非阻塞式套接字
except socket.error as e:
    print("Failed to create socket:", str(e))
    sys.exit(1)

# 注册SIGTERM信号处理函数
signal.signal(signal.SIGTERM, sigterm_handler)

# 设置等待时间间隔
wait_interval = 5  # 5秒

# 循环发送UDP广播消息
start_time = time.time()
while True:
    try:
        current_time = time.time()
        if current_time - start_time >= wait_interval:
            send_broadcast_message(sock)
            start_time = current_time
        time.sleep(1)  # 休眠1秒，降低CPU使用率
    except KeyboardInterrupt:
        # 如果收到键盘中断信号，退出循环
        break

# 关闭套接字
if sock:
    sock.close()
```


## 3.新建系统服务

* 将send_ipaddr_broadcast.py复制到/etc/systemd/system/

```shell
[Unit]
Description=Send UDP Broadcast Service
After=network.target

[Service]
ExecStart=/usr/local/bin/send_ipaddr_broadcast.py

[Install]
WantedBy=multi-user.target

```

* 开启服务
```bash
sudo systemctl start send_ipaddr_broadcast.service
```

* 开机自启
```bash
sudo systemctl enable send_ipaddr_broadcast.service
```

## 4.测试

* 打开网络调试助手

* 选择UDP模式

* 选择合适的本地主机地址（非127.0.0.1）
* 输入本地主机端口7893
* 点击开启按钮
* 接收设置选择：ASCII模式

![test](test.png)