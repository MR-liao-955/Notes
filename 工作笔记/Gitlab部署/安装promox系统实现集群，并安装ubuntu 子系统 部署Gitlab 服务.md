# 安装promox系统实现集群，并安装ubuntu 子系统 部署Gitlab 服务

### 下载并安装promox

> 参考教程
>
> https://www.bilibili.com/video/BV1hh411S71J/?vd_source=c2d05182ffbc2da978ff445af107c7ef

- 就当VMware 使用就行了

```makefile

#查看ip地址
# 此文件为web管理界面的登录地址
cat /etc/issue
#此文件为主机名的配置文件
cat /etc/hosts
#此文件为主机IP地址的配置文件
cat /etc/network/interfaces
#修改ip地址
vi /etc/issue
vi /etc/hosts
vi /etc/network/interfaces
#重启接口和web管理服务
/etc/init.d/networking restart
service pveproxy restart



```



- promox 内启动ubuntu 程序错误时

  参考解决方案：https://blog.csdn.net/zhanremo3062/article/details/115669540

  解决方案2：intel-VTX 启动失败  https://blog.csdn.net/qq_46499134/article/details/124231658

  ![image-20230518175749595](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161738463.png)
  
  ```rust
  去VMware 开启虚拟化
  
  但是需要先开启BIOS 主板的虚拟化才行 这是intel VT 这一部分
  
  
  ```
  
  ![image-20230518175944252](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161738464.png)

- ssh 配置

  ```rust
  
  // 安装ssh(如果没安装)
  sudo apt-get install openssh-server
  
  // 查看ssh 状态
  service ssh status
  
  // 如果没启动，启动一下
  systemctl enable ssh
  systemctl enable sshd
  
  // 如果有ufw 就检查一下
  sudo ufw allow ssh 
  
  // 配置sshd_config
  vim /etc/ssh/sshd_config
  
  /*
  打开 port 22 
  允许密码登录 PasswordAuthentication yes
  允许root用户名登录 PermitRootLogin yes
  */
  
  //重启ssh
  systemctl restart sshd
  
  ```

  

### Ubuntu 部署gitlab

#### 准备工作

- 更换软件源

  ```bash
  //安装vim 
  sudo apt-get install vim -y
  
  //进入etc/apt/sources.list 文件并修改软件源 为国内源
  vim /etc/apt/sources.list
  
  //复制粘贴下方阿里云源
  deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
  ##測試版源
  deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
  # 源碼
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
  ##測試版源
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
   
  
  
  ```
  
  [参考地址（华为云建站文档）](https://support.huaweicloud.com/bestpractice-ecs/zh-cn_topic_0165501097.html)
  
  [参考地址2](https://zhuanlan.zhihu.com/p/577082732)
  
  

### iSCSI 客户端

- 安装 配置 iSCSI 客户端 （暂时跳过这一节，因为没iSCSI 服务器测试 这里做的是客户端）

  iSCSI 类似于SMB 协议，但是iSCSI 有一个优点是，可以把网络磁盘映射成真正的本地磁盘，使用起来和本地磁盘一样

  [iSCSI 客户端配置地址](https://blog.51cto.com/cerana/5725318)

  ```
  apt install open-iscsi
  ```





#### 安装 Nginx Passenger(管理工具) Nodejs Yarn(包管理工具)

- 准备工作

  当依赖源报错时，首先考虑是软件源版本对不上，然后修改软件源

  >*最终解决办法： 打开https://oss-binaries.phusionpassenger.com/apt/passenger 找到你对应的内核的代号，14.04--Trusty  16.04--Xenial  18.04--Bionic*
  >https://releases.ubuntu.com/  这里会有对应内核代号的
  >
  >![image-20230519092616115](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161738465.png)

  ```bash
  // 密钥& 证书之类
  sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
  
  sudo apt-get install -y apt-transport-https ca-certificates
  
  //添加新的源   -- 这行配置 会出现依赖问题，建议使用
  
  /*最终解决办法： 打开https://oss-binaries.phusionpassenger.com/apt/passenger 找到你对应的内核的代号，14.04--Trusty  16.04--Xenial  18.04--Bionic*/
  //https://releases.ubuntu.com/  这里会有对应内核代号的
  
  /*
  我在Ubuntu 14.10上遇到了相同的问题
  sudo nano /etc/apt/sources.list.d/passenger.list
  deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main
  */
  
  sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bionic main > /etc/apt/sources.list.d/passenger.list'
  
  sudo apt-get update
  
  
  
  ```

  

- 安装Passenger 和 Nginx

  ```bash
  // 解决依赖问题之后 安装 nginx-extras & passenger
  sudo apt-get install nginx-extras passenger
  
  // 修改nginx 的路由配置
  sudo vim /etc/nginx/nginx.conf
  /*
  1.第一行添加
  env PATH;
  2.取消注释(但是我的环境中没有这行,所以我手动添加)
  include /etc/nginx/passenger.conf;
  */
  
  // 允许Nginx 的SSL 
  /* 使用openssl 工具
  1.创建路径 & 分配权限
  sudo mkdir -p /etc/ssl/evolute.in
  sudo chmod 700 /etc/ssl/evolute.in
  2.生成rsa 密钥&公钥
  sudo openssl genrsa -out /etc/ssl/evolute.in/ca.key 2048
  3.配置新的密钥
  sudo openssl req -new -key /etc/ssl/evolute.in/ca.key -out /etc/ssl/evolute.in/ca.csr
  
  4.可能是nodejs 服务端部分的密钥
  sudo openssl req -nodes -newkey rsa:2048 -keyout /etc/ssl/evolute.in/server.key -out /etc/ssl/evolute.in/ca.csr
  
  5.添加密钥
  sudo openssl x509 -req -days 3650 -in /etc/ssl/evolute.in/ca.csr -signkey /etc/ssl/evolute.in/server.key -out /etc/ssl/evolute.in/server.crt
  
  6.删除ca.csr  赋予权限server.crt  server.key
  sudo rm -v /etc/ssl/evolute.in/ca.csr
  sudo chmod 600 /etc/ssl/evolute.in/server.crt
  sudo chmod 600 /etc/ssl/evolute.in/server.key
  
  */
  
  
  //enable ssl on Nginx (后面再看nginx是否缺少这部分呢)
  
  
  
  
  
  ```

  遇到依赖问题

  [ Passenger & Nginx 依赖问题参考 (谢鸣)](https://www.codenong.com/28818597/)

  配置openssl 信息

  ![image-20230514215905643](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161738466.png)



- 安装 Node.js 和 yarn

  ```bash
  // 添加命令
  $ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
  $ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
  $ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
  
  //安装
  sudo apt-get update
  sudo apt-get install nodejs yarn
  
  
  ```



- 配置虚拟主机 LDAP Account Manager   （LDAP 类似一个 目录系统）

  ```rust
  sudo vi /etc/nginx/sites-available/lam.evolute.in
  /*****************************************/
  // nginx脚本内容如下
  server {
    listen *:80;
    server_name lam.evolute.in;
    access_log /data/nginx/log/lam-access.log; 
    error_log /data/nginx/log/lam-error.log;
    location / {
      proxy_pass https://192.168.1.32/lam;
      
  proxy_redirect https://192.168.1.32/lam http://lam.evolute.in/; 
      proxy_set_header X-Real-IP $remote_addr;
  
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    }
  
    error_page 500 502 503 504 /50x.html;
  
    location = /50x.html {
      root html;
    }
  }
  
  server {
    listen *:443 ssl;
    
  server_name lam.evolute.in;
    access_log /data/nginx/log/lam-access.log; 
    error_log /data/nginx/log/lam-error.log;
  
    location / {
      proxy_pass https://192.168.1.32/lam/;
      
  proxy_redirect https://192.168.1.32/lam/ https://lam.evolute.in/; 
      proxy_set_header X-Real-IP $remote_addr;
      
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
      }
  
    error_page 500 502 503 504 /50x.html;
    
  location = /50x.html {
      root html;
    }
  }
  
  /***************************************************/
  
  // 将脚本链接至nginx 目录
  cd /etc/nginx/sites-enabled
  
  sudo ln -s /etc/nginx/sites-available/lam.evolute.in lam.evolute.in
  
  ```

- 设置自身服务器的虚拟主机密码

  ```rust
  sudo vi /etc/nginx/sites-available/ssp.evolute.in
  
  /*******************************/
  server {
  	listen *:80;
  	server_name ssp.evolute.in;
  	access_log /data/nginx/log/ssp-access.log;
      error_log /data/nginx/log/ssp-error.log;
      location / {
          proxy_pass https://192.168.1.34;
          proxy_redirect https://192.168.1.34/ssp http://ssp.evolute.in/; 					proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
      error_page 500 502 503 504 /50x.html;
      location = /50x.html {
          root html;
      }
  }
  
  server {
      listen *:443 ssl;
  	server_name ssp.yinghad.com;
      
  	access_log /data/nginx/log/ssp-access.log;
      error_log /data/nginx/log/ssp-error.log;
      
  	location / {
          proxy_pass https://192.168.1.32/ssp/;
  		proxy_redirect https://192.168.1.32/ssp/ https://ssp.evolute.in/; 					proxy_set_header X-Real-IP $remote_addr;
  		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
      }
      error_page 500 502 503 504 /50x.html;
  	location = /50x.html {
          root html;
      }
  }
  
  
  /*******************************/
  
  //链接脚本至Nginx 
  $ cd /etc/nginx/sites-enabled
  $ sudo ln -s /etc/nginx/sites-available/ssp.evolute.in ssp.evolute.in
  
  
  
  ```





#### 安装 PostgreSQL 服务（先跳过，直接安装Gitlab）

gitlab 有内置PostgreSQL ,不过也能设置外部的服务





#### 安装Gitlab 

```rust
// 参考地址（亲测有效）： https://www.seasidecrab.com/server/991.html
curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb"

```

https://blog.csdn.net/u012360727/article/details/125357347

强力推荐https://www.bilibili.com/video/BV15g4y1W7FA/?vd_source=c2d05182ffbc2da978ff445af107c7ef

nginx & gitlab配置 http://www.manongjc.com/detail/54-jdzpthphqzjczxx.html

https://wiki.op81.com/pages/2f1bae/#_5-%E5%90%AF%E5%8A%A8gitlab

```rust
server {
      listen 86;
      server_name gitlab.gogobanking.com;
      location / {
          client_max_body_size 1024m;
          proxy_redirect off;
          #以下确保 gitlab中项目的 url 是域名而不是 http://git，不可缺少
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          # 反向代理到 gitlab 内置的 nginx
          proxy_pass http://127.0.0.1:6666;
          index index.html index.htm;
        }
    }
```

![image-20230515013513457](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161738467.png)

- 目前遇到的困难，nginx 解析脚本的问题，启动问题，





#### promox 使用xshell 访问ubuntu 

```rust
//1. 在promox web管理界面 打开ubuntu 然后直到它的 ip 地址。ifconfig  192.168.2.155
//    promox 中xshell  输入命令
ssh dearl@192.168.2.155 

//如果要退出 在ssh 里面使用 退出内部ssh
exit 

/*
	账户 dearl
	密码 1 
	
*/
//2 进入再用 su 进入管理员权限。 密码1

/*   安装好 gitlab-ce 之后 配置nginx   */
https://www.seasidecrab.com/server/939.html

// 关闭 gitlab 内置的nginx 

##禁用捆绑的 Nginx
# 将
nginx['enable'] = true
# 修改为
nginx['enable'] = false
# 并去掉注释 (前边的#)
...
##设置 gitlab-workhorse 监听 TCP 端口
gitlab_workhorse['listen_network'] = "tcp"
gitlab_workhorse['listen_addr'] = "127.0.0.1:8021"  //这个端口号和之后设置的 Nginx 代理的端口号要一致



// 查找nginx 位置
whereis nginx

cd /etc/nginx

sudo vim /etc/nginx/sites-available/gitlab.evolute.in

/*  配置代理
server {
    listen       8022;  #原作者的 gitlab 一般使用 8022 端口访问
    server_name  localhost;

    location / {
        root  html;
        index index.html index.htm;
        proxy_pass http://127.0.0.1:8021; #这里与前面设置过的端口一致
    }
}
*/
// 将配置文件连接到 promox 的nginx 中

cd /etc/nginx/sites-enabled

sudo ln -s /etc/nginx/sites-available/gitlab.evolute.in gitlab.evolute.in

// 重启Nginx 和 重置 gitlab-reconfig

systemctl restart nginx
sudo gitlab-ctl reconfigure

// 访问 
http://192.168.2.66:8022

// 初始化 用户名root 密码在
/etc/gitlab/initial_root_password

////////杂项////////////
//端口查看
netstat -tunlp


/*
修改了 username:root
	  password:dearl1234

*/
```



- gitlab 添加ssh-key 之后依然要输入密码的解决办法：（还是得配置ssh）

  https://cloud.tencent.com/developer/article/1671794

  https://blog.hobairiku.site/2018/02/26/gitlab-setup/#5-ssh%E7%AB%AF%E5%8F%A3%E4%BF%AE%E6%94%B9

  修改ssh 监听端口。通过git 并非访问22端口。

- gitlab 点 “ 我的工作 ”时会跳转到127.0.0.1:8001 这个地址。（需要修改yml 文件，静态页面跳转）亲测无效

  http://www.cppblog.com/jack-wang/archive/2019/06/21/216440.aspx
  
  - 已解决：因为 外部nginx 反向代理到本地端口。"我的工作" 指向的地址就是外部nginx 代理的地址"127.0.0.1:8021"。因此，目前暂时关闭了外部nginx 使用gitlab 自带







> 参考地址

[centos 通过 yum 安装gitlab](https://www.bilibili.com/video/BV11G4y1D7Gi/?vd_source=c2d05182ffbc2da978ff445af107c7ef)

[官方教程 ubuntu 安装gitlab runner](https://www.bilibili.com/video/BV1ob4y1Y75U/?vd_source=c2d05182ffbc2da978ff445af107c7ef)

[教程](https://mp.weixin.qq.com/s/dLgXthn1oKX4i792qljsZg)

























