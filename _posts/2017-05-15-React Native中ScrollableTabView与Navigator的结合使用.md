---
layout: post
title: React Native中ScrollableTabView与Navigator的结合使用
categories: React-Native
keywords: React-Native,ScrollableTabView,Navigator
---
React Native中ScrollableTabView使用Navigator跳转时，跳转后的页面仍存在ScrollableTabView导航栏，解决方案记录如下。

## 错误示例
ScrollableTabView放入各个tab显示页面，带有跳转功能的页面直接用Navigator包裹，导致跳转到其他页面后底部仍存在导航栏。
```java
render() {
        let tabNames = this.state.tabNames;
        let tabIconNames = this.state.tabIconNames;
        return (
            <ScrollableTabView         
                renderTabBar={() => <DfyTabBar tabNames={tabNames} tabIconNames={tabIconNames}/>}
                tabBarPosition='bottom'           
                locked={true}
                initialPage={0}
                prerenderingSiblingsNumber={1}
            >

                <Navigator
                    tabLabel="list"
                    initialRoute={{ name: 'list', component: List }}
                    //配置场景
                    configureScene=
                        {
                            (route) => {
                                return ({
                                    ...Navigator.SceneConfigs.PushFromRight,
                                    gestures: null
                                });
                            }
                        }
                    renderScene={
                        (route, navigator) =>
                        {
                            let Component = route.component;
                            return <Component {...route.params} navigator={navigator} />
                        }
                    } />

                <Edit tabLabel="edit" />
                <Picture tabLabel="pic" />
                <Account tabLabel="account"/>

            </ScrollableTabView>
        );
    }
```
## 解决方案
### 自定义TabBar组件
自定义一个组件，将tab中的页面放入该组件，具有跳转功能的也没传入相关的属性 {...this.props} 
```java
   render() {
        let tabNames = this.state.tabNames;
        let tabIconNames = this.state.tabIconNames;
        return (
            <ScrollableTabView
                // renderTabBar={() => <ScrollableTabBar/>}
                renderTabBar={() => <DfyTabBar tabNames={tabNames} tabIconNames={tabIconNames} />}
                tabBarPosition='bottom'
                locked={false}
                initialPage={0}
                prerenderingSiblingsNumber={1}
            >

                <List tabLabel="list"  {...this.props} />
                <Edit tabLabel="edit" />
                <Picture tabLabel="pic" />
                <Account tabLabel="account" />

            </ScrollableTabView>
        );
    }
```
### Navigator中直接放入TabBar
```java
 return (
            <Navigator
                {...this.props}
                initialRoute={{ name: 'TabBarView', component: TabBarView }}
                //配置场景
                configureScene=
                {
                    (route) => {
                        return ({
                            ...Navigator.SceneConfigs.PushFromRight,
                            gestures: null
                        });
                    }
                }
                renderScene={
                    (route, navigator) => {
                        let Component = route.component;
                        return <Component {...route.params} navigator={navigator} />
                    }
                } />
        );
```
## 总结
将Navigator包裹某个tab页面，则只是该页面进行跳转，因此底部导航栏仍存在；因此直接用Navigator包裹整个TabBar组件。


