---
layout: post
title: Annotation常用注解使用说明
excerpt: Annotation常用注解使用说明
categories: Android
keywords: Android, Annotation
---
AndroidAnnotations是一个开源框架，通过使用它开放出来的注解api，可以大大的减少无关痛痒的代码量，简洁代码。
[官方文档_github链接](https://github.com/excilys/androidannotations/wiki/AvailableAnnotations)

## 第三方库导入

目前最新版本为4.0.0
### 在app/目录下的build.gradle（局部gradle）中添加下面配置：
```
applyplugin:'com.android.application'
**applyplugin:'android-apt'
defAAVersion='4.0.0'**

android{
compileSdkVersion23
buildToolsVersion"23.0.2"

defaultConfig{
applicationId"com.xxx.demo"
minSdkVersion18
targetSdkVersion23
versionCode1
versionName"1.0"
}

buildTypes{
release{
minifyEnabledfalse
proguardFilesgetDefaultProguardFile('proguard-android.txt'),'proguard-rules.pro'
}
}
}

dependencies{
compilefileTree(dir:'libs',include：['*.jar'])
testCompile'junit:junit：4.12'
compile'com.android.support:appcompat-v7：23.1.1'
****apt"org.androidannotations:androidannotations：$AAVersion"
    compile "org.androidannotations:androidannotations-api：$AAVersion"****
}

**apt{
arguments{
androidManifestFilevariant.outputs[0].processResources.manifestFile
resourcePackageName"com.xxx.demo"(你项目的包名)
}
}**
```
项目包名可在AndroidManifest.xml中的package确认。

### 在gradle/目录下的build.gradle文件（全局gradle）中添加下面粗体字配置：

buildscript{
repositories{
jcenter()
}
dependencies{
// replace with the current version of the Android plugin   
classpath'com.android.tools.build:gradle:1.5.0'
// replace with the current version of the android-apt plugin
**classpath'com.neenbedankt.gradle.plugins:android-apt：1.4+'**
}
}

allprojects{
repositories{
jcenter()
}
}

taskclean(type:Delete){
deleterootProject.buildDir
}

## 常用注解
### 组件注解
```java
@EActivity(R.layout.acitvity_main)
public class MainActivity extends Activity{
...
}
```
常用的有@EActivity、@EFragment、@EService等，进行注解了的组件才可使用其他注解功能。


### 资源引用的注解
```java
 @ViewById（R.id.tv_title）//此处可去掉括号部分
 TextView tv_title;

 @ViewById
 ImageView img_menu;

 @ViewById
 RelativeLayout rl_light;

 @Extra
 String mTitle;
 
 @StringRes(R.string.hello)
 String hello;
```
简单的控件绑定，资源文件中的id与控件名一致即可不在注解后加上括号及对应控件的id，@Extra也是。其他地方需要声明控件id的皆同理。
当View相关的成员变量初始化完毕后，会调用拥有@AfterViews注解的方法，可以在里面初始化一些界面控件等。


### 事件绑定
```java
 @Click
    void img_back() {     
        finish();
        overridePendingTransition(R.anim.zoom_in, R.anim.zoom_out);
    }
```
还有@TextChange、@ItemClick、@SeekBarProgressChange等。

### 比较方便的一些注解
#### 异步线程与UI线程
```java
@UiThread
void doSomething(){
...
}

@Background
void doSomething(){
...
}
```
UI线程执行的方法加个@UiThread，异步线程方法加个@Background，两者的交互就是方法直接的相互调用，不用再使用Handler去发送接收Message了。

#### 广播接收
```java
 @Receiver(actions = Utils.ACTION_BLE_DISCONNETED)
    public void bleDisconnect() {
       ...
    }

 @Receiver(actions = Utils.ACTION_UPDATE_WATER_SHOW)
 public void updateWaterShow(@Receiver.Extra(Utils.VALUE_ADDRESS) long water) {
     if (switchIsOpen)
         edt_water.setText(water + "");
 }
 
```
注册广播接收，简单搞定，不需要其他操作。相比下传统的方式：
```java
Private final BroadcastReceiver mGattUpdateReceiver = newBroadcastReceiver(){
	@Override
	public void onReceive(Contextcontext,Intentintent){
	  final Stringaction=intent.getAction();
	  if(Stringaction.equal(Utils.ACTION_STOP_SCAN)){
        ...   
      }
	}
};

private IntentFilter makeGattUpdateIntentFilter() {
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction(Utils.ACTION_STOP_SCAN);
return intentFilter;
}

registerReceiver(mGattUpdateReceiver,makeGattUpdateIntentFilter());

unregisterReceiver(mGattUpdateReceiver);

```
瞬间简洁了很多吧

#### SharedPreferences
直接使用@SharedPref可以简单地使用SharedPreferences的功能。
首先，建一个类存放需要存取的数据：
```java
@SharedPref(value=SharedPref.Scope.UNIQUE)
public interface MyPrefs {
    @DefaultBoolean(true)
    boolean isFirstIn();

    @DefaultString("")
    String ignoreVersion();

    @DefaultInt(0)
    int shockLevel();

}
```
括号后面的是默认值，接下来就是简单的使用了,首先在用到的类里声明：
```java
 @Pref
 MyPrefs_ myPrefs;

 boolean isFirstIn = myPrefs.isFirstIn().get();
 myPrefs.isFirstIn().put(false);
```
使用起来特别方便，需要特别说明的是，**这些数据要在一些不同的组件中同步共享，需在@SharedPref加上(value=SharedPref.Scope.UNIQUE)**，之前在activity和service中的数据总是对不上，找了好久才找到原因。

#### @EBen
想要在普通的类中也用上注解，只需在类名加上@EBean
```java
@EBean
public class MyClass {
  @UiThread
  void updateUI() {
}
```
使用时，声明：
```java
@EActivity
public class MyActivity extends Activity {
  @Bean
  MyClass myClass;
}

```
有一些要注意的是：
@EBean注解的类，只能有一个构造方法，且这个构造方法必须无参数或者只有context参数。
在activity等组件内声明了后，不用再去new这个类，否则会出错。

## 总结
比较常用的一些方法及说明大概就是这些，当然Annotation还有不少东西，想要了解得更深入可以到文首的链接处查看官方的使用说明，进一步了解！
