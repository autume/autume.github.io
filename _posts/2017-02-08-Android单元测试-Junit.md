---
layout: post
title: Android单元测试-Junit
categories: Android
excerpt: Android单元测试-Junit
keywords: Android，Junit
---
## 基本用法实际操作

1、新建测试类
```java
public class Caculation {
    public double sum(double numA, double numB) {
        return numA + numB;
    }

    public double multiply(double numA, double numB) {
        return numA * numB;
    }
}

```
2、类名右键--Go To--Test，生成测试类
项目中androidTest文件夹里的就是UI测试代码，而test文件夹是Junit部分的单元测试代码。

3、编辑测试类,右键类名或者方法可运行得出测试结果
```java
public class CaculationTest {
    private Caculation mCaculation;

    @Before
    public void setUp() throws Exception {
        mCaculation = new Caculation();
    }

    @Test
    public void sum() throws Exception {
        assertEquals(2,mCaculation.sum(1,1),0);
    }

    @Test
    public void multiply() throws Exception {
        assertEquals(10,mCaculation.multiply(2,5),0);
    }

}
```
## Junit断言及注解说明
### 断言
- assertArrayEquals(expecteds, actuals)	查看两个数组是否相等。
- assertEquals(expected, actual)	查看两个对象是否相等。类似于字符串比较使用的equals()方法
- assertNotEquals(first, second)	查看两个对象是否不相等。
- assertNull(object)	查看对象是否为空。
- assertNotNull(object)	查看对象是否不为空。
- assertSame(expected, actual)	查看两个对象的引用是否相等。类似于使用“==”比较两个对象
- assertNotSame(unexpected, actual)	查看两个对象的引用是否不相等。类似于使用“!=”比较两个对象
- assertTrue(condition)	查看运行结果是否为true。
- assertFalse(condition)	查看运行结果是否为false。
- assertThat(actual, matcher)	查看实际值是否满足指定的条件
- fail()	让测试失败

注意：上面的方法，都有一个重载的方法，可以在前面加一个String类型的参数，表示如果验证失败的话，将用这个字符串作为失败的结果报告。
比如：
assertEquals("Current user Id should be 1", 1, currentUser.id());
## 注解
- @Before	初始化方法
- @After	释放资源
- @Test	测试方法，在这里可以测试期望异常和超时时间
- @Ignore	忽略的测试方法
- @BeforeClass	针对所有测试，只执行一次，且必须为static void
- @AfterClass	针对所有测试，只执行一次，且必须为static void
- @RunWith	指定测试类使用某个运行器
- @Parameters	指定测试类的测试数据集合
- @Rule	允许灵活添加或重新定义测试类中的每个测试方法的行为
- @FixMethodOrder	指定测试方法的执行顺序

一个测试类单元测试的执行顺序为：
@BeforeClass –> @Before –> @Test –> @After –> @AfterClass
每一个测试方法的调用顺序为：
@Before –> @Test –> @After

## 打包测试
同样，如果一个项目中有很多个测试用例，如果一个个测试也很麻烦，因此打包测试就是一次性测试完成包中含有的所有测试用例。
```java
@RunWith(Suite.class)  
@Suite.SuiteClasses({ AssertTests.class, CaculationTest.class, DemoTest.class })  
public class AllCaseTest {  
  
}  
```
需要向@RunWith注解传递一个参数Suite.class 。同时，我们还需要另外一个注解@Suite.SuiteClasses，来表明这个类是一个打包测试类。并将需要打包的类作为参数传递给该注解就可以了。至于AllCaseTest随便起一个类名，内容为空既可。运行AllCaseTest类即可完成打包测试

## 限时测试
在@Test后加入timeout参数
```java
 @Test(timeout=1000)
    public void multiply() throws Exception {
        assertEquals(10,mCaculation.multiply(2,5),0);
    }
```
直接用@Rule设置该类的timeout参数
```java
@Rule
    public Timeout globalTimeout = new Timeout(10000); // 10 seconds max per method tested  

```
## 验证方法会抛出某些异常
```java
public class Calculator {

    // Omit testAdd() and testMultiply() for brevity

    public double divide(double divident, double dividor) {
        if (dividor == 0) throw new IllegalArgumentException("Dividor cannot be 0");

        return divident / dividor;
    }}
```

```java
public class CalculatorTest {
    Calculator mCalculator;

    @Before
    public void setup() {
        mCalculator = new Calculator();
    }

    // Omit testAdd() and testMultiply() for brevity

    @Test(expected = IllegalArgumentException.class)
    public void test() {
        mCalculator.divide(4, 0);
    }

}
```
@Test(expected = IllegalArgumentException.class)表示验证这个测试方法将抛出IllegalArgumentException异常，如果没有抛出的话，则测试失败。

