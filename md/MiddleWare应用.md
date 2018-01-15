## MiddleWare的使用
### 1.logger中间件
我们改写了，dispatch方法实现了在更改状态时打印前后的状态,但是这种方案并不好。所以我们可以采用中间的方式。
```
let store = createStore(reducer);
let dispatch = store.dispatch;
store.dispatch = function (action) {
  console.log(store.getState().number);
  dispatch(action);
  console.log(store.getState().number)
};
export default store;
```

#### 实现logger中间件
```
let logger = store => dispatch => action=>{
  console.log(store.getState().number);
  dispatch(action);
  console.log(store.getState().number)
};
let applyMiddleWare = middleware => createStore => reducer =>{
  let store = createStore(reducer);
  let middle = middleware(store);
  let dispatch = middle(store.dispatch);
  return { //将中间返回的dispatch方法覆盖掉原有store中的dispatch
    ...store,
    dispatch
  }
};
export default applyMiddleWare(logger)(createStore)(reducer);
```


### 2.实现redux-thunk中间件
实现派发异步动作,actionCreator可以返回函数，可以把dispatch的权限交给此函数
```
// action
export default {
  add(amount){
    return function (dispatch,getState) {
      dispatch({type:Types.ADD,amount});
      dispatch({type:Types.ADD,amount});
      console.log(getState().number);
    }
  },
  minus(amount){
    return {type:Types.MINUS,amount}
  }
}
// store/index.js
let reduxThunk = store => dispatch => action=>{
  if(typeof action === 'function'){ //如果是函数将派发的权限传递给函数
    return action(dispatch,store.getState);
  }
  dispatch(action);
};
```

### 3.实现redux-promise中间件
```
// action
minus(amount){
    return {
      type:Types.MINUS,
      payload:new Promise(function (resolve,reject) {
        reject({amount:2});
      })
    }
}
//store/index.js
let reduxPromise = store => dispatch => action=>{
  if(action.then){
    return action.then(dispatch); //只支持成功
  }else if(action.payload&&action.payload.then){
    // 如果payload是一个promise 会对成功和失败都进行捕获并且将成功或失败的数据放到payload中进行派发
    return action.payload.then(function (data) {
      dispatch({...action,payload:data});
    },function (data) {
      dispatch({...action,payload:data});
    })
  }
  dispatch(action);
};
```

### 4.compose应用
```
function toResult(who,decorator) {
  return who+decorator;
}
function len(str) {
  return str.length
}
// 我们的目的是将第一个函数的返回结果传递给第二个函数
console.log(len(toResult('Mrs jiang', '很帅')));
```

#### 实现compose
这个compose也是redux中的一个方法
```
let compose = (...fns)=>(...args)=> {
  let last = fns.pop();
  return fns.reduceRight(function (prev,next) {
      return next(prev);
  },last(...args))
};
console.log(compose(len, toResult)('Mrs jiang', '很帅'));
```

### 5.applyMiddleware实现
```
let applyMiddleWare = (...middlewares) => createStore => reducer =>{
  let store = createStore(reducer);
  let middles = middlewares.map(middleware=>middleware(store))
  let dispatch = compose(...middles)(store.dispatch);
  return {
    ...store,
    dispatch
  }
};
```

### 6.简化applyMiddleWare应用
最终实现效果
```
export default createStore(reducer,applyMiddleware(reduxThunk,reduxPromise));
```
最终版redux库
```
let createStore = (reducer, fn) => {
    let state;
    let listeners = [];
    let getState = () => state;
    let dispatch = (action) => {
        state = reducer(state, action);
        listeners.forEach(item => item());
    };
    dispatch({});
    let subscribe = (l) => {
        listeners.push(l);
        return () => {
            listeners = listeners.filter(item => item !== l);
        }
    };
    if (typeof fn === 'function') {
        return fn(createStore, reducer);
    }
    return {
        createStore,
        dispatch,
        getState,
        subscribe
    }
};
let combineReducers = (reducers) => (newState = {}, action) => {
    for (let key in reducers) {
        newState[key] = reducers[key](newState[key], action)
    }
    return newState;
};
let bindActionCreators = (actions, dispatch) => {
    let obj = {};
    for (let key in actions) {
        obj[key] = (...args) => dispatch(actions[key](...args))
    }
    return obj
};
let applyMiddleware = (...middlewares) => (createStore, reducer) => {
    let store = createStore(reducer);
    let middles = middlewares.map(middleware => middleware(store));
    let dispatch = compose(...middles)(store.dispatch);
    return {
        ...store,
        dispatch
    }
};
let compose = (...fns) => {
    return (...args) => {
        let fn = fns.pop();
        return fns.reduceRight((prev, next) => {
            return next(prev);
        }, fn(...args));
    }
};
export {createStore, combineReducers, bindActionCreators, applyMiddleware, compose}
```


