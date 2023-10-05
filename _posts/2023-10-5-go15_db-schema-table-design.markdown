---
layout: post
title:  (教學) DB - schema-table-design
date:   2023-10-5 21:20:00 +0800
image:  02.jpg
tags:   Tutorial
---

## 基本介紹

小提醒：本身業務邏輯經驗不多, 設計上會有很多地方可以優化, 會以能動就好的方式進行介紹, 資深前輩歡迎略過xD

資料庫可以用 Excel 表格去做想像, 是由最上面一 row (橫的) metadata, 以及下方的資料組成

我們先以一張 Excel 表格來管理整間公司客戶的例子來想像, 如果公司有1000個客戶, 那 Excel表格就有 1000 row 客戶資料, 我們想要做搜尋的時候, 就需要跑全表掃描

資料庫是由作業系統 + Database Application 組成, 我們將資料本身儲存在硬碟上, 硬碟I/O很慢(跟硬碟的物理結構有關), 速度跟記憶體存取是完全不同數量級的慢, 所以關聯式資料庫如MySQL, Postgresql 底層資料結構在做設計的時候, 都會以越少的硬碟I/O來做考量, CS領域的研究人員發現 B+ Tree 的這個索引結構可以讓硬碟I/O的次數(也就是樹高)降到最低, 如此解決了第一個效能問題.

我們面臨第二個問題就是, 全表掃描. 一間網路軟體公司, 客戶/產品/定單...等等族繁不及備載的資料需要存起來, 如果都用類似 Excel 的結構, 一張表紀錄全部東西, 每次搜尋就做全表掃描, 那效能肯定跟很悲慘, 因此「關聯式資料庫」的方法就被提出來. 關聯式資料庫的核心 Primary Key (後面稱PK) 保證每一筆資料有個唯一的ID, 所以我們指定某一個ID, 不會找到相同的兩筆資料, Foreign Key (後面稱FK) 提供了可以把不同欄位拆開的功能, 舉例來說, 客戶放在一個 Table(Users), 商品放在一個 Table(Item), 訂單放在一個 Table(Order), 我們可以藉由 Order表格裡面的FK, 連結到 User 表格, Item表格裡面的FK, 連結到 Order 表格 如此我們就可以用 SQL 的 Join 語法, 查詢出某個 User 訂購了哪些產品！

以上兩點就是關聯式資料庫入門核心概念

接下來介紹如何拆分表格, 就需要知道資料庫「正規化」的概念. 資料庫正規化大概的思想就是, 同一張表, 盡量不要有紀錄重複資料的欄位, 如果會有, 就可以拆出去獨立一張表格, 

## 商品訂購範例

我們有間小電商, 會有使用者註冊, 購買會員, 訂購商品, 我們希望用Excel表格來管理, 然後庫存也是寫excel的 script 去核對, 接電話/簡訊靠人工 booking 訂購記錄下來 

### 單表
| id | name | email          | password | discount | order_date | order_item | quantity | item_name | item_vendor |
| -- | ---- | -------------- | -------- | -------- | ---------- | ---------- | -------- | --------- | ----------- |
| 1  | yale | yale@gmail.com | 12345    | 1        |            |            |          |           |             |
| 2  | yale | yale@gmail.com | 12345    | 0.85     |            |            |          |           |             |
| 3  | yale | yale@gmail.com | 12345    | 0.85     | 9/15       | a001       | 1        | 馬克杯(紅)    | ikea        |
| 4  | yale | yale@gmail.com | 12345    | 0.85     | 9/15       | a002       | 2        | 馬克杯(藍)    | ikea        |
| 5  | yale | yale@gmail.com | 12345    | 0.85     | 9/15       | b001       | 1        | 巧克力泡芙     | imei        |
| 6  | yale | yale@gmail.com | 12345    | 0.85     | 9/15       | b002       | 1        | 草莓泡芙      | imei        |
|    | node | node@gmail.com | 12345    | 1        |            |            |          |           |             |
|    | node | node@gmail.com | 12345    | 0.9      | 9/18       |            |          |           |             |
| 7  | node | node@gmail.com | 12345    | 0.9      | 9/18       | b001       | 4        | 巧克力泡芙     | imei        |
| 8  | hi   | hi@gmail.com   | 12345    | 1        |            |            |          |           |             |
|    | hi   | hi@gmail.com   | 12345    | 0.85     |            |            |          |           |             |
| 9  | hi   | hi@gmail.com   | 12345    | 0.85     | 9/18       | b001       | 5        | 巧克力泡芙     | imei        |
| 10 | hi   | hi@gmail.com   | 12345    | 0.85     | 9/18       | b003       | 10       | 巧克力牛奶     | imei        |
| 11 | yale | yale@gmail.com | 12345    | 1        | 10/16      | b001       | 3        | 巧克力泡芙     | imei        |
| 12 | yale | yale@gmail.com | 12345    | 1        | 10/17      | b003       | 5        | 巧克力牛奶     | imei        |

如果我們用單個資料表來管理公司會員註冊及訂單, 大概就是上面這種情形, 註冊時會紀錄1 row, 購買vip會員會紀錄1row, 接著是訂購會紀錄, 依照品項會紀錄好幾row (這邊看人設計), 大量欄位空著或重複, 能提供的功能也不多, 搜尋的時候需要跑整張表, 如果有1000個會員, 訂單品項有5000個, 這樣搜尋起來會很恐怖, 更何況還有物流資訊, 退貨資訊, 客戶修改, 金流串接資料, 等等一堆欄位還沒列上來, 最重要的, 沒辦法用上B Tree 索引技術, 所以我們來試著把表格正規化拆分

### 表格拆分設計成關聯式
業務增長上去, 希望可以數位化自動化管理, 不要有一堆人工, 所以使用關聯式資料庫(本例會使用postgres), 並且最後會有網頁訂購界面

users
| id(PK) | name | email          | password | discount |
| -- | ---- | -------------- | -------- | -------- |
| 1  | yale | yale@gmail.com | 12345    | 0.85     |
| 2  | node | node@gmail.com | 12345    | 0.9      |
| 3  | hi   | hi@gmail.com   | 12345    | 0.85     |

items
| id(PK)   | available | name   | vendor | unit_price |
| ---- | --------- | ------ | ------ | ---------- |
| 1 | 50        | 馬克杯(紅) | ikea   | 100        |
| 2 | 50        | 馬克杯(藍) | ikea   | 100        |
| 3 | 50        | 巧克力泡芙  | imei   | 40         |
| 4 | 50        | 草莓泡芙   | imei   | 40         |
| 5 | 50        | 巧克力牛奶  | imei   | 20         |

orders
| id(PK) | order_by(FK to users.id) | items                                 | total_price | discount |
| -- | -------- | ------------------------------------- | ----------- | -------- |
| 1  | 1        | [1,1,2,2,3,1,4,1] | 323         | 0.85     |
| 2  | 2        | [3,4]                            | 144         | 0.9      |
| 3  | 3        | [3,5,5,10]                  | 340         | 0.85     |
| 4  | 1        | [3,3,5,5]                   | 220         | 1        |

拆成分開的三張表, 如果需要用到關聯式的索引技術, 那就必須要有 PK, 然後目前設計上把 訂單的內容設計成一個陣列, 想說到時候在程式碼裡面再另外處理 (其實是筆者沒有設計過這種的經驗xD), 所以目前沒有 FK連到 item. 值得一提的是, 在 order 表格裡面, 餘冗了一個 discount 欄位, 雖然這違反了正規化思想, 但適當的餘冗欄位減少 join 的次數是很多公司會使用的提昇效能方式, 推薦使用(我的使用情境裡, 剛好 yale在第4筆訂單的訂購, 會員過期了, 所以將 discount 紀錄在 order 裡面應該也是好的)


## DB Schema

接下來來實作 schema

ec.sql
```sql
-- setup
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

-- teardown
ALTER TABLE IF EXISTS "orders" DROP CONSTRAINT IF EXISTS "orders_order_by_fkey";

DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS items;
DROP TABLE IF EXISTS users;
```


### SQL Editor 直接貼上執行 sql
我是用 [Beekeeper](https://github.com/beekeeper-studio/beekeeper-studio) ( 也有用 [DBeaver](https://dbeaver.io/download/) )

連線就是注意 port 要對, user是預設的 postgres, password 12345, 連進去後開個sql editor, 貼上 sql, 把 CREATE TABLE 的那些都 框起來 run (下面 DROP 先省略, 那邊是清掉DB的code)
