#centos 6 下postfix 配置
1. 使用虚拟用户发送邮件
   创建虚拟用户  
    

  1.2 vim /etc/postfix/main.cf
   myhostname = mail.palmjob.net
   mydomain = palmjob.net
   myorigin = $mydomain
   inet_interfaces = all
   inet_protocols = all
   mynetworks = 127.0.0.0/8
    
   #smtp 认证
   smtpd_sasl_auth_enable = yes
   broken_sasl_auth_clients = yes
   smtpd_sasl_type = dovecot
   smtpd_sasl_path = private/auth
   smtpd_sasl_security_options = noanonymous
   smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated,reject_unauth_destination
   
   #虚拟用户认证
     virtual_mailbox_domains = proxy:mysql:/etc/postfix/sql/mysql_virtual_domains_maps.cf
     virtual_alias_maps =
         proxy:mysql:/etc/postfix/sql/mysql_virtual_alias_maps.cf,
         proxy:mysql:/etc/postfix/sql/mysql_virtual_alias_domain_maps.cf,
	 proxy:mysql:/etc/postfix/sql/mysql_virtual_alias_domain_catchall_maps.cf
     virtual_mailbox_maps =
	 proxy:mysql:/etc/postfix/sql/mysql_virtual_mailbox_maps.cf,
	 proxy:mysql:/etc/postfix/sql/mysql_virtual_alias_domain_mailbox_maps.cf
		  
     virtual_mailbox_base = /data/mail/
     virtual_create_maildirsize = yes
     virtual_mailbox_extended = yes
     virtual_mailbox_limit_maps = mysql:/etc/postfix/sql/mysql_virtual_mailbox_limit_maps.cf
     virtual_mailbox_limit_override = yes
     virtual_maildir_limit_message = Sorry, the user's maildir has overdrawn his diskspace quota, please try again later.
     virtual_overquota_bounce = yes
     virtual_uid_maps = static:1000
     virtual_gid_maps = static:1000
     virtual_transport = virtual
   
   
   2. dovecot 配置
     listen = *
     protocols = imap pop3
     disable_plaintext_auth = no
     ssl = no
     auth_mechanisms = plain login
     mail_access_groups = vmail
     default_login_user = vmail
     first_valid_uid = 1000
     first_valid_gid = 1000
     mail_location = maildir:/data/mail/%d/%n

     userdb {
        driver = sql
        args = /etc/dovecot/dovecot-sql.conf
      }

      passdb {
         driver = sql
         args = /etc/dovecot/dovecot-sql.conf
      }

       service auth {
           unix_listener /var/spool/postfix/private/auth {
           group = postfix
           mode = 0660
           user = postfix
       }
         user = root
       }
        service imap-login {
        process_min_avail = 1
        user = vmail
   
      }  
      
      
  3. 启动服务
    /etc/init.d/postfix  restart 
    /etc/init.d/dovecot   restart 
    
  
  4.安装postfixadmin 
  
  
  
      
         
