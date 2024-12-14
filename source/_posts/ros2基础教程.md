---
title: ros2基础教程
date: 2024-07-03 22:50:30
tags: ROS2
---

## 1.基础知识
### 1.1工作空间

* 创建工作空间

  一个存放项目开发相关文件的文件夹，也是开发过程中存放所有资料的大本营。 

```bash
# 选择一个存储的目录，也可以放在根目录
cd ~/yahboomcar_ros2_ws
# 创建工作的空间
mkdir -p yahboomcar_ws/src
cd yahboomcar_ws
```

* 编译

```bash
colcon build
```

* 设置环境变量

```bash
# 仅在当前终端生效
source install/setup.bash   
# 所有终端均生效
echo "source ~/yahboomcar_ros2_ws/yahboomcar_ws/install/setup.bash" >> ~/.bashrc    
```



### 1.2功能包

 把不同功能的代码划分到不同的功能包中，尽量降低他们之间的耦合关系 

* 创建c++功能包

```bash
cd ~/yahboomcar_ros2_ws/yahboomcar_ws/src
# 创建C++功能包
ros2 pkg create pkg_helloworld_cpp --build-type ament_cmake --dependencies rclcpp --node-name helloworld  
```

* 创建python功能包

```bash
cd ~/yahboomcar_ros2_ws/yahboomcar_ws/src
ros2 pkg create pkg_helloworld_py --build-type ament_python --dependencies rclpy --node-name helloworld
```

* 目录结构

```bash
WorkSpace --- 自定义的工作空间。
    |--- build：存储中间文件的目录，该目录下会为每一个功能包创建一个单独子目录。
    |--- install：安装目录，该目录下会为每一个功能包创建一个单独子目录。
    |--- log：日志目录，用于存储日志文件。
    |--- src：用于存储功能包源码的目录。
        |-- C++功能包
            |-- package.xml：包信息，比如:包名、版本、作者、依赖项。
            |-- CMakeLists.txt：配置编译规则，比如源文件、依赖项、目标文件。
            |-- src：C++源文件目录。
            |-- include：头文件目录。
            |-- msg：消息接口文件目录。
            |-- srv：服务接口文件目录。
            |-- action：动作接口文件目录。
        |-- Python功能包
            |-- package.xml：包信息，比如:包名、版本、作者、依赖项。
            |-- setup.py：与C++功能包的CMakeLists.txt类似。
            |-- setup.cfg：功能包基本配置文件。
            |-- resource：资源目录。
            |-- test：存储测试相关文件。
            |-- 功能包同名目录：Python源文件目录。
```

* 其他文件

```bash
|-- C++或Python功能包
    |-- launch：存储launch文件。
    |-- rviz：存储rviz2配置相关文件。
    |-- urdf：存储机器人建模文件。
    |-- params：存储参数文件。
    |-- world：存储仿真环境相关文件。
    |-- map：存储导航所需地图文件。
    |-- ......
```



### 1.3节点

 每个节点都对应某一单一的功能模块  (例如：雷达驱动节点可能负责发布雷达消息，摄像头驱动节点可能负责发布图像消息) 

 一个完整的机器人系统可能由许多协同工作的节点组成，ROS2中的单个可执行文件(C++程序或Python程序)可以包含一个或多个节点。 

 在通信时，不论采用何种方式，通信对象的构建都依赖于节点(Node) 

`pkg_helloworld_py/helloworld.py` 来编写节点功能



setup.py 定义口入点

```python
entry_points = {
	'console_scripts':[
        # 入口函数名
        'helloworld = pkg_helloworld_py.helloworld:main'
        # 可执行文件名      功能包名称      编写节点功能的python文件名称
    ],
},
```

* 只编译特定功能包

```bash
cd ~/yahboomcar_ros2_ws/yahboomcar_ws
colcon build --packages-select pkg_helloworld_py
source install/setup.bash
```

* 启动单个节点

```bash
ros2 run pkg_helloworld_py helloworld
```

* 启动多个节点

```bash
# 示例为其他项目
ros2 launch rplidar_ros rplidar_s2_lanuch.py
```



## 2.激光雷达

### 2.1发布雷达扫描数据

* 进入工作空间 `cd ~/ros2_ws/src`

* 拉取项目代码

```bash
git clone -b ros2 https://github.com/Slamtec/rplidar_ros.git
```

接下来的操作可以查看项目中的README.md文件。注意运行测试时需要增加串口端口设置 ：`serial_port:=/dev/rplidar`

* 不带界面

```bash
ros2 run rplidar_ros rplidar_node serial_port:=/dev/rplidar
```

或

```bash
ros2 launch rplidar_ros rplidar_s2_launch.py serial_port:=/dev/rplidar
```

* 带界面

```bash
ros2 launch rplidar_ros view_rplidar_s2_launch.py serial_port:=/dev/rplidar
```



### 2.2订阅雷达数据并判断最小距离

* 创建python功能包

```bash
cd ~/ros2_ws/src
ros2 pkg create rplidar_warning --build-type ament_python --dependencies rclpy --node-name rplidar_warning
```

* 修改文件内容实现订阅及最小距离判断功能

~/ros2_ws/src/rplidar_warning/rplidar_warning/rplidar_warning.py

```python
#!/usr/bin/env python
# coding:utf-8
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from std_msgs.msg import Bool
import numpy as np
import math
RAD2DEG = 180 / math.pi

class laserWarning(Node):
    def __init__(self,name):
        super().__init__(name)

        self.Buzzer_state = False
        self.laserAngle = 70
        self.ResponseDist = 0.5
        self.pub_Buzzer = self.create_publisher(Bool, '/Buzzer', 1)
        self.sub_laser = self.create_subscription(LaserScan, '/scan', self.laserCallback, 1000)

    def laserCallback(self, scan_data):
        if not isinstance(scan_data, LaserScan): return
        # 记录激光扫描并发布最近物体的位置（或指向某点）
        # Record the laser scan and publish the position of the nearest object (or point to a point)
        ranges = np.array(scan_data.ranges)
        # 创建距离列表，将检测范围内的有效距离放入列表中
        # create distance list, put the effective distance within the detection range into the list
        minDistList = []
        # 创建序列号，将有效距离对应的ID放入列表中
        # Create a serial number and place the ID corresponding to the valid distance in the list
        minDistIDList = []
        # 按距离排序以检查从较近的点到较远的点是否是真实的东西
        # if we already have a last scan to compare to:
        for i in range(len(ranges)):
            angle = (scan_data.angle_min + scan_data.angle_increment * i) * RAD2DEG
            # if angle > 90: print "i: {},angle: {},dist: {}".format(i, angle, scan_data.ranges[i])
            # 通过清除不需要的扇区的数据来保留有效的数据
            if abs(angle) > (180 - self.laserAngle):
                minDistList.append(ranges[i])
                minDistIDList.append(angle)
        if len(minDistList) == 0: return
        # 找到最小距离
        # Find the minimum distance
        minDist = min(minDistList)
        # 找到最小距离对应的ID
        # Find the ID corresponding to the minimum distance
        minDistID = minDistIDList[minDistList.index(minDist)]
        self.get_logger().info(f"find minimum {minDist} {minDistID}")

        if minDist <= self.ResponseDist:
            if self.Buzzer_state == False:
                b = Bool()
                b.data = True
                self.pub_Buzzer.publish(b)
                self.Buzzer_state = True
        else:
            if self.Buzzer_state == True:
                self.pub_Buzzer.publish(Bool())
                self.Buzzer_state = False


    def dynamic_reconfigure_callback(self, config, level):
        self.laserAngle = config['laserAngle']
        self.ResponseDist = config['ResponseDist']
        return config


def main():
    rclpy.init()
    tracker = laserWarning("laser_Warning")
    rclpy.spin(tracker)
    tracker.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```



* 删除 build install log 文件夹

```bash
rm -rf build install log
```

* 编译所有功能包

```bash
colcon build
```

* 编译选中的功能包(选择使用)

```bash
colcon build --packages-select pkg_helloworld_py
```

* 开启两个终端，并且都更新一次环境变量

```bash
source install/setup.bash
```

* 终端1发布雷达数据

```bash
ros2 launch rplidar_ros view_rplidar_s2_launch.py serial_port:=/dev/rplidar
```

* 终端2订阅雷达数据

```bash
ros2 run rplidar_warning rplidar_warning
```
