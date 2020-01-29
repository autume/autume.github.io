---
layout: post
title: Android Mvp实践
categories: Android
description: 本文是参考google官方发布的MVP架构demo以及前人对MVP实现方式的一些总结。
keywords: Android, MVP
---

本文是参考google官方发布的MVP架构demo以及前人对MVP实现方式的一些总结做的一个简单demo，在这里记录一下一点心得，希望能给想用MVP的人一点帮助。

![](/images/posts/android/mvp01.png)
## 总体框架
### 工程目录结构
首先先看下整个工程的目录结构：
![](/images/posts/android/mvp02.png)
目录的代码组织方式是按照功能来组织的，功能内部分为xactivity、xcontract、xfragment、xpresenter四个类文件(x代表业务名称)。base文件夹存放一些公用的基类文件，data文件夹存放业务逻辑相关的代码，utils文件夹则放一些公用的工具类。本demo实现的功能为：通过点击界面上的按钮，获取手机相关信息，获取过程中加入延时及等待提示（模拟网络），最终将信息显示于界面上（简单演示，只是显示了系统时间）。

### 基类
先看下BasePresenter与BaseView这两个接口类，它们分别是所有Presenter与View的基类。
```java
public interface BasePresenter {
    void start();
}
```
BasePresenter中含有方法start(),该方法的作用是presenter开始获取数据并调用view中方法改变界面显示，其调用时机是在Fragment类的onResume方法中。

```java
public interface BaseView<T> {
    void setPresenter(T presenter);
}

```
BaseView中含方法setPresenter，该方法作用是将presenter实例传入view中，其调用时机是在activity的presenter实现类的构造函数中。

### 契约类

```java
public interface GetPhoneInfoContract {

    interface View extends BaseView<Presenter> {

        void setTime(String time);

        void showLoading();

        void hideLoading();
    }

    interface Presenter extends BasePresenter {

        void getTime();
    }

}
```

与比较常见的mvp实现不同，官方的实现中加入了契约类来统一管理view与presenter的所有的接口，这种方式使得view与presenter中有哪些功能，一目了然，维护起来也方便，该实例中presenter的接口实现获取系统时间，view的接口实现时间的显示以及提示对话框的显示及隐藏。

## MVP的组织方式
### activity的作用
activity作为全局的控制者，负责创建view以及presenter实例，并将二者联系起来，具体的view交由fragment来实现，两者各司其职。

```java
@EActivity(R.layout.get_phone_info_act)
public class GetPhoneInfoActivity extends ActionBarActivity {

    private FragmentManager fm;
    private GetPhoneInfoFragment mGetPhoneInfoFragment = new GetPhoneInfoFragment_();
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDefaultFragment();
        new GetPhoneInfoPresenter(mGetPhoneInfoFragment);
    }

    private void setDefaultFragment() {
        fm = getFragmentManager();
        FragmentTransaction transaction = fm.beginTransaction();
        transaction.add(R.id.fragcontent, mGetPhoneInfoFragment);
        transaction.commit();
    }

}
```
本例中，activity中通过setDefaultFragment()设置了fragment，之后实例化GetPhoneInfoPresenter，并将frament传递进去，实现在presenter中通过fragment的接口对view进行操作展示。

### presenter的实现
```java
public class GetPhoneInfoPresenter implements GetPhoneInfoContract.Presenter{

    private final GetPhoneInfoContract.View mGetPhoneInfoView;
    private PhoneInfoBiz phoneInfoBiz;

    public GetPhoneInfoPresenter(GetPhoneInfoContract.View getPhoneInfoView) {
        mGetPhoneInfoView = getPhoneInfoView;
        mGetPhoneInfoView.setPresenter(this);
        phoneInfoBiz = new PhoneInfoBizIml();
    }

    @Override
    public void start() {
        getTime();
    }

    @Override
    public void getTime() {
        mGetPhoneInfoView.showLoading();
        phoneInfoBiz.getPhoneInfo(new PhoneInfoBiz.GetPhoneInfoCallback() {
            @Override
            public void onGetPhoneInfo(PhoneInfo phoneInfo) {
                mGetPhoneInfoView.setTime(phoneInfo.getTime());
                mGetPhoneInfoView.hideLoading();
            }
        });
    }
}

```
presenter构造函数中调用了view的setPresenter方法将自身实例传入，start方法中处理了数据加载与展示。如果需要界面做对应的变化，直接调用view层的方法即可，这样view层与presenter层就能够很好的被划分。

### view的实现
```java
@EFragment(R.layout.get_phone_info_frag)
public class GetPhoneInfoFragment extends Fragment implements GetPhoneInfoContract.View {

    private GetPhoneInfoContract.Presenter mPresenter;
    ProgressDialog dialog;

    @ViewById
    TextView tv_time;

    @ViewById
    Button btn_get_time;

    @Click
    void btn_get_time() {
        mPresenter.getTime();
    }

    @AfterViews
    void initView() {
        dialog = new ProgressDialog(getActivity());
    }

    @Override
    public void onResume() {
        super.onResume();
        mPresenter.start();
    }

    @Override
    public void setPresenter(GetPhoneInfoContract.Presenter presenter) {
        if (presenter != null)
            mPresenter = presenter;
    }

    @Override
    @UiThread
    public void setTime(String time) {
        tv_time.setText(time);
    }

    @Override
    public void showLoading() {
        dialog.setTitle("请稍候");
        dialog.setMessage("loading!");
        dialog.show();
    }

    @Override
    public void hideLoading() {
        dialog.dismiss();
    }

}
```
setPresenter方法继承于父类，通过该方法，view获得了presenter得实例，从而可以调用presenter代码来处理业务逻辑。在onResume中还调用了presenter得start方法，处理数据的加载与展示。

### 简单归纳
- activity创建fragment及实例化presenter，在实例化的同时，将fragment作为参数传递进去，这样一来在presenter中就可调用view的接口对界面进行更新、展示
- presenter实例化时，还调用了view的setPresenter方法，将自身传递进去，这样一来fragment获得了presenter的实例，便可在view中调用presenter进行业务逻辑的操作
- view及presenter拥有彼此的实例，实现了在view中调用presenter处理业务，处理完后再presenter中更新view。

## model层
简单介绍下model层，PhoneInfo对象存储手机相关信息，PhoneInfoBiz为借口类实现该业务所需要的接口及回调接口，PhoneInfoBizIml为接口的实现类。直接贴代码：

```java
public interface PhoneInfoBiz {

    interface GetPhoneInfoCallback {

        void onGetPhoneInfo(PhoneInfo phoneInfo);
    }

    void getPhoneInfo(GetPhoneInfoCallback getPhoneInfoCallback);
}

```

```java
public class PhoneInfoBizIml implements PhoneInfoBiz{

    @Override
    public void getPhoneInfo(final GetPhoneInfoCallback getPhoneInfoCallback) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    PhoneInfo phoneInfo = new PhoneInfo();
                    phoneInfo.setTime(System.currentTimeMillis() + "");
                    phoneInfo.setMobileType(Build.MODEL);
                    phoneInfo.setMobileVer(Build.VERSION.RELEASE);
                    Thread.sleep(1000);
                    getPhoneInfoCallback.onGetPhoneInfo(phoneInfo);
                }catch(Exception e){
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

```

## 总结
至此，一个简单的mvp框架到此结束，对于mvp的使用目前也还在探索中，上例是结合官方发布的demo做的一个简化工程，有不足之处欢迎一起探讨交流！

最后附上本文demo及官方demo的地址：

[本文demo链接](https://download.csdn.net/detail/yaodong379/9495175)

[官方demo链接](https://github.com/googlesamples/android-architecture/tree/todo-mvp/)


