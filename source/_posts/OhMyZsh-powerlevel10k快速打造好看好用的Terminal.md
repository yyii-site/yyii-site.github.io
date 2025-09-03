---
title: OhMyZsh_powerlevel10k快速打造好看好用的Terminal
date: 2024-10-26 16:02:00
tags: linux
---

[原帖](https://holychung.medium.com/%E5%88%86%E4%BA%AB-oh-my-zsh-powerlevel10k-%E5%BF%AB%E9%80%9F%E6%89%93%E9%80%A0%E5%A5%BD%E7%9C%8B%E5%A5%BD%E7%94%A8%E7%9A%84-command-line-%E7%92%B0%E5%A2%83-f66846117921)

## 前情提要

此篇是以 ubuntu 20.04 作為範例，同時也可以使用在 mac、windows WSL 當中，稍微會有一點不同的設定可以參考 [My_Terminal_Theme](https://github.com/Holychung/My_Terminal_Theme)

![最終成果](pic1.gif)

## 安裝、設定方式

總共有以下步驟：

* 安裝 zsh
* 安裝 oh-my-zsh
* 安裝 zsh theme: powerlevel10k
* 安裝 zsh 好用 plugins
* 安裝 powerline font
* 修改 powerlevel10k 設定
* 修改 terminal 的 color scheme
* 固定指令列在終端機底部

首先，如果 ubuntu 系統是剛灌完，建議利用下面兩步驟先更新一下，如果是舊玩家直接跳過這一步驟進入 zsh 基本安裝即可。

```bash
sudo apt-get update
sudo apt-get upgrade
```

順便把一些套件裝一下。

```bash
sudo apt install vim curl git
```

## 安裝 zsh

```bash
sudo apt install zsh
```

如果裝好了可以用這個指令查看。

```bash
cat /etc/shells
```

更換 login shell，要記得 logout 才會生效。

```bash
chsh -s $(which zsh)
```

## 安裝 oh-my-zsh

[ohmyzsh github](https://github.com/ohmyzsh/ohmyzsh)

```bash
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
sh install.sh
```

## 安裝 zsh theme: powerlevel10k

[powerlevel10k github](https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#meslo-nerd-font-patched-for-powerlevel10k)

Oh My Zsh 安裝 powerlevel10k。

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

安裝好後要修改 zsh 設定檔 .zshrc，先把主題換成剛剛安裝的 powerlevel10k

```vim
ZSH_THEME="powerlevel10k/powerlevel10k"
```

然後下這個指令。

```bash
source ~/.zshrc
```

可以去對 prompt 做基本的設定，包含 prompt style 等。

```bash
p10k configure
```

## 安裝字型

Powerlevel10k 推薦使用 Meslo Nerd Font。

* [MesloLGS NF Regular.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
* [MesloLGS NF Bold.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
* [MesloLGS NF Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
* [MesloLGS NF Bold Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

下載好後，修改 GNOME Terminal (the default Ubuntu terminal) 步驟如下。
`Open Terminal` → `Preferences` and click on the selected profile under Profiles. Check Custom font under `Text Appearance` and select `MesloLGS NF Regular`.

如果是 xfce4 双击字体文件没有安装选项时，需安装[font-manager](https://askmeaboutlinux.com/2023/10/04/how-to-add-fonts-on-the-xfce-desktop-in-linux/) 

```bash
sudo apt-get install thunar-font-manager
font-manager
```

![Terminal Settings](TerminalSettings.jpg)

## 安裝 zsh 好用 plugins

* [zsh-completions](https://github.com/zsh-users/zsh-completions)
* [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
* [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

裝好後修改 .zshrc 把 plugins 加到設定檔。

```vim
plugins=(
git
zsh-completions 
zsh-autosuggestions 
zsh-syntax-highlighting
)
```

立即生效。

```bash
source ~/.zshrc
```

## 修改 powerlevel10k 設定

設定檔 `.p10k.zsh`，修改完要下 `source ~/.p10k.zsh`。

這邊我只簡單介紹兩個我有修改的東西，其他也是差不多改法，可以參考 [powerlevel10k](https://github.com/romkatv/powerlevel10k) 有很詳細的說明。

修改 prompt 資料夾路徑如果太長超過過一個長度，中間的資料夾名稱會變成縮寫如下圖，這邊我把它改成大於 20 個字，不過可以視自己螢幕大小修改。

`typeset -g POWERLEVEL9K_DIR_MAX_LENGTH=20`

修改 prompt 當前目錄的顏色，這邊可以參考 directory-is-difficult-to-see-in-prompt-when-using-rainbow-style 有很詳細的說明，底下是我自己更改的版本還要配合修改 terminal 的 color scheme，也可以依照自己喜好修改。

```
# Current directory background color.
typeset -g POWERLEVEL9K_DIR_BACKGROUND=4
# Default current directory foreground color.
typeset -g POWERLEVEL9K_DIR_FOREGROUND=0
# If directory is too long, shorten some of its segments to the shortest possible unique
# prefix. The shortened directory can be tab-completed to the original.
typeset -g POWERLEVEL9K_SHORTEN_STRATEGY=truncate_to_unique
# Replace removed segment suffixes with this symbol.
typeset -g POWERLEVEL9K_SHORTEN_DELIMITER=
# Color of the shortened directory segments.
typeset -g POWERLEVEL9K_DIR_SHORTENED_FOREGROUND=232
# Color of the anchor directory segments. Anchor segments are never shortened. The first
# segment is always an anchor.
typeset -g POWERLEVEL9K_DIR_ANCHOR_FOREGROUND=232
# Display anchor directory segments in bold.
typeset -g POWERLEVEL9K_DIR_ANCHOR_BOLD=false
```

## 修改 terminal 的 color scheme

這邊提供我自己改的顏色，有參考 Windows Terminal Themes 當中的 Brogrammer，還有 Mac 的 terminal 進行修改。

```
{
  "black": "#1f1f1f",
  "red": "#f81118",
  "green": "#85D29A",
  "yellow": "#ecba0f",
  "blue": "#5C97CF",
  "purple": "#4e5ab7",
  "cyan": "#1081d6",
  "white": "#ffffff",
  "brightBlack": "#d6dbe5",
  "brightRed": "#de352e",
  "brightGreen": "#1dd361",
  "brightYellow": "#f3bd09",
  "brightBlue": "#1081d6",
  "brightPurple": "#5350b9",
  "brightCyan": "#0f7ddb",
  "brightWhite": "#ffffff",

  "background": "#2D2D2D",
  "foreground": "#d6dbe5"
}
```

## 固定指令列在終端機底部

可以在 .zshrc 當中加入這行，把命令列固定在最底下，視線就不用一直改動，我自己是很喜歡這個功能，不過這個有時候會有一些時候還是會跑掉，e.g. 下 clear 指令的時候，期待有人能找到更好的方法XD

```
# Fix prompt at the bottom of the terminal window
printf '\n%.0s' {1..100}
```

## 最後

如果有看到哪個地方可以修改的或是有不錯的套件歡迎告訴我，一起打造一個高效率的開發環境。

歡迎參考 [My_Terminal_Theme](https://github.com/Holychung/My_Terminal_Theme) 當中的配置，也可以找到 Mac、WSL 的版本，都是可行的。

[Mac 設定](https://github.com/Holychung/My_Terminal_Theme/tree/main/mac)

[Windows WSL 設定](https://github.com/Holychung/My_Terminal_Theme/tree/main/WSL)

[作者Blog](https://holychung.github.io/)
