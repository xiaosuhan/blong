#### kubernetes-代码编译之hyperkube方式安装集群(一)

  1. 批量修改主机名 
  
        使用ansible-playbook 批量修改主机名为kubernetes-* 
   
        - hosts : testall
		   remote_user : root
		   tasks :
		     - name : show hostname
		       shell : hostname
		     - name : show ip
		       command : ip a
		     - hostname : name=web{{ ansible_default_ipv4.address.split('.')[-1] }}   
		       					
2. 安装工具
        
        yum install docker  git -y 		       					
3. 启动docker  
       
        systemctl  restart  docker  
        
4. 获取代码       
       
        git clone https://github.com/kubernetes/kubernetes.git
        git checkout  v1.11.2
        
5. 编译代码  
       
        [root@devops kubernetes]# ./build/run.sh make
        
        使用此命令编译需要翻墙pull  kube-build:build-26cc471648-5-v1.10.3-1 镜像，如果编译的机器可以翻墙的话就好，如果不能需要找台机器把这个镜像pull 下来
        
       ................................................
		
		Env for linux/amd64: GOOS=linux GOARCH=amd64 GOROOT=/usr/local/go CGO_ENABLED= CC=
		+++ [0919 15:07:38] Placing binaries
		+++ [0919 15:08:24] Syncing out of container
		+++ [0919 15:08:24] Stopping any currently running rsyncd container
		+++ [0919 15:08:24] Starting rsyncd container
		+++ [0919 15:08:26] Running rsync
		+++ [0919 15:08:44] Stopping any currently running rsyncd container
		
		编译完成之后，kubernetes 目录下会有一个_output 目录，二进制生成如下
		[root@devops kubernetes]# ls -l _output/dockerized/bin/linux/amd64/
		总用量 2322712
		-rwxr-xr-x 1 root root  59296482 9月  19 15:07 apiextensions-apiserver
		-rwxr-xr-x 1 root root 138048399 9月  19 15:07 cloud-controller-manager
		-rwxr-xr-x 1 root root   7699847 9月  19 15:00 conversion-gen
		-rwxr-xr-x 1 root root   7695690 9月  19 15:00 deepcopy-gen
		-rwxr-xr-x 1 root root   7669238 9月  19 15:00 defaulter-gen
		-rwxr-xr-x 1 root root 209959792 9月  19 15:07 e2e_node.test
		-rwxr-xr-x 1 root root 173387288 9月  19 15:07 e2e.test
		-rwxr-xr-x 1 root root  54111706 9月  19 15:07 gendocs
		-rwxr-xr-x 1 root root 226069072 9月  19 15:08 genkubedocs
		-rwxr-xr-x 1 root root 232008616 9月  19 15:08 genman
		-rwxr-xr-x 1 root root   5481582 9月  19 15:08 genswaggertypedocs
		-rwxr-xr-x 1 root root  54048394 9月  19 15:07 genyaml
		-rwxr-xr-x 1 root root  10645252 9月  19 15:07 ginkgo
		-rwxr-xr-x 1 root root   2835466 9月  19 15:01 go-bindata
		-rwxr-xr-x 1 root root 227294224 9月  19 15:08 hyperkube
		-rwxr-xr-x 1 root root  57246810 9月  19 15:07 kubeadm
		-rwxr-xr-x 1 root root  57908446 9月  19 15:07 kube-aggregator
		-rwxr-xr-x 1 root root 185147806 9月  19 15:08 kube-apiserver
		-rwxr-xr-x 1 root root 153798833 9月  19 15:08 kube-controller-manager
		-rwxr-xr-x 1 root root  55241523 9月  19 15:08 kubectl
		-rwxr-xr-x 1 root root 162720976 9月  19 15:08 kubelet
		-rwxr-xr-x 1 root root 160050880 9月  19 15:07 kubemark
		-rwxr-xr-x 1 root root  51912166 9月  19 15:07 kube-proxy
		-rwxr-xr-x 1 root root  55471287 9月  19 15:08 kube-scheduler
		-rwxr-xr-x 1 root root   6694536 9月  19 15:08 linkcheck
		-rwxr-xr-x 1 root root   2330265 9月  19 15:07 mounter
		-rwxr-xr-x 1 root root  13630237 9月  19 15:01 openapi-gen 
		
		
6. 制作镜像
    进入到kubernetes 目录下 cluster/images/hyperkube/    
    执行命令进行镜像制作 make build VERSION=v1.11.2 ARCH=amd64 
       
        [root@devops kubernetes]# cd cluster/images/hyperkube/	     [root@devops hyperkube]# ll
			总用量 20
			-rw-r--r-- 1 root root 1161 9月  19 14:24 BUILD
			-rw-r--r-- 1 root root 1589 9月  19 14:24 Dockerfile
			-rw-r--r-- 1 root root 1808 9月  19 14:25 Makefile
			-rw-r--r-- 1 root root   90 9月  19 14:25 OWNERS
			-rw-r--r-- 1 root root 1164 9月  19 14:24 README.md
		   [root@devops hyperkube]# make build VERSION=v1.11.2 ARCH=amd64
		   备注，每个版本都回拉取一个基础镜像，v1.11.2版本需要拉取k8s.gcr.io/debian-hyperkube-base-amd64:0.10 这个镜像，需要翻墙拉取
		  如果将镜像拉取到本地了，默认Makefile 里面是强制拉取需要修改
		  docker build --pull -t ${REGISTRY}/hyperkube-${ARCH}:${VERSION} ${TEMP_DIR} 
		  修改为即可
		  docker build  -t ${REGISTRY}/hyperkube-${ARCH}:${VERSION} ${TEMP_DIR} 
		 
		   
		  编译制作镜像完成时会有以下image 生成
		  staging-k8s.gcr.io/hyperkube-amd64:v1.11.2 目标镜像
		  kube-build:build-26cc471648-5-v1.10.3-1  编译二进制使用的镜像
		  k8s.gcr.io/kube-cross:v1.10.3-1    编译需要的镜像
		  k8s.gcr.io/debian-hyperkube-base-amd64:0.10 制作目标镜像的基础镜像
		    
        
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
    		   
		   