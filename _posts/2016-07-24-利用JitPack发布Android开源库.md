---
layout: post
title: 利用JitPack发布Android开源库
categories: Android
excerpt: 利用JitPack发布Android开源库
keywords: Android, JitPack
---

## 发布步骤
Gradle版本需在2.4以上
1、在项目的根节点的 build.gradle中添加如下代码:
```java
buildscript { 
  dependencies {
    classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5' // Add this line
```
2、在要发布的library的 build.gradle中添加如下代码：
```java
apply plugin: 'com.github.dcendents.android-maven'  
group='com.github.YourUsername'
```
3、提交项目到GitHub仓库
4、 Release仓库,填写版本号等信息
5、将仓库地址提交到JitPack
[JitPack地址](https://jitpack.io/)
![](/images/posts/android/jitpack01.png)

上传完成之后，JitPack会自动生成引用该仓库的配置信息:
![](/images/posts/android/jitpack02.png)

## 参考文章
[Publish an Android library](https://jitpack.io/docs/ANDROID/)
[优雅的发布Android开源库(论JitPack的优越性)](https://github.com/GcsSloop/AndroidNote/blob/master/Course/ReleaseLibraryByJitPack.md)
