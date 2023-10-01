---
layout: post
title:  (教學) jwt-token
date:   2023-10-1 21:48:00 +0800
image:  02.jpg
tags:   Tutorial
---

今天插播一下簡易的 jwt 的實做方式, 因為後面講 middleware 的地方會用到, 所以先簡略介紹

JWT Token 是一個 stateless 的 http 驗證方式, 常用在跨域的情況下, 每一個請求都是獨立的, 所以在使用者登入後, 會拿到一串伺服器做好的 jwt token, 這樣往後的請求只需要把 token 放在 header 裡面, 伺服器就知道這個 http 請求是誰送來的,解碼看看secret(jwt token 會設定一個 secretkey, 以防中間人竄改過的封包)對不對, 然後再去資料庫比對看一下有沒有這個使用者



jwt在後端實作上分成兩部份 ( 前端的 jwt 部份下一個章節補上)
1. 製作 jwt token (encode jwt)
2. 解碼 jwt token (decode jwt)

程式碼來說, 我們首先會需要一個 jwt claims ( 就是定義一下 jwt token 裡面要攜帶什麼資料, 這邊以 Email 跟 Id 為例), 以及一個定義好的 secret key 

### Jwt claims and secret
```go
type JwtClaims struct {
  // 這邊使用內建的 standardclaims 資料結構, 可以設置過期時間
	jwt.StandardClaims
  // 這邊是我們 PO文系統內, 會使用 email 以及 id 作為辨識
	Email string
	Id    int
}

const (
	secret = "12345" // secret 是自己定義好的字串, 編碼解碼需要用
	issuer = "test123"
)

```

### Encode JWT
```go
func genJWT(id int, email string) (string, error) {
	// 先把 id 跟 email 資訊放進去
  // email及id的資料來源是 /signin api client端會提供email, id是資料庫會產生
  claims := JwtClaims{
		Id:    id,
		Email: email,
		StandardClaims: jwt.StandardClaims{
			Issuer: issuer,
		},
	}

  // 產生 jwt token 的函式
  // 這邊使用 jwt.SigningMethodHS256 去做加密
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	// fmt.Printf("%#+v", token)


  // 將 token 使用我們提供的 secret 編碼成字串
	tokenString, err := token.SignedString([]byte(secret))
	if err != nil {
		return "", err
	}

	return tokenString, nil
}
```

## Decode JWT
```go
func DecodeJWT(target string) (*JwtClaims, error) {
  
  // 這邊是將 jwt 字串 做 parse
	token, err := jwt.ParseWithClaims(target, &JwtClaims{}, func(token *jwt.Token) (interface{}, error) {
    // 這邊的 secret 就是跟我們之前設定好的 secret 要是同一個, 不一樣的話就會報錯, 知道 jwt 可能被竄改過了
		return []byte(secret), nil
	})
	if err != nil {
		return nil, err
	}

  // 將 token 格式轉成 claims 格式
	claims, ok := token.Claims.(*JwtClaims)
	if ok && token.Valid {
		return claims, nil
	} else {
		return nil, err
	}

}
```

## Put All Together
```go
package main

import (
	"fmt"
	"log"

	"github.com/golang-jwt/jwt"
)

type JwtClaims struct {
	jwt.StandardClaims
	Email string
	Id    int
}

const (
	secret = "12345"
	issuer = "test123"
)

func genJWT(id int, email string) (string, error) {
	claims := JwtClaims{
		Id:    id,
		Email: email,
		StandardClaims: jwt.StandardClaims{
			Issuer: issuer,
		},
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	// fmt.Printf("%#+v", token)

	tokenString, err := token.SignedString([]byte(secret))
	if err != nil {
		return "", err
	}

	return tokenString, nil
}

func DecodeJWT(target string) (*JwtClaims, error) {
	token, err := jwt.ParseWithClaims(target, &JwtClaims{}, func(token *jwt.Token) (interface{}, error) {
		return []byte(secret), nil
	})
	if err != nil {
		return nil, err
	}

	claims, ok := token.Claims.(*JwtClaims)
	if ok && token.Valid {
		return claims, nil
	} else {
		return nil, err
	}

}

func main() {
	// generate a jwt token
	token, err := genJWT(1, "abc123@gmail.com")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("generated token: ")
	fmt.Println(token)

	// decode a jwt token
	claim, err := DecodeJWT(token)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("\n%#+v\n", claim)
	fmt.Println(claim.Id)
	fmt.Println(claim.Email)
}

```