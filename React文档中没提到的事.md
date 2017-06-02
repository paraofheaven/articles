## React�ĵ���û�ᵽ����

**ǰ��**

�Ժܶ� react ������˵���������ҵ�����Դ����Щ�򵥵� tutorial ��React�ĵ�Ҳ�Ƿǳ�ֵ��һ���ģ�Ӣ�İ�����ŷǳ���ϸ�Ľ��ܡ������̻ܽ������ʹ�� react �����������������ô��ʵ����Ŀ�����ŵ���֯�ͱ�д react ���롣�ùȸ������ġ� React ���ʵ��������ǰ��ҳ����ȫ����ͬһƪ�������µ�����...�������ܽ������Լ���ȥ�Ǹ���Ŀʹ�� React �ȹ���һЩ�ӣ�Ҳ������һЩ���˵Ĺ۵㣬ϣ���Բ��� react ʹ�����а�����

#### React��AJAX

Reactֻ������View��һ�㣬�������漰��������/AJAX���������������������������⣺

> - ��һ����ʲô�����ӷ���˻�ȡ���ݣ�
> - �ڶ�����ȡ��������Ӧ�÷���react�����ʲôλ�á�
 
React�ٷ��ṩ��һ�ֽ��������[Load Initial Data via AJAX](https://facebook.github.io/react/tips/initial-ajax.html)

ʹ��jQuery��Ajax��������һ�������``componentDidMount()``�з�ajax�����õ������ݴ�������Լ���state�У�������setState����ȥ����UI��������첽��ȡ���ݣ�����``componentWillUnmount``��ȡ����������

���ֻ��Ϊ��ʹ��jQuery��Ajax��������������jQuery�⣬����һ���˷��ּӴ�������Ӧ�õ�����������ǻ���ʲô������ѡ������ʵ�����кܶ�ģ�[fetch()](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch)��[fetch polyfill](https://github.com/github/fetch) ��[axios](https://github.com/mzabriskie/axios)...��������Ҫ���ǹ�ע����``window.fetch()``,����һ����ࡢ��׼����javascript��Ajax API����Chrome��Firefox���Ѿ�����ʹ�ã������Ҫ�������������������ʹ��fetch polyfill��

React�ٷ��ĵ�ֻ������������һ����һ��������ͨ��ajax�ӷ������˻�ȡ���ݣ�����û�и���������һ��������ʵ����Ŀ�е���Ӧ�ð����ݴ�����Щ����У��ⲿ�����ȱ���淶�Ļ����ᵼ��������Ŀ��û��ҡ�����ά��������������ֱȽϺõ�ʵ����

1. ���е���������͹��������Ψһ��һ�������
 
�ø����/��������з������е�ajax���󣬰Ѵӷ���˻�ȡ������ͳһ�������������state�У���ͨ��props�����ݴ�������������ַ�����Ҫ�������������Ǻܸ��ӵ�С��Ӧ�á�ȱ����ǵ�������Ĳ㼶������Ժ���Ҫ������һ��һ��ش����������д�����鷳������Ҳ���á�

2. ���ö���������ר�Ŵ�����������͹���

��ʵ����һ�ַ������ƣ�ֻ�������ö������������������������״̬��������������Ҫ�������ֲ�ͬ���͵������һ����չʾ�������presentational component������һ���������������container component����չʾ���������ӵ���κ�״̬�����е����ݶ�����������л�ã�����������з���ajax�������߸���ϸ�������������Ķ�����ƪ���£�[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.ch9xqg6s4)

һ����������ӣ�

����������Ҫչʾ�û���������ͷ�����ȴ���һ��չʾ�����``<UserProfile />``,����������Props��``name``��``profileImage``���������ڲ�û���κι���Ajax�Ĵ��롣

Ȼ�󴴽�һ���������``<UserProfileContainer />``��������һ��userId�Ĳ���������Ajax����ӷ�������ȡ���ݴ���``state``�У���ͨ��``props``����``<UserProfile />``�����

3. ʹ��Redux��Flux�����

Redux����״̬�����ݣ�Ajax�ӷ������˻�ȡ���ݣ����Ժ���Ȼ������ʹ����Reduxʱ��Ӧ�ð����е��������󶼽���redux�������������˵��Ӧ���Ƿ���Async Actions�������������Flux��Ļ��������ʽ����࣬������actions�з�����������ʹ��promise fetch����jquery��ajax�Ϳ���Ŀ����Ҫ���Լ���ϲ���ˡ�

#### ʹ�� PropTypes �� getDefaultProps()

> 1. һ��ҪдPropTypes����ĪΪ��ʡ�¶���д
> 2. ���һ��Props����requied��һ����getDefaultProps��������
``React.PropTypes``��Ҫ������֤������յ���props�Ƿ�Ϊ��ȷ���������ͣ��������ȷ��console�оͻ���ֶ�Ӧ��warning���������ܷ���Ŀ��ǣ����APIֻ�ڿ���������ʹ�á�

����ʹ�÷�����

```js
propTypes: {
    myArray: React.PropTypes.array,
    myBool: React.PropTypes.bool,
    myFunc: React.PropTypes.func,
    myNumber: React.PropTypes.number,
    myString: React.PropTypes.string��
     
     // You can chain any of the above with `isRequired` to make sure a warning
    // is shown if the prop isn't provided.
    requiredFunc: React.PropTypes.func.isRequired
} 
```

��������props�����������ͣ�����ӵ�и��ӽṹ�Ķ�����ô�죿�������������

```js
{
  text: 'hello world',
  numbers: [5, 2, 7, 9],
}
```

��Ȼ�����ǿ���ֱ����``React.PropTypes.object``,���Ƕ����ڲ�����������ȴ�޷���֤��

```js
propTypes: {
  myObject: React.PropTypes.object,
}
```

����ʹ�÷�����``shape()`` �� ``arrayOf()``

```js
propTypes: {
  myObject: React.PropTypes.shape({
    text: React.PropTypes.string,
    numbers: React.PropTypes.arrayOf(React.PropTypes.number),
  })
}
```

������һ�������ӵ�Props��

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

�ۺ����棬д����Ӧ�þͲ����ˣ�

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

#### �Ѽ���������ж϶����� render() ������

1 �����state�в��ܳ���props

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

��Ȼ�����ӵ�display logicҲӦ�ñ���ȫ�ѷ���render()�У���Ϊ�������ܵ�������render()�������ӷ�ף������š����ǿ��԰�һЩ���ӵ��߼�ͨ��helper function�Ƴ�ȥ��

```js
// GOOD: helper function
renderFullName () {
  return `${this.props.firstName} ${this.props.lastName}`;
}

render () {
  var fullName = this.renderFullName();
}
```

2 ����state�ļ�࣬��Ҫ���ּ��������state

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

3 ������Ԫ�жϷ����Ͳ���If��ֱ�ӷ���render()��

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

4 ����ֵ�����ܸ㶨�ģ�����IIFE��

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

5 ��Ҫ��display logicд��``componentWillReceiveProps``��``componentWillMount``�У������Ƕ��Ƶ�render()��ȥ��

#### ��ζ�̬���� classNames

1 ʹ�ò���ֵ

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

2 ʹ��[classnames](https://github.com/JedWatson/classnames)���С������ƴ��classNames��

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
// AFTER��
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

























