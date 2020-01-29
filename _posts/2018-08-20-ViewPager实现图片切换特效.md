---
layout: post
title: ViewPager实现图片切换特效
categories: Android
keywords: Android,ViewPager
---
实现要点
- ViewPager,显示左右两边，并留出一定间距，整个viewpger响应触摸事件
- 利用PageTransformer给viewpager添加切换动画，透明度及图片大小过渡变化的效果

实现如下效果
![](/images/posts/android/vpg.gif)

## 实现
### ViewPager显示左右两边
利用View的android:clipChildren属性
clipChildren:父View是否束缚子View的显示范围,如果父View有 padding, 那么子View则在padding区域是不能显示内容的, 
但是如果设置android:clipChildren为false时,则子View就可以在 父View的padding区域显示内容
布局如下
```
 <RelativeLayout
    android:id="@+id/rl_viewPager_box"
    android:clipChildren="false"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager_photo"
        android:layout_marginLeft="50dp"
        android:layout_marginRight="134dp"
        android:clipChildren="false"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
 </RelativeLayout>
```
外层的RelativeLayout及ViewPager同时设置android:clipChildren 为false
在ViewPager中的前一页及后一页的内容即可在layout_marginLeft及
layout_marginRight的范围内显示

### viewpager页面间设置间距
```
viewPagerPhoto.setPageMargin(ScreenUtil.dip2px(12));
```
传入的参数单位为px
### 设置触摸响应区域
以上方法实现后，只有中间的item可以触发左右滑动效果，使整个viewpager均可左右滑动需做以下设置
```
rlViewPagerBox.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        return viewPagerPhoto.dispatchTouchEvent(event);
    }
});
```
### 设置adapter
```
public class PhotoViewPagerAdapter extends PagerAdapter {
    private Context mContext;
    private List<String> mImages;
    private OnItemClickListener onItemClickListener;

    public PhotoViewPagerAdapter(Context context, List<String> mImages) {
        this.mContext = context;
        this.mImages = mImages;
    }

    public void setData(List<String> mImages) {
        this.mImages = mImages;
        notifyDataSetChanged();
    }

    @Override
    public int getCount() {
        return this.mImages.size();
    }

    @Override
    public boolean isViewFromObject(View arg0, Object arg1) {
        return arg0 == arg1;
    }

    @Override
    public void destroyItem(View container, int position, Object object) {
        ((ViewPager) container).removeView((View) object);
    }

    @Override
    public Object instantiateItem(View container, int position) {
        ImageView image = new ImageView(mContext);
        GlideUtils.INSTANCE.setRoundImageByUrl(mContext, image, mImages.get(position), R.drawable.loading_image_default, R.drawable.loading_image_default);
        image.setScaleType(ImageView.ScaleType.CENTER_CROP);
        image.setCropToPadding(true);
        image.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (onItemClickListener != null) {
                    onItemClickListener.onItemClick();
                }
            }
        });
        ((ViewPager) container).addView(image);
        return image;
    }

    public interface OnItemClickListener {
        void onItemClick();
    }

    public void setOnItemClickListener(OnItemClickListener onItemClickListener) {
        this.onItemClickListener = onItemClickListener;
    }
}
```
### viewpager切换增加动画效果
#### 实现PhotoPageTransformer implements ViewPager.PageTransformer
```
public class PhotoPageTransformer implements ViewPager.PageTransformer {
    private static final float MIN_SCALE = 0.9f;

    private float mMinAlpha = 0.3f;
    private float leftScaleY = 0.9f;
    private float rightScaleY = 0.5f;
    private float leftTransY = 0.05f; //(1-0.9)/2
    private float rightTransY = 0.25f;

    public void transformPage(View view, float position) {
        int pageHeight = view.getHeight();
        LogUtils.e("transformPage position: " + view + "," + position);
        if (position < -1) { // [-Infinity,-1)  This page is way off-screen to the left.
            view.setAlpha(mMinAlpha);
            view.setScaleY(leftScaleY);
            view.setTranslationY(pageHeight * leftTransY);
        } else if (position <= 1) {
            if (position <= 0) { //[-1，0) 左滑  a页滑动至b页：a页从 0.0~-1，b页从1 ~ 0.0
                //1到mMinAlpha
                float factor = mMinAlpha + (1 - mMinAlpha) * (1 + position);
                view.setAlpha(factor);
                view.setScaleY(leftScaleY + (1 - leftScaleY) * (1 + position));
                //0到leftTransY的变化
                view.setTranslationY(-pageHeight * (leftTransY * position));
            } else //[0,1]  1~0
            {
                //minAlpha到1的变化
                float factor = mMinAlpha + (1 - mMinAlpha) * (1 - position);
                view.setAlpha(factor);
                view.setScaleY(rightScaleY + (1 - rightScaleY) * (1 - position));
                //rightTransY到0的变化
                view.setTranslationY(pageHeight * (rightTransY * position));
            }
        } else {  // (1,+Infinity]
            // This page is way off-screen to the right.
            view.setAlpha(mMinAlpha);
            view.setScaleY(rightScaleY);
            view.setTranslationY(pageHeight * rightTransY);
        }
    }
}
```
postion：

- [-Infinity,-1)：左边的页面
- (1,+Infinity]：右边的页面
- [-1,1]：中间显示的页面

对于左边及右边的页面，直接设置页面的最终效果的状态值即可，这里设置了透明度及Y轴缩放，因为Y轴缩放是以中心进行缩放，因此对其进行了向下的平移操作使其底部对齐。

假设a为当前中间显示页面，b为右边下一个页面
a页滑动至b页即左滑，positon的变化范围：a页从0.0~-1，b页从1 ~ 0.0。

- a到左边，对应alpha应该是：1到minAlpha，setScaleY：1到leftScaleY，TranslationY：0到leftTransY
- b到中间，对应alpha应该是：minAlpha到1，setScaleY：rightScaleY到1，TranslationY：rightTransY到0

对于a页，计算1到minAlpha过程：

- position：0~-1
- 1+position：1~0
- (1-mMinAlpha)*(1+position)：(1 - mMinAlpha)~0
- mMinAlpha+(1-mMinAlpha)*(1 + position)：1~minAlpha
其他以此类推

#### viewPager.setPageTransformer
```
photoViewPagerAdapter = new PhotoViewPagerAdapter(this, mImageViews);
viewPagerPhoto.setPageTransformer(true, new PhotoPageTransformer());
```
## 总结
ok,至此便完成整个切换效果
## 参考文章
[Android 实现个性的ViewPager切换动画](https://blog.csdn.net/lmj623565791/article/details/40411921)
