## ansible 

### 1. centos 上安装ansible  
    yum install epel-release  -y
    yum install ansible   -y 
### 2. 配置文件
     ansible.cfg    hosts   roles  
    
     ansible.cfg ansible 配置文件
     hosts 主机组的文件
     roles 定义角色的   
      需要配置ssh 免登陆  
     ssh-copy-id  10.39.15.14  
### 查看有哪些模块
     ansible-doc -l 查看有哪些模块
     ansible-doc --help  查看命令帮助

### 使用命令帮助 
      1. ping 模块
      ansible-doc -s ping 
      ansible-doc -v  ping
        
      2. shell  模块
      ansible-doc -v shell
      ansible test -m shell -a "uptime" 
      
      3. command 模块 
      ansible test -m command -a "df -h"
      ansible test -m command -a "touch /tmp/a.txt"
      
      4. copy 模块 
      ansible-doc -v  copy   
      ansible all -m copy -a "src=/etc/hosts   dest=/tmp/hosts  mode=600  owner=elasticsearch"  
      mode 是文件权限
      owner 是用户 
      5. file 模块
      使用file 创建文件
      ansible all -m file -a "path=/tmp/abcd  state=touch"
      
      使用file 删除文件
      ansible all -m file -a "path=/tmp/abcd  state=absent"
      
      synchronizw 模块(rsync)
      模块参数:
      archive 递推标志，权限，时间等待
      delete=yes 使两边的内容一样(即以推送为主)
      compress=yes 开启压缩(默认yes)
      
      ansible all -m synchronize -a "src=~/prometheus.yml dest=/tmp"
      
      拉取远端的/etc/hosts 到本地的/tmp 
      ansible all -m synchronize -a "compress=yes   group=yes links=yes delete=yes  mode=pull   src=/etc/hosts   dest=/tmp" 
      
      
      script 模块 
      远程执行脚本
      ansible all -m script -a  "/tmp/a.sh"  
      
      
      
      
      user 模块 
      创建用户密码加密
      echo "xingxing" | openssl  passwd -1 -stdin  
      创建用户
      ansible all -m user -a "name=xx  password='$1$UxbCY0m7$EeyH442sFvwcTR/UudaT2'  uid=2000" 
      删除用户
      ansible all -m user -a "name=xx  password='$1$UxbCY0m7$EeyH442sFvwcTR/UudaT2'  uid=2000   state=absent" 
      修改uid 
      ansible all -m user -a "name=xx uid=3000"  
      
      ansible all -m user -a "name=xx remove=yes    state=absent"
      
      创建用户dba 
      ansible all -m user -a "name=dba shell=/bin/bash   append=yes home=/home/dba state=present"  
      
      删除用户
      ansible all -m user -a "name=dba remove=yes state=absent"
      
      
      cron 计划任务 
      ansible all -m cron -a "name='add ntpdata  sync'   minute=*/2   job='ntpdata cn.pool.ntp.org'  state=present"
      
      
     删除计划
     ansible all -m cron -a 'name="add ntpdata  sync"  state=absent'
     
     
     yum 模块
     
     ansible all -m yum -a "name=httpd state=latest"
     启动 
     ansible-doc -s service
     开机启动  
     ansible all -m service -a "name=httpd  enable=yes state=started" 
     
     
    
     
               
### 客户端执行命令
##### 2.1.1 拷贝秘钥    
    ansible test -m copy -a 'src=~/.ssh/id_rsa.pub dest=~' -k
    ansible test -a 'ls' -k 
    ansible test -m shell -a 'mkdir -p .ssh'
    ansible test -m shell -a 'cat ~/id_rsa.pub >>  ~/.ssh/authorized_keys' -k
    ansible test -m shell -a 'cat .ssh/authorized_keys'

    方法二：
    ansible all -m authorized_key -a "user=ubuntu key='{{ lookup('file','~/.ssh/id_rsa.pub')}}'  path='/ubuntu/.ssh/authorized_keys' manage_dir=no" --ask-pass -c paramiko
    
    


    