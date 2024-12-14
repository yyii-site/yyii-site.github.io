---
title: TheengsGateway应用指南
date: 2023-08-06 22:37:04
tags: IoT
---

Theengs Gateway Python

## 1.Advanced users - Build and install

1. 参考https://gateway.theengs.io/install/install.html#install-theengs-gateway-as-a-docker
2. 若使用自己修改的TheengsDecoder，注意在setup.py文件中修改依赖包的版本。
3. setup.py修改version版本号，例如`version="1.1.0"`
4. `__main__.py`调试项目，提示import失败，需将`from . import main`改为`from TheengsGateway import main`

增加对串口的支持

```python
    async def serial_wr_loop(self) -> None:
        async def read_from_serial(reader):
            count = 0
            self.running = True
            while not self.stopped:
                data = await reader.read(1000)
                count += 1

                logger.info(
                    "Received at `%s` from Serial: `%s`",
                    count,
                    data,
                )

                data_json = {}
                if 'None' in gw.serial_device:
                    logger.error("gw.serial_device is None, please check")
                    self.stopped = True
                elif 'YE600X' in gw.serial_device:
                    # YE600X 设备调试 使用串口助手以十六进制方式发送测试数据 1e 5f 00 41 00 4b 00 e1 07 07 0c 08 0a 19 40 00 00 04 00
                    try:
                        data_str = bytes.decode(data, encoding='utf8')
                        if data_str.find("CONNECTED") >= 0:
                            self.serial_mac = bytes.decode(data[data_str.find(",")+1:], encoding='utf8')
                            self.serial_connect_status = "online"
                            logger.info("serial device connected, mac: %s", self.serial_mac)
                        if data_str.find("DISCONN") >= 0:
                            self.serial_mac = bytes.decode(data[data_str.find(",")+1:], encoding='utf8')
                            self.serial_connect_status = "offline"
                            logger.info("serial device disconnected, mac: %s", self.serial_mac)
                    except:
                        logger.info("serial read ascii data")
                    data_json["name"] = "YE600X"
                    data_json["mac"] = self.serial_mac
                    data_json["connect_status"] = self.serial_connect_status
                    data_json["manufacturerdata"] = ''.join(['%02X' % b for b in data])
                elif 'sps' in gw.serial_device:
                    # sps 设备调试 使用串口助手以十六进制方式发送测试数据 af 09 7C 18 01 f9 9c 44 08
                    data_json["manufacturerdata"] = ''.join(['%02X' % b for b in data])
                    data_json["id"] = "88:33:22:FF:44:77"
                    data_json["name"] = "sps"
                    data_json["rssi"] = -15
                else:
                    logger.error("gw.serial_device is not defined, please check")
                    self.stopped = True

                decoded_json = decodeBLE(json.dumps(data_json))  # 将python对象编码成Json字符串
                if decoded_json is None:
                    logger.info("serial decodeBLE return None")
                else:
                    msg = decoded_json
                    gw.publish(
                        msg,
                        gw.presence_topic,
                    )
            logger.error("serial read loop stopped")
            self.running = False

        async def write_to_serial(writer):
            self.running = True
            while not self.stopped:
                writer.write(b'off\n')
                await writer.drain()
                await asyncio.sleep(2)
            logger.error("serial write loop stopped")
            self.running = False
        try:
            reader, writer = await serial_asyncio.open_serial_connection(url=gw.serial_port, baudrate=115200)
            task_1 = asyncio.create_task(read_from_serial(reader))
            task_2 = asyncio.create_task(write_to_serial(writer))
            await task_1
            await task_2
        except:
            self.running = False
            self.stopped = True
            logger.error("serial %s port open failed, please check", gw.serial_port)
            
            
def run(arg: str) -> None:
    # 修改
    thread = Thread(target=loop.run_forever, daemon=True)
    thread.start()
    asyncio.run_coroutine_threadsafe(gw.serial_wr_loop(), loop)
```

```python
def main() -> None:
    # 增加串口端口号和目标设备类型的选项
    parser.add_argument(
        "-sp", "--serial_port", dest="serial_port", type=str, help="Serial port"
    )
    parser.add_argument(
        "-si", "--serial_device_id", dest="serial_device", type=str, help="Serial device id, need check <TheengsDecoder/src/devices/list>"
    )
    # 将 serial_port  serial_device 加入配置文件的读写，并传递给 Gateway
```

