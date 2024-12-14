---
title: 在docker中运行ROS2容器并测试思岚激光雷达
date: 2024-07-03 22:34:36
tags: ROS2
---

## 1.docker安装及基础知识

[docker安装](https://www.yahboom.com/build.html?id=7178&cid=586)

[国内镜像站点]( https://linux.do/t/topic/114516)

[yahboomtechnology/ros2-base镜像](https://hub.docker.com/r/yahboomtechnology/ros2-base/tags)

[导出镜像并压缩、解压并导入镜像](https://blog.csdn.net/qq_34777982/article/details/102454739)

[创建容器 硬件挂载 GUI显示](https://www.yahboom.com/build.html?id=7182&cid=586)  脚本用来创建容器，仅需运行一次。后期使用docker start/stop开控制容器启停

[在docker中使用GPIO](https://github.com/NVIDIA/jetson-gpio/tree/master) `--privileged \` 此选项使docker镜像使用所有宿主机的硬件

建立[udev规则](https://github.com/Slamtec/rplidar_ros/blob/ros2/scripts/rplidar.rules)(将指定ttyUSB映射为rplidar并修改[使用权限](https://www.small09.top/posts/210311-serialmodeandudevinlinux/))

```bash
~$ ls -al /dev/ttyUSB0 /dev/rplidar 
lrwxrwxrwx 1 root root         7 6月  25 09:14 /dev/rplidar -> ttyUSB0
crwxrwxrwx 1 root dialout 188, 0 6月  25 09:14 /dev/ttyUSB0
```

创建docker的脚本 create.sh  此方式不会将ttyUSB0 映射为rplidar，也不需要在ros2中指定串口

```bash
!/bin/bash
xhost +

docker container run -it \
--net=host \
--env="DISPLAY" \
--env="QT_X11_NO_MITSHM=1" \
--privileged \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-v /home/zy/Code:/root/yahboomcar_ros2_ws/temp \
-p 9090:9090 \
-p 8888:8888 \
yahboomtechnology/ros2-base:2.0.2 /bin/bash
```

## 2.常用指令

```bash
docker可以使用界面。每次重启后必须要在VNC远程桌面中打开的Terminal后执行下面这段指令，开启GUI访问权限，再启动容器。
xhost +local:root
或
xhost +

查看容器运行情况
docker ps -a

运行容器
docker run 后面加一堆参数 包含镜像名称和容器标签

启动容器  运行过一次后面仅需使用管理容器指令
docker start [ID或标签]
docker stop
docker rm

再次进入容器
docker attach [ID或标签]
新建shell进入容器
docker exec -it [容器id] /bin/bash

exit        # 容器停止退出
ctrl+P+Q    # 容器不停止退出

运行软件，查看能否开启GUI画面
rviz2
```



## 3.进入docker

[ROS2示例测试](https://book.guyuehome.com/ROS2/1.%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84/1.3_ROS2%E5%AE%89%E8%A3%85%E6%96%B9%E6%B3%95/#ros2_2) 只按ROS2示例测试小节测试docker环境的的ROS2是否运行正常



拉取思岚雷达在ROS2中应用的[git仓库](https://github.com/Slamtec/rplidar_ros/tree/ros2)，切换到ros2分枝。按README.md说明建立和编译项目。

```bash
git clone -b ros2 https://github.com/Slamtec/rplidar_ros.git
```

注意：

项目默认使用 /dev/ttyUSB0 ，然而我们在建立docker容器时将宿主机的ttyUSB0最终映射为docker中的rplidar

故此时在docker中执行

```bash
~$ ls /dev
console  fd  full  mqueue  null ptmx  pts  random  rplidar  shm  stderr  stdin  stdout  tty  urandom  zero
```

发现是没有ttyUSB0的。



README.md中的运行指令

```bash
ros2 launch rplidar_ros view_rplidar_s2_launch.py
```

需要指定串口号


```bash
ros2 launch rplidar_ros view_rplidar_s2_launch.py serial_port:=/dev/rplidar
```