### esyasoft 测试记录

[toc]

#### - 测试流程 

```bash
/*       at 口设置         */
//查询备份文件
AT+CBAVLD?

//删除备份文件
AT+CBALIST=4,""
AT+CBALIST=5,""
AT+CBALIST=6,""
AT+CBALIST=7,""
AT+CBAINIT
AT+CBAPPS
/*       at 口设置         */


/*       dam 口设置         */
//删除本地网关配置信息
{"ac":"file","op":"del","path":"file/devicelvl","filetype":"water"}

//删除本地mqtt配置信息
{"ac":"file","op":"del","path":"file/mqttconf","filetype":"water"}

//删除本地apn配置信息
{"ac":"file","op":"del","path":"file/apnconf","fletype":"water"}

//删除本地电表水表的配置信息
{"ac":"file","op":"del","path":"file/gatewayconfig","fletype":"water"}

/*       dam 口设置         */
```

文件分析: 

1. 查询备份文件: 查到的内容

   ![image-20231207154637052](esyasoft%20%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95_%E5%91%A8%E5%93%A5.assets/image-20231207154637052.png)

   该部分内容为脉冲计数备份文件的内容，`json 命令部分为删除本地文件`， `AT 命令部分为删除备份文件`。

2. 打开 esyasoft 之后可以看到 ( 蓝色标记为 devicelvl 备份文件，如果devicelvl 备份文件不存在，那么就不会有Modle & Serial number )

   ![image-20231207154903614](esyasoft%20%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95_%E5%91%A8%E5%93%A5.assets/image-20231207154903614.png)

3. MQTT 文件如果被删除 ' {"ac":"file","op":"del","path":"file/mqttconf","filetype":"water"} '，下图选项不会有 IP

   ![image-20231207155138929](esyasoft%20%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95_%E5%91%A8%E5%93%A5.assets/image-20231207155138929.png)

4. APN 部分暂时先不用管

5. Meter 表数据部分 , 产看有没有数据吧。如果没有，可以通过 json 来写入测试

   ![image-20231207155350230](esyasoft%20%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95_%E5%91%A8%E5%93%A5.assets/image-20231207155350230.png)



#### - 测试步骤

- 删除本地网关配置信息 & mqtt 配置信息 & 电表水表的配置信息 & apn 配置信息

  删除完成之后重启， 查看 esyasoft 中的文件是否存在对应的信息

  > 现象：删除 MQTT 部分

  1. 单独删除 MQTT 的本地文件时，无论重启还是直接打开 esyasoft 软件查看 MQTT 的信息，它都还在

  2. 单独删除 MQTT 的备份文件时，esyasoft 中的 MQTT 信息依旧在

  3. 当先删除本地文件和备份文件时，重启之后查看 esyasoft , 发现配置信息发生初始化的变更。Host IP 地址发生改变

     ![image-20231207163139669](esyasoft%20%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95_%E5%91%A8%E5%93%A5.assets/image-20231207163139669.png)

     

  > 现象: 删除 Meter 表配置部分 ' {"ac":"file","op":"del","path":"file/gatewayconfig","fletype":"water"} '

  &emsp;&emsp;无论是删除备份文件，还是删除本地文件。重启之后依旧会有之前的表配置文件

  &emsp;&emsp;提示: 表配置文件无论在 esyasoft 或者在 AT 指令都无法删除它。

  > 全部删除 ( 本地以及备份 ) 之后，重启发现 Serial number 发生变更

  该种现象出现了一次，后面删除的时候并未出现。

  ![image-20231207173621354](esyasoft%20%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95_%E5%91%A8%E5%93%A5.assets/image-20231207173621354.png)

- 删完之后后续使用 ' AT+CBAVLD? ' 查找会找到模块新生成的备份配置文件。



- 后续使用 JSON 写入表数据，重启或者直接 Reload 发现只有原来的 5+5 个表数据

  ![image-20231208112857512](esyasoft%20%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95.assets/image-20231208112857512.png)

  > 原因 : 删掉备份文件和本地文件之后，它回退到上一个版本，
  >
  > 根据查询命令 ' {"ac":"getinfo","val":101} '，上一个版本只能保存 5 个表配置数据。



#### - 发现的问题

1. 删除水电表不成功，无论何种删除，重启之后表配置都存在。
2. 备份配置 和 本地配置全部删除之后，重启，表序列号会发生改变 ( MQTT 测得 )



#### - 测试结果



















