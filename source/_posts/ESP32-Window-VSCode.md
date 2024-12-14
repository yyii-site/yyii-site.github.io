---
title: ESP32 Windows VSCode IDE
date: 2022-05-02 15:50:41
tags: ESP32
---

[ESP-IDF Document](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html)

##  1.VSCode

[Download](https://code.visualstudio.com/download) and install  Visual Studio Code

## 2.VSCode Extension

1. [Espressif IDF](https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension)
2. [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
3. [Native Debug](https://marketplace.visualstudio.com/items?itemName=webfreak.debug)

## 3.Install ESP-IDF

<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd>  ESP-IDF: Configure ESP-IDF extension

Choose installation method (EXPRESS / ADVANCED / USE EXISTING SETUP)

## 4.Creat Project

<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> ESP-IDF: Show Examples Projects

get-started >hello_world > Create project using example hello_world

1. Select Port
2. Set Espressif device target
3. Build
4. Select flash method
5. flash device
6. Monitor device
