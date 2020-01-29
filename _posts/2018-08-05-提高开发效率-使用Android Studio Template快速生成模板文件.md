---
layout: post
title: 提高开发效率-使用Android Studio Template快速生成模板文件
categories: Android
keywords: Android
---
Android Studio Template 依靠 FreeMarker 引擎，将事先定义好的模板文件生成我们所需的 class 文件、layout 文件等等，可以极大减少样板式代码的编写。

模板位置，Windows 的路径在 /plugins/android/lib/templates/，Mac 下是 Android Studio.app/Contents/plugins/android/lib/templates/，将自己写好的模板文件拷贝到此次即可在项目中使用，在file->new->activity中快速生成模板文件。
# 模板编写
主要有以下几个文件,变量名用${var}表示
## Activity.java.ftl中写activity相关代码
```
package ${packageName};

public class ${activityName} extends BaseActivity {

    @Bind(R.id.public_tv_title_text)
    TextView publicTvTitleText;
    @Bind(R.id.public_tv_title_save)
    TextView publicTvTitleSave;

    @Override
    public int getLayoutId() {
        return R.layout.${layoutName};
    }

    @Override
    public void init() {
        ButterKnife.bind(this);
        SystemBarTintUtil.INSTANCE.setSystenBarTint(mActivity, findView(R.id.${layoutName}_root), true);
    }

    @Override
    public void initEvent() {

    }

    @Override
    public void initData() {
        publicTvTitleText.setText("${title}");
        publicTvTitleSave.setText(" ");
        publicTvTitleSave.setVisibility(View.GONE);
        publicTvTitleSave.setTextColor(Color.parseColor("#FF333333"));
    }

    @Override
    protected void onResume() {
        super.onResume();
    }

    @OnClick({R.id.rl_public_iv_back, R.id.public_tv_title_save})
    public void onViewClicked(View view) {
        switch (view.getId()) {
            case R.id.rl_public_iv_back:
                finish();
                break;
            case R.id.public_tv_title_save:
                break;
        }
    }
}

```
## xml
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white">

    <RelativeLayout
        android:id="@+id/${layoutName}_root"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_alignParentStart="true"
        android:layout_alignParentTop="true"
        android:background="@color/white"
        android:orientation="vertical">

        <include
            android:id="@+id/public_title"
            layout="@layout/public_title_layout_save" />

    </RelativeLayout>

</RelativeLayout>
```
## manifest
```
<#import "../../common/shared_manifest_macros.ftl" as manifestMacros>

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="${packageName}">
    <application>
        <activity android:name="${packageName}.${activityName}"
                  android:screenOrientation="portrait"
                  android:theme="@android:style/Theme.Translucent.NoTitleBar" />
    </application>

</manifest>
```

## template

```
<?xml version="1.0"?>
<template
    format="5"
    revision="5"
    name="WhiteTopbarActivity"
    minApi="9"
    minBuildApi="14"
    description="Creates a new activity">

    <category value="Oden" />
    <formfactor value="Mobile" />

    <parameter
        id="activityName"
        name="Activity Name"
        type="string"
        constraints="class|unique|nonempty"
        suggest="${layoutToActivity(layoutName)}"
        default="OdenActivity"
        help="The name of the activity class to create" />

    <parameter id="layoutName"
        name="Activity Layout Name"
        type="string" constraints="layout|unique|nonempty"
        suggest="${activityToLayout(activityName)}"
        default="activity_main"
        help="The name of the layout to create for the activity" />

    <parameter id="title"
        name="Title Name"
        type="string" constraints="layout|unique|nonempty"
        suggest="title"
        default="title"
        help="The Title of the activity" />

    <parameter
        id="packageName"
        name="Package name"
        type="string"
        constraints="package"
        default="com.mycompany.myapp" />

    <!-- 128x128 thumbnails relative to template.xml -->
    <thumbs>
        <!-- default thumbnail is required -->
        <thumb>template_blank_activity.png</thumb>
    </thumbs>

    <globals file="globals.xml.ftl" />
    <execute file="recipe.xml.ftl" />

</template>

```
1. name属性为模板名称，description属性为模板的描述。
1. category标签，定义了模板所属的分类，，分类名一样的模板会被归纳到同一目录下。
1. parameter 标签，定义了模板输入弹窗中的输入参数，每个 parameter 为一行
    - id 属性为参数唯一标识，我们可以在代码中通过 id来使用该参数。
    - name 属性为参数名称，显示在输入控件的前面或后面。
    - type属性为参数类型，根据该属性和constraints属性的值综合比较后参数会被渲染成不同的输入形式，比如 string 类型会显示输入框，而 boolean类型会显示一个选择框。
    - constraints属性为输入约束，常见的有class，代表类名；layout代表布局名；package 代表包路径； unique则是不能与现有的重复；nonemptye表示不能为空。
    - suggest 和 default标签，前者是建议名称，后者是默认名称，前者优先级高于后者。
    - help属性是参数输入提示，当该参数获取焦点后，对应的帮助信息会显示在对话框上。
1. globals标签指定了 global 文件,global 标签定义了一系列的全局参数,供后续模板文件使用。
1. execute标签，跟字面上的意思一样，执行 recipe.xml.ftl文件的内容，将模板文件生成具体的可用文件。

## recipe.xml 
作用是定义输出规则
```
<?xml version="1.0"?>
<recipe>
    <merge from="root/AndroidManifest.xml.ftl"
       	to="${escapeXmlAttribute(manifestOut)}/AndroidManifest.xml" />

    <instantiate from="root/src/app_package/Activity.java.ftl"
                   to="${escapeXmlAttribute(srcOut)}/${activityName}.java" />

    <instantiate from="root/res/activity.xml.ftl"
          to="${escapeXmlAttribute(resOut)}/layout/${layoutName}.xml" />

    <open file="${escapeXmlAttribute(srcOut)}/${activityName}.java" />

    <open file="${escapeXmlAttribute(resOut)}/layout/${layoutName}.xml" />
</recipe>

```
- <#include>标签那行表示包含了 recipe_manifest.xml.ftl 文件的内容，里面是 <merge> 标签，作用是将定义的 manifest.xml.ftl 文件转化为 manifest.xml后与项目中的 AndroidManifest.xml 文件合并,完成了 Activity 在 AndroidManifest.xml 文件中注册的工作
- instantiate 标签是 recipe.xml.ftl 文件的核心标签，它的作用是将 from 属性的 ftl 文件转化为 to 属性的文件。
- open 文件会打开对应的文件
