---
layout: post
title:  (教學) CLI Cobra
date:   2023-10-12 06:00:00 +0800
image:  02.jpg
tags:   Tutorial
---

最近自己也在做 cli tool, 熟悉了 cobra 發現算是好用(沒有其他語言做cli比較), 但自己體驗還不錯！來分享一下



## Install
這邊用 go install 安裝
```bash
go install github.com/spf13/cobra-cli@latest
```
`cobra-cli` 看一下有沒有安裝成功

## Getting-Started
init module
```bash
go mod init clii
```

init cobra
```bash
cobra-cli init
```

這樣會有以下檔案結構
```
cmd
  - root.go
- go.mod
- go.sum
- LICENSE
- main.go
```

其中 root.go 是指令的 root, 我們先新增一個指令 init

```
cobra-cli add init
```

在新增一個指令 fetch
```
cobra-cli add fetch
```

接著編譯一下, 我自己是慣用這樣的方法
說明: 把 clii 編譯出來之後, mv 到 /usr/local/bin 裡面, 這樣就可以直接使用
Makefile
```makefile
BIN=clii

.PHONY:inst build-win build-mac-intel build-mac-arm
.SILENT: inst
inst:
	go build .
	sudo mv $(BIN) /usr/local/bin 
```

接著 `make inst`

然後 `clii`

這樣就會有功能跳出來了

### 新增功能
我們目標是在 init 裡面加入新增一個 .env 檔案的功能

cmd/init.go
```go
/*
Copyright © 2023 NAME HERE <EMAIL ADDRESS>
*/
package cmd

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"
)

// initCmd represents the init command
var initCmd = &cobra.Command{
	Use:   "init",
	Short: "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	Run: func(cmd *cobra.Command, args []string) {

		if err := Init(); err != nil {
			fmt.Println("init func error")
			return
		}

	},
}

func init() {
	rootCmd.AddCommand(initCmd)

}

func Init() error {
	fd, err := os.Create(".env")
	if err != nil {
		return err
	}

	_, err = fd.Write([]byte("USER="))
	if err != nil {
		return err
	}
	return nil
}

```


cmd/fetch.go
```go
/*
Copyright © 2023 NAME HERE <EMAIL ADDRESS>
*/
package cmd

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"

	"github.com/spf13/cobra"
)

// fetchCmd represents the fetch command
var fetchCmd = &cobra.Command{
	Use:   "fetch",
	Short: "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	Run: func(cmd *cobra.Command, args []string) {
		url := "https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&current=temperature_2m,windspeed_10m&hourly=temperature_2m,relativehumidity_2m,windspeed_10m"
		body, err := fetch(url)
		if err != nil {
			fmt.Println("fetch failed")
			return
		}
		fmt.Println(string(body))

	},
}

func init() {
	rootCmd.AddCommand(fetchCmd)

}

func fetch(url string) ([]byte, error) {
	resp, err := http.Get(url)
	if err != nil {
		return nil, err
	}

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}

	return body, nil
}

func printJSON(v any) {
	// fmt.Println(v)
	json, err := json.MarshalIndent(v, "", "  ")
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(string(json))
}

```

執行 `make inst`

這樣就可以測試 clii fetch 了, 會打個get 去免費天氣開放資料, 然後回傳並印出來!

這樣30天就都結束了, 很高興可以參加這次的鐵人賽, 真的要有強大的意志力才能稱到30天, 希望明年還能夠開篇(許願看看), 也期許到時候的自己更進步!