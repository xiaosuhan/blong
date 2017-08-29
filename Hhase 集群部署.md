### Hhase 集群部署
     使用的软件 
     hadoop-2.7.4
     hbase-1.2.6
     jdk-8u144
     zookeeper-3.4.10
     Hbase 自带的有zookeeper，在这里使用自己部署的zookeeper  
#### zookeeper 集群部署  
    安装jdk 
    下载zookeeper 程序 
    修改zoo.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataLogDir=/zookeeper/logs
    dataDir=/zookeeper/data
    clientPort=2181
    server.1= 10.39.6.178:2888:3888
    server.2= 10.39.6.179:2888:3888
    server.3= 10.39.6.180:2888:3888
    
    添加myid，这里的myid 对应的server.n 一一对应。 
    这里的server.1 所以node 1节点myid=1  
    echo "1" /zookeeper/data/myid
    
    创建所需要的目录
    添加环境变量
    vi /etc/profile 
    export ZOOKEEPER_HOME=/application/zookeeper-3.4.10
    export PATH=$PATH:$ZOOKEEPER_HOME/bin  
    
 启动
     
    将node 1 的配置全部打包拷贝到其他节点上，启动zookeeper 就行了
    启动有错误可以使用zkServer.sh start-foreground 来追踪错误 
    
 角色
 
    zkServer.sh status 会显示zookeeper 状态
    Mode: leader 
    
    这里的Mode: leader 和follower 
    一个集群中只有leader 
    leader 领导者，用于负责进行投票的发起决议，更新系统状态 
    follower 跟随者 用于接受客户端请求并想客户端返回结果，在选主过程中参与投票 
    
配置参数详解
     
    tickTime 这个时间是作为zookeeper服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是说每个tickTime 时间就会发送一个心跳。 
    initLimit 这个配置项是用来配置zookeeper接受客户端初始化连接时最长能忍受多少个心跳时间间隔数。
    当已经超过10个心跳的时间(tickTime) 长度后zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败，总的时间长度就是10*2000=20秒 
    syncLimit 这个配置项标识leader 与follower 之间发送消息，请求和应答时间长度，最长不能超过多少个tickTime 的长度，总的时间长度是5*2000=10秒
    dataDir 保存数据目录
    clientPort 端口，这个端口是客户端连接zookeeper服务器端口，zookeeper 会监听这个端口接受客户端访问请求
    server.n=B:C:D 的n是一个数字，表示这个是第几号服务器，B是这个服务器的IP地址，C第一个端口用来集群成员的信息交换，表示这个服务器与集群中的leader 服务器交换信息的端口，D是leader 挂掉时专门用来进行选举leader 所用的端口 
    
连接zookeeper集群
 zkCli.sh  -server 10.39.6.178:2181    
    
    
#### Hadoop 安装 
  
  下载地址 [http://apache.fayea.com/hadoop/common/stable/hadoop-2.7.4.tar.gz]()
  
    hbase01 到hbase02   hbase03 需要使用ssh无密钥登录。 
    
#### hadoop 配置文件
  
| 配置文件       |  配置对象         | 主要内容       |
|:------------- |:---------------:| -------------:|
| core-site.xml | 集群全局参数      |用户定义系统级别的参数,如HDFS URL Hadoop临时目录|
| hdfs-site.xml |  HDFS 参数       |如名称节点和数据节点存放位置，文件副本的个数，文件读取权限 |
| mapred-site.xml| Mapreduce参数   |包括JobHistry Server 和应用程序参数两部分，如reduce 任务的默认个数，任务所能够使用内存的默认上下限|
| yarn-site.xml |集群资源管理系统参数|包括ResourceManager，NodeManager 的通信端口，web 监控端口等|

    
    集群配置
##### vi /application/hadoop-2.7.4/etc/hadoop/hadoop-env.sh 
    
     export  JAVA_HOME="/usr/java/jdk1.8.0_144" 
    （rpm 安装的jdk 存储位置）
 
##### vi /application/hadoop-2.7.4/etc/hadoop/core-site.xml
   
      <configuration>
     <property>
     <name>fs.defaultFS</name>
     <value>hdfs://hbase01:9000</value>
        <description>The name of the default file system</description>
      </property>

     <property>
        <name>hadoop.tmp.dir</name>
        <value>/zookeeper/hadoopdata/tmp</value>
        <description>A base for other temporary directories</description>
    </property>

     <property>
         <name>hadoop.native.lib</name>
         <value>true</value>
         <description>Should native hadoop libraries, if present, be used.</description>
    </property>
    </configuration>
    
##### vi /application/hadoop-2.7.4/etc/hadoop/hdfs-site.xml
    
    <configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>

    <property>
          <name>dfs.namenode.name.dir</name>
          <value>/zookeeper/hadoopdata/dfs/name</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/zookeeper/hadoopdata/dfs/data</value>
     </property>

    </configuration>
   
##### vi /application/hadoop-2.7.4/etc/hadoop/mapred-site.xml             
    
       <configuration>
     <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
     </property>
    </configuration>     
    
##### vi /application/hadoop-2.7.4/etc/hadoop/yarn-site.xml 
       
        <configuration>
         <property>
           <name>yarn.resourcemanager.hostname</name>
           <value>hbase01</value>
        </property>

       <property>
            <name>yarn.nodemanager.aux-services</name>
             <value>mapreduce_shuffle</value>
        </property>


     </configuration>
       
##### vi /application/hadoop-2.7.4/etc/hadoop/slaves   
         hbase02
         hbase03
   
  
#### 将所有的配置COPY 到hbase02 hbase03 
     
     
####  格式化HDFS存储
   
    1. 在namenode 上执行
       进入到hadoop 目录
       ./bin/hadoop namenode -format   
    2. 在datanode 
       ./bin/hadoop datanode -format 
       
#### 启动Hadoop
      
      1. 启动HDFS 
        ./sbin/start-dfs.sh
        ./sbin/stop-dfs.sh
      2. 启动Yarn
       ./sbin/start-yarn.sh 
       ./sbin/stop-yarn.sh
      3.启动MapReduce JobHistory Server
       ./sbin/mr-jobhistory-daemon.sh  start historyserver   
    
      jps 查看进程
      jps
      12016 ResourceManager
      11616 NameNode
      11828 SecondaryNameNode
      12317 JobHistoryServer
      31453 Jps

#### web 访问端口      
      NameNode    50070
      ResourceManager 8088
      MapReduce JobHistory Server 19888
       
### Hbase 安装
     
     hbase 配置文件修改
     vi conf/hbase-env.sh  
        export JAVA_HOME=/usr/java/jdk1.8.0_144
        export HBASE_MANAGES_ZK=false      
     
     vi conf/hbase-site.xml
        <configuration>
         <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://hbase01:9000/hbase</value>
        </property>
        <property>
           <name>hbase.zookeeper.quorum</name>
           <value>hbase01,hbase02,hbase03</value>
       </property>

       <property>
         <name>hbase.zookeeper.property.dataDir</name>
         <value>/zookeeper/data</value>
       </property>
    </configuration> 
   
    vi conf/regionservers 
        hbase02
        hbase03
     
     将上述配置同步到其他节点 
 
#### hbase 启动
      ./bin/start-hbase.sh 
      
      查看Hbase 的状态
      jps 
      12016 ResourceManager
      11616 NameNode
      12546 HMaster
      10403 QuorumPeerMain
      11828 SecondaryNameNode
      21225 Jps
      12317 JobHistoryServer
  
##### 进入hbase shell，使用命令查看hbase 状态
     
     ./bin/hbase shell 
     SLF4J: Class path contains multiple SLF4J bindings.
     SLF4J: Found binding in [jar:file:/application/hbase-1.2.6/lib/slf4j-l 
     HBase Shell; enter 'help<RETURN>' for list of supported commands.
     Type "exit<RETURN>" to leave the HBase Shell
     Version 1.2.6, rUnknown, Mon May 29 02:25:32 CDT 2017

     hbase(main):001:0> status 
    1 active master, 0 backup masters, 2 servers, 0 dead, 1.0000 average load

     hbase(main):002:0> 

   
    Hbase web ui 端口为16010 
    
    
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
      
  
    
    
    
    
      