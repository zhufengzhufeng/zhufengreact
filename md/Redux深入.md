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


### 2.实现多个counter
在redux中只能拥有一个store所以我们需要将多个状态进行合并,状态是通过reducer返回的，所以我们可以将多个reducer进行合并达到合并状态的目的。
```
│  index.js
│  redux.js
│
├─components
│      counter1.js
│      counter2.js
│
└─store
    │  action-types.js
    │  index.js
    │
    ├─actions
    │      counter1.js
    │      counter2.js
    │
    └─reducer
            counter1.js
            counter2.js
            index.js
```

> 这里我们将counter1的逻辑进行拷贝，粘贴出counter2

- action-types新增counter2处理的常量

    ```diff
    export const ADD = 'ADD';
    export const MINUS = 'MINUS';

    + export const INCREMENT = 'INCREMENT';
    + export const DECREMENT = 'DECREMENT';
    ```

- 对应的counter2中的action也进行更改

    ```
    import * as Types from '../action-types'
    export default {
      add(amount){
        return {type:Types.INCREMENT,amount}
      },
      minus(amount){
        return {type:Types.DECREMENT,amount}
      }
    }
    ```

- 同样reducer中处理也是一样的

    ```
    import * as Types from '../action-types'
    export default function reducer(state={number:0},action) {
      switch (action.type){
        case Types.INCREMENT:
          return {number:state.number + action.amount};
        case Types.DECREMENT:
          return {number:state.number - action.amount};
      }
      return state;
    }
    ```

    > 现在问题出现了我们拥有了两个reducer,我们要将两个reducer进行合并,合并成一个新的reducer

-  combineReducers
    ```
    import counter1 from './counter1';
    import counter2 from './counter2';
    let combineReducers = (reducers) => {
      return (state={},action)=>{
        let obj = {};
        for(let key in reducers){
          obj[key] = reducers[key](state[key],action); //调用原有的reducer将返回的结果放到对象上
        }
        return obj; // 将合并后的对象进行返回即可 {counter1:{number:0},counter2:{number:0}}
      }
    };
    export default combineReducers({
      counter1,counter2
    });
    ```

- 最后组件中获取状态要增加合并时的命名空间来获取
    ```
    constructor(){
        super();
        this.state = {number:store.getState().counter1.number}
    }
    componentDidMount(){
        store.subscribe( () => {
            this.setState({number:store.getState().counter1.number})
        })
    }
    ```





