---
layout: post
title:  (教學) DB - API CRUD
date:   2023-10-6 04:00:00 +0800
image:  02.jpg
tags:   Tutorial
---

延續[上一篇的程式碼](https://github.com/cbot918/ithelp/tree/main/go-junior-30/db/5_sqlc)今天要來CRUD了, 意思是說要搭上 HTTP API 實作功能了, 順便實作一下簡單的三層式架構以及讀入環境變數吧, 可能份量會比較重, 可以多練習幾遍唷！

### [程式碼]()

## Getting Started

專案結構會長這樣
```
- db
  - migrations
  - query
  - sqlc
- internal
  - controller
  - service
  api.go
  util.go
main.go
.env
Makefile
sqlc.yaml
```

### 設定檔
.env
```
DB_DRIVER="postgres"
DB_URL="postgres://postgres:12345@localhost:5435/blog?sslmode=disable"
HOST="localhost:3003"
```

### Config
internal/util.go
```go
package internal

import (
	"os"

	"github.com/joho/godotenv"
)

type Config struct {
	DB_DRIVER string
	DB_URL    string
	HOST      string
}

func NewConfig() (*Config, error) {

	err := godotenv.Load()
	if err != nil {
		return nil, err
	}

	return &Config{
		DB_DRIVER: os.Getenv("DB_DRIVER"),
		DB_URL:    os.Getenv("DB_URL"),
		HOST:      os.Getenv("HOST"),
	}, nil
}
```

插撥個開發熱更新神器 [gowatch](https://github.com/silenceper/gowatch), 安裝後, 直接在目錄裡 gowatch下去， 存檔就熱更新, 非常好用, 但記得把編譯出來的 binary file 加入 .gitignore唷, 不然會影響 推code到 github的速度！

## 三層式架構

### Repository 資料庫操作層
這一層我們就是 sqlc

### Service 業務邏輯層
internal/service/service.go
```go
package service

import (
	db "6/db/sqlc"
	"context"
	"database/sql"
	"fmt"
)

type Service struct {
	Q *db.Queries
}

func NewService(q *db.Queries) *Service {
	return &Service{Q: q}
}

func (s *Service) CreateUser(user db.CreateUserParams) (*db.User, error) {

	resUser, err := s.Q.CreateUser(context.Background(), user)
	if err != nil {
		return nil, err
	}

	return &resUser, err

}

func (s *Service) GetUser(id int32) (*db.User, error) {
	resUser, err := s.Q.GetUser(context.Background(), id)
	if err != nil {
		return nil, err
	}
	return &resUser, nil
}

func (s *Service) DeleteUser(id int32) error {

	_, err := s.Q.GetUser(context.Background(), id)
	if err != nil {
		if err == sql.ErrNoRows {
			return fmt.Errorf("user id doesn't exists")
		}
		return err
	}

	err = s.Q.DeleteUser(context.Background(), id)
	if err != nil {
		return err
	}
	return nil
}

```

#### Controller 控制層(就是處理HTTP請求層)
internal/controller/controller.go
```go
package controller

import (
	db "6/db/sqlc"
	"6/internal/service"
	"fmt"
	"strconv"

	"github.com/gofiber/fiber/v2"
)

type Controller struct {
	S service.Service
}

func NewController(q *db.Queries) *Controller {
	return &Controller{S: *service.NewService(q)}
}

func (c *Controller) Ping(ctx *fiber.Ctx) error {

	return ctx.JSON(fiber.Map{"message": "pong"})

}

func (c *Controller) CreateUser(ctx *fiber.Ctx) error {

	userReq := db.CreateUserParams{}

	err := ctx.BodyParser(&userReq)
	if err != nil {
		return ctx.Status(422).SendString("request body error")
	}

	fmt.Printf("%#+v", userReq)

	resUser, err := c.S.CreateUser(userReq)
	if err != nil {
		return err
	}
	return ctx.JSON(resUser)
}

func (c *Controller) GetUser(ctx *fiber.Ctx) error {
	idStr := ctx.Query("id")
	id64, err := strconv.ParseInt(idStr, 10, 32)
	if err != nil {
		return err
	}
	id32 := int32(id64)

	retUser, err := c.S.GetUser(id32)
	if err != nil {
		return err
	}

	return ctx.JSON(retUser)
}

func (c *Controller) DeleteUser(ctx *fiber.Ctx) error {
	idStr := ctx.Params("id")
	id64, err := strconv.ParseInt(idStr, 10, 32)
	if err != nil {
		return err
	}
	id32 := int32(id64)

	err = c.S.DeleteUser(id32)
	if err != nil {
		return ctx.Status(422).JSON(fiber.Map{"error": err.Error()})
	}

	return ctx.JSON(fiber.Map{"message": "delete user success"})
}


```

三層式定義好, 來接著做 api.go

### API 路由本體
api.go
```go
package internal

import (
	db "6/db/sqlc"
	"6/internal/controller"

	"github.com/gofiber/fiber/v2"
)

type API struct {
	APP *fiber.App
	C   controller.Controller
}

func NewAPI(app *fiber.App, q *db.Queries) *API {

	a := new(API)

	a.C = *controller.NewController(q)
	a.APP = app

	a.APP.Get("/ping", a.C.Ping)

	a.APP.Get("/getuser", a.C.GetUser)
	a.APP.Post("/createuser", a.C.CreateUser)
	a.APP.Delete("/deleteuser/:id", a.C.DeleteUser)
	return a
}

```

這邊先定義了一個 API 類別, NewAPI  constructor, 我們可以看到接收 sqlc 的 queries 當作參數進來, 然後我們pass到 controller, 然後 pass 到 service, 再 pass 到 repository 層, 此為依賴注入, 我們跟資料庫互動, 集中在 repository 層處理！

接著我們定義 function member Ping, 要來做測試用的!

### 程式入口
main.go
```go
package main

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"log"

	db "6/db/sqlc"

	"6/internal"

	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/fiber/v2/middleware/logger"
	_ "github.com/lib/pq"
)

func main() {

	cfg, err := internal.NewConfig()
	if err != nil {
		log.Fatal(err)
	}

	conn, err := newConn(cfg.DB_DRIVER, cfg.DB_URL)
	if err != nil {
		log.Fatal(err)
	}

	app := fiber.New()

	app.Use(logger.New())

	query := db.New(conn)

	api := internal.NewAPI(app, query)

	err = api.APP.Listen(cfg.HOST)
	if err != nil {
		log.Fatal(err)
	}

}

// create connection
func newConn(driver string, dsn string) (*sql.DB, error) {
	conn, err := sql.Open(driver, dsn)
	if err != nil {
		return nil, err
	}

	err = conn.Ping()
	if err != nil {
		return nil, err
	}
	fmt.Println("postgres good")
	return conn, nil
}

// 封個 printJson 好了
func printJson(v any) {
	json, err := json.MarshalIndent(v, "", "  ")
	if err != nil {
		fmt.Println("json marshal failed")
	}
	fmt.Println(string(json))
}


```


### API測試
POST /createuser
```bash
curl --location 'http://localhost:3003/createuser' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name":"crudtest",
    "email":"crudtest@gmail.com",
    "password":"12345",
    "discount":0.25
}'
```

GET /getuser
```bash
curl --location 'http://localhost:3003/getuser?id=1'
```

DELETE /deleteuser/:id
```bash
curl --location --request DELETE 'http://localhost:3003/deleteuser/1'
```