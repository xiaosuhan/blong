## calico 学习笔记(一)
    
    
   [https://docs.projectcalico.org/v3.2/reference/architecture/]()
      
#### calico 由以下组件组成:
       felix: 部署在每个节点的calico 代理
       plugin: 云平台的插件(OpenStack,kubernetes 主要使用cni plugin)使用插件绑定calico，管理calico 网络
       etcd: 存储calico的数据
       BIRD: 分发路由信息的BGP客户端
       BGP Route Reflector (BIRD): 可选的BGP路由反射器,适用于更高规模的网络
       
#### 组件介绍
     
     felix 是一个守护进程，部署在每个节点上运行，监听状态更新，负责路由规划，
     
     
               
    
