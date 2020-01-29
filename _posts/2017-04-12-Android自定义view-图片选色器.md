---
layout: post
title: Android自定义view-图片选色器. 
categories: Android
keywords: Android,自定义view
---
本文介绍该自定义view的使用及实现的方法，主要实现以下几个功能：
- 选取圆盘选色图片上的颜色，实时监听
- 可设置选色指示图片，跟随触摸位置、指示所选颜色，示例中为白色圆环
- 可自己设置选色图片（目前只支持圆形图片）

[github链接](https://github.com/autume/ColorPickerView)
## 使用效果
首先看下使用效果：
![](/images/posts/android/colorpicker.gif)

## 使用示例
### 在项目中导入该库
在工程的 build.gradle中加入：
```java
allprojects {
		repositories {
			...
			maven { url "https://jitpack.io" }
		}
	}
```
module的build.gradle中加入依赖：
```java
dependencies {
	       compile 'com.github.autume:ColorPickerView:1.0'
	}
```
### xml
```java
<RelativeLayout
        android:id="@+id/rl_picker"
        android:layout_below="@+id/img_color"
        android:layout_marginTop="30dp"
        android:layout_centerHorizontal="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

        <colorpickerview.oden.com.colorpicker.ColorPickerView
            android:id="@+id/color_picker"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>


        <ImageView
            android:id="@+id/img_picker"
            android:layout_centerInParent="true"
            android:src="@mipmap/color_picker"
            android:layout_width="25dp"
            android:layout_height="25dp" />

    </RelativeLayout>
```
### 选色代码
```java
   private void initRgbPicker() {
        colorPickerView = (ColorPickerView) findViewById(R.id.color_picker);
        colorPickerView.setImgPicker(MainActivity.this, img_picker, 25); //最后一个参数是该颜色指示圈的大小(dp)
        colorPickerView.setColorChangedListener(new ColorPickerView.onColorChangedListener() {
            @Override
            public void colorChanged(int red, int blue, int green) {
                img_color.setColorFilter(Color.argb(255, red, green, blue));
            }

            @Override
            public void stopColorChanged(int red, int blue, int green) {

            }
        });
    }
```
#### 对外公开的API
```java
 public void setImgPicker(final Context context, final ImageView imgPicker, final int pickerViewWidth)

 public void setImgResource(final int imgResource)

 public void setColorChangedListener(onColorChangedListener colorChangedListener)
```
 

## 实现过程
### attrs属性
可通过picture_resource属性设置用来选色的资源id,现仅支持圆形图片
```java
 <declare-styleable name="ColorPickerView">
        <attr name="picture_resource" format="reference"/>
    </declare-styleable>
```
### xml
布局中就是放入一个ImageView控件
```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/rl_root"
    tools:background="@color/black"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/img_color_rang"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:src="@mipmap/lights_colors" />


</RelativeLayout>
```
### 属性获取及view初始化
```java
 private void initAttrs(Context context, AttributeSet attrs) {
        if (null != attrs) {
            TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.ColorPickerView);
            imgResource = typedArray.getResourceId(R.styleable.ColorPickerView_picture_resource, 0);
            typedArray.recycle();
        }
    }

    private void initView(Context context) {
        View view = LayoutInflater.from(context).inflate(R.layout.color_picker, this);
        imgColorRang = (ImageView) view.findViewById(R.id.img_color_rang);
        rl_root = (RelativeLayout) view.findViewById(R.id.rl_root);

        if (imgResource != 0)
            imgColorRang.setImageResource(imgResource);

        bitmap = ((BitmapDrawable) imgColorRang.getDrawable()).getBitmap();//获取圆盘图片
    }
```

### 颜色回调监听
```java
    private onColorChangedListener colorChangedListener;//颜色变换监听

 public void setColorChangedListener(onColorChangedListener colorChangedListener) {
        this.colorChangedListener = colorChangedListener;
    }

    /**
     * 颜色变换监听接口
     */
    public interface onColorChangedListener {
        void colorChanged(int red, int blue, int green);
        void stopColorChanged(int red, int blue, int green);
    }
```

### 触摸事件
触摸事件写在父控件上，可以统一处理用来选色的view及指示选色位置的view(imgPicker)，imgPicker为指示显示位置的圆框，若设置了则跟随手指移动。
```java
 private void initTouchListener() {
        rl_root.setOnTouchListener(new OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {

                if (range_radius == 0) {
                    range_radius = imgColorRang.getWidth() / 2;  //圆盘半径
                    centreX = imgColorRang.getRight() - range_radius;
                    centreY = imgColorRang.getBottom() - imgColorRang.getHeight() / 2;
                    select_radius = range_radius - pickerViewPadding/5;
                }

                float xInView = event.getX();
                float yInView = event.getY();
                Log.d(TAG, "xInView: " + xInView + ",yInView: " + yInView + ",left: " + imgColorRang.getLeft() + ",top: " + imgColorRang.getTop() + ",right: " +imgColorRang.getRight() + ",bottom: " + imgColorRang.getBottom());

                //触摸点与圆盘圆心距离
                float diff = (float) Math.sqrt((centreY - yInView) * (centreY - yInView) + (centreX - xInView) *
                        (centreX - xInView));

                //在选色图片内则进行读取颜色等操作
                if (diff <= select_radius) {

                    //选色位置指示，若设置了则移动到点取的位置
                    if (imgPicker != null ) {
                        int xInWindow = (int) event.getX();
                        int yInWindow = (int) event.getY();
                        int left = xInWindow + v.getLeft() - imgPicker.getWidth() / 2;
                        int top = yInWindow + v.getTop() - imgPicker.getWidth() / 2;
                        int right = left + imgPicker.getWidth();
                        int bottom = top + imgPicker.getHeight();

                        imgPicker.layout(left, top, right, bottom);
                    }


                    if ((event.getY() - imgColorRang.getTop()) < 0)
                        return true;
                    //读取颜色
                    int pixel = bitmap.getPixel((int) (event.getX() - imgColorRang.getLeft()), (int) (event.getY() - imgColorRang.getTop()));   //获取选择像素
                    if (colorChangedListener != null) {
                        if (event.getAction() == MotionEvent.ACTION_UP) {
                            colorChangedListener.stopColorChanged(Color.red(pixel), Color.blue(pixel), Color.green(pixel));
                        }else {
                            colorChangedListener.colorChanged(Color.red(pixel), Color.blue(pixel), Color.green(pixel));
                        }
                    }
                    Log.d(TAG, "radValue=" + Color.red(pixel) + "  blueValue=" + Color.blue(pixel) + "  greenValue" + Color.green(pixel));
                }
                return true;
            }
        });
    }
```

### 设置指示图标
设置图标，同时根据图标的大小设置控件的padding避免在边界处显示不全的问题。
```java
  public void setImgPicker(final Context context, final ImageView imgPicker, final int pickerViewWidth) {
        this.imgPicker = imgPicker;
        pickerViewPadding = dip2px(context, pickerViewWidth/2);
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                rl_root.setPadding(pickerViewPadding, pickerViewPadding, pickerViewPadding, pickerViewPadding);
                bitmap = ((BitmapDrawable) imgColorRang.getDrawable()).getBitmap();//获取圆盘图片
            }
        },10);
    }
```

## 总结
ok，至此，一个比较简单的选色器就完成了。

