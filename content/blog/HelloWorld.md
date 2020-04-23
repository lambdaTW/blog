+++
title = "Hello Golang"
author = "lambda@lambda.tw"
categories = ["Golang"]
tags = ["Golang"]
date = "2018-07-04"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# Golang
這篇文章將從頭開始說起 Golang 的基本

## 安裝
Golang 在 archlinux 上面安裝很簡單下以下指令
```shell-script
sudo pacman -S go
```

## 設定 Golang 基本環境變數
```shell-script
# super 是我的 使用者名稱
export GOPATH=/home/super/go
export GOBIN=/home/super/go/bin
export PATH=$PATH:$GOBIN
```

## 編輯器
我大部分還是習慣在 [emacs](https://www.gnu.org/s/emacs/) 上開發，但是沒在使用 [vim](https://www.vim.org/) 或是 [emacs](https://www.gnu.org/s/emacs/) 的人還是建議使用 [vscode](https://code.visualstudio.com/) 比較方便
### 安裝 vscode
```shell-script
curl -O https://aur.archlinux.org/cgit/aur.git/snapshot/visual-studio-code-bin.tar.gz
tar -xvz -f visual-studio-code-bin.tar.gz
cd visual-studio-code-bin 
makepkg -sir .
```
### 設定 Golang
- 按下安裝 vscode 建議的 extension (選擇 `install`)

{{< img-post path="date" file="GolangInstallExtensions.jpg" alt="GolangInstallExtensions" type="center" >}}


- 重新載入 vscode (選擇 `reload`)

{{< img-post path="date" file="GolangVSCodeReload.jpg" alt="GolangVSCodeReload" type="center" >}}


## 寫下你的第一支程式
### 進入你的家目錄中的 go/src 資料夾 裡面建立你的第一支程式的目錄 `hello`
```shell-script
mkdir -p ~/go/src/hello
cd ~/go/src/hello
```
### 用 vscode 打開
```shell-script
code main.go
```
### 按下存檔 (ctl+s)，隨便打字
### 看到右下角叫你安裝套件把它全裝 (`install all`)


{{< img-post path="date" file="GolangInstallCommands.jpg" alt="GolangInstallCommands" type="center" >}}


### 等它一下然後重開

### 從新編輯 `main.go`

```golang
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello")
}
```

### 存檔 (ctl+s)

### 在 vscode 開啟終端機 (ctl+`) 並執行程式

```
go run main.go
```

# 備註

- 非 archlinux 的使用者可以在 golang 官方[下載](https://golang.org/dl/) golang
- 同上 vscode 也可以在官方[下載](https://code.visualstudio.com/download)
- 在 vscode 上面開啟終端機 View -> Integrated Terminal

