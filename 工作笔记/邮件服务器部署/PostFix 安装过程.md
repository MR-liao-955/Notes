### PostFix 搭建记录

原由：公司QQ邮箱企业版不给用了，目前使用本地的服务器来接管邮件服务。



#### 方案选择

1. 在原有的gitlab 所在的系统中安装postfix ，( 有一部分担忧端口映射，以及 服务的耦合，将来维护不方便。 )
2. 在proxmox 再安装一个ubuntu 系统，直接部署postfix ，(因为postfix 服务比较轻量化，单纯装一个系统来部署它有点浪费，)
3. 折中办法，proxmox 再装一个ubuntu，并使用docker 容器来进行管理，将来如果要开别的小型服务就在docker 之中安装









#### 搭建记录

- docker 的安装

  ubuntu 系统版本 （更新国内源就不再赘述）

  ![image-20230706104600219](PostFix%20%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B.assets/image-20230706104600219.png)

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







```
docker run --name w-mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
```

```
docker run --name w-wordpress --link w-mysql:db -v /root/uploads.ini:/usr/local/etc/php/conf.d/uploads.ini  -p 80:80 -p 443:443 -d wordpress:latest
```





































