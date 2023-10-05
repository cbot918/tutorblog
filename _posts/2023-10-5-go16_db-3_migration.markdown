---
layout: post
title:  (教學) DB - migrate
date:   2023-10-6 00:20:00 +0800
image:  02.jpg
tags:   Tutorial
---

小提醒：發現越來越多指令要記了, 尤其是 migrate 的時候, 今天會偷渡個 Makefile 進來, 希望大家使用起來順利

用到的連結：
[migration](https://github.com/golang-migrate/migrate)

## [程式碼](https://github.com/cbot918/ithelp/tree/main/go-junior-30/db/3_migration)

上面我們知道怎麼開表之後, 循序介紹一下 db migration一下(雖然筆者比較喜歡native自己做, 但適當的工具還是要熟一下ㄒㄒ)

migration 就是把我們的 sql code 送到 database 裡面, 上一篇我們是用 SQL Editor 直接貼 code 去做, 也沒什麼毛病, 但如果我們在開發過程中, 會改動table的狀況的時候, go 有個 migration 的工具, 讓我們可以有 table 的版本紀錄

覺得真的繁瑣, 先把 Makefile script 放上來, 讓有點概念的大大節省時間, 如果不熟的朋友可以往下, 有每個步驟的解說

### Makefile put all together (方便觀看用)
Makefile
```makefile
DB_URL=postgres://postgres:12345@localhost:5435/blog?sslmode=disable

mig-add:
	migrate create -ext sql -dir db/migrations -seq $(name)

mig-up:
	migrate -path db/migrations -database $(DB_URL) up

mig-down:
	migrate -path db/migrations -database $(DB_URL) down

mig-up1:
	migrate -path db/migrations -database $(DB_URL) up 1

mig-down1:
	migrate -path db/migrations -database $(DB_URL) down 1
```

## Getting Started

### Install Migration Cli

我是用 go 去安裝, 這邊有個要注意的 tags 就是包含的 driver, 目前使用 postgres 就這樣安裝就ok, 如果是要使用 mysql 或其他 db的朋友, 安裝的時候要換一下-tas的內容, 請參考官方說明
```bash
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@$latest
```

其他系統的朋友請參考 [cli安裝頁](https://github.com/golang-migrate/migrate/tree/master/cmd/migrate)

### Migrate ADD
這邊我們創建 Makefile, 裡面放入如下 script
Makefile
```makefile
mig-add:
	migrate create -ext sql -dir db/migrations -seq $(name)
```
然後我們在 terminal 執行 `make mig-add name=ec`

解釋:
1. migrate create 是指會創建 xxx.sql 的檔案出來, 而且會有 up 跟 down兩種分別負責 setup 跟 teardown, 
2. -ext sql 就是指定好格式, -dir 就是指定 folder路徑
3. -seq 是 migrate 的命名, 這邊用到 makefile pass parameter 的技巧(4.解釋)
4. `make mig-add name=ec` 就是呼叫 mig-add, 然後把 name變數取代成 ec, 最後我們會得到 000001_ec.up.sql 及 000001_ec.down.sql


接著把 ec.sql 的內容分別貼到 000001_ec.up.sql 及 000001_ec.down.sql 內

000001_ec.up.sql
```sql
CREATE TABLE IF NOT EXISTS users(
  id serial PRIMARY KEY,
  name varchar(255) NOT NULL,
  email varchar(255) NOT NULL,
  password varchar(255) NOT NULL,
  discount float NOT NULL
);

CREATE TABLE IF NOT EXISTS orders(
  id serial PRIMARY KEY,
  order_by int NOT NULL,
  items int[] NOT NULL,
  total_price int NOT NULL,
  discount float NOT NULL,
  FOREIGN KEY (order_by) REFERENCES users(id)
);

CREATE TABLE IF NOT EXISTS items(
  id serial PRIMARY KEY,
  available int NOT NULL,
  name varchar(255) NOT NULL,
  vender varchar(255) NOT NULL,
  unit_price int NOT NULL
);
```
000001_ec.down.sql
```sql
ALTER TABLE IF EXISTS "orders" DROP CONSTRAINT IF EXISTS "orders_order_by_fkey";

DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS items;
DROP TABLE IF EXISTS users;
```


### Migrate UP

在 Makefile 裡面新增 mig-up

Makefile
```makefile
DB_URL=postgres://postgres:12345@localhost:5435/blog?sslmode=disable
mig-up:
	migrate -path db/migrations -database $(DB_URL) up
```
然後 terminal 執行 `make mig-up`, 接著去 Beekeeper 或 DBeaver 看一下, table 有沒有如預期建立起來


解釋：
1. -path 指定我們的 sql 檔案位置
2. -database $(DB_URL) 使用 makefile 的變數, DB_URL 裡面內容注意一下根據自己的設定調整


### 怎麼看 migrate 版本 以及解決 dirty migrate

去 GUI 裡面 refresh 一下, 我們會發現, 多了一張表 schema_migrations, 我們下語法 `select * from schema_migrations`, 會撈出我們目前 version 或 dirty, 如果 dirty 是 true 的話, 就會鎖住, 不讓我們 migrate 東西, 可以用 GUI界面點一下那個欄位, 手動把 dirty 改成 false, 然後再修 migrate

這是防呆機制, 意思是卡住我們不正確的更新, 提醒工程師注意, 不然 migrate 犯錯, 資料受損有時候會很嚴重QQ

### Migrate Down
在 Makefile 裡面新增 mig-down

Makefile
```makefile
mig-down:
	migrate -path db/migrations -database $(DB_URL) down
```

然後執行 `make mig-down`, 再去 GUI 確認有沒有如預期把表格清掉 (有時候只是畫面沒更新, 界面 refresh按鈕 按一下)

### Migrate Version Control
接下來新增一張 test table, 執行指令 `make mig-add name=test`

將以下貼到 000002_test.up.sql
```sql
CREATE TABLE test();
```

將以下貼到 000002_test.down.sql
```sql
DROP TABLE IF EXISTS test;
```

加入以下script 到 Makefile
```makefile
mig-up1:
	migrate -path db/migrations -database $(DB_URL) up 1

mig-down1:
	migrate -path db/migrations -database $(DB_URL) down 1
```
解釋: 就是往上升一版 或往下降一版

接著執行 `make mig-up1`, 再去 GUI看, 果然多一張表了, 撈一下 schema_migrations 的內容, 我們的 version 變成 2 了

如果這時候執行 `make mig-down1`, 就會發現 test 被清掉, version 變回1

這樣我們就可以輕鬆新增表格刪除表格, 然後有個版本紀錄控制了！


