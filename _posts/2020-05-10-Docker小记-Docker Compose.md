---
layout: post
title: Docker小记-Docker Compose
excerpt: Compose描述完整的环境以及服务组件的交互
categories: Docker
keywords: Docker
---
## 命令
- Compose描述完整的环境以及服务组件的交互。一个Compose文件可能会描述四到五个单独的服务，它们都是相互关联的，但应保持隔离和独立伸伸缩。
- 启动：进入创建docker-compose.yml文件的目录并运行以下的命令：
```
docker-compose up
```
- 查看日志(想只看到一个或多个服务,就给出那些服务的命名)
```
docker-compose logs
```
- 列出当前目录下yml文件定义的所有容器
```
docker-compose ps
```
- 清理环境
```
docker-compose stop 
docker-compose rm -vf
``` 
会删除所有的服务或者一个由环境定义的特定服务。一个小的区别是，-f 选项并不强迫删除正在运行的容器，相反，它会关闭用户确认阶段。-v选项清理卷
- 重新构建环境:当你执行这个命令后，它将停止当前并删除运行的容器，接着创建新的容器，并重新附着前代环境挂载的数据卷，
```
docker-compose up -d
```
- 重建一个或所有服务(想只重建一个或多个服务,就给出那些服务的命名)
```
docker-compose build
```
- 拉取环境中不存在的镜像
```
docker-compose pull
```

## 构建、启动和重新构建服务
- 当Compose启动任何特定的服务时，它将启动所有其依赖的服务。
- 当你使用一个未经限定的 docker-compose up 命令时，Compose 将创建或重新创建环境中的每一个服务并启动所有的服务，如果Compose检测到有任何还没构建或者使用了缺失镜像的服务，它会触发一个构建或获取合适的镜像（就像docker run命令）。
- 如果知道一个运行正确的某个特定服务的依赖关系，你可以不需要依赖关系就可以启动或重新启动一个服务。为此，需要引入flag--no-dep。
```
docker-compose up --no-dep -d proxy
```

## 服务伸缩和删除
- Compose 能够支持服务的纵向伸缩，当这么做时，Compose 创建了提供该服务的容器的多个副本。这些副本将在你缩小规模时会自动清理
使用以下的命令扩展api服务：
```
docker-compose scale api=5
```
- 端口号0为主机的临时端口，当你绑定到端口0时，操作系统将在一个预定义范围内选择一个可用的端口

## 迭代和持久化状态
- 当服务重新构建时，附加的管理卷不会被删除。相反，它们重新附加到了那个服务更换后的容器上。这意味着可以自由地迭代而不用担心丢失数据
- docker-compose rm命令和flag-v删除时，管理卷最后也会被清理了。
- 如果你在 docker-compose.yml 文件中重命名或删除一个服务定义，那么你就失去了使用Compose管理这个服务的能力。重新构建并重启后，新的服务将会工作，而旧服务被孤立。
- 解决方法是直接使用docker命令清理环境或者回到docker-compose.yml文件中添加孤立的服务定义，用Compose清理。

## 网络和连接问题
- Docker构建的镜像是通过创建防火墙规则和注入服务发现信息到所依赖的容器的环境变量和/etc/hosts文件中来建立连接关系
- 在高度迭代的环境中，用户可能只要重启特定的服务，如果另外一个服务依赖于它的话，这可能会导致一些问题。举例来说，如果启动了Coffee API环境，然后选择性地重启coffee服务，proxy服务将不再能够追溯到它的上游依赖。当容器重新创建或者重新启动后，它们返回的是不同的IP地址。这一变化使得注入proxy服务的信息失效了。
- 在没有动态服务发现机制的环境中处理这个问题最好的方法是重启整个环境

## 示例
```
data:
  image: gliderlabs/alpine
  command: echo Data Container
  user: 999:999
  labels:
    com.dockerinaction.chapter: "11"
    com.dockerinaction.example: "Coffee API"
    com.dockerinaction.role: "Volume Container"

dbstate:
  extends:
    file: docker-compose.yml
    service: data
  volumes:
    - /var/lib/postgresql/data/pgdata

# the postgres image uses gosu to change to the postgres user
db:
  image: postgres
  volumes_from:
    - dbstate
  environment:
    - PGDATA=/var/lib/postgresql/data/pgdata
    - POSTGRES_PASSWORD=development
  labels:
    com.dockerinaction.chapter: "11"
    com.dockerinaction.example: "Coffee API"
    com.dockerinaction.role: "Database"

# the nginx image uses user de-escalation to change to the nginx user
proxy:
  image: nginx
  restart: always
  volumes:
    - ./proxy/app.conf:/etc/nginx/conf.d/app.conf
  ports:
    - "8080:8080"
  links:
    - coffee
  labels:
    com.dockerinaction.chapter: "11"
    com.dockerinaction.example: "Coffee API"
    com.dockerinaction.role: "Load Balancer"

coffee:
  build: ./coffee
  user: 777:777
  restart: always
  expose:
    - 3000
  ports:
    - "0:3000"
  links:
    - db:db
  environment:
    - COFFEEFINDER_DB_URI=postgresql://postgres:development@db:5432/postgres
    - COFFEEFINDER_CONFIG=development
    - SERVICE_NAME=coffee
  labels:
    com.dockerinaction.chapter: "11"
    com.dockerinaction.example: "Coffee API"
    com.dockerinaction.role: "Application Logic"
```
- build键的值是用于构建的Dockerfile文件所在位置的目录，你可以使用从YAML文件位置开始的相对路径
- dockerfile键来提供一个备选的Dockerfile文件的名称。
- environment键可以为一个服务设置环境变量
- 容器元数据可以设置为以 labels 为键，
- expose键接受容器端口的一个列表，这些端口应该根据防火墙规则公开。
- ports键按照与docker run命令中-p选项一样的格式接受描述了端口映射关系的字符串列表。
- links命令按照与docker run命令中flag --link一样的格式接受和定义了链接的列表。
- proxy服务使用一个卷来绑定挂载一个本地的配置文件到Nginx的动态配置位置，这是一个简单的注入配置而不用构建一个完整的新镜像。
- db服务使用volumes from键来列出那些定义了必需卷的服务，
- 服务扩展：必须指定文件和被扩展的服务名称，相关的键是extends以及内嵌的file和service。服务扩展工作类似于 Dockerfile 构建的方式。首先构建原型容器，然后提交。子容器是一个由新生成的层构建的新容器。就像Dockerfile构建，这些子容器继承了父容器所有的属性，包括元数据。
