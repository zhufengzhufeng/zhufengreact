## 什么是Redux
我们一直通过属性来进行组件中的数据传递,这种模式是非常脆弱的。在日常的开发中经常会遇到非父子组件传递的场景。原来的方式是找到共同的父级进行数据交互，这时通信就变得比较麻烦
我们先通过一个简单的例子实现一下redux的工作模式:
### 1).统一数据管理
```
let state = {
  title:{color:'red',text:'标题'},
  content:{color:'green',text:'内容'}
};
function renderContent() {
  let content = document.querySelector('.content');
  content.innerHTML = state.content.text;
  content.style.color = state.content.color;
}
function renderTitle() {
  let title = document.querySelector('.title');
  title.innerHTML = state.title.text;
  title.style.color = state.title.color;
}
function renderApp() {
  renderContent();
  renderTitle()
}
renderApp();
```

> 这里我们可以将renderContent,renderTitle看成两个组件将所需的数据提取到state中统一进行管理。当渲染后我们希望更改状态，封装更改状态的方法(dispatch)

### 2).实现dispatch
```
let CHANGE_TITLE_TEXT = 'CHANGE_TITLE_TEXT';
function dispatch(action) {
  switch (action.type){
    case CHANGE_TITLE_TEXT:
      state = {...state,title:{...state.title,text:action.text}};
  }
}
setTimeout(()=>{
  dispatch({type:CHANGE_TITLE_TEXT,text:'hello'});
  renderApp();
},1000);
```

> 不要直接更改状态而是使用dispatch方法进行状态的更改,派发一个带有type的属性来进行状态的更改，但是依然无法阻止用户更改状态.

### 3).createStore的实现
```
let CHANGE_TITLE_TEXT = 'CHANGE_TITLE_TEXT';
function createStore() {
  let state = {
    title:{color:'red',text:'标题'},
    content:{color:'green',text:'内容'}
  };
  let getState = () => JSON.parse(JSON.stringify(state)); // 创造一份和状态同样的对象给外界来用
  function dispatch(action) {
    switch (action.type){
      case CHANGE_TITLE_TEXT:
        state = {...state,title:{...state.title,text:action.text}};
    }
  }
  return {
    dispatch,
    getState,
  }
}
let store = createStore(); // 拿到createStore中返回的对象
function renderContent() {
  let content = document.querySelector('.content');
  content.innerHTML = store.getState().content.text;
  content.style.color = store.getState().content.color;
}
function renderTitle() {
  let title = document.querySelector('.title');
  title.innerHTML = store.getState().title.text;
  title.style.color = store.getState().title.color;
}
function renderApp() {
  renderContent();
  renderTitle()
}
renderApp();

setTimeout(()=>{
  store.dispatch({type:CHANGE_TITLE_TEXT,text:'hello'});
  renderApp();
},1000);
```

> 我们将状态放到了createStore函数中，目的是隔离作用域，并且再内部返回深度克隆的对象，这样用户无法再通过外界更改状态。但是状态应该由我们自身来控制，应该是外界传入的，所以要将状态拿出createStore。并且判断的逻辑也应该由我们自己来编写

### 4).reducer的实现
```
const CHANGE_TITLE_TEXT = 'CHANGE_TITLE_TEXT';
function createStore(reducer) {
  let state;
  let getState = () => JSON.parse(JSON.stringify(state));
  function dispatch(action) { 
    state  = reducer(state,action);//获取对应的状态覆盖掉store中的状态
  }
  dispatch({}); // 默认传入空对象获取reducer返回的默认结果
  return {
    dispatch,
    getState,
  }
}
let initState = {
  title:{color:'red',text:'标题'},
  content:{color:'green',text:'内容'}
};
// reducer应该具有默认状态,当更改状态后使用最新的状态
function reducer(state=initState,action) {
  switch (action.type){
    case CHANGE_TITLE_TEXT:
      return {...state,title:{...state.title,text:action.text}};
  }
  return state
}
```

> 此时我们已将需要自己处理的逻辑提取出来，但是我们每次dispatch时都需要自己触发视图的更新,我们希望采用发布订阅来实现。

### 5).订阅函数
```
function createStore(reducer) {
  let state;
  let listeners = []; // 放置所有订阅的函数
  let getState = () => JSON.parse(JSON.stringify(state));
  function dispatch(action) {
    state  = reducer(state,action);
    listeners.forEach(item=>item());//每次派发后执行订阅的函数
  }
  let subscribe = (fn)=>{ //主要用于订阅事件
    listeners.push(fn);
    return ()=>{ //返回一个移除监听的方法
      listeners = listeners.filter(l=>l!==fn);
    }
  };
  dispatch({});
  return {
    dispatch,
    getState,
    subscribe
  }
}
store.subscribe(renderApp); //通过suscribe订阅派发时需要触发的函数
setTimeout(()=>{
  store.dispatch({type:CHANGE_TITLE_TEXT,text:'hello'});
},1000);
```

> 此时我们redux中常用的方法已经封装完成！^_^,我们将封装好的逻辑抽离成redux.js

```
function createStore(reducer) {
  let state;
  let listeners = []; // 放置所有订阅的函数
  let getState = () => JSON.parse(JSON.stringify(state));
  function dispatch(action) {
    state  = reducer(state,action);
    listeners.forEach(item=>item());//每次派发后执行订阅的函数
  }
  let subscribe = (fn)=>{ //主要用于订阅事件
    listeners.push(fn);
    return ()=>{ //返回一个移除监听的方法
      listeners = listeners.filter(l=>l!==fn);
    }
  };
  dispatch({});
  return {
    dispatch,
    getState,
    subscribe
  }
}
```


## 2.应用redux+js实现counter
```
<p id="container"></p>
<button id="add">+</button>
<button id="minus">-</button>
<script src="redux.js"></script>
<script>
  const ADD = 'ADD';
  const MINUS = 'MINUS';
  function reducer(state={number:0},action) {
    switch (action.type){
      case ADD:
        return {number:state.number + action.amount};
      case MINUS:
        return {number:state.number - action.amount};
    }
    return state;
  }
  let store = createStore(reducer);
  function render() {
    container.innerHTML = store.getState().number
  }
  render();
  store.subscribe(render);
  add.addEventListener('click',function () {
    store.dispatch({type:ADD,amount:1})
  },false);
  minus.addEventListener('click',function () {
    store.dispatch({type:MINUS,amount:2})
  },false)
</script>
```


> 由此我们使用了自己的redux库链接了原生js进行使用。

## 3.应用redux+react实现counter
```
import React,{Component} from 'react'
import ReactDOM,{render} from 'react-dom';
import {createStore} from './redux'
const ADD = 'ADD';
const MINUS = 'MINUS';
function reducer(state={number:0},action) {
  switch (action.type){
    case ADD:
      return {number:state.number + action.amount};
    case MINUS:
      return {number:state.number - action.amount};
  }
  return state;
}
let store = createStore(reducer);
class Counter extends React.Component {
  constructor(){
    super();
    this.state = {number:store.getState().number}
  }
  componentDidMount(){
    store.subscribe( () => {
      this.setState({number:store.getState().number})
    })
  }
  handleAddClick=()=>{
    store.dispatch({type:ADD,amount:1});
  };
  handleMinusClick=()=>{
    store.dispatch({type:MINUS,amount:1});
  };
  render(){
    return <div>
      <p>{this.state.number}</p>
      <button onClick={this.handleAddClick}>+</button>
      <button onClick={this.handleMinusClick}>-</button>
    </div>
  }
}

ReactDOM.render(<Counter/>,window.root);
```

> 这里我们将redux数据映射到了组件自己的状态，并且订阅了setState事件。每次状态更新时都会重新刷新组件



