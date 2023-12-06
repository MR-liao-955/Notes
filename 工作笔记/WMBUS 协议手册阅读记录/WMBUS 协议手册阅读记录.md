### WMBUS 协议手册阅读记录

[toc]

- 概览

  由于原文档为英文文档，而机翻文档阅读起来有点难以理解。因此作此文档，但该文档有部分语言为自述，未必准确。

  | 简写名 | 全称( 中文 )                                                 |
  | :----: | :----------------------------------------------------------- |
  |  HES   | 头部终端系统 ( 理解为一种数据处理的网关 )                    |
  |  DCU   | 数据采集单元 ( 理解为一种设备( 可以外接各种传感器 ) )        |
  |  CSV   | 逗号分隔值 ( Comma Separated Values )( 我认为就是一种逗号分隔的数据库格式 ) .csv |
  |  MSN   | 表序列号                                                     |
  |  USN   | 单元序列号( 理解为 )                                         |
  |  GST   | Gulf Standard Time ( 一个时区 )                              |
  |        |                                                              |
  |        |                                                              |
  |        |                                                              |

  > 使用场景: ( 不同场景有不同的情形 )

  **= Read Meter Data  读表数据**

  ==== Meter Data- DCU to HES  设备向网关发送表数据

  ==== Meter Data- HES to DCU

  **= On-Demand Meter data  按需获取表数据**

  ==== On-Demand Meter data- HES to DCU 网关下发

  ==== On-Demand Meter data acknowledgement- DCU to HES 带确认位的设备上报

  ==== On-Demand Meter data- DCU to HES  设备上报

  **= Meter Configuration 表配置**

  ==== Meter Configuration- HES to DCU 表配置网关->DCU

  ==== Meter Configuration- DCU to HES 表配置设备上报确认

  **= Time Synchronization of Data Concentrator  采集器的时间同步**

  **= General Back Up Procedure  备份计划 ( 只是了解并无需配置 )**

  **= DCU Events/Alarms  设备 DCU 上报事件/告警信息**

  ==== DCU Events/Alarms- DCU to HES 设备主动上报告警信息给网关

  ==== Event Acknowledgement- HES to DCU 网关的事件确认



#### 公共部分

- 文件命名 DCWAxxxx_READ_DATA_DDMMYYHHMM.csv

  1. xxxx 表示地区。

     ![image-20231110172136852](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110172136852.png)
     
     
     
     



#### 1. Read Meter Data 读表数据

![image-20231110180007070](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110180007070.png)

工作流程不必介绍，SFTP 客户端连接服务器，发送上去之后会有校验位

##### 1.1  Meter Data- DCU to HES 

文件名格式：DCWAXXXX_READ_DATA_DDMMYYHHMM.csv  

- 单标题行

  格式: DCWA、IP 地址、制造商代码、日期、ACK 位

- 多行

  格式: 日期、MSN、USN、制造商代码、 volume consumption(体积消耗)、流量、温度、报警标志、质量标志
  
  ![image-20231111113237156](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231111113237156.png)
  
  SAP事件报警标志：
  
  ![image-20231110173750699](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110173750699.png)
  
  质量标志：
  
  ```bash
  00 - 成功
  01 - 无法到达仪表
  02 - 仪表未注册
  03 - 仪表 / 制造商不支持
  ```
  
  

##### 1.2 Meter Data- HES to DCU

文件名格式：DCWAXXXXX_READ_DDMMYYHHMM_ACK.csv

- 单行

  格式: DCWA、IP、设备商代码、日期时间、确认位

---

#### 2. On-Demand Meter data 按需取表数据

![image-20231110175953521](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110175953521.png)

1. HSE 服务器发送 .csv 文件到终端的 'ON_DEMAND_READ_REQ_IN' 
2. DCU 终端解析这些字段并创建一个 ACK 文件在 'ON_DEMAND_READ_REQ_IN_ACK' 文件夹中
3. HSE 服务器从 DCU 中读取 ACK 文件
4. DCU 读取 request 请求表，并创建单个文件夹 'ON_DEMAND_READ_OUT' 

##### 2.1 On-Demand Meter data- HSE to DCU  网关下发

文件名格式: DCWAXXXX_ONDEMAND_DDMMYYHHMM_REQ.csv

- 单行标题

  格式：DCWA, IP, 日期时间

- 每个表按需请求的多行数据 (Multiple Data Row for each meter On Demand REQ)

  格式：MSN, USN , 制造商代码

  

##### 2.2 On-Demand Meter data acknowledgement- DCU to HES  带确认位的设备上报

文件名格式： DCWAXXXX_ONDEMAND_DDMMYYHHMM_REQ_ACK.csv

- 单行标题

  格式: DCWA, ip, 制造商代码, 日期时间，确认位

##### 2.3 On-Demand Meter data- DCU to HES 设备上报

文件名格式： DCWAXXXXX_ONDEMAND_DDMMYYHHMM_READ.csv

- 单行标题

  格式: DCWA,IP

- 在每个表中多种数据行  (Multiple Data Row ONE for each meter)

  格式: 日期时间, MSN，USN，制造商代码，volume consumption(体积消耗)，流量，温度，报警标志，质量标志

----

#### 3. Meter Configuration  表配置

![image-20231110175931492](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110175931492.png)

##### 3.1 Meter Configuration- HES to DCU  表配置网关->DCU

文件名格式：DCWAXXXX_CONF_SETUP_VER_DDMMYYHHMM.csv

- 单行标题

  格式：DCWA, IP, 版本号(xxx), 日期时间

- 每个表的多行数据配置信息 ( Multiple Data Row ONE for each meter configuration )

  格式: MSN, USN, 制造商代码，状态码

  > 状态码

  ```bash
  01 - Active (如果已经加入了新表, 然后就会被设置为 Active flag, DC(数据采集器)用来开始读表 )
  			已存在的表也同样设置了 Active flag 用来继续读表
  02 - Deleted (如果表被删除掉，然后就会被设置为 Delete flag, DC用来停止读表)
  03 - connect / reconnect 
  04 - disconnect
  05 - 上传加密的 key 到 WMBUS 设备们			
  ```
  
  > 如果状态码是 5 那么格式会变为如下
  
  格式：MSN, USN, 制造商代码，05，密钥(encryption_key)
  
  ![image-20231110180634768](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110180634768.png)
  
  
  
  

##### 3.2 Meter Configuration- DCU to HES  表配置DCU->HES

文件名格式：DCWAXXXXX_CONF_VER_DDMMYYHHMM_ACK.csv

- 单行标题

  格式：DCWA，IP

- 多个表的多行ACK数据 ( Multiple Data Row ACK ONE for each meter )

  格式: 日期时间，MSN，USN，制造商代码，质量标志，状态码

  ![image-20231110175323711](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110175323711.png)

  ![image-20231110180551037](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110180551037.png)

  

  
  
  ---

#### 4. Time Synchronization of Data Concentrator   DCU的时间同步

- DCU 通过 NTP 时间服务器进行同步。

- HES 和DCU 都使用 GST(UTC+4) 时区。

---

#### 5. General Back Up Procedure 生成备份流程

![image-20231110175725898](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110175725898.png)

---

#### 6. DCU Events/Alarms  DCU事件/告警

![image-20231110175904761](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231110175904761.png)

##### 6.1 DCU Events/Alarms- DCU to HES

文件名格式: DCWAXXXXX_DCU_EVENT_DDMMYYHHMM.csv

- 单行标题

  格式： DCWA 、IP、制造商代码

- Multiple Data Row for alarms / events  警报的多行数据

  格式：时间日期、事件代码

  ![image-20231111142625503](WMBUS%20%E5%8D%8F%E8%AE%AE%E6%89%8B%E5%86%8C%E9%98%85%E8%AF%BB%E8%AE%B0%E5%BD%95.assets/image-20231111142625503.png)











































