### 代码编译(二)
     这部分主要完成etcd与 master 的安装 
    1. 系统环境
	     Linux kubernetes-153 3.10.0-327.4.5.el7.x86_64 #1 SMP Mon Jan 25 22:07:14 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux

		 CentOS Linux release 7.2.1511 (Core)
		  
    2. 机器规划 
       [master]
		10.39.10.157
		10.39.10.158
		10.39.10.153
		
		[node]
		10.39.10.182
		10.39.10.183
		
		[etcd]
		10.39.10.157
		10.39.10.158
		10.39.10.153 
      
  
### 安装详情     
    
     制作证书
     1. 创建CA证书配置
       mkdir ssl  
       cd ssl 
       vim  config.json  
         {
		  "signing": {
		    "default": {
		      "expiry": "87600h"
		    },
		    "profiles": {
		      "kubernetes": {
		        "usages": [
		            "signing",
		            "key encipherment",
		            "server auth",
		            "client auth"
		        ],
		        "expiry": "87600h"
		      }
		    }
		  }
		} 
     
      vim  csr.json 
        {
		  "CN": "kubernetes",
		  "key": {
		    "algo": "rsa",
		    "size": 2048
		  },
		  "names": [
		    {
		      "C": "CN",
		      "ST": "Beijing",
		      "L": "Beijing",
		      "O": "k8s",
		      "OU": "System"
		    }
		  ]
		 }
		
	   	生成CA 证书和私钥
	   	[root@devops ssl]# cfssl gencert -initca csr.json  | cfssljson -bare ca
	    [root@devops ssl]# ls -l
		总用量 20
		-rw-r--r-- 1 root root 1001 9月  20 14:35 ca.csr
		-rw------- 1 root root 1675 9月  20 14:35 ca-key.pem
		-rw-r--r-- 1 root root 1359 9月  20 14:35 ca.pem
		-rw-r--r-- 1 root root  292 9月  20 14:30 config.json
		-rw-r--r-- 1 root root  208 9月  20 14:33 csr.json
       
     2. 分发证书
       [root@devops k8s-1.11.0]# ansible -i hosts k8s -m shell -a "mkdir /etc/kubernetes/ssl  -p"
		 [WARNING]: Consider using file module with state=directory rather than running mkdir
		
		10.39.10.182 | SUCCESS | rc=0 >>
		
		
		10.39.10.158 | SUCCESS | rc=0 >>
		
		
		10.39.10.183 | SUCCESS | rc=0 >>
		
		
		10.39.10.153 | SUCCESS | rc=0 >>
		
		
		10.39.10.157 | SUCCESS | rc=0 >>

    3. 每台机器都安装docker
       [root@devops k8s-1.11.0]# ansible -i hosts k8s -m shell -a "yum  install docker -y"  
       [root@devops k8s-1.11.0]# ansible -i hosts k8s -m shell -a "systemctl restart  docker "
         
	4. 安装etcd 集群
	   
wget   [https://github.com/coreos/etcd/releases/download/v3.2.18/etcd-v3.2.18-linux-amd64.tar.gz]()   	
	    	
	 [root@devops k8s-1.11.0]# tar zxvf etcd-v3.2.18-linux-amd64.tar.gz
	 [root@devops k8s-1.11.0]# ansible -i hosts etcd -m copy -a "src=./etcd-v3.2.18-linux-amd64/etcdctl  dest=/usr/bin/"  -k
	 [root@devops k8s-1.11.0]# ansible -i hosts etcd -m copy -a "src=./etcd-v3.2.18-linux-amd64/etcd  dest=/usr/bin/"  -k
	 
	 
	创建etcd 证书
	vim  etcd-csr.json
	   	{
		  "CN": "etcd",
		  "hosts": [
		    "127.0.0.1",
		    "10.39.10.153",
		    "10.39.10.157",
		    "10.39.10.158"
		  ],
		  "key": {
		    "algo": "rsa",
		    "size": 2048
		  },
		  "names": [
		    {
		      "C": "CN",
		      "ST": "Beijing",
		      "L": "Beijing",
		      "O": "k8s",
		      "OU": "System"
		    }
		  ]
		}
			   	
	  生成etcd 密钥 
	  [root@devops ssl]# cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes etcd-csr.json  | cfssljson  -bare etcd


		[root@devops ssl]# ls -l
		总用量 36
		-rw-r--r-- 1 root root 1001 9月  20 14:35 ca.csr
		-rw------- 1 root root 1675 9月  20 14:35 ca-key.pem
		-rw-r--r-- 1 root root 1359 9月  20 14:35 ca.pem
		-rw-r--r-- 1 root root  292 9月  20 14:30 config.json
		-rw-r--r-- 1 root root  208 9月  20 14:33 csr.json
		-rw-r--r-- 1 root root 1062 9月  20 15:09 etcd.csr
		-rw-r--r-- 1 root root  296 9月  20 15:03 etcd-csr.json
		-rw------- 1 root root 1675 9月  20 15:09 etcd-key.pem
		-rw-r--r-- 1 root root 1436 9月  20 15:09 etcd.pem 	   
		#检查证书
		[root@devops ssl]# cfssl-certinfo -cert etcd.pem 
		
		#拷贝etcd 证书到etcd 服务器 
		[root@devops k8s-1.11.0]# ansible -i hosts etcd -m copy -a 'src=./ssl/  dest=/etc/kubernetes/ssl/' -k
		SSH password:
		10.39.10.158 | SUCCESS => {
		    "changed": true,
		    "dest": "/etc/kubernetes/ssl/",
		    "src": "/opt/k8s-1.11.0/./ssl/"
		}
		10.39.10.157 | SUCCESS => {
		    "changed": true,
		    "dest": "/etc/kubernetes/ssl/",
		    "src": "/opt/k8s-1.11.0/./ssl/"
		}
		10.39.10.153 | SUCCESS => {
		    "changed": true,
		    "dest": "/etc/kubernetes/ssl/",
		    "src": "/opt/k8s-1.11.0/./ssl/"
		}
		[root@devops k8s-1.11.0]# ansible -i hosts etcd -m shell  -a 'ls -l /etc/kubernetes/ssl/' -k
		SSH password:
		10.39.10.158 | SUCCESS | rc=0 >>
		总用量 36
		-rw-r--r-- 1 root root 1001 9月  20 15:23 ca.csr
		-rw-r--r-- 1 root root 1675 9月  20 15:23 ca-key.pem
		-rw-r--r-- 1 root root 1359 9月  20 15:23 ca.pem
		-rw-r--r-- 1 root root  292 9月  20 15:23 config.json
		-rw-r--r-- 1 root root  208 9月  20 15:23 csr.json
		-rw-r--r-- 1 root root 1062 9月  20 15:23 etcd.csr
		-rw-r--r-- 1 root root  296 9月  20 15:23 etcd-csr.json
		-rw-r--r-- 1 root root 1675 9月  20 15:23 etcd-key.pem
		-rw-r--r-- 1 root root 1436 9月  20 15:23 etcd.pem
		
		10.39.10.157 | SUCCESS | rc=0 >>
		总用量 36
		-rw-r--r-- 1 root root 1001 9月  20 15:23 ca.csr
		-rw-r--r-- 1 root root 1675 9月  20 15:23 ca-key.pem
		-rw-r--r-- 1 root root 1359 9月  20 15:23 ca.pem
		-rw-r--r-- 1 root root  292 9月  20 15:23 config.json
		-rw-r--r-- 1 root root  208 9月  20 15:23 csr.json
		-rw-r--r-- 1 root root 1062 9月  20 15:23 etcd.csr
		-rw-r--r-- 1 root root  296 9月  20 15:23 etcd-csr.json
		-rw-r--r-- 1 root root 1675 9月  20 15:23 etcd-key.pem
		-rw-r--r-- 1 root root 1436 9月  20 15:23 etcd.pem
		
		10.39.10.153 | SUCCESS | rc=0 >>
		总用量 36
		-rw-r--r-- 1 root root 1001 9月  20 15:23 ca.csr
		-rw-r--r-- 1 root root 1675 9月  20 15:23 ca-key.pem
		-rw-r--r-- 1 root root 1359 9月  20 15:23 ca.pem
		-rw-r--r-- 1 root root  292 9月  20 15:23 config.json
		-rw-r--r-- 1 root root  208 9月  20 15:23 csr.json
		-rw-r--r-- 1 root root 1062 9月  20 15:23 etcd.csr
		-rw-r--r-- 1 root root  296 9月  20 15:23 etcd-csr.json
		-rw-r--r-- 1 root root 1675 9月  20 15:23 etcd-key.pem
		-rw-r--r-- 1 root root 1436 9月  20 15:23 etcd.pem		
		非root用户，读取证书会提示没有权限
		[root@devops k8s-1.11.0]# ansible -i hosts etcd -m shell -a "chmod 644 /etc/kubernetes/ssl/etcd*" 
		
		
		[root@devops k8s-1.11.0]# ansible -i hosts etcd -m shell -a "useradd -s /sbin/nologin etcd;mkdir -p /opt/etcd;chown -R etcd:etcd /opt/etcd"
	
## etcd 部署
#### 配置etcd 文件

     # etcd-1 
      
	 vim  /etc/systemd/system/etcd.service
		[Unit]
		Description=Etcd Server
		After=network.target
		After=network-online.target
		Wants=network-online.target
		
		[Service]
		Type=notify
		WorkingDirectory=/opt/etcd/
		User=etcd
		# set GOMAXPROCS to number of processors
		ExecStart=/usr/bin/etcd \
		  --name=etcd-10-153 \
		  --cert-file=/etc/kubernetes/ssl/etcd.pem \
		  --key-file=/etc/kubernetes/ssl/etcd-key.pem \
		  --peer-cert-file=/etc/kubernetes/ssl/etcd.pem \
		  --peer-key-file=/etc/kubernetes/ssl/etcd-key.pem \
		  --trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
		  --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
		  --initial-advertise-peer-urls=https://10.39.10.153:2380 \
		  --listen-peer-urls=https://10.39.10.153:2380 \
		  --listen-client-urls=https://10.39.10.153:2379,http://127.0.0.1:2379 \
		  --advertise-client-urls=https://10.39.10.153:2379 \
		  --initial-cluster-token=k8s-etcd-cluster \
		  --initial-cluster=etcd-10-153=https://10.39.10.153:2380,etcd-10.157=https://10.39.10.157:2380,etcd-10-158=https://10.39.10.158:2380 \
		  --initial-cluster-state=new \
		  --data-dir=/opt/etcd/
		Restart=on-failure
		RestartSec=5
		LimitNOFILE=65536
		
		[Install]
		WantedBy=multi-user.target
	 	   	
	   	
	 # etcd 2 
	     
    vim  /etc/systemd/system/etcd.service
	   [Unit]
	   Description=Etcd Server
	   After=network.target
	   After=network-online.target
	   Wants=network-online.target
			
	   [Service]
	   Type=notify
	   WorkingDirectory=/opt/etcd/
	   User=etcd
	   # set GOMAXPROCS to number of processors
		ExecStart=/usr/bin/etcd \
	    --name=etcd-10-157 \
		 --cert-file=/etc/kubernetes/ssl/etcd.pem \
		 --key-file=/etc/kubernetes/ssl/etcd-key.pem \
		 --peer-cert-file=/etc/kubernetes/ssl/etcd.pem \
		 --peer-key-file=/etc/kubernetes/ssl/etcd-key.pem \
		 --trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
		 --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
		 --initial-advertise-peer-urls=https://10.39.10.157:2380 \
		 --listen-peer-urls=https://10.39.10.157:2380 \
		 --listen-client-urls=https://10.39.10.157:2379,http://127.0.0.1:2379 \
		 --advertise-client-urls=https://10.39.10.157:2379 \
		 --initial-cluster-token=k8s-etcd-cluster \
		 --initial-cluster=etcd-10-153=https://10.39.10.153:2380,etcd-10.157=https://10.39.10.157:2380,etcd-10-158=https://10.39.10.158:2380 \
		 --initial-cluster-state=new \
		 --data-dir=/opt/etcd/
		 Restart=on-failure
		 RestartSec=5
		 LimitNOFILE=65536
			
		 [Install]
		 WantedBy=multi-user.target  	
	   	
	   	
	   	
	 #etcd-3  
	   [Unit]
		Description=Etcd Server
		After=network.target
		After=network-online.target
		Wants=network-online.target
		
		[Service]
		Type=notify
		WorkingDirectory=/opt/etcd/
		User=etcd
		# set GOMAXPROCS to number of processors
		ExecStart=/usr/bin/etcd \
		  --name=etcd-10-158 \
		  --cert-file=/etc/kubernetes/ssl/etcd.pem \
		  --key-file=/etc/kubernetes/ssl/etcd-key.pem \
		  --peer-cert-file=/etc/kubernetes/ssl/etcd.pem \
		  --peer-key-file=/etc/kubernetes/ssl/etcd-key.pem \
		  --trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
		  --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
		  --initial-advertise-peer-urls=https://10.39.10.158:2380 \
		  --listen-peer-urls=https://10.39.10.158:2380 \
		  --listen-client-urls=https://10.39.10.158:2379,http://127.0.0.1:2379 \
		  --advertise-client-urls=https://10.39.10.158:2379 \
		  --initial-cluster-token=k8s-etcd-cluster \
		  --initial-cluster=etcd-10-153=https://10.39.10.153:2380,etcd-10.157=https://10.39.10.157:2380,etcd-10-158=https://10.39.10.158:2380 \
		  --initial-cluster-state=new \
		  --data-dir=/opt/etcd/
		Restart=on-failure
		RestartSec=5
		LimitNOFILE=65536
		
		[Install]
		WantedBy=multi-user.target 	

    
   
     # 修改二进制etcd 文件的权限
       ansible -i hosts etcd -m shell -a "chmod +x /usr/bin/etcd"
       ansible -i hosts etcd -m shell -a "chown -R etcd:etcd /usr/bin/etcd "
       

      [root@kubernetes-153 ~]# cat etcdstatus.sh
		etcdctl --endpoints=https://10.39.10.153:2379,https://10.39.10.157:2379,https://10.39.10.158:2379\
		        --cert-file=/etc/kubernetes/ssl/etcd.pem \
		        --ca-file=/etc/kubernetes/ssl/ca.pem \
		        --key-file=/etc/kubernetes/ssl/etcd-key.pem \
		        cluster-health
	   	
	   	
	   	[root@kubernetes-153 ~]# sh etcdstatus.sh
		member 5dd6ea155d92be23 is healthy: got healthy result from https://10.39.10.157:2379
		member be49a2e23f65ca13 is healthy: got healthy result from https://10.39.10.153:2379
		member d6cc9fc58b82a70b is healthy: got healthy result from https://10.39.10.158:2379
		cluster is healthy
	   	
	   	
	   	
	   	#查看etcd 成员 将脚本中的cluster-health 修改为  member list  
	   	[root@kubernetes-153 ~]# sh etcdstatus.sh
		5dd6ea155d92be23: name=etcd-10-157 peerURLs=https://10.39.10.157:2380 clientURLs=https://10.39.10.157:2379 isLeader=false
		be49a2e23f65ca13: name=etcd-10-153 peerURLs=https://10.39.10.153:2380 clientURLs=https://10.39.10.153:2379 isLeader=true
		d6cc9fc58b82a70b: name=etcd-10-158 peerURLs=https://10.39.10.158:2380 clientURLs=https://10.39.10.158:2379 isLeader=false
	   	

## master 部署	  
### 配置kubernetes 集群 
    
    # 安装kubectl  
     [root@devops k8s-1.11.0]# ansible -i hosts k8s -m copy  -a "src=./kubernetes/_output/dockerized/bin/linux/amd64/kubectl   dest=/usr/bin"  -k 
     [root@devops k8s-1.11.0]# ansible -i hosts k8s -m shell  -a "chmod +x /etc/kubernetes/ssl"
     
     
    # 创建admin 证书
     kubectl 与kube-apiserver 的安全端口通信，需要为安全通信提供TLS 证书和秘钥
     vim admin-csr.json
        {
		  "CN": "admin",
		  "hosts": [],
		  "key": {
		    "algo": "rsa",
		    "size": 2048
		  },
		  "names": [
		    {
		      "C": "CN",
		      "ST": "Beijing",
		      "L": "Beijing",
		      "O": "system:masters",
		      "OU": "System"
		    }
		  ]
		}
     
    
    # 生成admin 证书和私钥
    [root@devops ssl]# cfssl gencert -ca ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes admin-csr.json  | cfssljson -bare admin
    
    [root@devops ssl]# ls -l admin*
	-rw-r--r-- 1 root root 1009 9月  21 14:49 admin.csr
	-rw-r--r-- 1 root root  229 9月  21 14:43 admin-csr.json
	-rw------- 1 root root 1679 9月  21 14:49 admin-key.pem
	-rw-r--r-- 1 root root 1399 9月  21 14:49 admin.pem
	
	#分配证书
	[root@devops k8s-1.11.0]# ansible -i hosts k8s -m copy -a "src=./ssl/   dest=/etc/kubernetes/ssl"
	
	
	生成kubernetes 配置文件，生成证书相关的配置文件存储与/root/.kube  目录中
	
	#配置kubernetes 集群
	
	kubectl config set-cluster kubernetes \
	  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
	  --embed-certs=true \
	  --server=https://10.39.10.157:6443  #其他机器也需要操作
	  
	  
    #配置客户端认证
    kubectl config set-credentials admin \
	  --client-certificate=/etc/kubernetes/ssl/admin.pem \
	  --embed-certs=true \
	  --client-key=/etc/kubernetes/ssl/admin-key.pem  
	
	kubectl config set-context kubernetes \
	 --cluster=kubernetes \
	 --user=admin	
	       
    kubectl config use-context kubernetes
     
     
     
    # 配置kubernets 证书
      创建kubernetes-csr.json  文件
      
      
      vi kubernetes-csr.json
		{
		  "CN": "kubernetes",
		  "hosts": [
		    "127.0.0.1",
		    "10.39.10.153",
		    "10.39.10.157",
		    "10.39.10.158",
		    "10.254.0.1",
		    "kubernetes",
		    "kubernetes.default",
		    "kubernetes.default.svc",
		    "kubernetes.default.svc.cluster",
		    "kubernetes.default.svc.cluster.local"
		  ],
		  "key": {
		    "algo": "rsa",
		    "size": 2048
		  },
		  "names": [
		    {
		      "C": "CN",
		      "ST": "Beijing",
		      "L": "Beijing",
		      "O": "k8s",
		      "OU": "System"
		    }
		  ]
		}
     
 
     ## 这里 hosts 字段中 三个 IP 分别为 127.0.0.1 本机， 10.39.10.153 10.39.10.157 和 10.39.10.158 为 Master 的IP，多个Master需要写多个如果有api server ha 的话，需要将vip 加入到证书里面。  10.254.0.1 为 kubernetes SVC 的 IP， 一般是 部署网络的第一个IP , 如: 10.254.0.1 ， 在启动完成后，我们使用   kubectl get svc ， 就可以查看到

    #生成证书和私钥
     [root@devops ssl]# cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes kubernetes-csr.json  | cfssljson  -bare kubernetes
     
     [root@devops ssl]# ls -l kubernetes*
	-rw-r--r-- 1 root root 1261 9月  23 22:42 kubernetes.csr
	-rw-r--r-- 1 root root  478 9月  23 22:34 kubernetes-csr.json
	-rw------- 1 root root 1675 9月  23 22:42 kubernetes-key.pem
	-rw-r--r-- 1 root root 1627 9月  23 22:42 kubernetes.pem
	     
     
    #分发证书到master 上
    [root@devops k8s-1.11.0]# ansible -i hosts  master -m copy -a "src=./ssl/  dest=/etc/kubernetes/ssl"
    
    
    
    #配置kube-apiserver 
    
    kubelet 首次启动时向 kube-apiserver 发送 TLS Bootstrapping 请求，kube-apiserver 验证 kubelet 请求中的 token 是否与它配置的 token 一致，如果一致则自动为 kubelet生成证书和秘钥。

    # 生成token 
     [root@kubernetes-153 ~]# head -c 16 /dev/urandom | od -An -t x | tr -d ' '
     4c96a17853402bcd3ea5270b4ae72da2
   
    # 创建encryption-config.yaml 配置
       
       cat > encryption-config.yaml <<EOF
		kind: EncryptionConfig
		apiVersion: v1
		resources:
		  - resources:
		      - secrets
		    providers:
		      - aescbc:
		          keys:
		            - name: key1
		              secret: 4c96a17853402bcd3ea5270b4ae72da2
		      - identity: {}
		EOF
           
     [root@devops k8s-1.11.0]# ansible -i hosts k8s -m  copy  -a "src=encryption-config.yaml  dest=/etc/kubernetes/"
     
     
     # 创建 目录/etc/kubernetes/manifests以及kube-apiserver.json文件  
      
     [root@devops k8s-1.11.0]# ansible -i hosts k8s -m shell  -a "mkdir /etc/kubernetes/manifests"
     
     [root@kubernetes-158 manifests]# cat kube-apiserver.json
		{
		  "kind": "Pod",
		  "apiVersion": "v1",
		  "metadata": {
		    "name": "kube-apiserver",
		    "namespace": "kube-system",
		    "creationTimestamp": null,
		    "labels": {
		      "component": "kube-apiserver",
		      "tier": "control-plane"
		    }
		  },
		  "spec": {
		    "volumes": [
		      {
		        "name": "certs",
		        "hostPath": {
		          "path": "/etc/ssl/certs"
		        }
		      },
		      {
		        "name": "pki",
		        "hostPath": {
		          "path": "/etc/kubernetes"
		        }
		      }
		    ],
		    "containers": [
		      {
		        "name": "kube-apiserver",
		        "image": "harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.11.2",
		        "imagePullPolicy": "Always",
		        "command": [
		          "/apiserver",
		          "--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota",
		          "--anonymous-auth=false",
		          "--experimental-encryption-provider-config=/etc/kubernetes/encryption-config.yaml",
		          "--advertise-address=10.39.10.158",
		          "--allow-privileged=true",
		          "--apiserver-count=3",
		          "--audit-policy-file=/etc/kubernetes/audit-policy.yaml",
		          "--audit-log-maxage=30",
		          "--audit-log-maxbackup=3",
		          "--audit-log-maxsize=100",
		          "--audit-log-path=/var/log/kubernetes/audit.log",
		          "--authorization-mode=Node,RBAC",
		          "--bind-address=10.39.10.158",
		          "--secure-port=6443",
		          "--client-ca-file=/etc/kubernetes/ssl/ca.pem",
		          "--kubelet-client-certificate=/etc/kubernetes/ssl/kubernetes.pem",
		          "--kubelet-client-key=/etc/kubernetes/ssl/kubernetes-key.pem",
		          "--enable-swagger-ui=true",
		          "--etcd-cafile=/etc/kubernetes/ssl/ca.pem",
		          "--etcd-certfile=/etc/kubernetes/ssl/etcd.pem",
		          "--etcd-keyfile=/etc/kubernetes/ssl/etcd-key.pem",
		          "--etcd-servers=https://10.39.10.153:2379,https://10.39.10.157:2379,https://10.39.10.158:2379",
		          "--event-ttl=1h",
		          "--kubelet-https=true",
		          "--insecure-bind-address=127.0.0.1",
		          "--insecure-port=8080",
		          "--service-account-key-file=/etc/kubernetes/ssl/ca-key.pem",
		          "--service-cluster-ip-range=10.254.0.0/18",
		          "--service-node-port-range=30000-32000",
		          "--tls-cert-file=/etc/kubernetes/ssl/kubernetes.pem",
		          "--tls-private-key-file=/etc/kubernetes/ssl/kubernetes-key.pem",
		          "--enable-bootstrap-token-auth",
		          "--v=2"
		        ],
		        "resources": {
		          "requests": {
		            "cpu": "250m"
		          }
		        },
		        "volumeMounts": [
		          {
		            "name": "certs",
		            "mountPath": "/etc/ssl/certs"
		          },
		          {
		            "name": "pki",
		            "readOnly": true,
		            "mountPath": "/etc/kubernetes/"
		          }
		        ],
		        "livenessProbe": {
		          "httpGet": {
		            "path": "/healthz",
		            "port": 8080,
		            "host": "127.0.0.1"
		          },
		          "initialDelaySeconds": 15,
		          "timeoutSeconds": 15,
		          "failureThreshold": 8
		        }
		      }
		    ],
		    "hostNetwork": true
		  }
		}
     
     
    #创建kube-controller-manager.json 文件  
       vim  kube-controller-manager.json 
        {
		  "kind": "Pod",
		  "apiVersion": "v1",
		  "metadata": {
		    "name": "kube-controller-manager",
		    "namespace": "kube-system",
		    "labels": {
		      "component": "kube-controller-manager",
		      "tier": "control-plane"
		    }
		  },
		  "spec": {
		    "volumes": [
		      {
		        "name": "certs",
		        "hostPath": {
		          "path": "/etc/ssl/certs"
		        }
		      },
		      {
		        "name": "pki",
		        "hostPath": {
		          "path": "/etc/kubernetes"
		        }
		      },
		      {
		        "name": "plugin",
		        "hostPath": {
		          "path": "/usr/libexec/kubernetes/kubelet-plugins"
		        }
		      },
		      {
		        "name": "qingcloud",
		        "hostPath": {
		          "path": "/etc/qingcloud"
		        }
		      }
		    ],
		    "containers": [
		      {
		        "name": "kube-controller-manager",
		        "image": "harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.11.2",
		        "imagePullPolicy": "Always",
		        "command": [
		          "/controller-manager",
		          "--address=127.0.0.1",
		          "--master=http://127.0.0.1:8080",
		          "--allocate-node-cidrs=false",
		          "--service-cluster-ip-range=10.254.0.0/18",
		          "--cluster-cidr=10.254.64.0/18",
		          "--cluster-name=kubernetes",
		          "--cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem",
		          "--cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem",
		          "--service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem",
		          "--root-ca-file=/etc/kubernetes/ssl/ca.pem",
		          "--experimental-cluster-signing-duration=86700h0m0s",
		          "--leader-elect=true",
		          "--controllers=*,tokencleaner,bootstrapsigner",
		          "--feature-gates=RotateKubeletServerCertificate=true",
		          "--horizontal-pod-autoscaler-use-rest-clients",
		          "--horizontal-pod-autoscaler-sync-period=60s",
		          "--node-monitor-grace-period=40s",
		          " --node-monitor-period=5s",
		          "--pod-eviction-timeout=5m0s",
		          "--v=10"
		        ],
		        "resources": {
		          "requests": {
		            "cpu": "200m"
		          }
		        },
		        "volumeMounts": [
		          {
		            "name": "certs",
		            "mountPath": "/etc/ssl/certs"
		          },
		          {
		            "name": "pki",
		            "readOnly": true,
		            "mountPath": "/etc/kubernetes/"
		          },
		                        {
		            "name": "plugin",
		            "mountPath": "/usr/libexec/kubernetes/kubelet-plugins"
		          },
		          {
		            "name": "qingcloud",
		            "readOnly": true,
		            "mountPath": "/etc/qingcloud"
		          }
		
		        ],
		        "livenessProbe": {
		          "httpGet": {
		            "path": "/healthz",
		            "port": 10252,
		            "host": "127.0.0.1"
		          },
		          "initialDelaySeconds": 15,
		          "timeoutSeconds": 15,
		          "failureThreshold": 8
		        }
		      }
		    ],
		    "hostNetwork": true
		  }
		}
	
     # 创建kube-scheduler.json 文件
	   vim   kube-scheduler.json 
	    {
		  "kind": "Pod",
		  "apiVersion": "v1",
		  "metadata": {
		    "name": "kube-scheduler",
		    "namespace": "kube-system",
		    "creationTimestamp": null,
		    "labels": {
		      "component": "kube-scheduler",
		      "tier": "control-plane"
		    }
		  },
		  "spec": {
		    "containers": [
		      {
		        "name": "kube-scheduler",
		        "image": "harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.11.2",
		        "imagePullPolicy": "Always",
		        "command": [
		          "/scheduler",
		          "--address=127.0.0.1",
		          "--master=http://127.0.0.1:8080",
		          "--leader-elect=true",
		          "--v=2"
		        ],
		        "resources": {
		          "requests": {
		            "cpu": "100m"
		          }
		        },
		        "livenessProbe": {
		          "httpGet": {
		            "path": "/healthz",
		            "port": 10251,
		            "host": "127.0.0.1"
		          },
		          "initialDelaySeconds": 15,
		          "timeoutSeconds": 15,
		          "failureThreshold": 8
		        }
		      }
		    ],
		    "hostNetwork": true
		  }
		}
		  
	  
	配置kubelet 认证
    #创建token 
	将生成的kubeadm 二进制拷贝到master节点
	 [root@devops k8s-1.11.0]# ansible -i hosts master -m copy -a "src=./kubernetes/_output/dockerized/bin/linux/amd64/kubeadm dest=/usr/local/bin"
     [root@devops k8s-1.11.0]# ansible -i hosts master -m shell  -a "chmod +x /usr/local/bin/kubeadm"
	 [root@kubernetes-153 manifests]# kubeadm token create --description kubelet-bootstrap-token --groups system:bootstrappers:kubernetes-153 --kubeconfig ~/.kube/config
	ob8uwo.uebkbjx32b3xi8y3
	 [root@kubernetes-153 manifests]# kubeadm token create --description kubelet-bootstrap-token --groups system:bootstrappers:kubernetes-157 --kubeconfig ~/.kube/config
	53xuhz.a8fyivf0r394lgzq
	 [root@kubernetes-153 manifests]# kubeadm token create --description kubelet-bootstrap-token --groups system:bootstrappers:kubernetes-158 --kubeconfig ~/.kube/config
	bs891f.49z5o1dgnpwuku8d
	
	kubelet 授权kube-apiserver的操作exec run logs 等
	生成bootstrap.kubeconfig 
	#配置集群参数
	kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem   --embed-certs=true   --server=https://10.39.10.153:6443   --kubeconfig=bootstrap.kubeconfig   	
	#配置客户端认证
	
	kubectl config set-credentials kubelet-bootstrap   --token=ob8uwo.uebkbjx32b3xi8y3   --kubeconfig=bootstrap.kubeconfig
	
	#配置关联
	kubectl config set-context default   --cluster=kubernetes   --user=kubelet-bootstrap   --kubeconfig=bootstrap.kubeconfig 
	
	#配置默认关联
	kubectl config use-context default --kubeconfig=bootstrap.kubeconfig 
	
	
	#创建/etc/kubernetes/audit-policy.yaml 文件
	vim  /etc/kubernetes/audit-policy.yaml 
	apiVersion: audit.k8s.io/v1beta1
	kind: Policy
	rules:
	- level: Metadata

	#拷贝生成的bootstrap.kubeconfig 文件	
	scp /etc/kubernetes/bootstrap.kubeconfig  root@10.39.10.158:/etc/kubernetes/

    # 创建自动批准相关CSR请求ClusterRole 
      root@kubernetes-153 ~]# cat /etc/kubernetes/tls-instructs-csr.yaml
		kind: ClusterRole
		apiVersion: rbac.authorization.k8s.io/v1
		metadata:
		 name: system:certificates.k8s.io:certificatesigningrequests:selfnodeserver
		rules:
		 - apiGroups: ["certificates.k8s.io"]
		   resources: ["certificatesigningrequests/selfnodeserver"]
		   verbs: ["create"]
	
	  #查看 
	  [root@kubernetes-153 ~]# kubectl describe ClusterRole/system:certificates.k8s.io:certificatesigningrequests:selfnodeserver
	  
	  #将ClusterRole 绑定到适当的用户组
     #自动批准 system:bootstrappers 组用户 TLS bootstrapping 首次申请证书的 CSR 请求	  [root@kubernetes-153 ~]#   kubectl create clusterrolebinding node-client-auto-approve-csr --clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient --group=system:bootstrappers
     
    #自动批准 system:nodes 组用户更新 kubelet 自身与 apiserver 通讯证书的 CSR 请求
      [root@kubernetes-153 ~]#  kubectl create clusterrolebinding node-client-auto-renew-crt --clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeclient --group=system:nodes  
    
    #自动批准 system:nodes 组用户更新 kubelet 10250 api 端口证书的 CSR 请求
    [root@kubernetes-153 ~]#  kubectl create clusterrolebinding node-server-auto-renew-crt --clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeserver --group=system:nodes
    
 
	创建kubelet.service 
	[root@kubernetes-153 ~]# vim /etc/systemd/system/kubelet.service
	[Unit]
	Description=Kubernetes Kubelet
	Documentation=https://github.com/GoogleCloudPlatform/kubernetes
	After=docker.service
	Requires=docker.service
	
	[Service]
	WorkingDirectory=/var/lib/kubelet
	ExecStart=/usr/local/bin/kubelet \
	  --hostname-override=master-10.153 \
	  --pod-infra-container-image=harbor.enncloud.cn/paas/pause-amd64:3.1 \
	  --bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig \
	  --pod-manifest-path=/etc/kubernetes/manifests  \
	  --kubeconfig=/etc/kubernetes/kubelet.config \
	  --config=/etc/kubernetes/kubelet.config.json \
	  --cert-dir=/etc/kubernetes/pki \
	  --allow-privileged=true \
	  --kube-reserved cpu=500m,memory=512m \
	  --image-gc-high-threshold=85  --image-gc-low-threshold=70 \
	  --logtostderr=true \
	  --enable-controller-attach-detach=true --volume-plugin-dir=/usr/libexec/kubernetes/kubelet-plugins/volume/exec/ \
	  --v=2
	
	[Install]
	WantedBy=multi-user.target

	
	#拷贝kubelet 二进制文件到master 的/usr/local/bin 目录下
	[root@devops k8s-1.11.0]# ansible -i hosts master -m copy -a "src=./kubernetes/_output/dockerized/bin/linux/amd64/kubelet dest=/usr/local/bin" 
    [root@devops k8s-1.11.0]# ansible -i hosts master -m shell  -a "chmod +x /usr/local/bin/kubelet"
    	
    # 创建  /etc/kubernetes/kubelet.config.json  文件
     vim  /etc/kubernetes/kubelet.config.json 
	     {
		  "kind": "KubeletConfiguration",
		  "apiVersion": "kubelet.config.k8s.io/v1beta1",
		  "authentication": {
		    "x509": {
		      "clientCAFile": "/etc/kubernetes/ssl/ca.pem"
		    },
		    "webhook": {
		      "enabled": true,
		      "cacheTTL": "2m0s"
		    },
		    "anonymous": {
		      "enabled": false
		    }
		  },
		  "authorization": {
		    "mode": "Webhook",
		    "webhook": {
		      "cacheAuthorizedTTL": "5m0s",
		      "cacheUnauthorizedTTL": "30s"
		    }
		  },
		  "address": "10.39.10.153",
		  "port": 10250,
		  "readOnlyPort": 10255,
		  "cgroupDriver": "systemd",
		  "hairpinMode": "promiscuous-bridge",
		  "serializeImagePulls": false,
		  "RotateCertificates": true,
		  "featureGates": {
		    "RotateKubeletClientCertificate": true,
		    "RotateKubeletServerCertificate": true
		  },
		  "MaxPods": "512",
		  "failSwapOn": false,
		  "containerLogMaxSize": "10Mi",
		  "containerLogMaxFiles": 5,
		  "clusterDomain": "cluster.local",
		  "clusterDNS": ["10.254.0.2"]
		}	

    
     	
	
	启动 systemctl  restart kubelet 
	自动的拉起 apiserver  controller-manager  scheduler 这三个容器
	docker ps 查看启动容器，如果有错误使用docker ps -a 或者docker logs 查看相应的错误 
	如果镜像拉取不下来，需要手动的去拉取
	 pause-amd64:3.1
    hyperkube-amd64:v1.11.2
   
	#故障解决
	提示以下错误
	 summary.go:102] Failed to get system container stats for "/system.slice/dock....service"
	9月 25 14:20:10 kubernetes-153 kubelet[11287]: E0925 14:20:10.151863   11287 summary.go:102] Failed to get system container stats for "/system.slice/kubelet.servi...
	9月 25 14:20:20 kubernetes-153 kubelet[11287]: E0925 14:20:20.165881   11287 summary.go:102] Failed to get system container stats for "/system.slice/dock....service"
	9月 25 14:20:20 kubernetes-153 kubelet[11287]: E0925 14:20:20.167415   11287 summary.go:102] Failed to get system container stats for "/system.slice/kubelet.servi...
	
    解决方法
      	[root@kubernetes-153 kubernetes]# mkdir  /etc/systemd/system/kubelet.service.d/
      	[root@kubernetes-153 kubernetes]# vim /etc/systemd/system/kubelet.service.d/11-cgroups.conf
      	
	   	[Service]
		CPUAccounting=true
		MemoryAccounting=true
	   
	   
	   修改kubelet 启动参数
	   添加以下参数
	   --runtime-cgroups=/systemd/system.slice \
      --kubelet-cgroups=/systemd/system.slice \
	   	
	   
	  #重启kubelet   	
	   systemctl daemon-reload
	   systemctl restart kubelet
      
      # 创建RBAC 
      [root@kubernetes-153 ~]# kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user kubernetes
      
      [root@kubernetes-153 ~]# kubectl get cs
		NAME                 STATUS    MESSAGE              ERROR
		scheduler            Healthy   ok
		controller-manager   Healthy   ok
		etcd-0               Healthy   {"health": "true"}
		etcd-1               Healthy   {"health": "true"}
		etcd-2               Healthy   {"health": "true"}
      
    添加其他master
     查看token 
     [root@kubernetes-153 ~]# kubeadm token list --kubeconfig ~/.kube/config
     #生成10.39.10.157 的bootstrap.kubeconfig 
     #配置集群参数
     [root@kubernetes-153 ~]# kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem   --embed-certs=true   --server=https://10.39.10.153:6443   --kubeconfig=kubernetes-157-bootstrap.kubeconfig
	   	
	 #配置客户端认证
	  [root@kubernetes-153 ~]# kubectl config set-credentials kubelet-bootstrap   --token=ob8uwo.uebkbjx32b3xi8y3   --kubeconfig=bootstrap.kubeconfig  	 
	 #配置关联
	[root@kubernetes-153 ~]#  kubectl config set-context default   --cluster=kubernetes   --user=kubelet-bootstrap   --kubeconfig=kubernetes-157-bootstrap.kubeconfig
		  
	#配置默认关联
	[root@kubernetes-153 ~]# kubectl config use-context default --kubeconfig=kubernetes-157-bootstrap.kubeconfig
	
	拷贝生成kubernetes-157-bootstrap.kubeconfig  
    [root@kubernetes-153 ~]# scp kubernetes-157-bootstrap.kubeconfig root@10.39.10.157:/etc/kubernetes/bootstrap.kubeconfig
   
   
    # 创建  /etc/kubernetes/kubelet.config.json  文件
	  vim  /etc/kubernetes/kubelet.config.json 
		     {
			  "kind": "KubeletConfiguration",
			  "apiVersion": "kubelet.config.k8s.io/v1beta1",
			  "authentication": {
			    "x509": {
			      "clientCAFile": "/etc/kubernetes/ssl/ca.pem"
			    },
			    "webhook": {
			      "enabled": true,
			      "cacheTTL": "2m0s"
			    },
			    "anonymous": {
			      "enabled": false
			    }
			  },
			  "authorization": {
			    "mode": "Webhook",
			    "webhook": {
			      "cacheAuthorizedTTL": "5m0s",
			      "cacheUnauthorizedTTL": "30s"
			    }
			  },
			  "address": "10.39.10.153",
			  "port": 10250,
			  "readOnlyPort": 10255,
			  "cgroupDriver": "systemd",
			  "hairpinMode": "promiscuous-bridge",
			  "serializeImagePulls": false,
			  "RotateCertificates": true,
			  "featureGates": {
			    "RotateKubeletClientCertificate": true,
			    "RotateKubeletServerCertificate": true
			  },
			  "MaxPods": "512",
			  "failSwapOn": false,
			  "containerLogMaxSize": "10Mi",
			  "containerLogMaxFiles": 5,
			  "clusterDomain": "cluster.local",
			  "clusterDNS": ["10.254.0.2"]
			}	
	
     
    配置bootstrap RBAC 权限
    [root@kubernetes-153 ~]# kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
    
    #创建/etc/kubernetes/audit-policy.yaml 文件
    vim  /etc/kubernetes/audit-policy.yaml 
      apiVersion: audit.k8s.io/v1beta1
	   kind: Policy
	   rules:
      	- level: Metadata
   
    
    [root@kubernetes-153 ~]# kubectl get node
	NAME             STATUS    ROLES     AGE       VERSION
	kubernetes-153   Ready     <none>    3h        v1.11.2
	kubernetes-157   Ready     <none>    1h        v1.11.2
	kubernetes-158   Ready     <none>    5m        v1.11.2    
		   
		    
    master 的部分配置完毕了，这里并没有配置master  apiserver 的高可用，在后面在部署apiserver的高可用
    
## node 部署
    配置kube-proxy 
    创建kube-proxy 证书
    [root@devops ssl]#  vim  kube-proxy-csr.json
		{
		  "CN": "system:kube-proxy",
		  "hosts": [],
		  "key": {
		    "algo": "rsa",
		    "size": 2048
		  },
		  "names": [
		    {
		      "C": "CN",
		      "ST": "ShenZhen",
		      "L": "ShenZhen",
		      "O": "k8s",
		      "OU": "System"
		    }
		  ]
		}

    
    [root@devops ssl]# cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem -config=config.json  -profile=kubernetes kube-proxy-csr.json  | cfssljson -bare kube-proxy
    [root@devops ssl]# ls -l kube-proxy*
	-rw-r--r-- 1 root root 1013 9月  25 21:36 kube-proxy.csr
	-rw-r--r-- 1 root root  233 9月  25 18:07 kube-proxy-csr.json
	-rw------- 1 root root 1679 9月  25 21:36 kube-proxy-key.pem
	-rw-r--r-- 1 root root 1403 9月  25 21:36 kube-proxy.pem
	   
	#创建kube-proxy  kubeconfig 文件
	配置集群
	以下操作在master 上进行
	#配置集群
	[root@kubernetes-153 ~]# kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem   --embed-certs=true   --server=https://10.39.10.153:6443   --kubeconfig=kube-proxy.kubeconfig
	
	 
	#配置客户端认证
	[root@kubernetes-153 ~]# kubectl config set-credentials kube-proxy   --client-certificate=/etc/kubernetes/ssl/kube-proxy.pem   --client-key=/etc/kubernetes/ssl/kube-proxy-key.pem   --embed-certs=true   --kubeconfig=kube-proxy.kubeconfig
    
    #配置关联
    [root@kubernetes-153 ~]# kubectl config set-context default   --cluster=kubernetes   --user=kube-proxy   --kubeconfig=kube-proxy.kubeconfig
	   
	 #配置默认关联
	 [root@kubernetes-153 ~]# kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
     
     
    拷贝到node 客户端 
    1. kube-proxy 证书拷贝到/etc/kubernetes/pki/ 
    2. kube-proxy.kubeconfig 文件拷贝到/etc/kubernetes/ 
    3. 拷贝编译的二进制kubelet 拷贝到node 的/usr/local/bin/
    
       
    # 创建node 的token
    [root@kubernetes-153 ~]# kubeadm token create --description kubelet-bootstrap-token --groups system:bootstrappers:kubernetes-182 --kubeconfig ~/.kube/config
    jtwjhb.m09f9ye5rrgr1rvw
    
    
    #配置集群参数
    [root@kubernetes-153 ~]# kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem  --embed-certs=true  --server=https://10.39.10.153:6443 --kubeconfig=kubernetes-182-bootstrap.kubeconfig
    
    #配置客户端认证
    [root@kubernetes-153 ~]#  kubectl config set-credentials  kubelet-bootstrap  --token=jtwjhb.m09f9ye5rrgr1rvw   --kubeconfig=kubernetes-182-bootstrap.kubeconfig
    
    #配置关联
    [root@kubernetes-153 ~]# kubectl config set-context default  --cluster=kubernetes  --user=kubelet-bootstrap  --kubeconfig=kubernetes-182-bootstrap.kubeconfig
    
    #配置默认关联 
    [root@kubernetes-153 ~]# kubectl config use-context default  --kubeconfig=kubernetes-182-bootstrap.kubeconfig
    
    拷贝到182的node 节点
    [root@kubernetes-153 ~]# scp kubernetes-182-bootstrap.kubeconfig  root@10.39.10.182:/etc/kubernetes/bootstrap.kubeconfig
    
    
    #创建kubelet.config.json文件
    [root@kubernetes-182 ~]# vim /etc/kubernetes/kubelet.config.json
	    {
	  "kind": "KubeletConfiguration",
	  "apiVersion": "kubelet.config.k8s.io/v1beta1",
	  "authentication": {
	    "x509": {
	      "clientCAFile": "/etc/kubernetes/ssl/ca.pem"
	    },
	    "webhook": {
	      "enabled": true,
	      "cacheTTL": "2m0s"
	    },
	    "anonymous": {
	      "enabled": false
	    }
	  },
	  "authorization": {
	    "mode": "Webhook",
	    "webhook": {
	      "cacheAuthorizedTTL": "5m0s",
	      "cacheUnauthorizedTTL": "30s"
	    }
	  },
	  "address": "10.39.10.182",
	  "port": 10250,
	  "readOnlyPort": 10255,
	  "cgroupDriver": "systemd",
	  "hairpinMode": "promiscuous-bridge",
	  "serializeImagePulls": false,
	  "RotateCertificates": true,
	  "featureGates": {
	    "RotateKubeletClientCertificate": true,
	    "RotateKubeletServerCertificate": true
	  },
	  "MaxPods": "512",
	  "failSwapOn": false,
	  "containerLogMaxSize": "10Mi",
	  "containerLogMaxFiles": 5,
	  "clusterDomain": "cluster.local",
	  "clusterDNS": ["10.254.0.2"]
	}
	    
    #创建kubelet.service 
     [root@kubernetes-182 ~]# vim /etc/systemd/system/kubelet.service
		[Unit]
		Description=Kubernetes Kubelet
		Documentation=https://github.com/GoogleCloudPlatform/kubernetes
		After=docker.service
		Requires=docker.service
		
		[Service]
		WorkingDirectory=/var/lib/kubelet
		ExecStart=/usr/local/bin/kubelet \
		  --hostname-override=kubernetes-182 \
		  --pod-infra-container-image=harbor.enncloud.cn/paas/pause-amd64:3.1 \
		  --bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig \
		  --kubeconfig=/etc/kubernetes/kubelet.config \
		  --config=/etc/kubernetes/kubelet.config.json \
		  --cert-dir=/etc/kubernetes/pki \
		  --allow-privileged=true \
		  --kube-reserved cpu=500m,memory=512m \
		  --image-gc-high-threshold=85  --image-gc-low-threshold=70 \
		  --logtostderr=true \
		  --enable-controller-attach-detach=true --volume-plugin-dir=/usr/libexec/kubernetes/kubelet-plugins/volume/exec/ \
		  --v=2
		
		[Install]
		WantedBy=multi-user.target
	
	 # 启动kubelet 
	  [root@kubernetes-182]# systemctl restart kubelet
	 
	 
	 # 在master 上创建kube-proxy的ds文件，
	   这个文件在
[https://github.com/kubernetes/kubernetes/tree/v1.11.2/cluster/addons/kube-proxy]()

	 vim  kube-proxy.yaml
	   apiVersion: extensions/v1beta1
		kind: DaemonSet
		metadata:
		  labels:
		    component: kube-proxy
		    k8s-app: kube-proxy
		    kubernetes.io/cluster-service: "true"
		    name: kube-proxy
		    tier: node
		  name: kube-proxy
		  namespace: kube-system
		spec:
		  selector:
		    matchLabels:
		      component: kube-proxy
		      k8s-app: kube-proxy
		      kubernetes.io/cluster-service: "true"
		      name: kube-proxy
		      tier: node
		  template:
		    metadata:
		      annotations:
		        scheduler.alpha.kubernetes.io/affinity: '{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"beta.kubernetes.io/arch","operator":"In","values":["amd64"]}]}]}}}'
		        scheduler.alpha.kubernetes.io/tolerations: '[{"key":"dedicated","value":"master","effect":"NoSchedule"}]'
		      labels:
		        component: kube-proxy
		        k8s-app: kube-proxy
		        kubernetes.io/cluster-service: "true"
		        name: kube-proxy
		        tier: node
		    spec:
		      containers:
		      - command:
		        - /proxy
		        - --cluster-cidr=10.254.64.0/18
		        - --kubeconfig=/run/kubeconfig
		        - --logtostderr=true
		        - --proxy-mode=iptables
		        - --v=2
		        image: harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.11.2
		        imagePullPolicy: IfNotPresent
		        name: kube-proxy
		        securityContext:
		          privileged: true
		        volumeMounts:
		        - mountPath: /var/run/dbus
		          name: dbus
		        - mountPath: /run/kubeconfig
		          name: kubeconfig
		        - mountPath: /etc/kubernetes/pki
		          name: pki
		      dnsPolicy: ClusterFirst
		      hostNetwork: true
		      restartPolicy: Always
		      volumes:
		      - hostPath:
		          path: /etc/kubernetes/kube-proxy.kubeconfig
		        name: kubeconfig
		      - hostPath:
		          path: /var/run/dbus
		        name: dbus
		      - hostPath:
		          path: /etc/kubernetes/pki
		        name: pki
		  updateStrategy:
		    type: OnDelete	
	  
	 [root@kubernetes-153]# kubectl apply -f  kube-proxy.yaml  
	 
	 [root@kubernetes-153 ~]# kubectl get pod -n kube-system
		NAME                                     READY     STATUS    RESTARTS   AGE
		kube-apiserver-kubernetes-153            1/1       Running   34         1d
		kube-apiserver-kubernetes-157            1/1       Running   6          22h
		kube-apiserver-kubernetes-158            1/1       Running   0          21h
		kube-controller-manager-kubernetes-153   1/1       Running   1          1d
		kube-controller-manager-kubernetes-157   1/1       Running   0          22h
		kube-controller-manager-kubernetes-158   1/1       Running   0          21h
		kube-proxy-4mvqm                         1/1       Running   11         19m
		kube-proxy-fw5bb                         1/1       Running   0          3h
		kube-proxy-xvh5z                         1/1       Running   0          5m
		kube-proxy-zfgh9                         1/1       Running   8          19m
		kube-scheduler-kubernetes-153            1/1       Running   1          1d
		kube-scheduler-kubernetes-157            1/1       Running   0          22h
		kube-scheduler-kubernetes-158            1/1       Running   0          21h
					   
	备注，如果master 是单独的话不需要将kube-proxy跑到master 上去，可以给master 打上标签，禁止调度即可。如果kubelet第一次注册的时候有问题，可以将kubelet.config 删除，然后再重新启动kubelet即可。
	
	#安装calico
[https://docs.projectcalico.org/v3.2/getting-started/kubernetes/]() 	

    需要在kubelet 服务启动参数添加 --network-plugin=cni \ 参数
    vim /etc/systemd/system/kubelet.service
    
    ..........
       --network-plugin=cni \
    .........
    
    systemctl daemon-reload
    systemctl restart kubelet
    
    如果开启了rabc 需要下载
    wget https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/rbac.yaml 
    [root@kubernetes-153 ~]# kubectl apply -f rbac.yaml
    clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
    clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
    clusterrole.rbac.authorization.k8s.io/calico-node created
    clusterrolebinding.rbac.authorization.k8s.io/calico-node created  
	
	需要修改 calico.yaml文件中的
	[root@kubernetes-153 ~]# vim calico.yaml
	 etcd_endpoints: "https://10.30.10.153:2379,https://10.39.10.157:2379,https://10.39.10.158:2379"
	  etcd_ca: "/calico-secrets/etcd-ca"
     etcd_cert: "/calico-secrets/etcd-cert"
     etcd_key: "/calico-secrets/etcd-key"
     
     # 这里面要写入 base64 的信息

	data:
     etcd-key: (cat /etc/kubernetes/ssl/etcd-key.pem | base64 | tr -d '\n')
     etcd-cert: (cat /etc/kubernetes/ssl/etcd.pem | base64 | tr -d '\n')
     etcd-ca: (cat /etc/kubernetes/ssl/ca.pem | base64 | tr -d '\n')


    - name: CALICO_IPV4POOL_CIDR
              value: "10.253.0.0/18"
  
   
    备注 10.253.0.0/18 这个地址需要和kube-controller-manager.json 里面的配置一样
    
 
    #创建calico 服务
    [root@kubernetes-153 ~]# kubectl apply -f calico.yaml
	configmap/calico-config created
	secret/calico-etcd-secrets created
	daemonset.extensions/calico-node created
	serviceaccount/calico-node created
	deployment.extensions/calico-kube-controllers created
	serviceaccount/calico-kube-controllers created
	
	

	
  
    # 安装calicoctl 
    cd /usr/local/bin/ 
    wget -c  https://github.com/projectcalico/calicoctl/releases/download/v3.2.1/calicoctl  
    
    创建/etc/calico/calicoctl.cfg  文件
    mkdir /etc/calico
    [root@kubernetes-153 ~]# vim   /etc/calico/calicoctl.cfg
	apiVersion: projectcalico.org/v3
	kind: CalicoAPIConfig
	metadata:
	spec:
	  datastoreType: "etcdv3"
	  etcdEndpoints: "https://10.39.10.153:2379,https://10.39.10.157:2379,https://10.39.10.158:2379"
	  etcdKeyFile: "/etc/kubernetes/ssl/etcd-key.pem"
	  etcdCertFile: "/etc/kubernetes/ssl/etcd.pem"
	  etcdCACertFile: "/etc/kubernetes/ssl/ca.pem”
	  
    # 查看calico 的模式  ipip bgp 	   
	 [root@kubernetes-153 ~]# calicoctl get ippool -o yaml
		apiVersion: projectcalico.org/v3
		items:
		- apiVersion: projectcalico.org/v3
		  kind: IPPool
		  metadata:
		    creationTimestamp: 2018-09-26T11:06:11Z
		    name: default-ipv4-ippool
		    resourceVersion: "161060"
		    uid: 2d3408b1-c17c-11e8-851d-525469f66d5d
		  spec:
		    cidr: 192.168.0.0/16
		    ipipMode: Always
		    natOutgoing: true
		kind: IPPoolList
		metadata:
		  resourceVersion: "165191"
   
    # 部署kube-dns  
  [https://github.com/kubernetes/kubernetes/tree/v1.11.2/cluster/addons/dns/kube-dns]() 
  
    [root@kubernetes-153 ~]# wget  https://raw.githubusercontent.com/kubernetes/kubernetes/v1.11.2/cluster/addons/dns/kube-dns/kube-dns.yaml.base
    [root@kubernetes-153 ~]# mv kube-dns.yaml.base kube-dns.yaml
    修改前
    [root@kubernetes-153 ~]# vim kube-dns.yaml 
        clusterIP: __PILLAR__DNS__SERVER__
          - --domain=__PILLAR__DNS__DOMAIN__.
          - --server=/__PILLAR__DNS__DOMAIN__/127.0.0.1#10053    
          - --probe=kubedns,127.0.0.1:10053,kubernetes.default.svc.__PILLAR__DNS__DOMAIN__,5,SRV
        - --probe=dnsmasq,127.0.0.1:53,kubernetes.default.svc.__PILLAR__DNS__DOMAIN__,5,SRV
  
    修改后  
    [root@kubernetes-153 ~]# vim kube-dns.yaml   
      clusterIP: 10.254.0.2
      - --domain=cluster.local
      - --server=/cluster.local./127.0.0.1#10053
      - --probe=kubedns,127.0.0.1:10053,kubernetes.default.svc.cluster.local,5,SRV
      - --probe=dnsmasq,127.0.0.1:53,kubernetes.default.svc.cluster.local,5,SRV
    [root@kubernetes-153 ~]# kubectl apply -f kube-dns.yaml
	service/kube-dns created
	serviceaccount/kube-dns created
	configmap/kube-dns created
	deployment.extensions/kube-dns created
	
	#集群部署完毕
	[root@kubernetes-153 ~]# kubectl get pod -n kube-system
	NAME                                       READY     STATUS    RESTARTS   AGE
	calico-kube-controllers-6985547989-784m8   1/1       Running   0          1h
	calico-node-48xz8                          2/2       Running   0          1h
	calico-node-64cbz                          2/2       Running   0          1h
	calico-node-hm4r6                          2/2       Running   0          1h
	calico-node-kfxvt                          2/2       Running   0          1h
	kube-apiserver-kubernetes-153              1/1       Running   34         1d
	kube-apiserver-kubernetes-157              1/1       Running   6          1d
	kube-apiserver-kubernetes-158              1/1       Running   0          1d
	kube-controller-manager-kubernetes-153     1/1       Running   1          1d
	kube-controller-manager-kubernetes-157     1/1       Running   0          1d
	kube-controller-manager-kubernetes-158     1/1       Running   0          1d
	kube-dns-7b99bf7c44-sclxm                  3/3       Running   0          18s
	kube-proxy-4mvqm                           1/1       Running   11         5h
	kube-proxy-fw5bb                           1/1       Running   0          8h
	kube-proxy-xvh5z                           1/1       Running   0          5h
	kube-proxy-zfgh9                           1/1       Running   8          5h
	kube-scheduler-kubernetes-153              1/1       Running   1          1d
	kube-scheduler-kubernetes-157              1/1       Running   0          1d
	kube-scheduler-kubernetes-158              1/1       Running   0          1d
	
	
	
	#总结
	在按照部署的过程中可能会遇到镜像拉取不下来，那么就需要翻墙