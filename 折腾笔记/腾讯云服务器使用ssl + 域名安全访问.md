### 腾讯云服务器使用ssl + 域名安全访问

#### 重新修改服务器设置

> 使用前先将wordpress 端口从80挪开

- 原因：宿主机需要安装nginx 用来申请ssl。而之前wordpress 服务把80端口占用了。需要将它移走。

- 遇到的问题：
  1. docker 配置端口映射问题( 宿主机5000-> 容器80 )
  
     ```react
     // 先暂停容器
     docker stop w-wordpress
     
     // 进入容器所在的文件夹并修改端口映射
     cd /var/lib/docker/containers/36e3e45f529e60050287a3526b5152668b68efb5f28c51a80147de7fc901b33a
     
     // 修改hostconfig.json 与 config.v2.json 文件
     vim config.v2.json
     // 使用 `/` 查找 `ExposedPort` 并修改里面的端口
     "ExposedPorts":{"80/tcp":{}},  // 左边的就是容器内要暴露的端口。
         
         
     
     
     ```
  
     
  
  2. 端口访问wordpress 界面会被重新指向80端口。



> 配置nginx 指向服务。并设置ssl 

- 强制映射80 端口到443，保障https 的安全性





> 将申请的ssl 证书 映射到服务器里面( 共享文件夹的方式 )

- wordpress 容器的处理
  1. 进入 wordpress 容器  `docker exec -it w-wordpress bash` 
  2. 创建ssl 映射的文件  `mkdir ./ssl`
- 宿主机的处理
  1. `docker container update --mount-add type=bind,source=/etc/letsencrypt/live/dearl.top,target=/var/www/html/ssl w-wordpress`









> 配置wordpress 的apache2 的ssl ，配置nextcloud 的ssl

[重磅！参考地址](https://www.jianshu.com/p/e8ae8bb1ad0a)



#### 端口迁移（已完成）

docker 端口重新映射之后导致图片访问不了。根据chatGPT 的方案，进行数据库的修改最终解决图片重命名的问题。( 注意：用该方法之前请务必备份 )

1. 安装插件

   ![image-20230703181634951](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/image-20230703181634951.png)

2. 修改数据表的内容（wp_posts 表中存放的是博客的文字性内容，以及别的。wp_postmeta 表中存放的可能是索引之类的）chatGPT 建议修改这2个表的内容。( 下方那个选项框是查找，如果取消勾选就是替换 )

   ![image-20230703181817610](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/image-20230703181817610.png)

3. 此时图片就能正常显示了，但是还有一部分是博客依旧是ip地址显示，因此我们要去管理台设置新站点地址。

   ![image-20230703182105802](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/image-20230703182105802.png)







#### 使用certbot 申请ssl 证书（完成度80% 还差自动续签）

```react

root@VM-8-11-ubuntu:/etc/nginx/sites-available# sudo certbot --nginx -d dearl.top -d www.dearl.top
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for dearl.top
http-01 challenge for www.dearl.top
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/default
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/default

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/default
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/default

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://dearl.top and
https://www.dearl.top

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=dearl.top
https://www.ssllabs.com/ssltest/analyze.html?d=www.dearl.top
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/dearl.top/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/dearl.top/privkey.pem
   Your cert will expire on 2023-09-12. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

root@VM-8-11-ubuntu:/etc/nginx/sites-available# 

docker container update --mount-add type=bind,source=/etc/letsencrypt/live/dearl.top,target=/var/www/html/ssl w-wordpress
```





#### Nginx 部分 把端口从80 映射到 5000( 已完成 )

- 设置5000端口为sll 访问同时80端口映射过去

  ```bash
  
  
  
  
  
  ```
  
  



#### Docker 下的w-wordpress w-nextcloud容器开启ssl（施工中）

- docker 容器内修改软件源并安装vim 编辑器

  ```bash
  // 查看系统的镜像内容
  root@kuboard-5967d77d89-h2hgn:/# cat /etc/issue
  Debian GNU/Linux 10 \n \l
  
  // 更新软件源 apt update (国外源，慢，但是没办法)
  
  
      
  // 主机地址
  ssl_certificate /etc/letsencrypt/live/dearl.top/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/dearl.top/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
  
  
  # 容器内地址
  /var/www/html/my_ssl/fullchain.pem
  /var/www/html/my_ssl/privkey.pem
  
  
  # 容器间拷贝
  docker cp /etc/letsencrypt/live/dearl.top/fullchain.pem w-wordpress:/
  
  docker cp /etc/letsencrypt/live/dearl.top/fullchain.pem w-wordpress:/
  
  修改的地址： /etc/apache2/sites-available/default-ssl.conf
  ```

  【该方法我只在测试ubuntu 上完成过，公钥和私钥都是阿里云等申请的正式公私钥】

![image-20230706152429354](%E8%85%BE%E8%AE%AF%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%BF%E7%94%A8ssl%20+%20%E5%9F%9F%E5%90%8D%E5%AE%89%E5%85%A8%E8%AE%BF%E9%97%AE.assets/image-20230706152429354.png)





- 目前想到的合适的办法

  外层Nginx 作为443端口的接收，将接收到的请求通过http 转发给docker 的wordpress 容器的apache2 的80端口。

  也就是：外部使用https 加密，内部使用http

  ```bash
  // gitlab曾用过的nginx 设置
  server {
      listen       8022;  #原作者的 gitlab 一般使用 8022 端口访问
      server_name  localhost;
  
      location / {
          root  html;
          index index.html index.htm;
          proxy_pass http://127.0.0.1:8021; #这里与前面设置过的端口一致
      }
  }
  // 设置nginx 代理到目标地址
  
  
  
  
  ```

  

















### 遇到的问题：

- 如果在网页后台站点地址被误操作更改。那就只能去服务器修改数据库的内容了。

  > 参考地址：https://cuijiahua.com/blog/2017/10/website_1.html

  `docker exec -it w-mysql bash`   ==★== 进入docker 容器

  `USE wordpress;`   ==★== 使用wordpress 数据库

  `select * from wp_options limit 20;`  ==★== 查找数据表

  ![image-20230705113717048](%E8%85%BE%E8%AE%AF%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%BF%E7%94%A8ssl%20+%20%E5%9F%9F%E5%90%8D%E5%AE%89%E5%85%A8%E8%AE%BF%E9%97%AE.assets/image-20230705113717048.png)

  `UPDATE wp_options SET option_value="http://dearl.top:5000" WHERE option_name="siteurl";` ==★== 修改数据表内容

```bash

# temp

UPDATE wp_options SET option_value="https://gitlab.qiot.cn:8269" WHERE option_name="siteurl";


```



- https 证书绑定完成，但是



- Nginx 学习

  ```c
  
  "ExposedPorts":{"80/tcp":{},"443/tcp":{}},
  
  
  {"80/tcp":[{"HostIp":"","HostPort":"5000"}],"443/tcp":[{"HostIp":"","HostPort":"4999"}]}
  
  
  ```
  
  



![image-20230710161419274](%E8%85%BE%E8%AE%AF%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%BF%E7%94%A8ssl%20+%20%E5%9F%9F%E5%90%8D%E5%AE%89%E5%85%A8%E8%AE%BF%E9%97%AE.assets/image-20230710161419274.png)



- 更新docker 版本
  1. 先备份/var/lib/docker  --> /var/lib/docker-bak
  2. https://cloud.tencent.com/developer/article/2195198   安装新版docker-ce
  3. https://www.jianshu.com/p/9261f29ea64a   处理掉使用docker 命令的报错异常
  4. https://zhuanlan.zhihu.com/p/422427865    恢复原来镜像









