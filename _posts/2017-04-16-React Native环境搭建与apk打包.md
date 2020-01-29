---
layout: post
title: React Native环境搭建与apk打包
categories: React-Native
description: 本文是参考google官方发布的MVP架构demo以及前人对MVP实现方式的一些总结。
keywords: React-Native
---
以下内容为在window下开发React Nativ时环境搭建及APK打包的笔记。

## 环境搭建
1、安装Node.js与Git

2、如果没有合适的梯子，先设置镜像
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global

3、安装React Native命令行工具
github上下载后解压，[下载地址](https://github.com/facebook/react-native)
在react-native-cli目录下用git运行命令：
npm install -g react-native-cli
或者
npm install -g yarn react-native-cli

4、创建项目
react-native init 文件夹名称

5、运行packager 进入工程目录
react-native start

可以用浏览器访问http://localhost:8081/index.android.bundle?platform=android

6、准备模拟器或真机 运行android
react-native run-android

7、gradle-2.14.1-all.zip下载慢的解决方法
- 首先 把对应版本的gradle载到本地任意一个磁盘里
- 然后拖曳这个文件夹到浏览器中，就会得到 file:///...的访问地址
- 替换项目中 android/gradle/wrapper/gradle-wrapper.properties 的 distributionUrl ，
- 即  distributionUrl=file\:///D:/gradle/gradle-2.14.1-all.zip （注意这里需要加上转义字符\）

8、出现红色提示，没有连接上服务器，则在设置中输入电脑本机ip地址及8081端口号：如192.168.2.1：8081

至此，示例代码将运行在android上

## 打包发布(android)
1、找到路径/android/app/src/main，并在该目录下新建assets文件夹

2、在工程目录下将index.android.bundle下载并保存到assets资源文件夹中，运行以下命令：

`curl -k "http://localhost:8081/index.android.bundle" > android/app/src/main/assets/index.android.bundle`

这里需先安装curl：[Windows下安装使用curl命令](http://jingyan.baidu.com/article/a681b0dec4c67a3b1943467c.html)

3、添加gradle的android keystore配置
 //签名
`signingConfigs{
    release {
        storeFile file("/my-release-key.keystore")
        storePassword "密码"
        keyAlias "keyAlias的名字"
        keyPassword "密码"
    }
}
 buildTypes {
    release {
        minifyEnabled enableProguardInReleaseBuilds
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        signingConfig signingConfigs.release //添加这句话引用签名配置
    }
}`
4、启用Proguard代码混淆来缩小APK文件的大小
def enableProguardInReleaseBuilds = true

5、在/android/目录中执行gradle assembleRelease命令，打包后的文件在 android/app/build/outputs/apk目录中，例如app-release.apk。如果打包碰到问题可以先执行 gradle clean 清理一下。

这里需先安装gradle工具（版本与android\gradle\wrapper下的一致），并配置环境变量，配置GRADLE_HOME到你的gradle根目录当中，然后把%GRADLE_HOME%/bin（linux或mac的是$GRADLE_HOME/bin）加到PATH的环境变量。
配置完成之后，运行gradle -v，检查一下是否安装无误
