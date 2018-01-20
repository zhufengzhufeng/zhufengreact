## 路由的使用
本章我们来介绍react中路由的使用,现在使用的路由是React-Router-4版本,我们需要下载的包叫react-router-dom

### 安装
```
npm install react-router-dom
```

> 这里我们下载好后需要他内部的路由容器组件,主要包含BrowserRouter,HashRouter,MemoryRouter

### 容器组件的区别
- BrowserRouter: 浏览器自带的H5 API,restful风格,需要配合后台；
- HashRouter: 使用hash方式进行路由,路径后均有#；
- MemoryRouter: 在内存中管理history，地址栏不会变化。在reactNative中使用。

> 在开发时我们一般使用HashRouter,上线后我们改用BrowserRouter

### 跑通基本路由
我们来声明三个组件Home,User,Profile希望访问不同的路径可以实现显示不同的组件
```
import React from 'react';
import ReactDOM from 'react-dom';
import {HashRouter,Route} from 'react-router-dom';
let Home = () =><div>home</div>
let Profile = () =><div>Profile</div>
let User = () =><div>User</div>
ReactDOM.render(<HashRouter>
    <div>
        <Route path="/home" component={Home}/>
        <Route path="/profile" component={Profile}/>
        <Route path="/user" component={User}/>
    </div>
</HashRouter>,document.querySelector('#root'));
```

> 这里我们使用了HashRouter,代码一目了然,这里用到了Route组件,Route组件上有path和component属性,对应的path会显示对应的component,这里还需注意HashRouter必须只能包含一个根元素,所以我们在所有的Route外层包了一个div标签

### 路由的匹配
```
<div>
    <Route path="/home" component={Home}/>
    <Route path="/profile" component={Profile}/>
    <Route path="/profile/user" component={User}/>
</div>
```

> 这里我们稍作了下更改当访问/profile/user时你会发现/profile路由也会命中。所以说明只要路径开头匹配成功既会显示对应的组件,假如你希望不管访问任何路径时都能显示某一个组件你可以将path写成'/'


### Link组件
我们先来引入bootstrap,将刚才的代码逻辑进行拆分,增加导航条点击不同的导航显示不同的组件。
```
import React from 'react';
import ReactDOM from 'react-dom';
import {HashRouter,Route} from 'react-router-dom';
import 'bootstrap/dist/css/bootstrap.css';
import App from './App';
import Home from './components/Home';
import Profile from './components/Profile';
import User from './components/User';
ReactDOM.render(<HashRouter>
    <App>
        <Route path="/home" component={Home}/>
        <Route path="/profile" component={Profile}/>
        <Route path="/user" component={User}/>
    </App>
</HashRouter>,document.querySelector('#root'));
```

> 这里我们增加了App组件,为什么这样做呢?原因是我们并不希望将所有的逻辑都写在index中,这并不方便我们的管理,还记得children属性吗？我们可以直接在App中通过children的方式引入


### 增加导航
```
import React from 'react';
import {Link} from 'react-router-dom';
export default class App extends React.Component{
    render(){
        return (
            <div>
                <div className="navbar-inverse navbar">
                    <div className="container-fluid">
                        <div className="navbar-header">
                            <div className="navbar-brand">
                                用户管理系统
                            </div>
                        </div>
                        <ul className="navbar-nav nav">
                            <li>
                                <Link to={'/home'}>首页</Link>
                            </li>
                            <li>
                                <Link to={'/user'}>用户管理</Link>
                            </li>
                            <li>
                                <Link to={'/profile'}>个人中心</Link>
                            </li>
                        </ul>
                    </div>
                </div>
                <div className="container">
                    <div className="row">
                        <div className="col-md-12">
                            {this.props.children}
                        </div>
                    </div>
                </div>
            </div>
        )
    }
}
```

> 这里使用Link组件它可以替代我们自己写的a标签,因为后面我们可能会用到browserHistory，跳转可能需要用到H5的api进行跳转,Link组件是react路由中提供的声明式组件,可以帮我们区分路由的模式来实现路由的跳转。

### 页面级组件
这里我们先将对应的组件代码代码贴出来,后面我们来完善对用的逻辑
```
// Home.js
import React from 'react';
export default class Home extends React.Component{
    render(){
        return (
            <div>Home</div>
        )
    }
}
// Profile.js
import React from 'react';
export default class Profile extends React.Component{
    render(){
        return (
            <div>Profile</div>
        )
    }
}
// User.js
import React from 'react';
export default class User extends React.Component{
    render(){
        return (
            <div>User</div>
        )
    }
}
```



### 二级路由
刚才我们实现了一级导航,但是一般情况下管理系统都会拥有二级导航,比如说我们希望用户管理中包含用户列表和添加用户,这两个菜单应该属于用户列表下的子导航,我们先来看下效果

#### 实现二级路由
二级导航就是在某个一级路由中继续嵌套路由
```
import React from 'react';
import {Route,Link} from 'react-router-dom'
import UserList from './UserList'
import UserAdd from './UserAdd'
export default class User extends React.Component{
    render(){
        return (
            <div className="row">
                <div className="col-md-2">
                    <div className="nav nav-stacked">
                        <li><Link to='/user/list'>用户列表</Link></li>
                        <li><Link to='/user/add'>增加用户</Link></li>
                    </div>
                </div>
                <div className="col-md-10">
                    <Route path="/user/list" component={UserList}/>
                    <Route path="/user/add" component={UserAdd}/>
                </div>
            </div>
        )
    }
}
```

> 这里并没有什么需要注意的新用法,只是路径要特殊处理一下因为是二级路由,要保证一级路由也同时显示所以开头要和一级路由路径相同。如果多次点击相同路由时会触发`Hash history cannot PUSH the same path;`这样的一个警告,这个是无法去除的但是改成BrowserHistory就不会触发此警告了。所有不用担心.


#### UserList和Add组件
```
// UserList.js
import React from 'react';
export default class UserList extends React.Component{
    render(){
        return (
            <div>UserList</div>
        )
    }
}
// UserAdd.js
import React from 'react';
export default class UserAdd extends React.Component{
    render(){
        return (
            <div>UserAdd</div>
        )
    }
}

```

> 这里我们先不写任何逻辑，先将具体的功能实现出来。这样我们就实现了二级路。



### 路由跳转和路径参数
本节我们完善一下内部的逻辑,进入到添加列表页可以实现用户的添加并且可以跳转到列表页面渲染出添加的用户列表,页面间的通信我们采用localStorage。并且点击某个用户可以进入到用户详情页。

#### 实现添加用户
```
import React from 'react';
export default class UserAdd extends React.Component{
    handleSubmit = (e) =>{
        e.preventDefault();
        let localStr = localStorage.getItem('lists');
        let list = JSON.parse(localStr)|| [];
        list.push({id:Math.random(),name:this.name.value});
        localStorage.setItem('lists',JSON.stringify(list));
        this.props.history.push('/user/list')
    };
    render(){
        return (
            <div>
                <form onSubmit={this.handleSubmit}>
                    <div className="form-group">
                        <label htmlFor="name" className="control-label">
                            用户名:
                        </label>
                        <input type="text" className="form-control" ref={x=>this.name = x}/>
                    </div>
                    <div className="form-group">
                        <button className="btn btn-primary">添加</button>
                    </div>
                </form>
            </div>
        )
    }
}
```

> 这里我们在props中使用了history的API,所有通过路由渲染的组件都拥有一些路由的属性后面我们会一一介绍到。这里我们采用编程式的方式跳转了路径。

#### 列表页展示
```
import React from 'react';
import {Link} from 'react-router-dom'
export default class UserList extends React.Component{
    constructor(){
        super();
        this.state = {}
    }
    componentWillMount(){
        let userList = JSON.parse(localStorage.getItem('lists'));
        this.setState({
            userList
        })
    }
    render(){
        return (
            <div>
                <ul className="list-group">
                    {this.state.userList.map((user,index)=>(
                        <li className="list-group-item" key={index}>
                            <Link to={"/user/detail/"+user.id}>{user.name}</Link>
                        </li>
                    ))}
                </ul>
            </div>
        )
    }
}
```

> 这里的逻辑比较简单,我们又添加了一个detail路径同样也是一个二级路由，点击用户名可以显示具体的用户id和用户名。

#### 引入UserDetail组件
```diff
+ import UserDetail from './UserDetail';
  <Route path="/user/list" component={UserList}/>
  <Route path="/user/add" component={UserAdd}/>
+ <Route path="/user/detail/:id" component={UserDetail}/>
```

> 这里我们采用了模糊匹配的方式,这就是我们常说的路径参数。也就是说id可以代表任何值，我们可以在详情页中通过this.props.match.params.id获取到真实传入的id

#### UserDetail详情页
```
import React from 'react';
export default class UserDetail extends React.Component{
    render(){
        let {id } = this.props.match.params;
        let {name} = JSON.parse(localStorage.getItem('lists')).find(item=>item.id == id);
        return (
            <table className="table table-bordered">
                <thead>
                    <tr>
                        <th>id号</th>
                        <th>标题</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>{id}</td>
                        <td>{name}</td>
                    </tr>
                </tbody>
            </table>
        )
    }
}
```

> 这里要注意通过params取出的结果都是字符串类型,所有匹配到的参数都会放在match.params的属性上。


### 路由匹配
有的时候我们希望对路由匹配有些限制,比如说严格对某个路径进行匹配,或者匹配到某个路径时就不在匹配

#### 新增路由匹配
```diff
<App>
+       <Route path="/" render={()=><h1>首页</h1>}/>
+       <Route path="/:name" render={()=><h1>zfpx</h1>}/>
        <Route path="/home" component={Home}/>
        <Route path="/profile" component={Profile}/>
        <Route path="/user" component={User}/>
</App>
```

> 在此我们会发现当访问/home时以上`首页`和`zfpx`和`home组件`都会访问到,而我们只希望在访问/时才会显示首页,我们可以在某个Route上增加exact属性

#### exact
```
<Route path="/" exact render={()=><h1>首页</h1>}/>
```

> 你会发现这样就实现啦~,只会当访问/时才可以匹配到。

#### Switch
但是现在访问/home时我们依然有两个组件会被匹配到，我们希望匹配到一个后就停止匹配，不在继续匹配下一个路由，我们可以使用Switch组件
```
import {HashRouter,Route,Switch} from 'react-router-dom';
<Switch>
    <Route path="/" exact render={()=><h1>首页</h1>}/>
    <Route path="/:name" render={()=><h1>首页</h1>}/>
    <Route path="/home" component={Home}/>
    <Route path="/profile" component={Profile}/>
    <Route path="/user" component={User}/>
</Switch>
```

> 现在访问/home你会发现Home组件永远都不会显示出来啦！

### 受保护的路由
我们想对一些路由进行屏蔽,例如登录后才能访问,这里我们在本地存一个变量来表示是否登录,增加一个登录路由，点击登录按钮将本地变量改为登录成功状态,即可以访问用户列表页面


### 受保护路由
我们匹配到/user路由时要根据状态判断是否有权限，如果没权限需要跳转到登录页面，主要靠的是高阶组件的思想来实现:

#### Protected组件
```
import React from 'react';
import {Route,Redirect} from 'react-router-dom'
export default ({component:Component,...others})=>{
   return <Route {...others} render={(props)=>{
       return localStorage.getItem('loginSystem')?<Component {...props}/>:<Redirect to={{
           pathname:'/login',
           from:props.match.url
       }}/>
   }}/>
}
```

> Redirect组件是用来重定向的，我们新增from属性来记录当前匹配的url,为了保证登录后可以在跳回到当前匹配的路径

#### 新增Login组件
```diff
<div className="container">
    <Route path="/home" component={Home}/>
-   <Route path="/profile" component={Profile}/>
+   <PrivateRoute path="/profile" component={Profile}/>
    <Route path="/user" component={User}/>
+   <Route path="/login" component={Login}/>
</div>

Login.js
import React from 'react';
export default class Login extends React.Component{
    render(){
        return (
            <div>
                <button className="btn btn-primary" onClick={()=>{
                    window.localStorage.setItem('loginSystem',true);
                   this.props.history.push(this.props.location.from)
                }}>登录</button>
            </div>
        )
    }
}
```

> 我们发现默认点击profile会默认跳转到login组件中，点击登录可以再次跳回profile。这样我们就实现了受保护的路由。

### 自定义菜单
我们想给点击后的菜单增加激活样式，同样依然采用高阶组件的方式进行包装

#### 实现MenuLink组件
此组件是用来替换掉原有的Link组件，并且在内部进行判断是否增加激活状态
```
import React from 'react';
import {Route,Link} from 'react-router-dom'
export default ({to,label})=>{
    return <Route children={(props)=>{
        return <li className={props.match?'active':''}><Link to={to}>{label}</Link></li>
    }}/>
}
```

> 这里有个children属性和以前render不同，children无论是否路由匹配到都会执行此函数。而render只要在匹配到后才会执行


### Prompt
我们希望在添加页的输入框中输入内容后点击其他路由时先询问一下是否需要跳转。
```
constructor(){
    super();
    this.state = {show:false}
}
<div>
    <Prompt when={this.state.show} message={location => (
        `Are you sure you want to go to ${location.pathname}?`
    )}/>
    <form onSubmit={this.handleSubmit}>
        <div className="form-group">
            <label htmlFor="username">用户名</label>
            <input type="text" className="form-control" ref={(x)=>this.x=x}
               onChange={(e)=>{
                   if(e.target.value.length>0){
                       this.setState({show:true})
                   }
               }
            />
        </div>
        <div className="form-group">
            <button className="btn btn-primary" >添加</button>
        </div>
    </form>
</div>
```

> 监听输入框中的内容,当有内容时将状态show变为true,当需要跳转路由时Prompt的组件when属性为true就会提示对应的message,当然我们点击添加时不需要弹出,所以先将状态改为false在进行跳转即可。

```diff
handleSubmit=(e)=>{
    e.preventDefault();
    let list = JSON.parse(localStorage.getItem('lists'))||[];
    list.push({id:Math.random(),name:this.x.value});
    localStorage.setItem('lists',JSON.stringify(list));
+   this.setState({show:false},()=>{
        this.props.history.push('/profile/list')
+   })
};
```

> 要等待状态改变后在执行跳转，因为setState是异步的所以要将跳转逻辑放的回调函数中。


## NotFound页面
我们需要当路由都匹配不到时显示一个404页面,增加一个404组件
```
import React from 'react';
export default class NotFound extends React.Component{
    render(){
        return (
            <div>NotFound</div>
        )
    }
}
```

### 新增404路由
```diff
<div className="container">
    <Switch>
        <Route path="/home" component={Home}/>
        <PrivateRoute path="/profile" component={Profile}/>
        <Route path="/user" component={User}/>
        <Route path="/login" component={Login}/>
+       <Route component={NotFound}/>
    </Switch>
</div>
```

> 这里我们使用switch组件当全部匹配不到时会默认渲染404路由,这样我们就实现了404页面
