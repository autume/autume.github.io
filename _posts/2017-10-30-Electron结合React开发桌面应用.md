---
layout: post
title: Electron结合React开发桌面应用
categories: React-Native
excerpt: Electron结合React开发桌面应用
keywords: React-Native,Electron
---

## 环境搭建
	前提是安装好npm和nodejs，安装以下开发环境，若安装失败则尝试翻墙或者使用淘宝镜像cnmp
### electron环境安装
	npm install -g electron-prebuilt
	npm install -g electron-packager
### create-react-app安装
	npm install -g create-react-app
	npm install -g react-scripts
	## 创建demo
### 在要创建工程的目录下使用命令create-react-app projectName创建工程:
	create-react-app desktop-demo
### 安装electron以及增加启动命令:
	使用命令npm install -save electron安装electron
	在脚本里添加启动命令"electron-start": "electron ." 
### 创建Electron的启动文件main.js
	在React项目的根目录创建main.js文件,可以直接拷贝[electron-quick-start](https://github.com/electron/electron-quick-start)仓库里的main.js
### package.json文件修改
	在React项目中的package.json文件中增加main字段，值为"main.js"，几homepage字段 "homepage":"."
### 修改main.js
	修改main.js中的win.loadURL，改为
	```java
	 mainWindow.loadURL(url.format({
	        pathname: path.join(__dirname, './build/index.html'),
	        protocol: 'file:',
	        slashes: true
	    }))
	```
	修改src目录下的APP.js则修改页面内容
### 运行
	在根目录下运行npm run build + npm run electron-start即可看到界面，
	报错：npm ERR! desktop-demo@0.1.0 build: `react-scripts build`
	运行npm install解决
	## 使用热调试
	在main.js文件中将启动页修改为 http://localhost:3000/：
	mainWindow.loadURL('http://localhost:3000/')
	开发时，需要启动两个终端，一个终端启动npm start， 另一个终端启动npm run electron-start，这样在electron中就可以热调试了。
	不过编译发布时需要改回从build/index.html，可以在package.json中定义个DEV字段，设置为true/false，然后修改main.js，代码如下：
	```java
	if(pkg.DEV) { 
	  win.loadURL("http://localhost:3000/")
	} else { 
	  win.loadURL(url.format({
	    pathname:path.join(__dirname, './build/index.html'), 
	    protocol:'file:', 
	    slashes:true 
	  }))
	}
	```
	运行npm run electron-start之前，根据需要修改DEV为true/false就行了
	
	## 代码
	最终代码如下
### package.json
	```java
	{
	  "name": "desktop-demo",
	  "version": "0.1.0",
	  "private": true,
	  "DEV":false,
	  "main": "main.js",
	  "homepage": ".",
	  "dependencies": {
	    "electron": "^1.7.9",
	    "npm": "^5.4.2",
	    "react": "^16.0.0",
	    "react-dom": "^16.0.0",
	    "react-scripts": "1.0.14"
	  },
	  "scripts": {
	    "start": "react-scripts start",
	    "build": "react-scripts build",
	    "test": "react-scripts test --env=jsdom",
	    "eject": "react-scripts eject",
	    "electron-start": "electron ."
	  }
	}
	
	```

### main.js：
	```java
	const electron = require('electron')
	// Module to control application life.
	const app = electron.app
	// Module to create native browser window.
	const BrowserWindow = electron.BrowserWindow
	
	const path = require('path')
	const url = require('url')
	const pkg = require('./package.json') // 引用package.json 
	
	// Keep a global reference of the window object, if you don't, the window will
	// be closed automatically when the JavaScript object is garbage collected.
	let mainWindow
	
	function createWindow() {
	    // Create the browser window.
	    mainWindow = new BrowserWindow({width: 800, height: 600})
	
	    // and load the index.html of the app.
	
	    if(pkg.DEV) { 
	    mainWindow.loadURL("http://localhost:3000/")
	} else { 
	  mainWindow.loadURL(url.format({
	    pathname:path.join(__dirname, './build/index.html'), 
	    protocol:'file:', 
	    slashes:true 
	  }))
	} 
	
	    // Open the DevTools.
	    // mainWindow.webContents.openDevTools()
	
	    // Emitted when the window is closed.
	    mainWindow.on('closed', function () {
	        // Dereference the window object, usually you would store windows
	        // in an array if your app supports multi windows, this is the time
	        // when you should delete the corresponding element.
	        mainWindow = null
	    })
	}
	
	// This method will be called when Electron has finished
	// initialization and is ready to create browser windows.
	// Some APIs can only be used after this event occurs.
	app.on('ready', createWindow)
	
	// Quit when all windows are closed.
	app.on('window-all-closed', function () {
	    // On OS X it is common for applications and their menu bar
	    // to stay active until the user quits explicitly with Cmd + Q
	    if (process.platform !== 'darwin') {
	        app.quit()
	    }
	})
	
	app.on('activate', function () {
	    // On OS X it's common to re-create a window in the app when the
	    // dock icon is clicked and there are no other windows open.
	    if (mainWindow === null) {
	        createWindow()
	    }
	})
	
	// In this file you can include the rest of your app's specific main process
	// code. You can also put them in separate files and require them here.
	
	```
