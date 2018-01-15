## React-Redux应用
### 1.为什么需要高阶组件
我们先看一个非常常见的例子，一个输入框需要从本地获取数据将获取的数据放到输入框内
```
export default class Username extends React.Component {
  constructor(){
    super();
    this.state = {val:''}
  }
  componentDidMount(){
    let username = localStorage.getItem('username')||'';
    this.setState({
      val:username
    })
  }
  render(){
    return <div>
      <input type="text" value={this.state.val} onChange={()=>{}}/>
    </div>
  }
}
```

> 这段逻辑可能在Password组件中也要使用，那么从本地存储中获取数据放到输入框内的逻辑应该就是公用逻辑。这时我们就要使用高阶组件，也就是将组件在原有的基础上进行包装。

### 2.实现高阶组件
```
import React from 'react';
let local = (key)=>(Component)=>{
  return class HighOrderComponent extends React.Component{
    constructor(){
      super();
      this.state = {val:''}
    }
    componentDidMount(){
      let username = localStorage.getItem(key)||'';
      this.setState({
        val:username
      })
    }
    render(){
      return <Component {...this.state}/>
    }
  }
};
export default local;

import Local from './Local'
class Username extends React.Component {
  render(){
    return <div>
      <input type="text" value={this.props.val} onChange={()=>{}}/>
    </div>
  }
}
export default Local('username')(Username);
```

> 我们将公共的逻辑拿到外层组件，处理好后以属性的方式传递给原本的组件，为此高阶组件就是一个 React 组件包裹着另外一个 React 组件


### 3.context的用法
react是单向数据流，我们想传递数据需要一层层向下传递，数据传递变得非常麻烦,我们可以用context实现数据的交互

1) 父 childContextTypes getChildContext函数

2) 子 contextTypes

```
App |-> header -> title
```

#### 跨组件交互

```
import React from 'react';
import PropTypes from 'prop-types'
import Header from "./Header";
export default class App extends React.Component {
  constructor(){
    super();
    this.state = {color:'red'}
  }
  static childContextTypes = { //定义子组件上下文的类型
    color:PropTypes.string,
    setColor:PropTypes.func
  };
  setColor = (color) =>{
      this.setState({
        color
      })
  };
  getChildContext(){ // 定义子组件上下文的数据
    return {color:this.state.color,setColor:this.setColor}
  }
  render(){
    return <div>
      <Header/>
    </div>
  }
}

export default class Header extends React.Component {
  static contextTypes = {
     setColor:PropTypes.func
  };
  render(){
    return <div>
      <button onClick={()=>{
        this.context.setColor('green');
      }}>变绿</button>
      <Title/>
    </div>
  }
}

export default class Title extends React.Component {
  static contextTypes = {
    color:PropTypes.string
  };
  render(){ // 通过context获取父组件定义的数据
    return <div style={{color:this.context.color}}>Title</div>
  }
}
```


### 4.使用react-redux实现todo
```
Todos |-> TodoHeader
      |-> TodoItems
      |-> TodoFooter
```


### 5.实现react-redux库
#### react-redux计数器
和以前写过的逻辑一致,这回加上react-redux的逻辑
```
import React from 'react';
import ReactDOM from 'react-dom';
import Counter from "./components/Counter";
import store from './store/index';
import {Provider} from 'react-redux';
ReactDOM.render(
  <Provider store={store}>
    <Counter/>
  </Provider>,window.root);

// counter组件
class Counter extends React.Component {
  render(){
    return <div>
      数量:{this.props.number}
      <button onClick={()=>{this.props.add(1)}}>+</button>
      <button  onClick={()=>{this.props.minus(1)}}>-</button>
      </div>
  }
}
export default connect(state=>({...state}),dispatch=>({
  add:(amount)=>{dispatch(actions.add(amount))},
  minus:(amount)=>{dispatch(actions.minus(amount))}
}))(Counter)
```

#### 编写react-redux库
```
import React from 'react';
import PropTypes from 'prop-types';
class Provider extends React.Component{
  static childContextTypes = {
    store:PropTypes.object
  };
  getChildContext(){
    return {store:this.props.store}
  }
  constructor(){
    super();
  }
  render(){
    return this.props.children;
  }
}
let connect = (mapStateToProps,mapDispatchToProps) => (Component) =>{
  return class Proxy extends React.Component{
    static contextTypes = {
      store:PropTypes.object
    };
    componentDidMount(){
      this.unsubscribe = this.context.store.subscribe(()=>{
        this.setState(mapStateToProps(this.context.store.getState()))
      });
    }
    componentWillUnmount(){
      this.unsubscribe();
    }
    constructor(props,context){
      super();
      this.state = mapStateToProps(context.store.getState());
    }
    render(){
      return <Component {...this.state} {...mapDispatchToProps(this.context.store.dispatch)}/>
    }
  }
};
export {Provider,connect}
```


#### bindActionCreators方法
```
let bindActionCreators = (actions,dispatch) => {
  let obj = {}
  for(let key in actions){
    obj[key] = (...args)=>{
      dispatch(actions[key](...args))
    }
  }
  return obj
};

export default connect(state=>({...state}),dispatch=>bindActionCreators(actions,dispatch))(Counter)
```

> bindActionCreators是redux中的一个方法，并且这样的逻辑过于复杂，我们依旧希望可以在react-redux中内部可以简化操作

#### 简化mapDispatchToProps
```
export default connect(state=>({...state}),actions)(Counter);

import {bindActionCreators} from './redux'
render(){
  let r ={}
  if(typeof mapDispatchToProps === 'object'){
    r = bindActionCreators(mapDispatchToProps,this.context.store.dispatch)
  }else{
    r = mapDispatchToProps(this.context.store.dispatch)
  }
  return <Component {...this.state} {...r}/>
}
```

> 这样我们在组件中更改状态时可以直接传入actionCreator对象。