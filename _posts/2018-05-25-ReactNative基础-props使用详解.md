---
layout: post
title: Reactnative基础 Props使用详解 
categories: React-Native
excerpt: Reactnative基础 Props使用详解 
keywords: React-Native
---
## 什么是props
props是属性，它是为了描述组件的特征而存在的，它是父组件传递给子组件的。
## 如何使用props
属性是只读的；如果是上个页面传递过来的属性，就不能修改；但它可以在本页面定义默认属性。
ES6定义默认属性
```
static defaultProps={
    name:'小红'
}
```
这样，当父组件没有给子组件传递属性时，就可以使用默认属性。
## 属性的类型检查和约束
proptypes用来类型检查,.isrequired指定所必需的属性
```
static PropsTypes={ 
    name:PropsTypes.string, 
    sex:PropsTypes.string.isRequired 
}
```
## props使用技巧--延展操作符
当要传递很多个属性时,let params = {name:'张'，age:18, sex:'女' }; 使用的时候就是 
```
<PropsTest name={params.name} sex={params.sex}/>
```
使用延展操作符：
```
<PropsTest { ...params } />(使用 大括号里放三个点 ...，然后接着 params 就可以在下一个页面被使用了。 )
```
## props使用技巧--解构赋值
延展操作符是将属性全部进行赋值，但如果只想取出部分来进行赋值，就可以使用解构赋值。它比传统的方式好是它可以从一组属性中获取指定属性。
```
let params = {name:'张'，age:18, sex:'女' }; 
let {name,sex}=params; 
<PropsTest name={name} sex={sex} /> 
```
