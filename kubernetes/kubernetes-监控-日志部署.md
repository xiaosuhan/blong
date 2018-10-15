#### kubernetes 监控部署
     
   [https://github.com/coreos/prometheus-operator.git]()
    
    [root@kubernetes-153 ~]# git clone https://github.com/coreos/prometheus-operator.git
    [root@kubernetes-153 ~]# cd prometheus-operator/
    [root@kubernetes-153 prometheus-operator]# git checkout v0.23.2
   
   
    
    # 创建服务
    [root@kubernetes-153 prometheus-operator]# kubectl apply -f bundle.yaml -n monitoring
      
      