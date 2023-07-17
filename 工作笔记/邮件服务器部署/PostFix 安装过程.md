### PostFix 搭建记录

原由：公司QQ邮箱企业版不给用了，目前使用本地的服务器来接管邮件服务。



#### 方案选择

1. 在原有的gitlab 所在的系统中安装postfix ，( 有一部分担忧端口映射，以及 服务的耦合，将来维护不方便。 )
2. 在proxmox 再安装一个ubuntu 系统，直接部署postfix ，(因为postfix 服务比较轻量化，单纯装一个系统来部署它有点浪费，)
3. 折中办法，proxmox 再装一个ubuntu，并使用docker 容器来进行管理，将来如果要开别的小型服务就在docker 之中安装









#### 搭建记录

- docker 的安装

  ubuntu 系统版本 （更新国内源就不再赘述）

  ![image-20230706104600219](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202307171022956.png)

  1. 安装docker 本体，`apt-get install docker docker-compose -y`
  2. 拉取postfix 镜像，



```bash
curl -L "https://github.com/docker/compose/releases/download/1.27.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#############
curl 是一个用于发送HTTP请求并下载文件的命令行工具。
-L 选项表示要跟随重定向，以确保正确获取下载文件。
"https://github.com/docker/compose/releases/download/1.27.2/docker-compose-$(uname -s)-$(uname -m)" 是要下载的文件的URL地址。其中，$(uname -s) 表示当前操作系统的名称（例如Linux、Darwin等），$(uname -m) 表示当前机器的架构（例如x86_64、armv7l等）。
-o /usr/local/bin/docker-compose 是指定下载的文件保存到/usr/local/bin/docker-compose路径下。
通过执行这个命令，将会从GitHub的发布页面下载相应版本的Docker Compose二进制文件，并保存到指定的路径。接下来，您可以通过运行docker-compose命令来使用安装的Docker Compose。

请确保您具有足够的权限来写入/usr/local/bin目录。如果没有足够的权限，您可能需要使用管理员权限或sudo命令来执行该命令。
#############


chmod +x /usr/local/bin/docker-compose





```

- docker-compose 的安装，我采用直接去github 下载镜像，然后传到服务器

  https://github.com/docker/compose/releases
  
  复制到
  
  ```c
  curl -fsSL https://get.docker.com | bash -s docker
  curl -L "https://github.com/docker/compose/releases/download/1.26.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  chmod +x /usr/local/bin/docker-compose
  ```
  
  实际地址 /usr/bin/docker-compose



```bash
// 配置NAT 映射 将8051 作为域名访问gitlab 的入口。6665设置为调试ubuntu 的入口
iptables -t nat -A POSTROUTING -s '172.16.1.11/24' -o vmbr0 -j MASQUERADE

iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 8051 -j DNAT --to 172.16.1.11:8051
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 6665 -j DNAT --to 172.16.1.11:22
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 6666 -j DNAT --to 172.16.1.11:80

iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 8268 -j DNAT --to 10.10.10.10:80


iptables -t nat -A POSTROUTING -s '10.10.10.11/24' -o vmbr0 -j MASQUERADE
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 3000 -j DNAT --to 10.10.10.11:22
```





```
docker run --name w-mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
```

```
docker run --name w-wordpress --link w-mysql:db -v /root/uploads.ini:/usr/local/etc/php/conf.d/uploads.ini  -p 80:80 -p 443:443 -d wordpress:latest
```



















- linux 和windows 下命令行尾格式不一致修改方法

```
sed -i 's/\r//' start.sh
```







sudo curl -L https://github.com/docker/compose/releases/download/v2.19.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose





#### 测试方案：

 使用FRP 和我的云服务器相连， 外网地址就是我的服务器地址，域名也是我的服务器域名，但是具体的邮件服务器设置在 虚拟机中，frp 只起到转发验证的作用



frp 使用mailu 依旧未能访问到网站，因此打算使用 docker-mailserver 来实现服务器搭建



需要使用到的端口

```bash

      - "192.168.2.11:80:80"
      - "192.168.2.11:443:443"
      - "192.168.2.11:25:25"
      - "192.168.2.11:465:465"
      - "192.168.2.11:587:587"
      - "192.168.2.11:110:110"
      - "192.168.2.11:995:995"
      - "192.168.2.11:143:143"
      - "192.168.2.11:993:993"


```





```

mailserver  | [  ERROR  ]  TLS Setup [SSL_TYPE=manual] | File /home/ssl/dearl.top/privkey.pem or /home/ssl/dearl.top/fullchain.pem does not exist!
mailserver  | [  ERROR  ]  Shutting down
mailserver  | 2023-07-13 07:10:59,539 WARN received SIGTERM indicating exit request
mailserver  | [   INF   ]  Welcome to docker-mailserver 12.1.0
mailserver  | [   INF   ]  Checking configuration
mailserver  | [   INF   ]  Configuring mail server
mailserver  | [  ERROR  ]  TLS Setup [SSL_TYPE=manual] | File /home/ssl/dearl.top/privkey.pem or /home/ssl/dearl.top/fullchain.pem does not exist!


```







```
// 记录


v=DMARC1; p=quarantine; rua=mailto:dmarc.report@dearl.top; ruf=mailto:dmarc.report@dearl.top; fo=0; adkim=r; aspf=r; pct=100; rf=afrf; ri=86400; sp=quarantine

```



```bash

root@VM-8-11-ubuntu:/home# certbot certonly --manual --preferred-challenge dns -d  mail.dearl.top
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for mail.dearl.top

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.mail.dearl.top with the following value:

NACw76dAuiwJIZ3xWUv_UaE7JBnEOrZ9NREUmo-PdCQ

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue


```





apt-get install php-imap php-mbstring php-fpm php-mysql php-sqlite3 php-cli php-common







#### mysql 的初始化

- 初始化mysql

  ```bash
  apt-get install mysql-server-5.7
  
  CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword';  // 所有ip 都可连接
  CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
  
  GRANT ALL PRIVILEGES ON *.* TO '用户名'@'主机';
  
  FLUSH PRIVILEGES;
  
  SHOW DATABASES;
  
  ```

  

- 创建数据表并赋予权限

  ```mysql
  grant all privileges on postfix.* to postfix@localhost identified by 'postfix';
  
  
  flush privileges;
  
  ```

  



#### 安装postfix



```bash
##/etc/postfix/mysql_virtual_alias_maps.cf
 user = postfixadmin
 password = postfixadmin
 hosts = 127.0.0.1
 dbname = postfixadmin
 table = alias
 select_field = goto
 where_field = address
 
 ##/etc/postfix/mysql_virtual_domains_maps.cf
 user = postfixadmin
 password = postfixadmin
 hosts = 127.0.0.1
 dbname = postfixadmin
 table = domain
 select_field = domain
 where_field = domain
 
 ##/etc/postfix/mysql_virtual_mailbox_maps.cf
 user = postfixadmin
 password = postfixadmin
 hosts = 127.0.0.1
 dbname = postfixadmin
 table = mailbox
 select_field = maildir
 where_field = username
```





- nginx 的问题

  ```bash
  
  # 当启动nginx 报错时，很有可能是端口被占用
  nginx -t  # 测一下nginx 配置文件是否有错
  
  #查看端口被占用情况，哪怕nginx 占用了端口都应该kill 掉
  sudo netstat -tulnp | grep -E '(:80|:443)' 
  
  # 清理掉占用端口的进程
  kill PID
  
  
  ```

  





postconf -e virtual_mailbox_domains=mysql:/etc/postfix/sql/mysql_virtual_domains_maps.cf

postmap -q dearl.top mysql:/etc/postfix/sql/mysql_virtual_domains_maps.cf




postconf -e virtual_mailbox_maps=mysql:/etc/postfix/sql/mysql_virtual_mailbox_maps.cf
postmap -q dearl@dearl.top mysql:/etc/postfix/sql/mysql_virtual_mailbox_maps.cf





postconf -e virtual_alias_maps=mysql:/etc/postfix/sql/mysql_virtual_alias_maps.cf
postmap -q dearl@dearl.top mysql:/etc/postfix/sql/mysql_virtual_alias_maps.cf





```bash




# 添加邮箱过时自动删除策略
vim /etc/dovecot/conf.d/15-mailboxes.conf
####################################################################################
namespace inbox {
  mailbox Drafts {
    special_use = \Drafts
  }
  mailbox Junk {
    special_use = \Junk
    autoexpunge = 30d
  }
  mailbox Trash {
    special_use = \Trash
    autoexpunge = 30d
  }
  mailbox Sent {
    special_use = \Sent
  }
  mailbox "Sent Messages" {
    special_use = \Sent
  }
}
####################################################################################


# 授权







作者: ZhongJin
链接: https://zhongjin.io/2022/09/03/IT%E6%8A%80%E6%9C%AF/%E8%BF%90%E7%BB%B4/%E5%9F%BA%E7%A1%80%E8%AE%BE%E6%96%BD/%E9%82%AE%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%90%AD%E5%BB%BA%E4%B9%8BPostfix-Dovecot-Postfixadmin/index.html
来源: ZhongJin's Blog
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

```



useradd -r -u 5000 -g vmail -d /opt/vmail -s /sbin/nologin -c "Virtual Mail User" vmail





- 日志文件保存在 /varl/log/mail.log 中







