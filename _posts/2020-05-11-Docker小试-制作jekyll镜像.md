---
layout: post
title: Docker小试-制作jekyll镜像
excerpt: 提供两种制作jekyll镜像的方式及最终的镜像文件
categories: Docker
keywords: Docker
---
## 方式一，进入容器中制作镜像
- 拉取ubuntu18.04镜像：docker pull ubuntu:18.04
- 运行并进入容器：docker run -it --rm --name ubuntu_base ubuntu:18.04 /bin/sh
- 安装相关依赖
	- apt-get update
	- apt-get upgrade
	- apt-get install build-essential
	- apt-get install ruby ruby-dev
	- gem install jekyll bundler
- 新开个命令窗口，提交新镜像：docker commit ubuntu_base oden379/jekyll
- 上传镜像：docker push oden379/jekyll (镜像已上传，可下载测试使用)
### 运行
- ~/Documents/JekyllProject/demo中存放的为jekyll的工程，修改jekyll中_config中的host为0.0.0.0,或者启动的时候指定--host 0.0.0.0
- 运行新镜像：docker run -it --rm --name jekyll_test -v ~/Documents/JekyllProject/demo/docker_demo:/www/jekyll -p 4000:4000 oden379/jekyll /bin/sh，在容器中/www/jekyll目录下运行：jekyll server
	- 或者直接一条语句搞定：docker run -it --rm --name jekyll_test -v ~/Documents/JekyllProject/demo/docker_demo:/www/jekyll -p 4000:4000 oden379/jekyll sh -c "cd /www/jekyll/my-demo && jekyll serve"
- 主机通过localhost：4000即可访问网页，修改主机目录下的文件后刷新网页可看到实时更新

## 方式二，使用Dockerfile制作镜像
```
FROM ubuntu:18.04

RUN echo Y | apt-get update \
     && echo Y | apt-get upgrade  \
     && echo Y | apt-get install build-essential \
     && echo Y | apt-get install ruby ruby-dev \
     && echo Y | gem install jekyll bundler \
     && mkdir -p /www/jekyll

COPY ./start.sh /usr/src/app/

WORKDIR /usr/src/app/

EXPOSE 4000
ENTRYPOINT ["/bin/sh", "./start.sh"]

```
或者

```
FROM ubuntu:18.04

RUN echo Y | apt-get update \
     && echo Y | apt-get upgrade  \
     && echo Y | apt-get install build-essential \
     && echo Y | apt-get install ruby ruby-dev \
     && echo Y | gem install jekyll bundler \
     && mkdir -p /www/jekyll


EXPOSE 4000
ENTRYPOINT ["/bin/sh"]
CMD ["-c","cd /www/jekyll/my-demo && jekyll serve"]
```
- start.sh:

```
#!/bin/bash
cd /www/jekyll/my-demo && jekyll serve
```

- build：docker build -t jekyll_test2 .
- 运行：docker run -it --rm --name jekyll_test2 -v ~/Documents/JekyllProject/demo/docker_demo:/www/jekyll -p 4000:4000 jekyll_test2
