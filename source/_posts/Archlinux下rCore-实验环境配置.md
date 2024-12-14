---
title: Archlinux下rCore 实验环境配置
date: 2021-03-21 11:55:21
tags: Rust
---

## 1.Archlinux下rCore 实验环境配置

原文：[rCore-Tutorial-Book 第三版 实验环境配置](https://rcore-os.github.io/rCore-Tutorial-Book-v3/chapter0/5setup-devel-env.html#rust)

+ 系统环境配置（使用Archlinux）
+ Rust开发环境配置
+ Qemu模拟器安装
+ GDB 调试支持

## 2.Rust 开发环境配置

首先安装Rust包管理器cargo和Rust版本管理器rustup

```bash
sudo pacman -S cargo
sudo pacman -S rustup
```

修改Rust Crates源 [ustc](https://mirrors.ustc.edu.cn/help/crates.io-index.html)，在 `$HOME/.cargo/config` 中添加如下内容：

```bash
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```

可通过如下命令安装rustc的nightly版本（rCore要求用rustc的nightly版本），并把该版本设置为rustc的缺省版本

```bash
rustup install nightly
rustup default nightly
```

接下来安装一些Rust相关的软件包

```bash
rustup target add riscv64gc-unknown-none-elf
cargo install cargo-binutils
rustup component add llvm-tools-preview
rustup component add rust-src
```

## 3.Qemu 模拟器安装

```bash
sudo pacman -S qemu-emulators-full
```

随后即可在当前终端 `source ~/.bashrc` 更新系统路径，或者直接重启一个新的终端。此时我们可以确认 Qemu 的版本

```bash
qemu-system-riscv64 --version
qemu-riscv64 --version
```

## 4.GDB 调试支持

在 `os` 目录下 `make debug` 可以调试我们的内核，这需要安装终端复用工具 `tmux` ，还需要基于 riscv64 平台的 gdb 调试器 `riscv64-unknown-elf-gdb` 。该调试器包含在 riscv64 gcc 工具链中。

```bash
yay -S riscv64-unknown-elf-gdb
```

## 5.运行 rCore-Tutorial-v3

如果是在 Qemu 平台上运行，只需在`os`目录下 `make run` 即可。在内核加载完毕之后，可以看到目前可以用的 应用程序。 `usertests` 打包了其中的很大一部分，所以我们可以运行它，只需输入在终端中输入它的名字即可。运行后，可以先按下 `Ctrl+A` ，再按下 `X` 来退出 Qemu。