---
title: ESP-USB-Bridge-JTAG-Debug
date: 2022-05-08 17:55:32
tags: ESP32
---

## 1.Create Project

Open VSCode

<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>    `ESP-IDF: Show Examples Projects`  `hello_world`

**Create project using example hello_world**

## 2.Setup

Select port to use (COMx)

set device targete

Build project

Select flash method (JTAG)

![image-20220508171334927](image-20220508171334927.png)

![image-20220508171702904](image-20220508171702904.png)

edit file: `.vscode/settings.json`  (注意修改**COMx**)

``` json
{
  "C_Cpp.intelliSenseEngine": "Tag Parser",
  "idf.portWin": "COMx",
  "idf.openOcdConfigs": [
    "interface/esp_usb_bridge.cfg",
    "target/esp32.cfg"
  ],
  "idf.flashType": "JTAG"
}
```

<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>  `ESP-IDF: OpenOCD Manager`  `start openocd`

![image-20220508173310006](image-20220508173310006.png)

[.vscode/settings.json](https://github.com/espressif/vscode-esp-idf-extension/blob/master/docs/DEBUGGING.md)

``` c
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "GDB",
      "type": "cppdbg",
      "request": "launch",
      "MIMode": "gdb",
      "miDebuggerPath": "${command:espIdf.getXtensaGdb}",
      "program": "${workspaceFolder}/build/${command:espIdf.getProjectName}.elf",
      "windows": {
        "program": "${workspaceFolder}\\build\\${command:espIdf.getProjectName}.elf"
      },
      "cwd": "${workspaceFolder}",
      "environment": [{ "name": "PATH", "value": "${config:idf.customExtraPaths}" }],
      "setupCommands": [
        { "text": "target remote :3333" },
        { "text": "set remote hardware-watchpoint-limit 2"},
        { "text": "mon reset halt" },
        { "text": "thb app_main" },
        { "text": "flushregs" }
      ],
      "externalConsole": false,
      "logging": {
        "engineLogging": true
      }
    }
  ]
}
```

## 3.Flash And Debug

flash

![image-20220508174242796](image-20220508174242796.png)

<kbd>F5</kbd>  

   *Try more times*

![image-20220508174435026](image-20220508174435026.png)

Other

  [https://github.com/espressif/vscode-esp-idf-extension/blob/master/docs/tutorial/debugging.md](https://github.com/espressif/vscode-esp-idf-extension/blob/master/docs/tutorial/debugging.md)


