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





#### Mailu 搭建记录



> mailu 官网填写信息并获取镜像下载地址

- https://setup.mailu.io/2.0/

- 关于mailu 证书的问题

  ![image-20230720111005846](PostFix%20%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B.assets/image-20230720111005846.png)

  &emsp;&emsp;之前我选择的cert 选项，而且我并没有放置证书到相应地址，导致我的`mailu` 的web 界面没打开同时使得我采用了别的办法，比如`Postfix + Postfixadmin + sqlite + Dovecot `来搭建邮件服务器。使用postfix 搭建遇到配置上的问题，使得只能发邮件，而收件箱没找到邮件，它也没有收件箱管理的web ，因此很难排错。

  &emsp;&emsp;后来试了一下mailu 的letsencrypt 获取证书，没想到成功了。 [mailu官方文档](https://mailu.io/2.0/compose/setup.html#tls-certificates)，值得注意的是，mailu 官方的文档并不完全准确，后来我进docker 看`/config.py` 才回过神来是证书的问题

  ```bash
  docker exec -it mailu-front-1 bash
  cd /
  vi config.py
  ## 此时你会找到证书存放路径是容器里面的 /certs
  
  exit
  # 然后在docker-compose.yml  你会发现 这里的磁盘卷映射到容器内的 /certs 路径。就很能说明问题。
  说明官方给的文档的路径不是很清除
  之后按照官方给的教程  把证书文件放在 宿主机 /mailu/certs
  ```

  ![image-20230720112331103](PostFix%20%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B.assets/image-20230720112331103.png)

  ![image-20230720112410162](PostFix%20%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B.assets/image-20230720112410162.png)

  ![](PostFix%20%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B.assets/image-20230720112122082.png)

- 关于端口问题，如上图的ports 端口映射，该端口必须得是你宿主机的端口，而不是任意端口都能访问。尽量别设置成`0.0.0.0` ，防范 有坏人破解。









```
docker compose -p mailu exec admin flask mailu admin admin qiot.cn PASSWORD
```



> 安装docker && docker-compose
>
> [docker 安装教程](https://cloud.tencent.com/developer/article/2195198)



> 安装mailu 







> 修改docker-compose 配置文件







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

  Gitlab: 使用的内网网段 172.16.1.0/24

  mailu: 使用的内网网段 172.16.2.0/24

  ```bash
  ################### 添加内网网卡 ###################
  # 方法1.修改配置文件 
  nano /etc/network/interfaces
  # 方法2.去proxmox web 页面添加 CIDR为172.16.2.254   (必须设置为桥接哈！！不然没网)
  CIDR 也就是子设备的网关地址
  
  # 如果proxmox 没有开启NAT 转发
  nano /etc/sysctl.conf
  #看看有没有如下行，如果没有就添加。
  net.ipv4.ip_forward=1
  #让它生效
  /sbin/sysctl -p
  
  
  
  ################### 添加NAT转发 ###################
  #### 此处默认之前搭建过Gitlab 服务器，因此大部分已经配置完，只需要转发端口就行
  # 从172.16.2.0/24 网段出来的走 vmbr0 接口
  iptables -t nat -A POSTROUTING -s '172.16.2.0/24' -o vmbr0 -j MASQUERADE
  
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 80 -j DNAT --to 172.16.2.11:80
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 443 -j DNAT --to 172.16.2.11:443
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 25 -j DNAT --to 172.16.2.11:25
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 465 -j DNAT --to 172.16.2.11:465
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 587 -j DNAT --to 172.16.2.11:587
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 110 -j DNAT --to 172.16.2.11:110
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 995 -j DNAT --to 172.16.2.11:995
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 143 -j DNAT --to 172.16.2.11:143
  iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 993 -j DNAT --to 172.16.2.11:9
  ```





      - "80:80"
      - "443:443"
      - "25:25"
      - "465:465"
      - "587:587"
      - "110:110"
      - "995:995"
      - "143:143"
      - "993:993"









> 迁移到mysql 数据库

- 由于mailu 内置的是sqlite 数据库类型，不适用于高并发，因此我们使用外部mysql 数据库。

  数据库直接放在宿主机，方便管理。

  ```bash
  
  
  
  
  
  ```

  







 





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

  


