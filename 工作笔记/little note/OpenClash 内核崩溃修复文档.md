### OpenClash 内核崩溃修复文档

[toc]

> 根据西哥建议

- https://openwrt.ai/ 编译带多种 VPN 的固件
- 尝试优化: cloudflare CDN 加速



#### - Proxmox 安装 openWRT

> 注意：

&emsp;&emsp;OpenWRT 下载好的镜像无法直接在 PVE 中启动。因为 .img.gz 解压之后获得的就是免安装镜像，直接拖到 Proxmox 的硬盘中，再把它映射到目标机器的虚拟硬件上。

[安装 OpenWRT 执行的教程](https://4xu.net/posts/koolshare-1.html/)

- PVE 界面下创建虚拟机

- 剥离虚拟硬盘 , 并删除剥离掉的硬盘

  ![image-20231213105929886](OpenClash%20%E5%86%85%E6%A0%B8%E5%B4%A9%E6%BA%83%E4%BF%AE%E5%A4%8D%E6%96%87%E6%A1%A3.assets/image-20231213105929886.png)

- 将 OpenWRT 镜像拷贝到 PVE 的目录下 ( 重命名为 openwrt.img 方便添加硬盘镜像 )。参考路径 `/root/temp/openwrt.img`

  ![image-20231213110504437](OpenClash%20%E5%86%85%E6%A0%B8%E5%B4%A9%E6%BA%83%E4%BF%AE%E5%A4%8D%E6%96%87%E6%A1%A3.assets/image-20231213110504437.png)

- 添加磁盘到虚拟机上 ( 目前的虚拟机编号为 111 )

  ```bash
  #输入命令行
  qm importdisk 111 /root/temp/openwrt.img local-lvm
  ```

- 添加磁盘 ( 引用参考博客的图，因为我已经弄好了，不想再做一次 )

  ![https://cdn.jsdelivr.net/gh/weiwuji1/tupian@master/20211117/LEDE-13.pdnw1m2anvk.png](OpenClash%20%E5%86%85%E6%A0%B8%E5%B4%A9%E6%BA%83%E4%BF%AE%E5%A4%8D%E6%96%87%E6%A1%A3.assets/LEDE-13.pdnw1m2anvk.png)

  > 点击添加即可

  ![https://cdn.jsdelivr.net/gh/weiwuji1/tupian@master/20211117/LEDE-14.1a98m7j03jhc.png](OpenClash%20%E5%86%85%E6%A0%B8%E5%B4%A9%E6%BA%83%E4%BF%AE%E5%A4%8D%E6%96%87%E6%A1%A3.assets/LEDE-14.1a98m7j03jhc.png)

  > 根据需要调整硬盘大小。

  

- 设置引导顺序为刚才添加的磁盘

  ![image-20231213111008897](OpenClash%20%E5%86%85%E6%A0%B8%E5%B4%A9%E6%BA%83%E4%BF%AE%E5%A4%8D%E6%96%87%E6%A1%A3.assets/image-20231213111008897.png)

安装结束, 即可配置 OpenWRT



#### - OpenWRT 下的设置

- 由于测试时需要跟软路由避免冲突。需要修改 IP 地址

  ```bash
  # 临时修改为 192.168.2.254 的地址
  nano /etc/config/network
  service network restart
  
  ```

- 方案1: 网络结构改变，软路由只负责拨号。科学上网则交给大的服务器 ( 存在性能损失，但是保证稳定性, 服务器再拉一条网线给软路由, 出去的网络走服务器的另外一个口 )。

  1. e 

  2. 添加 Trojan 地址

     ![image-20231213105419819](OpenClash%20%E5%86%85%E6%A0%B8%E5%B4%A9%E6%BA%83%E4%BF%AE%E5%A4%8D%E6%96%87%E6%A1%A3.assets/image-20231213105419819.png)

  3. 启动订阅

     ![image-20231213105457546](OpenClash%20%E5%86%85%E6%A0%B8%E5%B4%A9%E6%BA%83%E4%BF%AE%E5%A4%8D%E6%96%87%E6%A1%A3.assets/image-20231213105457546.png)

  4. R86S 软路由指定网关为 192.168.2.254 ( 搜索: 在 openWRT 下指定别的 IP 作网关 )

     [参考教程地址](https://codeantenna.com/a/ZpzK1giadB)

     ```bash
     # 在 192.168.2.1 软路由的 openwrt 的 WEB 界面中
     网络->接口->LAN(编辑)->DHCP服务器->高级设置
     # 在 DHCP 选项中
     设置为 3,192.168.2.254
     ```

  5. 关闭 R86S 中的 DHCP 功能？( 暂时不处理，因为担心 大服务器重启或者其它情况导致上不了网 )

  

- 方案2: openWRT 安装 passwall 软件，还是路由线路走 R86S ( 后续在带宽不够用的时候再考虑，现目前手藓考虑稳定性 )。