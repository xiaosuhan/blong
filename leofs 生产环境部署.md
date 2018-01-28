# leofs 生产环境部署


## leofs 生产环境规划    
    [manger]         主机名
    10.39.20.29      leofs-M0
    10.39.20.25      leofs-M1
    [GW]
    10.39.10.14      leofs-GW01
    10.39.10.15      leofs-GW02  
    [storage]
    10.39.10.26      leofs-S01
    10.39.10.27      leofs-S02 
    10.39.10.28      leofs-S03
    10.39.10.24      leofs-S04  
 
## 环境部署

     1. 安装组件
     ansible -i hosts leofs  -m shell -a "yum install gcc gcc-c++ glibc-devel make ncurses-devel openssl-devel autoconf  libuuid-devel cmake check check-devel -y" 
     2. 安装leofs 
     ansible -i hosts leofs -m copy -a "src=leofs-1.3.7-1.erl-19.3.el7.x86_64.rpm   dest=/tmp"  -k 
     3. ansible -i hosts leofs -m shell -a "cd /tmp; yum install *.rpm -y " 

     4. storage xfs 安装插件  
        yum --enablerepo=centosplus install kmod-xfs xfsprogs xfsprogs-devel
## leofs配置
 
    1. leofs-M0 配置
       manager.partner = manager_1@10.39.20.25
       system.dc_id = Enn
       system.cluster_id = Ennleofs
       consistency.num_of_replicas = 3
       consistency.write = 3
       consistency.read = 3
       consistency.delete = 3
       nodename = manager_0@10.39.20.29     
       M0 启动
       /usr/local/leofs/1.3.7/leo_manager_0/bin/leo_manager start    
    2. leofs-M1 配置
       manager.partner = manager_0@10.39.20.29
       nodename = manager_1@10.39.20.25
    3. storage 配置
       managers = [manager_0@10.39.20.29, manager_1@10.39.20.25]
       obj_containers.path = [/data/leofs] 
       nodename = storage_01@10.39.20.26
       
       创建目录 
       mkdir /data/leofs -p  
       chown -R leofs:leofs /data/leofs  
    
    
    4. 启动leofs_adm start   
       leofs-adm start 
       Generating RING...
       Generated RING
       OK  33% - storage_03@10.39.20.28
       OK  67% - storage_02@10.39.20.27
       OK 100% - storage_01@10.39.20.26
       OK
       
       查看状态
       leofs-adm status  
       [System Confiuration]
        -----------------------------------+----------
        Item                              | Value    
        -----------------------------------+----------
        Basic/Consistency level
        -----------------------------------+----------
                    system version | 1.3.7
                        cluster Id | enn_leofs
                             DC Id | enn
                    Total replicas | 3
          number of successes of R | 3
          number of successes of W | 3
          number of successes of D | 3
    number of rack-awareness replicas | 0
                         ring size | 2^128
    -----------------------------------+----------
    Multi DC replication settings
    -----------------------------------+----------
    [mdcr] max number of joinable DCs | 2
    [mdcr] total replicas per a DC    | 1
    [mdcr] number of successes of R   | 1
    [mdcr] number of successes of W   | 1
    [mdcr] number of successes of D   | 1
    -----------------------------------+----------
    Manager RING hash
    -----------------------------------+----------
                 current ring-hash | 3fc7f82f
                previous ring-hash | 3fc7f82f
    -----------------------------------+----------

    [State of Node(s)]
 
     type  |            node             |    state     |  current ring  |   prev ring    |          updated at         
    -------+-----------------------------+--------------+----------------+----------------+----------------------------
      S    | storage_01@10.39.20.26      | running      | 3fc7f82f       | 3fc7f82f       | 2017-10-12 09:51:29 +0800
      S    | storage_02@10.39.20.27      | running      | 3fc7f82f       | 3fc7f82f       | 2017-10-12 09:51:29 +0800
      S    | storage_03@10.39.20.28      | running      | 3fc7f82f       | 3fc7f82f       | 2017-10-12 09:51:29 +0800
      S    | storage_04@10.39.20.24      | attached     |                |                | 2017-10-12 09:54:54 +0800
      G    | gateway_01@10.39.20.14      | running      | 3fc7f82f       | 3fc7f82f       | 2017-10-12 09:56:55 +0800
      G    | gateway_02@10.39.20.15      | running      | 3fc7f82f       | 3fc7f82f       | 2017-10-12 09:58:06 +0800

## 测试leofs  
    
    
    
    