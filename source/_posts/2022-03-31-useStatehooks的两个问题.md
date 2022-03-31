---
layout: usestate
title: useState hooks的两个问题
date: 2022-03-31 20:15:08
tags: JavaScript
categories: JavaScript
excerpt: "不要在useEffect以外的任何hooks里去调用函数或者进行类实例化"
---
# useState hooks的两个问题

## 1.别这样用useState  达咩

```javascript
import { useState } from 'react'

function App() {
  const [data, setData] = useState(getComputeredData())
  
  // 假如这是一段处理数据的方法  为了方便就用数字吧
  function getComputeredData() {
    return 1 + 2 + 3;
  }
  
  return (
    <div className="App">
      <header className="App-header">
        <button type="button" onClick={() => setData((data) => data + 1)}>
          data value is  {data}
        </button>
      </header>
    </div>
  )
}

export default App
```

其实这段代码也可以跑的~~(众所周知 人跟代码有一个能跑就行)~~ 但这样写可能会带来一些问题

当每次点击onclick的时候 setData都会执行getComputeredData()  但我的目的是只是想让data这个state自己加一  跟getComputeredData没有任何关联，如果getComputeredData方法中有其他副作用的函数调用执行，那点击几次就会(getComputeredData和其方法中的其他函数)执行几次。

举个栗子:

```javascript
import { useState } from 'react'

// 假装someEffectHandle是一个会发请求的函数
function someEffectHandle() {
  console.log("获取最新的数据");
  console.log("=============");
  return Number((Math.random()*100).toFixed(2));
}

function App() {
  const [data, setData] = useState(getComputeredData())
  
  function getComputeredData() {
    const a = someEffectHandle();
    return a + 2 + 3;
  }

  return (
    <div className="App">
      <header className="App-header">
        <button type="button" onClick={() => setData((data) => data + 1)}>
          data value is  {data}
        </button>
      </header>
    </div>
  )
}

export default App
```

这就会导致除了第一次页面加载渲染 之后的每次点击都会发送无效请求加大服务端负荷

如果getComputeredData不仅仅是一个方法 而是一个class  多次创建实例也会对垃圾回收处理机制带来压力，页面复杂的话也会影响到性能。

怎样解决这个问题呢 很简单 利用好 useState的惰性初始化 lazy initialization** 将函数传就进去就好了

```javascript
 const [data, setData] = useState(getComputeredData);
```

不成熟的小建议: 不要在useEffect以外的任何hooks里去调用函数或者进行类实例化。



## 2.useState在存入引用类型时只存入引用

```javascript
import { useState, useEffect } from 'react'

const textObj: {
  name: String,
  age?: Number
} = { name: 'tom' }

function App() {
  const [useState1, setUseState1] = useState(textObj);
  const [useState2, setUseState2] = useState(textObj);

  useEffect(() => {
    console.log(useState1) 
    console.log(useState2)
  }, [useState1]);
  
  return (
    <div className="App">
        <button type='button' onClick={() => setUseState1((oldUseState1) => {
          oldUseState1["age"] = 18
          return { ...oldUseState1 }
        })}>second</button>
    </div>
  )
}
export default App
```

点击按钮会输出 {name:"tom",age:"18"}

其实这也是一个深拷贝浅拷贝的问题

解决方法 - 把obj深拷贝过来就行了

### 经典问题：深拷贝实现

#### 1.简单版本

```javascript
JSON.parse(JSON.stringify(obj))
```

#### 2.三方小工具(lodash)

```javascript
lodash.cloneDeep(obj)
```

#### 3.自己造轮子

偷个懒放个链接

[deepClone](https://github.com/ConardLi/ConardLi.github.io/blob/master/demo/deepClone/src/clone_6.js)

