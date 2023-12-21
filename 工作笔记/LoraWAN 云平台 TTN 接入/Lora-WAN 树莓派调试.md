### 树莓派环境 Lora-WAN 调试

[toc]

> 运行 LoraWAN 网关程序
>
> [参考教程](https://www.waveshare.net/wiki/SX1302_LoRaWAN_Gateway_HAT)

1. 环境搭建 -> 安装 Git 以及

2. 拉取 Github 仓库

   ```shell
   mkdir workspace
   cd workspce
   # 拉取仓库
   git clone https://github.com/siuwahzhong/sx1302_hal.git
   # 切换分支 
   git checkout ws-dev
   # 清理仓库 & 生成对应二进制文件( 这里的 make clean all 就同时处理了)
   make clean all
   
   #
   cp tools/reset_lgw.sh util_chip_id/
   cp tools/reset_lgw.sh packet_forwarder/
   ```

3. 获取 LoraWAN 网关 EUI 码

   ```shell
   # 查看 chip_id 的 EUI 码
   cd ~/workspace/sx1302_hal/util_chip_id  # 不同环境地址可能不一样，酌情修改
   ./chip_id
   
   ```
   
   - 由于没有传感器，因此程序运行到这里会报错 如下图
   
   ![image-20231012170126261](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450760.png)
   
4. 找到并修改传感器部分的代码。屏蔽传感器的 I2C 以及相关 GPIO 函数

   - 这里推荐使用 VsCode 的远程连接。( 使用 winSCP 也可 ) 以方便修改调试

     ![image-20231012165755297](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450762.png)

     创建和设备的连接，找到 IP 和端口连接。

   - 根据上面报错的 0x38 0x3B 0x39 的地址信息，找到相关函数位置

     ![image-20231012170905560](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450763.png)

   - 找到使用该 `静态常量` 的代码块，并注释。 随后 `make clean all ` 再次获取 EUI 。出现别的报错

     ![image-20231012171149253](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450764.png)

     ![image-20231012172232372](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450765.png)

   - 查找 ' failed to close I2C temperature sensor ' 并将其调用的函数注释掉

     ![image-20231012171524921](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450766.png)

   - 获取到 EUI 码

     ![image-20231012171750298](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450767.png)

5. 拷贝启动配置文件

   ```shell
   # 进入程序的启动文件所在地
   cd ~/workspace/sx1302_hal/packet_forwarder
   # 赋值默认启动配置文件(后续修改用)
   cp global_conf.json.sx1250.EU868 test_conf
   
   ```

6. 修改配置文件

   ```shell
   # 修改 packet_forwarder 文件夹下的 test_conf 配置文件
   vim test_conf
   
   ```

   修改下图部分内容

   ![image-20231012174635011](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450768.png)

   改为：

   ![image-20231012174714598](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450769.png)

7. 启动程序

   `/lorawan/test/sx1302_hal/packet_forwarder $ ./lora_pkt_fwd -c test_conf`

   ![image-20231012174853277](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450770.png)

8. 出现报错：

   ![image-20231012180821788](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450771.png)

   查找并注释掉相关函数

   ![image-20231012180931151](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450772.png)

   ![image-20231012181014088](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450773.png)

   ![image-20231012181056495](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450774.png)

   > 执行 'make clean all' 并重新运行程序
   >
   > 后继续出现报错

   ![image-20231012181231019](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450775.png)

   继续查找并注释掉函数

   ![image-20231012181310423](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450776.png)

   ![image-20231012181346617](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450777.png)

- 重新运行程序，这次就顺利运行



#### TTS 平台接入 Lora 模块

TTS 平台官网：https://eu1.cloud.thethings.network/console

> 网关部分：

- 创建网关

  填入获取到的 EUI 码

  ![image-20231012172442809](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450778.png)

  

- 下载 TTS 网关的 JSON 配置信息文件

  ![image-20231012174308090](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450779.png)

  查看到网关相关信息

  ![image-20231012174045412](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450780.png)

  修改进网关服务程序配置文件中。并启动 **参考运行 LoRa-WAN 程序的启动配置**

  

> 设备部分

- 创建产品应用

  ![image-20231012175131850](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450781.png)

- 添加终端设备

  ![image-20231012175213096](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450782.png)

- 配置信息

  ![image-20231012175920627](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450783.png)

  ![image-20231012175721803](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450784.png)

  ```shell
  ######### AT 命令，测试时根据设备的信息而设置 #########
  # 进入 AT 命令模式
  +++\r
  
  # 查询版本信息
  at+ver=?\r
  
  # 设置通讯方式为 mac
  at+mode=mac\r
  
  # 设置频段为 eu868
  at+band=eu868\r
  at+ch=eu868,1\r
  
  # 设置 appkey, 需要和后台设置的 OTAA keys 一致
  at+appkey=9bdf551c54b6d31907326xxxxxx\r
  
  # 调整为 class c 速率
  at+class=c\r
  
  # 重启生效
  at+reboot\r
  
  # 手动入网
  at+join\r
  
  # 发送无需应答的数据, 端口号2
  at+mactx=uncnf,2,hello\r
  
  # 发送需应答的数据, 端口号8
  at+mactx=cnf,8,world\r
  
  # 设备参数
  at+parm=?\r
  
  
  ```
  
  

> 测试

1. 服务端下发命令到设备

   ![image-20231012181913136](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450785.png)

2. 上传数据到服务端测试:

   ![image-20231012182033572](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202310181450786.png)

   
