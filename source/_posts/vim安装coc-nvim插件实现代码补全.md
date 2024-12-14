---
title: vim安装coc-nvim插件实现代码补全
date: 2024-10-06 14:10:59
tags: vim
---

## vim 插件管理器

[vim-plug](https://github.com/junegunn/vim-plug) Minimalist Vim Plugin Manager 

### 安装

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

### 使用

edit `~/.vimrc`

```vim
call plug#begin()

" List your plugins here
Plug 'tpope/vim-sensible'

call plug#end()
```

重启 vim, 即可执行以下命令

* `:PlugInstall` to install the plugins
* `:PlugUpdate` to install or update the plugins
* `:PlugDiff` to review the changes from the last update
* `:PlugClean` to remove plugins no longer in the list


## coc.nvim 插件

[coc.nvim](https://github.com/neoclide/coc.nvim) Make your Vim/Neovim as smart as VS Code

编辑 `~/.vimrc` 加入 vim-plug 插件列表

```vim
" Use release branch (recommended)
Plug 'neoclide/coc.nvim', {'branch': 'release'}

" Or build from source code by using npm
Plug 'neoclide/coc.nvim', {'branch': 'master', 'do': 'npm ci'}
```

重启 vim 然后执行 `:PlugInstall` 命令安装coc.nvim

安装完成后，再次编辑 `~/.vimrc` 加入默认配置参数(快捷键等) [链接](https://github.com/neoclide/coc.nvim#example-vim-configuration)

## c/c++ 补全插件

[coc-clangd](https://github.com/clangd/coc-clangd) clangd extension for coc.nvim 

vim 中执行 `:CocInstall coc-clangd` 安装插件，此插件用于连接下一小节安装的clangd服务

修改配置参数 `:CocConfig` ，配置参数也可参考[llvm.org](https://clangd.llvm.org/installation.html) 

```vim
{
  "clangd.arguments": ["--background-index", "--clang-tidy"],
  "clangd.fallbackFlags": ["-std=c++23"]
}
```

## 安装 clangd

[llvm.org](https://clangd.llvm.org/installation.html) 中 Editor plugins 小节

```bash
sudo apt install clangd
```

## 使用

vim新建main.cpp文件编写代码

Coc快捷键：

1. `Tab`切换补全 `Enter`确认
2. `gd` to jump to definition
3. `gr` for references
4. `gy` for type definition
5. `K`  for documentation
6. `<leader>rn` for renaming  `<leader>`默认为 `\` 按键
7. `if`, `ic` for func/class selection in visual mode
8. `<space>a` to list diagnostics
9. `[g` and `]g` to go prev/next in diagnostics
10. `Ctrl+o` 在普通模式下，按 Ctrl + o 可以返回到上一个光标位置
11. `Ctrl+i` 可以前进到下一个光标位置（即回到使用 Ctrl + o 返回的地方）

## 其他插件

1. auto-pairs 自动补括号
2. fzf 文件搜索
3. nerdtree 文件列表
4. lightline 好看的状态栏

```vim
call plug#begin()

Plug 'jiangmiao/auto-pairs'
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
"Plug 'voldikss/vim-floaterm'
Plug 'preservim/nerdtree', { 'on': 'NERDTreeToggle' }
Plug 'itchyny/lightline.vim'
Plug 'neoclide/coc.nvim', { 'branch': 'release' }

call plug#end()
```

