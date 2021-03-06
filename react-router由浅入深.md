# React-router怎么实现侦听浏览器路由变化的？

**React Router是React官方提供的标准路由库。一直以来前端缺乏一个统一的路由解决方案，我们之前通常都是自己封装统一的路由，由于使用的框架不同，实现的方式也是五花八门。React Router的出现是伴随着React一起普及的，让我们开始吧，看看React Router完成的哪些工作，又做出了哪些创新。**

这个教程主要给你介绍React Router 的v4版本，以及你使用它可以做的大部分事情。我们在使用的inferno-router,基本也是沿用React-router的文档和API。

> Inferno Router is a routing library for Inferno. It is a port of react-router 4.

![React Set](https://ask.qcloudimg.com/http-save/yehe-1120945/pmlkf7y2i3.jpeg?imageView2/2/w/1620)

## 开始

熟悉`React`或者`Webpack`的应该都知道客户端渲染创建的单页应用（SPAs）。一个SPA会包含很多视图，对应`webpack`中配置的entry（也可以称为入口页面），和传统的多页应用不同的是，视图之间的跳转不应该导致整个页面被重新加载，而是应该就在当前页面里渲染。具象化来说是一个`portal`(入口)，里面包含多个页面view。习惯于多页应用的最终用户，期望在一个SPA中应该满足以下功能：

- 记录当前页面的状态（保存或分享当前页的url，再次打开该url时，网页还是保存（分享）时的状态）；

- 浏览器的前进后退功能

- 动态URL的适配。动态生成的URL使用嵌套视图来对应。 - 例如：`example.com/product/shoes/101`，101是动态的产品id。

作为开发者，要实现这几个功能，我们需要做到：
1. 改变url且不让浏览器向服务器发出请求
2. 监测url的变化
3. 截获url地址，并解析出需要的信息来匹配路由规则

**路由定义**是指在初始化React Router时，声明的路由项配置。
```
<Route path="/about" component={About}/>
```
这是一个最常见的Route配置，配置项非常语义化，`path`对应路由路径，`component`对应[React Component](https://react.docschina.org/docs/react-component.html)。你可以把`<Route>`组件放在任何你想要路由渲染的地方。当然，`<Route>`,`<Link>`以及其他React Router都已经封装成了组件，所以可以非常方便的在React里使用。

## 概述

React Router介绍的目录
1. 静态路由和动态路由
2. 基本路由
3. 嵌套路由和动态路由
4. History
5. 在服务端（SSR）的应用
6. 总结
7. 未来

主要围绕构建这些路由所涉及的概念进行讨论。

React Router库包含三个包： `react-router`, `react-router-dom`, 和 `react-router-native`。 `react-router`是路由的核心包，而其他两个是基于特定环境的。如果你在开发一个网站，你应该使用 `react-router-dom`，如果你在移动应用的开发环境使用React Native，你应该使用 `react-router-native`。

React Router 是建立在[history](https://github.com/ReactTraining/history)之上的。简而言之，一个 `history` 知道如何去监听浏览器地址栏的变化， 并解析这个 URL 转化为 location 对象， 然后 router 使用它匹配到路由，最后正确地渲染对应的组件。

## 静态路由和动态路由的选择

首先我们从历史的发展开始讲起，当时，React Router的作者Michael和Ryan都有着开发[Ember](https://emberjs.com)的经验，自然React Router的第一个版本和Ember的路由机制很相似--都是有一个静态文件去定义路由规则，作为初始化应用的一部分。

使用静态路由的通常都会有这么一段代码在项目定义文件内：
```
const routes = (
  <Router>
    <Route path='/' component={Main}>
      <IndexRoute component={Home} />
      <Route path='playerOne' component={Prompt} />
      <Route path='playerTwo/:playerOne' component={Prompt} />
      <Route path='battle' component={ConfirmBattle} />
    </Route>
  </Router>
)
export default routes
```

这种静态路由的概念应用也很普遍，我们所熟知的Express、Angular、Ember都是使用的这种静态路由。

我们知道，React Router4运用的是动态路由，那么问题来了，**静态路由不好用吗？**

答案是，“不，好用”。但是，大家觉得静态路由不符合React的设计模式。为什么呢？

我们来看一个特殊路由的定义
```
<Route onEnter={checkAuth} path='home' component={Home} />
```
这个路由特殊在他是一个鉴权路由，也就是说，在路由初始化时会判断当前用户有没有权限进入，这是非常普遍的场景。这听起来像是在`Home`组件的`ComponentDidMount`做的事情，的确也是这样。

但是如果加上了嵌套路由，比如我们需要在`/home/a`和`/home/b`两个嵌套路由中增加鉴权，我们需要怎么办？应该是这样：
```
<Route onEnter={checkAuth} path='home' component={Home} />
<Route onEnter={checkAuth} path='home/a' component={HomeA} />
<Route onEnter={checkAuth} path='home/b' component={HomeB} />
```
这样一来，鉴权操作就不能只在`Home`组件的`ComponentDidMount`中执行了。这个问题，React的路由组件化概念是这么实现的：
**Home组件**
```
const Home = () => (
  <div>
    <h2>Home</h2>
    <ul>
      <li>
        <Link to={`/home`}>
          Rendering with React
        </Link>
      </li>
    </ul>
    <Route path={`/home/a`} component={HomeA} />
    <Route path={`/home/b`} component={HomeB} />
  </div>
)
```
这样一来，我们的鉴权操作始终都能在`Home`组件的`ComponentDidMount`中执行,这就是React Router 4和传统静态路由的区别。所有的API都是基于`Component`，我们可以在任何一个React组件中去使用它，而不光光是路由定义文件。

## 基础路由
下面是普遍的路由的例子：
```
<Router>
  <Route exact path="/" component={Home}/>
  <Route path="/category" component={Category}/>
  <Route path="/login" component={Login}/>
  <Route path="/products" component={Products}/>
</Router>
```
上面的例子中，我们使用了`<Router>`组件和一些`<Route>`组件来创建一个基本的路由结构。

## 嵌套路由
创建嵌套路由之前，我们需要更深入的理解`<Route>如何运行`。开始吧。

`<Route>`有三个可以用来定义要渲染内容的props：

- **component**.在上面我们已经看到，当URL匹配时，router会将传递的组件使用`React.createElement`来生成一个React元素。

- **render** 适合行内渲染。路径匹配时，`render`接受一个函数返回一个元素。适用于根据`props`的不同来渲染不同的component。

- **children** `children`和`render`类似，也接受一个函数返回一个React元素，不同的是，不管路径是否匹配，children都会渲染。

#### Path and match

**path**用来标识路由匹配的URL部分。React Router使用了Path-to-RegExp库将路径字符串转为正则表达式，然后和当前路径进行匹配。

当路由路径和当前路径成功匹配，会生成一个对象，我们叫它**match**。match对象有更多关于URL和Path的信息。这些信息可以通过它的属性获取，如下所示：

- `match.url` 返回URL匹配部分的字符串。对于创建嵌套的`<Link>`很有用。
- `match.path` 返回路由路径完整字符串
- `match.isExact` 使用===来准确匹配
- `match.params` 返回一个对象包含Path-to-RegExp包从URL解析的键值对。

#### 什么时候需要Switch组件？
我们先来看一段代码：
```
<Route exact path="/" component={Home}/>
<Route path="/products" component={Products}/>
<Route path="/category" component={Category}/>
<Route path="/:id" render = {()=> (<p> I want this text to show up for all routes other than '/', '/products' and '/category' </p>)}/>
```
如果当前URL为`/products`，那么所有匹配`/products`路径的route都会被渲染。所以，最后一个路由，`path = /:id`的route最终会被render，设计就是如此，但这可能不是我们想要的结果，我们希望只有第一个被匹配到的路径才会被渲染，这就需要`<Switch>`组件了。

其实之前react-router-dom还提供了一个IndexRoute组件，现在已经被废弃了，我们现在使用`Switch`来替代它。

现在我们完全了解了`<Route>`，开始创建一个嵌套路由吧。

#### 嵌套路由的实现

我们给`/category`和`/products`创建了路由。但如果我们想要`/category/shoes`这种形式的URL呢？

**src/App.js**

```
import React, { Component } from 'react';
import { Link, Route, Switch } from 'react-router-dom';
import Category from './Category';

class App extends Component {
  render() {

    return (
      <div>
        <nav className="navbar navbar-light">
          <ul className="nav navbar-nav">
            <li><Link to="/">Homes</Link></li>
            <li><Link to="/category">Category</Link></li>
            <li><Link to="/products">Products</Link></li>
          </ul>
       </nav>

    <Switch>
      <Route exact path="/" component={Home}/>
      <Route path="/category" component={Category}/>
       <Route path="/products" component={Products}/>
    </Switch>

    </div>
    );
  }
}
export default App;
```
不像React Router之前的版本，在版本4中，嵌套的 <Route>最好放在父元素里面。所以，Category组件就是这里的父组件，我们将在父组件中定义 `category/:name` 路由。

**src/Category.jsx**
```
import React from 'react';
import { Link, Route } from 'react-router-dom';

const Category = ({ match }) => {
return( <div> <ul>
    <li><Link to={`${match.url}/shoes`}>Shoes</Link></li>
    <li><Link to={`${match.url}/boots`}>Boots</Link></li>
    <li><Link to={`${match.url}/footwear`}>Footwear</Link></li>

  </ul>
  <Route path={`${match.path}/:name`} render= {({match}) =>( <div> <h3> {match.params.name} </h3></div>)}/>
  </div>)
}
export default Category;
```

首先，我们给嵌套路由定义了一些Link。之前提到过， `match.url`用来构建嵌套链接， `match.path`用来构建嵌套路由。如果你对match有不理解的概念， `console.log(match)`会提供一些有用的信息来帮助你了解它。

## History

通过阅读源码，我们知道了所有的路由操作都是通过操作`history`来进行的，接下来正式介绍下`history`。

> `history`是一个统一了所有DOM和非DOM环境会话记录的Javascript库。history提供了简洁的API,让你可以管理history堆栈，跳转，跳转前确认，以及保持会话之间的状态。

history是react router所必需的两个依赖之一。
每个router组件创建了一个history对象，用来记录当前路径( `history.location` )，上一步路径也存储在堆栈中。当前路径改变时，视图会重新渲染，给你一种跳转的感觉。

history对象有 `history.push()`和 `history.replace()`这些方法来实现。当你点击 `<Link>`组件会触发 `history.push()`，使用 `<Redirect>`则会调用 `history.replace()`。其他方法 - 例如 `history.goBack()`和 `history.goForward()` - 用来根据页面的后退和前进来跳转history堆栈。

那么history内部又是怎么样实现路由监听和响应的呢？

#### History的实现

在了解history实现之前，我们首先知道浏览器给我们提供的原生路由功能，比如[`location`](https://developer.mozilla.org/zh-CN/docs/Web/API/Location)，接下来的很多方法中将出现它们的影子。

`history`提供了三种路由实现的方式。
1. `browserHistory`，支持HTML5 history API的路由实现（[兼容性](https://caniuse.com/#feat=history)）
2. `hashHistory`，向下兼容传统浏览器的hash路由实现
3. `createMemoryHistory`，React实现的定制化路由，不依赖任何环境，可用于非DOM环境，比如`React Native` 或 `SSR`

#### browserHistory
通过`history.createBrowserHistory`创建，下面是主要封装的方法和原生方法的映射。

原生方法 ---> 封装方法
1. `pushState` ---> `push`
2. `popState`  ---> `goBack`
3. `replaceState` ---> `replace`

```
window.history.pushState(state, title, url) 
// state：需要保存的数据，这个数据在触发popstate事件时，可以在event.state里获取
// title：标题，基本没用，一般传 null
// url：设定新的历史记录的 url。新的 url 与当前 url 的 origin 必须是一樣的，否则会抛出错误。url可以是绝对路径，也可以是相对路径。
//如 当前url是 https://www.baidu.com/a/,执行history.pushState(null, null, './qq/')，则变成 https://www.baidu.com/a/qq/，
//执行history.pushState(null, null, '/qq/')，则变成 https://www.baidu.com/qq/

window.history.replaceState(state, title, url)
// 与 pushState 基本相同，但她是修改当前历史记录，而 pushState 是创建新的历史记录

window.history.back() // 后退
window.history.forward() // 前进
window.history.go(1) // 前进一步，-2为后退两步，window.history.lengthk可以查看当前历史堆栈中页面的数量
```
> 重要!! `history` 模式改变 url 的方式会导致浏览器向服务器发送请求，这不是我们想看到的，我们需要在服务器端做处理：如果匹配不到任何静态资源，则应该始终返回同一个 html 页面。

#### hashHistory
通过`history.createHashHistory`创建

这里的hash就是指url尾巴后的#号以及后面的字符。也称作锚点，本身是用来做页面定位的，她可以使对应id的元素显示在可视区域内。

由于hash值变化不会导致浏览器向服务器发出请求，而且hash改变会触发`hashChange`事件，浏览器的前进后退也能对其进行控制，所以在html5的`history`出现前，基本都是使用hash来实现前端路由的，这也是hash模式能兼容老版浏览器的原因。

#### 监听路由变化

`history`提供了对外提供了封装的 **listen** 方法监听路由的变化。这里采用了观察者模式(publish-subscription)，每一个注册在`history.listen(callback)`中的`callback `都是一个监听者，它接受[location](https://reacttraining.com/react-router/core/api/location)和`action`两个参数。当路由发生变化时，`notifyManager`回调触发所有的监听者，从而实现事件的同步。listen返回了解绑listen的方法。实现上是很值得参考的。
```
// 绑定监听事件
unlisten = history.listen((location,action) => {
   console.log(location)  // 返回路由的信息，如pathname，hash等信息
   console.log(action)  // 返回POP,PUSH,REPLACE中
});
// 解绑监听事件
unlisten();
```

我们可以翻看一下`createBrowserHistory.js`，可以看到`history`内部是怎么实现监听浏览器路由变化的。

**[window.popstate](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event)**

> The popstate event of the `Window` interface is fired when the active history entry changes。

源码中显示，通过`window.addEventListener('popstate',cb)`注册的回调，需要注意的是，调用`history.pushState`或`history.replaceState`时不会触发`popstate`事件，只有在做出浏览器动作时，才会触发该事件，如用户点击浏览器的回退按钮（或者在JS中调用history.back()).

路由变化的也是同理。
**[window.hashchange](https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event)**

> The `hashchange` event is fired when the fragment identifier of the URL has changed

`window.addEventListener('hashchange',cb)`注册的回调，需要注意的是浏览器原生只会监听#符号后的变化，而我们在react或者类react项目中使用的`#/`，都是框架层做的封装。

## 在服务端（SSR）的应用

`react-router`在SSR端提供了`StaticRouter`的封装路由入口，由于不支持`location`，服务端的功能受到了不少限制，只提供了push、replace方法，`StaticRouter`意思就是说，是静态的路由，它实际上并没有改变当前的路由，而只是记录了路由的改变。

## 总结：

现在我们捋一下`react-router`(下称RR)和`history`(下称H)的流程：

RR初始化Router时，调用H的`listen`方法，开始监听路由变化，回调为CB。
更改浏览器URL(或者hash) --> 回调CB,开始调用注册在`transitionManager`上的listeners --> RR中的location变化 --> 利用`path-to-regexp`匹配到`Component`或者`children node` --> `React.render(node)` --> 完成页面渲染。

如你在本文中所看到的，React Router是一个帮助React构建更完美，更声明式的路由库。不像React Router之前的版本，在V4中，一切都是组件。而且，新的设计模式也更完美的使用React的构建方式来实践。比如我们提到的ReactContext.Provider，ReactContext.Consumer。

这次教程，我们由浅入深，学到了：
- 路由的基础功能，设计思路和React Router的基础组件
- History的实践和原理
- 浏览器原生支持的监听路由事件的方法

## React Router的未来

Ryan Florence 在[The Future of React Router](https://reacttraining.com/blog/reach-react-router-future/)中阐述了V4版本的react router设计中的更新，也提出了更严格的要求。

- 更小的bundle大小，大约约束在3kb左右
- 分离核心功能，不依赖任何其他库
- 组织更好的嵌套路由的配置(和V3版本和@react/router的更像)
- 相对性的Links,Navigation
- 使用`useLocation`来更好的管理`location`和`navigate`
- Route 参数验证场景

## Tips

#### 我经常在链接上看到`?_k=ckuvup`这样的标识符有用吗？
有用！当一个`history`通过应用程序的push或replace跳转时，它可以在新的location中存储"location state",而不显示在URL中，这就像是在POST一个表单数据。

在DOM API中，这些hash history通过`window.location.hash = newHash `，很简单的利用hash来实现跳转。但是这种行为是不具备回溯性的，我们想要全部的history都能够使用location state,这就要求我们为每一个location创建一个唯一的key，并把它们的状态存储在 **session storage** 中。当访客点击"前进"和"后退"时，我们就可以使用key:value的机制找到这个location 去恢复它的state。


#### 参考资料
[React-router wiki](http://react-guide.github.io/react-router-cn/docs/guides/basics/Histories.html)

[React History](https://github.com/ReactTraining/history)

[React Router v4 完全指北](https://cloud.tencent.com/developer/article/1407614)

[前端路由的两种模式](https://www.cnblogs.com/JRliu/p/9025290.html)




