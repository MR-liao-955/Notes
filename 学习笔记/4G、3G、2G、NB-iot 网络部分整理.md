### 4G、3G、2G、NB-iot 网络部分整理



#### acquire : 

1. 整理 4G、3G、2G、NB-iot 网络部分RSSI  imei rsrp rsrq 等代表的含义 以及取值范围 之类。
2. 整理 网络部分 的频宽 LTE-FDD LTE-TDD  之类的网络的频率。国外客户和国内有差异，主要帮助客户完成设备选型，部分频段国外禁用，因此要合规合法。
3. 整理 4G 模块之类的上传下载的 速度 DL  download （下载速度），UL upload （上传速度）





- 以 中移物联网 MN316下 `cm_modem.h`头文件的结构体定义为例子

```c
/** 运营商信息 */
typedef struct
{
    uint8_t mode;         /*!<网络选择模式,0:自动搜网,1:手动搜网,2:去激活网络*/
    uint8_t format;       /*!<运营商信息格式,固定值2 */
    uint8_t oper[17];     /*!<运营商信息 */
    uint8_t act;          /*!<网络接入技术,固定值9 */
}cm_cops_info_t;

/** 无线信息 */
typedef struct 
{
    uint16_t rsrp;           /*!<信号接收功率 */
    uint16_t rsrq;           /*!<信号接收质量 */
    uint16_t rssi;           /*!<信号强度指示 */
    uint16_t rxlev;          /*!<信号接收电平 */
    uint16_t tx_power;       /*!<最近一次发射功率 */
    uint32_t tx_time;        /*!<上行累计发送时长 */
    uint32_t rx_time;        /*!<下行累计接收时长 */
    uint32_t last_cellid;    /*!<上一次SB1服务小区ID */
    uint8_t  last_ecl;       /*!<上一次ECL值 */
    uint16_t last_snr;       /*!<上一次信噪比 */
    uint32_t last_earfcn;    /*!<上一次绝对射频频道编号 */
    uint16_t last_pci;       /*!<上一次小区物理ID */
} cm_radio_info_t;

/** 小区信息 */
typedef struct 
{
    bool     primary_cell;   /*!<是否为当前驻留小区 */
    uint32_t earfcn;         /*!<绝对射频频道编号 */
    uint16_t pci;            /*!<小区物理ID */
    uint16_t rsrp;           /*!<信号接收功率 */
    uint16_t rsrq;           /*!<信号接收质量 */
    uint16_t rssi;           /*!<信号强度指示 */
    uint16_t snr;            /*!<信噪比 */
} cm_cell_info_t;

/** EDRX set配置 */
typedef struct
{
    uint8_t mode;               /*!<EDRX配置模式 0:不使用EDRX,1:使用EDRX,3:不使用EDRX,丢弃参数设置*/
    uint8_t act_type;           /*!<网络访问技术 */
    uint8_t edrx_value;         /*!<EDRX周期 */
    uint8_t paging_time_window; /*!<EDRX窗口期 */
} cm_edrx_cfg_set_t;

/** EDRX get配置 */
typedef struct
{
    uint8_t act_type;           /*!<网络访问技术*/
    uint8_t edrx_value;         /*!<EDRX周期 */
    uint8_t paging_time_window; /*!<EDRX窗口期 */
} cm_edrx_cfg_get_t;

/**PSM配置*/
typedef struct
{
    uint8_t mode;    /*!<PSM配置模式 0:不使用PSM,1:使用PSM,2:不使用PSM，并丢弃参数设置*/
    uint8_t periodic_tau; /*!<T3412值*/
    uint8_t active_time;  /*!<T3324值*/
} cm_psm_cfg_t;

/**PS网络注册状态*/
typedef struct
{
    uint8_t n;            /*!<模式*/
    uint8_t state;        /*!<网络注册状态*/
    uint16_t lac;         /*!<位置区码*/
    uint32_t ci;          /*!<小区识别码*/
    uint8_t act;          /*!<网络访问技术*/
    uint8_t rac;          /*!<路由区域编码*/
    uint8_t cause_type;   /*!<reject_cause类型,仅当state=3时有效*/
    uint8_t reject_cause; /*!<注册失败原因,仅当state=3时有效*/
    uint8_t active_time;  /*!<T3324值*/
    uint8_t periodic_tau; /*!<T3412值*/
} cm_cereg_state_t;
```



#### IMEI、IMSI、SN、ICCID

https://blog.csdn.net/Mark_md/article/details/117221351



#### 通用：rssi、rsrq、rsrp、ecl等信息的含义

- NB-iot 私有。

  ```rust
  // T3412 定时器
  // 表示PSM 模式下最大的休眠时长定义。  之后可以使用定时器在最大休眠周期内唤醒
  /*  设置T3412 定时器时长，   在MN316模块中，该定时器 bit1-5, 表示时间  bit6-8 表示单位。
      1-5bit 为定时时间，6-8bit 为定时单位
      目前来说如果设定时间单位为10 小时高bit 位就定义为10
      000--10分钟  001--1小时    010--10小时
      011--2秒     100--30秒    101--1分钟  110--320小时
  */
   psm_cfg.periodic_tau = (uint8_t)max_updTime; 
  
  
  // T3324 定时器
  // 表示线程执行结束之后Idle 模式持续多久 再进入PSM 休眠
  Idle状态 设备依旧存活，还未断电休眠
   psm_cfg.active_time = 0b01;
  
  ```



- 公用 无线信息

  [参考知乎文档](https://zhuanlan.zhihu.com/p/391527680)

  [华为参考文档](https://bbs.huaweicloud.com/blogs/110246)

  > RSRP（Reference Signal Receiving Power，参考信号接收功率）

  RSRP是代表无线信号强度的关键参数，反映当前信道的路径损耗强度，用于小区覆盖的测量和小区选择/重选。
  RSRP的取值范围：-44~-140dBm，值越大越好。(-44~-140dbm 对应规范0-97 值越大越好 实际值=规范值-140)

  

  |  RSRP 下rx 值   | 覆盖强度等级 |                             描述                             |
  | :-------------: | :----------: | :----------------------------------------------------------: |
  |    Rx ≤ -105    |      6       |                       业务基本无法连接                       |
  | -105 < Rx ≤ -95 |      5       | 表示覆盖差。室外业务能够连接，但连接成功率低，室内业务基本无法连接 |
  | -95 < Rx ≤ -85  |      4       |         表示覆盖一般，室外能够连接，室内连接成功率低         |
  | -85 < Rx ≤ -75  |      3       |                表示覆盖较好，室内外都能够连接                |
  | -75 < Rx ≤ -65  |      2       |              表示覆盖好，室内外都能够很好的连接              |
  |    -65 < Rx     |      1       |                        表示覆盖非常好                        |

  

  

  > RSRQ (Reference Signal Receiving Quality 参考信号接收质量)

  表示LTE参考信号接收质量，这种度量主要是根据信号质量来对不同LTE候选小区进行排序。这种测量用作切换和小区重选决定的输入。
  RSRQ被定义为N*RSRP/(LTE载波RSSI）之比，其中N是LTE载波RSSI测量带宽的资源块（RB）个数。RSRQ实现了一种有效的方式报告信号强度和干扰相结合的效果。
  取值范围：(-3~-19.5) ，越大越好

  |        RSRQ        | 信号质量 |                             描述                             |
  | :----------------: | :------: | :----------------------------------------------------------: |
  |     -10dB ≤ Rx     |   极好   |                强信号，且有最大的数据传输速度                |
  | -15dB ≤ Rx ≤ -10dB |   较好   |                      信号较强，速率较大                      |
  | -20dB ≤ Rx ≤ -15dB |   一般   | 可以获取可靠的数据连接速度，但是边缘数据可能会丢失，当值接近-20dB 时性能将大幅度下降 |
  |     Rx ≤ -20dB     |  无信号  |                           断开连接                           |

  

  > NB-iot  CEL（Coverage Enhancement Level，覆盖增强等级）

  CEL共分三个等级，数值从0到2，分别对应可对抗144dB、154dB、164dB的信号衰减。基站和UE之间会根据其所在的CEL来选择相对应的信息重发次数。

  |   覆盖等级    |    参数区间     |               描述               |
  | :-----------: | :-------------: | :------------------------------: |
  | 0表示常规覆盖 |    MCL<144dB    |        与现有GPRS覆盖一致        |
  | 1表示扩展覆盖 | 144dB<MCL<154dB | 在现有GPRS覆盖的基础上提升了10dB |
  | 2表示极端覆盖 | 154dB<MCL<164dB | 在现有GPRS覆盖的基础上提升了20dB |

  

  > ECL: 信号覆盖等级(根据SNR和RSRQ参数，取值：0-2,0最高，基站与NB-IoT终端之间会根据其所在的CE Level来选择相对应的信息重发次数)

  

  

  >SINR： （Signal to Interference plus Noise Ratio）信号与干扰加噪声比

  指接收到的有用信号的强度与接收到的干扰信号（噪声和干扰）的强度的比值；
  可以简单的理解为“信噪比”。

  取值范围:（0~30） ，值越大越好

  - 中国移动测试要求

  |  参数区间  | 描述 |
  | :--------: | :--: |
  |  SINR>25   | 极好 |
  | SINR:16-25 | 较好 |
  | SINR:11-15 | 一般 |
  | SINR:3-10  |  差  |
  |   SINR<3   | 极差 |

  

  

  > RSSI:    (Received Signal Strength Indication) 接收的信号强度指示

  无线发送层的可选部分，用来判定链接质量，以及是否增大广播发送强度。

  ​	在这个Symbol内接收到的所有信号（包括导频信号和数据信号,邻区干扰信号,噪音信号等）功率的平均值。

  取值范围：

  ​		RSSI的正常范围取决于使用的设备和具体的场景。对于无线路由器和许多其他设备来说，-30 dBm是一个相当强的信号值，而-80 dBm则是较弱的信号。一般来说，**-60 dBm至-70 dBm**之间的RSSI值被认为是一个比较好的信号。

  ​		在CDMA网络中，**RSSI的范围在-110dbm — -20dbm之间**。一般来说，如果RSSI<-95dbm，说明当前网络信号覆盖很差，几乎没什么信 号；-95dmb<RSSI<-90dbm，说明当前网络信号覆盖很弱；RSSI〉-90dbm，说明当前网络信号覆盖较好。所以，一般都是 以-90dbm为临界点，来初略判断当前网络覆盖水平。

  

  > rxlev:  (接收电平)

  Received Signal Level 接收信号电平描述收到信号强度（电平）的统计参数，作为RF功率控制和切换过程的依据，**收信信号电平将被映射到0~63之间的某个RXLEV值**。

  | 接收等级 |     区间范围      |
  | :------: | :---------------: |
  |    0     |     < -110dBm     |
  |    1     | -110dBm ~ -109dBm |
  |    2     | -109dBm ~ -108dBm |
  |    3     | -108dBm ~ -107dBm |
  |   ....   |        ...        |
  |    62    |  -49dbm ~ -48dBm  |
  |    63    |     > -48dBm      |

  

  > earfcn （E-UTRA绝对射频频道编号）通常称为“绝对频点编号”或“射频频点编号”

  用来描述LTE 网络中不同频段和频段组合的频点标识

  

  ```react
  //test 
  
  /*
  
  */
  ```

  

  

  ```react
  // rsrp  取值范围：-44~-140dBm 值越大越好  OK
  
  Reference Signal ReceivedPower 参考信号接收功率
  
  // rsrq
  
  // rssi
  
  // rxlev
  
  // tx_power
  
  // tx_time  --不重要，因为言简意赅
  
  // rx_time  -- 下行累计接收时长
  
  // last_cellid  -- 上一次SB1 服务小区ID   
  
  // last_ecl   -- 上一次ecl 值
  
  // last_snr  -- 上一次信噪比
  
  // last_earfcn   -- 上一次绝对射频频道编号
  
  // last_pci   --上一次小区物理ID
  
  
  
  ```

  

  

  











#### 4G、3G、2G、NB-iot频率区间



![image-20230906105044839](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309061051327.png)





![image-20230906105053931](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309061051328.png)





- chatGPT 给出的频段回复

  以下是常见的4G LTE频段及其对应的频段和频带信息：

| 频段    | 频带              |            区域            |
| ------- | :---------------- | :------------------------: |
| 频段 1  | 2100 MHz          |            全球            |
| 频段 2  | 1900 MHz PCS      |   美国、加拿大、拉丁美洲   |
| 频段 3  | 1800 MHz DCS      | 欧洲、亚洲、非洲、澳大利亚 |
| 频段 4  | 1700/2100 MHz AWS |    美国、加拿大、墨西哥    |
| 频段 5  | 850 MHz CLR       |      北美、南美、亚太      |
| 频段 7  | 2600 MHz IMT-E    |            全球            |
| 频段 8  | 900 MHz           | 欧洲、亚洲、非洲、澳大利亚 |
| 频段 12 | 700 MHz           |   美国、加拿大、拉丁美洲   |
| 频段 13 | 700 MHz           |            美国            |
| 频段 17 | 700 MHz           |        美国、加拿大        |
| 频段 20 | 800 MHz EU DD     |      欧洲、亚洲、非洲      |
| 频段 25 | 1900 MHz          |        美国、加拿大        |
| 频段 26 | 850 MHz           |   美国、加拿大、拉丁美洲   |
| 频段 28 | 700 MHz APT       |          亚太地区          |
| 频段 38 | 2600 MHz          |      亚洲、欧洲、中东      |
| 频段 40 | 2300 MHz          |         亚洲、欧洲         |
| 频段 41 | 2500 MHz TDD      |            全球            |

请注意，以上仅列举了一些常见的频段和频带，不同地区和运营商可能会使用不同的频段和频带组合来提供4G LTE网络。此外，还有其他频段和频带用于特定地区和特定运营商的4G LTE网络。





#### 国内运营商所用频段

```react
以下是中国国内主要运营商（中国移动、中国联通、中国电信）使用的4G LTE频段的一般信息整理：

中国移动（China Mobile）:
LTE FDD频段：1、3、5、8
LTE TDD频段：34、38、39、40、41

中国联通（China Unicom）:
LTE FDD频段：1、3
LTE TDD频段：34、39、40、41

中国电信（China Telecom）:
LTE FDD频段：1、3
LTE TDD频段：34、39、40、41
```



[各类运营商频段信息参考文档](https://www.zhihu.com/tardis/zm/art/375852060?source_id=1003)

























































































































