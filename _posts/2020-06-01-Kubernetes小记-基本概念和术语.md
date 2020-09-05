---
layout: post
title: Kubernetes小记-基本概念和术语
excerpt: Kubernetes基本概念
categories: Kubernetes
keywords: Kubernetes
---
## Kubernetes
- Kubernetes是基于容器技术的分布式架构领先方案，目的是实现资源管理的自动化，以及跨多个数据中心的资源利用率的最大化。
- Kubernetes提供了强大的自动化机制，所以系统后期的运维难度和运维成本大幅度降低。

## 基本概念和术语
- Node、Pod、Replication Controller、Service等都可以被看作一种资源对象，几乎所有资源对象都可以通过Kubernetes提供的kubectl工具（或者API编程调用）执行增、删、改、查等操作并将其保存在etcd中持久化存储。从这个角度来看，Kubernetes其实是一个高度自动化的资源控制系统，它通过跟踪对比etcd库里保存的“资源期望状态”与当前环境中的“实际资源状态”的差异来实现自动控制和自动纠错的高级功能
- 属性apiVersion,Kubernetes大部分常见的核心资源对象都归属于v1这个核,1.9版本之后引入了apps/v1这个正式的扩展API组，正式淘汰（deprecated）了extensions/v1beta1、apps/v1beta1、apps/v1beta2这三个API组。
- Kubernetes为每个资源对象都增加了类似数据库表里备注字段的通用属性Annotations
- Kubernetes API Server（kube-apiserver）：提供了HTTP Rest接口的关键服务进程，是Kubernetes里所有资源的增、删、改、查等操作的唯一入口，也是集群控制的入口
- Kubernetes Controller Manager（kube-controller-manager）：Kubernetes里所有资源对象的自动化控制中心，
-  Scheduler（kube-scheduler）：负责资源调度（Pod调度）的进程
- kubelet：负责Pod对应的容器的创建、启停等任务，同时与Master密切协作，实现集群管理的基本功能。
- kube-proxy：实现Kubernetes Service的通信与负负载均衡机制的重要组件。
- Docker Engine（docker）：Docker引擎，负责本机的容器创建和管理工作
- Kubernetes将集群中的机器划分为一个Master和一些Node。在Master上运行着集群管理相关的一组进程kube-apiserver、kube-controller-manager和kubescheduler，这些进程实现了整个集群的资源管理、Pod调度、弹性伸缩、安全控制、系统监控和纠错等管理功能，并且都是自动完成的。
## Master
Kubernetes里的Master指的是集群控制节点，在每个Kubernetes集群里都需要有一个Master来负责整个集群的管理和控制，基本上Kubernetes的所有控制命令都发给它，它负责具体的执行过程

## Node
- Node是Kubernetes集群中的工作负载节点，每个Node都会被Master分配一些工作负载（Docker容器），当某个Node宕机时，其上的工作负载会被Master自动转移到其他节点上。
- 在默认情况下kubelet会向Master注册自己，这也是Kubernetes推荐的Node管理方式。一旦Node被纳入集群管理范围，kubelet进程就会定时向Master汇报自身的情报，例如操作系统、Docker版本、机器的CPU和内存情况，以及当前有哪些Pod在运行等，这样Master就可以获知每个Node的资源使用情况，并实现高效均衡的资源调度策略。而某个Node在超过指定时间不上报信息时，会被Master判定为“失联”，Node的状态被标记为不可用（Not Ready），随后Master会触发“工作负载大转移”的自动流程。
- Node作为集群中的工作节点，运行真正的应用程序，在Node上Kubernetes管理的最小运行单元是Pod。在Node上运行着Kubernetes的kubelet、kube-proxy服务进程，这些服务进程负责Pod的创建、启动、监控、重启、销毁，以及实现软件模式的负载均衡器。

## Pod
- 每个Pod都有一个特殊的被称为“根容器”的Pause容器。Pause容器对应的镜像属于Kubernetes平台的一部分，除了Pause容器，每个Pod还包含一个或多个紧密相关的用户业务容器。
- Pod里的多个业务容器共享Pause容器的IP，共享Pause容器挂接的Volume，这样既简化了密切关联的业务容器之间的通信问问题，也很好地解决了它们之间的文件共享问题。
- Kubernetes为每个Pod都分配了唯一的IP地址，称之为Pod IP，一个Pod里的多个容器共享Pod IP地址。
- 在Kubernetes里，一个Pod里的容器与另外主机上的Pod容器能够直接通信。
- 在默认情况下，当Pod里的某个容器停止时，Kubernetes会自动检测到这个问题并且重新启动这个Pod（重启Pod里的所有容器），如果Pod所在的Node宕机，就会将这个Node上的所有Pod重新调度到其他节点上
- Pod的IP加上这里的容器端口（containerPort），组成了一个新的概念—Endpoint，它代表此Pod里的一个服务进程的对外通信地址
- Pod Volume是被定义在Pod上，然后被各个容器挂载到自己的文件系统中的。
- 当我们发现某个Pod迟迟无法创建时，可以用kubectl describe pod xxxx来查看它的描述信息，以定位问题的成
- 每个Pod都可以对其能使用的服务器上的计算资源设置限额，当前可以设置限额的计算资源有CPU与Memory两种
    - CPU的资源单位为CPU（Core）的数量，是一个绝对值而非相对值，在Kubernetes里通常以千分之一的CPU配额为最小单位，用m来表示。通常一个容器的CPU配额被定义为100～300m，即占用0.1～0.3个CPU。由于CPU配额是一个绝对值，所以无论在拥有一个Core的机器上，还是在拥有48个Core的机器上，100m这个配额所代表的CPU的使用量都是一样的。
    - Memory配额也是一个绝对值，它的单位是内存字节数。
    - Requests：该资源的最小申请量，系统必须满足要求。
    - Limits：该资源最大允许使用的量，不能被突破，当容器试图使用超过这个量的资源时，可能会被Kubernetes“杀掉”并重启。

- Pod运行在一个被称为节点（Node）的环境中，这个节点既可以是物理机，也可以是私有云或者公有云中的一个虚拟机，通常在一个节点上运行几百个Pod；
- 在每个Pod中都运行着一个特殊的被称为Pause的容器，其他容器则为业务容器，这些业务容器共享Pause容器的网络栈和Volume挂载卷，因此它们之间的通信和数据交换更为高效，在设计时可以充分利用这一特性将一组密切相关的服务进程放入同一个Pod中；

下面这段定义表明MySQL容器申请最少0.25个CPU及64MiB内存，在运行过程中MySQL容器所能使用的资源配额为0.5个CPU及128MiB内存

```
apiVersion: v1
kind: Pod
metadata:
  name: myweb
  labels:
    app: myweb
spec:
  containers:
  - name: myweb
    image: kubeguide/tomcat-app:v1
    ports:
    - containerPort: 8080
    env:
    - name: MYSQL_SERVICE_HOST
      value: 'mysql'
    - name: MYSQL_SERVICE_PORT
      value: '3306'
    resources：
        request:
            memory:"64Mi"
            cpu:"250m"
        limits：
        memory:"128Mi"
        cpu:"500m"
```

## Label
- 可以通过多个Label Selector表达式的组合实现复杂的条件选择，多个表达式之间用“，”进行分隔即可，几个条件之间是“AND”的关系，即同时满足多个条件
- 表达式
  - name=redis-slave
  - env!=production
  - name in（redis-master, redis-slave）
  - name not in（php-frontend）

- matchLabels用于定义一组Label，与直接写在Selector中的作用相同；matchExpressions用于定义一组基于集合的筛选条件，可用的条件运算符包括In、NotIn、Exists和DoesNotExist。
- 如果同时设置了matchLabels和matchExpressions，则两组条件为AND关系

## RC
- 删除RC并不会影响通过该RC已创建好的Pod。为了删除所有Pod，可以设置replicas的值为0，然后更新该RC。
- kubectl提供了stop和delete命令来一次性删除RC和RC控制的全部Pod。
- 在运行时，我们可以通过修改RC的副本数量，来实现Pod的动态缩放（Scaling），也可以通过执行kubectl scale命令来一键完成
- 当前很少单独使用Replica Set，它主要被Deployment这个更高层的资源对象所使用，从而形成一整套Pod创建、删除、更新的编排机制。
- Kubernetes会通过在RC中定义的Label筛选出对应的Pod实例并实时监控其状态和数量，如果实例数量少于定义的副本数量，则会根据在RC中定义的Pod模板创建一个新的Pod，然后将此Pod调度到合适的Node上启动运行，直到Pod实例的数量达到预定目标

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
```
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: app-demo
        tier: frontend
    spec:
      containers:
      - name: tomcat-demo
        image: tomcat
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
```

## Service
- 一个Service通常由多个相关的服务进程提供服务，每个服务进程都有一个独立的Endpoint（IP+Port）访问点，但Kubernetes能够让我们通过Service（虚拟Cluster IP +Service Port）连接到指定的Service。
	- 有了Kubernetes内建的透明负载均衡和故障恢复机制，不管后端有多少服务进程，也不管某个服务进程是否由于发生故障而被重新部署到其他机器，都不会影响对服务的正常调用
- Kubernetes给每个Pod都贴上一个标签（Label），结合Service的标签选择器的选择，巧妙解决了Service与Pod的关联问题。
- 通常，Cluster IP是在Service创建后由Kubernetes系统自动分配的，其他Pod无法预先知道某个Service的Cluster IP地址，因此需要一个服务发现机制来找到这个服务。
    - 方式一，Kubernetes巧妙地使用了Linux环境变量（Environment Variable）来解决这个问题
    - 方式二，每个Kubernetes中的Service都有唯一的Cluster IP及唯一的名称，而名称是由开发者自己定义的，部署时也没必要改变，所以完全可以被固定在配置中
        - Kubernetes通过Add-On增值包引入了DNS系统，把服务名作为DNS域名，这样程序就可以直接使用服务名来建立通信连接了
- NodePort的实现方式是在Kubernetes集群里的每个Node上都为需要外部访问的Service开启一个对应的TCP监听端口，外部系统只要用任意一个Node的IP地址+具体的NodePort端口号即可访问此服务，在任意Node上运行netstat命令，就可以看到有NodePort端口被监听
- 下例，type=NodePort和nodePort=30001的两个属性表明此Service开启了NodePort方式的外网访问模式。在Kubernetes集群之外，比如在本机的浏览器里，可以通过30001这个端口访问myweb（对应到8080的虚端口上）。
```
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
spec:
 type: NodePort
 ports:
  - port: 8080
    nodePort: 31002
 selector:
    tier: frontend
```
- 既然每个Pod都会被分配一个单独的IP地址，而且每个Pod都提供了一个独立的Endpoint（Pod IP+ContainerPort）以被客户端访问，现在多个Pod副本组成了一个集群来提供服务，那么客户端如何来访问它们？一般的做法是部署一个负载均衡器（软件或硬件），为这组Pod开启一个对外的服务端口如8000端口，并且将这些Pod的Endpoint列表加入8000端口的转发列表，客户端就可以通过负载均衡器的对外IP地址+服务端口来访问此服务。客户端的请求最后会被转发到哪个Pod，由负载均衡器的算法所决定。
	- 运行在每个Node上的kube-proxy进程其实就是一个智能的软件负载均衡器，负责把对Service的请求转发到后端的某个Pod实例上，并在内部实现服务的负载均衡
  - 服务发现这个棘手的问题在Kubernetes的架构里也得以轻松解决：只要用Service的Name与Service的Cluster IP地址做一个DNS域名映射即可完美解决问题
- 几种IP:
  - Node IP：Node的IP地址,在Kubernetes集群之外的节点访问Kubernetes集群之内的某个节点或者TCP/IP服务时，都必须通过Node IP通信。
  - Pod IP：Pod的IP地址,Pod IP是每个Pod的IP地址，它是Docker Engine根据docker0网桥的IP地址段进行分配的，通常是一个虚拟的二层网络,Kubernetes里一个Pod里的容器访问另外一个Pod里的容器时，就是通过Pod IP所在的虚拟二层网络进行通信的，而真实的TCP/IP流量是通过Node IP所在的物理网卡流出的。
  - Cluster IP：Service的IP地址,Service的Cluster IP，它也是一种虚拟的IP，但更像一个“伪造”的IP网络,Service的Cluster IP属于Kubernetes集群内部的地址，无法在集群外部直接使用这个地址
    - Cluster IP无法被Ping，因为没有一个“实体网络对象”来响应。
    - Cluster IP只能结合Service Port组成一个具体的通信端口，单独的Cluster IP不具备TCP/IP通信的基础，并且它们属于Kubernetes集群这样一个封闭的空间


```
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
spec:
  ports:
  - port: 8080
   name: service-port
  - port: 8005
   name: shutdown-port
  selector:
    tier: frontend
```
- 在spec.ports的定义中，targetPort属性用来确定提供该服务的容器所暴露（EXPOSE）的端口号，即具体业务进程在容器内的targetPort上提供TCP/IP接入；port属性则定义了Service的虚端口。
- 运行下面的命令即可看到tomct-service被分配的Cluster IP及更多的信息：
```
```


## Horizontal Pod Autoscaler
- 通过追踪分析指定RC控制的所有目标Pod的负载变化情况，来确定是否需要有针对性地调整目标Pod的副本数量，这是HPA的实现原理。
- CPUUtilizationPercentage是一个算术平均值，即目标Pod所有副本自身的CPU利用率的平均值。
    - 一个Pod自身的CPU利用率是该Pod当前CPU的使用量除以它的Pod Request的值，比如定义一个Pod的Pod Request为0.4，而当前Pod的CPU使用量为0.2，则它的CPU使用率为50%，这样就可以算出一个RC控制的所有Pod副本的CPU利用率的算术平均值。
    - 如果某一时刻CPUUtilizationPercentage的值超过80%，则意味着当前Pod副本数量很可能不足以支撑接下来更多的请求，需要进行动态扩容，而在请求高峰时段过去后，Pod的CPU利用率又会降下来，此时对应的Pod副本数应该自动减少到一个合理的水平。如果目标Pod没有定义Pod Request的值，则无法使用CPUUtilizationPercentage实现Pod横向自动扩容。
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    kind: Deployment
    name: php-apache
  targetCPUUtilizationPercentage: 90
```

## StatefulSet
- StatefulSet里的每个Pod都有稳定、唯一的网络标识，可以用来发现集群内的其他成员。假设StatefulSet的名称为kafka，那么第1个Pod叫kafka-0，第2个叫kafka-1，以此类推。
- StatefulSet里的Pod采用稳定的持久化存储卷，通过PV或PVC来实现，删除Pod时默认不会删除与StatefulSet相关的存储卷（为了保证数据的安全）。
- StatefulSet在Headless Service的基础上又为StatefulSet控制的每个Pod实例都创建了一个DNS域名，这个域名的格式为：
```
${podname}.${head less service name}
```
  - 比如一个3节点的Kafka的StatefulSet集群对应的Headless Service的名称为kafka，StatefulSet的名称为kafka，则StatefulSet里的3个Pod的DNS名称分别为kafka-0.kafka、kafka-1.kafka、kafka-3.kafka，这些DNS名称可以直接在集群的配置文件中固定下来。

## Job
- Job所控制的Pod副本是短暂运行的，可以将其视为一组Docker容器，其中的每个Docker容器都仅仅运行一次。当Job控制的所有Pod副本都运行结束时，对应的Job也就结束了

## Volume
- 在大多数情况下，我们先在Pod上声明一个Volume，然后在容器里引用该Volume并挂载（Mount）到容器里的某个目录上。
- Kubernetes中的Volume被定义在Pod上，然后被一个Pod里的多个容器挂载到具体的文件目录下；
- Kubernetes中的Volume与Pod的生命周期相同，但与容器的生命周期不相关，当容器终止或者重启时，Volume中的数据也不会丢失。
- Kubernetes支持多种类型的Volume，例如GlusterFS、Ceph等先进的分布式文件系统。
- 一个emptyDir Volume是在Pod分配到Node时创建的。从它的名称就可以看出，它的初始内容为空，并且无须指定宿主机上对应的目录文件，因为这是Kubernetes自动分配的一个目录，当Pod从Node上移除时，emptyDir中的数据也会被永久删除。
- hostPath为在Pod上挂载宿主机上的文件或目录

## Persistent Volume
- PV可以被理解成Kubernetes集群中的某个网络存储对应的一块存储
- PV只能是网络存储，不属于任何Node，但可以在每个Node上访问。
- PV并不是被定义在Pod上的，而是独立于Pod之外定义的。
```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /somepath
    server: 172.17.0.2

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
```

## Namespace
- Namespace通过将集群内部的资源对象“分配”到不同的Namespace中，形成逻辑上分组的不同项目、小组或用户组，便于不同的分组在共享使用整个集群的资源的同时还能被分别管理。
- 如果不加参数，则kubectl get命令将仅显示属于default命名空间的资源对象。
- 可以在kubectl命令中加入--namespace参数来查看某个命名空间中的对象
- 当给每个租户创建一个Namespace来实现多租户的资源隔离时，还能结合Kubernetes的资源配额管理，限定不同租户能占用的资源，例如CPU使用量、内存使用量等


## ConfigMap
- 这些配置项可以作为Map表中的一个项，整个Map的数据可以被持久化存储在Kubernetes的Etcd数据库中，然后提供API以方便Kubernetes相关组件或客户应用CRUD操作这些数据，专门用来保存配置参数的Map就是Kubernetes ConfigMap资源对象。
- 接下来，Kubernetes提供了一种内建机制，将存储在etcd中的ConfigMap通过Volume映射的方式变成目标Pod内的配置文件，不管目标Pod被调度到哪台服务器上，都会完成自动映射。
