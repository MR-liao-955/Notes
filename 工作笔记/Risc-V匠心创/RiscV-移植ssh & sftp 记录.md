#### Risc-V 移植 ssh 与 sftp 记录

> 前言：

&emsp;&emsp;感谢公司提供的平台，本来就打算向 Linux 方向靠拢，之前有跟领导和师傅交流过，如果有 Linux 部分的开发，我愿意向这方向学习与进步。正好后续可能会做 Linux 的产品开发，应用层面的开发。

&emsp;&emsp;大的方向说，Linux 的产品移植性会更好，平台更加完善，而且开源资料多且复杂。小的方面说，个人的提升，做嵌入式我认为还是要走 Linux 方向，无论是驱动还是应用开发，后续无论是让自己更有竞争力，或者为公司产品做出优化迭代都会更好。不过，产品线肯定不全是 Linux 的，这项技能还是要落到实处。

&emsp;&emsp;为何我如此中意 Linux，因为爱折腾。无论是腾讯云服务器，还是家里的二手 X86 小台式机，上面运行个服务，自己随时随地能访问，而且数据更为安全，这种掌控感是很好的。买了一块 RK的开发板，将来用这块板子学习。后续再买个电视机盒子，刷 armbian 用来跑个人网盘、家庭影院服务。。。放家里，挂个硬盘，DDNS服务 + IPv6 随时随地访问。

> 关于 Risc-V

&emsp;&emsp;天下苦 intel 久矣，而 ARM 的授权费也不低，导致市面上的 SOC 要么都很贵，要么厂家和品类没那么丰富，全志、ST、TI、RK、晶晨、海思、高通、联发科。。。都是中大规模的公司，小厂做不了，感觉限制了它的发展。

&emsp;&emsp;后来出了个 Risc-V 指令集目前 Risc-V 出现之后，阿里云的平头哥公司推出 玄铁 C906 架构，支持 mmu 内存管理，可以运行 Linux。当然 Risc-V 不止是有运行 Linux 架构的，还可以裸机或者 RTOS，类似于 ARM cortex-M 系列的 MCU。目前我已知的 Risc-V 玄铁C906 架构的 soc 芯片有：全志D1、匠芯创D21x....，后续感觉这类芯片( 小厂 or )会如雨后春笋一般冒 出来。。





> reference

1. [知乎总览参考地址](https://zhuanlan.zhihu.com/p/387939051)

   该部分用于参考移植 zlib、openssl 库的部分，以及大纲。

   tip: 对于移植到 RISC-V 架构，该文档的 openssh 配置参数有误。会导致报错，且编译不通过。

2. [编译 openssl 并连接 zlib，openssl 参考地址](https://blog.itpub.net/70018165/viewspace-2930560/)

   tip: 比较详尽的移植到 ARM 开发板的教程。主要看它编译 openssh、以及安装、修改passwd、生成密钥的部分。

3. [报错 PC 连接时报错Connection reset by peer](https://bbs.archlinux.org/viewtopic.php?id=234837)

   

   ```bash
   root@lan-server:~# ssh root@192.168.2.215
   ssh_exchange_identification: read: Connection reset by peer
   ```



> 准备操作：

1. 在 Linux 下要将交叉编译工具链的路径添加到环境变量中

   ```bash
   # 下载并解压 risv64-linux-x86_64... 的交叉编译工具链
   
   # 用你喜欢的方式修改用户变量
   vim ~/.bashrc
   # PATH=$PATH:/root/RISC-V/linux-sdk/env_toolchain/riscv64-linux-x86_64-20210512/bin
   
   # 将用户变量添加到环境中
   source ~/.bashrc
   ```

   ![image-20240515171237668](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954574.png)

   ![image-20240515171452833](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954575.png)

2. 检查环境变量 与 网络

   ```bash
   # 输入 risc 后多按几个 `TAB` 键，你就会看到交叉编译器了。
   
   # udhcpc -i eth0
   # 如果有兴趣的话，还能用 ifconfig 或者 ipaddr 静态添加别的IP，这样就能收到多IP了。
   ```

   

##### 一、移植 openssh

> 大纲

&emsp;&emsp;openssh 分为 3 个子模块的编译和移植。1. 移植 zlib 压缩服务。2. 移植 openssl 加密服务。3. 整合移植 openssh 服务。

###### 1. 编译 zlib 压缩库

[官网下载地址](https://www.zlib.net/fossils/)

这里我选择最新的版本 1.3 url: https://www.zlib.net/fossils/zlib-1.3.tar.gz

- 创建安装目录 （不想污染 Ubuntu 环境，因此单独弄一个文件夹）

  ```bash
  tar -zxvf zlib-1.3.tar.gz
  
  # 创建 install_dir 文件夹
  cd zlib-1.3
  mkdir install_dir
  ```

- 自动构建 Makefile，并指定安装路径。修改编译工具

  ```bash
  # 在 zlib-1.3 的根目录下指定安装路径，并执行下方命令生成 Makefile
  ./configure --prefix=/root/RISC-V/linux-sdk/port_lib/ssh/zlib-1.3/install_dir
  
  # 此时会自动生成 Makefile, 修改编译工具为 riscv64..的
  1. 将 CC=gcc 
  改为 CC=riscv64-unknown-linux-gnu-gcc
  
  2. 将 LDSHARED=gcc -shared -Wl,-soname,libz.so.1,--version-script,zlib.map
  改为 LDSHARED=riscv64-unknown-linux-gnu-gcc -shared -Wl,-soname,libz.so.1,--version-script,zlib.map
  
  3. 将 CPP= 改为 CPP=riscv64-unknown-linux-gnu-gcc
  
  ```

- make 编译后 make install 安装。



- 成果

  ![image-20240515180931313](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954576.png)

---

###### 2. 编译 openssl 加密库

[openssl 官网源码 release](https://www.openssl.org/source/old/index.html)

这里我选择的版本为 3.2.1 url: https://www.openssl.org/source/openssl-3.2.1.tar.gz

- 创建 install_dir 并生成 Makefile 文件

  ```bash
  # 
  mkdir install_dir
  # 构建 Makefile 文件
  ./Configure linux64-riscv64 --prefix=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir
  
  
  # 如果使用旧版本可以使用 no-asm shared no-async 这种配置选项 
  # such 3.0.12
  ./Configure linux64-riscv64  no-asm shared no-async --prefix=/root/RISC-V/linux-sdk/port_lib/ssh-oldversion/openssl-3.0.12/install_dir
  
  # 修改交叉编译工具链
  vim Makefile
  
  # 修改 CROSS_COMPILE 为目标架构
  CROSS_COMPILE=riscv64-unknown-linux-gnu-
  
  
  ```

- 多核编译 & 安装

  make -j15

  make install



- 成果

  1. libcrypto.so.3 动态链接库

     ![image-20240515180753521](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954577.png)

  2. 目标 install 文件夹

     ![image-20240515181125698](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954578.png)



- 遇到问题：

  1. **目标架构选择**

     ![image-20240515180307925](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954579.png)

     我们的目标架构是 riscv64, 因此在该版本下查找 riscv，我们选用: linux64-riscv64

  2. **3.2.1 版本的 openssl 编译报错**![image-20240515174204855](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954580.png)

     解决：使用 openssl-3.2.1 源码时编译选项去掉 no-asm shared no-async 不然会报上述错误, 使用如下的配置选项。（改版本不能使用如上编译选项）

     ```bash
     ./Configure linux64-riscv64 --prefix=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir
     ```

---

###### 3. 编译 openssh 整合前 2个库

[openssh移植官网](https://www.openssh.com/portable.html)

[阿里云openssh官方源码镜像](https://mirrors.aliyun.com/pub/OpenBSD/OpenSSH/portable/)

- 查看该 ssh 版本支持的目标架构

  ```bash
  ./Configure os-specific
  ```

  ![image-20240515182637179](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954581.png)

  

- 生成 Makefile文件

  - 使用知乎文档的配置命令会 **报错** 如下：

    > 没有在配置文件的时候就声明编译器种类。因此报的这个错误

    ```bash
    
    ./Configure --host=riscv64-linux --with-libs --with-zlib=/root/RISC-V/linux-sdk/port_lib/ssh/zlib-1.3/install_dir --with-ssl-dir=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir --disable-etc-default-login 
    ```

    ![image-20240516094510844](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405160954582.png)

  - 注意openssh 和 openssl 的版本要匹配。

    如果不匹配可能在生成 openssh 时会报错，该报错时运行的版本: zlib-1.2.9、openssl-3.0.12、openssh-7.6p1

    ![image-20240516100728672](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108396.png)

  

  

  - 改用 [参考文档](https://blog.itpub.net/70018165/viewspace-2930560/) 的生成 openssh Makefile 的配置命令 (可被执行的安装命令)

    ```bash
    # 将各类静态链接库以及之前编译好的程序整合。
    ./configure --host=riscv64-linux --with-libs --with-zlib=/root/RISC-V/linux-sdk/port_lib/ssh/zlib-1.3/install_dir --with-ssl-dir=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir --disable-etcdefault-login  CC=riscv64-unknown-linux-gnu-gcc AR=riscv64-unknown-linux-gnu-ar
    ```



- make 编译



- 含义解释

  1. --disable-etc-default-login 

     该配置指令是禁用自带的 openssh 配置文件，改用 **sshd_config**.



---

###### 4. 安装 openssh

[参考安装教程](https://blog.itpub.net/70018165/viewspace-2930560/)

- 打包编译好的成果。

  使用 `tar -zcvf output_openssh.tar.gz output_openssh/` 添加压缩包，后上传到开发板。

  ![image-20240516101330145](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108398.png)

  

- 在开发板上创建目录并复制到指定位置

  1. 创建目录

     ```bash
     mkdir -p /usr/local/bin
     
     mkdir -p /usr/local/sbin
     
     mkdir -p /usr/local/libexec/
     
     mkdir -p /usr/local/etc
     
     mkdir -p /var/run
     
     mkdir -p /var/empty/
     ```

  2. 复制到指定文件夹

     ```bash
     - 将scp、sftp、ssh、ssh-add、ssh-agent、ssh-keygen、ssh-keyscan复制到/usr/local/bin目录下；
     
     - 将sshd复制到/usr/local/sbin目录下；
     
     - 将moduli、ssh_config、sshd_config复制到/usr/local/etc目录下；
     
     - 将sftp-server、ssh-keysign复制到 /usr/local/libexec目录下；
     
     ```

- 创建并修改 password 优先级

  ```bash
  # 创建当前root 用户密码
  passwd
  
  # 修改passwd的优先级分隔
  vi /etc/passwd
  在后面添加: 
  sshd:x:74:74:Privilege-separatedSSH:/var/empty/sshd:/sbin/nologin
  
  ```

  

- 生成密钥对

  ```bash
  #在/usr/local/etc/目录下，使用如下命令生成密钥
  cd /usr/local/etc
  
  ssh-keygen -t rsa -f ssh_host_rsa_key -N ""
  
  ssh-keygen -t dsa -f ssh_host_dsa_key -N ""
  
  ssh-keygen -t ecdsa -f ssh_host_ecdsa_key -N ""
  
  ssh-keygen -t ed25519 -f ssh_host_ed25519_key -N ""
  ```

  - 如果发现不识别 ssk-keygen 命令

    ![image-20240516102620220](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108399.png)

    ```bash
    此时应当改用 debugs 原生的接口。
    用 ADB shell 会导致不识别该命令，抑或是不认识 libcrypto.so.3 库。
    
    # 后续如果需要解决该问题，将它设置为系统变量
    ```

    



- 从 openssl 库中的 libcrypto.so.3 传入开发板，**『并设置为环境变量』**

  export 添加或者其它方式。[参考文档](https://blog.51cto.com/u_15127704/3794711)

  ```bash
  # 修改用户变量 下方命令行二者选其一都行
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/root
  export LD_LIBRARY_PATH=/root:$LD_LIBRARY_PATH
  
  # 或者 
  vi ~/.bashrc
  # 末尾添加上方命令行之一，
  source ~/.bashrc
  ```

  >  **WARNING**: 
  >
  > &emsp;&emsp;如果不添加环境变量，直接去含有 libcrypto.so.3 的文件夹下也能执行 sshd，但是在 PC 用客户端连接的时候会报错。





- 修改 sshd_config 与运行程序

  - 报错 /var/empty 必须仅被 root 用户拥有

    ![image-20240516103011996](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108400.png)

    ![image-20240516103051449](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108401.png)

    ```bash
    # 产生原因: 权限给太多了
    # 解决办法：修改用户权限 为 755
    chmod 755 /var/empty
    
    ```

  - 重磅异常！ 用户连不上时

    ![image-20240516103411167](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108402.png)

    ![image-20240516103358917](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108403.png)

    ```bash
    # 开发板执行下方命令准备Debug 排错
    /usr/local/sbin/sshd -Dd
    # 开启 sshd 之后观察当前终端，
    # 当用客户端或者其它方式连接 开发板时，如果报错会出现下方红色方框的提示。
    
    定位错误点：libcrypto.so.3 未能被 sshd 程序识别。
    ```

    url: [排错时参考的英文文档 #6 楼回答 2018-02-05 ](https://bbs.archlinux.org/viewtopic.php?id=234837)  

    ![image-20240516104240078](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108404.png)

    

    



---

###### - 遇到的问题

- 重磅! -> openssh 移植成功，但是连接时报错：

  1. 开发板没有添加环境变量导致的，不识别 crypto 库

  ![image-20240516110216659](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161108405.png)

  

  2. 在 sshd 服务端获取单次连接的日志

     ![image-20240516104240078](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161148903.png)

  3. 定位问题点： libcrypto.so.3 未引入环境变量导致



- 存储空间不够用，移动 ./bin/ 里的文件会导致系统崩溃

  





> 编译后的文件太大，使用 riscv64-unknown-linux-gnu-strip 进行瘦身。

```react
# 直接针对文件瘦身， 
root@lan-server:~/RISC-V/linux-sdk/port_lib/ssh/openssh-9.7p1# riscv64-unknown-linux-gnu-strip sshd
```





##### 二、SFTP

###### 1. 开启 SFTP

> 很简单，直接修改 sshd_config 的 sftp 的地址，因为之前给的默认配置文件的地址时有误的
>
> 本来 openssh 生成的默认 sshd_config 就开启了 sftp，只是它路径不对而已。

![image-20240516115939768](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161204573.png)

- 将上图的 /usr/libexec/sftp-server 改为 /usr/local/libexec/sftp-server 即可。



###### 2. 限制访问路径

[参考教程](https://www.51cto.com/article/606669.html)

[参考教程2](https://www.cnblogs.com/binarylei/p/9201975.html)



###### 3. 单用户下无法 限制访问路径后 使 SFTP 与 SSH 共存



- 建立软连接，使 SSH 看起来像 SFTP 程序

  ```
  ln -sf /usr/local/sbin/sshd /usr/local/sbin/sftpd
  ```

- 修改 sftp_config 的端口号，并限制它的访问路径

  ```bash
  # 复制配置文件
  cp /usr/local/etc/sshd_config /usr/local/etc/sftp_config
  
  ```

  修改 sftp_config 端口、访问路径与权限

  ![image-20240516171000861](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405161710283.png)



- sshd 强制使用其它的配置文件

  ```bash
  
  
  # 强制使用 sftp配置文件
  sudo /usr/local/sbin/sftpd -f /usr/local/sbin/sftp_config
  ```

  



###### -  问题点

1. openssh 开启 SFTP 之后限制文件访问路径后，该用户只能使用 SFTP，而 SSH 功能无法正常访问。

   原因推测: ssh 默认进入的是 / 根目录，而 SFTP 限制了文件访问之后，ssh 就无法直接进入 / 目录，因而导致无法连接 SSH。

2. 对于目前这个裁剪后的 Linux  平台，它不支持 ( 创建新用户 useradd )用户管理与 ( systemctl )进程管理。

   因此无法创建新的用户单独用于 SFTP，考虑再开一个 ssh 进程，单独使用一个端口用于 SFTP 访问。

   



---

#### Reference URL

1. 知乎参考文档，openssh 移植到 arm 的 Linux 开发板上。

   https://zhuanlan.zhihu.com/p/387939051

2. 。。。





BUG1

```bash
解压riscv64-ssh.tar.gz 然后删除压缩包

# 先执行 创建目录，
然后 mv /bin* /usr/local/bin/
后面再 mv 时就发现无此命令。


# 推测可能是包太大，把内存搞崩了？？



```





















- 
  为仿真环境添加指令踪迹功能。

<details class="info" open="" style="box-sizing: inherit; background-color: var(--md-admonition-bg-color); border: 0.05rem solid rgb(0, 184, 212); border-radius: 0.2rem; box-shadow: var(--md-shadow-z1); color: rgba(0, 0, 0, 0.87); display: flow-root; font-size: 0.64rem; margin: 1.5625em 0px; padding: 0px 0.6rem; break-inside: avoid; transition: box-shadow 125ms ease 0s; overflow: visible; font-family: Roboto, -apple-system, BlinkMacSystemFont, Helvetica, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><summary style="box-sizing: border-box; background-color: rgba(0, 184, 212, 0.1); border-top: none; border-right: none; border-bottom: none; border-left: 0.2rem none; border-image: initial; font-weight: 700; margin: 0px -0.6rem; padding: 0.4rem 1.8rem 0.4rem 2rem; position: relative; cursor: pointer; display: block; min-height: 1rem; border-top-left-radius: 0.1rem; border-top-right-radius: 0.1rem; -webkit-tap-highlight-color: transparent; outline: none;">完成本实验需要安装 Verilator</summary><p style="box-sizing: border-box; margin-bottom: 0.6rem;">请参考<a href="https://soc.ustc.edu.cn/CECS/lab0/verilator/" style="box-sizing: inherit; -webkit-tap-highlight-color: transparent; color: var(--md-typeset-a-color); text-decoration: none; word-break: break-word; transition: color 125ms ease 0s;">实验环境搭建</a>中的步骤安装 Verilator。</p></details>

<details class="info" open="open" style="box-sizing: inherit; background-color: var(--md-admonition-bg-color); border: 0.05rem solid rgb(0, 184, 212); border-radius: 0.2rem; box-shadow: var(--md-shadow-z1); color: rgba(0, 0, 0, 0.87); display: flow-root; font-size: 0.64rem; margin: 1.5625em 0px; padding: 0px 0.6rem; break-inside: avoid; transition: box-shadow 125ms ease 0s; overflow: visible; font-family: Roboto, -apple-system, BlinkMacSystemFont, Helvetica, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><summary style="box-sizing: border-box; background-color: rgba(0, 184, 212, 0.1); border-top: none; border-right: none; border-bottom: none; border-left: 0.2rem none; border-image: initial; font-weight: 700; margin: 0px -0.6rem; padding: 0.4rem 1.8rem 0.4rem 2rem; position: relative; cursor: pointer; display: block; min-height: 1rem; border-top-left-radius: 0.1rem; border-top-right-radius: 0.1rem; -webkit-tap-highlight-color: transparent; outline: none;">请获取 Lab2 所需的代码框架</summary><p style="box-sizing: border-box;">请将工作目录切换到 Lab2，运行以下命令：</p><div class="language-shell highlight" style="box-sizing: border-box; margin-bottom: 0.6rem;"><span class="filename" style="box-sizing: inherit; background-color: var(--md-code-bg-color); border-bottom: .05rem solid var(--md-default-fg-color--lightest); border-top-left-radius: 0.1rem; border-top-right-radius: 0.1rem; display: flow-root; font-size: 0.85em; font-weight: 700; margin-top: 1em; padding: 0.661765em 1.17647em; position: relative;">shell</span><pre id="__code_0" style="box-sizing: inherit; font-feature-settings: &quot;kern&quot;; font-family: var(--md-code-font-family); margin-bottom: 1em; margin-top: 0px; color: var(--md-code-fg-color); direction: ltr; font-variant-ligatures: none; display: flow-root; line-height: 1.4; position: relative;"><span style="box-sizing: inherit;"></span><button class="md-clipboard md-icon" title="复制" data-clipboard-target="#__code_0 > code" style="box-sizing: inherit; -webkit-tap-highlight-color: transparent; background: rgba(0, 0, 0, 0); border: 0px; font-family: inherit; font-size: inherit; margin: 0px; padding: 0px; border-radius: 0.1rem; color: var(--md-default-fg-color--lightest); cursor: pointer; height: 1.5em; outline: none; outline-offset: 0.1rem; position: absolute; right: 0.5em; top: 0.5em; transition: color 0.25s ease 0s; width: 1.5em; z-index: 1;"></button><code style="box-sizing: inherit; font-feature-settings: &quot;kern&quot;; font-family: var(--md-code-font-family); color: var(--md-code-fg-color); direction: ltr; font-variant-ligatures: none; background-color: var(--md-code-bg-color); border-radius: 0px 0px 0.1rem 0.1rem; -webkit-box-decoration-break: slice; font-size: 0.85em; padding: 0.772059em 1.17647em; word-break: normal; box-shadow: none; display: block; margin: 0px; outline: none; overflow: auto; scrollbar-color: var(--md-default-fg-color--lighter) #0000; scrollbar-width: thin; touch-action: auto; -webkit-tap-highlight-color: transparent;"><span id="__span-0-1" style="box-sizing: inherit;"><a id="__codelineno-0-1" name="__codelineno-0-1" href="https://soc.ustc.edu.cn/CECS/lab2/intro/#__codelineno-0-1" style="box-sizing: inherit; -webkit-tap-highlight-color: transparent; color: var(--md-typeset-a-color); text-decoration: none; word-break: break-word; transition: color 125ms ease 0s; position: absolute; visibility: hidden;"></a>$<span class="w" style="box-sizing: inherit;"> </span>./zinit.sh
</span></code></pre></div></details>

## **1 仿真环境解读**

<details class="info" open="open" style="box-sizing: inherit; background-color: var(--md-admonition-bg-color); border: 0.05rem solid rgb(0, 184, 212); border-radius: 0.2rem; box-shadow: var(--md-shadow-z1); color: rgba(0, 0, 0, 0.87); display: flow-root; font-size: 0.64rem; margin: 1.5625em 0px; padding: 0px 0.6rem; break-inside: avoid; transition: box-shadow 125ms ease 0s; overflow: visible; font-family: Roboto, -apple-system, BlinkMacSystemFont, Helvetica, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><summary style="box-sizing: border-box; background-color: rgba(0, 184, 212, 0.1); border-top: none; border-right: none; border-bottom: none; border-left: 0.2rem none; border-image: initial; font-weight: 700; margin: 0px -0.6rem; padding: 0.4rem 1.8rem 0.4rem 2rem; position: relative; cursor: pointer; display: block; min-height: 1rem; border-top-left-radius: 0.1rem; border-top-right-radius: 0.1rem; -webkit-tap-highlight-color: transparent; outline: none;">这一部分需要先了解 Verilator 的原理</summary><p style="box-sizing: border-box; margin-bottom: 0.6rem;">如果你还不清楚 Verilator 的基本功能和用法，那么请先阅读<span>&nbsp;</span><a href="https://soc.ustc.edu.cn/CECS/lab2/verilator/" style="box-sizing: inherit; -webkit-tap-highlight-color: transparent; color: var(--md-typeset-a-color); text-decoration: none; word-break: break-word; transition: color 125ms ease 0s;">Verilator 使用指南</a>。</p></details>

我们需要使用 Verilator 对编写的处理器核进行仿真，以验证其正确性。然而，只有处理器核的代码是无法完成仿真的，我们还需要构建一个仿真环境，以产生时钟激励信号，模拟存储器、外设与 CPU 核进行交互，以及获取并输出一些调试信息。

- 计，之后会回到 `sdb_mainloop` 函数中，等待下一条指令。

<details class="question" style="box-sizing: inherit; background-color: var(--md-admonition-bg-color); border: 0.05rem solid rgb(100, 221, 23); border-radius: 0.2rem; box-shadow: none; color: rgba(0, 0, 0, 0.87); display: flow-root; font-size: 0.64rem; margin: 1.5625em 0px; padding: 0px 0.6rem; break-inside: avoid; transition: box-shadow 125ms ease 0s; overflow: visible; font-family: Roboto, -apple-system, BlinkMacSystemFont, Helvetica, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><summary style="box-sizing: border-box; background-color: rgba(100, 221, 23, 0.1); border-top: none; border-right: none; border-bottom: none; border-left: 0.2rem none; border-image: initial; font-weight: 700; margin: 0px -0.6rem; padding: 0.4rem 1.8rem 0.4rem 2rem; position: relative; cursor: pointer; display: block; min-height: 1rem; border-radius: 0.1rem; -webkit-tap-highlight-color: transparent; outline: none;">为什么要编写这么复杂的仿真环境？直接导出波形不就好了？</summary><p style="box-sizing: border-box;"></p><p style="box-sizing: border-box; margin-bottom: 0.6rem;"></p></details>

<details class="success" open="open" style="box-sizing: inherit; background-color: var(--md-admonition-bg-color); border: 0.05rem solid rgb(0, 200, 83); border-radius: 0.2rem; box-shadow: var(--md-shadow-z1); color: rgba(0, 0, 0, 0.87); display: flow-root; font-size: 0.64rem; margin: 1.5625em 0px; padding: 0px 0.6rem; break-inside: avoid; transition: box-shadow 125ms ease 0s; overflow: visible; font-family: Roboto, -apple-system, BlinkMacSystemFont, Helvetica, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><summary style="box-sizing: border-box; background-color: rgba(0, 200, 83, 0.1); border-top: none; border-right: none; border-bottom: none; border-left: 0.2rem none; border-image: initial; font-weight: 700; margin: 0px -0.6rem; padding: 0.4rem 1.8rem 0.4rem 2rem; position: relative; cursor: pointer; display: block; min-height: 1rem; border-top-left-radius: 0.1rem; border-top-right-radius: 0.1rem; -webkit-tap-highlight-color: transparent; outline: none;">Task 1.1</summary><p style="box-sizing: border-box; margin-bottom: 0.6rem;">这个任务不需要大家编写任何代码，只需要大家把模拟器从<span>&nbsp;</span><code style="box-sizing: inherit; font-feature-settings: &quot;kern&quot;; font-family: var(--md-code-font-family); color: var(--md-code-fg-color); direction: ltr; font-variant-ligatures: none; background-color: var(--md-code-bg-color); border-radius: 0.1rem; -webkit-box-decoration-break: clone; font-size: 0.85em; padding: 0px 0.294118em; word-break: break-word; -webkit-tap-highlight-color: transparent; outline: none;">main</code><span>&nbsp;</span>函数开始运行，到进入<span>&nbsp;</span><code style="box-sizing: inherit; font-feature-settings: &quot;kern&quot;; font-family: var(--md-code-font-family); color: var(--md-code-fg-color); direction: ltr; font-variant-ligatures: none; background-color: var(--md-code-bg-color); border-radius: 0.1rem; -webkit-box-decoration-break: clone; font-size: 0.85em; padding: 0px 0.294118em; word-break: break-word; -webkit-tap-highlight-color: transparent; outline: none;">sdb_mainloop</code><span>&nbsp;</span>函数等待命令，再到驱动 CPU 执行指令，最后到仿真结束的过程完整地看一遍。这个过程中，有任何的疑问可以立刻询问助教。</p></details>

## **2 仿真环境的构建**

<details class="warning" open="open" style="box-sizing: inherit; background-color: var(--md-admonition-bg-color); border: 0.05rem solid rgb(255, 145, 0); border-radius: 0.2rem; box-shadow: var(--md-shadow-z1); color: rgba(0, 0, 0, 0.87); display: flow-root; font-size: 0.64rem; margin: 1.5625em 0px; padding: 0px 0.6rem; break-inside: avoid; transition: box-shadow 125ms ease 0s; overflow: visible; font-family: Roboto, -apple-system, BlinkMacSystemFont, Helvetica, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><summary style="box-sizing: border-box; background-color: rgba(255, 145, 0, 0.1); border-top: none; border-right: none; border-bottom: none; border-left: 0.2rem none; border-image: initial; font-weight: 700; margin: 0px -0.6rem; padding: 0.4rem 1.8rem 0.4rem 2rem; position: relative; cursor: pointer; display: block; min-height: 1rem; border-top-left-radius: 0.1rem; border-top-right-radius: 0.1rem; -webkit-tap-highlight-color: transparent; outline: none;">醒醒！这不是 Lab1！</summary><p style="box-sizing: border-box;">我们在 Lab1 中强调的裸机环境是针对在我们自己设计的硬件上运行的程序，而我们的 simulator 是在实验平台上运行的！所以不要再“什么都不敢用了”！</p><p style="box-sizing: border-box; margin-bottom: 0.6rem;"><del style="box-sizing: inherit;">我直接上来就是一个<span>&nbsp;</span><code style="box-sizing: inherit; font-feature-settings: &quot;kern&quot;; font-family: var(--md-code-font-family); color: var(--md-code-fg-color); direction: ltr; font-variant-ligatures: none; background-color: var(--md-code-bg-color); border-radius: 0.1rem; -webkit-box-decoration-break: clone; font-size: 0.85em; padding: 0px 0.294118em; word-break: break-word; -webkit-tap-highlight-color: transparent; outline: none;">#include &lt;bits/stdc++.h&gt;</code>。</del></p></details>

大家在阅读了仿真环境代码之后，肯定发现了其中有大量的空白。我们逐一来看：

### **2.1 时钟驱动函数**

还记得你是如何在 Vivado 的测试激励文件中模拟时钟的吗？





