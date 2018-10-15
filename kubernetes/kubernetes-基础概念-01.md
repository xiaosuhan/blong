### kubernetes 基础概念-学习笔记
 
    
    
#### 简介
     kubernetes 是Google团队发起的一个开源项目，目标是管理跨多个主机的容器，用于自动部署、扩展和管理容器化得应用程序。 
     
#### Pod 
     Pod是一组紧密关联的容器集合，它们共享PID、IPC、Network和UTS namespace,是kubernetes 调度的基本单位。Pod的设计理念是支持多个容器在一个Pod中共享网络和文件系统，可以通过进程间通信和文件共享完成服务
     
     在kubernetes中，所有的对象都使用mainfest(yaml/json)来定义，可以定义为nginx.yaml 
       	apiVersion: v1 
			kind: Pod
			metadata:
			  name: nginx 
			  labels:
			  app: nginx 
			
			spec: 
			  containers: 
			  -name: nginx 
			  -image: nginx 
			  ports:
			   - containerPort: 80 
      
#### label 
     label 是识别kubernetes 对象的标签，以key/value 的方式附加到对象上(key最长不能超过63字节，value可以为空,也可以是不超过253字节的字符串)
     label 不提供唯一性，并且实际上经常是很多对象(Pods)都使用相同label来标志具体应用。
     label 定义好后其他对象可以使用Label Selector来选择一组相同label的对象(比如Service用label来选择一组Pod).label selector支持以下几种方式
     > 等式, 如app=nginx和 env!=production 
     > 集合, 如env in(production,qa)
     > 多个label(它们之间是AND关系),如app=nginx,env=test 
     
#### namespace 
     
     namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的pods,services和deployments 等都是属于某一个namespace的(默认是default),而Node，PersistentVolumes等则不属于任何namespace 
     
       
#### Deployment 
   
    Deployment 确保任意时间都有指定数量的Pod “副本”在运行，如果为某个Pod创建了Deployment 并且指定3个副本，它会创建3个Pod，并且持续监控它们.如果某个Pod不响应，那么Deployment 会替换它，确保总数为3 
    如果之前不响应的Pod恢复了，现在就有4个Pod了，那么Deployment 会将其中一个终止保持总数为3，如果在运行中将副本总数改为5，Deployment会立刻启动2个新的Pod，保证总数为5，Deployment 还支持回滚和滚动升级
    
    当创建Deployment时需要指定两个东西:
    > Pod 模板: 用来创建Pod副本的模板
    > label标签: Deployment 需要监控的Pod的执照
    
#### Service 
    
     Service 是应用服务的抽象,通过labels为应用提供负载均衡和服务发现.匹配labels和Pod IP 和端口列表组成endpoints,由kube-proxy负责将服务IP 负载均衡到这些endpoints上。
     每个Service 都会自动分配一个cluster IP(仅在集群内部可以访问的虚拟地址)和DNS名，其他容器可以通过该地址或DNS来访问服务,而不需要了解后端容器的运行。 
     
     
#### kubernetes 组件
   
  Master: 
  
    Master节点是kubernetes集群的控制节点，负责整个集群的管理和控制。Master 节点包含以下组件：
     > master: API Server, Scheduler, Controller-Manager 
     组件的功能:
     kube-apiserver: 集群控制的入口，提供HTTP REST 服务
     kube-controller-manager: kubernetes 集群中所有资源对象自动化控制中心
     kube-schedule: 负责Pod 调度
      
   Node
   
    Node节点是kubernetes集群中的工作节点，node上的工作负载由master节点分配，主要运行容器应用. 包含以下组件:
    > node: kubelet, docker 
         Pod, Label, Label Selector  
     .Label: key = value 
     .Label Selector: 
     
     Pod: 
      .自主Pod 
      .控制器管理的Pod 
        .ReplicationController  副本控制器
        .RelicaSet 
        .Deployment   无状态副本集
        .StatefulSet  有状态副本集
        .DaemonSet    
        .Job,Ctonjob 
        HPA 
        .自动伸缩
        
   Node各个组件的功能:  
   
     kubelet: 负载pod 的创建 启动 监控 重启 销毁等工作，同时与master节点写作，实现集群管理的基本功能。
     kube-proxy: 实现kubernetes Service 的通信和负载均衡
     Pod: Pod是kubernetes最基本的部署调度单元，每个pod可以由一个或多个业务容器和一个根容器(pause)组成，一个pod 表示某个应用的一个实例 
     ReplicaSet:是pod副本的抽象，用于解决pod的扩容和伸缩
     Deployment: Deployment 表示部署，在内部使用RelicaSet来实现，可以通过Deployment 来生成相应的RelicaSet完成pod 副本的创建
     Service: service 是kubernetes最重要的资源对象，kubernetes中的Service 对象可以对应微服务架构中的微服务。Service 定义了服务的访问入口，服务的调用者通过这个地址访问Service后端的Pod副本实现，Service 通过label Selector 同后端的pod 副本建立关系，Deployment 保证后端pod 副本的数量，也就是保证服务的伸缩性。 
     
     
   
   kubernetes核心组件
   
   ![](image/kubernetes-核心组件.png)
   
    kubernetes 主要由以下几个核心组件组成
      > etcd 保存了整个集群的状态, 就是一个数据库
      > apiserver 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现
      > controller manager 负责维护集群的状态，比如故障检测、自动扩展、滚动升级
      > kubelet 负责维护容器的声明周期，同时也负责Volume(CVI)和网络CNI的管理
      > Container runtime 负责镜像管理以及pod 和容器的真正运行(CRI)
      > kube-proxy 负责为service 提供cluster 内部的服务发现和负载均衡
      
      
      
  
  
        
        
          
        