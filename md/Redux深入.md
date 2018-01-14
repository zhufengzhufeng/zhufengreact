## Redux深入
### 1.redux文件拆分
我们将计数器的案例进行文件拆分,使代码更加容易维护和阅读,我们来增加一个store文件夹
```
store
    │  action-types.js
    │  index.js
    │
    ├─actions
    │      counter.js
    │
    └─reducer
            counter.js
```

- action-types用来存放需要的常量
    ```
    export const ADD = 'ADD';
    export const MINUS = 'MINUS';
    ```

- counter中存放reducer的逻辑
    ```
    import * as Types from '../action-types'
    export default function reducer(state={number:0},action) {
      switch (action.type){
        case Types.ADD:
          return {number:state.number + action.amount};
        case Types.MINUS:
          return {number:state.number - action.amount};
      }
      return state;
    }
    ```

- store中的index文件用来创建store
    ```
    import {createStore} from '../redux';
    import reducer from './reducer/counter';
    export default createStore(reducer);
    ```

- 组件中的内容可更改为

    ```
    import React,{Component} from 'react'
    import ReactDOM,{render} from 'react-dom';
    import store from './store';
    import * as Types from './store/action-types'
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
        store.dispatch({type:Types.ADD,amount:1});
      };
      handleMinusClick=()=>{
        store.dispatch({type:Types.MINUS,amount:1});
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

> 此时我们发现在redux和组件中都应用了action-types所以我们希望继续进行拆分,在store下创建action文件夹,用来生成action对象,我们管action文件中的方法称之为actionCreator

- action文件夹中的counter,用来生成对应组件的action对象
    ```
    import * as Types from '../action-types'
    export default {
      add(amount){
        return {type:Types.ADD,amount}
      },
      minus(amount){
        return {type:Types.MINUS,amount}
      }
    }
    ```

- 再次更改组件代码
    ```
    import React,{Component} from 'react'
    import ReactDOM,{render} from 'react-dom';
    import store from './store';
    import actions from './store/actions/counter'
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
        store.dispatch(actions.add(1));
      };
      handleMinusClick=()=>{
        store.dispatch(actions.minus(1));
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
    
> 我们已经将redux的流程进行了详细的拆分,让我们来总结一下redux的流程吧：

![](http://son.fullstackjavascript.cn/redux.png)





### 2.在react中使用redux(willUnmount触发多次)(分离操作)
### 3.多个counter -> combineReducers
### 4.增加actionCreator
### 5.高阶组件 -> 本地取出数据传递给js组件(mixin)
### 6.Post 发送ajax fetch的用法
### 7.context的用法
    1) 父 childContextTypes getChildContext函数
    2) 子 contextTypes

App |-> header -> title
    |-> content

### 8.使用react-redux 实现todo (mapPropsToDispatch)
Todos |-> TodoHeader
      |-> TodoItems
      |-> TodoFooter
### 9.计数器 -> 实现react-redux
### 10.bindActionCreators
### 11.store.dispatch的改写 分析中间件 state,dispatch,action
### 12.logger -> applyMiddleware
### 13.redux-thunk (action中进行更改)
### 14.redux-promise (payload的用法)
### 15.compose函数
### 16.更新applyMiddleware
