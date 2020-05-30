---
layout: post
title: Docker小记-Docker中运行软件
excerpt: 介绍在容器中运行软件及相关的命令
categories: Docker
keywords: Docker
---
## Docker命令
- docker help <command>:显示Docker命令行工具的基本语法
- docker ps：显示每个运行的容器
- docker logs <dockerName>: 后接docker名称或id显示日志
    - 写入的日志将持久化保存、持续增长，只要该容器还存在。长期持久性会成为长期进程的一个问题。一个更好的方式是，使用存储卷来处理日志数据
    - docker logs命令有一个标志，--follow或-f，用来显示整个日志，然后将继续监视和更新日志的显示，不放过任何日志中的变化
- docker restart <dockerName>
- docker stop <dockerName>
- docker exec：在运行的容器中运行额外的进程。
- docker create：和docker run很类似，主要区别在于该容器是被停止状态创建
- docker rename：重命名该容器
- docker top：显示的是主机为每一个容器中的进程所分配的 PID
- docker rm: 删除容器，可以通过在命令中指定--rm来避免清理工作的负担。这样做，当容器进入退出状态时，就会被自动删除。

```
docker exec web ps
```
## 守护式容器
- 适合那些在后台静默运行的程序，这类程序被称为守护程序。守护程序通常通过网络或其他通信工具和其他程序或人进行交互。
- 在后台运行容器的守护程序，使用--detach标志或其缩写形式-d。
```
docker run --detach --name web nginx:latest
```
## 交互式容器
- 使用--interactive（或-i）和--tty（或-t）
    - --interactive选项告诉Docker保持标准输入流（stdin，标准输入）对容器开放，即使容器没有终端连接。
    - --tty 选项告诉 Docker 为容器分配一个虚拟终端，这将允许你发信号给容器
- 启动容器后，得让程序在容器内运行起来。在这种情况下，运行一个叫作sh的shell程序。这样就可以在容器内运行任何程序
- 输入exit来关闭这个互动容器。这将终止shell程序，并停止该容器。
- 按住【Ctrl】（或【Control】）键，然后按【P】键，再按【Q】键，之后就会返回到主机的shell且该容器继续运行。
```
// 创建虚拟终端并绑定标准输入
docker run --interactive --tty 
-- link web:web \
--name web_test \
busybox:latest /bin/sh
```

## PID命名空间
- 每一个运行的程序或进程，在Linux机器都有一个唯一编号，叫作进程标识符（PID）。
- 每个命名空间拥有一套完整的PID,为每个容器创建一个PID命名空间是Docker的关键特征。
- 从进程的一个命名空间角度来看，PID1可能是指像runit或supervisord这样的init系统进程。在不同的命名空间中，PID1可能是指诸如bash的shell命令。

## 只读文件系统
- --read-only

```
docker run -d --name wp --read-only wordpress:4
```
- docker inspect命令将显示Docker为该容器保留的所有元数据（一个JSON文件）。格式选项会改变元数据。除了该容器的运行状态，下例中其会滤除元数据的所有字段。

```
docker inspect --format ＂{{.State.Running}}＂ wp
```
## 环境变量的注入
- -env标志或-e缩写，可用于注入任何环境变量。如果变量已经由镜像或Docker设置，则该值将被覆盖

## 自动重启容器
### restart
- -restart 标志，就可以通知Docker完成以下操作：
    - 从不重新启动（默认）
    - 检测到故障时尝试重新启动
    -当检测到故障时，在一段预预定的时间后重新开始尝试重启
    - 不管任何条件，始终重新启动容器
- 回退策略决定了连续尝试重新启动所需要的时间间隔。指数回退策略会将花在前一次等待连续尝试的时间加倍。例如，如果第一次容器重新启动Docker需要等待1秒钟，然后第二次尝试将等待2秒，第三次等待4秒，第四次等待8秒，以此类推。

```
docker run -d --name backoff-detector --restart always wp
```
### init、supervisor
- init或supervisor进程，用于启动和维护其他程序状态。在Linux系统中，PID #1是init进程。它启动所有其他系统进程，并在出现意外故障时重新启动它们。容器中使用类似的模式来启动和管理进程，是一个常见的做法。
- 容器中的supervisor进程用来保持容器始终运行，即使目标进程（如一个web服务器），出现故障并重新启动。一个容器中可能有多个这样程序，最流行的包括init、systemd、runit、upstart和supervisord
- 使用init或supervisor程序的一个常见替代方法是使用一个启动脚本，该脚本至少会检查软件成功启动的先决条件。这些脚本有时会用作容器的默认命令
- 使用--entrypoint标志来运行指定程序，并传递参数
```
docker run --entrypoint="cat" \  // 使用cat作为容器执行的入口
nginx:latest /entrypoint.sh      // 将/entrypoint.sh作为cat命令的参数
```