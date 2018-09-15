## elasticsearch 集群部署

### 环境要求
     JDK 1.8.0以上
   [curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.3.tar.gz
]()
           
        

-Xms16g
-Xmx16g




### 模块 

  参考 [https://www.elastic.co/guide/en/elasticsearch/reference/5.4/modules-node.html]()
    
    es中的节点介绍 
    1. master-eligible  node (主节点)
     一个节点node.master 设置为 true(默认),这个节点才有资格参加master 节点选举，来控制集群 
    2. Data node (数据节点)
     一个节点，node.data 设置为 true(默认), 数据节点保存数据和执行数据相关的操作，如增删改查，搜索和聚合。 
    
    4. tribe  node(部落节点)
       部落节点是一种特殊类型的仅协调节点，可以连接到多个集群并在所有连接的集群中执行搜索和其他操作的，       
       
    5. inget node
    
    3. 架构
  ![](image/es.png)
    
    
    
    
### elasticsearch 优化
     
    配置线程池 
    int ((核心数  3 )/2) +  1  
    
    CPU 是16核数，线程池的大小 = (16x3)/2+1 = 25 
    同时满足CPU核数为16，16和25 取最小值，应该选择16 
    
    默认队列大小是1000 
    
    ## Threadpool Settings
    ### Search pool
      thread_pool.search.size: 16
      thread_pool.search.queue_size: 2000

    #Bulk pool
      thread_pool.bulk.size: 16
      thread_pool.bulk.queue_size: 1000
    
    
    # Index pool
      thread_pool.index.size: 16
      thread_pool.index.queue_size: 1000
     
     ### 配置堆内存 
        为内存的1/2  
     ### 文件描述符
     vim   /etc/security/limits.conf  
     
         * soft nofile 65536
         * hard nofile 65536
     
     es 对各种文件混合使用了NioFs(非阻塞文件系统)和MMapFs(内存映射文件系统)，需要配置最大映射数量，以便有足够的虚拟内存可用于mmapped 文件，
     
       sysctl -w vm.max_map_count=262144    
    
    
     
    
    
    
    
    
    
    
    
    
    
      
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
      
     