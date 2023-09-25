---
layout: post
title:  (教學) react05_useContext-redux
date:   2023-09-26 03:20:00 +0800
image:  02.jpg
tags:   Tutorial
---

useContext 是負責跨組件傳遞資料用, 不像是 props, 需要用參數傳遞, 在本篇會搭配 redux, 直接用程式碼去說明 context 怎麼傳遞物件

提醒：redux 觀念上可能會比較不好懂, 但算是相對精簡扼要的示範, 希望對學習的人有幫助

在下面的程式裡面 
- context 負責將 UserContext 跨檔案傳遞出去, UserContext 裡面攜帶了 state 及 dispatch 
- redux 負責傳遞程式內 name 的狀態 (使用 dispatch )


0. 先建立一個 useReducer 物件
/reducer/useReducer.jsx
```js
// 初始狀態 設為 null
export const initialState = null

// reducer 是一個狀態改變函數, 根據傳進來的 state 以及 action.type 去決定回傳值, 這邊只做簡單的範例示範
export const reducer = (state, action) =>{
  
  if(action.type === "USER"){
    return action.payload
  }

	// 這個 判斷式是寫假的, 讓讀者容易理解, redux 的用法
	// 真實場景就是在上傳大頭之後, 讓顯示大頭的地方有資料
	if(action.type=="UPDATEPIC"){
    return{
      ...state,
      pic:action.payload
    }
  }

  return state
}
```

1. 先建立一個 UserContext 
App.jsx
```js
import EnemyList from './components/enemylist/EnemyList'
import Login from './components/login/Login'
import {  Routes, Route  } from "react-router-dom";
import { createContext, useReducer } from 'react';
import { reducer,initialState } from './reducer/useReducer'

// 這邊建立一個全域物件
export const UserContext = createContext()

function App() {

	// 這邊使用 useReducer 將 /reducer/useReducer.js 的 initialState 跟 dispatch 引用, 變成 state 狀態, 以及 dispatch 函式, 用來更新狀態
  const [state, dispatch] = useReducer(reducer,initialState)

  return (
    <>
			{/* 這邊 UserContext.Provider 指定了 UserContext 攜帶的變數*/}
			{/* 攜帶了 {state, dispatch} , 後面的 useContext 函式都會看到 */}
      <UserContext.Provider value={{state, dispatch}}>

				{/* 這邊為了login後的頁面跳轉做了 react router, 用法有放在參考連結*/}
        <Routes>
          <Route path="*" element={<Login />}></Route>
          <Route path="/enemylist" element={<EnemyList/> } />
        </Routes>
      </UserContext.Provider> 
    </>
  )
}

export default App

```

2. Login
/components/login/Login.jsx
```js
import './login.css'

import { useState, useContext} from 'react'
import { useNavigate } from 'react-router-dom'
// 引入 UserContext 
import { UserContext } from '../../App'

function Login(){
	
	// 把 state dispatch 物件拿出來
  const {state, dispatch} = useContext(UserContext)

  const [name, setName] = useState("")

	// 這個是用來做頁面跳轉的
  const navigate = useNavigate()

  return (
    <div className="outer">
      <div> enter your name </div>

      <input 
        className="username"
        type="text"
        placeholder='username'
        onChange={(e)=>{
          setName(e.target.value)
        }}
      />

      <div>
        <input 
          className="submit"
          type="button" 
          value="登入" 
          onClick={(e)=>{
						// 這邊的 dispatch 就會更新觸發 reducer, 去更新 state的值
            dispatch({type:"USER", payload:name})
						// 頁面跳轉到 /enemylist
            navigate("/enemylist")
          }}
        />
      </div>
      

    </div>
  )
}

export default Login
```

3. enemyList
/component/enemylist/EnemyList.jsx
```js
import { useState, useEffect, useContext} from 'react'
// 引入 UserContext 物件
import { UserContext } from '../../App'
import Enemy from './Enemy'

function EnemyList(){

  const [enemy, setEnemy] = useState([{}])
	// 取出 state dispatch 物件
  const {state, dispatch} = useContext(UserContext)
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
    <>
		{/* 這邊就可以使用 reactive 的 state 變數 */}
		{/* 就是靠 useContext 傳遞 加上 reducer dispatch 達成的唷*/}
    <h2 className="title"> {state} Challenge List </h2>
    {
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
4. enemy
/component/enemylist/Enemy.jsx
```js
import  './enemy.css'

function Enemy(props){

	// 這邊示範參數 props 傳遞
  const data = {props}.props.props

  return (
    <>
    { console.log( data ) }
    
		{/* 以下把 props 傳遞進來的資料, 顯示在畫面上*/}
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


## [專案 github](https://github.com/cbot918/ithelp/tree/main/go-junior-30/react-context-redux)

參考連結：
[React hooks — useContext 到底怎麼用，看完這篇絕對懂！](https://molly1024.medium.com/react-hooks-usecontext%E5%88%B0%E5%BA%95%E6%80%8E%E9%BA%BC%E7%94%A8-%E7%9C%8B%E5%AE%8C%E7%AF%87%E7%B5%95%E5%B0%8D%E6%87%82-125fae4a1e86)

[React route—useNavigate介紹、控制history stack、傳遞參數、重新導向](https://ithelp.ithome.com.tw/articles/10306611)