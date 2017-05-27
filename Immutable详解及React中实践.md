## Immutable 详解及React中实践

> Shared mutable state is the root of all evil(共享的可变状态是万恶之源)
> --Pete Hunt

有人说 Immutable 可以给 React 应用带来数十倍的提升，也有人说 Immutable 的引入是近期 JavaScript 中伟大的发明，因为同期 React 太火，它的光芒被掩盖了。这些至少说明 Immutable 是很有价值的，下面我们来一探究竟。

JavaScript 中的对象一般是可变的（Mutable），因为使用了引用赋值，新的对象简单的引用了原始对象，改变新的对象将影响到原始对象。如 
`` foo ={a:1};bar =foo;bar.a=2`` ，你会发现此时``foo.a``也被改成了``2``。虽然这样做可以节约内存，但当应用复杂后，这就造成了非常大的隐患，Mutable带来的优点变得得不偿失。为了解决这个问题，一般的做法是使用shallowCopy(浅拷贝)或deepCopy(深拷贝)来避免被修改，但这样做造成了CPU和内存的浪费。

Immutable 可以很好地解决这些问题。

#### 什么是Immutable Data

Immutable Data 就是一旦创建，就不能再被更改的数据。对 Immutable 对象的任何修改或添加删除操作都会返回一个新的 Immutable 对象。
Immutable 实现的原理是**Persistent Data Structure**(持久化数据结构)。也就是使用旧数据创建新数据时，要保证旧数据同时可用且不变。同时为了避免deepCopy把所有节点都复制一遍带来的性能损耗，Immutable使用了Structural Sharing(结构共享)，即如果对象树中一个节点发生变化，只修改这个节点和受他影响的父节点，其他节点则进行共享，请参考下面动画：

![](http://img.alicdn.com/tps/i2/TB1zzi_KXXXXXctXFXXbrb8OVXX-613-575.gif)

目前流行的Immutable库有两个：

**Immutable.js**

Facebook 工程师 Lee Byron 花费 3 年时间打造，与 React 同期出现，但没有被默认放到 React 工具集里（React 提供了简化的 Helper）。它内部实现了一套完整的 Persistent Data Structure，还有很多易用的数据类型。像**Collection**、 **List** 、**Map**、**Set **、**Record ** 、**Seq**。有非常全面的**map**、**filter**、**groupBy**、**reduce find**函数式操作方法。同时ApI也尽量与Object或Array类似。

其中有3种最重要的数据结构说明以下：

+ Map:键值对集合，对于Object，ES6也有专门的Map对象

+ List:有序可重复的列表，对应于Array

+ Set:无序且不可重复的列表

**seamless-immutable**

与Immutable.js学院派的风格不同，seamless-immutable并没有实现完整的Persistent Data Structure，而是使用Object.defineProperty(因此兼容性只能在IE9及以上使用)扩展了JavaScript的Array和Object对象来实现，只支持Array和Object两种数据类型，API基于Array和Object。代码库非常小，压缩下载只有2k。而Immutable.js压缩后下载有16k。

下面上代码来感受一下两者的不同：

```js
// 原来的写法
let foo ={a: {b : 1}};
let bar =foo;
bar.a.b=2;
console.log(foo.a.b);      // 打印 2
console.log(foo === bar);  // 打印 true

// 使用immutable.js后
import Immutable from 'immutable';
foo =Immutable.fromJS({a: {b: 1}});
bar =foo.setIn(['a','b'],2);  // 使用setIn赋值
console.log(foo.getIn(['a','b']));  // 使用getIn取值，打印 1
console.log(foo === bar ); // 打印false

// 使用 seamless-immutable.js后
import SImmutable from 'seamless-immutable';
foo =SImmutable({a:{b:1}});
bar =foo.merge({a:{b:2}});   // 使用merge赋值
console.log(foo.a.b);  // 像原生Object一样取值，打印 1
console.log(foo === bar);  //打印false
	
```

#### Immutable 优点

1 **Immutable 降低了Mutable带来的复杂度**

可变数据(Mutable Data)耦合了Time 和 Value 的概念，造成了数据很难被回溯。

比如下面一段代码：

```js
function touchAndLog(touchFn){
	let data ={
		key : 'value'
	};
	touchFn(data);
	console.log(data.key)  
}
```

在不查看touchFn代码的情况下，因为不确定它对``data``做了什么，你不可能知道会打印什么（这当然是废话）。但如果``data``是Immutable的，你可以很肯定的知道打印的是``value``;

2 **节省内存**

Immutable.js 使用了Structure Sharing 会尽量复用内存，甚至以前使用的对象也可以再次被复用。没有被引用的对象会被垃圾回收。

```js
import { Map} from 'immutable'
let a = Map({
	select: 'users',
	filter: Map({ name: 'Cam'})
});
let b =a.set('select','people');
	
a === b; // false
	
a.get('filter') === b.get('filter');  // true
```

上面a 和 b 共享了没有变化的`` filter `` 节点。

3 **Undo/Redo ,Copy/Paste,甚至时间旅行这些功能做起来小菜一碟**

因为每次数据都是不一样的，只要把这些数据放到一个数组里存储起来，想回退到哪里就拿出对应数据即可，很容易开发出撤销重做这种功能。

后面我会提供Flux做Undo的示例。

4 **并发安全**

传统的并发非常难做，因为要处理各种数据不一致的问题，因此【聪明人】发明了各种锁来解决。但使用了Immutable之后，数据天生是不可变的，并发锁就不需要了。

然而现在并没什么卵用，因为JavaScript还是单线程运行的，但未来可能会加入，提前解决未来的问题不也挺好的吗。

5 **拥抱函数式编程**

Immutable 本身就是函数式编程中的概念，纯函数式编程比面向对象更适用于前端开发。因为只要输入一致，输出必然一致，这样开发的组件更易于调试和组装。

像 ClojureScript，Elm 等函数式编程语言中的数据类型天生都是 Immutable 的，这也是为什么 ClojureScript 基于 React 的框架 --- Om 性能比 React 还要好的原因。

#### Immutable 缺点

1 需要学习新的API

No Comments

2 增加了资源文件大小

No Comments

3 容易与原生对象混淆

这点是我们使用 Immutable.js 过程中遇到最大的问题。写代码要做思维上的转变

虽然 Immutable.js 尽量尝试把 API 设计的原生对象类似，有的时候还是很难区别到底是 Immutable 对象还是原生对象，容易混淆操作。

Immutable 中的 Map 和 List 虽对应原生 Object 和 Array，但操作非常不同，比如你要用```map.get('key')```而不是```map.key```,```array.get(0)```而不是```array[0]```。另外Immutable每次修改都会返回新对象，也很容易忘记赋值。

当使用外部库时，一般需要使用原生对象，也很容易忘记转换。

下面给出一些办法来避免类似问题发生：

1 使用Flow 或TypeScript这类有静态类型检查的工具。
2 约定变量命名规则：如所有Immutable类型都以$$开头。
3 使用immutable.fromJS而不是Immutable.map 或Immtable.List来创建对象，这样可以避免Immutable和原生对象间的混用。

#### 更多认识

``` Immutable.js ```

两个immutable 对象可以使用 ```===``` 来比较，因为```===```是直接比较内存地址，性能最好。虽然两个对象的值是一样的，也会返回```false```:

```
let map1 =Immtable.map({a:1, b: 1,c: 1});
let map2 =Immtable.map({a:1, b: 1,c: 1});
	
map1 === map2;   // false
```

为了直接比较对象的值，immutable.js提供了is来做值比较，结果如下：

```js
Immtable.is(map1,map2);  //true
```

```Immtable.js```比较的是两个对象的``hashCode`` 或 ``valueOf``(对于Javascript对象)。由于immutable内部使用了Trie数据结构来存储，只要两个对象的``hashCode``相等，值就是一样的。这样的算法避免了深度遍历比较，性能非常好。

后面会使用```Immutable.js```来减少React重复渲染，提高性能。

#### 与Object.freeze、const 的区别

ES6中新加入的```Object.freeze``` 和 ``const`` 都可以达到防止对象被篡改的功能，但它们是shallowCopy的。对象层级一深就要特殊处理了。

#### Cursor 的概念

这个Cursor 和数据库中的游标是完全不同的概念。

由于Immutable数据一般嵌套比较深，为了便于访问深层数据，Cursor提供了可以直接访问这个深层数据的引用。

```
import Immtable from 'immutable';
import Cursor from 'immutable/contrib/cursor';

let data =Immtable.fromJs({a: {b: {c:1}}});
// 让cursor指向 { c:1};
let cursor =Cursor.from(data,['a','b'],newData => {
	// 让cursor 或其子cursor执行update时调用
	console.log(newData);
});
cursor.get('c'); // 1
cursor =cursor.update('c', x =>x +1);
cursor.get('c'); // 2
```

#### 实践

**与React搭配使用，Pure Render**

熟悉React的都知道，React做性能优化时有一个避免重复渲染的大招，就是使用```shouldComponentUpdate()```，但它默认返回```true```，即始终会执行```render()```方法，然后做Virtual DOM比较，并得出是否需要做真实DOM更新，这里往往会带来很多不必要的渲染并成为性能瓶颈。

当然我们也可以在```shouldComponentUpdate()```中使用deepCopy和deepCompare来避免不必要的``render``，但**deepCopy**和**deepCompare**一般都是**非常耗性能的**。

**Immutable 则提供了简洁高效的判断数据是否变化的方法**，只需``===``和``is``比较就能知道是否需要执行``render()``，而这几个操作几乎零成本，所以可以极大提高性能。修改后的```shouldComponentUpdate```是这样的：

```js
	import {is} from 'immutable';
	shouldComponentUpdate :(nextProps,nextState)=>{
		return !(this.props === nextProps || is(this.props, nextProps)) ||
			   !(this.state === nextState || is(this.state, nextState));
	}
```

使用Immutable后，如下图，当红色节点的state变化后，不会再渲染树中所有节点，而是只渲染图中绿色的部分：

![](http://img.alicdn.com/tps/i3/TB1VinpKXXXXXXAXpXXZ_OdNFXX-715-324.png)

你也可以借助 ```React.addons.PureRenderMixin``` 或支持class语法的[pure-render-decorator](https://yq.aliyun.com/articles/16969)来实现。

**setState的一个技巧**

React建议把```this.state```当作Immutable的，因此修改前需要做一个deepCopy，显得麻烦：

```js
import _ from 'lodash';

const Component =React.createClass({
	getInitialState(){
		return {
			data: {times: 0}
		}
	},
	handleAdd(){
		let data =_.cloneDeep(this.state.data);
		data.times =data.times +1;
		this.setState({data: data});
		// 如果上面不做cloneDeep，下面打印的结果是已经加1后的值。
		console.log(this.state.data.times);
	}
})
```

使用Immutable后：

```js
getInitialState(){
	return {
		data: Map({times: 0})
	}
},
handleAdd(){
	this.setState({data: this.state.data.update('times', v => v + 1)});
	// 这时的times并不会改变
	console.log(this.state.data.get('times'));
}
```

上面的```handleAdd```可以简写成：

```
handleAdd(){
	this.setState(({data})=> ({
		data: data.update('times', v => v + 1)}
	));
}
```

**与Flux搭配使用**

由于 Flux 并没有限定 Store 中数据的类型，使用 Immutable 非常简单。

现在是实现一个类似带有添加和撤销功能的 Store：

```js
import {Map , OrderedMap } from 'immutable';
let todos =OrderedMap();
let history =[];  // 普通数组，存放每次操作后产生的数据

let TodoStore =createStore({
	getAll(){ return todos; }
});
Dispatcher.register(action => {
	if(action.actionType === 'create'){
		let id =createGUID();
		history.push(todos);  //记录当前操作前的数据，便于撤销
		todos = todos.set(id, Map({
			id: id,
			complete: false,
			text: action.text.trim()
		}));
		TodoStore.emitChange();
	}else if(action.actionType === 'undo'){
		// 这里是撤销功能实现，
		只需从history数组中取前一次todos即可
		if(history.length >0){
			todos =history.pop();
		}
		TodoStore.emitChange();
	}
})
```

**与Redux搭配使用**

Redux 是目前流行的 Flux 衍生库。它简化了 Flux 中多个 Store 的概念，只有一个 Store，数据操作通过 Reducer 中实现；同时它提供更简洁和清晰的单向数据流（View -> Action -> Middleware -> Reducer），也更易于开发同构应用。目前已经在我们项目中大规模使用。

由于 Redux 中内置的 ```combineReducers``` 和 reducer 中的 ```initialState``` 都为原生的 Object 对象，所以不能和 Immutable 原生搭配使用。

幸运的是，Redux 并不排斥使用 Immutable，可以自己重写 ```combineReducers``` 或使用 [redux-immutablejs](https://github.com/indexiatech/redux-immutablejs) 来提供支持。

上面我们提到 Cursor 可以方便检索和 update 层级比较深的数据，但因为 Redux 中已经有了 select 来做检索，Action 来更新数据，因此 Cursor 在这里就没有用武之地了。

#### 总结

Immutable 可以给应用带来极大的性能提升，但是否使用还要看项目情况。由于侵入性较强，新项目引入比较容易，老项目迁移需要评估迁移。对于一些提供给外部使用的公共组件，最好不要把 Immutable 对象直接暴露在对外接口中。

#### 参考

- [Lee Byron - Immutable Data and React]
- [Immutable Data Structures and JavaScript](http://jlongster.com/Using-Immutable-Data-Structures-in-JavaScript)






