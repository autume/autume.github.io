---
layout: post
title: Android自定义view-继承LinearLayout
excerpt: Android自定义view-继承LinearLayout
categories: Android
keywords: Android, 自定义view
---
## 直接在代码中通过代码动态生成
```
public class MyView extends LinearLayout {

    private Button button;

    public MyView(final Context context) {
        super(context);
        button = new Button(context);
        /**
         * 方式1：
         */
//        LinearLayout.LayoutParams layoutParams = new LinearLayout.LayoutParams(300,100);
//        button.setLayoutParams(layoutParams);
        /**
         * 方式2：
         */
        addView(button);
        LinearLayout.LayoutParams layoutParams = (LayoutParams) button.getLayoutParams();
        layoutParams.weight = 300;
        layoutParams.height = 100;
        button.setLayoutParams(layoutParams);
        button.setText("我是一个按钮");
        button.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(context,"点击了",Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

### LayoutParams
描述了宽高，宽和高都可以设置成三种值：
- 一个确定的值
- FILL_PARENT，即填满(和父容器一样大小)
- WRAP_CONTENT，即包裹住组件就好

```
setLayoutParams(new LayoutParams(LayoutParams.FILL_PARENT, LayoutParams.FILL_PARENT));
```
```
LinearLayout.LayoutParams layoutParams = new LinearLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
layoutParams.gravity = Gravity.CENTER_HORIZONTAL; 
View.setLayoutParams(layoutParams);
```
移除其包含的某个子元素，只需调用removeView(View view)方法即可

## 通过使用映射机制加载布局文件
```
    public class CustomLayout extends LinearLayout{  
      
           public  CustomLayout(Context context){  
                     LayoutInflater mInflater = LayoutInflater.from(context);  
                    View myView = mInflater.inflate(R.layout.receive, null);  
                    addView(myView);  
      
           }  
      
    }  
```
```
< LinearLayout  
  xmlns:android="http://schemas.android.com/apk/res/android"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  androidundefinedrientation="vertical" >  
  <LinearLayout   
          android:layout_width="wrap_content"  
          android:layout_height="wrap_content"  
          androidundefinedrientation="horizontal">  
          <TextView  
                   android:layout_width="wrap_content"  
                  android:layout_height="wrap_content" />  
         <Button  
                 android:layout_width="wrap_content"  
                  android:layout_height="wrap_content"   
                  android:id="@+id/button" />  
  </LinearLayout>  
    
           <TextView  
                   android:layout_width="wrap_content"  
                  android:layout_height="wrap_content" />          
                     
< /LinearLayout> 
```
