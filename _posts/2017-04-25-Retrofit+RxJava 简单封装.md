---
layout: post
title: Retrofit+RxJava简单封装
categories: Android
keywords: Android,Retrofit,RxJava简单封装
---
本文对Retrofit+RxJava的使用进行简单封装，简化使用。
请求结果统一封装成HttpResult类，并利用泛型对不同结果统一处理。
[上一篇文章：Retrofit简要笔记](http://blog.csdn.net/yaodong379/article/details/70545811)

## 导入
导入依赖
```java  
    //导入retrofit，的版本号必须一样
    compile 'com.squareup.retrofit2:retrofit:2.1.0'
    compile 'com.squareup.retrofit2:converter-gson:2.1.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
    compile 'com.squareup.retrofit2:converter-scalars:2.1.0'
    //与rxjava配合使用需导入以下的库
    compile 'io.reactivex:rxandroid:1.1.0'
    compile 'io.reactivex:rxjava:1.1.0'
```
## Retrofit封装
首先看下业务请求接口：
```java
public interface BaseService {
    String BASE_URL = ApiUrl.URL_BASE;

    @FormUrlEncoded
    @POST("user/emailRegister")
    Observable<HttpResult> emaiRegister(@Field("email") String email,
                                        @Field("password") String password,
                                        @Field("name") String name,
                                        @Field("language") int language);

}
```
### retrofit封装
```java
public class NetManager {
    private static NetManager netManager = null;
    private static Retrofit retrofit = null;

    private NetManager() {
        retrofit = new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .baseUrl(ApiUrl.URL_BASE)
                .build();
    }

    public synchronized static NetManager getInstance() {
        if (netManager == null) {
            netManager = new NetManager();
        }
        return netManager;
    }

    public <S> S create(Class<S> service) {
        return retrofit.create(service);
    }

    /**
     * 返回原始的值
     * @param service
     * @param <S>
     * @return
     */
    public <S> S createString(Class<S> service) {
        Retrofit retrofit = new Retrofit.Builder()
                .addConverterFactory(ScalarsConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .baseUrl(getBaseUrl(service))
                .build();
        return retrofit.create(service);
    }

    /**
     * URL不一致时调用
     * @param service
     * @param <S>
     * @return
     */
    public <S> S createWithUrl(Class<S> service) {
        Retrofit retrofit = new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .baseUrl(getBaseUrl(service))
                .build();
        return retrofit.create(service);
    }

    /**
     * 解析接口中的BASE_URL，解决BASE_URL不一致的问题
     *
     * @param service
     * @param <S>
     * @return
     */
    private <S> String getBaseUrl(Class<S> service) {
        try {
            Field field = service.getField("BASE_URL");
            return (String) field.get(service);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "";
    }

}
```
封装成单例模式，使用的时候直接调用获取请求接口实例
```java
BaseService baseService = NetManager.getInstance().create(BaseService.class);
```
### 使用说明
1、addConverterFactory(GsonConverterFactory.create())表示返回的网络请求结果转换成json格式
2、服务器中有些数据并不是按照json格式传输，则调用createString方法，其中使用addConverterFactory(ScalarsConverterFactory.create())，表示返回原始的字符串数据
3、baseUrl改变的情况下，调用createWithUrl，则会传递进url，这个url定义在请求接口中，即BaseService中(String BASE_URL = ApiUrl.URL_BASE)
## RxJava封装
### Subscriber封装
对Subscriber进行一个简单的封装，对错误统一处理
```java
public abstract class RxSubscriber<T> extends Subscriber<T> {
    private MyLog myLog = new MyLog("[RxSubscriber] ", true);
    private Context context;

    public RxSubscriber() {
        context = MyApplication.getContext();
    }

    @Override
    public void onCompleted() {
        myLog.d("onCompleted");
    }

    @Override
    public void onError(Throwable e) {
        if (e instanceof IOException) {
            myLog.e("onError: " + e.toString() + ",网络链接异常");
        } else {
            myLog.d("onError: " + e.toString());
        }
        _onError(e);
    }

    @Override
    public void onNext(T t) {
        _onNext(t);
    }

    public Context getContext() {
        return context;
    }

    protected abstract void _onError(Throwable e);

    protected abstract void _onNext(T t);
}

```
### Subscription封装
```java
public class RxManager {
    private MyLog myLog = new MyLog("[RxManager] ");
    private static RxManager rxManager = null;

    private RxManager() {

    }

    public synchronized static RxManager getInstance() {
        if (rxManager == null) {
            rxManager = new RxManager();
        }
        return rxManager;
    }

    public Subscription doUnifySubscribe(Observable<HttpResult> observable, Subscriber<HttpResult> subscriber) {
        return observable
                .map(new Func1<HttpResult, HttpResult>() {
                    @Override
                    public HttpResult call(HttpResult httpResult) {
                        if (!httpResult.isStatus()) {
                            throw new ApiException(httpResult.getMsg());
                        }
                        return httpResult;
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subscriber);
    }

    public <T> Subscription doSubscribeT(Observable<HttpResultT<T>> observable, Subscriber<T> subscriber) {
        return observable
                .map(new Func1<HttpResultT<T>, T>() {
                    @Override
                    public T call(HttpResultT<T> httpResult) {
                        if (!httpResult.isStatus()) {
                            throw new ApiException(httpResult.getMsg());
                        }
                        return httpResult.getData();
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subscriber);
    }

    public <T> Subscription doSubscribe(Observable<T> observable, Subscriber<T> subscriber) {
        return observable
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subscriber);
    }

    public <T> Subscription doIoSubscribe(Observable<T> observable, Subscriber<T> subscriber) {
        return observable
                .subscribeOn(Schedulers.io())
                .observeOn(Schedulers.io())
                .subscribe(subscriber);
    }

    public <T> Observable<T> doSubscribe(Observable<T> observable) {
        return observable
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread());
    }

}
```
同样封装成单例，使用如下：
```java
BaseService baseService = NetManager.getInstance().create(BaseService.class);
RxManager.getInstance().doUnifySubscribe(baseService.emaiRegister(email, password, name, 0), new RxSubscriber<HttpResult>() {
            @Override
            protected void _onError(Throwable e) {
                MyToast.showShort(getActivity(), e.getMessage());
            }

            @Override
            protected void _onNext(HttpResult httpResult) {
                MyToast.showShort(getActivity(), getActivity().getString(R.string.sign_up_success));
                myLog.d("test s: " + httpResult);
                getActivity().finish();
            }
        });
```
### 使用说明
1、doUnifySubscribe方法为处理结果比较简单的请求，HttpResult类只返回了请求是否成功及相应信息，具体如下：
```java
public class HttpResult {
    boolean status;
    String msg;

    public boolean isStatus() {
        return status;
    }

    public void setStatus(boolean status) {
        this.status = status;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    @Override
    public String toString() {
        return "HttpResult [status=" + status + ", msg=" + msg + "]";
    }

}
```
2、doSubscribeT方法为处理结果比较多的请求，HttpResultT<T>类中，泛型放具体要处理的请求结果，具体如下：
```java
public class HttpResultT<T> {
    private boolean status;
    private String msg;
    private T data;

    public boolean isStatus() {
        return status;
    }

    public void setStatus(boolean status) {
        this.status = status;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    @Override
    public String toString() {
        return "HttpResultT [status=" + status + ", msg=" + msg + "]";
    }
}

```
例如，登陆请求结果的返回，定义如下：
```java
public class LoginResult {
    int status;
    long registerTime;
    long lastActivateTime;
    String uid;
    String name;
    String password;
    String account;
    String validateCode;
    String head_portrait_url;
    
    //...省略get、set方法

}
```
使用时
```java
BaseService baseService = NetManager.getInstance().create(BaseService.class);
RxManager.getInstance().doSubscribeT(baseService.login(account, Tools.getMd5Value(password)), new RxSubscriber<LoginResult>() {
    @Override
    protected void _onError(Throwable e) {
        MyToast.showShort(context, e.getMessage());
    }

    @Override
    protected void _onNext(LoginResult loginResult) {
        MyToast.showShort(context, context.getString(R.string.login_success));
        //存储请求结果
        myPrefs.account().put(account);
        myPrefs.password().put(password);
        ...
    }
});
```
3、其他一些方法则是定义在IO线程中返回，按具体需求可以删改
4、上面的请求结果的处理，需要和服务器统一协定返回数据的格式
## 总结
通过上述的封装，对大多数请求均可直接通过相对固定的方式进行简单的处理
```java
BaseService baseService = NetManager.getInstance().create(BaseService.class);
RxManager.getInstance().doSubscribeT(...);
```
