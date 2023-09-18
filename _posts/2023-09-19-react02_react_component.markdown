---
layout: post
title:  (教學) react01_react_component
date:   2023-09-19 04:43:00 +0800
image:  02.jpg
tags:   Tutorial
---

個人體會的 React Component, 頁面由一個個組件構成, 每個組件維護自己本身的狀態跟頁面渲染, 組件之間能溝通, 以此達到程式碼共用以及提高擴展性等等好處

## React Component

以下是一個很簡單的組件示範, 功能是在畫面上提供輸入名字的欄位, 填入資料後, 按下送出, 可以把名字發送到後端, 然後印出後端的回應！

```js
function RegistName() {

  // 管理區域狀態
  const [name,setName] = useState("")

  // 一個跟伺服器 api 溝通的 http fetch 函式
  const postData = ()=>{
    fetch("http://localhost:5000/regist_name",{
      method:"post",
      headers:{
          "Content-Type":"application/json"
      },
      body:JSON.stringify( { "name":name } )
    })
      .then(res=>res.json())
      .then(data=>{
        console.log(data)
      })
      .catch(err=>{
        console.log(err)
      })
  }

  // return 內就是 jsx 程式碼
  return (
    <>
      <input 
        type="text" 
        placeholder="text your name here" 
        onChange = {(e)=>{
          setName(e.target.value)
        }}
      />
      
      <input
        type="button"
        value="submit"
        onClick={()=>{
          postData()
        }}
      >

    </>
  );
}

export default App;
```

以上就是一個 function component 的示範, 當然不是每個組件都這麼簡單, 但是做一個最小示範, 讓閱讀的人可以快速知道, 到底在做些什麼！