---
layout: post
title: Docker小记-Dockerfile
excerpt: Dockerfile是一个文件，由构建镜像的指令构成。本文介绍使用Dockerfile自动化打包
categories: Docker
keywords: Docker
---
## Dockerfile
- docker build构建镜像：
	- --tag（或-t）选项的值指定了你想要使用的完整仓库设计。下例中，使用了ubuntu-git:auto。最后的参数则指定了Dockerfile的位置，表示在当前目录寻找文件。
	- docker build 命令还有另外一个选项--file（或-f），这个选项让你能够设置Dockerfile的名字。Dockerfile是默认的文件名字。这个选项只能设置文件的名字，而不能设置文件的位置。最后一个参数是设置位置	
	- 如果需要完整地从零开始构建，使用--no-cache选项来禁止缓存的使用。但注意，确保完全需要时才禁止缓存。
- 例子：在包含Dockerfile文件的目录中使用docker build命令，从Dockerfile文件创建一个新镜像，并将新镜像的标签设为auto
```
docker build -tag ubuntu-git:auto .
```

## 元数据指令
```
FROM debian：wheezy
MAINTAINER yourName "xx@xx.com"
RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server
ENV VERSION="0.1.1" \   
    APPROOT="/APP"
LABEL base.version="${VERSION}"
WORKDIR ${APPROOT}
ADD . ${APPROOT}
ENTRYPOINT ["/app/mailer.sh"]
EXPOSE 33333
```
- 定义哪些文件永远不应该被复制进镜像中。你可以在名为.dockerignore的文件中指定这些信息。
- 每个 Dockerfile 指令都会导致一个新层被创建。指令应该尽可能合并，这是因为构建程序不会进行任何的优化。
- FROM指令使得栈从某个镜像顶部开始，任何被构建的新层都会被放置在这个镜像的最上层。
- MAINTAINER指令设置了镜像元数据Author的值。
- ENV，类似于docker run或docker create命令的--env选项，ENV指令设置了镜像的环境变量。
	- Dockerfile文件中声明的环境变量不仅对产生的镜像有效，它们还能够在其他Dockerfile指令中使用
- LABEL 指令用来定义键值对，这些键值对被记录为镜像或容器的额外元数据
- 使用WORKDIR指令指定默认工作目录，当指定的WORKDIR目录不存在，那么这个目录会被自动创建。
- EXPOSE指令创建了一个层，对外开放端口
- ENTRYPOINT指令则设置了在容器启动时需要被运行的可执行程序。比如说入口点被设置为exec ./mailer.sh，这使用了该指令的shell格式。
	- ENTRYPOINT指令有两种格式：shell格式和exec格式。shell格式类似于一个shell命令，其中的参数以空格为界限分隔开来。exec格式是一个字符串的数组，其中第一个值是要执行的命令，剩下的值则是参数。
	- shell格式指定的命令将会被作为默认shell的一个参数来执行具体点说，指定的命令在运行时会以/bin/sh -c ＇exec ./mailer.sh＇的形式执行。
	- 如果ENTRYPOINT使用了shell格式，那么CMD指令提供的所有其他参数，或docker run命令在运行时指定的额外参数都会被忽略。这使得ENTRYPOINT的shell格式不那么灵活可变动了。
- 可以在基础镜像创建用户和用户组账户，然后让具体的实现者在他们完成构建时再设置默认的用户。

## 文件系统指令
- Dockerfile 定义了三个指令来修改文件系统：COPY、VOLUME和ADD。
- COPY指令至少需要两个参数。最后一个参数是目的目录，其他所有的参数则为源文件。
	- 任何被复制的文件的所有权都会被设设置为root用户，因此，最好在所有需要更新的文件都复制到镜像后，再使用RUN指令来修改文件的所有权。
	- COPY指令同样支持shell格式和exec格式。但是如果任何一个参数包含了空格，那么你必须要使用exec格式。
- VOLUME:在字符串数组参数中的每一个值都会在产生的新层中被创建为一个新的卷定义。在镜像构建时定义卷比在运行时更加受到限制
- CMD 指令表示入口点的一个参数列表。一个容器的默认入口点是/bin/sh。如果一个容器的入口点没有被设置，这个默认值会被使用。如果入口点被设
设置了，并且使用的是exec格式，那么将使用CMD指令来设置默认参数。
- ADD指令：如果指定了一个URL，会拉取远程源文件；会将被判定为存档文件的源中的文件提取出来
	- 使用ADD指令的远程拉取功能并不是一个好的实践。原因在于尽管这个特性非常方便，但是它没有提供任何机制来清理不被使用的文件，这会导致额外的层。作为替代品，你应该使用链状的RUN指令
```
curl "https://bootstrap.pypa.io/get-pie.py" -o "get-pip.py"
```

## 注入下游镜像在构建时发生的操作
- 如果生成的镜像被作为另一个构建的基础基础镜像，则 ONBUILD 指令定义了需要被执行的那些指令
- 上游的 Dockerfile一般会使用类似以下形式的指令：
```
ONBUILD COPY[“.”,“/var/myapp”]
ONBUILD RUN go build/var/myapp
```
- 跟随在 ONBUILD 后的指令不会在包含它们的 Dockerfile 被构建时被执行。这些指令会被记录在生成镜像的元数据ContainerConfig.OnBuild下。
- 这个元数据会一直被保留，直到生成的镜像被另外的 Dockerfile 作为基础镜像。当一个下游的 Dockerfile 通过 FROM指令使用了上游的镜像（带有ONBUILD指令的Dockerfile产
产生的镜像），那么这些在ONBUILD 后跟随的指令将会在 FROM 指令后，下一条指令前被执行。


## 初始化进程
- 基于UNIX的计算机通常会先启动一个初始化（init）进程。这个init进程负责启动所有其他的系统服务，让它们持续运行，然后负责关闭它们。使用一个 init 风格的系统来启动、管理、重启、关闭容器进程通常是恰当的。
- 使用一个 init 进程是启动多个程序、清理遗弃的进程、监控进程和自动化重启失败进程的最佳方式。
- 轻量级的init程序:主流的选择包括runit、Busybox init、Supervisord和DAEMON工具


## 加固应用镜像
- 加固一个镜像就是塑造镜像，使得基于这个镜像创建的任何Docker容器的攻击面减减少的过程
- 加固应用镜像的一个通用策略就是最小化包含在其中的软件。
- 通用指南就是尽可能地削减特权
- 可以强制基于某个特定的镜像来构建镜像,镜像作者就能够强制从一个特定的，且未改动的起点开始构建。
	- 在标准的tag位置后面添加一个@符号，符号后面跟随的就是摘要。一旦你拥有了这个摘要，你可以将它作为ID在Dockerfile中的FROM指令中使用
- 你能够确保无论容器如何基于你的镜像来构建，它们都会拥有一个合适的默认用户
    - Dockerfile包含了一个USER指令，和docker run或docker create命令一样，它能够设置用户和用户组。
- 你应该去除root用户提权的通用途径
- SUID和SGID权限:一个设置有 SUID 位的可执行文件总是会以它的所有者用户来执行,会从拥有该程序的用户组的上下文执行，而不是程序所有者。
```
// 将镜像中所有文件的 SUID和SGID权限都去除
RUN for i in $(find/-type f ( -perm +6000 -o -perm +2000 ));
do chmod ug-s $i; done
```
