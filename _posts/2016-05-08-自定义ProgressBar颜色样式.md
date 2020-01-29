---
layout: post
title: 自定义ProgressBar颜色样式
excerpt: 自定义ProgressBar颜色样式
categories: Android
keywords: Android, ProgressBar, 自定义view
---
## 使用
```java
  <ProgressBar
            android:id="@+id/pro_search"
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:layout_centerVertical="true"
            android:layout_toLeftOf="@+id/img_search"
            android:indeterminateDrawable="@drawable/circle_progressbar_style" />
```
在indeterminateDrawable中设置引用的样式

## xml样式文件
```java
<?xml version="1.0" encoding="utf-8"?>
<!-- 自定义圆形progressbar的颜色 -->
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:pivotX="50%"
    android:pivotY="50%"
    android:fromDegrees="0"
    android:toDegrees="1440">
    <shape android:shape="ring"
        android:innerRadiusRatio="3"
        android:thicknessRatio="12"
        android:useLevel="false"
        >

        <gradient android:type="sweep"
            android:useLevel="false"
            android:startColor="#0F7D27"
            android:endColor="#DFF3E3"
            android:centerY="0.5"
            android:centerColor="#00FF00"/>
    </shape>

</rotate>
```

改变toDegress的值，可以设置进度圆转动的速度
改变startColor、centerColor、endColor可改变圆圈的颜色
