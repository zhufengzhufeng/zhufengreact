## 学习
学习要自信,学技术的目的是让我们更有自信。最害怕的事是越学越觉得恐惧。觉得自己什么都不会。

不要忘记你努力的时候别人也再努力，但是你不努力就会被落下很远!

## React基础篇
### 1.什么是React?
- React 是一个用于构建用户界面的JavaScript库
- 核心专注于视图,目的实现组件化开发

### 2.组件化的概念
我们可以很直观的将一个复杂的页面分割成若干个独立组件,每个组件包含自己的逻辑和样式 再将这些独立组件组合完成一个复杂的页面。 这样既减少了逻辑复杂度，又实现了代码的重用
- 可组合：一个组件可以和其他的组件一起使用或者可以直接嵌套在另一个组件内部
- 可重用：每个组件都是具有独立功能的，它可以被使用在多个场景中
- 可维护：每个小的组件仅仅包含自身的逻辑，更容易被理解和维护

*https://pan.baidu.com/s/1hsivfN2*

### 3.跑通react开发环境
```
$ npm install create-react-app -g
$ create-react-app <project-name>
$ cd <project-name> && npm start
```

> 默认会自动安装React,react由两部分组成,分别是:

- react.js 是 React 的核心库
- react-dom.js 是提供与DOM相关的功能,会在window下增加ReactDOM属性,内部比较重要的方法是render,将react元素或者react组件插入到页面中。

### 4.简介JSX
- 是一种JS和HTML混合的语法,将组件的结构、数据甚至样式都聚合在一起定义组件,会编译成普通的Javascript。

> 需要注意的是JSX并不是html,在JSX中属性不能包含关键字，像class需要写成className,for需要写成htmlFor,并且属性名需要采用驼峰命名法！

### 5.createElement
JSX其实只是一种语法糖,最终会通过[babel](https://babeljs.io/repl/)转译成createElement语法,以下代码等价
```
ReactDOM.render(<div>姜,<span>帅哥</span></div>);
ReactDOM.render(React.createElement("div",null,"姜,",React.createElement("span",null,"帅哥")));
```

> 我们一般使用React.createElement来创建一个虚拟dom元素。

### 6.react元素/JSX元素
```
function ReactElement(type,props) {
    this.type = type;
    this.props = props;
}
let React = {
    createElement(type,props={},...childrens){
        childrens.length===1?childrens = childrens[0]:void 0
        return new ReactElement(type,{...props,children:childrens})
    }
};
```

> ReactElement就是虚拟dom的概念，具有一个type属性代表当前的节点类型，还有节点的属性props

### 7.模拟render实现
```
let render = (eleObj,container)=>{
    // 先取出第一层 进行创建真实dom
    let {type,props} = eleObj;
    let elementNode = document.createElement(type); // 创建第一个元素
    for(let attr in props){ // 循环所有属性
        if(attr === 'children'){ // 如果是children表示有嵌套关系
            if(typeof props[attr] == 'object'){ // 看是否是只有一个文本节点
                props[attr].forEach(item=>{ // 多个的话循环判断 如果是对象再次调用render方法
                    if(typeof item === 'object'){
                        render(item,elementNode)
                    }else{ //是文本节点 直接创建即可
                        elementNode.appendChild(document.createTextNode(item));
                    }
                })
            }else{ // 只有一个文本节点直接创建即可
                elementNode.appendChild(document.createTextNode(props[attr]));
            }
        }else if(attr === 'className'){ // 是不是class属性 class 属性特殊处理
            elementNode.setAttribute('class',props[attr]);
        }else{
            elementNode.setAttribute(attr,props[attr]);
        }
    }
    container.appendChild(elementNode)
};
```

### 8.JSX表达式的用法
- 1) 可以放JS的执行结果
- 2) 如果换行需要用()包裹jsx代码
- 3) 可以把JSX元素当作函数的返回值
- 4) <{来判断是表达式还是js

```
import React from 'react';
import ReactDOM from 'react-dom';

function toResult({name,age}) {
    return <span>今年{name},{age}岁了!</span>
}
let arrs =  [{name:'zfpx',age:8},,{name:'姜文',age:28}];
ReactDOM.render(<div>
    {arrs.map(((item,index)=>(
        typeof item==='object'?<li key={index}>{toResult(item)}</li>:null
    )))}
</div>,document.getElementById('root'));
```

> null也是合法元素,循环时需要带key属性

### 9.JSX属性
- 在JSX中分为普通属性和特殊属性，像class要写成className,for要写成htmlFor
- style要采用对象的方式
- dangerouslyInnerHTML插入html

### 10.组件的特点声明方式
react元素是是组件组成的基本单位
- 首字母必须大写,目的是为了和JSX元素进行区分
- 组件定义后可以像JSX元素一样进行使用
- 每个组件必须返回唯一的顶级JSX元素
- 可以通过render方法将组件渲染成真实DOM

### 11.组件的两种定义方式
react怎么区分是组件还是jsx元素？组件名需要开头大写，react组件当作jsx来进行使用
- 第一种方式是函数声明
```
function Build(props) {
    return <p>{props.name} {props.age}</p>
}
render(<div>
    <Build name={school1.name} age={school1.age}/>
    <Build {...school2} />
</div>,window.root);
```

- 第二种方式是类声明
```
class Build extends Component{
    render(){
        let {name,age} = this.props;
        return <p>{name} {age}</p>
    }
}
```

> 类声明有状态，this，和声明周期

### 12.组件中属性和状态的区别
- 组件的数据来源有两个地方
    - props 外界传递过来的(默认属性，属性校验)
    - state 状态是自己的,改变状态唯一的方式就是setState

> 属性和状态的变化都会影响视图更新

### 13.绑定事件
- 给元素绑定事件，事件绑定方式
```
class Clock extends Component {
    constructor(){
        super();
        this.state = {date:new Date().toLocaleString()}
    }
    componentDidMount(){ //组件渲染完成，当渲染后会自动触发此函数
        this.timer = setInterval(()=>{ // 箭头函数 否则this 指向的是window
            this.setState({date:new Date().toLocaleString()})
        },1000);
    }
    componentWillUnmount(){ //组件将要卸载，当组件移除时会调用
        clearInterval(this.timer); //一般在这个方法中 清除定时器和绑定的事件
    }
    destroy=()=>{ //es7 箭头函数
        // 删除某个组件
        ReactDOM.unmountComponentAtNode(window.root);
    }
    render(){
        // 给react元素绑定事件默认this是undefined,bind方式 在就是箭头函数
        return <h1 onClick={this.destroy}>{this.state.date}</h1>
    }
}
// 执行顺序 constructor -> render -> componentDidMount -> setState-> render - onClick-> unmountComponentAtNode -> componentWillUnmount -> clearInterval
ReactDOM.render(<Clock/>,window.root);

```

> 给jsx元素绑定事件要注意事件中的this指向，事件名采用 on+"开头大写事件名"的方式

### 14.属性校验,默认属性
```
import React from 'react';
import ReactDOM from 'react-dom';
import PropTypes from 'prop-types'; //引入属性校验的模块
class School extends React.Component{ // 类上的属性就叫静态属性
    static propTypes = { // 校验属性的类型和是否必填
        age:PropTypes.number.isRequired, // 支持的类型可以参考prop-types的readme文件
    };
    static defaultProps = { // 先默认调用defaultProps
        name:'珠峰',
        age:1
    }; // 默认属性
    constructor(props){ //如果想在构造函数中拿到属性需要通过参数的方式
         //不能在组件中更改属性 不能修改属性*
        super();
    }
    render(){
        return <h1>{this.props.name} {this.props.age}</h1>
    }
}
```

> propTypes和defaultProps名字不能更改，这是react规定好的名称

### 15.状态的使用
- setState方法的应用
- 状态操作的合并

### 16.复合组件
- 组件的多次复用
- 拆分panel组件

### 17.受控组件和非受控组件

### 18.声明周期

### 19.评论面板

### 20.React实现百度搜索框

### 21.react轮播图考试