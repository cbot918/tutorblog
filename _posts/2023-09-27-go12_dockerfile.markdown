---
layout: post
title:  (教學) dockerfile
date:   2023-10-2 21:48:00 +0800
image:  02.jpg
tags:   Tutorial
---

docker 的第二篇， 來介紹 dockerfile, 就是自己做 image, 然後 run 成 container, 所以 docker 是一種很流行的部屬方式. 


## 1. 做個最簡單的 http server
[程式碼](https://github.com/cbot918/ithelp/tree/main/go-junior-30/docker/dockerweb)

專案架構
```
- main.go
- dockerfile
- ui
  - dist
  - src
  - package.json
```

先執行一下 go mod init web, 加入以下檔案

main.go
```go
package main

import (
	"fmt"
	"log"

	"github.com/gofiber/fiber/v2"
)

const (
	port      = ":3001"
	staticDir = "./ui/dist"
)

func main() {
	app := fiber.New()

	app.Static("/", staticDir)

	fmt.Printf("Server is listening on port %s\n", port)

	err := app.Listen(port)
	if err != nil {
		log.Fatalf("Error starting server: %v\n", err)
	}
}
```

dockerfile
```dockerfile
FROM golang:1.21.1-alpine3.17

WORKDIR /app

COPY . .

CMD ["go","run","."]
```

web ui
1. `npm create vite ` ( 名字ui, react, javascript )
2. `yarn` ( 安裝需要的依賴套件 )
3. `yarn run dev ` ( 先看一下畫面成功ok )
4. `yarn run build` ( build成功會有一個 dist 資料夾 )


材料準備好了, 來開始製作 image

```bash
docker build -t goweb .
```

來跑成 container
```bash
docker run -it -p 3001:3001 --name goweb goweb
```

最後就可以檢查結果啦, 瀏覽 localhost:3001, 應該會看到網頁畫面


### 後記

上面的範例, 其實很多值得吐槽的地方, 太肥, 不必要的檔案, 很多東西需要優化, 但覺得現階段先這樣,能動, 優化的東西再加就好, multi-stage build 也之後再來介紹好了, 後面我們繼續介紹 docker-compose