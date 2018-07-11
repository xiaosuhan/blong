## gitlab 容器化部署
    
    1. docker pull 对应的版本的容器
  [https://hub.docker.com/r/gitlab/gitlab-ce/]() 
       
     通过Dockerfile 查看需要挂载的文件目录有
     /etc/gitlab   配置文件 
     /var/log/gitlab   日志
     /var/opt/gitlab   数据文件
    
    2. 使用容器启动 
      docker run -d \
       -p 80:80 \
       -p 443:443 \
       -p 22:22 \
       --name gitlab \
       --restart unless-stopped \
       -v /data/gitlab/config:/etc/gitlab \
       -v /data/gitlab/logs:/var/log/gitlab \
       -v /data/gitlab/data:/var/opt/gitlab \
       gitlab/gitlab-ce:10.7.3-ce
    
    3. 配置https 
       external_url 'https://gitlab.xxx.com'
       
          
         
    
          
     
    
     












## gitlab 开启hooks 
   
   
   
   
   
   
     curl -X GET --header "PRIVATE-TOKEN: VPDYDHdeyzYPYHc3nVz3" 'http://10.39.15.24/api/v4/application/settings?'
   
   
     "allow_local_requests_from_hooks_and_services":true,