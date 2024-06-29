---
title: React学习笔记
excerpt: React学习笔记，包含基本概念、组件（受控组件和非受控组件），组件通信等
published: true
time: "2024/06/29 20:10:00"
index_img: https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/1719661929726.png
category: 
- [编程,前端,React]
tags:
  - 编程
  - React
  - 学习笔记
---

## 1、安装与使用

1. 安装

```
npm install -g create-react-app
```

2. 创建项目

```
create-react-app myapp
```

3. 开始运行
   进入到根目录里，输入

```
npm start
```

## 2、初始React

### 2.1 基本流程

```jsx
// 引入模块
import React from 'react'
import ReactDOM  from 'react-dom'

// 渲染dom
ReactDOM.render(
    <h1>Hello World</h1>,
    document.getElementById('root')
)
```

### 2.2 基本JSX语法

```jsx
class App extends React.Component {
render () {
    return (
        <div className='app' id='appRoot'>
            <h1 className='title'>欢迎进入React的世界</h1>
            <p>React.js 是一个构建页面 UI 的库</p>
        </div>
    )
}
}
ReactDOM.render(
    <App></App>,
    document.getElementById('root')
)

```

> 双音节要使用驼峰命名，如className fontSize

### 2.3 class组件

```jsx
class App extends React.Component{
    a = {
        id:1,
        name:'小明'
    };
    render(){
        return (
            <div>
                <button onClick={()=>this.ceshi(this.a)}>Click</button>
            </div>
        )
    }
    ceshi = (props)=>{
        console.log(props);
    }
}
```

> 函数的写法推荐箭头函数

### 2.4 函数组件

```jsx
const App = (props) =>{
    return <h1>我来了</h1>
}
```

简写

```jsx
const App = (props) =><h1>我来了</h1>
```

> 组件名称一定要大写

### 2.5 组件样式

1. 行内样式

- 第一个括号表示里面写js表达式，第二个代表一个对象

```jsx
<p style={{color:'red', fontSize:'14px'}}>Hello world</p>
```

2. class样式

- 要写成className

```jsx
<p className="hello">Hello world</p>
```

### 2.6 事件处理

1. 绑定事件onclick
2. 推荐这种函数写法
3. event事件

```jsx
class App extends React.Component{
    a = {
        id:1,
        name:'小明'
    };
    render(){
        return (
            <div>
                <button onClick={()=>this.ceshi(this.a)}>Click</button>
            </div>
        )
    }
    ceshi = (props)=>{
        console.log(props);
    }
}
```

<span style="background-color:yellow">**event事件**</span>

```jsx
class App extends React.Component{
    a = {
        id:1,
        name:'小明'
    };
    render(){
        return (
            <div>
            {//这里箭头函数要传递参数，才能捕获到}
                <button onClick={(e)=>{this.ceshi(e)}}>Click</button>
            </div>
        )
    }
    ceshi = (event)=>{
        console.log(event);
    }
}
```

![image-20220515142102870](http://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20220515142102870.png)

### 2.7 Ref使用

1. 创建```myref = React.createRef()```
2. 绑定标签```ref={this.myref}```
3. 使用

```jsx
class App extends React.Component{
    a = {
        id:1,
        name:'小明'
    };
    myref = React.createRef();
    render(){
        return (
            <div>
                <input ref={this.myref}></input>
                <button onClick={()=>{this.ceshi(this.myref.current.value)}}>click</button>
            </div>
        )
    }
    ceshi = (event)=>{
        console.log(event);
    }
}
```

> 获取当前标签使用```this.myref.current```

## 3、组件的数据挂载

### 3.1 状态

状态就是组件描述某种显示情况的数据，由组件自己设置和更改，也就是说由组件自己维护，使用状态的目的就是为了在不同的状态下使组件的显示不同(自己管理)

1. 定义state

> 直接定义，与render()同一级

```jsx
// 类组件
class App extends React.Component{
    state = {
        name:'React',
        isLiked:false
    }
    render(){
        return (
            <div>
                <h1>欢迎来到{this.state.name}</h1>
                <button>{this.state.isLiked?'取消':'收藏'}</button>
            </div>
        )
    }
}
```

或者

> 通过constructor初始化

```jsx
// 类组件
class App extends React.Component{
    constructor(){
        super()
        this.state = {
            name:'React',
            isLiked:true
        }
    }
    render(){
        return (
            <div>
                <h1>欢迎来到{this.state.name}</h1>
                <button>{this.state.isLiked?'取消':'收藏'}</button>
            </div>
        )
    }
}
```

### 3.2 修改状态

1. 只能使用setState重新赋值

```jsx
class App extends React.Component{
    constructor(){
        super()
        this.state = {
            name:'React',
            isLiked:true
        }
        this.ceshi=()=>{
            this.setState({
                isLiked:!this.state.isLiked
            })
        }
    }
    render(){
        return (
            <div>
                <h1>欢迎来到{this.state.name}</h1>
                <button onClick={()=>{this.ceshi()}}>{this.state.isLiked?'取消':'收藏'}</button>
            </div>
        )
    }
}
```

setState 是异步的，所以想要获取到最新的state，没有办法获取，就有了第二个参数，这是一个可选的回调函数

```jsx
this.setState((prevState, props) => {
    return {
    	isLiked: !prevState.isLiked
    }, () => {
    	console.log('回调里的',this.state.isLiked)
    })
console.log('setState外部的',this.state.isLiked)
```

### 3.3 列表渲染

1. 使用map循环

```jsx
class App extends React.Component{
    state = {
        lists:['111','222','333']
    }
    render(){
        return (
            <div>
               <ul>
                   {
                   this.state.lists.map(item=><li >{item}</li>)
                   }
               </ul>
            </div>
        )
    }
}
```

以往的

```jsx
list.map(item=>`<li>{$item}</li>`)
```

> 这里是react的jsx语法，所以不用加引号

### 3.4 TodoList

添加

```jsx
add=(value)=>{
    console.log(value);
    this.state.lists.push(value)
    this.setState({
        lists:this.state.lists
    })
}
```

不能写成

```jsx
add=(value)=>{
    console.log(value);
    this.setState({
        lists: this.state.lists.push(value)
    })
}
```

因为push方法改变的是原数组，而没有返回值

代码

1. 使用push方法追加，改变原数组
2. 使用splice删除，改变原数组

```jsx
class App extends React.Component{
    myref = React.createRef()
    state = {
        lists:['111','222','333']
    }
    add=(value)=>{
        let newList = this.state.lists
        newList.push(value)
        this.setState({
            lists:newList
        },()=>{
            this.myref.current.value = '' // 置空
        })
    }
    delete=(index)=>{
        let newList = this.state.lists
        newList.splice(index,1)
        this.setState({
            lists:newList
        },()=>{
            console.log('删除成功');
        })
    }
    render(){
        return (
            <div>
                <input ref={this.myref}></input>
                <button onClick={()=>{this.add(this.myref.current.value)}}>Add</button>
               <ul>
                   {this.state.lists.map((item,index)=><li key={item}>{item} <button onClick={()=>{this.delete(index)}}>delete</button></li>)}
               </ul>
            </div>
        )
    }
}
```

### 3.5 条件渲染

```jsx
{this.state.lists.length==0?<div >暂无待办事项</div>:null}
```

或者

```jsx
{this.state.lists.length==0 && <div >暂无待办事项</div>}
```

或者

```jsx
.active{
    display: none;
}
```

```jsx
<div className={this.state.lists.length!=0 && 'active'}>暂无带吧</div>
```

### 3.6 状态再体验

1. setState函数异步

在异步逻辑中，是同步更新状态，更新真实dom

在同步逻辑中，是异步更新状态，更新真实dom

2. setState回调函数

```jsx
this.setState({
   	 lists:newList
	},()=>{
    this.myref.current.value = ''
})
```

### 3.7 属性初试

1. 在组件上写```key=‘value’```形式，在组件内部用```this.props```接收;
2. 属性是描述性质、特点的，组件自己不能随意更改；

<span style="background-color:yellow">**1**</span>

```jsx
class App extends React.Component{
    render(){
        return (
            <div>
            {// 传入}
                <NavaBar title='导航栏'></NavaBar>
            </div>
        )
    }
}
```

```jsx
class NavaBar extends React.Component{
    render(){
        // 接受
        let {title} = this.props
        return (
            <div>
                NavaBar - {title}
            </div>
        )
    }
}
```

### 3.8 属性验证

在组件内部

```jsx
class NavaBar extends React.Component{
    static propTypes = {
        title:kerwinPropTypes.string
    }
    render(){
        let {title} = this.props
        return (
            <div>
                NavaBar - {title}
            </div>
        )
    }
}
```

### 3.9 默认属性

在组件内部

```jsx
class NavaBar extends React.Component{
    static defaultProps = {
        title:'我是'
    }
    render(){
        let {title} = this.props
        return (
            <div>
                NavaBar - {title}
            </div>
        )
    }
}
```

## 4、表单中的受控组件与非受控组件

### 4.1 非受控组件

1. React要编写一个非受控组件，可以 使用 ref 来从 DOM 节点中获取表单数据，就是非受控组件。
2. 比如在使用input输入框获取输入框内容时，是通过操作原生DOM节点来获取的，不受控制；

![image-20220515182305778](http://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20220515182305778.png)

只能使用```defaultValue```

### 4.2 受控

1. 通过value动态绑定状态，这里可以把input标签看成一个组件；

```jsx
import React from "react";


class App extends React.Component{
    state = {
        usename:'我是大傻逼'
    }
    render(){
        return (
            <div>
                <input value={this.state.usename} onChange={(e)=>{
                    this.setState({
                        usename:e.target.value
                    })
                }}></input>
                <button onClick={()=>{
                    console.log(this.state.usename);
                }}>登录</button>
                <button onClick={()=>{
                    this.setState({
                        usename:''
                    })
                }}>重置</button>
            </div>
        )
    }
}

export default App
```

### 4.3 受控应用1

```jsx
import React from "react";


class App extends React.Component{
    state = {
        inputValue:'',
        cinma:[
            {
                id:0,
                name:'a'
            },
            {
                id:1,
                name:'b'
            },
            {
                id:2,
                name:'c'
            },
            {
                id:3,
                name:'d'
            },
        ]
    }
    render(){
        return (
            <div>
                <input value={this.state.inputValue} onChange={(e)=>{
                    this.setState({
                        inputValue:e.target.value
                    })
                }}></input>
                {
                    this.getList().map(item=><li key={item.id}>{item.name}</li>)
                }
            </div>
        )      
    }
    getList(){
        return this.state.cinma.filter(item=>item.name.toUpperCase().includes(this.state.inputValue.toLocaleUpperCase()
        ))
    }
}

export default App
```

### 4.4 todoList受控



```jsx
import React from "react"

export default class App extends React.Component{
    state = {
        inputValue:'',
        lists:[
            {
                id:0,
                name:'a',
                isChecked:false
            },
            {
                id:1,
                name:'b',
                isChecked:false
            },
            {
                id:2,
                name:'c',
                isChecked:false
            },
        ]
    }
    add(){
        let newList = this.state.lists
        newList.push({
            id:Math.random()*100000,
            name:this.state.inputValue,
            isChecked:false
        })
        this.setState({
            lists:newList,
            inputValue:''
        })
    }
    delete=(index)=>{
        let newList = this.state.lists
        newList.splice(index,1)
        this.setState({
            lists:newList
        })
    }
    handchange=(index)=>{
        console.log(index);
        let newList = [...this.state.lists]
        newList[index].isChecked = !newList[index].isChecked
        this.setState({
            lists:newList,
        })
    }
    render(){
        return (
            <div>
                <input value={this.state.inputValue} onChange={(e)=>{
                    this.setState({
                        inputValue:e.target.value
                    })
                }}></input>
                <button onClick={()=>{this.add()}}>Add</button>
               <ul>
                   {this.state.lists.map((item,index)=>
                   <li key={item.id}>
                       <input type='checkbox' checked={item.isChecked} onChange={()=>{this.handchange(index)}}></input>
                       <span style={{textDecoration:item.isChecked?"line-through":''}}>{item.name}</span>
                       <button onClick={()=>{this.delete(index)}}>delete</button>
                    </li>
                   )}
               </ul>
               {this.state.lists.length==0 && <div>暂无待办事项</div>}
            </div>

        )
    }
}
```



## 5、组件通信方式

### 5.1 子传父

1. 子组件不能直接访问父组件的状态，但是可以通过回调函数通知父组件改变状态；
2. 孩子不能动父亲手里的钱，但是孩子可以打电话给父亲，让父亲发钱；

```react
import React, { Component } from 'react'

class NavBar extends Component{
    render(){
        return(
            <div>我是导航栏
                <button onClick={()=>{
                    this.props.isShowList();                    
                }}>点我</button>
                
            </div>
        )
    }
}

class ListBar extends Component{
    render(){
        return(
            <div>
                <ul style={{backgroundColor:'yellow',width:'200px'}}>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                </ul>
            </div>
        )
    }
}



export default class App extends Component {
  state = {
    isShow:true
  }
  render() {
    return (
      <div>
          <NavBar isShowList={()=>{
              this.setState({
                  isShow:!this.state.isShow
              })
          }}></NavBar>
          {this.state.isShow && <ListBar></ListBar>}
      </div>
    )
  }
}

```







### 5.2 中间人





![image-20220517230844506](http://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20220517230844506.png)

```react
import React, { Component } from 'react'
import axios from 'axios'
import './06-index.css'

export default class App extends Component {
    constructor(){
        super()
        this.state={
            filmList:[],
            detail:""
        }
        axios.get(`../text.json`).then(res=>{
            console.log(res.data.data)
            this.setState({
                filmList:res.data.data
            })
        })
    }

  render() {
    return (
      <div>
          {
              this.state.filmList.map(item=><Film key={item.id} {...item} onEvent={(value)=>{
                this.setState({
                    detail:value
                })
              }}></Film>)
          }
          <FilmDetial detail={this.state.detail}></FilmDetial>
      </div>
    )
  }
}


class Film extends Component{
    render(){

        let {name,src,detail} = this.props
        return <div className='filmitem' onClick={()=>{
            this.props.onEvent(detail)
        }}>
            <img src={src} alt={name}></img>
            {this.props.name}
        </div>
    }
}


class FilmDetial extends Component{
    render(){
        return <div className='filmdetail'>
            {this.props.detail}
        </div>
    }
}
```

### 5.3 子传父表单

1. 子组件输入框动态绑定父组件上的状态值

```react
import React, { Component } from 'react'

export default class App extends Component {
    state = {
        name:'小明',
        password:'123456'
    }
  render() {
    return (
      <div>
        <h2>登录页面</h2>
        <Filed lable = "用户名" type = "text" value = {this.state.name} onEvent={(value)=>{
            this.setState({
                name:value
            })
        }}/>
        <Filed lable = "密码" type = "password" value = {this.state.password} onEvent={(value)=>{
            this.setState({
                password:value
            })
        }}/>
        <button onClick={()=>{
            console.log(this.state.name,this.state.password);
        }}>登录</button>
        <button onClick={()=>{
            this.setState({
                name:'',
                password:''
            })            
        }}>取消</button>
      </div>
    )
  }
}

class Filed extends Component{
    render(){
        let {lable,type,onEvent,value} = this.props
        return(
        <div>
            <span>{lable}</span>
            <input type={type} value={value} onChange={(e)=>{
                onEvent(e.target.value)
            }}></input>
        </div>
    )}
}

```

### 5.4 ref表单

1. 在组件上挂载ref可以获得组件；
2. 父组件给子组件挂载ref可以拿到子组件的```dom```；
3. 子组件内部自己设置状态和绑定value值；
4. 通过点击按钮```this.username.current```获取组件标签；
5. 就可以拿到子组件内的状态和函数（清空函数在子组件自己内部）；

```react
import React, { Component } from 'react'

export default class App extends Component {
    username = React.createRef();
    password = React.createRef();

  render() {
    return (
      <div>
        <h2>登录页面</h2>
        {/* 这里使用ref能拿到组件标签 */}
        <Filed lable = "用户s名" type = "text" ref={this.username}/>
        <Filed lable = "密码" type = "password" ref={this.password}/>
        <button onClick={()=>{
            console.log(this.username.current.state.value);
            console.log(this.password.current.state.value);
        }}>登录</button>
        <button onClick={()=>{
            this.username.current.clear()
            this.password.current.clear()
        }}>取消</button>
      </div>
    )
  }
}

class Filed extends Component{
    state = {
        value:""
    }
    clear(){
        // 虽然这个函数能执行，但是并没有使render重新执行
        // 需要加将input输入框绑定value
        this.setState({
            value:""
        })
    }
    render(){
        let {lable,type} = this.props
        return(
        <div>
            <span>{lable}</span>
            <input type={type} value={this.state.value} onChange={(e)=>{
                console.log(e.target.value);
                this.setState({
                    value:e.target.value
                })
            }}></input>
        </div>
    )}
}
```

> 子组件内的输入框要与自己的状态value绑定，否则执行clear函数不会触发render()

## 6、Hooks

1. 高阶组件为了复用，导致代码层级复杂
2. 生命周期的复杂
3. 写成functional组件,无状态组件 ，因为需要状态，又改成了class,成本高

### 6.1 ```useState```

1. ```[a,b] = arr;```此时arr是一个数组，解构赋值；
2. 使用```useState```会返回一个数组，里面有一个值和函数；

```jsx
import React,{useState} from 'react'

export default function App() {

    const [username,setusername] = useState("mtl")

  return ( 
    <div>
       <button onClick={()=>{
           setusername("大白")
       }}>click</button> 
        {username}</div>
  )
}

```

重写todolist

```jsx
import React,{useState} from 'react'

export default function APP() {

   const [text,setText] = useState("")
   const [list,setList] = useState(['aaa','bbb','ccc']) //初始列表值

    const add=()=>{
        // 结构列表并添加新值
        setList([...list,text])
    }

    const del=(index)=>{
        console.log(index);
        // 不要直接更改原数组，创建一个新数组
        const newlist = [...list]
        // 删除索引位置1个元素
        newlist.splice(index,1)
        setList(newlist)

    }

  return (
    <div>
        <input onChange={(e)=>{
            setText(e.target.value)
        }}></input>
        <button onClick={()=>{
            add()
        }}>add</button>
        <ul>
            {
                list.map((item,index)=><li key={index}>{item}<button onClick={()=>{
                    del(index)
                }}>del</button></li>)
            }
        </ul>
    </div>
  )
}

```

























