---
layout: post
title:  (教學) how-to-write-simple-redux
date:   2023-09-26 03:20:00 +0800
image:  02.jpg
tags:   Tutorial
---



本篇介紹如何自己手刻簡單的 redux  

#### 小note：此篇為興趣導向, 對工作不一定有實質幫助, 供參

### [程式碼](https://github.com/cbot918/ithelp/tree/main/go-junior-30/hand-redux)



簡單介紹下, 共有三個檔案

1. index.html: 就基本index.html, 多了一行 type="module", 為了使用 import
2. redux.js: 
- reducer: 就是判斷 action 是什麼, 把狀態做改變的函式
- createStore: 這邊用閉包做了個全域用的 state,  subscribe 是將想訂閱的函數加入, dispatch 時, 會先將狀態改變一次, 然後 去 eventhub 裡面把訂閱的函數一個一個執行, 如此做到 reactive
- 注意 redux.js export 出來的是一個已經初始化過的 store 物件

3. main.js: 第一行引入 store 物件, 然後用js將 兩個按鈕以及一個 view 宣告出來, 在最下方有個 render 函式, 這邊是傳入 store, 然後 innerHTML 去把狀態更新出來, 由於 render 函式會放入 store 的 eventhub裡面, 在 dispatch 函式內, 會先做一次 state = reducer, 因此 render 出來的 state 會是最新的資料 
### index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  
  <script type="module" src="main.js"></script>

</body>
</html>
```


### redux.js
```js
function reducer(state = 0, action){
  if( action && action.type === "add"){
    return state + action.payload
  } 
  if ( action && action.type === "minus"){
    return state - action.payload
  }

  return state
}


function createStore( reducer ){
  let state = reducer()
  let eventhub = []

  function getState(){
    return state
  }

  function dispatch( action ){
    state = reducer(state, action)
    eventhub.forEach( (fn) => fn() )
  }

  function subscribe(fn){
    eventhub.push(fn)
  }

  return {
    getState,
    dispatch,
    subscribe
  }
}

const store = createStore(reducer)

export { store }
```


### main.js
```js
import { store } from './redux.js'

const log = console.log

window.onload = ()=>{
  

  let body = document.querySelector("body")
  
  const addThree = document.createElement('input')
  addThree.setAttribute("type","button")
  addThree.setAttribute("value","+3")
  addThree.addEventListener('click',()=>{
    store.dispatch( {type:"add", payload:3} )
  })
  body.appendChild(addThree)
  
  const minusTwo = document.createElement('input')
  minusTwo.setAttribute("type","button")
  minusTwo.setAttribute("value","-2")
  minusTwo.addEventListener('click',()=>{
    store.dispatch( {type:"minus", payload:2} )
  })
  body.appendChild(minusTwo)

  const view = document.createElement('div')
  body.appendChild(view)
  
  
  store.subscribe(()=>{
    render( view, store)
  })

  log(store.getState())


}

function render( target, store ){

  target.innerHTML = `
  <div> count: ${store.getState()}</div>
`
}

```


