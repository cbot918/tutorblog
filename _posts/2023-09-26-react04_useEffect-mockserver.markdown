---
layout: post
title:  (教學) react04_useEffect-mockserver
date:   2023-09-26 00:36:00 +0800
image:  02.jpg
tags:   Tutorial
---

今天介紹 useEffect, 類似 addEventListener, 可以監聽變數, 或是等畫面載入後, 再去做事情, 是一個很重要的 api

為了介紹 useEffect, 需要資料 api 來模擬情境, 但外部 api 在開放的文章裡, 怕會被打爆. 所以程式內提供了一個 自己寫的 go mock server, 相信大家都有 golang runtime, 可以自己跑起來！

### Getting Started
專案準備環節跳過, 請參考前兩天

1. App.jsx 放入以下內容
```js
import EnemyList from './EnemyList'

function App() {
  
  return (
    <>
      <div> Your Challenger's List </div>
      <EnemyList/>
    </>
  )
}

export default App
```

2. 做一個對手列表 EnemyList.jsx

components/EnemyList.jsx
```js
import { useState, useEffect} from 'react'
import Enemy from './Enemy'
function EnemyList(){

  const [enemy, setEnemy] = useState([{}])

  useEffect(()=>{
    
    fetch("http://localhost:8886/mock")
    .then(res=>res.json())
    .then(data=>{
      // console.log(data)
      setEnemy(data)
    })
    .catch(err=>{
      console.log(err)
    })

  }, [])

  return(
    <>{
      enemy.map((e)=>{
        return (         
          <Enemy props = {e} key={e.id}/>
        )
      })
    }
    </>
  )
  
}

export default EnemyList

```

3. 做一個 Enemy 元件 Enemy.jsx

components/Enemy.jsx
```js
import  './enemy.css'

function Enemy(props){

  const data = {props}.props.props

  return (
    <>
    { console.log( data ) }
    
    <div className="container">
      <div className="name"> Name: {data.name} </div>
      <div className="name"> Job: {data.job} </div>
      <div className="name"> Level: {data.level} </div>
      <div className="name"> Weapon: {data.weapon} </div>
    </div>

    </>
  )

}

export default Enemy
```

4. 設定一下 Enemy.jsx 的基本css

components/enemy.css
```css
.container{
  margin-top:10px;
  border: 1px gray solid;
  padding: 5px;
  border-radius: 5px;
}

.container div {
  padding:2px;
}
```

5. golang mock server
main.go
```go
package main

import (
	"encoding/json"
	"net/http"
)

const port = ":8886"

func main() {

	http.HandleFunc("/mock", mockData)

	http.ListenAndServe(port, nil)
}

func mockData(w http.ResponseWriter, r *http.Request) {
	mobs := []struct {
		Name   string `json:"name"`
		Job    string `json:"job"`
		Level  string `json:"level"`
		Weapon string `json:"weapon"`
	}{
		{
			Name:   "Yale",
			Job:    "Knight",
			Level:  "10",
			Weapon: "Spear",
		},
		{
			Name:   "Bubble",
			Job:    "Sword Man",
			Level:  "20",
			Weapon: "Sword",
		},
		{
			Name:   "Jack",
			Job:    "Berserker",
			Level:  "30",
			Weapon: "BareHand",
		},
		{
			Name:   "Paker",
			Job:    "Assassin",
			Level:  "40",
			Weapon: "Dagger",
		},
		{
			Name:   "Jiar",
			Job:    "Hunter",
			Level:  "50",
			Weapon: "Bow",
		},
		{
			Name:   "Wa Brother",
			Job:    "Magician",
			Level:  "60",
			Weapon: "Staff",
		},
		{
			Name:   "Sam",
			Job:    "Advanture",
			Level:  "70",
			Weapon: "Hammer",
		},
		{
			Name:   "Kasper",
			Job:    "Scientist",
			Level:  "80",
			Weapon: "Books",
		},
	}

	// set headers with allow cors and application/json
	w.Header().Set("Access-Control-Allow-Origin", "*") // Replace "*" with your allowed origins
	w.Header().Set("Access-Control-Allow-Methods", "GET, POST, OPTIONS")
	w.Header().Set("Access-Control-Allow-Headers", "Content-Type")
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	// write data to client
	err := json.NewEncoder(w).Encode(mobs)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

}

```


專案可以這邊找 [react-useeffect-mockserver](https://github.com/cbot918/ithelp/tree/main/go-junior-30/react-useeffect-mockserver)