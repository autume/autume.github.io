---
layout: post
title: Android中使用SVG 
categories: Android
keywords: Android，SVG
---
## SVG简介
- SVG 的文件里存储了绘制图片的相关信息，在要用图的时候再把图画出来，所以在图片显示的时候会花费更多的时间消耗更多的资源。
- SVG 的文件体积远小于传统的位图文件，因为没有存储任何图像的像素信息。 
- SVG的文件画出来的图像是矢量图，所以不会存在失真的问题，理论上支持任何级别的缩放，适应性强于传统的位图。

## SVG简单使用
### 获取SVG文件
[SVG图片下载地址](http://www.iconfont.cn/plus/home/index?spm=a313x.7781069.1998910419.2.lEXKWX)
选择要下载的图片，下载时选择svg下载

[svg2android](http://inloop.github.io/svg2android/)
奖svg图片转换成android中的drawable文本
### 在android上显示svg图片

在drawable文件夹中新建文件，将之前转换后的文本信息复制到里面
```java
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="1024dp"
    android:height="1024dp"
    android:viewportHeight="1024"
    android:viewportWidth="1024">

    <path
        android:fillColor="#1296db"
        android:pathData="M693.403-9.263l-376.691 4.628c-93.005 1.142-141.47 64.931-141.47
121.987v760.555c0 70.909 48.497 147.571 146.098
145.113l367.435-9.257c88.235-2.223 160.61-37.038
160.001-135.856l-4.628-751.298c-0.55-89.394-34.658-137.298-150.745-135.872zM777.275
882.536c0 38.027-35.535 68.978-79.226 68.978h-372.081c-43.673
0-79.207-30.951-79.207-68.978v-755.926h0.018c0-38.043 35.535-68.978
79.207-68.978h372.063c43.673 0 79.226 30.937 79.226 68.978zM737.025
124.46h-450.014c-5.407 0-9.776 3.851-9.776 8.617v669.925c0 4.749 4.369 8.633
9.776 8.633h450.014c5.407 0 9.794-3.867
9.794-8.633v-669.925c0-4.766-4.387-8.617-9.794-8.617zM704.108
766.6l-379.551-4.628-4.628-592.51 379.551 4.628zM512.018 825.994c-32.969
0-62.476 23.634-62.476 52.691 0 29.058 33.092 58.667 72.035 51.497 32.423-5.97
56.768-24.541 56.768-53.583 0-29.058-33.359-50.606-66.327-50.606zM512.018
914.128c-22.191 0-40.231-15.901-40.231-35.443s18.041-35.459 40.231-35.459c22.173
0 50.361 14.874 50.361 34.416 0 19.542-28.188 36.486-50.361 36.486zM447.319
90.589c0-3.515 3.895-6.353 8.702-6.353h111.975c4.805 0 8.684 2.841 8.684 6.353 0
3.514-3.877 13.472-8.684 13.523l-111.975 1.195c-4.788
0.051-8.684-11.205-8.702-14.719z" />
</vector>
```
布局文件中的imageview直接调用该文件
```java
  <ImageView
        android:src="@drawable/phone_svg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
```
setColorFilter()可改变图片颜色
```java
        img_phone.setColorFilter(Color.RED);

```

## 兼容问题
### 不需要添加任何依赖包
Gradle在编译时会自动生成Vectordrawable对应的位图资源(如果你支持的最低sdk小于api21, 若大于等于21就不存在兼容性问题了)
缺点是同时打包了位图与矢量图资源APK包会变大. 生成哪种分辨率下的位图资源可以通过下面的Gradle配置指定:
```java
defaultConfig { 
      vectorDrawables.generatedDensities = ['hdpi','xxhdpi']
}
```
### 使用Support Library 23.2+(不会自动生成位图)
需要在build.gradle配置文件中增加如下配置:
```java
android { 
     defaultConfig { 
           vectorDrawables.useSupportLibrary = true  
      }
  }
```
然后在引用Vectordrawable资源时使用app:srcCompat取代android:src

## SVG动画
网上有个开源库[PathAnimView](https://github.com/mcxtzhang/PathAnimView)
```java
compile 'com.github.mcxtzhang:PathAnimView:V1.0.0'
```
以下为简要流程，详情可以去看作者的博客[极速get花式Path](http://blog.csdn.net/zxt0601/article/details/54018970)
1、获取png图片
2、利用vmde将图片转成svg格式[vmde下载地址](http://www.pc6.com/softview/SoftView_49725.html)
3、将svg图片进行转化[Android SVG to VectorDrawable](http://liuyouth.github.io/utils/svg2android/index.html)
4、将转化的结果写到strings中,如下
```java
 <string name="toys">M 3.33 3.52 C 7.15 1.59 12.31 4.86 12.95 9.01 C 15.02 20.49 16.86 32.02 18.84
43.51 C 19.10 45.60 20.73 47.74 23.00 47.70 C 33.34 47.83 43.69 47.73 54.04
47.73 C 55.56 47.81 57.80 47.37 58.39 49.31 C 57.98 50.42 57.23 50.97 56.14
50.98 C 45.42 51.04 34.69 50.98 23.97 51.01 C 20.36 51.33 16.61 48.88 15.90
45.24 C 13.66 33.35 11.97 21.34 9.68 9.46 C 9.18 6.31 5.59 6.39 3.20 5.74 C 3.23
5.18 3.30 4.07 3.33 3.52 Z</string>
```
5、布局文件中使用PathAnimView控件
```java
  <com.mcxtzhang.pathanimlib.PathAnimView
        android:id="@+id/pathAnimView1"
        android:layout_width="wrap_content"
        android:layout_height="60dp"
        android:layout_marginTop="260dp"
        android:background="@color/colorAccent"
        android:padding="5dp"/>
```
6、代码中加载动画
```java
        pathAnimView1 = (PathAnimView) findViewById(R.id.pathAnimView1);
        SvgPathParser svgPathParser = new SvgPathParser();
        try {
            Path path = svgPathParser.parsePath(getString(R.string.toys));
            pathAnimView1.setSourcePath(path);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        pathAnimView1.startAnim();
```



## 参考文章
[SVG 在Android中的应用](http://www.jianshu.com/p/12208e36dfb0)
[Android Vector曲折的兼容之路](http://www.jianshu.com/p/e3614e7abc03)
[android 中使用svg](http://www.jianshu.com/p/30dfa5920658)
[Android SVG技术入门：线条动画实现原理](http://www.jianshu.com/p/1263df796dfe)
