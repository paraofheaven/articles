## State 数据格式设计中的思考

#### 前言

Redux中，State是整个应用的数据，本质上是一个普通对象。

State决定了整个应用的组件如何渲染，渲染的结果是什么。可以说，State是应用的灵魂，组件是应用的肉体。

所以，在项目开发初期，设计一份健壮灵活的State尤其重要，对后续的开发有很大的帮助。（请注意，并不强制要求所有的数据都保存到State中，有些属于组件的数据是完全可以留给组件自身去维护的。）

在设计State的过程中，对State进行拆分和改造是很有必要的，通常来说，我们可以从横向和纵向两个维度来对State进行拆分和改造。

#### 如何横向拆分State

通常，应用越庞大，State所包含的数据也就越多，数据结构也就越复杂。为了便于管理这复杂的数据结构，我们通常会根据数据类别做一个横向的拆分。
先来个 **不太好的示范** ，我们有首页列表、研究所列表：

```js
{
	homeList: [{},{}],
	paperList: [{},{}]
}
```

接下来，试想一下：还有其他类型的列表，比如tag列表、category列表：

```js
{
	homeList: [{},{}],
	paperList: [{},{}],
	tagList: [{},{}],
	categoryList: [{},{}]
}
```

所以我们的数据结构应该设计成这个样子吗？
这样的话，我们的State很快就会变得臃肿并且难以维护，并且下一步的Reducer设计会变得格外吃力。那我们尝试根据 **数据类别** 做一个横向拆分的设计：

```js
{
	// 文章相关的数据结构
	articles: {
		// 首页列表页
		list: [{},{}],
		// Tag列表页
		tagList: [{},{}]
	},
	// 研究所相关的数据结构
	paperList: {
		//研究所列表页
		list: [{},{}]
	} 
}
```

是的，本质上就是将同一类别的数据归档：

+ articles：state.articles，保存文章相关的数据，比如各种列表页
+ papers：state.papers，保存研究所相关的数据，比如各种列表页

那么，这样设计的好处是什么呢？

+ 分治法让State数据解耦，彼此间独立，我们可以对articles、papers分别管理。
+ 数据按类别区分，同一类数据的处理逻辑(Reducer)可以放到一起，代码上更便于维护。

有人会问了，那怎么样针对不同state管理不同的reducer呢？reducers怎么对articles、papers分别管理呢？可以参考 [Redux 文档](http://cn.redux.js.org/docs/basics/Reducers.html)

拆分State其实就是分治法，articles数据有一个专门的管理器，papers数据有一个专门的管理器，这样才能保证代码逻辑清晰，且彼此不关联。
数据分拆了，我们可以更细粒度的管理数据，这是横向的拆分，那纵向的问题怎么解决呢？所谓纵向，其实就是数据嵌套深的问题。

#### 如何纵向改造State？

##### 为什么要解决数据嵌套的问题呢？

举两个栗子你就清楚了。

- 场景一：有很多列表页，每个列表都存储了一份article/paper的详细字段，严重冗余，并且没有办法同步更新。

```js
{
	// 文章相关的数据结构，显然id=3的文章数据冗余了一份
	articles: {
        // 首页列表页
        list: [{ id: 1, title: xxx }, { id: 3, title: xxx }],
        // Tag列表页
        tagList: [{ id: 2, title: xxx }, { id: 3, title: xxx }]
    }
}
```

- 场景二：表结构复杂，想要增删改查都需要三次循环，简直是噩梦。

```js
// 首先是paper表，包含paper的相关信息，以及一个question数组
// 其次是question表，包含question的相关信息，以及一个option数组
// 最后才是option表，包含option的相关信息。
papers: [{
   id: xxx,
    title: xxx,
    questions: [{
        id: xxx,
        title: xxx,
        sequence: xxx,
        options: [{
            id: xxx,
            title: xxx,
            sequence: xxx
        }]
    }] 
}]
```

现在假如我们选中某个option，需要更新option的``selected=true``，怎么办呢？

我们需要三层循环，找到option，然后设置``selected=true``。而这仅仅是一个很简单的需求而已。类似的场景有很多，我们需要纵向的遍历数据结构，不仅性能差，代码冗余，维护起来也相当困难。

那么我们的想法就是数据扁平化，所谓的数据扁平化是什么意思呢？看以下代码就清楚了：

```js
// 扁平化之后的数据
papers: [id1, id2]
papersHash: {
    id1: {
        id: xxx,
        title: xxx,
        questions: [id1, id2]
    }
}
questionsHash: {
    id1: {
        id: xxx,
        title: xxx,
        sequence: xxx,
        options: [id1, id2]
    }
}
optionsHash: {
    id1: {
        id: xxx,
        title: xxx,
        sequence: xxx
    }
}
```

从上面的结构可以看到，扁平化之后，数据被抽离到一个hash对象中，根据``key-value`` 键值对可以轻松获取指定对象的值。而关联部分只存储id作为索引。
比如前面的场景，我们想要更新某个option，很简单，直接根据id更新optionsHash中的值即可。

#### 那么，如何才能简单的扁平化数据呢？

推荐 [normalizr.js](https://github.com/paularmstrong/normalizr) 这个库，使用方法也很简单，定义好``schema``，声明各个``schema``之间的关系，然后就OK啦。

简单给个``schema``的例子，更多的内容，大家可以去看官方文档。

```js
import { Schema, arrayOf, normalize } from 'normalizr';

const paperSchema =new Schema("papers");
const questionSchema =new Schema("questions");
const optionSchema =new Schema("options");

// 声明paper.quesitons字段，是一个questionSchema的数组
paperSchema.define({
	questions:arrayOf(questionSchema);
});

// 声明question.options字段，是一个optionsSchema的数组
questionSchema.define({
    options: arrayOf(optionsSchema)
});

// 比如我们获取到的数据是papers
let normalizeData = normalize(papers, arrayOf(paperSchema));

// 最后得到的normalizeData结果就是
// normalizeData = {
//     result: [id1, id2, ...],
//     entities: {
//         papers: {
//             id1: {}
//         },
//         questions: {
//             id1: {}
//         },
//         options: {
//             id1: {}
//         }
//     }
// }

```

到此为止，State的横向拆分（分治法）和纵向拆分（扁平化）都已经完成。
Redux给我们的启示是面向数据编程。所以开发之前根据实际需求设计好State的数据结构，是一个项目的灵魂。也影响和决定了后续的开发。

当然，开发初期遇到State需要调整，请大胆的调整，而产品趋于成熟，对State的调整应该越谨慎越好。

#### 总结

请记住，开发之前先梳理需求，然后设计State的数据结构，这个重要性就好比数据库设计之于后台。它影响和决定了后续的开发。

State本身就是一个普通对象，为了能够进行更细粒度的管理和维护，我们考虑从横向/纵向两个维度来对State进行拆分和改造。

+ 横向设计State：把State扔到一个超级reducer中是不明智的，我们应该根据数据类别对State进行拆分，为不同类别的数据提供专门的管理器。
+ 纵向设计State：尽量将State进行扁平化处理，这样可以更灵活的对数据进行增删改查，并且避免冗余数据的问题。

此外，State的本质就是普通对象，之所以如此“处心积虑”的设计State，无非是为了让开发速度更快，逻辑更清晰，维护更方便。
如果你的应用本身就很简单，那就不要过度设计State，甚至可以不引入Redux。