# cobber 使用
##  第一 cobber 简介
      Cobbler 可使用 kickstart 模板。基于 Red Hat 或 Fedora 的系统使用 kickstart 文件来自动化安装流程。 通过使用模板，您就会拥有基本的 kickstart 模板，然后定义如何针对一种配置文件或机器配置而替换其中 的变量。例如，一个模板可能包含两个变量 $domain 和 $machine_name。在 Cobbler 配置中，一个配置文件 指定 domain=mydomain.com，并且每台使用该配置文件的机器在 machine_name 变量中指定其名称。该配置文 件中的所有机器都使用相同的 kickstart 安装且针对 domain=mydomain.com 进行配置，但每台机器拥有其自 己的机器名称。您仍然可以使用 kickstart 模板在不同的域中安装其他机器并使用不同的机器名称。 工作原理:
 
     Server 端:
     第一步，启动 Cobbler 服务
     第二步，进行 Cobbler 错误检查，执行 cobbler check 命令 
     第三步，进行配置同步，执行 cobbler sync 命令 
     第四步，复制相关启动文件文件到 TFTP 目录中 
     第五步，启动 DHCP 服务，提供地址分配
     第六步，DHCP 服务分配 IP 地址
     第七步，TFTP 传输启动文件
     第八步，Server 端接收安装信息
     第九步，Server 端发送 ISO 镜像与 Kickstart 文件
     
     Client 端:
     第一步，客户端以 PXE 模式启动
     第二步，客户端获取 IP 地址 
     第三步，通过 TFTP 服务器获取启动文件
     第四步，进入 Cobbler 安装选择界面 
     第五步，客户端确定加载信息 
     第六步，根据配置信息准备安装系统 
     第七步，加载 Kickstart 文件 
     第八步，传输系统安装的其它文件 
     第九步，进行安装系统

## 安装yum 源

### 2.1 选择 epel 源
    Centos5 32 位: rpm -Uvh http://mirrors.ustc.edu.cn/fedora/epel/5/i386/epel-release-5-4.noarch.rpm Centos5 64 位 : rpm -Uvh http://mirrors.ustc.edu.cn/fedora/epel/5/x86_64/epel-release-5-4.noarch.rpm
    Centos6 32 位 : rpm -Uvh 'http://mirrors.ustc.edu.cn/fedora/epel/6/i386/epel-release-6-7.noarch.rpm'
    Centos6 64 位 : rpm -Uvh 'http://mirrors.ustc.edu.cn/fedora/epel/6/x86_64/epel-release-6-7.noarch.rpm
    
### 2.2 下载安装 yum 源
    [root@localhost src]# wget http://mirrors.ustc.edu.cn/fedora/epel/5/x86_64/epel-release-5-4.noarch.rpm
   
    [root@localhost src]# ls
    epel-release-5-4.noarch.rpm
    [root@localhost src]# rpm -ivh epel-release-5-4.noarch.rpm
    warning: epel-release-5-4.noarch.rpm: Header V3 DSA signature: NOKEY, key ID 217521f6
     Preparing...                ########################################### [100%]
     1:epel-release           ########################################### [100%]
    [root@localhost src]
    
### 2.3 安装 cobber
     安装软件 yum install cobbler tftp tftp-server xinetd dhcp httpd rsync 安装 cobbler [root@localhost src]# yum install pykickstart debmirror python-ctypes cman -y 安装 cobbler 所需的软件包

### 2.4 配置 cobbler 2.4.1 设置httpd服务
     [root@localhost src]# cat   /etc/httpd/conf.d/wsgi.conf
	# NOTE: mod_wsgi can not coexist in the same apache process as
	# mod_wsgi_python3.  Only load if mod_wsgi_python3 is not
	# already loaded.
	<IfModule !wsgi_module>
	LoadModule wsgi_module modules/mod_wsgi.so
	</IfModule>
	[root@localhost src]# /etc/init.d/httpd restart
	Stopping httpd:
	Starting httpd:
	[root@localhost src]# chkconfig httpd on
	# 取消前面的注释
	            [FAILED]
	            [  OK  ]
	            
###  2.4.2 设置tftp
		 [root@localhost src]# cat  /etc/cobbler/tftpd.template
		# default: off
		# description: The tftp server serves files using the trivial file transfer \
		#       protocol.  The tftp protocol is often used to boot diskless \
		#       workstations, download configuration files to network-aware printers, \
		#       and to start the installation process for some operating systems.
		service tftp
		{
		disable
		socket_type
		protocol
		wait
		user
		server
		server_args
		per_source
		cps
		flags
		=no ##将no改为no = dgram
		= udp
		= yes
		= $user
		= $binary
		= -B 1380 -v -s $args
		= 11
		= 100 2
		= IPv4
		}
	[root@localhost src]# cat  /etc/xinetd.d/rsync
	# default: off
	# description: The rsync server is a good addition to an ftp server, as it \
	#       allows crc checksu	
mming etc.
	service rsync
	
	  {
	disable = no
	socket_type
	wait
	user
	##将 yes 改为 no = stream
	= no
	= root
	server
	server_args
	log_on_failure  += USERID
	}
	[root@localhost src]# /etc/init.d/xinetd start [root@localhost src]# vim /etc/debmirror.conf 注释
	#@dists="sid";
	#@arches="i386";
	= /usr/bin/rsync
	= --daemon
### 2.4.3 创建用户密码
    [root@localhost src]# openssl passwd -1 -salt 'osyunwei' '123456' $1$osyunwei$sEV8iwXXuR4CqzLXyLnzm
	修改 vim /etc/cobber/settings
	default_kickstart: /var/lib/cobbler/kickstarts/default.ks default_password_crypted: "$1$osyunwei$sEV8iwXXuR4CqzLXyLnzm0" manage_dhcp: 1
	next_server: 192.168.209.128
	server: 192.168.209.128
	2.4.4 搭建dhcp服务
	 [root@localhost ~]# grep -v "#" /etc/cobbler/dhcp.template
	ddns-update-style interim;
	allow booting;
	allow bootp;
	ignore client-updates;
	set vendorclass = option vendor-class-identifier;
	option pxe-system-type code 93 = unsigned integer 16;
	subnet 192.168.209.0 netmask 255.255.255.0 {
	     option routers             192.168.209.1;
	     option domain-name-servers 8.8.8.8,8.8.4.4;
	option subnet-mask
	range dynamic-bootp
	filename
	255.255.255.0;
	192.168.209.100 192.168.209.254;
	"/pxelinux.0";
	
	  default-lease-time
	max-lease-time
	next-server
	}
	21600;
	43200;
	$next_server;
	[root@localhost ks_mirror]# cat /etc/sysconfig/dhcpd # Command line options here
	DHCPDARGS=eth0
	启动dhcp使用cobbersync 就可以启动dhcp
	同步 cobber sync 就可以加载 dhcp 服务启动了 确保这些服务都开启就可以进行测试安装了， /etc/init.d/xinted start /etc/init.d/httpd start /etc/init.d/cobber start

### 2.4.5 创建镜像站点
	挂载映像文件到 httpd 的站点目录
	mkdir -p /var/www/html/os/centos_x64_x86
	mount /dev/cdrom /media/
	rsync -avP /media/* /var/www/html/os/centos_x64_x86/
	使用 rsync 或者 scp
	cobbler import --path=/var/www/html/os/centos_x64_x86/ --name=centos_x64_x86 --arch=x86_64 导入镜像的命令参数
	命令格式:cobbler import --path=镜像路径 -- name=安装引导名 --arch=32 位或 64 位， 重复上面的操作， 把其他的系统镜像文件导入到 cobber 中，
	cobbler distro list 列出已经安装镜像

### 2.4.6 创建自动化安装脚本
	  创建自动化安装脚本
	进入到 kickstarts 模板目录
	cd /var/lib/cobbler/kickstarts
	[root@localhost kickstarts]# ls
	default.ks legacy.ks
	esxi4-ks.cfg pxerescue.ks
	esxi5-ks.cfg sample_autoyast.xml sample_esxi4.ks sample_old.seed
	/etc/init.d/cobblerd restart
	sample_end.ks    sample_esxi5.ks  sample.seed
	sample_esx4.ks   sample.ks
	
## 2.3 测试
    在这里就可以测试 pxe 引导安装了
 
    安装 kickstart
	yum install system-config-kickstart
	yum groupinstall "X Window System"
	startx 进入到图形界面生成 ks.cf 文件
	system-config-kickstart 运行生成 ks.cf 文件
	修改文件名称添加到 cobber 中
	[root@localhost ~]# cobbler profile add --name=centos_x64-x86_64 --distro=centos_x64-x86_64 --kickstart=/var/lib/cobbler/kickstarts/cenots5.8_x86_64.cfg
	exception on server: "it seems unwise to overwrite this object, try 'edit'"
	添加文件的命令参数
	命令:cobbler profile add|edit|remove --name=安装引导名 --distro=系统镜像名 --kickstart=kickstart 自动安装文件路径
	--name:自定义的安装引导名，注意不能重复 --distro:系统安装镜像名，用 cobbler distro list 可以查看 --kickstart:与系统镜像文件相关联的 kickstart 自动安装文件
## 2.4 cobbler 帮助
	[root@localhost ~]# man cobbler 查看 cobbler 帮助 cobbler list
	cobbler report
	cobbler profile report
	cobbler distro list
	
	  [root@localhost ~]# cobbler list
	distros:
	   centos_x64-x86_64
	   centos_x64-xen-x86_64
	profiles:
	   centos_x64-x86_64
	   centos_x64-xen-x86_64
	systems:
	repos:
	images:
	mgmtclasses:
	packages:
	files:
	[root@localhost ~]# cobbler distro list
	   centos_x64-x86_64
	   centos_x64-xen-x86_64
	[root@localhost ~]#
