---
layout: post
title: Android数据库管理 
categories: Android
keywords: Android，Activeandroid
---
ActiveAndroid是一个轻量级的ORM框架，可以以类的方式简单快捷地进行数据库的管理，而无需编写一个单独的SQL语句。
[ActiveAndroid git地址](https://github.com/pardom/ActiveAndroid)

## 配置
1、AndroidManifeset中添加如下配置：
```java
        <application
        ...
        android:name="com.activeandroid.app.Application"
        >

        <meta-data
            android:name="AA_DB_NAME"
            android:value="xxx.db" />
        <meta-data
            android:name="AA_DB_VERSION"
            android:value="7" />

        <meta-data
            android:name="AA_MODELS"
            android:value="com.syd.oden.odendemo.entity.sqltab.LocationTab, com.syd.oden.odendemo.entity.sqltab.MusicFavorTab" />
```
AA_MODELS为数据库中表的实体类

2、Application继承com.activeandroid.app.Application
```java
public class MyApplication extends com.activeandroid.app.Application {}
```
或者
```java
public class MyApplication extends SomeLibraryApplication {
    @Override
    public void onCreate() {
        super.onCreate();
        ActiveAndroid.initialize(this);
    }
        @Override
    public void onTerminate() {
        super.onTerminate();
        ActiveAndroid.dispose();
    }
}

```

3、创建表
```java
@Table(name = "PictureTabs")
public class PictureTab extends Model {
    private static MyLog myLog = new MyLog("[PictureTab] ");

    @Column(name = "dirName")
    String dirName;

    @Column(name = "fileName")
    String fileName;

    @Column(name = "describe")
    String describe;

    @Column(name = "longitude")
    double longitude;

    @Column(name = "latitude")
    double latitude;

    public PictureTab() {
        super();
    }

    public PictureTab(String dirName, String fileName, double longitude, double latitude) {
        super();
        this.dirName = dirName;
        this.fileName = fileName;
        this.longitude = longitude;
        this.latitude = latitude;
    }
}
```

## 增删改查
### 增
```java
       for (int i=0; i<5; i++) {
            DbBlesGroup dbBleGroup = new DbBlesGroup();
            dbBleGroup.groupIndex = i;
            dbBleGroup.groupName = "groupName" + i;
            dbBleGroup.addr = "addr" + i;
            dbBleGroup.name = "name" + i;
            dbBleGroup.save();
        }
```

### 查
查出所有
```java
List<DbBlesGroup> dbBleGroupList = new ArrayList<>();
        dbBleGroupList = new Select()
                .from(DbBlesGroup.class)
                .orderBy("groupName ASC")
                .execute();

        for (int i=0; i<dbBleGroupList.size(); i++)
        {
            L.d("dbBleGroupList :" + dbBleGroupList.get(i).groupName);
        }
```
指定条件查找
```java
List<DbBlesGroup> dbBleGroupList = new ArrayList<>();
        dbBleGroupList = new Select()
                .from(DbBlesGroup.class)
                .where("groupName = ?", "groupName3")
                .orderBy("groupName ASC")
                .execute();

```
多条件查找
```java
newSelect().from(UserViewTab.class).where("viewId=? and bleAddr=?",viewId,addr).executeSingle();
```
使用事务(transaction)
```java
ActiveAndroid.beginTransaction();
try {
        for (int i = 0; i < 100; i++) {
            Item item = new Item();
            item.name = "Example " + i;
            item.save();
        }
        ActiveAndroid.setTransactionSuccessful();
}
finally {
        ActiveAndroid.endTransaction();
}

```
.orderBy("id DESC")降序
.orderBy("id ASC")升序

### 删除
```java
new Delete().from(DbBlesGroup.class).where("groupName = ?", "groupName2").execute();
```

### 改
```java
new Update(DbBlesGroup.class).set("addr = ?", "123").where("groupName = ?", "groupName2").execute();
```
也可直接用save修改

## 注意事项
1、构造方法中记得加入super();

2、在sudio2.2运行报错解决：
```java
erro： 'java.lang.String com.activeandroid.TableInfo.getTableName()' on a null object reference.
```
关掉Instant Run
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-vII9ExEQ-1579785381898)(http://i.imgur.com/lUDmLhb.png)]
3、表中包含另一个表，则保存的时候要先保存另一个表；
发现一个bug，表中包含另一个表，查另一个表里的数据可能有误
```java
 recipeAlarmList.add(RecipeAlarmTab.getById(recipeTab.getRecipeAlarmTab1().getId())); //activeAndroid貌似有bug，故通过ID重新查询一次
```


