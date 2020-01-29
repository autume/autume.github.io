---
layout: post
title: Android ViewPager实现引导页
categories: Android
keywords: Android
---
利用ViewPager简单地实现引导页功能，到达最后一页后再次滑动跳转到其他Activity

## xml布局
放入ViewPager控件，同时放入几个小圆点图片用来指示当前滑到第几页
```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        />

    <RelativeLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="21dp">

        <ImageView
            android:id="@+id/img_dot1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/select"
            android:background="#00000000"/>

        <ImageView
            android:id="@+id/img_dot2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:layout_toRightOf="@id/img_dot1"
            android:src="@drawable/no_select"
            android:background="#00000000"/>
        
        <ImageView
            android:id="@+id/img_dot3"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:layout_toRightOf="@id/img_dot2"
            android:src="@drawable/no_select"
            android:background="#00000000"/>

    </RelativeLayout>

</RelativeLayout>
```
## 新建ViewPager的adapter
主要就是传入view的list
```java
public class IntroductionsAdapter extends PagerAdapter {

    //界面列表
    private List<View> views;

    public IntroductionsAdapter (List<View> views){
        this.views = views;
    }

    //销毁arg1位置的界面
    @Override
    public void destroyItem(View arg0, int arg1, Object arg2) {
        ((ViewPager) arg0).removeView(views.get(arg1));
    }

    @Override
    public int getCount() {
        if (views != null)
        {
            return views.size();
        }
        return 0;
    }

    //初始化arg1位置的界面
    @Override
    public Object instantiateItem(View arg0, int arg1) {
        ((ViewPager) arg0).addView(views.get(arg1), 0);
        return views.get(arg1);
    }

    //判断是否由对象生成界面
    @Override
    public boolean isViewFromObject(View arg0, Object arg1) {
        return (arg0 == arg1);
    }

    @Override
    public Parcelable saveState() {
        // TODO Auto-generated method stub
        return null;
    }

}
```
## 使用viewpager
首先，初始化引导页图片，将其列表传入adapter，绑定到viewpager中。滑动的同时更新指示圆点的状态，同时加入滑到最后一页的判断。

```java
@NoTitle
@EActivity(R.layout.activity_introduction)
public class IntroductionsActivity extends Activity implements ViewPager.OnPageChangeListener {
    private IntroductionsAdapter vpAdapter;
    private List<View> views;
    //引导图片资源
    private static final int[] pics = { R.drawable.introduction01, R.drawable.introduction02, R.drawable.introduction03 };
    EdgeEffectCompat leftEdge;
    EdgeEffectCompat rightEdge;

    @ViewById
    ViewPager viewPager;

    @ViewById
    ImageView img_dot1;

    @ViewById
    ImageView img_dot2;

    @ViewById
    ImageView img_dot3;

    @AfterViews
    void init() {
        views = new ArrayList<View>();

        LinearLayout.LayoutParams mParams = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.FILL_PARENT,
                LinearLayout.LayoutParams.FILL_PARENT);

        //初始化引导图片列表
        for (int pic : pics) {
            ImageView iv = new ImageView(this);
            iv.setLayoutParams(mParams);
            iv.setImageResource(pic);
            views.add(iv);
        }
        //初始化Adapter
        vpAdapter = new IntroductionsAdapter(views);
        viewPager.setAdapter(vpAdapter);
        //绑定回调
        viewPager.setOnPageChangeListener(this);

        try {
            Field leftEdgeField = viewPager.getClass().getDeclaredField("mLeftEdge");
            Field rightEdgeField = viewPager.getClass().getDeclaredField("mRightEdge");
            if (leftEdgeField != null && rightEdgeField != null) {
                leftEdgeField.setAccessible(true);
                rightEdgeField.setAccessible(true);
                leftEdge = (EdgeEffectCompat) leftEdgeField.get(viewPager);
                rightEdge = (EdgeEffectCompat) rightEdgeField.get(viewPager);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

    }

    @Override
    public void onPageSelected(int position) {
        setCurDot(position);
    }

    @Override
    public void onPageScrollStateChanged(int state) {
        if(rightEdge!=null&&!rightEdge.isFinished()){//到了最后一张并且还继续拖动则跳转到其他activity
            final Intent intent = new Intent(this, OtherActivity.class);
            startActivity(intent);
            finish();
        }
    }

    private void setCurDot(int position) {
        resetDot();
        switch (position) {
            case 0:
                img_dot1.setImageResource(R.drawable.select);
                break;
            case 1:
                img_dot2.setImageResource(R.drawable.select);
                break;
            case 2:
                img_dot3.setImageResource(R.drawable.select);
                break;
        }
    }

    private void resetDot() {
        img_dot1.setImageResource(R.drawable.no_select);
        img_dot2.setImageResource(R.drawable.no_select);
        img_dot3.setImageResource(R.drawable.no_select);
    }

}
```
