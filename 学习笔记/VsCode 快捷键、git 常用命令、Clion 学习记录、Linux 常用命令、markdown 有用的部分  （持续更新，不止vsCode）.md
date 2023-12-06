#### **vscode 常用快捷键**
&emsp;&emsp;  vscode 快捷键， 如果要连续修改一个东西，可以找个相似的，然后ctrl+d 快速选择，同时进行添加
  1. `Ctrl+D` :快速选择，可以在一个字符串中选择后面多个相同的目标，同时进行修改。很常用

     ![image-20230731141336436](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202307311413502.png)

     而且这个快捷键，可以选中光标前面 直至 上一个空格的字，就类似于鼠标点两下选中的高亮区域

  2. `Ctrl+Shift+K` :快速删除一整行，直接删完

  3. `Ctrl+\` :在右侧创建一个对照窗口，我经常用它来比对代码，比如定义部分和调用部分

  4. `Alt+Shift+↓` :直接复制一整行，类似于IDEA 的 `Ctrl+D` 快速复制快捷键

  5. `F11`  ：全屏

  6. **Ctrl+\`** : 直接打开终端框，可以在里面执行代码运行的命令

  7. `Ctrl+Shift+[` :快速将函数中的括号合并起来，方便查看文件中函数的架构

  8. `Ctrl + W` :快速关闭当前页面

  9. `Alt + ← `:返回上一次显示的页面位置。特别好用

  10. `Ctrl + B` :隐藏左边资源管理器面板，看代码更全面
  
  11. `Ctrl + L` :快速选择一整行
  
  12. `Alt + ↓/↑` :将一整行上移下移
  
  13. `Ctrl + k  Ctrl + 0`: 折叠所有代码，注意先按 Ctrl + k 然后再按 Ctrl + 0，这个 0 是键盘 上面那一条的0

  14. `Ctrl + k Ctrl + j`: 展开所有代码

  15. `//TODO `，`//!` : 不同的高亮，`TODO` 表橙色，`!` 表红色

  13. ``


#### Linux 常用命令
1. `lscpu`  :--查看Cpu 状态，以及架构（X86,armv7,amd64,x86_64）
2. `nano`   :-- 文本编辑器 Ctrl+O 保存, Ctrl+x 退出
3. `vim`   :--文本编辑器，:i -插入文本，:wq -退出并保存， :q! -强制退出， /text ：-查找对象(按回车后按n 查找下一个，N 查找上一个),  d d  -先按一下d，再按d就会删掉一整行(不用加冒号:),    
4. `ls` ,`ll`  :- 显示当前文件夹下的文件
5. `cd -` :-返回上一次的目录
6. `netstat -tunlp`,`netstat -tunlp | grep 端口号`  :- 查看各类端口占用的信息, :-查看进程端口号
7. `systemctl restart sshd`  :- 重启sshd 服务
8. `docker ps -a`, `docker run w-wordpress`, `docker start w-mysql`  :-d :- docer 容器显示所有容器，创建w-wordpress容器，开启w-mysql容器
9. `free -mh `  :-查看当前内存占用情况
10. `fdisk -l`,`df -h`,`df -l`   :-查看硬盘状况
11. `umount `  :- linux 挂载分区 TODO://///////待整理
12. `sudo passwd` :- 适用于刚装好的系统添加 root 权限密码
13. `chmod 777 test.sh`, `chmod -R test.sh`  :- 给test.sh 文件添加777权限。（数字 4 、2 和 1表示读、写、执行权限）。我们可以用用三个8进制数字分别表示 拥有者 、群组 、其它组( u、 g 、o)的权限详情
14. `ifconfig `  :- 查看网络信息
15. `sudo ln -s /etc/nginx/sites-available/lam.evolute.in lam.evolute.in `  :- 这里的案例表示把lam.evolute.in 连接到当前的文件夹里面(要先cd 到待连接的地方)。
16. `netstat -anp | grep 3306 `  :- 查看端口3306被占用情况
17. `curl -L http://url -o /path/fileName.tar.gz` :- 下载东西到本地，并以fileName.tar.gz 等方式保存
18. `find ./Document -name Search* ` :-  在相对路径./Document 中查找 Search 开头的文件
19. `iptables -L -t nat` :- 查看NAT 的状态，详情请看proxmox 章节以及添加路由的命令
20. `sz    &&   rz` :- 'apt-get install lrzsz'  命令特别有用方便传输东西。
21. `lsof -i :80 && kill -9 PID` :- 查找80端口被谁占用， kill 干掉该端口的进程
22. `du -h fileName` :- 查看fileName 文件/文件夹 所占用的大小
23. `tar -xvf abc.tar.gz` :- 解压文件，当然不同的压缩方式解压命令可能不同
24. `make -j $(($(nproc)+1)) || make -j 1 || make -j1 V=s` :- 执行 `make -j $(($(nproc)+1)) `，如果返回 '非' 就会执行第二条
25. 16.TODO:///////// curl   ----   bash    ----  wget ---yum  等命令待整理


```react
// 查看内存
free -mh

// 查看硬盘
fdisk -l

df -h

//远程SSH 连接
ssh root@192.168.1.1 -p 端口

// 查看nat 状态
iptables -L -t nat

// ubuntu 新版本网络配置文件
vim /etc/netplan/00-installer-config.yaml
// 旧版本
vim /etc/network/interfaces

// 设置初始密码
sudo passwd

// 赋予执行权限  +- 分别代表赋予/消除
chmod +x 文件名

// iptables 添加路由部分 请查看部署gitlab 文档章节

// 修改登录界面样式
vim /etc/motd
/*
            #############################
            #                           #
            #     道路千万条，安全第一条。  #
            #     删库一时爽，亲人两行泪！  #
            #                           #
            #############################

  |\_/|     *****************************    (\_/)
 / @ @ \    *                           *   (='.'=)
( > º < )   *       莫删库，删库必被抓！    *   (")_(")
 `>>x<<     *                           *
 /  O  \    *****************************
*/

// 


```


> linux 通常遇到依赖的问题，你就要关心一下 apt-get 的源的问题,在 /etc/apt/sources.list 文件中。
> sources.list.d 中存放的是用户自定义的外部源地址


#### Clion 学习记录


```react

function replace_text_wps($text){
$replace = array(
'1111' => '<a href="#">111</a>',
'222' => '<a href="#">222/a>',
333' => '<a href="#">333</a>' // '我是要被替换的文本' => '我是被替换后的文本'
);
$text = str_replace(array_keys($replace), $replace, $text);
return $text;
}
add_filter('the_content', 'replace_text_wps');
add_filter('the_excerpt', 'replace_text_wps');

//////////////////////
function replace_text_wps($text){
$replace = array(
‘http://175.178.35.245’ => ‘http://dearl.top:5000’,
    );
 $text = str_replace(array_keys($replace), $replace, $text);
return $text;
}
add_filter(‘the_content’, ‘replace_text_wps’);
add_filter(‘the_excerpt’, ‘replace_text_wps’);

```



#### docker 常用部分

```bash
# 使用docker 拉取容器 
docker pull

# 查看docker 正在运行的容器
docker ps -a
docker stop w-wordpress
docker start w-wordpress
docker restart w-wordpress

# 执行docker 新的容器 并连接到数据库，映射文件，映射端口
docker run --name w-wordpress --link w-mysql:db -v /root/docker/wordpress/uploads.ini:/usr/local/etc/php/conf.d/uploads.ini  -p 80:80  -d wordpress:latest

# 进入容器内部
docker exec -it w-wordpress bash  # w-wordpress 可以用容器名字的一部分代替

# docker compose启动mailu 服务器的命令
docker compose -p mailu up -d

# 容器 / 宿主机之间拷贝文件命令  从左边拷贝到右边，下面这是容器拷贝到宿主机。反之亦然
docker cp w-wordpress:/root /var/www/html/mypathOrFile

# docker compose 清除容器(根据docker-compose.yml 里面的内容来清除) ，容器清除了，但下次up 的时候之前的容器里面的东西还在，也就是没有在硬盘上清除。
docker compose down

# 在已up 的容器中修改宿主机和容器的映射，请看下方的章节提醒
腾讯云服务器使用ssl + 域名安全访问(wordpress + nextcloud + certbot->letsencrypt)
主要内容是修改 /var/lib/docker/container/容器ID/ 路径下 hostconfig.json 和 config.v2.json 文件

# 查找docker 容器库
docker search nextcloud


```






#### git 常用命令
```rust
//1. 克隆地址
git clone `url`

//2. 修改分支 
git batch  TODO:////

//3. ssh密钥生成
在C盘 的用户文件夹的.shh 中右键 Git Bash here,然后执行
ssh-keygen -t rsa -C "dearl@qq.com"

//4. 上传一条龙命令
git pull
git status
git add .
git commit -m "updateMessage"
git push




```


#### markdown 有用的部分

```react
1. 写笔记个人喜欢用 ```react   ``` 这种代码风格

2. &emsp;是输出中文字符的空格，正常情况下markdown 不能输出空格、
  &emsp;是输出英文字符的空格。

3. ~~删除线~~ 

4. Ctrl+B 加粗快捷键

5. ` ` 添加局部高亮

6. TODO:// 修改字体颜色



```