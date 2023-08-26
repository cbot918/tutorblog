---
layout: post
title:  (教學) go01_helloworld
date:   2023-08-26 23:11:00 +0800
image:  02.jpg
tags:   Tutorial
---

因為自己新手期學得很辛苦, 想做新手教學滿久了, 希望整理出來的資訊可以真正意義上的好吸收, 幫助到想學習寫程式的新人

## 解決問題
從0到 Golang 第一支程式 Hello World

## 小提醒
1. 比較適合對程式興趣比較高且以自學為主的朋友
2. 稍微不適合把學寫程式單純當成工作技能, 因為可能有其他更好的選擇(js/java/C#/php/python)

## 語言選擇
新人朋友第一步面臨的問題就是語言選擇, 自己會選擇Golang作為新手教學的原因如下
1. 缺點：Golang初階工作幾乎沒有, 因此教學的方向是讓新手快速搞懂程式設計, 之後可以快速上手第二語言, 如果對自己學習比較沒自信或需要短時間內快速找到工作的朋友, Golang會不太適合
2. 強行別編譯式語言, 語法關鍵字少, 內建的標準函式庫, 語言特性容易做解底層觀念教學
3. 自己正在設計一個程式的線上遊戲教學, 伺服器需要比較節省資源, Golang是目前考慮起來符合的語言
4. Golang類似Python有方便快速學習CS概念的特性, 這對自己學習上幫助很大, 因此也是理由之一


## 筆者環境
1. Host OS: [Linux Ubuntu 20.04](https://ubuntu.com/download)
2. IDE: VScode
3. 語言: Golang / js

## Getting Started

## 1. 開發環境建制
這邊會有環境上的差別
1. 如果是windows, 建議花點時間設定WSL2, 確保是在Linux(Ubuntu)下做開發
2. 如果是MacOS, 確保自己有Homebrew套件管理工具, 並且update過
3. 如果是Linux, 也先用套件管理工具(Ubuntu Linux為例就是 sudo apt update)

### 1.0 安裝vscode
- 可以照著[官網](https://code.visualstudio.com/download)步驟安裝, 其中 windows直接執行就會有視窗可以安裝, mac要注意一下自己電腦是 intel晶片還是arm晶片不要選錯. linux的話注意一下版本, Ubuntu的話是.deb, 下載下來後直接滑鼠點兩下安裝
- 安裝一下golang的 extension, 可以在左側market place 搜尋 golang, 找下載數最高的, 安裝. 然後右下角可能會跳出一不少golang安裝extension的提示, 就一直壓install就對了, 重點就是看到有個 golps的東西裝好, 非常好用

### 1.1 安裝golang (不好意思這邊先只提供linux的版本, 未來再補上, google會有很多安裝教學供參)
- 跟著 https://go.dev/doc/install 的指示
- 或是跟著以下的步驟
```bash
sudo apt install curl make -y # 安裝 curl 跟 make
curl -OL https://go.dev/dl/go1.21.0.linux-amd64.tar.gz # 下載 golang
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz # 把舊的go移除(如果有的話) 把新的解壓縮到/usr/local目錄
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc # 這邊要把/usr/local/go/bin 加入環境變數, 之後才抓的到執行檔
source ~/.bashrc # 更新一下環境變數
go version # 預期會看到go的說明跳出來
```

### 1.2 開專案資料夾, 並且init
開個資料夾, 並用vscode開啟
```bash
# 先開啟一個 terminal 黑視窗
mkdir -p ~/coding/hello world # 在 ~ 開個 coding資料夾, 裡面再開個hello world資料夾
cd ~/coding/hello world # 把位置移動到hello world資料夾裡面
code . # code是vscode的執行檔, . 是指說當前資料夾, 所以這個指令是用vscode打開當前資料夾
# 如果 code . 失敗的話, 表示需要裝一下 code這個東西, 那就另外自己先開啟 vscode, 按住 ctrl+shift+p 打開面板, 輸入 shell code 安裝, 安裝完之後, 就能用code指令了
```
新增go.mod檔案
```bash
go mod init helloworld
# 會多一個go.mod
go mod tidy
# 會多一個go.sum
```
這個檔案是讓go編譯器(就是執行golang的工具)知道當前目錄有個golang專案, 這樣可以方便編譯器做靜態分析, 套件管理之類的, 所以算是必備動作
go.mod是管理當前專案路徑
go.sum是管理套件
目前細節大概知道就好, 主要是程式能動就ok

### 1.3 Hello World
寫hello world程式碼
```bash
touch main.go # 指令新增了 main.go
code main.go # 一個懶人方式, 也可以在vscode左側用滑鼠點main.go就好
```
在 main.go內新增以下
```golang
package main // 告訴編譯器這個檔案這是程式進入點, 一個資料夾內只能有一個main(稍後詳述)

import (   // 引入fmt模組 (format print 就是用來顯示字串的)
  "fmt"
)

func main(){  // 程式進入點的函式
  fmt.Println("hello world") // 印出 hello world
}
```

執行程式碼
先打開小黑窗, ctrl + ~ (鍵盤左上角), 可以在下方新增一個terminal
輸入
```
go run .
```
說明：使用go編譯器的run指令 .是代表當前資料夾, 編譯器會去抓 package main 裡面的 main function出來跑, 此時應該會看到小黑視窗跑出 hello world的字樣, 代表成功, 如果程式沒有成功, 有問題, 歡迎寄信聯絡我, 幫忙排除


以上程式碼假如可以執行, 但對新手來說應該還是有很多不懂的地方, 因此以下簡述程式的基本觀念
1. package main 這個先背起來用
2. import ("fmt") 理論上不用自己輸入, 如果有成功安裝 golps, 那我們在下面打 fmt.Println("hello world")然後存檔(ctrl+s)後, vscode很貼心會幫我們引入
3. func 是關鍵字 function的意思, main是 function 的名稱, ()括弧是代表要傳入的參數, {} 大括弧代表一個程式碼區塊, fmt是format print的意思, 這邊有物件導向觀念, 之後再詳述, 先會用就好. println是指 print line的意思, 所以印完會跳下一行(也代表會有函式印完不會跳下一行, 以後再介紹)
4. function是程式設計一個很重要的單位, 概念就是把程式碼放到一個袋子裡面備用的感覺(定應的func是不會執行, 但main預設是會執行, main是比較特別的function), 給他一個名稱, 未來要使用的時候呼叫他


## 後記
雖然盡量詳細紀錄, 但新手學程式通常會一直遇到問題, 所以歡迎email(yale918@gmail.com)給我, 可以一起討論也可以加加我的工程師line群