---
title: nodejs安装
date: 2024-10-06 11:20:39
tags:  Node.js
---

## 简介

### 维基百科

Node.js 是跨平台、开源的 JavaScript 运行环境，可在 Windows、Linux、macOS 等操作系统上运行。

Node.js 大部分基本模块都用 JavaScript 语言编写。在 Node.js 出现之前，JavaScript 通常作为客户端程序设计语言使用，以JavaScript 写出的程序常在用户的浏览器上执行。Node.js 的出现使 JavaScript 也能用于服务端编程。Node.js 含有一系列内置模块，使得程序可以脱离 Apache HTTP Server 或 IIS，作为独立服务器执行。 

[wikipedia](https://zh.wikipedia.org/zh-cn/Node.js)

### 个人理解

在 Node.js 出现之前，JavaScript 是在浏览器中执行的，主要用于前端开发。Node.js 为 JavaScript 提供了一个独立运行环境，而无需依赖浏览器，使其能够用于编写后端逻辑。

## 软件

### 管理 Node.js

安装和切换 Node.js 的不同版本

[nvs](https://github.com/jasongin/nvs) (Node Version Switcher)。

```bash
nvs
nvs add latest
nvs add lts
nvs add <version>
nvs use lts
nvs use <version>
nvs list
nvs link lts
nvs alias default <version>
```

[nvm](https://github.com/nvm-sh/nvm) (Node Version Manager)

### 管理包

用于安装和管理 JavaScript 包

[npm](https://www.npmjs.com/) (Node Package Manager) npm 是 Node.js 的默认包管理器。

```bash
npm install <package-name>
npm install -g <package-name>
npm uninstall <package-name>
```

[yarn](https://yarnpkg.com/) 是Facebook 开发的一个替代 npm 的包管理工具

```bash
yarn add <package-name>
yarn remove <package-name>
yarn upgrade <package-name>
```

### npx

npx 是 npm 附带的工具，用于执行 npm 包中的命令，而无需全局安装它们。

```bash
npx <package-name>
npx <package-name>@<version>
```
