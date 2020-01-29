---
layout: post
title: RxAndroid基础
excerpt: RxAndroid基础
categories: Android
keywords: Android,RxAndroid
---
参考[hi大头鬼hi ](http://blog.csdn.net/lzyzsd/article/details/41833541)的微博，写代码进行测试学习，以下记录共享，同时以便之后查阅。 由于不熟悉lambda，同时开始学习也不建议直接使用lambda，以下大部分代码均使用常规方法编写。

[RxJava在github上的地址](https://github.com/ReactiveX/RxJava)
[RxAndroid在github上的地址](https://github.com/ReactiveX/RxAndroid)

首先，工程中引入：
```java
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.0.1'
  
    //引入RxAndroid----begin
    compile 'io.reactivex:rxandroid:1.1.0'
    compile 'io.reactivex:rxjava:1.1.0'
    //引入RxAndroid----end

    // RxBinding
    compile 'com.jakewharton.rxbinding:rxbinding:0.4.0'
    compile 'com.jakewharton.rxbinding:rxbinding-appcompat-v7:0.4.0'
    compile 'com.jakewharton.rxbinding:rxbinding-design:0.4.0'
    }
```
    
## 一、基本使用 ##
### 常规语法 ###
```java
Observable<String> myObservable = Observable.create(
                new Observable.OnSubscribe<String>() {
                    @Override
                    public void call(Subscriber<? super String> sub) {
                        sub.onNext("Observable Hello, world!");
                        sub.onCompleted();
                    }
                }
        );
        
        Subscriber<String> mySubscriber = new Subscriber<String>() {
            @Override
            public void onNext(String s) {
                L.d("onNext s: " + s);
               }

            @Override
            public void onCompleted() { }

            @Override
            public void onError(Throwable e) { }
        };

        myObservable.subscribe(mySubscriber);
```
    
### 简化语法 ###
```java
Observable.just(1)
                .subscribe(new Action1<Integer>() {
                    @Override
                    public void call(Integer integer) {
                        L.d("onNext integer: " + integer);
                    }
                });
```
    
### map操作符变换 ###
```java
 Observable.just("Hello, world!")
                .map(new Func1<String, String>() {
                    @Override
                    public String call(String s) {
                        return s + " -Map";
                    }
                })
                .subscribe(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        L.d("onNext s: " + s);
                    }
                });
```
   
## 二、操作符 ##
### Observable.from() ###
Observable.from()方法，它接收一个集合作为输入，然后每次输出一个元素给subscriber：
```java
List<String> urls = new ArrayList<>();
        urls.add("url1");
        urls.add("url2");
        urls.add("url3");
        Observable.from(urls)
                .subscribe(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        L.d("onNext s: " + s);
                    }
                });
```
     
输出如下：
nNext s: url1
onNext s: url2
onNext s: url3
```java
Observable<List<String>> myObservableUrls = Observable.create(
                new Observable.OnSubscribe<List<String>>() {
                    @Override
                    public void call(Subscriber<? super List<String>> subscriber) {
                        List<String> urls = new ArrayList<>();
                        urls.add("url1");
                        urls.add("url2");
                        urls.add("url3");
                        subscriber.onNext(urls);
                    }
                }
        );

        myObservableUrls.subscribe(new Action1<List<String>>() {
            @Override
            public void call(List<String> strings) {
                Observable.from(strings)
                        .subscribe(new Action1<String>() {
                            @Override
                            public void call(String s) {
                                L.d("myObservableUrls onNext s: " + s);
                            }
                        });
            }
        });
```
       

输出如下：
myObservableUrls onNext s: url1
myObservableUrls onNext s: url2
myObservableUrls onNext s: url3
在前一个Observable输出是list时，嵌套太多，看起来比较乱。

### Observable.flatMap() ###
Observable.flatMap()接收一个Observable的输出作为输入，同时输出另外一个Observable。
```java
myObservableUrls.flatMap(new Func1<List<String>, Observable<String>>() {
            @Override
            public Observable<String> call(List<String> urls) {
                return Observable.from(urls);
            }
        }).subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                L.d("myObservableUrls.flatMap onNext s: " + s);
            }
        }); 
```   

避免了上面直接使用Observable.from时需要内如再次subscribe导致的内部嵌套。理解flatMap的关键点在于，flatMap输出的新的Observable正是我们在Subscriber想要接收的。现在Subscriber不再收到List<String>，而是收到一些列单个的字符串，就像Observable.from()的输出一样。

flatMap()，它可以返回任何它想返回的Observable对象。
接着前面的例子，现在我不想打印URL了，而是要打印收到的每个网站的标题。

现在的方法每次只能传入一个URL，并且返回值不是一个String，而是一个输出String的Observabl对象：
```java
private Observable<String> getTitle(final String url) {
        return Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("title: " + url);
            }
        });
    }

```
      
实现：
```java
myObservableUrls.flatMap(new Func1<List<String>, Observable<String>>() {
            @Override
            public Observable<String> call(List<String> urls) {
                return Observable.from(urls);
            }
        }).flatMap(new Func1<String, Observable<?>>() {
            @Override
            public Observable<?> call(String s) {
                return getTitle(s);
            }
        }).subscribe(new Action1<Object>() {
            @Override
            public void call(Object o) {
                L.d("myObservableUrls.flatMap onNext o: " + o);
            }
        });
``` 
### filter() ###
filter()输出和输入相同的元素，并且会过滤掉那些不满足检查条件的
```java
myObservableUrls.flatMap(new Func1<List<String>, Observable<String>>() {
            @Override
            public Observable<String> call(List<String> urls) {
                return Observable.from(urls);
            }
        }).flatMap(new Func1<String, Observable<?>>() {
            @Override
            public Observable<?> call(String s) {
                return getTitle(s);
            }
        }).filter(new Func1<Object, Boolean>() {
            @Override
            public Boolean call(Object o) {
                return !o.equals("title: url2");
            }
        }).subscribe(new Action1<Object>() {
            @Override
            public void call(Object o) {
                L.d("myObservableUrls.flatMap onNext o: " + o);
            }
        });
```
     
### take() ###
take()输出最多指定数量的结果。

### doOnNext() ###
doOnNext()允许我们在每次输出一个元素之前做一些额外的事情，比如这里的保存标题。

直接上网上的示例:
```java
query("Hello, world!")  
    .flatMap(urls -> Observable.from(urls))  
    .flatMap(url -> getTitle(url))  
    .filter(title -> title != null)  
    .take(5)  
    .doOnNext(title -> saveTitle(title))  
    .subscribe(title -> System.out.println(title));  
```  

## 三、响应式的好处 ##

### 错误处理 ###
每一个Observerable对象在终结的时候都会调用onCompleted()或者onError()方法，

这种模式有以下几个优点：

1.只要有异常发生onError()一定会被调用

这极大的简化了错误处理。只需要在一个地方处理错误即可以。

2.操作符不需要处理异常

将异常处理交给订阅者来做，Observerable的操作符调用链中一旦有一个抛出了异常，就会直接执行onError()方法。

3.你能够知道什么时候订阅者已经接收了全部的数据。

知道什么时候任务结束能够帮助简化代码的流程。（虽然有可能Observable对象永远不会结束）
   
**使用RXJAVA，OBSERVABLE对象根本不需要知道如何处理错误！操作符也不需要处理错误状态-一旦发生错误，就会跳过当前和后续的操作符。所有的错误处理都交给订阅者来做。**
    
### 调度器 ###
使用RxJava，你可以使用subscribeOn()指定观察者代码运行的线程，使用observerOn()指定订阅者运行的线程：
```java
 myObservableServices.retrieveImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));
```

### 订阅（Subscriptions） ###
调用Observable.subscribe()，会返回一个Subscription对象。这个对象代表了被观察者和订阅者之间的联系。
```java
 ubscription subscription = Observable.just("Hello, World!")
    .subscribe(s -> System.out.println(s));
```
   
你可以在后面使用这个Subscription对象来操作被观察者和订阅者之间的联系.
```java
subscription.unsubscribe();
    System.out.println("Unsubscribed=" + subscription.isUnsubscribed());
```
RxJava的另外一个好处就是它处理unsubscribing的时候，会停止整个调用链。如果你使用了一串很复杂的操作符，调用unsubscribe将会在他当前执行的地方终止。不需要做任何额外的工作！

当然也不是所有的代码都使用响应式的方式，仅仅当代码复杂到我想将它分解成简单的逻辑的时候，我才使用响应式代码。

## 四、在Android中使用响应式编程 ##
这部分主要是摘录，有些方法现在已过时，后续在实际应用中有用到再进行补充更新，

### RxAndroid ###

首先，AndroidSchedulers提供了针对Android的线程系统的调度器。需要在UI线程中运行某些代码？很简单，只需要使用AndroidSchedulers.mainThread():
```java
   retrofitService.getImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));

```

throttleFirst(): 在每次事件触发后的一定时间间隔内丢弃新的事件。常用作去抖动过滤，例如按钮的点击监听器：
```java
 RxView.clickEvents(button) // RxBinding 代码
    .throttleFirst(500, TimeUnit.MILLISECONDS) // 设置防抖间隔为 500ms
    .subscribe(subscriber);
```
 
### 遗留代码，运行极慢的代码 ###
绝大多数时候Observable.just() 和 Observable.from() 能够帮助你从遗留代码中创建 Observable 对象:
```java
   private Object oldMethod() { ... }

	public Observable<Object> newMethod() {
	    return Observable.just(oldMethod());
	}
```

上面的例子中如果oldMethod()足够快是没有什么问题的，但是如果很慢呢？调用oldMethod()将会阻塞住他所在的线程。 
为了解决这个问题，可以参考我一直使用的方法–使用defer()来包装缓慢的代码：
```java
 private Object slowBlockingMethod() { ... }

	public Observable<Object> newMethod() {
	    return Observable.defer(() -> Observable.just(slowBlockingMethod()));
	}

```  
现在，newMethod()的调用不会阻塞了，除非你订阅返回的observable对象。


### 生命周期 ###
如何处理Activity的生命周期？主要就是两个问题： 
1.在configuration改变（比如转屏）之后继续之前的Subscription。
比如你使用Retrofit发出了一个REST请求，接着想在listview中展示结果。如果在网络请求的时候用户旋转了屏幕怎么办？你当然想继续刚才的请求，但是怎么搞？

2.Observable持有Context导致的内存泄露
这个问题是因为创建subscription的时候，以某种方式持有了context的引用，尤其是当你和view交互的时候，这太容易发生！如果Observable没有及时结束，内存占用就会越来越大。

第一个问题的解决方案就是使用RxJava内置的缓存机制，这样你就可以对同一个Observable对象执行unsubscribe/resubscribe，却不用重复运行得到Observable的代码。cache() (或者 replay())会继续执行网络请求（甚至你调用了unsubscribe也不会停止）。这就是说你可以在Activity重新创建的时候从cache()的返回值中创建一个新的Observable对象。
```java
Observable<Photo> request = service.getUserPhoto(id).cache();
	Subscription sub = request.subscribe(photo -> handleUserPhoto(photo));
	
	// ...When the Activity is being recreated...
	sub.unsubscribe();
	
	// ...Once the Activity is recreated...
	request.subscribe(photo -> handleUserPhoto(photo));
```
	
注意，两次sub是使用的同一个缓存的请求。当然在哪里去存储请求的结果还是要你自己来做，和所有其他的生命周期相关的解决方案一延虎，必须在生命周期外的某个地方存储。（retained fragment或者单例等等）。

第二个问题的解决方案就是在生命周期的某个时刻取消订阅。一个很常见的模式就是使用CompositeSubscription来持有所有的Subscriptions，然后在onDestroy()或者onDestroyView()里取消所有的订阅。
```java
 private CompositeSubscription mCompositeSubscription
    = new CompositeSubscription();

	private void doSomething() {
	    mCompositeSubscription.add(
	        AndroidObservable.bindActivity(this, Observable.just("Hello, World!"))
	        .subscribe(s -> System.out.println(s)));
	}
	
	@Override
	protected void onDestroy() {
	    super.onDestroy();
	
	    mCompositeSubscription.unsubscribe();
	}
```
你可以在Activity/Fragment的基类里创建一个CompositeSubscription对象，在子类中使用它。

注意! 一旦你调用了 CompositeSubscription.unsubscribe()，这个CompositeSubscription对象就不可用了, 如果你还想使用CompositeSubscription，就必须在创建一个新的对象了。

最后，附上测试时使用的完整工程，代码上面均有贴出来了。
[RxAndroid基本使用测试代码](http://download.csdn.net/detail/yaodong379/9486905)
