### WMBUS 协议手册阅读记录

[toc]

- 概览

  由于原文档为英文文档，而机翻文档阅读起来有点难以理解。因此作此文档，但该文档有部分语言为自述，未必准确。

  | 简写名 |                         全称( 中文 )                         |
  | :----: | :----------------------------------------------------------: |
  |  HES   |          头部终端系统 ( 理解为一种数据处理的网关 )           |
  |  DCU   |    数据采集单元 ( 理解为一种设备( 可以外接各种传感器 ) )     |
  |  CSV   | 逗号分隔值 ( Comma Separated Values )( 我认为就是一种逗号分隔的数据库格式 ) .csv |
  |  MSN   |                           表序列号                           |
  |  USN   |                     单元序列号( 理解为 )                     |
  |  GST   |               Gulf Standard Time ( 一个时区 )                |
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

     



#### 1. Read Meter Data 读表数据

工作流程不必介绍，SFTP 客户端连接服务器，发送上去之后会有校验位

##### 1.1  Meter Data- DCU to HES 

- 单标题行

  格式: DCWA、IP 地址、制造商代码、日期、ACK 位

- 多行

  格式: 日期、MSN、USN、制造商代码、 volume consumption(体积消耗)、流量、温度、报警标志、质量标志



##### 1.2 Meter Data- HES to DCU

- 单行

  格式: DCWA、IP、设备商代码、日期、确认位



#### On-Demand Meter data 按需取表数据



































