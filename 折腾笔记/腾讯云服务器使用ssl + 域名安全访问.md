### 腾讯云服务器使用ssl + 域名安全访问

#### 重新修改服务器设置

![image-20230621160715465](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306211607281.png)

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





#### 端口迁移

docker 端口重新映射之后导致图片访问不了。根据chatGPT 的方案，进行数据库的修改最终解决图片重命名的问题。( 注意：用该方法之前请务必备份 )

1. 安装插件

   ![image-20230703181634951](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/image-20230703181634951.png)

2. 修改数据表的内容（wp_posts 表中存放的是博客的文字性内容，以及别的。wp_postmeta 表中存放的可能是索引之类的）chatGPT 建议修改这2个表的内容。( 下方那个选项框是查找，如果取消勾选就是替换 )

   ![image-20230703181817610](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/image-20230703181817610.png)

3. 此时图片就能正常显示了，但是还有一部分是博客依旧是ip地址显示，因此我们要去管理台设置新站点地址。

   ![image-20230703182105802](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/image-20230703182105802.png)















#### 使用certbot 申请ssl 证书

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





#### Nginx 部分 把端口从80 映射到 5000

- 设置5000端口为sll 访问同时80端口映射过去

  ```bash
  
      listen [::]:443 ssl ipv6only=on; # managed by Certbot
      listen 443 ssl; # managed by Certbot
      ssl_certificate /etc/letsencrypt/live/dearl.top/fullchain.pem; # managed by Certbot
      ssl_certificate_key /etc/letsencrypt/live/dearl.top/privkey.pem; # managed by Certbot
      include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
      ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
  
  
  
  }
  
  
  
  
  
  
  ```

  



#### Docker 下的w-wordpress w-nextcloud容器开启ssl







