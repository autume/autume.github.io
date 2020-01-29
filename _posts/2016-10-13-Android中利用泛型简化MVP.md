---
layout: post
title: Android中利用泛型简化MVP
categories: Android
keywords: Android
---
封装MvpFragment以及MvpPresenter，简化MVP的构建，达到偷懒的目的。
可以参考之前的另一篇文章:
[Android Mvp实践](http://blog.csdn.net/yaodong379/article/details/51184460) 

## 最终使用效果
Fragment和Presenter只需分别继承MvpFragmen、MvpPresenter即可进行绑定。
### Activity
Activity容器,里面放置了两个Fragment，在activity中将Fragment和其present绑定。
```java
@EActivity(R.layout.activity_main)
public class MainActivity extends AppCompatActivity {
    FragmentManager fm;
    private Fragment mFragmentNow;
    MapFragment mapFragment  = new MapFragment_();
    UserFragment userFragment = new UserFragment_();

    @Bean
    MapPresenter mapPresenter;

    @Bean
    UserPresenter userPresenter;

    @AfterViews
    void init() {
        setDefaultFragment();
        mapPresenter.setView(this, mapFragment); //view和presenter绑定
        userPresenter.setView(this, userFragment);//view和presenter绑定
    }

    private void setDefaultFragment() {
        fm = getFragmentManager();
        FragmentTransaction transaction = fm.beginTransaction();
        transaction.add(R.id.fragcontent, mapFragment);
        transaction.commit();
        mFragmentNow = mapFragment;
    }

    private void switchContent(Fragment from, Fragment to) {
        if (mFragmentNow != to) {
            mFragmentNow = to;
            FragmentTransaction transaction = fm.beginTransaction();
            if (!to.isAdded()) {    // 先判断是否被add过
                transaction.hide(from).add(R.id.fragcontent, to).commit(); // 隐藏当前的fragment，add下一个到Activity中
            } else {
                transaction.hide(from).show(to).commit(); // 隐藏当前的fragment，显示下一个
            }
        }
    }
//点击切换fragment略..
}
```
### Contract
Presenter和View的接口
```java
public class MapContract {

    public interface View extends BaseView<Presenter> {
     //...
    }

    public interface Presenter extends BasePresenter {
     //...
    }

}
```
### View层
```java
@EFragment(R.layout.fragment_map)
public class MapFragment extends MvpFragment<MapContract.Presenter> implements MapContract.View{
   //...实现接口中的方法
}
```
### Presenter层
```java
@EBean
public class MapPresenter extends MvpPresenter<MapContract.View> implements MapContract.Presenter {
   //...实现接口中的方法
}
```
这样，简单地就将Presenter层和View关联起来，在各自的层中处理不同的业务。

## 实现过程
### base类
```java
public interface BaseView<T> {
    void setPresenter(T presenter);
}
```

```java
public interface BasePresenter {
   
}
```

### MvpFragment
通过泛型传入Presenter，并覆写BaseView中的setPresenter
```java
public class MvpFragment <P extends BasePresenter> extends Fragment implements BaseView<P>{
    public P mPresenter;

    @Override
    public void setPresenter(BasePresenter presenter) {
        if (presenter != null)
            mPresenter = (P) presenter;
    }
}
```
### MvpPresenter
通过泛型传入BaseView,实现setView方法
```java
public class MvpPresenter <V extends BaseView> {
    public Context context;
    public V mView;

    public void setView(Context context, V mView) {
        this.context = context;
        this.mView = mView;
        mView.setPresenter(this);
    }

}
```
### 小结
通过以上方法，Activity中调用mapPresenter.setView(this, mapFragment)将view传入，而在setView中又调用setPresenter将view和Presenter绑定，于是可以在view层和presenter调用相关接口。



