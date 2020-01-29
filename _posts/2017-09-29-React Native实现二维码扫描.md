---
layout: post
title: React Native实现二维码扫描
categories: React-Native
excerpt: React Native实现二维码扫描
keywords: React-Native,二维码
---
基于[react-native-camera](https://github.com/lwansbrough/react-native-camera)

## 依赖包导入步骤
详细步骤可看github上的说明，简要记录如下:
1. npm install react-native-camera --save
2. react-native link react-native-camera

android手动link方式:
1. android/settings.gradle加入：
   include ':react-native-camera'
   project(':react-native-camera').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-camera/android')
2. android/app/build.gradle加入:
   compile project(':react-native-camera')
3. getPackages()方法中加入new RCTCameraPackage()

## 使用方法
### 直接使用Camera作为容器
```java
 <Camera style={styles.container}
         onBarCodeRead={this.onBarCodeRead}
      >              
       {this.renderOtherView()}
       ...
  </Camera>
```
### 返回二维码扫描结果
```java
onBarCodeRead(data) {
        //将返回的结果转为对象
        var result = JSON.parse(data.data);
        console.log(result);
    }
```

### 二维码扫描框
```java
 this.state = {
           animatedValue: new Animated.Value(0),
        };

 componentDidMount() {
        this.scannerLineMove();
    }

//扫描框
 renderQrScanView() {
        const animatedStyle = {
            transform: [
                {translateY: this.state.animatedValue}
            ]
        };
        return (
            <View style={styles.container}>            
                <View style={styles.middleView}>
                    //下面四个view为扫描框的四个角
                    <View style={[styles.topLeftCorner, {
                        borderLeftWidth: 2,
                        borderTopWidth: 2,
                    }]}/>
                    <View style={[styles.topRightCorner, {
                        borderRightWidth: 2,
                        borderTopWidth: 2,
                    }]}/>
                    <View style={[styles.bottomLeftCorner, {
                        borderLeftWidth: 2,
                        borderBottomWidth: 2,
                    }]}/>
                    <View style={[styles.bottomRightCorner, {
                        borderRightWidth: 2,
                        borderBottomWidth: 2,
                    }]}/>
                    //扫描条
                    <Animated.View
                        style={[animatedStyle, {alignItems: 'center'}]}>
                        <View style={[{
                            backgroundColor: '#5E8EF8',
                            height: WindowUtil.pxToDp(6),
                            width: WindowUtil.pxToDp(590) - 4,
                        }]}/>
                    </Animated.View>
                </View>
                <View style={styles.middleRightView}/>
            </View>
        )
    }


   /**
     * 扫描条动画
     */
    scannerLineMove() {
        this.state.animatedValue.setValue(0);  //重置Rotate动画值为0
        Animated.timing(this.state.animatedValue, {
            toValue: WindowUtil.pxToDp(middleViewHeight),
            duration: 2000,
            easing: Easing.linear
        }).start(() => this.scannerLineMove());
    }

//部分style如下：
topLeftCorner: {
        borderColor: 'white',
        width: WindowUtil.pxToDp(100),
        height: WindowUtil.pxToDp(100),
        position: 'absolute',
        top: 0,
        left: 0,
    },

    topRightCorner: {
        borderColor: 'white',
        width: WindowUtil.pxToDp(100),
        height: WindowUtil.pxToDp(100),
        position: 'absolute',
        top: 0,
        right: 0,
    },

    bottomLeftCorner: {
        borderColor: 'white',
        width: WindowUtil.pxToDp(100),
        height: WindowUtil.pxToDp(100),
        position: 'absolute',
        bottom: 0,
        left: 0,
    },

    bottomRightCorner: {
        borderColor: 'white',
        width: WindowUtil.pxToDp(100),
        height: WindowUtil.pxToDp(100),
        position: 'absolute',
        bottom: 0,
        right: 0,
    },

```

