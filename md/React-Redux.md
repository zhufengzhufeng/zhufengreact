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
    |-> content
```



### 4.使用react-redux实现todo
```
Todos |-> TodoHeader
      |-> TodoItems
      |-> TodoFooter
```


### 5.实现react-redux库

### 6.redux中bindActionCreators方法