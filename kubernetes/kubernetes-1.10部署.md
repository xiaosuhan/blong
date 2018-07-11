### k8s 1.7.6 部署  
     
     环境规划                
     10.39.10.160   master  节点   4c8g         
     10.39.10.159   node    节点   4c8g
     10.39.10.156   node    节点   4c8g
     10.39.1.43     编译k8s，安装golang    


### 环境安装 
    1. docker install
       所有节点都按照docker  
       yum-config-manager     --add-repo     https://docs.docker.com/v1.13/engine/installation/linux/repo_files/centos/docker.repo 
       yum -y install  docker-engine-selinux-1.13.1-1.el7.centos docker-engine-1.13.1-1.el7.centos 
         
    2. 安装golang
        10.39.1.43上安装golang 
        wget https://dl.google.com/go/go1.10.3.linux-amd64.tar.gz 
        tar -C /usr/local/ -xzf go1.10.3.linux-amd64.tar.gz  
        vim /etc/profile 
        export PATH=$PATH:/usr/local/go/bin
        
        设置GOROOT  GOPATH 
        export GOROOT=$HOME/go1.
        export PATH=$PATH:$GOROOT/bin 
        cd $GOPATH  &&  mkdir  {pkg,src,bin,lib}
        go  version 
   
    3. kubernetes 编译      
     
       下载kubernetes 源码  
       cd $GOPATH/src 
       git clone https://github.com/kubernetes/kubernetes.git 
       git tag 
       git checkout 
       git branch -v
       
       
       
       

     
     
     
     
     
    