---
layout: post
title:  (教學) react03_useState
date:   2023-09-26 00:36:00 +0800
image:  02.jpg
tags:   Tutorial
---

前面介紹了 React render, 接著理解了 component, 接下來理解 react 怎麼 維護資料

React Hook 是 React 用來處理資料的各式各樣 api, 最基礎這邊先介紹3個最常用的
1. [React.useState](https://zh-hant.legacy.reactjs.org/docs/hooks-state.html) : 響應式的狀態變數
2. [React.useContext](https://react.dev/reference/react/useContext): 在元件之間傳遞資料的變數
3. [React.useEffect](https://zh-hant.legacy.reactjs.org/docs/hooks-effect.html): 監聽狀態的函式, 功能有點類似 eventListener


## Getting Started
1. 建立專案 
```bash
npm create vite
# ✔ Project name:  react_hook
# ✔ Select a framework: › React
# ✔ Select a variant: › JavaScript
```

2. 到專案內, 安裝依賴套件, 並將專案跑起來
```bash
cd react_hook
yarn
yarn run dev
```
預期看到預設的畫面

3. 我們寫 code 的入口是 /src/App.jsx
更改 App.jsx 為以下內容
```js
import Hello from './Hello'

function App() {
  
  return (
    <>
      <Hello/>
    </>
  )
}

export default App

```
先將原本的 code 拿掉, 我們要放入我們的 code, 這時候瀏覽網頁應該會是空白畫面

4. 做一個 Hello component
開一個檔案 Hello.jsx, 放入以下內容
```js
import { useState } from 'react'

function Hello(){
  // name 跟 setName 是 useState() 執行後回傳的變數
  // useState("") 意思是放一個空字串當作 name 的初始值
  // name 本身是 reactive 的變數, setName 是負責更新 name 變數的函式 
  const [name, setName] = useState("")
  const [enemy, setEnemy] = useState("")

  // 這邊封裝了個函式, 要來請求 api 資料用
  function requestData(){
    
    // 前面有練習過 鏈式呼叫, 這邊就是一個鏈式呼叫
    // fetch 的參數是呼叫的網址, 是一個開放的 api
    // .then 內 是對方回傳的資料, 通常用 json來溝通, 所以需要 .json()做解析
    // 第二個 .then 就是取出來的資料, 我們用 setEnemy 函式, 將 enemy 變數, 設定為資料的內容
    // .catch 是如果上面有出錯, 就會執行, 把錯誤資訊印出來
    // 所以 enemy 這個變數, 就是一個 reactive 的變數, 可以讓網頁自動重新渲染
    fetch("https://swapi.dev/api/people/1/")
    .then(res=>res.json())
    .then(data=>{
      console.log(data)
      setEnemy(data)
    })
    .catch(err=>{
      console.log(err)
    })
  }

  return (
    <>
      {/* 這邊我們做一個輸入框, 語法類似 純js, 監聽 input 有異動的時候, 把 name 變數變更*/}      
      <input 
        type="text" 
        placeholder='your name?' 
        onChange={(e)=>{
          setName(e.target.value)
        }}
      />

      {/* 因為 name 這個變數是 reactive, 所以會即時觸發渲染在網頁上 */}
      <div> Hello { name } </div>

      {/* 這邊做一個按鈕, 按下去後, 會呼叫 requestData 函式, 最後資料回來時, 會觸發選染, 顯示在網頁上*/}
      <input
        type="button"
        value="require an challenger"
        onClick={()=>{
          requestData()
        }}
      />
      <div> Here comes {enemy.name}</div>
    </>
  )

}

export default Hello
```
這樣就會有網頁出現了