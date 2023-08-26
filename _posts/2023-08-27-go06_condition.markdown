---
layout: post
title:  (教學) go06_condition
date:   2023-08-27 03:38:00 +0800
image:  02.jpg
tags:   Tutorial
---

講解判斷式(if statement)

### 1.判斷式語法1
修改 main.go
```go
package main

import (
  "fmt"
)

var lg = fmt.Println

func main(){
  
  a := 1
  b := 2

  if a + b == 3 {
    lg(" answer is true")
  } else {
    lg("answer is false")
  }

}
```
執行結果：印出 answer is true
說明：
1. if 關鍵字後面的 a + b == 3 , 編譯器會判斷等於符號(==) 的左右兩邊, 右邊為3不用另外計算, 左邊 1+2會先經過計算也是3, 所以編譯器會得到一個 3 == 3 的式子, 因為這句話是正確的, 所以就會回傳 true
2. 然後程式就變成 if true
3. 接著會執行上面的{ lg("answer is true") }(這在程式裡面稱為block)
4. 所以當條件正確的時候執行上面block裡面的程式碼, 如果條件錯誤就執行else block 裡面的程式碼


### 2.判斷式語法加上錯誤處理
修改 main.go
```go
package main

import (
  "fmt"
)

var lg = fmt.Println

func compare(user string, target string) bool{
  return user == target
}

func main(){
  
  user1 := "yale"
  user2 := "jack"

  if compare(user1, user2){
    lg( user1 +" is "+ user2)
  } else {
    lg( user1 +" is not "+ user2 )
  }

}
```
執行結果：印出 yale is not jack
說明：
1. 定義了func compare(user string, target string) bool
2. 其中user跟target是接傳參的變數, 返回值型態是bool
3. 宣告了兩個變數 user1和user2
4. 判斷式 if compare(user1, user2),  將 user1, user2的值"yale","jack" 傳入函式, return回一個判斷式
5. 以編譯器的角度, 看到 return user == target 時, 會先計算好 "yale" == "jack" 這個判斷式, 回傳一個false
6. 所以 return user == target >> return "yale" == "jack" >> return false,  最後就是回傳一個false出去
7. 所以程式碼就變成 if false { } else { }
8. 所以就會執行後面的block的lg( user1 + "i is not " + user2 )
