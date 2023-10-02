---
layout: post
title:  (教學) router-group 和 middleware
date:   2023-10-1 21:49:00 +0800
image:  02.jpg
tags:   Tutorial
---

今天接著介紹, groupRouter 以及 middleware, 筆者用一個 註冊跟驗證的 PO文系統, 來解釋一下 group router 跟 middleware

假設我們目前有 /signup, /signin 兩隻 api 負責使用者註冊以及登入. 在登入後我們會在後端生成 jwt token 回傳給前端！

接著使用者會呼叫 /allpost 去撈出所有的PO文, 然後使用者會使用 /post 去做 PO 文的動作 ,這時候 /post 就需要做驗證！

小弟業界經驗不多, 純粹按照運作邏輯來規劃, 如果有講得不夠完善的再請前輩不吝指點！

## 上 code
### main.go
```go
func main() {
	app := fiber.New()

  // 這是之前就有的處理 cors 的 middleware
  // 其實 allow cors 自己手寫不難, 之後篇幅有空的話再講解一下
	app.Use(cors.New())
  // logger 負責把打進來的 request 都做個 log 在螢幕上
	app.Use(logger.New())
  // recover 是當 server panic 的時候會自動恢復
	app.Use(recover.New())

  // 登入跟註冊不用另外加
	app.Post("/signup", Signup)
	app.Post("/signin", Signin)

  // 這邊做一個 router group, 從 /api 進來的路徑, 都會經過 RequireAuth 這個 middleware, 定義寫在 middleware.go 裡面
	api := app.Group("/api", RequireAuth())
	api.Post("/post", Postt)

	if err := app.Listen(port); err != nil {
		log.Fatal(err)
	}
}
```

新增一個 middleware.go的檔案
### middleware.go
middleware 也是一個 fiber.Handler, 只不過會有個 call c.Next()的動作, c.Next()之後會執行哪些動作, 資料結構是紀錄在 router group 裡面(go 的實作, 其他語言可能不一樣)
```go
func RequireAuth() fiber.Handler {
  return func(c *fiber.Ctx) error {
		token := c.Get("Authorization")
		_, err := DecodeJWT(token)
		if err != nil {
			fmt.Println("422 in here")
			return c.Status(422).JSON(fiber.Map{"message": "token is invalid, please check your token"})
		}
		return c.Next()
	}
}
```
