---
title: Gitblit 安装使用
date: 2020-11-22 19:33:27
tags: Gitblit
---

Gitblit是一个在Java环境中运行的Git服务器。在这里记录一下在Windows上搭建Gitblit服务的过程。

[Gitblit](https://gitblit.github.io/gitblit/) is an open-source, pure Java stack for managing, viewing, and serving Git repositories.
It's designed primarily as a tool for small workgroups who want to host centralized repositories.

## 1.准备文件

* jdk-11.0.2_windows-x64_bin.zip[华为源](https://repo.huaweicloud.com/java/jdk/)
* gitblit安装包[Gitblit](https://gitblit.github.io/gitblit/)

## 2.安装Java

请参考[菜鸟教程](https://www.runoob.com/java/java-environment-setup.html)

## 3.配置Gitblit
1. 下载的zip文件只要解压缩即可，不用安装。建议放到D:\Program Files\gitblit-xxx目录中（xxx指版本号）
2. 新建一个用于存放git服务器数据的文件夹，如D:\GitBlit_repository
3. 配置gitblit.properties 文件。  
    * 在D:\Program Files\gitblit-xxx\data目录中将defaults.properties文件复制一份，改名为my.properties  
    * 修改gitblit.properties文件，然后将 `include = defaults.properties` 注释掉  
    * 添加一代码`include = my.properties`表示使用my.properties这个配置。  
4. 修改my.properties文件中的端口和服务器IP地址：  
    * 修改git.repositoriesFolder = D:/Gitblit_repository（注意其中的D:\Gitblit_repository 中的"\"一定要用"/"。）  
    * 修改server.httpPort = 10101  
    * 修改server.httpBindInterface = 192.168123.15（我自己的服务器IP地址为192.168.123.15）  
    * 修改server.httpsBindInterface = localhost  
    * 最后修改server.certificateAlias = localhost
5. 运行Gitblit服务。在D:\Program Files\gitblit-xxx目录下运行gitblit.cmd命令。注意看命令行中的提示，检查程序是否运行正常。
6. 命令行中会有服务器ip和端口号请注意查看。

## 4.管理GitBlit

在浏览器地址栏中输入命令行中提示的服务器ip和端口号

如果成功加载，说明服务器搭建完毕。默认账号密码均为admin

## 5.设置开启自启动

设置以Windows Service方式启动Gitblit.  
1. 在Gitblit目录下，找到installService.cmd文件。鼠标右键使用记事本打开。  
2. 设置 CD 为程序目录 `SET CD=D:\Program Files\gitblit-xxx(xxx修改为你自己的目录)`  
3. 保存，关闭文件
4. 右键installService.cmd文件，选择以管理员的身份运行  
5. 点击开始菜单输入services.msc 打开Windows服务查找gitblit，如果未启动，请手动启动。注意确保为自动模式，这样每次windows启动后都自动启动此项服务。