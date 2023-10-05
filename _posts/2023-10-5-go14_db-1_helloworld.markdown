---
layout: post
title:  (教學) DB - helloworld
date:   2023-10-2 21:48:00 +0800
image:  02.jpg
tags:   Tutorial
---

開始介紹 database, 重點項目篇幅比較多
1. hello world
2. schema
3. migration
3. sql
4. sqlc
5. crud ( 以 ec service 為例 )
6. wal
7. db backup

### [程式碼](https://github.com/cbot918/ithelp/blob/main/go-junior-30/db/connect-instance/main.go)

### 用到的連結:
1. [postgres docker](https://hub.docker.com/_/postgres)
2. [go postgres driver](https://github.com/lib/pq)
3. [go sql package](https://pkg.go.dev/database/sql)

## 起 instance
```bash
docker run -it --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=12345 -e POSTGRES_DB=test postgres
```
參數解說:
- -it: 持續運行
- --name: 指定 containe_name
- -p: 把 container 內的5432 bind 到 local 的 5432 (又稱作把port開出來, 讓我們可以直接連線)
- -e POSTGRES_PASSWORD:  設定密碼, 需要有密碼才能連線
- -e POSTGRES_DB=test: 創建一個 test db (方便)

## cli 去看一下 instance
```bash
docker exec -it postgres bash # 進到 docker 內
psql -U postgres -W  # 密碼 12345, 會進到 postgres內
\l # 看有哪些資料庫, 應該會看到 test
\c test # 切到 test 資料庫內
\dt # 看有哪些 table, 此時應該沒有
```
檢查沒問題, 準備 golang 來連線

## Hello World (ping db)
main.go
```go
package main

import (
	"database/sql"
	"fmt"
	"log"

  // 這是 go 特別的慣例, driver 要用底線引入, 就可以使用
	_ "github.com/lib/pq"
)

// user: postgres
// password: 12345
// host: localhost
// port: 5432
// db: test
const dsn = "postgresql://postgres:12345@localhost:5432/test?sslmode=disable"

func main() {
	db, err := sql.Open("postgres", dsn)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(db)

  // ping 就是確認一下 db 狀態的官方提供 api
	err = db.Ping()
	if err != nil {
		log.Fatal(err)
		return
	}
	fmt.Println("ping db success!")

  // 這邊涉及 sql 語法, 後面的文章再介紹
  res, err := db.Exec("CREATE TABLE IF NOT EXISTS users(id SERIAL PRIMARY KEY, email text, password text)")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(res)
  
}

```
直接執行, 應該會看到 ping db success,  然後再回去剛剛的 cli界面, \dt,  應該會看到 users table了！

## DB GUI

這邊推薦 

DBMS: [DBeaver](https://dbeaver.io/download/) 

SQLEditor: [Beekeeper CE](https://github.com/beekeeper-studio/beekeeper-studio)

DBeaver 是全免費開源的資料庫管理工具, 功能全面, 支援的資料庫非常多 (冷門的 cockroachdb 或是最近想用 cassandra 也都有辦法使用)

Keekeeper 我是用免費社群版, 軟體響應快速, table 點擊就會列出資料, 輕巧方便, 個人很喜歡

至於 GUI 怎麼連線進去, 網路上應該超多教學, 可以搜尋一下, 如果真的連不到, 再請留言我再介紹一篇