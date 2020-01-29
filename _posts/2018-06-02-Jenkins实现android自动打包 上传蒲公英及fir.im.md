---
layout: post
title: Jenkins实现android自动打包 上传蒲公英及fir.im
excerpt: Jenkins实现android自动打包 上传蒲公英及fir.im
categories: Android
keywords: Android, Jenkins
---
## 下载jenkins
 https://jenkins.io/index.html 
 下载后得到jenkins.msi文件，直接安装
 
## 访问http://localhost:8080
选择安装推荐的插件之后进入主界面

## 创建项目
选择构建一个自由风格的软件项目
配置如下
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-f0DUZa4O-1580199892275)(https://i.imgur.com/VeATqY3.png)]
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-FzsTBuFR-1580199892275)(https://i.imgur.com/zlTmwVc.png)]
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-LlZld5bu-1580199892276)(https://i.imgur.com/zaCdqyn.png)]
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-RhAfw8Mp-1580199892277)(https://i.imgur.com/xYURJAd.png)]

保存后点击立即构建即可开始构建，构建成功后项目目录下会生成apk
## 自动上传应用到蒲公英
[使用 Jenkins 插件上传应用到蒲公英](https://www.pgyer.com/doc/view/jenkins_plugin)
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ZdiDRhhc-1580199892278)(https://i.imgur.com/Pe8nYy4.png)]
## 自动上传应用到fir.im
[fir.im Jenkins 插件使用方法](https://www.jianshu.com/p/9a245918a219)
新版本插件需要上传dysm file否则会报错，旧版插件（链接：https://pan.baidu.com/s/14CHeexAwrpvUkwQUfcKQjA 密码：s246）
## Jenkins参数化构建
通过配置一下参数，来满足一些需求，比如根据渠道打不同版本的包、根据Tag打不同的包等
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-5L1t32vO-1580199892279)(https://i.imgur.com/t3XARan.png)]
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-3aWyPAfY-1580199892279)(https://i.imgur.com/DSgBLdh.png)]
Pass all job parameters as Project properties选项可以帮我们侵入到gradle.properties文件中替换值，并且build.gradle文件能够直接引用gradle.properties文件中的属性，所以起到了侵入的效果
gradle.properties

```
isDebugServer=false
```

gradle:

```
  buildTypes {
        debug {
            signingConfig signingConfigs.release
            if (isDebugServer.toBoolean()) {
                buildConfigField "boolean", "isDebugServer", "true"
                resValue "string", "app_name", "app_debug"
            }else {
                buildConfigField "boolean", "isDebugServer", "false"
                resValue "string", "app_name", "app"  
            }
        }
        release {
           //...
        }
    }
```


## Jenkins相关操作
### 关闭Jenkins

只需要在访问jenkins服务器的网址url地址后加上exit。例如jenkins的地址http://localhost:8080/，那么只需要在浏览器地址栏上敲下http://localhost:8080/exit 网址就能关闭jenkins服务.
### 重启Jenkies

    http://localhost:8080/restart
### 重新加载配置信息

    http://localhost:8080/reload
## 参考文章
[Jenkins+Gradle实现android开发 自动打包 上传蒲公英](https://blog.csdn.net/liudao7994/article/details/54177126)
[Android Jenkins+Git+Gradle持续集成-实在太详细](https://www.jianshu.com/p/38b2e17ced73)

 
