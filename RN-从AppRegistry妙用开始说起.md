在构建React Native项目中，我们常常使用官方提供的脚手架`react-native-cli`，优点是能快速启动一个强大完整的RN项目，缺点就是失去了自己搭建脚手架带来的那份快乐，也常常会忽略掉一些脚手架内核心API。今天我将介绍一个不起眼却强大的API - `AppRegistry`。

![React Native AppRegistry](https://upload-images.jianshu.io/upload_images/10386532-53572d8c04360137.png?imageMogr2/auto-orient/strip|imageView2/2/w/1034/format/webp)

`AppRegistry`表示所有React Native应用的JS入口，运行一个RN项目大致会进行下述操作，`AppRegistry.registerComponent`注册根组件， `AppRegistry.runApplication`运行代码包。

## 查看方法

- registerComponent
- setWrapperComponentProvider
- registerHeadlessTask/startHeadlessTask

## registerComponent()
1.注册App根组件（react-native-cli根组件App.js），常用于**单入口项目**

```
import {AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json'; //唯一的入口名称

AppRegistry.registerComponent(appName, () => App);
```

2.动态插入组件到根组件[1]
由于`registerComponent()`是一个全局静态方法，所以我们可以在已经注册过入口的组件内任意地方调用该方法，只要保持`appKey`一致性，就能随时改变根组件，但我们一般不会一下就把整个根组件替换，常用于在根组件基础上动态插入一些全局组件，以方法调用的形式控制隐藏和显示。比如我们可以非原生地实现使用`Toast.show()`方法显示自定义Toast组件。

```
//将已被注册过的赋予变量名，防止重复注册报错
const originRegister = AppRegistry.registerComponent; 

AppRegistry.registerComponent = ( appKey,component)=>{
    return originRegister(appKey,function(){
        const OriginAppComponent = component();  //之前被注册过的根组件

        return class extends Component {
            render() {
                return (  //返回一个全新的根组件
                    <View style={styles.container}
                          onStartShouldSetResponder={()=>Toast.hide()}//手动隐藏toast
                    >
                        <OriginAppComponent/>
                        <Toast/>  //这里添加自定义Toast全局组件
                    </View>
                );
            };
        };
    })
};
```

**tips:**
缺点是根组件动态添加的组件无法使用`react-redux/connect`，因为被`<Redux.Provider>`包裹的组件与其是平级关系。

## setWrapperComponentProvider()

1.包装根组件[2]
上述我们提到使用`registerComponent`动态改变根组件，如果你有尝试过会发现`RootView.show()`方法不能用在`constructor`内。除了使用`registerComponent`封装一个高阶组件返回新的根组件，还可以直接使用`setWrapperComponentProvider`来包装根组件。

## registerHeadlessTask/startHeadlessTask
`registerHeadlessTask`是在APP未开启的情况下在JavaScript层运行任务的方法。例如：同步新数据，处理推送通知或播放音乐。
常和`startHeadlessTask`一起使用。

#### 待续

## 参考
- [1] [react-native-siblings](https://github.com/magicismight/react-native-root-siblings)
- [2] [react-native-root-siblings](https://github.com/magicismight/react-native-root-siblings)







