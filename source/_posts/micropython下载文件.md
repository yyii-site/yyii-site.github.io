---
title: micropython下载文件
date: 2025-04-16 21:14:06
tags: micropython
---

## micropython 使用 requests 库下载文件

库源码：[requests github](https://github.com/micropython/micropython-lib/blob/master/python-ecosys/requests/requests/__init__.py)

socket 中文文档：http://micropython.86x.net/en/latet/library/socket.html

socket 英文文档：https://docs.micropython.org/en/latest/library/socket.html

### 本地HTTP服务器

启动用于测试的简单本地HTTP服务器： https://www.cnblogs.com/shengshengwang/p/17939492

```python
python -m http.server
```

运行这个命令后，默认会在当前目录下启动一个 HTTP 服务器，监听在本地的 8000 端口。可以通过浏览器访问 http://localhost:8000 来查看当前目录的文件列表。 也可以自定义端口号 `python -m http.server 8888`


### 问题

下面是 requests 库的 content 属性的实现，会通过 socket 库的 read() 函数将数据一次性全部去读到 self._cached 变量，也就是内存中。这种方式在读取小文件时没有问题，但是文件体积稍大一些后就会出现**响应缓慢**，严重的话会**爆内存**的情况，毕竟运行 micropython 的嵌入式设备资源都比较紧张。

```python
@property
def content(self):
    if self._cached is None:
        try:
            self._cached = self.raw.read()
            finally:
                self.raw.close()
                self.raw = None
                return self._cached
```

### 解决

参考 python requests 库的用法，链接：https://www.cnblogs.com/hls-code/p/15012629.html

流式下载能满足要求，但是 micropython 的 requests 库并没有 `iter_content()` 函数的实现。

通过循环，每次读取一定量的数据是否可行？代码如下：

```
url = "http://xx.xxx.xxx.xxx:8888/yolov8n_224.kmodel"
r = libs.requests.get(url, stream=True)
if r.status_code == 200:
	print("Downloading file...")
	with open('/sdcard/yolov8n_224.kmodel', 'wb') as pic_file:  # 保存文件
		while True:
			chunk = r.raw.recv(1024*20)
			if not chunk:
				break
			pic_file.write(chunk)
			pic_size += len(chunk)
			print(pic_size)
	print("Downloading finish")
r.close()
```

提示：在开发板运行代码时注意将 xxx 替换为 http 服务器的 ip，而不是127.0.0.1 或者 localhost；检查拉取和存储的文件名称和路径。

### 其他

1. read()  最后一包数据丢失。

   使用 r.raw.read(1024) 发现最后一包数据总是收不到，查看 socket 文档 `socket.read([size])` 有一段说明

   *This function tries to read as much data as requested (no “short reads”). This may be not possible with non-blocking socket though, and then less data will be returned.* 

   **没有短读**：即不会返回“短读取”（即少于请求字节数的数据） 造成最后一包数据丢失。

2. content()  非常慢


```python
with open('/sdcard/6.bmp', 'wb') as pic_file:  # 保存文件
    pic_file.write(r.content)
```



### 完整示例

```python
import network,time
from machine import Pin
import libs.requests

#WIFI连接函数
def WIFI_Connect():

    WIFI_LED=Pin(52, Pin.OUT) #初始化WIFI指示灯

    wlan = network.WLAN(network.STA_IF) #STA模式
    wlan.active(True)                   #激活接口
    start_time=time.time()              #记录时间做超时判断

    if not wlan.isconnected():
        print('connecting to network...')
        for i in range(3): #重复连接3次
            #输入WIFI账号密码（仅支持2.4G信号）, 连接超过5秒为超时
            wlan.connect('xxxxx', 'xxxx')
            if wlan.isconnected(): #连接成功
                break

    if wlan.isconnected(): #连接成功
        #LED蓝灯点亮
        WIFI_LED.value(1)

        #等待获取IP地址
        while wlan.ifconfig()[0] == '0.0.0.0':
            pass

        #串口打印信息
        print('network information:', wlan.ifconfig())
        return True

    else: #连接失败

        #LED闪烁3次提示
        for i in range(3):
            WIFI_LED.value(1)
            time.sleep_ms(300)
            WIFI_LED.value(0)
            time.sleep_ms(300)

        wlan.active(False)

        return False


def DownloadFile():
    pic_size = 0
    try:
        url = "http://192.168.1.100:8888/6.bmp"
        r = libs.requests.get(url, timeout=None, stream=True)
        if r.status_code == 200:
            print("Downloading file...")
            with open('/sdcard/6.bmp', 'wb') as pic_file:  # 保存文件
#                pic_file.write(r.content)  # 非常慢
                while True:
#                    chunk = r.raw.read(1024*20)  # 没有短读 即不会返回“短读取”（即少于请求字节数的数据） 造成丢最后一包数据
                    chunk = r.raw.recv(1024*20)
                    if not chunk:
                        break
                    pic_file.write(chunk)
                    pic_size += len(chunk)
                    print(pic_size)
            print("Downloading finish")
        r.close()
    except UnicodeError as e:
        print("Unicode error:", e)
    except Exception as e:
        print("HTTP request error:", e)


WIFI_Connect()
DownloadFile()
```

修改 wifi 文件名 和存储路径