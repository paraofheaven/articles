## React文档中没提到的事

**前言**

对很多 react 新手来说，网上能找到的资源大都是些简单的 tutorial ，React文档也是非常值得一读的，英文版的有着非常详细的介绍。它们能教会你如何使用 react ，但并不会告诉你怎么在实际项目中优雅的组织和编写 react 代码。用谷歌搜中文“ React 最佳实践”发现前两页几乎全都是同一篇国外文章的译文...所以我总结了下自己过去那个项目使用 React 踩过的一些坑，也整理了一些别人的观点，希望对部分 react 使用者有帮助。

#### React与AJAX

React只负责处理View这一层，它本身不涉及网络请求/AJAX，所以这里我们需求考虑两个问题：

> - 第一，用什么技术从服务端获取数据；
> - 第二，获取到的数据应该放在react组件的什么位置。
 
React官方提供了一种解决方案：[Load Initial Data via AJAX](https://facebook.github.io/react/tips/initial-ajax.html)

使用jQuery的Ajax方法，在一个组件的``componentDidMount()``中发ajax请求，拿到的数据存在组件自己的state中，并调用setState方法去更新UI。如果是异步获取数据，则在``componentWillUnmount``中取消发送请求。

如果只是为了使用jQuery的Ajax方法就引入整个jQuery库，既是一种浪费又加大了整个应用的体积。那我们还有什么其他的选择吗？事实上是有很多的：[fetch()](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch)、[fetch polyfill](https://github.com/github/fetch) 、[axios](https://github.com/mzabriskie/axios)...其中最需要我们关注的是``window.fetch()``,它是一个简洁、标准化的javascript的Ajax API。在Chrome和Firefox中已经可以使用，如果需要兼容其他浏览器，可以使用fetch polyfill。

React官方文档只告诉了我们在一个单一组件中如何通过ajax从服务器端获取数据，但并没有告诉我们在一个完整的实际项目中到底应该把数据存在哪些组件中，这部分如果缺乏规范的话，会导致整个项目变得混乱、难以维护。下面给出三种比较好的实践：

1. 所有的数据请求和管理都存放在唯一的一个根组件
 
让父组件/根组件集中发送所有的ajax请求，把从服务端获取的数据统一存放在这个组件的state中，再通过props把数据传给子组件。这种方法主要是针对组件树不是很复杂的小型应用。缺点就是当组件树的层级变多了以后，需要把数据一层一层地传给子组件，写起来麻烦，性能也不好。

2. 设置多个容器组件专门处理数据请求和管理

其实跟第一种方法类似，只不过设置多个容器组件来负责数据请求和状态管理。这里我们需要区分两种不同类型的组件，一种是展示性组件（presentational component），另一种是容器性组件（container component）。展示性组件本身不拥有任何状态，所有的数据都从容器组件中获得，在容器组件中发送ajax请求。两者更详细的描述，可以阅读下这篇文章：[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.ch9xqg6s4)

一个具体的例子：

假设我们需要展示用户的姓名和头像，首先创建一个展示性组件``<UserProfile />``,它接受两个Props：``name``和``profileImage``。这个组件内部没有任何关于Ajax的代码。

然后创建一个容器组件``<UserProfileContainer />``，它接受一个userId的参数，发送Ajax请求从服务器获取数据存在``state``中，再通过``props``传给``<UserProfile />``组件。

3. 使用Redux或Flux的情况

Redux管理状态和数据，Ajax从服务器端获取数据，所以很显然当我们使用了Redux时，应该把所有的网络请求都交给redux来解决。具体来说，应该是放在Async Actions。如果用其他类Flux库的话，解决方式都差不多，都是在actions中发送网络请求，使用promise fetch或者jquery的ajax就看项目的需要和自己的喜好了。

#### 使用 PropTypes 和 getDefaultProps()

> 1. 一定要写PropTypes，切莫为了省事而不写
> 2. 如果一个Props不是requied，一定在getDefaultProps中设置它
``React.PropTypes``主要用来验证组件接收到的props是否为正确的数据类型，如果不正确，console中就会出现对应的warning。出于性能方面的考虑，这个API只在开发环境下使用。

基本使用方法：

```js
propTypes: {
    myArray: React.PropTypes.array,
    myBool: React.PropTypes.bool,
    myFunc: React.PropTypes.func,
    myNumber: React.PropTypes.number,
    myString: React.PropTypes.string，
     
     // You can chain any of the above with `isRequired` to make sure a warning
    // is shown if the prop isn't provided.
    requiredFunc: React.PropTypes.func.isRequired
} 
```

假如我们props不是以上类型，而是拥有复杂结构的对象怎么办？比如下面这个：

```js
{
  text: 'hello world',
  numbers: [5, 2, 7, 9],
}
```

当然，我们可以直接用``React.PropTypes.object``,但是对象内部的数据我们却无法验证。

```js
propTypes: {
  myObject: React.PropTypes.object,
}
```

进阶使用方法：``shape()`` 和 ``arrayOf()``

```js
propTypes: {
  myObject: React.PropTypes.shape({
    text: React.PropTypes.string,
    numbers: React.PropTypes.arrayOf(React.PropTypes.number),
  })
}
```

下面是一个更复杂的Props：

```js
[
  {
    name: 'Zachary He',
    age: 13,
    married: true,
  },
  {
    name: 'Alice Yo',
    name: 17,
  },
  {
    name: 'Jonyu Me',
    age: 20,
    married: false,
  }
]
```

综合上面，写起来应该就不难了：

```js
propTypes: {
    myArray: React.PropTypes.arrayOf(
        React.propTypes.shape({
            name: React.propTypes.string.isRequired,
            age: React.propTypes.number.isRequired,
            married: React.propTypes.bool
        })
    )
}
```

#### 把计算和条件判断都交给 render() 方法吧

1 组件的state中不能出现props

```js
 // BAD:
  constructor (props) {
    this.state = {
      fullName: `${props.firstName} ${props.lastName}`
    };
  }

  render () {
    var fullName = this.state.fullName;
    return (
      <div>
        <h2>{fullName}</h2>
      </div>
    );
  }
```

```js
// GOOD: 
render () {
  var fullName = `${this.props.firstName} ${this.props.lastName}`;
}
```

当然，复杂的display logic也应该避免全堆放在render()中，因为那样可能导致整个render()方法变得臃肿，不优雅。我们可以把一些复杂的逻辑通过helper function移出去。

```js
// GOOD: helper function
renderFullName () {
  return `${this.props.firstName} ${this.props.lastName}`;
}

render () {
  var fullName = this.renderFullName();
}
```

2 保持state的简洁，不要出现计算得来的state

```js
// WRONG:
  constructor (props) {
    this.state = {
      listItems: [1, 2, 3, 4, 5, 6],
      itemsNum: this.state.listItems.length
    };
  }
  render() {
      return (
          <div>
              <span>{this.state.itemsNum}</span>
          </div>
      )
  }
```

```
// Right:
render () {
  var itemsNum = this.state.listItems.length;
}
```

3 能用三元判断符，就不用If，直接放在render()里

```
// BAD: 
renderSmilingStatement () {
    if (this.state.isSmiling) {
        return <span>is smiling</span>;
    }else {
        return '';
    }
},

render () {
  return <div>{this.props.name}{this.renderSmilingStatement()}</div>;
}
```

```
// GOOD: 
render () {
  return (
    <div>
      {this.props.name}
      {(this.state.smiling)
        ? <span>is smiling</span>
        : null
      }
    </div>
  );
}
```

4 布尔值都不能搞定的，交给IIFE吧

[Immediately-invoked function expression](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)

```
return (
  <section>
    <h1>Color</h1>
    <h3>Name</h3>
    <p>{this.state.color || "white"}</p>
    <h3>Hex</h3>
    <p>
      {(() => {
        switch (this.state.color) {
          case "red":   return "#FF0000";
          case "green": return "#00FF00";
          case "blue":  return "#0000FF";
          default:      return "#FFFFFF";
        }
      })()}
    </p>
  </section>
);
```

5 不要把display logic写在``componentWillReceiveProps``或``componentWillMount``中，把它们都移到render()中去。

#### 如何动态处理 classNames

1 使用布尔值

```
// BAD:
constructor () {
    this.state = {
      classes: []
    };
  }

  handleClick () {
    var classes = this.state.classes;
    var index = classes.indexOf('active');

    if (index != -1) {
      classes.splice(index, 1);
    } else {
      classes.push('active');
    }

    this.setState({ classes: classes });
  }
  
  
  
  // GOOD:
  constructor () {
    this.state = {
      isActive: false
    };
  }

  handleClick () {
    this.setState({ isActive: !this.state.isActive });
  }
```

2 使用[classnames](https://github.com/JedWatson/classnames)这个小工具来拼接classNames：

```
// BEFORE:
var Button = React.createClass({
  render () {
    var btnClass = 'btn';
    if (this.state.isPressed) btnClass += ' btn-pressed';
    else if (this.state.isHovered) btnClass += ' btn-over';
    return <button className={btnClass}>{this.props.label}</button>;
  }
});
```

```
// AFTER：
var classNames = require('classnames');

var Button = React.createClass({
  render () {
    var btnClass = classNames({
      'btn': true,
      'btn-pressed': this.state.isPressed,
      'btn-over': !this.state.isPressed && this.state.isHovered
    });
    return <button className={btnClass}>{this.props.label}</button>;
  }
});
```

























