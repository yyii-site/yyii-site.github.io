---
title: neovim和插件
date: 2024-10-20 13:28:50
tags: nvim
---

## 安装neovim

[neovim-releases](https://github.com/neovim/neovim-releases/releases)下载安装包

```bash
sudo dpkg -i  nvim-linux64.deb
```

## 新建配置文件

[kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim)

[YouTube教程](https://youtu.be/stqUbv-5u2s?si=MZ-nfA6-Y0KOlqe9)

```bash
git clone https://github.com/nvim-lua/kickstart.nvim.git "${XDG_CONFIG_HOME:-$HOME/.config}"/nvim
```

## 使用

### 命令

`:Lazy` 插件状态

`:Mason` LSP自动补全插件

`:Telescope keymap`

### 快捷键

`C+F` 查找  `i` 安装

`C+N` 下一条  `C+P` 上一条 `C+Y` 接受

`[` `]` 跳转

`leader` 被配置为 Space空格键

`<leader>?` 最近打开的文件

`<leader><leader>` 当前打开的缓冲区

`<leader>/` 当前文件搜索

`gd` Goto Definiton

`gr` Goto References

`gI` Goto Implementation

