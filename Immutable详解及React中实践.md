## Immutable ��⼰React��ʵ��

> Shared mutable state is the root of all evil(����Ŀɱ�״̬�����֮Դ)
> --Pete Hunt

����˵ Immutable ���Ը� React Ӧ�ô�����ʮ����������Ҳ����˵ Immutable �������ǽ��� JavaScript ��ΰ��ķ�������Ϊͬ�� React ̫�����Ĺ�â���ڸ��ˡ���Щ����˵�� Immutable �Ǻ��м�ֵ�ģ�����������һ̽������

JavaScript �еĶ���һ���ǿɱ�ģ�Mutable������Ϊʹ�������ø�ֵ���µĶ���򵥵�������ԭʼ���󣬸ı��µĶ���Ӱ�쵽ԭʼ������ 
`` foo ={a:1};bar =foo;bar.a=2`` ����ᷢ�ִ�ʱ``foo.a``Ҳ���ĳ���``2``����Ȼ���������Խ�Լ�ڴ棬����Ӧ�ø��Ӻ��������˷ǳ����������Mutable�������ŵ��õò���ʧ��Ϊ�˽��������⣬һ���������ʹ��shallowCopy(ǳ����)��deepCopy(���)�����ⱻ�޸ģ��������������CPU���ڴ���˷ѡ�

Immutable ���Ժܺõؽ����Щ���⡣

#### ʲô��Immutable Data

Immutable Data ����һ���������Ͳ����ٱ����ĵ����ݡ��� Immutable ������κ��޸Ļ����ɾ���������᷵��һ���µ� Immutable ����
Immutable ʵ�ֵ�ԭ����**Persistent Data Structure**(�־û����ݽṹ)��Ҳ����ʹ�þ����ݴ���������ʱ��Ҫ��֤������ͬʱ�����Ҳ��䡣ͬʱΪ�˱���deepCopy�����нڵ㶼����һ�������������ģ�Immutableʹ����Structural Sharing(�ṹ����)���������������һ���ڵ㷢���仯��ֻ�޸�����ڵ������Ӱ��ĸ��ڵ㣬�����ڵ�����й�����ο����涯����

![](http://img.alicdn.com/tps/i2/TB1zzi_KXXXXXctXFXXbrb8OVXX-613-575.gif)

Ŀǰ���е�Immutable����������

**Immutable.js**

Facebook ����ʦ Lee Byron ���� 3 ��ʱ����죬�� React ͬ�ڳ��֣���û�б�Ĭ�Ϸŵ� React ���߼��React �ṩ�˼򻯵� Helper�������ڲ�ʵ����һ�������� Persistent Data Structure�����кܶ����õ��������͡���**Collection**�� **List** ��**Map**��**Set **��**Record ** ��**Seq**���зǳ�ȫ���**map**��**filter**��**groupBy**��**reduce find**����ʽ����������ͬʱApIҲ������Object��Array���ơ�

������3������Ҫ�����ݽṹ˵�����£�

+ Map:��ֵ�Լ��ϣ�����Object��ES6Ҳ��ר�ŵ�Map����

+ List:������ظ����б���Ӧ��Array

+ Set:�����Ҳ����ظ����б�

**seamless-immutable**

��Immutable.jsѧԺ�ɵķ��ͬ��seamless-immutable��û��ʵ��������Persistent Data Structure������ʹ��Object.defineProperty(��˼�����ֻ����IE9������ʹ��)��չ��JavaScript��Array��Object������ʵ�֣�ֻ֧��Array��Object�����������ͣ�API����Array��Object�������ǳ�С��ѹ������ֻ��2k����Immutable.jsѹ����������16k��

�����ϴ���������һ�����ߵĲ�ͬ��

```js
// ԭ����д��
let foo ={a: {b : 1}};
let bar =foo;
bar.a.b=2;
console.log(foo.a.b);      // ��ӡ 2
console.log(foo === bar);  // ��ӡ true

// ʹ��immutable.js��
import Immutable from 'immutable';
foo =Immutable.fromJS({a: {b: 1}});
bar =foo.setIn(['a','b'],2);  // ʹ��setIn��ֵ
console.log(foo.getIn(['a','b']));  // ʹ��getInȡֵ����ӡ 1
console.log(foo === bar ); // ��ӡfalse

// ʹ�� seamless-immutable.js��
import SImmutable from 'seamless-immutable';
foo =SImmutable({a:{b:1}});
bar =foo.merge({a:{b:2}});   // ʹ��merge��ֵ
console.log(foo.a.b);  // ��ԭ��Objectһ��ȡֵ����ӡ 1
console.log(foo === bar);  //��ӡfalse
	
```

#### Immutable �ŵ�

1 **Immutable ������Mutable�����ĸ��Ӷ�**

�ɱ�����(Mutable Data)�����Time �� Value �ĸ����������ݺ��ѱ����ݡ�

��������һ�δ��룺

```js
function touchAndLog(touchFn){
	let data ={
		key : 'value'
	};
	touchFn(data);
	console.log(data.key)  
}
```

�ڲ��鿴touchFn���������£���Ϊ��ȷ������``data``����ʲô���㲻����֪�����ӡʲô���⵱Ȼ�Ƿϻ����������``data``��Immutable�ģ�����Ժܿ϶���֪����ӡ����``value``;

2 **��ʡ�ڴ�**

Immutable.js ʹ����Structure Sharing �ᾡ�������ڴ棬������ǰʹ�õĶ���Ҳ�����ٴα����á�û�б����õĶ���ᱻ�������ա�

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

����a �� b ������û�б仯��`` filter `` �ڵ㡣

3 **Undo/Redo ,Copy/Paste,����ʱ��������Щ����������С��һ��**

��Ϊÿ�����ݶ��ǲ�һ���ģ�ֻҪ����Щ���ݷŵ�һ��������洢����������˵�������ó���Ӧ���ݼ��ɣ������׿����������������ֹ��ܡ�

�����һ��ṩFlux��Undo��ʾ����

4 **������ȫ**

��ͳ�Ĳ����ǳ���������ΪҪ����������ݲ�һ�µ����⣬��ˡ������ˡ������˸��������������ʹ����Immutable֮�����������ǲ��ɱ�ģ��������Ͳ���Ҫ�ˡ�

Ȼ�����ڲ�ûʲô���ã���ΪJavaScript���ǵ��߳����еģ���δ�����ܻ���룬��ǰ���δ�������ⲻҲͦ�õ���

5 **ӵ������ʽ���**

Immutable ������Ǻ���ʽ����еĸ��������ʽ��̱���������������ǰ�˿�������ΪֻҪ����һ�£������Ȼһ�£�������������������ڵ��Ժ���װ��

�� ClojureScript��Elm �Ⱥ���ʽ��������е����������������� Immutable �ģ���Ҳ��Ϊʲô ClojureScript ���� React �Ŀ�� --- Om ���ܱ� React ��Ҫ�õ�ԭ��

#### Immutable ȱ��

1 ��Ҫѧϰ�µ�API

No Comments

2 ��������Դ�ļ���С

No Comments

3 ������ԭ���������

���������ʹ�� Immutable.js �����������������⡣д����Ҫ��˼ά�ϵ�ת��

��Ȼ Immutable.js �������԰� API ��Ƶ�ԭ���������ƣ��е�ʱ���Ǻ������𵽵��� Immutable ������ԭ���������׻���������

Immutable �е� Map �� List ���Ӧԭ�� Object �� Array���������ǳ���ͬ��������Ҫ��```map.get('key')```������```map.key```,```array.get(0)```������```array[0]```������Immutableÿ���޸Ķ��᷵���¶���Ҳ���������Ǹ�ֵ��

��ʹ���ⲿ��ʱ��һ����Ҫʹ��ԭ������Ҳ����������ת����

�������һЩ�취�������������ⷢ����

1 ʹ��Flow ��TypeScript�����о�̬���ͼ��Ĺ��ߡ�
2 Լ��������������������Immutable���Ͷ���$$��ͷ��
3 ʹ��immutable.fromJS������Immutable.map ��Immtable.List�����������������Ա���Immutable��ԭ�������Ļ��á�

#### ������ʶ

``` Immutable.js ```

����immutable �������ʹ�� ```===``` ���Ƚϣ���Ϊ```===```��ֱ�ӱȽ��ڴ��ַ��������á���Ȼ���������ֵ��һ���ģ�Ҳ�᷵��```false```:

```
let map1 =Immtable.map({a:1, b: 1,c: 1});
let map2 =Immtable.map({a:1, b: 1,c: 1});
	
map1 === map2;   // false
```

Ϊ��ֱ�ӱȽ϶����ֵ��immutable.js�ṩ��is����ֵ�Ƚϣ�������£�

```js
Immtable.is(map1,map2);  //true
```

```Immtable.js```�Ƚϵ������������``hashCode`` �� ``valueOf``(����Javascript����)������immutable�ڲ�ʹ����Trie���ݽṹ���洢��ֻҪ���������``hashCode``��ȣ�ֵ����һ���ġ��������㷨��������ȱ����Ƚϣ����ܷǳ��á�

�����ʹ��```Immutable.js```������React�ظ���Ⱦ��������ܡ�

#### ��Object.freeze��const ������

ES6���¼����```Object.freeze``` �� ``const`` �����Դﵽ��ֹ���󱻴۸ĵĹ��ܣ���������shallowCopy�ġ�����㼶һ���Ҫ���⴦���ˡ�

#### Cursor �ĸ���

���Cursor �����ݿ��е��α�����ȫ��ͬ�ĸ��

����Immutable����һ��Ƕ�ױȽ��Ϊ�˱��ڷ���������ݣ�Cursor�ṩ�˿���ֱ�ӷ������������ݵ����á�

```
import Immtable from 'immutable';
import Cursor from 'immutable/contrib/cursor';

let data =Immtable.fromJs({a: {b: {c:1}}});
// ��cursorָ�� { c:1};
let cursor =Cursor.from(data,['a','b'],newData => {
	// ��cursor ������cursorִ��updateʱ����
	console.log(newData);
});
cursor.get('c'); // 1
cursor =cursor.update('c', x =>x +1);
cursor.get('c'); // 2
```

#### ʵ��

**��React����ʹ�ã�Pure Render**

��ϤReact�Ķ�֪����React�������Ż�ʱ��һ�������ظ���Ⱦ�Ĵ��У�����ʹ��```shouldComponentUpdate()```������Ĭ�Ϸ���```true```����ʼ�ջ�ִ��```render()```������Ȼ����Virtual DOM�Ƚϣ����ó��Ƿ���Ҫ����ʵDOM���£���������������ܶ಻��Ҫ����Ⱦ����Ϊ����ƿ����

��Ȼ����Ҳ������```shouldComponentUpdate()```��ʹ��deepCopy��deepCompare�����ⲻ��Ҫ��``render``����**deepCopy**��**deepCompare**һ�㶼��**�ǳ������ܵ�**��

**Immutable ���ṩ�˼���Ч���ж������Ƿ�仯�ķ���**��ֻ��``===``��``is``�ȽϾ���֪���Ƿ���Ҫִ��``render()``�����⼸������������ɱ������Կ��Լ���������ܡ��޸ĺ��```shouldComponentUpdate```�������ģ�

```js
	import {is} from 'immutable';
	shouldComponentUpdate :(nextProps,nextState)=>{
		return !(this.props === nextProps || is(this.props, nextProps)) ||
			   !(this.state === nextState || is(this.state, nextState));
	}
```

ʹ��Immutable������ͼ������ɫ�ڵ��state�仯�󣬲�������Ⱦ�������нڵ㣬����ֻ��Ⱦͼ����ɫ�Ĳ��֣�

![](http://img.alicdn.com/tps/i3/TB1VinpKXXXXXXAXpXXZ_OdNFXX-715-324.png)

��Ҳ���Խ��� ```React.addons.PureRenderMixin``` ��֧��class�﷨��[pure-render-decorator](https://yq.aliyun.com/articles/16969)��ʵ�֡�

**setState��һ������**

React�����```this.state```����Immutable�ģ�����޸�ǰ��Ҫ��һ��deepCopy���Ե��鷳��

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
		// ������治��cloneDeep�������ӡ�Ľ�����Ѿ���1���ֵ��
		console.log(this.state.data.times);
	}
})
```

ʹ��Immutable��

```js
getInitialState(){
	return {
		data: Map({times: 0})
	}
},
handleAdd(){
	this.setState({data: this.state.data.update('times', v => v + 1)});
	// ��ʱ��times������ı�
	console.log(this.state.data.get('times'));
}
```

�����```handleAdd```���Լ�д�ɣ�

```
handleAdd(){
	this.setState(({data})=> ({
		data: data.update('times', v => v + 1)}
	));
}
```

**��Flux����ʹ��**

���� Flux ��û���޶� Store �����ݵ����ͣ�ʹ�� Immutable �ǳ��򵥡�

������ʵ��һ�����ƴ�����Ӻͳ������ܵ� Store��

```js
import {Map , OrderedMap } from 'immutable';
let todos =OrderedMap();
let history =[];  // ��ͨ���飬���ÿ�β��������������

let TodoStore =createStore({
	getAll(){ return todos; }
});
Dispatcher.register(action => {
	if(action.actionType === 'create'){
		let id =createGUID();
		history.push(todos);  //��¼��ǰ����ǰ�����ݣ����ڳ���
		todos = todos.set(id, Map({
			id: id,
			complete: false,
			text: action.text.trim()
		}));
		TodoStore.emitChange();
	}else if(action.actionType === 'undo'){
		// �����ǳ�������ʵ�֣�
		ֻ���history������ȡǰһ��todos����
		if(history.length >0){
			todos =history.pop();
		}
		TodoStore.emitChange();
	}
})
```

**��Redux����ʹ��**

Redux ��Ŀǰ���е� Flux �����⡣������ Flux �ж�� Store �ĸ��ֻ��һ�� Store�����ݲ���ͨ�� Reducer ��ʵ�֣�ͬʱ���ṩ�����������ĵ�����������View -> Action -> Middleware -> Reducer����Ҳ�����ڿ���ͬ��Ӧ�á�Ŀǰ�Ѿ���������Ŀ�д��ģʹ�á�

���� Redux �����õ� ```combineReducers``` �� reducer �е� ```initialState``` ��Ϊԭ���� Object �������Բ��ܺ� Immutable ԭ������ʹ�á�

���˵��ǣ�Redux �����ų�ʹ�� Immutable�������Լ���д ```combineReducers``` ��ʹ�� [redux-immutablejs](https://github.com/indexiatech/redux-immutablejs) ���ṩ֧�֡�

���������ᵽ Cursor ���Է�������� update �㼶�Ƚ�������ݣ�����Ϊ Redux ���Ѿ����� select ����������Action ���������ݣ���� Cursor �������û������֮���ˡ�

#### �ܽ�

Immutable ���Ը�Ӧ�ô���������������������Ƿ�ʹ�û�Ҫ����Ŀ��������������Խ�ǿ������Ŀ����Ƚ����ף�����ĿǨ����Ҫ����Ǩ�ơ�����һЩ�ṩ���ⲿʹ�õĹ����������ò�Ҫ�� Immutable ����ֱ�ӱ�¶�ڶ���ӿ��С�

#### �ο�

- [Lee Byron - Immutable Data and React]
- [Immutable Data Structures and JavaScript](http://jlongster.com/Using-Immutable-Data-Structures-in-JavaScript)






