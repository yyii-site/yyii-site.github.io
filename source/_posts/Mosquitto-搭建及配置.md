---
title: Mosquitto 搭建及配置
date: 2021-03-21 11:52:37
tags: IoT
---

## 1.概述

Eclipse Mosquitto是一个开源消息代理，实现了MQTT协议版本3.1和3.1.1。Mosquitto轻量，适用于低功耗单板计算机到完整服务器的所有设备。Mosquitto项目还提供了用于实现MQTT客户端的C库以及非常受欢迎的mosquitto_pub和mosquitto_sub命令行MQTT客户端。

其他服务器代理实现:https://github.com/mqtt/mqtt.github.io/wiki/servers
 各操作系统安装指引:https://mosquitto.org/download/

## 2.下载安装

以Ubuntu为例

- 更新源

```bash
sudo apt-get update
```

- 安装服务器端

```bash
sudo apt-get install mosquitto
```

+ 安装客户端

```bash
sudo apt-get install mosquitto-clients
```

## 3.配置

### 3.1主配置文件mosquitto.conf

```nginx
id_file /var/run/mosquitto.pid

# 消息持久存储
persistence true
persistence_location /var/lib/mosquitto/

# 日志文件
log_dest file /var/log/mosquitto/mosquitto.log

# 其他配置
include_dir /etc/mosquitto/conf.d

# 禁止匿名访问
allow_anonymous false
# 认证配置
password_file /etc/mosquitto/pwfile
# 权限配置
acl_file /etc/mosquitto/aclfile
```

### 3.2认证配置pwfile

- 创建密码存储文件

```bash
sudo touch /etc/mosquitto/pwfile
```

- 服务开启后,输入如下命令,根据提示输入两遍密码

```bash
mosquitto_passwd /etc/mosquitto/pwfile 用户名
```

### 3.3权限配置aclfile

- 打开文件

```bash
vim /etc/mosquitto/aclfile
```

- 编辑内容

```nginx
# 李雷只能发布以test为前缀的主题,订阅以$SYS开头的主题即系统主题
user lilei
topic write test/#
topic read $SYS/#

# 韩梅梅只能订阅以test为前缀的主题
user hanmeimei
topic read test/#
```

## 4.启动

-c：指定特定配置文件启动
-d：后台运行

```bash
mosquitto -c /etc/mosquitto/mosquitto.conf -d
```

### 4.1测试

发布使用mosquitto_pub命令，订阅使用mosquitto_sub命令。常用参数介绍：

| 参数 | 描述                      |
| ---- | ------------------------- |
| -h   | 服务器主机，默认localhost |
| -t   | 指定主题                  |
| -u   | 用户名                    |
| -P   | 密码                      |
| -i   | 客户端id，唯一            |
| -m   | 发布的消息内容            |

订阅

```bash
mosquitto_sub -h localhost -t "test/#" -u hanmeimei -P 123456 -i "client1"
```

订阅系统主题

```bash
# 订阅客户端存活连接数
mosquitto_sub -h localhost –t '$SYS/broker/clients/active' -u lilei -P 123456 -i "client2"
```

发布

```bash
mosquitto_pub -h localhost -t "test/abc" -u lilei -P 123456 -i "client3" -m "How are you?"
```

## 5.参考链接

- 项目网站：https://www.eclipse.org/paho
- Eclipse项目信息：https://projects.eclipse.org/projects/iot.paho
- GitHub：https://github.com/eclipse/paho.mqtt.java
- MQTT Java客户端的使用：https://www.jianshu.com/p/65e1748a930c
- Spring支持：https://www.jianshu.com/p/6b60858b7d44