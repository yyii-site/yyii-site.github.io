---
title: TheengsDecoder应用指南
date: 2023-08-06 22:35:24
tags: IoT
---

Theengs Decoder Python

## 1.项目搭建

1. clone工程
2. 将下载arduino_json项目并复制到src\arduino_json文件夹中
3. 如果是python使用，查看https://decoder.theengs.io/use/python.html#dependencies (windows 下不用apt-get install cmake指令，在pip install . 指令脚本会自动安装依赖)
4. 在src\device\中增加新的硬件解码规则，device.h中增加头文件，并加入列表
5. decoder.h中将新硬件加入列表（BLE_ID_MAX的前面）
6. 注意每次修改完解码规则文件，必须重新编译(例如python下使用pip install .)生效后方可测试

## 2.测试脚本

```python
import json
from TheengsDecoder import decodeBLE as dble

data_json = {"manufacturerdata": "1E5F0041004B00E107070C080A194000000400", "name": "YE600X"}
# data_json = {"manufacturerdata": "af097C1801f99c4408", "name": "sps"}
data = dble(json.dumps(data_json))
print(data)
```

## 3.新增解码规则

YE600X_json.h

被`/*R""""(`注释的代码要符合json格式

删除空格回车，将`"`前增加转义符`\`

索引编号从0开始，每个字符占一个；因为每个字符都是分开计算的，故位编号范围0-3

```
const char* _YE600X_json = "{\"brand\":\"YUWELL\",
/*R""""(            //血压
{
   "brand":"YUWELL",                   //品牌：鱼跃
   "model":"BloodPressureMeters",      //型号：血压仪
   "model_id":"YE600X",                //型号ID：YE600X
   "cidc":false,                       //起始字节不符合Bluetooth标准
   "condition":["manufacturerdata", "=", 38, "&", "name", "index", 0, "YE600X"],  //条件：name = YE600X
   "properties":{
      "pressure_unit":{
         "decoder":["bit_static_value", "manufacturerdata", 1, 0, "mmHg", "kPa"]  //血压单位： 第0字节 第0位  true：mmHg false：kpa
      },
      "systolic_pressure":{
         "decoder":["value_from_hex_data", "manufacturerdata", 2, 4, true]
      }
    }
}

const char* _YE600X_json_props = "{\"properties\":
/*R""""(
{
   "properties":{
      "pressure_unit":{
         "unit":"string",
         "name":"contact"
      },
      "systolic_pressure":{
         "unit":"string",
         "name":"contact"
      }
    }
}
```
