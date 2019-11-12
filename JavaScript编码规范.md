## 笛风新平台前端JavaScript编码规范起草

#### 编码规范

在团队协作过程中，编码规范的作用想必不用多说，本文主要介绍如何选择及相应的辅助工具，最大程度上保证代码质量。

##### 选择标准

+ 认可度高，该规范现在已经或者即将成为国际 JavaScript 标准了
+ 支持项目的技术选型
+ 完备的插件支持

参考以上几条标准， airbnb 团队在 GitHub 维护的 [airbnb/javascript](https://github.com/airbnb/javascript) 瞬间脱颖而出。

+ 目前 GitHub 上最有热度的规范，有恐怖的 35000+ 个 star
+ 支持 ES6、React 等
+ 官方提供 ESlint、JSCS 插件支持

##### 规则配置

airbnb 团队在分享编码风格的同时，还提供了与之对应的 eslint 配置文件。他们提供的配置文件，根据使用环境不同，分三种：

1 `eslint-config-airbnb`  

支持`ECMAScript 6+`和`React`

- `npm install --save-dev eslint-config-airbnb eslint-plugin-import eslint-plugin-react eslint-plugin-jsx-a11y eslint`

- add `"extends" : "airbnb"` to your `.eslintrc` 

2 `eslint-config-airbnb-base`

支持`ECMAScript 6+`

- `npm install --save-dev eslint-config-airbnb-base eslint-plugin-import eslint`

- add `"extends": "airbnb-base"` to your `.eslintrc`

3 `eslint-config-airbnb-base/legacy`

支持ES5

- `npm install --save-dev eslint-config-airbnb-base eslint-plugin-import eslint`

- add `"extends": "airbnb-base/legacy"` to your `.eslintrc`

##### Eslint Parser

如果程序中还有ES5的写法，建议使用`base-eslint`作为eslint的eslint的parser,只要在`.eslintrc`中加入如下代码即可。

```js
"parser": "babel-eslint"
```

##### 可视化提示

借助webstorm提供的ESlint插件，在`Setting\Languages&Frameworks\JavaScript\Code Quality Tools\ESlint `选中Enbale即可，webstorm会自动查找`.eslintrc`和`pacjage.json`，来对js文件进行lint处理，另外，还可以自定义eslintConfig文件，和Additional rules 。

webstorm除了可以进行lint操作，还可以自动对一些简单的错误和警告进行修复，右击选项`fix ESlint problems`，这样可以大大减少我们修复编码规范的时间，不得不说，webstorm真是前端开发的神器啊。
