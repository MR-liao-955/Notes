### Mailu 搭建记录

原由：公司QQ邮箱企业版不给用了，目前使用本地的服务器来接管邮件服务。



#### docker 环境搭建记录

- docker 本体的安装

  ubuntu 系统版本 （更新国内源就不再赘述）

  ![image-20230807181555731](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071816641.png)

  ```bash
  # 卸载原有Docker
  sudo apt-get remove docker docker-engine docker.io
  # 安装新dockers
  sudo apt-get update
  # 安装依赖包
  sudo apt-get install \
      apt-transport-https \
      ca-certificates \
      curl \
      software-properties-common
      
  # 添加官方密钥
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  
  # 添加仓库
  sudo add-apt-repository \
     "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) \
     stable"
  
  # 更新软件源
  sudo apt-get update
  
  # 执行安装程序
  sudo apt-get install docker-ce
  
  systemctl enable docker 
  systemctl start docker
  
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



#### Mailu 搭建记录

> mailu 官网填写信息并获取镜像下载地址

- https://setup.mailu.io/2.0/

- 关于mailu 证书的问题

  ![image-20230807181605818](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071816642.png)

  &emsp;&emsp;之前我选择的cert 选项，而且我并没有放置证书到相应地址，导致我的`mailu` 的web 界面没打开同时使得我采用了别的办法，比如`Postfix + Postfixadmin + sqlite + Dovecot `来搭建邮件服务器。使用postfix 搭建遇到配置上的问题，使得只能发邮件，而收件箱没找到邮件，它也没有收件箱管理的web ，因此很难排错。

  &emsp;&emsp;后来试了一下mailu 的letsencrypt 获取证书，没想到成功了。 [mailu官方文档](https://mailu.io/2.0/compose/setup.html#tls-certificates)，值得注意的是，mailu 官方的文档并不完全准确，后来我进docker 看`/config.py` 才回过神来是证书的问题

  ```bash
  docker exec -it mailu-front-1 bash
  cd /
  vi config.py
  ## 此时你会找到证书存放路径是容器里面的 /certs
  
  exit
  # 然后在docker-compose.yml  你会发现 这里的磁盘卷映射到容器内的 /certs 路径。就很能说明问题。
  官方给的文档的路径不是很清楚
  之后按照官方给的教程  把证书文件放在 宿主机 /mailu/certs
  ```

  ![image-20230807181610900](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071816643.png)

  ![image-20230807181617783](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071816644.png)

  ![image-20230807181625212](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071816645.png)

- 关于端口问题，如上图的ports 端口映射，该端口必须得是你宿主机的端口，而不是任意端口都能访问。尽量别设置成`0.0.0.0` ，防止局域网别的主机被黑导致邮件服务器出问题



> 安装docker && docker-compose
>
> [docker 安装教程](https://cloud.tencent.com/developer/article/2195198)



> 安装mailu 

[参考博客教程](https://qing.su/article/mail-hosting-with-mailu-io.html)

[mailu 官方docker 镜像下载路径生成地址](https://setup.mailu.io/2.0/)

[TLS证书 路径官方文档说明](https://mailu.io/2.0/compose/setup.html#tls-certificates)

```bash
# 设置域名 mail.xxxx.cn

# 证书选择 letsencrypt /certs 如果选certs 需要
# 在/mailu/certs 放置公钥cert.pem 私钥key.pem，
# 且私钥不能是原装的.key 格式

# 如果私钥为.key 格式那么通过openssh 转化其格式
openssl rsa -in privatekey.key -outform PEM -out key.pem

# Linked Website URL 尽量选择 mail.xxxx.cn 因为公司xxxx.cn 是官网，所以得用mail.xxxx.cn
# 实际上mail.xxxx.cn 也只是发邮件用， web 界面我们使用 https://xxxxx.xxxx.cn:xxxx 来访问

# 记得勾选 antivirus 和 webdav 服务，防病毒和web 界面

# ipv4 选择0.0.0.0 ,进去之后再该，不要信博客里说的写公网地址，因为直连docker 的根本就不是公网，
# 公司使用的是proxmox NAT 映射端口过来，而我的腾讯云内网和公网是隔开了的。

# wget 下载docker compose 和 mailu.env 文件之后就可以拉取镜像了
docker compose -p mailu up -d

# 建立初始化用户名和密码。
docker compose -p mailu exec admin flask mailu admin admin xxxx.cn PASSWORD

```



> 修改配置文件

- 修改本地端口

  ```bash
  ports:
      - "80:80"
      - "443:443"
      - "25:25"
      - "465:465"
      - "587:587"
      - "110:110"
      - "995:995"
      - "143:143"
      - "993:993"
  ```

  

- mailu 本体使用外置mysql 

  ```bash
  # 在mailu.env 文件中添加一行
  SQLALCHEMY_DATABASE_URI= mysql+mysqlconnector://mailu:xxxxxxxxxxxxx@172.16.xxx.xxx/mailu
  ```

- web 界面的docker 容器使用mysql

  官方参考文档

  [官方文档参考 roundcube 文档]( https://github.com/roundcube/roundcubemail/blob/master/config/defaults.inc.php#L28)

  ```bash
  # 进入webmail容器内部
  docker exec -it mailu-webmail-1 bash
  
  # 修改配置文件
  vi /var/www/roundcube/config/config.inc.php
  #数据库字段更改为
  $config['db_dsnw'] = 'mysql://mailu:xxxxxxxxxxxxxx@172.16.xxx.xxx/roundcube';
  
  # 保存退出  此时数据库中还没有表(暂时登录不了web页面)，需要设置数据库，请往后看。
  
  ```



> proxmox 公网地址映射

```bash
# 需要开放的端口
      - "80:80"
      - "443:443"
      - "25:25"
      - "465:465"
      - "587:587"
      - "110:110"
      - "995:995"
      - "143:143"
      - "993:993"
```

- 与Gitlab 服务器做隔离

  Gitlab: 使用的内网网段 172.16.xxx.xxx/24

  mailu: 使用的内网网段 172.16.xxx.xxx/24

  ```bash
  ################### 添加内网网卡 ###################
  # 方法1.修改配置文件 
  nano /etc/network/interfaces
  # 方法2.去proxmox web 页面添加 CIDR为172.16.xxx.xxx   (必须设置为桥接哈！！不然没网)
  CIDR 也就是子设备的网关地址
  
  # 如果proxmox 没有开启NAT 转发
  nano /etc/sysctl.conf
  #看看有没有如下行，如果没有就添加。
  net.ipv4.ip_forward=1
  #让它生效
  /sbin/sysctl -p
  
  ################### 添加NAT转发 ###################
  #### 此处默认之前搭建过Gitlab 服务器，因此大部分已经配置完，只需要转发端口就行
  # 从172.16.xxx.0/24 网段出来的走 vmbr0 接口
  iptables -t nat -A POSTROUTING -s '172.16.xxx.0/24' -o vmbr0 -j MASQUERADE
  
  #这两个特殊端口单独拧出来，80 就不开放了，没必要。443 该为xxxx
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 80 -j DNAT --to 172.16.xxx.xxx:80
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport xxxx -j DNAT --to 172.16.xxx.xxx:443
  
  
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 25 -j DNAT --to 172.16.xxx.xxx:25
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 465 -j DNAT --to 172.16.xxx.xxx:465
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 587 -j DNAT --to 172.16.xxx.xxx:587
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 110 -j DNAT --to 172.16.xxx.xxx:110
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 995 -j DNAT --to 172.16.xxx.xxx:995
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 143 -j DNAT --to 172.16.xxx.xxx:143
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 993 -j DNAT --to 172.16.xxx.xxx:993
  # 开放ssh 连接的端口
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport xxxx -j DNAT --to 172.16.xxx.xxx:22
  
  ```



> 迁移到mysql 数据库

- 由于mailu 内置的是sqlite 数据库类型，不适用于高并发，因此我们使用外部mysql 数据库。

  数据库直接放在宿主机，方便管理。

  ```bash
  # 安装mysql 服务
  apt-get install mysql-server-5.7 -y
  systemctl start mysql
  # 开机启动
  systemctl enable mysql
  
  # 初始化数据库
  sudo mysql_secure_installation
  
  ```
  
- 创建mysql 数据库

  ```mysql
  # 连接数据库
  mysql -uroot -p
  
  # 创建数据库
  CREATE DATABASE mailu;
  
  # 创建用户 
  CREATE USER 'mailu'@'%' IDENTIFIED BY '用户密码';
  # 赋予权限  后面把它改为localhost
  GRANT ALL PRIVILEGES ON mailu.* TO 'mailu'@'%' IDENTIFIED BY '用户密码';
  
  # 刷新权限
  FLUSH PRIVILEGES;
  
  # 创建ROUNDCUBE 数据库
  CREATE DATABASE roundcube;
  # 给权限
  GRANT ALL PRIVILEGES ON roundcube.* TO 'mailu'@'%' IDENTIFIED BY '用户密码';
  
  # 刷新权限
  FLUSH PRIVILEGES;
  
  
  ###### 测试完成 修改回数据库的权限     ####### 
  # 查询当前mailu 用户能用何种方式登录并访问数据库
  SELECT user, host FROM mysql.user WHERE user = 'mailu';
  # 改为特定IP 地址才能访问  改为docker 容器的IP 地址，别用localhost 不然访问不了
  
  ### 老老实实一个网段一个网段地写吧。。。
  192.168.203.1
  172.25.0.1
  172.26.0.1
  172.24.0.1
  172.17.0.1
  
  # 保证docker 的容器才能访问数据库。
  GRANT ALL PRIVILEGES ON mailu.* TO 'mailu'@'172.25.%.%' IDENTIFIED BY '用户密码';
  GRANT ALL PRIVILEGES ON mailu.* TO 'mailu'@'172.26.%.%' IDENTIFIED BY '用户密码';
  GRANT ALL PRIVILEGES ON mailu.* TO 'mailu'@'172.24.%.%' IDENTIFIED BY '用户密码';
  GRANT ALL PRIVILEGES ON mailu.* TO 'mailu'@'172.17.%.%' IDENTIFIED BY '用户密码';
  
  GRANT ALL PRIVILEGES ON roundcube.* TO 'mailu'@'172.25.%.%' IDENTIFIED BY '用户密码';
  GRANT ALL PRIVILEGES ON roundcube.* TO 'mailu'@'172.26.%.%' IDENTIFIED BY '用户密码';
  GRANT ALL PRIVILEGES ON roundcube.* TO 'mailu'@'172.24.%.%' IDENTIFIED BY '用户密码';
  GRANT ALL PRIVILEGES ON roundcube.* TO 'mailu'@'172.17.%.%' IDENTIFIED BY '用户密码';
  GRANT ALL PRIVILEGES ON roundcube.* TO 'mailu'@'192.168.203.%' IDENTIFIED BY '用户密码';
  GRANT ALL PRIVILEGES ON mailu.* TO 'mailu'@'192.168.203.%' IDENTIFIED BY '用户密码';
  
  
  # 撤销远程登录权限
  REVOKE ALL PRIVILEGES ON mailu.* FROM 'mailu'@'%';
  REVOKE ALL PRIVILEGES ON roundcube.* FROM 'mailu'@'%';
  
  
  # 查看用户权限
  SHOW GRANTS FOR 'mailu'@'%';
  
  # 刷新权限
  FLUSH PRIVILEGES;
  ###### 测试完成 修改回数据库的权限 END ####### 
  
  # 测试远程连接，如果连不上就说明正确
  mysql -h 服务器ip地址 -P 3306 -u root -p
  ```




> DNS 解析 (注意以下 markdown语法转义字符   xxxx.cn.\_report._dmarc 这一部分，的\  )

|        主机记录         | 记录类型 |            记录值             | 备注(优先级之类的) |
| :---------------------: | :------: | :---------------------------: | :----------------: |
|            @            |    MX    |         mail.xxxx.cn          |         10         |
|            @            |   TXT    | v=spf1 mx a:mail.xxxx.cn ~all |    这是SPF记录     |
|          mail           |    A     |        xxx.xxx.xxx.xxx        |                    |
|                         |          |                               |                    |
|         _dmarc          |   TXT    |       见下方代码块DMARC       |                    |
|     dkim._domainkey     |   TXT    |       见下方代码块DKIM        |                    |
| xxxx.cn.\_report._dmarc |   TXT    |           v=DMARC1            |                    |
|                         |          |                               |                    |
|                         |          |                               |                    |

- DKIM （rsa; 后面的是空格而不是回车）

  ```bash
  v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA3Lo0bUWvRKgnPZ后面还有一段删掉，
  ```

- DMARC

  ```bash
  v=DMARC1; p=reject; rua=mailto:admin@mail.xxxx.cn; ruf=mailto:admin@mail.xxxx.cn; adkim=s; aspf=s
  ```

  

> 密码之类的整理

- mysql

  ```bash
  
  ```



- ubuntu 虚拟机

  ```bash
  
  ```
  
  
  
- 邮件服务器地址

  ```bash
  
  ```
  
  
