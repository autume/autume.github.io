---
layout: post
title: Reactnative基础 创建组件的三种方式 
categories: React-Native
excerpt: Reactnative基础 创建组件的三种方式 
keywords: React-Native
---
# 方式一 ES6创建组件的方式
```
export default class HelloComponent extends Component { 
render() { 
    return (<Text style={{fontSize:20,backgroundColor:'red'}}>Hello sun</Text>);
}}
```
# 方式二 ES5创建组件的方式
```
var HelloComponent=React.createClass({ 
render() { 
    return ( <Text style={{fontSize:20,backgroundColor:'red'}}>Hello ES5创建组件的方式</Text> ); 
    }
}) 
module.exports=HelloComponent; //导出
```

# 方式三 函数式
-  无状态，无法使用this，因为其没有指针，没有生命周期方法
-  pros 为函数的参数
-  通过使用{pros.name}来获得从index.js中传过来的值， 
 ```
 function HelloComponent(pros){ 
     return ( <Text style={{fontSize:20,backgroundColor:'red'}}>Hello 函数式 {pros.name}</Text> ); 
 } 

 module.exports=HelloComponent; //导出
 ```
 
 在index.js中 
 ```
 import HelloComponent from './HelloComponent' //加入此行 
 
 export default class simple extends Component {
 render() { 
     return (
     <View style={styles.container}> 
         <HelloComponent name = "小明"/> //使用HelloComponent组件 
     </View> );
 } }
 ```
 第一种和第二种方式也可以通过{this.pros.name}来获得index.js中传过来的值
