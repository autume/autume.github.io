---
layout: post
title: unity3d与android交互
categories: Android
keywords: Android，unity3d
---
实现unity3d导出到android studio工程并封装为library，供其他工程导入module直接使用。
最终效果：
- 点击ZoomIn、ZoomOut的按钮，通过android端调用unity中方法，进行放大放小；
- 触摸unity中的3D立方体，调用android端的ShowDialog方法显示原生的dialog。

最近看了下unity3d，关于unity3d和android端的交互参考了一些资料进行测试，现简单总结记录，同时以供参考。

软件版本如下：
unity3d：5.5.0f3
android studio：2.2
## 具体操作步骤
### 1、在场景中添加一个简单的立方体Cube，加入以下测试用的脚本
```java
using UnityEngine;
using System.Collections;

public class CubeScripts : MonoBehaviour {

	/// 定义旋转速度
	public float RotateSpeed=45;

	/// 定义摄像机的最近距离	
	private float mNear=2.5F;
    
	/// 摄像机当前距离
	private float mDistance=5F;

	/// 定义摄像机的最远距离
	private float mFar=7.5F;

	/// 摄像机的缩放速率
	private float mZoomRate=0.5F;

	/// 主摄像机
	private Transform mCamera;
	
	/// 在Start()方法中我们设定了游戏体的名称，因为我们在
	/// Android项目中需要用到这个名称,同时获取主相机对象
	void Start () 
	{
		this.name="Main Cube";
		mCamera=Camera.main.transform;
	}
	
	/// 在Update()方法中我们让Cube按照一定的速度进行旋转
	void Update () 
	{
		transform.Rotate(Vector3.up * Time.deltaTime * RotateSpeed);
	}

	/// 定义一个放大的方法供外部调用
	public void ZoomIn()
	{
		mDistance-=mZoomRate;
		mDistance=Mathf.Clamp(mDistance,mNear,mFar);
		mCamera.position=mCamera.rotation * new Vector3(0,0,-mDistance)+transform.position;
	}

	/// 定义一个缩小的方法供外部调用
	public void ZoomOut()
	{
		mDistance+=mZoomRate;
		mDistance=Mathf.Clamp(mDistance,mNear,mFar);
		mCamera.position=mCamera.rotation * new Vector3(0,0,-mDistance)+transform.position;
	}

	/// 触摸立方体，调用android端的ShowDialog方法
    void OnMouseDown()
    { 
        ZoomOut();
        Debug.Log("MOUSE DOWN");

        using (AndroidJavaClass jc = new AndroidJavaClass("com.oden.u2as.UnityPlayerActivity"))
        {
           // jc.CallStatic("ShowDialog");
            jc.CallStatic("ShowDialog", "str");
        }

    }

}
```
### 2、导出android studio工程
注意修改包名
导出后工程加入以下方法：
为unity工程中预先设置好的调用方法
```java
public static void ShowDialog(final String string) {
		UnityPlayer.currentActivity.runOnUiThread(new Runnable() {
			@Override
			public void run() {
				Log.d("SYD", "ShowDialog： " + string);
				AlertDialog.Builder builder = new AlertDialog.Builder(UnityPlayer.currentActivity)
						.setMessage("哈哈哈这是Android的原生弹窗")
						.setPositiveButton("确定", new DialogInterface.OnClickListener() {
							@Override
							public void onClick(DialogInterface dialog, int which) {
								UnityPlayer.UnitySendMessage("Camera", "NativeTipClosed", "");
							}
						});
				builder.show();
			}
		});
	}
```

### 3、将导出的工程设置为library
apply plugin: 'com.android.library'
注：遇到报错解决记录：
- 在manifest下添加  xmlns:tools="http://schemas.android.com/tools" 
- 在application下添加tools:replace="android:icon, android:theme"

### 4、其他工程import该module

### 5、其他工程中需要用到该3D效果的activity继承UnityPlayerNativeActivity
```java
public class MainActivity extends UnityPlayerNativeActivity {
    private Button BtnZoomIn, BtnZoomOut;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //获取显示Unity视图的父控件
        LinearLayout mParent = (LinearLayout) findViewById(R.id.UnityView);
        //获取Unity视图
        View mView = mUnityPlayer.getView();
        //将Unity视图添加到Android视图中
        mParent.addView(mView);

        //放大
        BtnZoomIn = (Button) findViewById(R.id.BtnZoomIn);
        BtnZoomIn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View arg0) {
                UnityPlayer.UnitySendMessage("Main Cube", "ZoomIn", "");
            }
        });
        //缩小
        BtnZoomOut = (Button) findViewById(R.id.BtnZoomOut);
        BtnZoomOut.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View arg0) {
                UnityPlayer.UnitySendMessage("Main Cube", "ZoomOut", "");
            }
        });
    }

}

```


## 参考文章：
[[Unity3D]Unity3D游戏开发之从Unity3D到Eclipse](http://blog.csdn.net/qinyuanpei/article/details/39717795)

[如何进行Unity3D与Android消息传递](https://www.zhihu.com/question/35041097)

[与iOS、Android的交互 实践篇——主动调用](http://www.jianshu.com/p/83c5736007f6)

