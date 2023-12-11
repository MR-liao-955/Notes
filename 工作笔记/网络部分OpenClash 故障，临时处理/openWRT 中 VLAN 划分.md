### OpenWRT 划分 Vlan

[toc]

#### - 前言

- 目的:

  Clash 作者删库，可能是 OpenClash 使用了 Clash 的 DNS 映射文件之类。导致 OpenWRT 中的 OpenClash 服务会出现不定时的内核崩溃。( 也许 OpenClash 的作者也没想到 Clash 会删库跑路吧 ) , 下方代码块为 Clash 内核崩溃的日志。

  ```bash
  2023-12-06 10:44:06 守护程序：检测到 Clash 内核崩溃，重启中...
  2023-12-06 10:43:59【/tmp/openclash_last_version】下载失败：【curl: (6) Could not resolve host: ftp.jaist.ac.jp】
  2023-12-06 10:43:41【/tmp/openclash_last_version】下载失败：【curl: (6) Could not resolve host: testingcf.jsdelivr.net】
  2023-12-06 10:43:41【/tmp/openclash_last_version】下载失败：【curl: (6) Could not resolve host: testingcf.jsdelivr.net】
  2023-12-06 10:43:41【/tmp/openclash_last_version】下载失败：【curl: (6) Could not resolve host: testingcf.jsdelivr.net】
  	github.com/miekg/dns@v1.1.50/server.go:533 +0x485
  created by github.com/miekg/dns.(*Server).serveUDP
  	github.com/miekg/dns@v1.1.50/server.go:603 +0x1dc
  github.com/miekg/dns.(*Server).serveUDPPacket(0xc00001cfc0, 0xc005124c10?, {0xc0042aa600, 0x2d, 0x200}, {0xd77f60?, 0xc00009c2f0}, 0xc0006b9d20, {0x0, 0x0})
  	github.com/miekg/dns@v1.1.50/server.go:659 +0x462
  github.com/miekg/dns.(*Server).serveDNS(0xc00001cfc0, {0xc0042aa600, 0x2d, 0x200}, 0xc002941500)
  	github.com/Dreamacro/clash/dns/server.go:35 +0x2f
  github.com/Dreamacro/clash/dns.(*Server).ServeDNS(0xc0042aa600?, {0xd7a8b0, 0xc002941500}, 0xc000b9ca20?)
  	github.com/Dreamacro/clash/dns/server.go:60 +0x14e
  github.com/Dreamacro/clash/dns.handlerWithContext(0xc000123518, 0xc000b9ca20)
  	github.com/Dreamacro/clash/dns/middleware.go:31 +0x56e
  github.com/Dreamacro/clash/dns.withHosts.func1.1(0xc00467f9e0?, 0xc000b9ca20?)
  	github.com/Dreamacro/clash/dns/middleware.go:120 +0x91
  github.com/Dreamacro/clash/dns.withFakeIP.func1.1(0xc00467f9e0, 0x2?)
  	github.com/Dreamacro/clash/dns/middleware.go:76 +0xec
  github.com/Dreamacro/clash/dns.withMapping.func1.1(0xc00014fef0?, 0xc000b9ca20?)
  	github.com/Dreamacro/clash/dns/middleware.go:139 +0xe6
  github.com/Dreamacro/clash/dns.withResolver.func1(0x2e3130392e36342e?, 0xc000b9ca20)
  	github.com/Dreamacro/clash/dns/resolver.go:129
  github.com/Dreamacro/clash/dns.(*Resolver).Exchange(...)
  	github.com/Dreamacro/clash/dns/resolver.go:151 +0x305
  github.com/Dreamacro/clash/dns.(*Resolver).ExchangeContext(0xc0001e2ab0, {0xd73d50?, 0xc00003a030}, 0xc000b9ca20)
  	github.com/Dreamacro/clash/dns/resolver.go:159 +0x11b
  github.com/Dreamacro/clash/dns.(*Resolver).exchangeWithoutCache(0xc0001e2ab0, {0xd73d50, 0xc00003a030}, 0xc000b9ca20)
  	github.com/Dreamacro/clash/common/compatible/singleflight.go:11 +0x5c
  github.com/Dreamacro/clash/common/compatible.(*Group[...]).Do(0x6d657a, {0xc0052ac5d0?, 0x21?}, 0xa76140?)
  	golang.org/x/sync@v0.1.0/singleflight/singleflight.go:104 +0x165
  golang.org/x/sync/singleflight.(*Group).Do(0xc0001e2b20, {0xc0052ac5d0, 0x25}, 0xc2c52d?)
  	golang.org/x/sync@v0.1.0/singleflight/singleflight.go:196 +0xc2
  golang.org/x/sync/singleflight.(*Group).doCall(0xb1f700?, 0xc000308150?, {0xc0052ac5d0?, 0x25?}, 0xc2c52d?)
  	golang.org/x/sync@v0.1.0/singleflight/singleflight.go:161 +0x2d3
  golang.org/x/sync/singleflight.(*Group).doCall.func1()
  goroutine 184842 [running]:
  	github.com/miekg/dns@v1.1.50/server.go:533 +0x485
  created by github.com/miekg/dns.(*Server).serveUDP
  	github.com/miekg/dns@v1.1.50/server.go:603 +0x1dc
  github.com/miekg/dns.(*Server).serveUDPPacket(0xc00001cfc0, 0xc005124c10?, {0xc0042aa600, 0x2d, 0x200}, {0xd77f60?, 0xc00009c2f0}, 0xc0006b9d20, {0x0, 0x0})
  	github.com/miekg/dns@v1.1.50/server.go:659 +0x462
  github.com/miekg/dns.(*Server).serveDNS(0xc00001cfc0, {0xc0042aa600, 0x2d, 0x200}, 0xc002941500)
  	github.com/Dreamacro/clash/dns/server.go:35 +0x2f
  github.com/Dreamacro/clash/dns.(*Server).ServeDNS(0xc0042aa600?, {0xd7a8b0, 0xc002941500}, 0xc000b9ca20?)
  	github.com/Dreamacro/clash/dns/server.go:60 +0x14e
  github.com/Dreamacro/clash/dns.handlerWithContext(0xc000123518, 0xc000b9ca20)
  	github.com/Dreamacro/clash/dns/middleware.go:31 +0x56e
  github.com/Dreamacro/clash/dns.withHosts.func1.1(0xc00467f9e0?, 0xc000b9ca20?)
  	github.com/Dreamacro/clash/dns/middleware.go:120 +0x91
  github.com/Dreamacro/clash/dns.withFakeIP.func1.1(0xc00467f9e0, 0x2?)
  	github.com/Dreamacro/clash/dns/middleware.go:76 +0xec
  github.com/Dreamacro/clash/dns.withMapping.func1.1(0xc00014fef0?, 0xc000b9ca20?)
  	github.com/Dreamacro/clash/dns/middleware.go:139 +0xe6
  github.com/Dreamacro/clash/dns.withResolver.func1(0x2e3130392e36342e?, 0xc000b9ca20)
  	github.com/Dreamacro/clash/dns/resolver.go:129
  github.com/Dreamacro/clash/dns.(*Resolver).Exchange(...)
  	github.com/Dreamacro/clash/dns/resolver.go:151 +0x305
  github.com/Dreamacro/clash/dns.(*Resolver).ExchangeContext(0xc0001e2ab0, {0xd73d50?, 0xc00003a030}, 0xc000b9ca20)
  	github.com/Dreamacro/clash/dns/resolver.go:159 +0x11b
  github.com/Dreamacro/clash/dns.(*Resolver).exchangeWithoutCache(0xc0001e2ab0, {0xd73d50, 0xc00003a030}, 0xc000b9ca20)
  	github.com/Dreamacro/clash/common/compatible/singleflight.go:11 +0x5c
  github.com/Dreamacro/clash/common/compatible.(*Group[...]).Do(0x6d657a, {0xc0052ac5d0?, 0x21?}, 0xa76140?)
  	golang.org/x/sync@v0.1.0/singleflight/singleflight.go:104 +0x165
  golang.org/x/sync/singleflight.(*Group).Do(0xc0001e2b20, {0xc0052ac5d0, 0x25}, 0xc2c52d?)
  	golang.org/x/sync@v0.1.0/singleflight/singleflight.go:191 +0xa5
  golang.org/x/sync/singleflight.(*Group).doCall(0xb1f700?, 0xc000308150?, {0xc0052ac5d0?, 0x25?}, 0xc2c52d?)
  	golang.org/x/sync@v0.1.0/singleflight/singleflight.go:189 +0x6f
  golang.org/x/sync/singleflight.(*Group).doCall.func2(0xc00298f916, 0xc004c6c050, 0x50?)
  	github.com/Dreamacro/clash/common/compatible/singleflight.go:12 +0x28
  github.com/Dreamacro/clash/common/compatible.(*Group[...]).Do.func1()
  	github.com/Dreamacro/clash/dns/resolver.go:189 +0x351
  github.com/Dreamacro/clash/dns.(*Resolver).exchangeWithoutCache.func1()
  	github.com/Dreamacro/clash/dns/resolver.go:165 +0x6d
  github.com/Dreamacro/clash/dns.(*Resolver).exchangeWithoutCache.func1.1()
  	github.com/Dreamacro/clash/dns/util.go:24 +0x279
  github.com/Dreamacro/clash/dns.putMsgToCache(0xc0029a4a50?, {0xc0052ac8a0?, 0xc000b9ca20?}, 0x0?)
  	runtime/panic.go:884 +0x212
  panic({0xbc7e80, 0xc005623d88})
  	golang.org/x/sync@v0.1.0/singleflight/singleflight.go:184 +0x3b
  golang.org/x/sync/singleflight.(*Group).doCall.func2.1()
  	golang.org/x/sync@v0.1.0/singleflight/singleflight.go:35 +0x2c
  golang.org/x/sync/singleflight.newPanicError({0xbc7e80?, 0xc005623d88})
  	runtime/debug/stack.go:24 +0x65
  runtime/debug.Stack()
  panic: runtime error: index out of range [0] with length 0
  2023-12-06 10:24:26 守护程序：检测到 Clash 内核崩溃，重启中...
  ```

#### - VLAN 划分

- VLAN 是二层网络中在数据帧上打上一个 VLAN 标签 ( PVID )，二层的设备根据标签来转发

  ![image-20231206180733200](openWRT%20%E4%B8%AD%20VLAN%20%E5%88%92%E5%88%86.assets/image-20231206180733200.png)

  参考部分拓扑图

- VLAN 转发与 去除标记

  [参考 openwrt 进阶教程博客地址, 关于 VLAN 部分参数说明](https://iyzm.net/openwrt/545.html)

  ```bash
  - 未标记的出口：表示允许具有此 VLAN ID 号的数据包通过，但出站时移除 VLAN ID 号标记，同一个物理端口只允许一个 VLAN ID 号使用“未标记”，在各类企业路由系统中称之为 “Access 端口”，一般用于连接计算机等设备。
  
  - 已标记的出口：表示允许具有此 VLAN ID 号的数据包通过，出站时仍然打上 VLAN ID 号标记，对端需要具备同样 VLAN ID 号的端口才能收到此数据包，在各类企业路由系统中称之为 “隧道接口”、“trunk 端口”。
  
  - 不参与：表示此 VLAN ID 号的数据包不通过此端口，等同于禁用端口。
  ```

  ![image-20231207102006460](openWRT%20%E4%B8%AD%20VLAN%20%E5%88%92%E5%88%86.assets/image-20231207102006460.png)

  <center> test </center>

  > 思路分析：所有的交换机的数据都要汇聚到 br-lan 中。再根据不同的 PVID ( VLAN 标签 ) 进行对数据的分流处理。( reject: 因为带 PVID 的数据包到达交换机时，交换机未做 vlan 的管理配置，交换机不会将数据转发给网关，网关也无法收到带 PVID  的数据帧 )

  - 理论实施细节 ( 暂时无法解决上述问题 )

    1. 划分 VLAN 时需要添加虚拟接口，添加接口时要设置 br-lan.10 ( VLAN10 )、br-lan.20 ( VLAN20 ) , 如此即可在设备一栏查看到新添加的 VLAN。
    2. 接口设置的时候 IP 网关要设置为同一网关。( 同一网段下 , 不同 VLAN )
    3. 比如 VLAN10 即为走国内路由

  - 配置思路:  设置二层交换机口设置为 hybrid , 允许 VLAN 10 , 路由层面将 VLAN10 转发 到 WAN 口，如果 VLAN 0 则转发到 br-lan 口。( 方案理论可行，但是后期维护还得处理交换机。 )

    > 热知识
    >
    > 1. access 接口只能配置一个 VLAN , 适用于交换机和 PC 机接入
    > 2. trunk 接口可以配置多个 VLAN， 适用于交换机和其它网络设备的连接
    > 3. hybrid 接口可以和交换机或者别的网络设备，也可以和 PC 机相连，可以自定义是否去掉 PVID 标签。

    交换机所有接口都设置为 hybrid，同时允许 vlan10 的转发。

    软路由部分在 br-lan 后面做 2 个接口设置根据 VLAN 分包，不使用 VPN 的走 WAN，( 这部分估计得设置类似于 ACL 的策略管理 )。

































