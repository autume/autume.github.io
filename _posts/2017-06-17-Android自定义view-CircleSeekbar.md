---
layout: post
title: Android自定义view-CircleSeekbar 
categories: Android
keywords: Android,自定义view
---
自定义view练手，效果图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200123232325526.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhb2RvbmczNzk=,size_16,color_FFFFFF,t_70)
## 实现功能
- 可设置圆环颜色和线宽及触摸后的颜色和线宽
- 可设置圆环内圈显示的文本内容及字体大小、颜色
- 可设置触摸点的图片
- 可设置触摸的有效范围

[源码github链接](https://github.com/autume/CircleSeekbar)
## 使用示例
```java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="circlebar.oden.com.circleseekbardemo.MainActivity">

    <circlebar.oden.com.circleseekbar.CircleSeekbar
        android:id="@+id/circle_seekbar"
        android:layout_centerInParent="true"
        android:layout_width="300dp"
        android:layout_height="300dp" />

    <TextView
        android:id="@+id/tv_progress"
        android:text="0%"
        android:textColor="#FF383838"
        android:textSize="18sp"
        android:layout_centerInParent="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />


</RelativeLayout>

```
```java
public class MainActivity extends AppCompatActivity {
    String TAG = "MainActivity";
    CircleSeekbar circleSeekbar;
    TextView tv_progress;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        circleSeekbar = (CircleSeekbar) findViewById(R.id.circle_seekbar);
        tv_progress = (TextView) findViewById(R.id.tv_progress);

        circleSeekbar.setOnSeekBarChangeListener(new CircleSeekbar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(CircleSeekbar circleSeekbar, int progress, boolean fromUser) {
                Log.d(TAG, "onProgressChanged progress: " + progress);
                tv_progress.setText(progress + "%");
            }
        });

    }
}

```
## 实现过程
### attrs中定义属性
可设置的属性如下
```java
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <declare-styleable name="CircleSeekbar">
        <attr name="circleColor" format="color" />
        <attr name="progressColor" format="color" />
        <attr name="textColor" format="color" />
        <attr name="textSize" format="dimension" />
        <attr name="progress" format="integer" />
        <attr name="maxProgress" format="integer" />
        <attr name="progressWidth" format="dimension" />
        <attr name="circleWidth" format="dimension" />
        <attr name="thumb" format="reference" />
        <attr name="enabled" format="boolean" />
        <attr name="touchInside" format="boolean" />
    </declare-styleable>

</resources>
```

### 获取属性及初始化画笔
```java
 private void init(Context context, AttributeSet attrs) {
        float density = context.getResources().getDisplayMetrics().density;
        int thumbHalfheight = 0;
        int thumbHalfWidth = 0;

        circleColor = getResources().getColor(R.color.circleseekbar_gray);
        progressColor = getResources().getColor(R.color.circleseekbar_blue_light);
        textColor = getResources().getColor(R.color.circleseekbar_text_color);
        progressWidth = (int) (progressWidth * density);
        circleWidth = (int) (circleWidth * density);
        textSize = (int) (textSize * density);

        padding = (int) (padding * density);
        mThumb = getResources().getDrawable(R.drawable.seekbar_control_selector);

        if (null != attrs) {
            TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleSeekbar);
            circleColor = typedArray.getColor(R.styleable.CircleSeekbar_circleColor, circleColor);
            progressColor = typedArray.getColor(R.styleable.CircleSeekbar_circleColor, progressColor);
            textColor = typedArray.getColor(R.styleable.CircleSeekbar_textColor, textColor);
            textSize = typedArray.getColor(R.styleable.CircleSeekbar_textSize, textSize);
            circleWidth = (int) typedArray.getDimension(R.styleable.CircleSeekbar_circleWidth, circleWidth);
            progressWidth = (int) typedArray.getDimension(R.styleable.CircleSeekbar_progressWidth, progressWidth);
            mProgress = typedArray.getInteger(R.styleable.CircleSeekbar_progress, mProgress);
            maxProgress = typedArray.getInteger(R.styleable.CircleSeekbar_maxProgress, maxProgress);
            mTouchInside = typedArray.getBoolean(R.styleable.CircleSeekbar_touchInside, mTouchInside);
            Drawable thumb = typedArray.getDrawable(R.styleable.CircleSeekbar_thumb);
            if (thumb != null) {
                mThumb = thumb;
            }

            thumbHalfheight = (int) mThumb.getIntrinsicHeight() / 2;
            thumbHalfWidth = (int) mThumb.getIntrinsicWidth() / 2;
            mThumb.setBounds(-thumbHalfWidth, -thumbHalfheight, thumbHalfWidth,
                    thumbHalfheight);
            typedArray.recycle();
        }

        padding = thumbHalfheight + padding;

        maxWidth = circleWidth > progressWidth ? circleWidth : progressWidth;
        mProgress = (mProgress > maxProgress) ? maxProgress : mProgress;
        mProgress = (mProgress < 0) ? 0 : mProgress;

        circlePaint = new Paint();
        circlePaint.setColor(circleColor);
        circlePaint.setAntiAlias(true);
        circlePaint.setStyle(Paint.Style.STROKE);
        circlePaint.setStrokeWidth(circleWidth);

        progressPaint = new Paint();
        progressPaint.setColor(progressColor);
        progressPaint.setAntiAlias(true);
        progressPaint.setStyle(Paint.Style.STROKE);
        progressPaint.setStrokeWidth(progressWidth);

        paintDegree = new Paint();
        paintDegree.setColor(textColor);
        paintDegree.setAntiAlias(true);
        paintDegree.setStyle(Paint.Style.FILL);
        paintDegree.setStrokeWidth(1);
        paintDegree.setTextSize(textSize);
    }
```
### 测量
获取圆的中心坐标、半径、以及用来画弧的RectF
```java
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        int height = getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec);
        int width = getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec);
        mWidth = Math.min(width, height);
        centreX = mWidth / 2;
        centreY = mWidth / 2;

        float left = getPaddingLeft() + maxWidth / 2 + padding;
        float top = getPaddingTop() + maxWidth / 2 + padding;
        float right = mWidth - getPaddingRight() - maxWidth / 2 - padding;
        float bottom = mWidth - getPaddingBottom() - maxWidth / 2 - padding;

        diameter = mWidth - getPaddingLeft() - getPaddingRight() - maxWidth - padding * 2;
        radius = diameter / 2;
        mCircleRect.set(left, top, left + diameter, top + diameter);

        updateThumbPosition(mCurAngle);

        setTouchInSide(mTouchInside);
        setMeasuredDimension(mWidth, mWidth);
    }
```

### 绘制view
```java
  @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawColor(Color.RED);
        canvas.drawCircle(mWidth / 2, mWidth / 2, radius, circlePaint);
        canvas.drawArc(mCircleRect, -90, (float) mCurAngle, false, progressPaint);
        drawText(canvas);
        canvas.translate(mThumbXPos, mThumbYPos);
        mThumb.draw(canvas);
    }

  private void drawText(Canvas canvas) {
        float rotation = 360 / degreeText.length;
        for (int i = 0; i < degreeText.length; i++) {
            canvas.drawText(degreeText[i], centreX - paintDegree.measureText(degreeText[i]) / 2, 100/*字体的高度*/, paintDegree);
            canvas.rotate(rotation, centreX, centreY);
        }
    }
```
### 设置触摸事件
```java
 @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (enable) {
            this.getParent().requestDisallowInterceptTouchEvent(true);//通知父控件勿拦截本次触摸事件

            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    updateOnTouch(event);
                    break;
                case MotionEvent.ACTION_MOVE:
                    updateOnTouch(event);
                    break;
                case MotionEvent.ACTION_UP:
                    isFirstTouch = true;
                    setPressed(false);
                    this.getParent().requestDisallowInterceptTouchEvent(false);
                    break;
                case MotionEvent.ACTION_CANCEL:
                    isFirstTouch = true;
                    setPressed(false);
                    this.getParent().requestDisallowInterceptTouchEvent(false);
                    break;
            }
            return true;
        }
        return false;
    }
```
触摸事件中计算相应的角度及进度条的值
```java
 private void updateOnTouch(MotionEvent event) {
        boolean ignoreTouch = ignoreTouch(event.getX(), event.getY());
        if (ignoreTouch) {
            return;
        }
        setPressed(true);
        getTouchDegrees(event.getX(), event.getY());
        mProgress = getProgressForAngle(mCurAngle);
        updateProgress(mProgress, true);
    }

    //根据触摸点的坐标获取角度值
    private double getTouchDegrees(float xPos, float yPos) {
        float x = xPos - centreX;
        float y = yPos - centreY;

        // convert to arc Angle
        double angle = Math.toDegrees(Math.atan2(y, x) + (Math.PI / 2));

        if (angle < 0) {
            angle = 360 + angle;
        }

        if (isOnlyScrollOneCircle) {
            if (mCurAngle > 270 && angle < 90) {
                mCurAngle = 360;
            } else if (mCurAngle < 90 && angle > 270) {
                mCurAngle = 0;
            } else {
                mCurAngle = angle;
            }
        } else {
            mCurAngle = angle;
        }
        if (isFirstTouch) {  //如果是滑动前第一次点击则总是移动到该度数
            mCurAngle = angle;
            isFirstTouch = false;
        }

        Log.d(TAG, "getTouchDegrees: " + angle);
        return mCurAngle;
    }

    //根据触摸的角度值获取滑动条progree的值
    private int getProgressForAngle(double angle) {
        int touchProgress = (int) Math.round(maxProgress * angle / 360);

        touchProgress = (touchProgress < 0) ? INVALID_PROGRESS_VALUE
                : touchProgress;
        touchProgress = (touchProgress > maxProgress) ? INVALID_PROGRESS_VALUE
                : touchProgress;
        return touchProgress;
    }

    //更新view
    private void updateProgress(int progress, boolean fromUser) {
        if (progress == INVALID_PROGRESS_VALUE) {
            return;
        }

        progress = (progress > maxProgress) ? maxProgress : progress;
        progress = (progress < 0) ? 0 : progress;
        mProgress = progress;

        if (onSeekBarChangeListener != null) {
            onSeekBarChangeListener.onProgressChanged(this, progress, fromUser);
        }

        updateThumbPosition(mCurAngle);
        invalidate();
    }

    //更新thum的坐标
    private void updateThumbPosition(double angle) {
        double cos = -Math.cos(Math.toRadians(angle));

        if (angle < 180) {
            mThumbXPos = (int) (getMeasuredWidth() / 2 + Math.sqrt(1 - cos * cos) * radius);
        } else {
            mThumbXPos = (int) (getMeasuredWidth() / 2 - Math.sqrt(1 - cos * cos) * radius);
        }

        mThumbYPos = (int) (centreX + radius * (float) cos);
    }
```
设置触摸的生效范围同时根据触摸状态同步改变thum的状态
```java
//设置触摸生效范围
    public void setTouchInSide(boolean isEnabled) {
        int thumbHalfheight = (int) mThumb.getIntrinsicHeight() / 2;
        int thumbHalfWidth = (int) mThumb.getIntrinsicWidth() / 2;
        mTouchInside = isEnabled;
        if (mTouchInside) {
            mTouchIgnoreRadius = (float) radius / 4;
        } else {
            // Don't use the exact radius makes interaction too tricky
            mTouchIgnoreRadius = radius - Math.min(thumbHalfWidth, thumbHalfheight);
        }
    }

    private boolean ignoreTouch(float xPos, float yPos) {
        boolean ignore = false;
        float x = xPos - centreX;
        float y = yPos - centreY;

        float touchRadius = (float) Math.sqrt(((x * x) + (y * y)));
        if (touchRadius < mTouchIgnoreRadius) {
            ignore = true;
        }
        return ignore;
    }

    //重写drawableStateChanged，同步改变thum的状态
    @Override
    protected void drawableStateChanged() {
        super.drawableStateChanged();
        if (mThumb != null && mThumb.isStateful()) {
            int[] state = getDrawableState();
            mThumb.setState(state);
        }
        invalidate();
    }
```
设置监听器
```java
public interface OnSeekBarChangeListener {
        void onProgressChanged(CircleSeekbar circleSeekbar, int progress, boolean fromUser);
    }

    public void setOnSeekBarChangeListener(OnSeekBarChangeListener onSeekBarChangeListener) {
        this.onSeekBarChangeListener = onSeekBarChangeListener;
    }
```




