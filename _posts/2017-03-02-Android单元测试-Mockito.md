---
layout: post
title: Android单元测试-Mockito
categories: Android
keywords: Android，Mockito
---
本文记录Mockito的简单使用。


## Mockito概念相关
Mock就是创建一个类的虚假对象，在测试环境中，用来替换掉真实的对象，主要提供两大功能：
- 验证这个对象的某些方法的调用情况，调用了多少次，参数是什么等等
- 指定这个对象的某些方法的行为，返回特定的值，或者是执行特定的动作
Mockito是Java界使用最广泛的一个mock框架。
添加 Mockito 依赖
```java
    testCompile "org.mockito:mockito-core:2.0.57-beta"
```

- Mockito.mock()并不是mock一整个类，而是根据传进去的一个类，mock出属于这个类的一个对象，并且返回这个mock对象；而传进去的这个类本身并没有改变，用这个类new出来的对象也没有受到任何改变！
- Mockito.verify()的参数必须是mock对象，也就是说，Mockito只能验证mock对象的方法调用情况。
- mock出来的对象并不会自动替换掉正式代码里面的对象，你必须要有某种方式把mock对象应用到正式代码里面

## Mockito使用
### 验证方法调用
```java
Mockito.verify(mockUserManager).performLogin("xiaochuang", "xiaochuang password");
```
这行代码其实是：
Mockito.verify(mockUserManager, Mockito.times(1)).performLogin("xiaochuang", "xiaochuang password");
的简写.
对于调用次数的验证，除了可以验证固定的多少次，还可以验证最多，最少从来没有等等，方法分别是：atMost(count), atLeast(count), never()等等

Mockito.verify(mockUserManager).performLogin(Mockito.anyString(), Mockito.anyString());
anyString()表示任何一个字符串都可以。
类似anyString，还有anyInt, anyLong, anyDouble等等。anyObject表示任何对象，any(clazz)表示任何属于clazz的对象

### 指定mock对象的某些方法的行为
指定返回
```java
Mockito.when(mockObject.targetMethod(args)).thenReturn(desiredReturnValue);

//当调用mockValidator的verifyPassword方法时，返回true，无论参数是什么
Mockito.when(validator.verifyPassword(anyString())).thenReturn(true);
```
执行特定的动作
```java
Mockito.doAnswer(desiredAnswer).when(mockObject).targetMethod(args);

//例子
 Mockito.doAnswer(new Answer() {
    @Override
    public Object answer(InvocationOnMock invocation) throws Throwable {
        //这里可以获得传给performLogin的参数
        Object[] arguments = invocation.getArguments();

        //callback是第三个参数
        NetworkCallback callback = (NetworkCallback) arguments[2];

        callback.onFailure(500, "Server error");
        return 500;
    }
}).when(mockUserManager).performLogin(anyString(), anyString(), any(NetworkCallback.class));
```
当调用mockUserManager的performLogin方法时，会执行answer里面的代码，上面的例子是直接调用传入的callback的onFailure方法，同时传给onFailure方法500和"Server error"
### spy
如果不指定的话，一个mock对象的所有非void方法都将返回默认值：int、long类型方法将返回0，boolean方法将返回false，对象方法将返回null等等；而void方法将什么都不做。
spy与mock的唯一区别就是默认行为不一样：spy对象的方法默认调用真实的逻辑，mock对象的方法默认什么都不做，或直接返回默认值。

## Mockito注解的使用
### @Mock的基本用法
需要mock的对象可用@Mock注解，同时需要调用 MockitoAnnotations.initMocks(target);
```java
   @Mock
    UserManager mockUserManager;

    @Mock
    PasswordValidator mockValidator;

@Before
public void setup() {
    MockitoAnnotations.initMocks(this);
    loginPresenter = new LoginPresenter(mockUserManager, mockValidator);
}
```

### @InjectMocks
我们可以使用@InjectMocks来让Mockito自动使用mock出来的mockUserManager和mockValidator构造出一个LoginPresenter
```java
 @InjectMocks
    LoginPresenter loginPresenter;
```

## 参考文章
[Android 单元测试--Mock及Mockito](http://hujiandong.com/2016/11/07/android-unit-test-mock/)
[Mockito](http://www.vogella.com/tutorials/Mockito/article.html)
[使用Mockito Annotation快速创建Mock](http://www.jianshu.com/p/7f6a1d3aa516)
