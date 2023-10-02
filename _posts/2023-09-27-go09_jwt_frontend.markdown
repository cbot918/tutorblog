---
layout: post
title:  (教學) jwt frontend
date:   2023-10-1 21:48:00 +0800
image:  02.jpg
tags:   Tutorial
---

昨天介紹 jwt token 的 backend 實作, 今天介紹 frontend

前端處理 jwt的方式就是, 接收到後端來的 token及使用者id資料, 將這兩筆資料存到localStorage裡面, 之後需要驗證的時候, 就從localStorage拿出來, 放到 headers 裡面, 打出去給 backend api驗證

但這邊 backend 會需要兩隻api, 所以在本節我們就順便簡單實作一下 /signup /signin (先不用資料庫, 把帳密存在記憶體裡)

[程式碼](https://github.com/cbot918/ithelp/tree/main/go-junior-30/jwt-frontend)

小提醒：這次篇幅會比較大, 因為包含了 api實作

因為要實作前端, 所以用 fiber 搭了一個簡易的 api
1. /signin
2. /signup
3. /post

### jwt backend

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"

	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/fiber/v2/middleware/cors"
)

const port = ":8886"

// 記憶體中的臨時資料庫 users
var userCount int
var users []User

// 臨時資料庫 posts
var postCount int
var posts []Post

func main() {
	api := fiber.New()

	api.Use(cors.New())
	api.Post("/signup", Signup)
	api.Post("/signin", Signin)
	api.Post("/post", Postt)

	if err := api.Listen(port); err != nil {
		log.Fatal(err)
	}
}

// handle request
func Signup(c *fiber.Ctx) error {
	// bodyparser 負責把 request body 存到 User{} 結構裡面
	user := User{}
	err := c.BodyParser(&user)
	if err != nil {
		return err
	}
	userCount += 1 // 這邊就是類似 auto increment 的功能 
	user.Id = userCount // 把資料加上一個序號
	users = append(users, user) // user 加到 users 裡
	jsonFormat, err := json.MarshalIndent(users, "", "  ")
	if err != nil {
		return err
	}
	fmt.Println(string(jsonFormat))
	return c.JSON(user) // 把加入的 user 資料 return 回去給使用者
}

func Signin(c *fiber.Ctx) error {
	user := User{}
	err := c.BodyParser(&user)
	if err != nil {
		return err
	}

	// 一開始登入flag false
	loginSuccess := false

	// 進臨時資料庫比對 先找到 email, 再比對 password
	for _, item := range users {
		if user.Email == item.Email {
			if user.Password == item.Password {
				loginSuccess = true // flag 設為 true
				user.Id = item.Id // 把 id 加到我們要用的 user資料裡
			}
		}
	}
	var token string
	if loginSuccess {
		// 產生 jwt token 給前端使用者
		token, err = genJWT(user.Id, user.Email)
		if err != nil {
			return err
		}
		fmt.Println(token)
	} else {
		return c.JSON(fiber.Map{"message": "invalid email or password"})
	}

	// 把userid 及 token 返回, 前端需要存到 localStorage
	return c.JSON(fiber.Map{
		"id":    user.Id,
		"token": token,
	})
}

func Postt(c *fiber.Ctx) error {
	// 將發文資料拿出來
	post := Post{}
	fmt.Println("in post")
	err := c.BodyParser(&post)
	if err != nil {
		fmt.Println(err.Error())
		return err
	}
	fmt.Println(post)
	token := c.Get("Authorization")
	fmt.Println("token: ")
	fmt.Println(token)

	// decode jwt 看 token secret 有沒有被竄改過
	claim, err := DecodeJWT(token)
	if err != nil {
		return c.JSON(fiber.Map{"message": "token invalid"})
	}

	// 這邊進資料庫看 id 是否存在, 存在則驗證通過
	authSuccess := false
	for _, item := range users {
		if item.Id == claim.Id {
			authSuccess = true
		}
	}

	// 這邊把 post 加入臨時資料庫 posts
	var resultPost Post
	if authSuccess {
		postCount += 1
		resultPost = Post{
			Id:       postCount,
			Title:    post.Title,
			Body:     post.Body,
			PostedBy: claim.Id,
		}
		posts = append(posts, resultPost)
	}

	return c.JSON(fiber.Map{
		"post_id":   resultPost.Id,
		"title":     resultPost.Title,
		"body":      resultPost.Body,
		"posted_by": resultPost.PostedBy,
	})
}

// types
type User struct {
	Id       int    `json:"id"`
	Email    string `json:"email"`
	Password string `json:"password"`
}

type Post struct {
	Id       int    `json:"post_id"`
	Title    string `json:"title"`
	Body     string `json:"body"`
	PostedBy int
}

```

### jwt frontend
```js
import { useState } from 'react'


function App() {

  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const [title, setTitle] = useState("")
  const [body, setBody] = useState("")

  function signupPost({email,password}){
    
    fetch("http://localhost:8886/signup",{
      method:"POST",
      headers:{"Content-Type":"application/json"},
      body:JSON.stringify({email,password})
    })
    .then(res=>res.json())
    .then(data=>{
      console.log(data)
    })
    .catch(err=>{
      console.log(err)
    })
  }

  function signinPost({email,password}){
    fetch("http://localhost:8886/signin",{
      method:"POST",
      headers:{"Content-Type":"application/json"},
      body:JSON.stringify({email,password})
    })
    .then(res=>res.json())
    .then(data=>{
      console.log(data)
      localStorage.setItem("id",data.id)
      localStorage.setItem("token",data.token)
    })
    .catch(err=>{
      console.log(err)
    })
  }

  function postPost({title,body}){
    const id = localStorage.getItem("id")
    const token = localStorage.getItem("token")
    fetch("http://localhost:8886/post",{
      method:"POST",
      headers:{
        "Content-Type":"application/json",
        "Authorization":token
      },
      body:JSON.stringify({id,title,body})
    })
    .then(res=>res.json())
    .then(data=>{
      console.log(data)
    })
    .catch(err=>{
      console.log(err)
    })
  }

  return (
    <>
      <div>
        註冊
        <input 
          type="text"
          placeholder="email"
          onChange={(e)=>{
            setEmail(e.target.value)
          }}
        />
        <input 
          type="text"
          placeholder="password"
          onChange={(e)=>{
            setPassword(e.target.value)
          }}
        />
        <input 
          type="button"
          value="submit"
          onClick={(e)=>{
            e.preventDefault()
            signupPost({email,password})
          }}
        />
      </div>
      <div>
        登入
        <input 
          type="text"
          placeholder="email"
          onChange={(e)=>{
            setEmail(e.target.value)
          }}
        />
        <input 
          type="text"
          placeholder="password"
          onChange={(e)=>{
            setPassword(e.target.value)
          }}
        />
        <input 
          type="button"
          value="submit"
          onClick={(e)=>{
            e.preventDefault()
            signinPost({email,password})
          }}
        />
      </div>
      <br />
      <div>
        發文
        <input 
          type="text"
          placeholder="title"
          onChange={(e)=>{
            setTitle(e.target.value)
          }}
        />
        <input 
          type="text"
          placeholder="body"
          onChange={(e)=>{
            setBody(e.target.value)
          }}
        />
        <input 
          type="button"
          value="submit"
          onClick={(e)=>{
            e.preventDefault()
            postPost({title,body})
          }}
        />
      </div>
    </>
  )
}

export default App

```