---
title: 一主机多客户端websocket通讯
date: 2024-08-25 16:25:39
tags: Python
---

uvicorn FastAPI WebSocket

## 1.概述

websocket服务器，接收多个客户端连接。接收客户端的数据经由算法处理后，将结果返回给对应客户端。

fastapi同时支持http协议，可自行增加可提供网页服务。


## 2.程序文件

自行准备一张jpg格式的图片，复制到程序文件夹中，并重命名为`picture_in.jpg`

将以下代码存储为对应名称的文件，并全部放到程序文件夹中

```python
# pyqt_app.py
import sys
import cv2
from PyQt5.QtCore import Qt
from PyQt5.QtCore import QThread
from PyQt5.QtWidgets import QApplication, QMainWindow, QVBoxLayout, QWidget, QLabel
from web_api import WebTask, ProcessingTask


class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("FastAPI + PyQt")

        # 创建一个垂直布局
        layout = QVBoxLayout()

        # 创建一个标签并添加到布局中
        self.label = QLabel()
        layout.addWidget(self.label)

        self.lab_img = QLabel()
        layout.addWidget(self.lab_img)

        # 创建一个 widget 并设置布局
        central_widget = QWidget()
        central_widget.setLayout(layout)
        self.setCentralWidget(central_widget)

    def change_txt(self, msg):
        self.label.setText(msg)

    def display_img(self, id, num, im):
        print(f'socket:{id}, image:{num}')
        cv2.imshow('im', im)
        cv2.waitKey(1)


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()

    # Web
    webThread = QThread()
    web = WebTask()
    web.moveToThread(webThread)
    webThread.started.connect(web.long_running)
    web.signal_text.connect(window.change_txt)
    # web.signal_client_msg.connect(window.display_img)

    # Processing
    procThread = QThread()
    proc = ProcessingTask()
    proc.moveToThread(procThread)
    procThread.started.connect(proc.long_running)
    # web.signal_client_msg.connect(proc.receive_msg, Qt.DirectConnection)

    webThread.start()
    procThread.start()

    sys.exit(app.exec_())

```


```python
# web_api.py
import base64
import asyncio
import threading
import time
import cv2
import numpy as np
import uvicorn
from io import BytesIO
from typing import List
from PIL import Image
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from PyQt5.QtCore import QObject, pyqtSignal
from collections import deque
from IDQueue import IDQueue

max_len_queue = 5
socket_to_ai_queue = deque(maxlen=max_len_queue)
ai_to_socket_queue = IDQueue()
socket_to_ai_lock = threading.Lock()
ai_to_socket_lock = threading.Lock()


class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)


class WebTask(QObject):
    signal_text = pyqtSignal(str)
    signal_client_msg = pyqtSignal(int, int, np.ndarray)

    def __init__(self):
        super().__init__()
        self.app = FastAPI()
        self.manager = ConnectionManager()

    def long_running(self):
        @self.app.websocket("/ws_bytes/{client_id}")
        async def websocket_client_msg(websocket: WebSocket, client_id: int):
            await self.manager.connect(websocket)
            ai_to_socket_queue.clear(client_id)
            try:
                while True:
                    data = await websocket.receive_json()
                    images = data.get("images", {})
                    img_b64 = images.get("encoded", "")
                    img_id_send = images.get("image_id", -1)
                    if len(img_b64) > 0:
                        img_file = BytesIO(base64.b64decode(img_b64))
                        # 方式一  Image
                        # im = Image.open(img_file)
                        # im.save('picture_out.jpg', "JPEG")
                        # 方式二  cv2
                        img_array = np.frombuffer(img_file.getvalue(), dtype=np.uint8)
                        im = cv2.imdecode(img_array, cv2.IMREAD_COLOR)
                        # cv2.imshow('im', im)
                        # cv2.waitKey(1)
                        # cv2.destroyAllWindows()
                        # self.signal_client_msg.emit(client_id, img_id, im)

                        # 发送给处理线程
                        if len(socket_to_ai_queue) < max_len_queue:
                            with socket_to_ai_lock:
                                print("socket_to_ai_queue append")
                                socket_to_ai_queue.append((client_id, img_id_send, im))
                        else:
                            print("socket_to_ai_queue full")
                        # 从处理线程接收结果
                        timeout = 1000  # 超时时长 1000*0.003=3秒
                        while timeout:
                            timeout -= 1
                            if ai_to_socket_queue.len(client_id):
                                with ai_to_socket_lock:
                                    print("ai_to_socket_queue pop")
                                    img_id_read, count = ai_to_socket_queue.pop(client_id)
                                    if img_id_send == img_id_read:
                                        break
                                    else:
                                        print(f"img_id_send:{img_id_send} != img_id_read:{img_id_read}")
                            else:
                                await asyncio.sleep(0.003)

                    await self.manager.send_personal_message(f"You client_id:{client_id}, img_id:{img_id_send}, count:{count}", websocket)
            except WebSocketDisconnect:
                self.manager.disconnect(websocket)

        # 启动 Uvicorn 服务器
        asyncio.create_task(uvicorn.run(self.app, host="0.0.0.0", port=8000))


class ProcessingTask(QObject):
    def __init__(self):
        super().__init__()

    def work(self):
        print("work start")
        time.sleep(0.1)
        print("work finsh")

    def send_result(self, client_id, val):
        if ai_to_socket_queue.len(client_id) < max_len_queue:
            with ai_to_socket_lock:
                print("ai_to_socket_queue append")
                ai_to_socket_queue.add(client_id, val)
        else:
            print("ai_to_socket_queue full")

    def long_running(self):
        count = 0
        print(f'ProcessingTask running')
        while True:
            if len(socket_to_ai_queue) > 0:
                with socket_to_ai_lock:
                    print("socket_to_ai_lock popleft")
                    client_id, img_id, im = socket_to_ai_queue.popleft()
                print(f"thread count:{count}")
                count += 1
                self.work()
                self.send_result(client_id, (img_id, count))
            else:
                time.sleep(0.003)

```

```python
# IDQueue.py
from collections import OrderedDict, deque

class IDQueue:
    def __init__(self):
        self.queue_dict = OrderedDict()
        self.queue_lengths = {}

    def add(self, id, item):
        if id not in self.queue_dict:
            self.queue_dict[id] = deque()
            self.queue_lengths[id] = 0
        self.queue_dict[id].append(item)
        self.queue_lengths[id] += 1
        self.queue_dict.move_to_end(id)

    def pop(self, id):
        if id not in self.queue_dict or not self.queue_dict[id]:
            return None
        item = self.queue_dict[id].popleft()
        self.queue_lengths[id] -= 1
        if self.queue_lengths[id] == 0:
            del self.queue_dict[id]
            del self.queue_lengths[id]
        return item

    def len(self, id):
        return self.queue_lengths.get(id, 0)

    def get_queue(self, id):
        return self.queue_dict.get(id, None)

    def clear(self, id):
        print(f"IDQueue {id} clear")
        if id in self.queue_dict:
            del self.queue_dict[id]
            del self.queue_lengths[id]


queue = IDQueue()

# 添加元素
queue.add(1, 'apple')
queue.add(1, 'banana')
queue.add(2, 'cherry')
queue.add(2, 'date')

# 获取队列长度
print(queue.len(1))  # Output: 2
print(queue.len(2))  # Output: 2

# 弹出元素
print(queue.pop(1))  # Output: 'apple'
print(queue.pop(2))  # Output: 'cherry'

# 获取队列
print(list(queue.get_queue(1)))  # Output: ['banana']
print(list(queue.get_queue(2)))  # Output: ['date']

```

```python
# fastapi_websockets_client1.py
import asyncio
import base64, json
import time
import websockets

last_time = time.time()
last_step = 0


def ptime():
    global last_time, last_step
    now = time.time()
    print(f"step{last_step}: {now - last_time}s    now: {now}")
    last_time = now
    last_step += 1


async def send_image(client_id, filename):
    async with websockets.connect(f'ws://127.0.0.1:8000/ws_bytes/{client_id}') as websocket:
        # 替换为你的图片文件路径
        for i in range(0, 20):
            with open(filename, 'rb') as imgfile:
                base64_bytes = base64.b64encode(imgfile.read())
                base64_encoded = base64_bytes.decode()
            data = {
                'hard_info': {
                    'num': 1,
                    "name": "1号工位",
                    'ip': "192.168.0.10"
                },
                'images': {
                        'camera_index': 1,
                        'image_id': i,
                        'encoded': base64_encoded
                }
            }
            print(f"start{i}")
            await websocket.send(json.dumps(data))
            response = await websocket.recv()
            print(response)
            ptime()


# 替换为你想要使用的客户端 ID
client_id = 1
img_path = 'picture_in.jpg'
asyncio.get_event_loop().run_until_complete(send_image(client_id, img_path))
```


## 3.服务器端说明

文件： pyqt_app.py  web_api.py  IDQueue.py


主线程：  PyQt界面
子线程1：  **WebTask**  uvicorn + fastapi
子线程2 ： **ProcessingTask**  算法


数据流一：从websocket到计算线程
```
websocket1       ---
websocket2       ---==== 队列 ====    计算线程
websocket3       ---
```

数据流二：从计算线程到websocket
```
                                          ---    websocket1
计算线程  ==== IDQueue 带socket Id的队列 ====---    websocket2
                                          ---    websocket3
```


## 4.客户端

fastapi_websockets_client.py

多复制几份，修改`client_id = 1`为其他数字。

## 5.测试

1号终端运行

```bash
python  pyqt_app.py
```

1号终端运行服务器

```bash
python  pyqt_app.py
```

2号终端运行客户端1

```bash
python  fastapi_websockets_client1.py
```

3号终端运行客户端2

```bash
python  fastapi_websockets_client2.py
```

客户端日志

![client_log.png](client_log.png)