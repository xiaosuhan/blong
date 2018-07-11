## ansible playbook 

     playbooks 是ansible 的配置 部署 编排语言，他们可以被描述为一个需要希望远程主机执行命令的方案，或者一组IT程序运行的命令
     
    是以YAML 格式的来操作的，所以遵循如下格式
    
    规则一 缩进
      yaml 使用一个固定的缩进风格表示数据层结构关系，需要每个缩进级别由两个空格组成，一定不能使用tab 键 
      
    规则二 冒号
       每个冒号后面一定要有一个空格(以冒号结尾不需要空格，表示文件路径的模板可以不需要空格)
       
    规则三 短横线
        想要表示列表项，使用一个短横杠加一个空格，多个项使用同样的缩进级别作为同一个列表的一部分
        
        
### 配置
      有以下核心组件
      tasks: 任务，由模块定义的操作的列表
      variables: 变量
      templates: 模板，即使用了模板语法的文本文件
      handlers:  由特定条件出发的tasks 
      roles:  角色
      
      
      基础组件
      Hosts: 运行指定任务的目标主机
      remote_user: 在远程主机以哪个用户身份运行
         sudo_user: 非管理员用户由哪一些组成
      tasks:  任务列表
         由模块与木块参数组成
      ansible-playbook --syntax-check  /path/to/playbook.yaml   使用--syntan-check 做语法检测
      
     测试运行
     -C  --list-host 影响的主机   --list-tasks  列出任务列表     --list-tags 列出所有标签
    
    运行ansible-playbook 
    
    
    例子：
    - hosts: all 
      remote_user: root 
      tasks:
      - name: install  nginx package 
        yum:  name=nginx
        ignore_errors:  yes  
      - name: start nginx service 
        service: name=nginx   state=started 
    忽略错误ignore_errors:  yes
      
       
 	                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          
### 变量
    ansible-playbook -e pkgname=httpd  op.yaml
    - hosts: all 
      remote_user: root 
      tasks: 
        -name: install {{pkgname}}
          yum: name={{ pkgname }}
         tags: install {{pkgname}}
        -name: start {{pkgname}}
         service: name={{ pkgname }}  state=started  enabled=true 
    
    也可以指定变量
      - hosts: all 
       remote_user: root
       vars: 
         -  pkgname: nginx  
       tasks: 
        -name: install {{pkgname}}
          yum: name={{ pkgname }}
         tags: install {{pkgname}}
        -name: start {{pkgname}}
         service: name={{ pkgname }}  state=started  enabled=true 
    
### 实战
     shell 脚本与playbook的转换
     - hosts: all
       tasks:
         - name: "安装apche"
          command: yum install httpd
         - name: 复制配置文件
          command: cp /tmp/httpd.conf /etc/httpd/conf/httpd.conf
     
     ansible-playbook ansible-nginx.yaml --list-host    
     --list-host 
              
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
             
           

    