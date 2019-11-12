## Javascript中的new关键字

Javascript中，实例化一个对象，会用到new关键字。

经常有人会问我，对于一个函数，什么时候该使用new关键字。

在回答这个问题之前，需要先了解清楚new的本质，在调用new Function的时候，new做了什么操作。

先看如下代码：

```js
	function classA(){
		this.name=1;
	}
	classA.prototype.show = function(){
		alert(this.name)
	}
	var b =new classA();
	b.show();
```

``` var b =new classA();```

这句中，new做了以下几件事情。

1 创建一个新的对象，这个对象的类型是object;
2 查找class的prototype上的所有方法、属性，复制一份给创建的Object
3 将构造函数classA内部的this指向创建的Object
4 创建的Object的__proto__指向class的prototype
5 执行构造函数class
6 返回新创建的对象给变量b

这个流程应该比较好理解的。这里再解释一下：

1 构造函数：我们一般把new 后面的函数称为构造函数，如new classA()，其中classA就为构造函数
2 第四点的__proto__，可能比较难理解。

每个对象都会在其内部初始化一个属性，就是__proto__，可以在chrome中的调试器里写个对象查看下。**当我们访问一个对象的属性时，如果这个对象内部不存在这个属性，那么他就会去__proto__里找这个属性，这个__proto__又会有自己的__proto__，于是就这样一直找下去，也就是我们平时所说的原型链的概念。

当我们调用b.show()时，首先b中没有show这个属性，于是，它就需要到它的__proto__中去找，也就是ClassA.prototype，
而我们在上面定义了ClassA.prototype.show=function(){}; 于是，就找到了这个方法。

再用下面的代码来理解

```js
	var b={};
	b.__proto__ =classA.prototype;
	classA.call(b);
```

最后再用一句话总结：new关键字以classA()为模板创建了一个新的对象，它复制了classA构造器中的所有成员变量，同时this指向新创建的对象。

有2点需要注意：

1 如果构造函数内没有返回值，则默认是返回this（当前上下文），要不然就返回**任意非原始类型值**。
2 如果不用new关键字，如

```js
	var b = classA();
```

**则classA值会返回undefined**,并且**this(执行上下文)是window对象**。
也就是说**如果你不new的话，this指的就是window全局对象了**。
如果要理解这点，需要理清楚this的指向。

####this的指向：

1 总的来说，this在函数内部使用，用来引用包含函数的对象，而不是函数本身。
2 当this值的宿主函数被封装在另一个函数的内部或在另一个函数的上下文中被调用时，this值将永远是对head对象的引用（即全局window对象）
3 使用new关键字调用构造函数，this引用"即将创建的对象"

如下面代码：

```js
	function classA(){
		this.name=1;
	}
	classA();
	console.log(name);
```
此时输出`` 1 ``

看到这里，相信可以都了解new的用法。