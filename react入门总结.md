# React 入门总结

# 目录

1. 构建配置
2. React组件、css module
3. React Router 使用
4. Redux Redux 使用
5. 注意问题
6. 资料整理


# 一、构建配置
## 1）使用 [react-app-rewired](https://github.com/timarney/react-app-rewired/blob/master/README_zh.md) 对 create-react-app 的默认配置进行自定义

1、 引入 react-app-rewired 并修改 package.json 里的启动配置。由于新的 react-app-rewired@2.x 版本的关系，你还需要安装 [customize-cra](https://github.com/arackaf/customize-cra)

```
yarn add react-app-rewired customize-cra babel-plugin-import less less-loader --dev
```

```json
/* package.json */
"scripts": {
-   "start": "react-scripts start",
+   "start": "react-app-rewired start",
-   "build": "react-scripts build",
+   "build": "react-app-rewired build",
-   "test": "react-scripts test",
+   "test": "react-app-rewired test",
}
```

2、根目录创建 config-overrides.js
```js
const path = require('path');
const { override, fixBabelImports, addLessLoader, addWebpackAlias } = require('customize-cra');

const addWebpackConfig = () => (config) => {
  if (process.env.NODE_ENV === 'production') {
    config.devtool = false
    config.output.publicPath = '//static.kuaizi.co/super-recommend/'
  }

  return config
}

module.exports = override(
  // 按需加载
  fixBabelImports('lodash', {
    libraryDirectory: '',
    camel2DashComponentName: false
  }),
  // 按需加载
  fixBabelImports('import', {
    libraryName: 'antd',
    libraryDirectory: 'es',
    style: true
  }),
  addLessLoader({
    noIeCompat: true,
    javascriptEnabled: true,
    localIdentName: '[local]--[hash:base64:5]', // 开启less module
    modifyVars: { // less 变量
      '@primary-color': '#0999aa',
      '@success-color': '#45A767',
      '@layout-header-background': '#0999aa'
    }
  }),
  addWebpackAlias({
    "@": path.resolve(__dirname, "src")
  }),
  addWebpackConfig()
)
```

## 2）PUBLIC_URL
1、添加环境变量到build, dev访问需要配合proxy
```
/* package.json */
"scripts": {
  "build": "PUBLIC_URL=//xxx.cn react-app-rewired build",
}
```

2、使用

index.html
```html
<script src="%PUBLIC_URL%/common/js/utils.js"></script>
```

JavaScript
```js
render() {
    return <img src={`${process.env.PUBLIC_URL}/common/img/logo.png`} />;
}
```

## 3）proxy
1 安装http-proxy-middleware依赖
```
yarn add http-proxy-middleware --dev
```

2 在src目录下新建 setupProxy.js

```js
const proxy = require("http-proxy-middleware");

module.exports = function(app) {
  app.use(
    // api代理
    proxy("/api", {
      target: "http://xxx.cn",
      secure: false,
      changeOrigin: true,
      pathRewrite: {
         "^/api": ""
      }
    }),

    // cnd资源代理
    proxy("/common", {
      target: "http://xxx.cn",
      secure: false,
      changeOrigin: true
    })
  );
};
```

# 二、React组件
## 1)有状态组件 class component
```jsx
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}

```

## 2)无状态函数组件
```jsx
import PropTypes from 'prop-types'
const Example = props => {
  const { count, onClick } = props;
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={onClick}></button>
    </div>
  )
}

Example.propTypes = {
  count: PropTypes.number,
  onClick: PropTypes.func,
}
Example.defaultProps = {
  count: 0,
  onClick: (() => {})
}
```

## 3) HOC高阶组件
HOC（High Order Component） 是 react 中对组件逻辑复用部分进行抽离的高级技术，但HOC并不是一个 React API 。 它只是一种设计模式，类似于装饰器模式。

在 Vue 中通常我们采用: mixins

```jsx
const dataStorageHoc = WrappedComponent => {
  return class extends Component{
    constructor(props){
      super(props)
      this.myRef = React.createRef()
    }
    componentWillMount() {
      const data = localStorage.getItem('data')
      this.setState({ data })
    }

    render() {
      return <WrappedComponent data={this.state.data} {...this.props} /> 
    }
  }
}
```

使用
```jsx
import React, { Component } from 'react'
import dataStorageHoc from '@/lib/hoc/data-storage.jsx'

class HomePage extends Component{
  render() {
    return <h2>{this.props.data}</h2>
  }
}

export default dataStorageHoc(HomePage)

// 装饰器(decorator)模式
@dataStorageHoc
class HomePage extends Component{
  render() {
    return <h2>{this.props.data}</h2>
  }
}

export default HomePage
```

## 4) hooks是有状态的函数
hook作用: <b>将可复用的最小单元从组件层面进一步细化到逻辑层面。状态逻辑和UI是解耦的。<b>

```jsx
import { useState } from 'react'
const Example = () => {
    const [count, setCount] = useState(0);
    return (
      <div>
        <p>You clicked {count} times</p>
        <button onClick={() => setCount(count + 1)}>
          Click me
        </button>
      </div>
    );
}
```

## 5) 自定义hooks
```jsx
function useCount(initialValue = 0) {
  const [count, setCount] = useState(initialValue)
  useEffect(() => {
    service.getInitialCount().then(data => {
      setCount(data)
    })
    return () => {
     console.log('计数完成')
    }
  }, [])
  function addCount() {
    setCount(c => c + 1)
  }
  return { count, addCount }
}

function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth)

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth)

    window.addEventListener('resize', handleResize)

    return () => {
      window.removeEventListener('resize', handleResize)
    }
  })

  return width
}

const App = () => {
  const { count, addCount } = useCount(0)
  const width = useWindowWidth()

  const onChange = (e) => {
    handleNameChange(e.target.value)
  }
  
  return (
    <div>
      <p>window width {width}</p>
      <p>You clicked {count} times</p>
      <button onClick={addCount}>Click me</button>
    </div>
  )
}
```

## 6) css动态样式功能

[classnames](https://github.com/JedWatson/classnames)

1、安装
```
yarn add classnames
```

2、使用方法
```jsx
import classnames from 'classnames'
 
<div className=classnames({
    'class1': true,
    'class2': true
    )>
</div>

// 其他类型
classNames('foo', 'bar'); // => 'foo bar'
classNames('foo', { bar: true }); // => 'foo bar'
classNames({ 'foo-bar': true }); // => 'foo-bar'
classNames({ 'foo-bar': false }); // => ''
classNames({ foo: true }, { bar: true }); // => 'foo bar'
classNames({ foo: true, bar: true }); // => 'foo bar'
```

## 7) less module

* 文件名规则 xxx.module.less， 必须以.module.less结尾
* 命名：可以使用驼峰或者连接线命名

index.module.less

```less
.list {
  height: 100%;
  overflow-y: auto;
}
.listItem {
  height: 50px;
}

.itemActive {
  color: aqua;
}

.item-disabled {
  color: #999;
}
```

使用样式

```jsx
import classNames from 'classnames'
import styles from './index.module.less'

const Example = props => {
  return (
    <ul class={styles.listScroll)}>
      <li className={classNames('item', styles.listItem, {styles.itemActive : true})}>...</li>

      <li className={classNames('item', styles.listItem, {styles['item-disabled'] : true})}>...</li>
    </ul>
  )
}
```


# 三、React Router

## 1） 安装
创建 Web应用，使用
```
yarn add react-router-dom
```
创建 navtive 应用，使用
```
yarn add react-router-native
```

## 2) 路由模式

BrowserRouter模式 创建的 URL 形式如下：

```css
http://example.com/some/path
```

HashRouter模式 创建的 URL 形式如下：
```css
http://example.com/#/some/path
```

## 3) Route组件

\<Route\>组件是react router v4里最有用的组件。无论何时你需要在匹配某个路径的时候绘制一个组件，那么就可以使用Route组件。

Route组件可以使用如下的属性：
* path属性，字符串类型，它的值就是用来匹配url的。
* component属性，它的值是一个组件。在path匹配成功之后会绘制这个组件。
* exact属性，这个属性用来指明这个路由是不是排他的匹配 (绝对匹配)。
* strict属性， 这个属性指明路径只匹配以斜线结尾的路径。

还有其他的一些属性，可以用来代替component属性
* render属性，一个返回React组件的方法，只要你的路由匹配了，这个函数才会执行
* children属性，返回一个React组件的方法。只不过这个总是会绘制，即使没有匹配的路径的时候。

使用component
```jsx
<Route exact path="/" component={HomePage} />
```

使用render
```jsx
<Route path="/" render={(props) => (
    <HomePage {...props} />
)} />
```

使用children
```jsx

<Route path="/" children={(props) => (
    <div>children</div>
)} />
```

exact
```jsx
http://example.com/#/path
<Route exact path="/" component={HomePage} />  // 不匹配
```

非exact
```jsx
http://example.com/#/path
<Route  path="/" component={HomePage} />  // 匹配
```

## 4）Switch组件

只有第一个匹配的路由\<Route>或者\<Redirect>会被绘制，匹配到一个就不再匹配

```jsx
import { Switch, Route } from 'react-router-dom'
<Switch>
  <Route exact path="/" component={HomePage} />
  <Route path="/about" component={AboutPage} />
  <Route component={NotFound} />
</Switch>
```

## 5）Link、NavLink、Redirect组件

Link组件需要用到to属性，这个属性的值就是react router要跳转到的地址

NavLink是Link的一个子类，在Link组件的基础上增加了绘制组件的样式
```js
import { Link, NavLink } from 'react-router-dom'

<Link to="/">Home</Link>

<Link to={{
  pathname: '/posts',
  search: '?sort=name',
  hash:'#the-hash',
  state: { fromHome: true}
}}></Link>

<NavLink to="/me" activeStyle={{SomeStyle}} activeClassName="selected">
  My Profile
</NavLink>

// 重定向
<Redirect to="/register" />
```

除了使用Link外，我们还可以使用 history 对象手动实现导航。history 中最常用的两个方法是 push(path,[state]) 和 replace(path,[state]),push会向浏览器记录中新增一条记录，replace 会用新记录替换记录。
```js
history.push('/posts')
history.replace('/posts')
```

## 6）Router Cache

>原理：display "block" "none"

1、安装 react-router-cache-route
```
yarn add react-router-cache-route --dev
```

2、使用
```jsx
import { Route } from 'react-router-dom'
import CacheRoute, { CacheSwitch } from 'react-router-cache-route'

const App = () => {
  return (
    <CacheSwitch>
      <CacheRoute path="/login" exact component={LoginPage} />
      <CacheRoute path="/register" exact component={RegisterPage} />
      <Route component={NotFoundPage} />
    </CacheSwitch>
  )
}
```

## 7）例子

```jsx
// app.jsx

import { HashRouter, Route, Switch } form 'react-router-dom'

const MenuLayout = ({ location }) => (
  <div className="layout">
    <header>
      <p>React Router v4 Browser Example</p>
      <nav>
        <ul>
          <li><Link to="/menu/user">User</Link></li>
          <li><Link to="/menu/order">Order</Link></li>
        </ul>
      </nav>
    </header>
    <div className="container">
      <Switch>
        <Route path="/menu/user" exact component={AboutPage} />
        <Route path="/menu/order" exact component={ProfilePage} />
        <Route render={() => <div>404 Not Found</div>} />
      </Switch>
    </div>
    <footer>
      React Router v4 Browser Example (c) 2017
    </footer>
  </div>
)

const App = () => {
  return (
    <HashRouter>
      <Route path="/login" exact component={LoginPage} />
      <Route path="/register" exact component={RegisterPage} />
      // 不要使用 exact
      <Route path="/menu" component={MenuLayout}></Route>
      <Route component={NotFoundPage} />
    </HashRouter>
  )
}

export default App
```

# 四、 Redux Redux

## 1) 定义reducer
redux/action-type.js
```js
export const SET_USER_INFO = 'SET_USER_INFO'
export const SET_USER_LIST = 'SET_USER_LIST'
export const SET_NEWS_LIST = 'SET_NEWS_LIST'
```

redux/actions.js

```js
import { SET_USER_INFO, SET_NEWS_LIST, SET_USER_LIST } from './action-type'

export const setUserInfo = (data) => {
  return {
    type: SET_USER_INFO,
    data
  }
}

export const setUserList = (data) => {
  return {
    type: SET_USER_LIST,
    data
  }
}

export const setNewsList = (data) => {
  return {
    type: SET_NEWS_LIST,
    data
  }
}
```

redux/reducers/user.js
```js
import { SET_USER_INFO, SET_USER_LIST } from './action-type'

const initialState = {
  userInfo: null,
  userList: []
}
const reducer = (state = initialState, action) = {
  switch(action.type){
    case SET_USER_INFO:
      return Object.assign({}, state, {
        userInfo: action.data
      })
    case SET_USER_LIST:
      return Object.assign({}, state, {
        userList: action.data
      })  
    default:
      return state 
  }
}

export default reducer
```

redux/reducers/news.js
```js
import { SET_NEWS_LIST } from './action-type'

const initialState = {
  newsList: [],
  total: 0
}
const reducer = (state = initialState, action) = {
  switch(action.type){
    case SET_NEWS_LIST:
      return Object.assign({}, state, {
        newsList: action.data.list,
        total: action.data.total
      })
    default:
      return state  
}
export default reducer
```

## 2) 合并reducer redux/reducers/index.js
```js
import {combineReducers} from 'redux'
import user from './user'
import news from './news'

export default combineReducers({
  user,
  news
})
```

## 3)创建store redux/index.js
```js
import {createStore} from 'redux'
import reducers from './reducers'

// 创建store对象
const store = createStore(reducers)

export default store
```

## 4）redux connect, 将react与redux关联起来

connect()接收四个参数，它们分别是mapStateToProps，mapDispatchToProps，mergeProps和options。

* mapStateToProps 将state映射到 UI 组件的参数（props）

* mapDispatchToProps 将action映射到 UI 组件的参数（props）

* mergeProps 选项，如果指定，则定义如何确定自己包装的组件的最终道具。如果不提供mergeProps，则包装的组件默认会收到{...ownProps，...stateProps，...dispatchProps}

* options 选项
  * [forwardRef = false] 如果已传递{forwardRef：true}进行连接，则向连接的包装器组件添加ref实际上将返回被包装组件的实例。默认为false

  * ...

```
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])
```

## 5) 使用redux

index.js
```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'
import store from './redux'
import App from './App'

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>
  document.getElementById('root')
);
```

App.js

```js
import { connect } from "react-redux"
import { setUserInfo } from '@/redux/action'

const App = (props) => {
  const { userInfo, setUserInfo, newsList } = props

  const onClick = () => {
    setUserInfo({
      name: 'oo'
    })
  }

  return (
    <div>
      <p>{userInfo.name}</p>
      <button onClick={onClick}>click</button>
      <ul>
        {
          newsList.map((o) => {
            return <li>{o.title}</li>
          })
        }
      </ul>
    </div>
  )
}

App.defaultProps = {
  userInfo: {},
  newsList: []
}

const mapStateToProps = (state) => ({
  userInfo: state.user.userInfo,
  newsList: state.news.newsList
})

const mapDispatchToProps = (dispatch) => ({
  setUserInfo: (data) => dispatch(setUserInfo(data))
})

export default connect(
  mapStateToProps,
  mapDispatchToProps,
  null,
  {forwardRef: true}
)(App)

```

# 五、注意问题

## 1）ant design Form.create 之后如果拿不到 ref

经过 Form.create 之后如果要拿到 ref，可以使用 rc-form 提供的 wrappedComponentRef，[详细内容可以查看这里](https://github.com/react-component/form#note-use-wrappedcomponentref-instead-of-withref-after-rc-form140)。

```jsx
class CustomizedForm extends React.Component { ... }

// use wrappedComponentRef
const EnhancedForm =  Form.create()(CustomizedForm);


class Example extends React.Component {
  render () {
    return (
     <EnhancedForm wrappedComponentRef={(form) => this.form = form} />
    )
  }
}
```

## 2) redux connect 之后拿不到ref

connect 之后拿不到ref，配置options.forwardRef=true

```jsx
class Example extends React.Component { ... }

export default connect(
  mapStateToProps,
  mapDispatchToProps,
  null,
  {forwardRef: true}
)(Example)
```

# 资料整理
[react](https://github.com/facebook/react)

[create-react-app](https://github.com/facebook/create-react-app)

[react-app-rewired](https://github.com/timarney/react-app-rewired)

[customize-cra](https://github.com/arackaf/customize-cra)

[react-router](https://github.com/ReactTraining/react-router)

[react-redux](https://github.com/reduxjs/react-redux)

[classnames](https://github.com/JedWatson/classnames)
