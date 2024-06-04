## GitLab 域名证书过期处理

> 前言：

&emsp;&emsp;去年大概这个时候公司购买了一台服务器，我负责处理服务器的搭建，并维护。目前该服务器运行了 GitLab、Mailu 邮件服务器、OpenWRT 软路由( 旁路由 )、Lan-server (ubuntu 系统跑编译)、Win10 ( 如果同事需要在 win10 上跑网页服务就开通端口 )，当时是通过向域名提供商申请的 SSL 证书。这几天快到期了，而一年的 SSL 证书太贵，3 个月的证书太麻烦。

&emsp;&emsp;不惯着那些云服务提供商，坚持使用白嫖的免费 SSL 证书。毕竟云服务提供商吃相太难看了，域名证书比服务器还贵，这玩意本来就是几乎 0 成本的东西。。。

&emsp;&emsp;加入 『 Let's encrypt 』 神教吧！！ **Let's encrypt 万岁、Https 万岁，免费 SSL 万岁，非对称加密万岁！！！**

> 感谢：

&emsp;&emsp;感谢各种开源大佬以及项目，such：Let's encrypt, Linus, Risc-V, Proxmox...., 如何反思一下国内那些做得很大的厂商为何没做一些开源的公益事业呢。diss 一下华为、腾讯、阿里 (百度这种垃圾没必要提及)。华为广告费和公关费收得够多了吧，以前我也是华为粉丝，但是以后的华为产品一律不考虑。:laughing:

---



[toc]



### 一、问题描述

&emsp;&emsp;域名证书到期，虽然用过期的证书也可以使用 https , 但是总感觉被浏览器标红看着不舒服。

- 为何之前不使用 『 Let's encrypt 』

  1. 当时搭建 GitLab 时向领导要了一个证书，一年期的，一年一年更换并不费事。
  2. 后续搭建 mailu 邮件服务的时候考虑过使用 Let's encrypt，但是后续觉得麻烦，就还是用的 GitLab 的证书。
  3. 我自己的 dearl.top 当时好像都没使用 https。
  4. 当时... 我以 80、443 端口没开放，不好申请 mail 子域名的 SSL 证书为由拒绝了使用它。

  

- 为何这次要使用 『 Let's encrypt 』
  1. 申请不到一年期的证书，如果 3 个月就手动更换，我觉得还是用过期证书好了。。。
  2. 领导也觉得 3 个月就要手动申请证书麻烦。
  3. 之前知道可以在没有 80、443 端口的情况下申请 Let's encrypt 证书，但是当时嫌麻烦。
  4. 后面我的服务器到期了，改用 DDNS 搭博客，也要避开 80、443 来申请免费的 SSL 证书。



- 这项任务的吐槽

  :open_hands: 这个任务是我跟领导提醒域名证书要过期了，然后就创造了这个任务。不属于本职工作范畴。

  :trumpet: 后续我自己的网页也会使用这种方式，算是一种安慰吧。

  :disappointed_relieved: 这算杂活，属于维护部分，吃力不讨好。做好了没有实质性的改变，看不出来和之前有什么不同。但是涉及在 proxmox 上改动，不能把服务器搞坏了，搞坏了就重新搭建麻烦大了。



### 二、方案确立

> 思路：

1. Let's encrypt 提供了为没有 80、443 端口的用户的域名验证方式，在 DNS 解析添加 TXT 记录，来验证是你的域名。

2. certbot 可以接入 DNS 提供商的 API，每次修改 TXT 记录 ( 证书续期的时候用 )。

3. 由于证书存在多个存放点， Proxmox、GitLab、Mailu 等不同的 Linux 系统下，因此考虑证书的同步分发。

   这一块就交给 Proxmox 宿主机申请证书，通过它分发给它的 GitLab 和 Mailu 虚拟机。

4. 不同服务可能需求的证书名和格式不一样。因此考虑自己写 shell 脚本来转化。



> 证书分发: 考虑使用 ansible Playbook 软件的功能。



### 三、具体实施

思路来自 chatGPT。

#### 1. Proxmox 申请证书 acme.sh 方式

- 





- 执行脚本报错

  ![image-20240530170041098](GitLab%20%E5%9F%9F%E5%90%8D%E8%AF%81%E4%B9%A6%E8%BF%87%E6%9C%9F%E5%A4%84%E7%90%86.assets/image-20240530170041098.png)

  原因: Linux 和 Windows 换行符格式错误，使用 `sed -i 's/\r//' init.sh` 来将其转化为可识别格式



- 登录服务器后安装 certbot

- ```
  acme.sh --issue --dns -d gitlab.qiot.cn \
   --yes-I-know-dns-manual-mode-enough-go-ahead-please
  ```

- 执行生成

  ```bash
  root@pve:~/.acme.sh# ./acme.sh --issue --dns -d gitlab.qiot.cn  --yes-I-know-dns-manual-mode-enough-go-ahead-please
  [Thu 30 May 2024 05:19:07 PM CST] Using CA: https://acme.zerossl.com/v2/DV90
  [Thu 30 May 2024 05:19:07 PM CST] Create account key ok.
  [Thu 30 May 2024 05:19:08 PM CST] No EAB credentials found for ZeroSSL, let's get one
  [Thu 30 May 2024 05:19:09 PM CST] Registering account: https://acme.zerossl.com/v2/DV90
  [Thu 30 May 2024 05:19:12 PM CST] Registered
  [Thu 30 May 2024 05:19:12 PM CST] ACCOUNT_THUMBPRINT='hl48Nn4uOwQfYrgIAcJ9xh6xrzvh3uKh-W-zJkVZ5Xw'
  [Thu 30 May 2024 05:19:12 PM CST] Creating domain key
  [Thu 30 May 2024 05:19:12 PM CST] The domain key is here: /root/.acme.sh/gitlab.qiot.cn_ecc/gitlab.qiot.cn.key
  [Thu 30 May 2024 05:19:12 PM CST] Single domain='gitlab.qiot.cn'
  [Thu 30 May 2024 05:19:15 PM CST] Getting webroot for domain='gitlab.qiot.cn'
  [Thu 30 May 2024 05:19:15 PM CST] Add the following TXT record:
  [Thu 30 May 2024 05:19:15 PM CST] Domain: '_acme-challenge.gitlab.qiot.cn'
  [Thu 30 May 2024 05:19:15 PM CST] TXT value: 'vFvuA7AM2PC8CY_qsHr_nljzFwkv0BzadRldGJjYLzY'
  [Thu 30 May 2024 05:19:15 PM CST] Please be aware that you prepend _acme-challenge. before your domain
  [Thu 30 May 2024 05:19:15 PM CST] so the resulting subdomain will be: _acme-challenge.gitlab.qiot.cn
  [Thu 30 May 2024 05:19:15 PM CST] Please add the TXT records to the domains, and re-run with --renew.
  [Thu 30 May 2024 05:19:15 PM CST] Please add '--debug' or '--log' to check more details.
  [Thu 30 May 2024 05:19:15 PM CST] See: https://github.com/acmesh-official/acme.sh/wiki/How-to-debug-acme.sh
  root@pve:~/.acme.sh# 
  
  ```

  







#### 2. Ansible 分发证书到虚拟机





#### 3. Proxmox 自动续期，与自动更新









#### 4. 需要使用证书的服务的配置















































































































