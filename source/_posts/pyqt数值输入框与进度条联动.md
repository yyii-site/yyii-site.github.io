---
title: pyqt数值输入框与进度条联动
date: 2023-08-06 22:46:24
tags: PyQt
---

QSlider QSpinBox QDoubleSpinBox

## 1.背景

1. QSpinBox/QDoubleSpinBox（数值输入框）：

   使用`valuechange`信号，在修改数值的过程中会发送信号，如输入123，则会触发1、12、123三条信号。若是删除原有数据，产生的信号会更多。`editingFinished`在输入完成后鼠标离开控件（失去焦点）或按下回车才会触发信号。

2. QSlider （进度条）：

   使用`valuechange`信号，在滑块拖动过程中，信号将会被频繁发送。 如果是上位机通过串口向下位机发送指令，一般下位机（简单MCU做主控）无法响应高频率的指令。	`released`信号只有在释放滑块时才会发送，但点击slider部件造成的滑块移动，并不会发送released信号，就造成当前滑块位置与实际参数不一致的BUG。



## 2.控件联动和信号的产生

1. 创建控件

```python
# 整数
self.d_slider = QtWidgets.QSlider(self.xxx)
self.d_spin_box = QtWidgets.QSpinBox(self.xxx)
# 小数
self.f_slider = QtWidgets.QSlider(self.xxx)
self.f_slider = QtWidgets.QDoubleSpinBox(self.xxx)
```



2. double数值输入和整数进度条控件相互转换需特殊处理（此处固定为1个小数点）

```python
def xx_sTb
	self.f_spin_box.setValue(self.f_slider.value() / 10)
    
def xx_bTs
	self.f_slider.setValue(self.f_spin_box.value() * 10)
```



3. 实现输入框和进度条联动（信号和槽）

```python
# 整数直接相互设置
self.d_slider.valueChanged.connect(self.d_spin_box.setValue)
self.d_spin_box.valueChanged.connect(self.d_slider.setValue)

#小数需通过函数转换
self.f_slider.valueChanged.connect(self.xx_sTb)
self.f_spin_box.valueChanged.connect(self.xx_bTs)
```



4. 槽函数

```python
# 槽函数
self.event_list = [
	lambda: self._send_message(0), lambda: self._send_message(1)
]

# 具体操作内容
def _send_message(self, id: int):
    if id == 0:
        print("send0")
        buf = self.d_spin_box.value()  # 数值输入框的值 进度条仅为整数需放大和缩小
    if id == 1:
        print("send1")
        buf = self.f_spin_box.value()
```



5. 实现信号发出（信号和槽）

```
# 
self.d_slider.sliderReleased.connect(self.event_list[0])
self.d_spin_box.editingFinished.connect(self.event_list[0])

self.f_slider.sliderReleased.connect(self.event_list[1])
self.f_spin_box.editingFinished.connect(self.event_list[1])

```



6. 以上实现了控件联动和信号的产生发送，但是开头提到的**滑块位置与实际参数不一致的BUG**没有解决



## 3.解决滑块单机无信号的问题

1. 安装事件过滤器

```python
# 为滑块安装过滤器
self.d_slider.installEventFilter(self)
self.f_slider.installEventFilter(self)
```



2. 重写事件过滤方法。将滑块的左键单击，修改为滑块释放信号

```python
def eventFilter(self, a0: 'QObject', a1: 'QEvent') -> bool:
    name = a0.objectName()
    # 判断控件名称 防止与其他安装了过滤器的控件相互影响
    if name.endswith("_slider") or name.endswith("_dslider"):
        if a1.type() == a1.MouseButtonRelease and a1.button() == 1: # 判断左键
            0.setSliderDown(True)  #模拟滑块的点击
            a0.setSliderDown(False) #模拟滑块的点击
            # 必需先按下后释放，才能触发sliderReleased信号
    return False
```
