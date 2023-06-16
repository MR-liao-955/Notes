# MN316 NB-iot多点测温 项目总结



### C语言代码部分（踩坑实录）

- 结构体指针，字符串定义

  之前只用C++ 较多，直接一个string 就行了，加之之前使用Lua 一段时间，所以 C语言的字符串定义就存在一些迷糊，我之前采用 `字符型数组指针 char *str[16] = "hello,world!";` 来定义字符串（这其实有问题，因为指针指向字符串，而你放一个string 进去），实际应当就用普通字符型数组就行



- 数组越界问题

  1. upd[ 104 ] 定义过程中，数组空间没给够，导致内存出错 。
  2. 在  `pUploadDATA->dev.rssi = (uint16_t)(((int16_t)pRadio_info->rssi) / 10);` &emsp;存数据的时候使用了 int16_t ,导致数据异常。
  3. RTOS 的事件接收中 使用 `int32_t flag = osEventFlagsGet(init_task_flag);`&emsp;而我之前使用的是uint32_t 导致判断位出错。
  
- **逻辑或 ||** , **位或 |**

  之前有一次写的 **逻辑或** ||,  导致并没有修改其内容。**因为逻辑或|| 是逻辑运算符**。。。。，应当使用**按位或 |**,才会获取到正确的值

  ```c
  int32_t flag = osEventFlagsWait(init_task_flag, (EVNT_INIT_SUCC | EVNT_FORMAT_SUCC), osFlagsWaitAny, 30000);
  
  if (flag & (EVNT_INIT_SUCC | EVNT_FORMAT_SUCC) == (EVNT_INIT_SUCC | EVNT_FORMAT_SUCC))
  {
      // 数据上报部分函数
      tcp_sendMessage();
  }
  ```

  

- 动态数组

  使用&emsp;~~`uint8_t *const arr[ var ] = { 0 }; `~~ &emsp;会报错，C中不能用变量定义数组，但是可以用malloc( ) 来进行动态开辟数组空间。

  ```c
  int uint_SIZE = sizeof(uploadDATA.dev) + sizeof(uploadDATA.sensor);
  uint8_t *const toArray = (uint8_t *)malloc(uint_SIZE);
  ```

  数组是引用数据类型， 指针不可变，内容可变。因此 创建的时候使用 **指针常量** 来进行定义。



- 大端小端问题，ADC温度转化二分法查表，

  - 大端小端

    &emsp;&emsp;2个字节或者更多字节的数存入到结构体中，它会高位和低位进行交换。问题存在与uint16_t  中存放 比如说温度数据，就需要高位低位转换

    ```c
    /**
     * @auther: DearL
     * @brief: 2个字节的数据转换大小端转换
     * @param: uint16_t
     * @return: void
     */
    void Byte_Convert(uint16_t *source)
    {
        int first = (*source) / 256;
        int second = (*source) % 256;
        *source = second * 256 + first;
    }
    ```

  - ADC 二分法查找温度

    &emsp;&emsp;项目中，使用MCU 进行对3 个NTC 热敏电阻进行电压的ADC 读取。MN316芯片和MCU 进行uart 通信，uart 回传的数据进行电压 转化为 电阻， 然后使用查表法，找出对应温度。

    &emsp;&emsp;向结构体存放数据的时候，将温度扩大10倍，解析的时候再缩小10倍，以此来获得小数。（由于NTC 精度不太精准，因此以产品的思维，我这使用了一个随机数来进行数据处理）

    ```c
    float tp_table[141] = {
        930.50, 880.14, 832.84, 788.37, 746.54, 707.18, 670.11, 635.18, 602.26, 571.20, //-20~-11
        541.90, 514.23, 488.10, 463.41, 440.07, 417.99, 397.10, 377.34, 358.62, 340.89, //-10~-1
        324.10, 307.68, 292.34, 277.99, 264.53, 251.89, 240.00, 228.78, 218.20, 208.19, // 0~9
        198.71, 189.72, 181.19, 173.07, 165.35, 157.98, 150.96, 144.26, 137.85, 131.72, // 10~19
        125.86, 120.24, 114.86, 109.70, 104.75, 100.00, 95.642, 91.506, 87.577, 83.845, // 20~29
        80.299, 76.926, 73.718, 70.665, 67.759, 64.991, 62.355, 59.842, 57.446, 55.161, // 30~39
        52.980, 50.900, 48.913, 47.016, 45.203, 43.471, 41.815, 40.231, 38.717, 37.267, // 40~49
        35.880, 34.552, 33.280, 32.061, 30.894, 29.775, 28.702, 27.673, 26.687, 25.740, // 50~59
        24.831, 23.959, 23.122, 22.317, 21.545, 20.803, 20.089, 19.404, 18.744, 18.110, // 60~69
        17.501, 16.914, 16.350, 15.806, 15.283, 14.780, 14.295, 13.828, 13.378, 12.945, // 70~79
        12.527, 12.124, 11.736, 11.362, 11.001, 10.653, 10.317, 9.9932, 9.6806, 9.3789, // 80~89
        9.0876, 8.8064, 8.5349, 8.2727, 8.0195, 7.7749, 7.5385, 7.3102, 7.0895, 6.8762, // 90~99
        6.6700, 6.4729, 6.2827, 6.0991, 5.9218, 5.7506, 5.5852, 5.4254, 5.2709, 5.1216, // 100~109
        4.9773, 4.8377, 4.7026, 4.5720, 4.4456, 4.3232, 4.2047, 4.0900, 3.9789, 3.8713, // 110~119
        3.7670,                                                                         // 120
    };
    
    /**
     * @brief 二分法进行查表
     * @param ADC_valu
     * @return 具体温度值℃
     */
    int16_t judge_tp_NTC(float ADC_valu)
    {
        uint8_t half_Cycle = 0;
        uint16_t index_start = 0; // index 为最终的值
        uint8_t index_end = 141;  // index 为最终的值
    
        /*判断边界值，超出范围或者低于范围直接return*/
        if (ADC_valu >= tp_table[0])
        {
            index_start = 0;
            return -20; // 返回最低温度值
        }
        else if (ADC_valu <= tp_table[140])
        {
            index_start = 140;
            return 120; // 返回最大温度值
        }
    
        // 使用二分法查找  改变index 下标来确定查找范围
        for (half_Cycle = 71; index_end - index_start != 1;)
        {
            // cm_demo_printf("half_Cycle ====== %d \n", half_Cycle);
            if (ADC_valu > tp_table[half_Cycle])
            {
                index_end = half_Cycle;
                half_Cycle = (index_end + index_start) / 2;
            }
            else if (ADC_valu < tp_table[half_Cycle]) // 如果是后半部分，需要索引就变了
            {
                index_start = half_Cycle;
                half_Cycle = (index_start + index_end) / 2;
            }
            else // 刚好取到中值
            {
                index_start = half_Cycle;
                break;
            }
    
            if (index_end - index_start <= 1) // 当ADC的值在 start~end 这个索引区间时且，索引区间为1，就能确定数据在1摄氏度误差
            {
                break;
            }
        }
    
        return index_start - 20;
        // return -13; // 测试用，传负数
    }
    
    uint16_t ADC_convert_temp(const uint16_t voltage)
    {
        /*
            NTC电阻值 R = 10 * ADCval / (0xFFF -ADCval  )
            ADCval 为采集的NTC值，十六进制
        */
        float temp_R;
        temp_R = (voltage * 10) / (4095 - voltage);
        int16_t temp = judge_tp_NTC(temp_R);
        if (temp < 0)
            return (uint16_t)(-temp + (1 << 12));
        return temp;
    }
    
    
    
    ```

    

- 16进制的字符转化成string 方法，小写string( 0x1->1 , 0x2->2 , ..., 0xa->10, 0xb->11,0xf->15 ) 转化为16进制Numb方法(其实就是atoi 函数 被我自己低配仿造了)

  ```c
  /**
   *
   * @auther: DearL
   * @param : 原始string , 字符串长度
   * @brief : 将16进制的string 转化为int，string必须小写，
   * @date : 2023.5.13
   * @return :
   *
   */
  uint32_t str_toNumb(char *str, uint16_t str_len)
  {
      char char_Base[] = {'1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};
      uint32_t ret_num = 0;
      for (size_t i = 0; i < str_len; i++)
      {
          /* code */
          for (size_t j = 0; j < 15; j++)
          {
              /* code */
              if (char_Base[j] == str[i])
              {
                  uint32_t mutply = 1;
                  for (size_t k = 0; k < str_len - i - 1; k++)
                  {
                      mutply = mutply * 16;
                  }
                  ret_num = ret_num + mutply * (j + 1); // j+1 代表char_Base 所匹配到的真实的数
                  break;
              }
          }
      }
      return ret_num;
  }
  ```

  



### 逻辑部分

- 设备注册

  设备上电就读flash, 判断设备是否注册。

  ```c
  char IsREG[17] = {0};
  cm_flash_read(REGISTER_FLAG_ADDR, IsREG, sizeof(IsREG)); 
  if (strstr(IsREG, "register success") == NULL)
  {
      device_register();
  }
  ```



- 组连接包，并与云平台建立TCP 长连接，设置软件看门狗90s，防止死循环/异常

  ```c
  typedef struct
  {
      char ip[16];
      int16_t port;
      char prod_id[16];
      char spt_name[16];
  } net_conf;
  
  net_conf s_conf = { // 建立socket 连接时 需要添加端口
      "183.230.40.40",
      1811,         // 服务器端口号
      "596096",     // production_ID 产品ID
      "newScript"    // 服务器上脚本的文件名
  }; 
  
  // 建立长连接
  struct sockaddr_in server_addr;
  memset(&server_addr, 0, sizeof(server_addr));
  server_addr.sin_len = sizeof(server_addr);
  server_addr.sin_family = AF_INET;
  server_addr.sin_port = htons(s_conf.port);
  server_addr.sin_addr.s_addr = inet_addr(s_conf.ip); // 这里获取到IP 地址和端口，可以建立连接
  
  ret = connect(socketid, (const struct sockaddr *)&server_addr, sizeof(server_addr)); // 先建立连接，之后再发起请求 上方有提到IP地址和端口
  
  
  ```

  

- 连接后设定tcp 接收阻塞5s 用以接收服务端交互的数据 JSON 格式

  ```c
  // 设置TCP 阻塞超时， 这个也适用于linux 的阻塞超时。
  struct timeval tv_out;
  tv_out.tv_sec = 5;
  tv_out.tv_usec = 0;
  setsockopt(socketid, SOL_SOCKET, SO_RCVTIMEO, (char *)&tv_out, sizeof(tv_out));
  
  // 接收JSON 
  cm_demo_printf("test_rx_buf === %s \n", test_rx_buf);
  json = cJSON_Parse(test_rx_buf);
  ac = cJSON_GetObjectItem(json, "ac"); // 判断是set ele 或者 updata 指令
  
  ```



- 防钝化放电、接收交互的休眠时间信息

  &emsp;&emsp;在JSON 交互部分，判断接收到服务器的数据是否有 ele 的键，根据它的值 来开启相应时长的定时器，在定时器回调函数中释放GPIO 资源结束钝化。同时定义标志位，( 如果程序上报完之后还没放钝化完，那就等待标志位 )

  ```c
  // 数据交互部分 伪函数
  if (batterty_time >= 30)
  {
      // 防钝化放电
      batterty_time = 30;
  }
  cm_demo_printf("Battery_time === %d \n", batterty_time);
  Battery_Active(batterty_time); // 防钝化执行
  
  ```

  ```c
  // 电池放钝化执行函数
  
  int8_t battery_Status = 0; // 默认为0 ，表电池不需要执行放电
  static void way_timer_callback(void *arg);
  
  /**
   *  @brief 电池放电定时器回调函数
   */
  static void batterTimer_callback(void *arg)
  {
      (void)arg;
      cm_gpio_set_level(CM_GPIO_NUM_6, CM_GPIO_LEVEL_LOW); // 执行到回调，设备放钝化 完成。
      battery_Status = 0;  // 标志位置0 表示设备放钝化完成可以进入休眠
      cm_demo_printf("Battery_Active Success\n");
  }
  
  /**
   *  @brief 电池放电定时器
   */
  static void *s_timer_id = NULL;
  int battery_timer(uint32_t time)
  {
      osTimerAttr_t timer_attr = {.name = "Battery_Active"};
      s_timer_id = osTimerNew(batterTimer_callback, osTimerOnce, NULL, &timer_attr);
  
      if (s_timer_id == NULL)
      {
          cm_demo_printf("Battery_Active Timer create fail\n");
          return -1;
      }
      cm_demo_printf("Battery_Active Timer create success\n");
      cm_gpio_set_level(CM_GPIO_NUM_6, CM_GPIO_LEVEL_HIGH);
      battery_Status = 1;
      osTimerStart(s_timer_id, time * 1000);
      return 0;
  }
  
  /**
   * @brief: 电池防钝化放电,拉高gpio
   * @param: 放钝化时间
   */
  void Battery_Active(uint8_t act_time)
  {
      cm_gpio_cfg_t initGPIO;
      /*`GPIO_6` 电池防钝化放电*/
      initGPIO.mode = CM_GPIO_MODE_OUTPUT_PP; // 推挽
      initGPIO.direction = CM_GPIO_DIRECTION_OUTPUT;
      initGPIO.pull = CM_GPIO_PULL_UP;
      gpio_Cfg(&initGPIO, CM_GPIO_NUM_6);                  // GPIO6
      cm_gpio_set_level(CM_GPIO_NUM_6, CM_GPIO_LEVEL_LOW); // 默认低电平
      if (battery_timer(act_time) == -1)                   // 执行设备防钝化
      {
          cm_demo_printf("Battery Active timmer sleep error!\n");
      }
  }
  ```

  ```c
  // task_data_upload.c 文件的 判断放钝化是否完成，如果完成就跳出循环准备休眠
  /*电池放钝化判断*/
  extern int8_t battery_Status;
  while (1)
  {
      if (battery_Status == 0)
          break;
      osDelay(500);
      cm_demo_printf("Battery_Active running please wait! \n");
  }
  ```

  



- 获取传感器/ 设备/ 网络信息，数据组包成16 进制数组` uint8_t upd[54]` 类型等待上报

  &emsp;&emsp;思路：使用结构体来存放数据，让上报数据结构更分明，层次更清晰，期间注意大端小端问题就好。

  - 结构体构建完成并写入数据之后。需要进行内存拷贝，将结构体的内存拷贝到待发送数据 字符数组 的缓存中，注意数据的长度。防止内存溢出，数组越界

  ```c
  // 数据内存拷贝的处理。通过结构体来进行拷贝
  
  /* 创建指针，指针指向后续需要把 整形 转化为字符串类型的第一个地址。
         通过长度来进行数据拷贝做成字符数组 */
  int uint_SIZE = sizeof(uploadDATA.dev) + sizeof(uploadDATA.sensor);
  cm_demo_printf("uint_SIZE == %d \n", uint_SIZE); // 29
  uint8_t toArray[29] = {0}; // 本质就是数组（指针常量，指针不可变，即为引用&）
  
  uint8_t *pStruct = &uploadDATA.dev.ver; // 创建结构体指针，
  char temp[29] = {0};
  memcpy(toArray, pStruct, uint_SIZE);   //   ！！通过指针进行内存的拷贝，数据大小为uint8_t 需要转化为string 的长度
  ArrayToStr(toArray, uint_SIZE, temp);  // toArray 转化为 temp字符串
  
  // 这一块字符串拼接，拼接到sendMESSAGE
  char sendMESSAGE[109] = {0};
  strcat(sendMESSAGE, uploadDATA.sim_Info.imei);
  strcat(sendMESSAGE, uploadDATA.sim_Info.imsi);
  strcat(sendMESSAGE, uploadDATA.sim_Info.iccid);
  strcat(sendMESSAGE, temp);
  
  cm_demo_printf("imei = %s \n", uploadDATA.sim_Info.imei);
  cm_demo_printf("iccid == = %s \n", uploadDATA.sim_Info.iccid);
  cm_demo_printf("sendMessage = %s \n", sendMESSAGE);
  
  // 内存拷贝 sendMESSAGE 拷贝到形参中，实现对外部传的地址进行修改
  int size = sizeof(sendMESSAGE); // size = 55 字节
  cm_demo_printf("sendMSG lenth = %d \n", size);
  memset(str, 0, 112);
  memcpy((void *)str, &sendMESSAGE, 112);
  
  
  ```

  

- 发送数据（等待云平台回复）

  ```c
  // 数据发送函数 之前定义过线程阻塞，因此。可以跑循环，确保在一定时间内等待设备的数据
  // 发送数据 增强代码健壮性
  uint8_t send_UpdWaitTime = 2; // do-while() 循环3次
  do
  {
      ret = send(socketid, (char *)upds, 1005, 0); // 不要在意发送数据大小。。。这个我没改
      if (ret > 0)
      {
          break;
      }
  } while (send_UpdWaitTime--);
  if (send_UpdWaitTime < 0) // TODO:  如果没收到 send_ConWaitTimes 就是-1；无论是do while 还是 while
  {
      cm_demo_printf("sendUPD() TIMEOUT !!,sleep now\n");
      close(socketid);
      return;
  }
  uint8_t test_rx_buf[64] = {0}; // 自定义
  uint8_t waitUPLOAD_Count = 5;
  while (waitUPLOAD_Count--)
  {
      memset(test_rx_buf, 0, sizeof(test_rx_buf));
      cm_demo_printf("test RCV \" succ\" time out 5s\n");
      ret = recv(socketid, test_rx_buf, sizeof(test_rx_buf), 0); // 线程阻塞，不必加系统延时
      cm_demo_printf("test RCVsucc TimeOUT working!!!!!\n");
      if (ret > 0)
      {
          // 测试recv 超时方法： 服务器脚本不写succ 让它故意超时，测间隔时间是否有5s
          cm_demo_printf("tcp recv data,len:%d, [%s]\n", ret, test_rx_buf);
          if (strstr(test_rx_buf, "upload") != NULL)
          {
              upload_suc = 1;
              break;
          }
      }
      else
      {
          cm_demo_printf("recv message error\n");
          continue;
      }
      }
  
  
  ```

  

- 休眠

  注意：设备休眠之前一定要退出所有线程 + 之前休眠需要10-40s 不等，后来查找到原因，可能是网络/运营商的什么机制导致，因此在休眠之前开启飞行模式，可以直接进入休眠。

  ```c
  /********休眠部分的函数**************/
  extern int sim_ready;
      uploadTime = sim_ready ? uploadTime : 2; // 最小上报时长设置为2min 防止休眠失败无法重启
  
      // 测试用
      // uploadTime = 2;
      cm_demo_printf("(uploadTime * 60)  ======= %d \n", (uploadTime * 60));
      cm_rtc_timer_start(CM_RTC_TIMER_ID_0, (uploadTime * 60), rtc_timer_callback, NULL);
      cm_demo_printf("\n===============wating for sleep==================\n");
      // 处理定时器
      cm_modem_set_cfun(0); // 休眠之前进入飞行模式
      if (timer_handle != NULL)
      {
          osTimerStop(timer_handle);   // 删除定时器 并置空
          osTimerDelete(timer_handle); // 删除定时器 并置空
          timer_handle = NULL;
      }
      extern void *way_timer_handle;
      if (way_timer_handle != NULL)
      {
          osTimerStop(way_timer_handle);   // 删除定时器 并置空
          osTimerDelete(way_timer_handle); // 删除定时器 并置空
          way_timer_handle = NULL;
      }
      if (timer_handle != NULL)
      {
          osTimerStop(timer_handle);   // 删除定时器 并置空
          osTimerDelete(timer_handle); // 删除定时器 并置空
          timer_handle = NULL;
      }
  
      /* 关闭Uart 清除事件*/
      cm_uart_close(CM_UART_DEV_0);
      cm_uart_close(CM_UART_DEV_1);
      cm_gpio_deinit(CM_GPIO_NUM_1);
      cm_gpio_deinit(CM_GPIO_NUM_6);
      osEventFlagsDelete(init_task_flag);
      osEventFlagsDelete(conet_task_flag);
  
      cm_pm_work_unlock(); // 对应main 中的lock 线程退出后然后马上进入睡眠模式 线程锁最好比别的线程晚一点
      osThreadExit();
  
  
  
  ```
  
  



### Lua语言 服务端脚本改写部分

> 前言： 由于之前项目使用Lua 语言开发的，Air302模块停产，转为移动MN316模块，
>
> 导致服务端脚本和C 写的上报代码有点水土不服，因此决定修改服务端脚本。

- 服务端脚本，代码处理部分( 服务端回调函数，函数名不能变，且不能在脚本中调用 )

  &emsp;&emsp;设备建立连接之后，服务端会回复succ ，设备如果接收到合适长度的数据，就说明设备上报的数据时正确的，回复upload。用来响应终端设备状态。

  ```lua
  function device_timer_init(dev)
      -- 添加用户自定义代码 --
      -- 例如： --
      dev:timeout(3)
      -- 每隔30s定时下发心跳包
      -- dev:add(30, "dev", 1)
      -- 登录回复
      dev:send("succ")
      ----------------------------------已在设备端处理
      -- --设备登陆后请求一次LBS数据
      -- --记录LBS信息
      -- local cmd = {
      --     action = "init"
      -- }
      -- dev:send(to_str(cmd))
      -----------------------------------
  end
  
  function device_data_analyze(dev)
      -- 返回状态码
      local code = 0
      local streamTab = {} -- 服务器端必须的数据流table
      local dlen = dev:size() -- 数据长度
  
      -- 有效数据长度
      local alen = 52
      local head = to_hex(dev:bytes(1, 3))
      --
      if (dlen < alen) then
          dev:send("err")
          return
      end
      -- 添加数据处理结果
      local add_data_result = nil
  
      -- 长度校验
      if (dlen >= alen) then
  
          local str = ""
          for i = 1, dlen do
              local source = dev:byte(i)
              str = str .. string.format("%02x", source)
          end
  
          chip, nets, info, way = data_decode(str)
  
          add_data_result = add_val(streamTab, "Chip", 0, chip)
  
          add_data_result = add_val(streamTab, "Nets", 0, nets)
  
          add_data_result = add_val(streamTab, "Info", 0, info)
  
          if dlen > 52 then
              -- 开机原因
              add_data_result = add_val(streamTab, "Way", 0,
                                        {way = str_toNumb(str:sub(107, 108))})
          end
      end
  
      -- 判断执行结果
      if not add_data_result then code = 1 end -- 只有false 和nil 才表示 false 其余都为true
  
      -- 响应操作
      dev:response()
      if code == 0 then
          if dlen >= alen then
              dev:send("upload") -- 关闭发送upload 测试recv 超时
          else
              dev:send("ok")
          end
      end
      return dlen, to_json(streamTab)
  end
  ```

- 服务端脚本，自定义函数，用来代码解析。

  思路：使用  lua 语言的string.format( ) 函数格式化数据，变成string 之后在截取子串，看情况是否再转化为number 类型。需要有负数的，截取定义的符号的字符，来判断是否是负数。再判断是否 * -1。

  ```lua
  local str = ""
  for i = 1, dlen do
      local source = dev:byte(i)
      str = str .. string.format("%02x", source)
  end
  ```

  ```lua
  -- 867677060940050460082687208776898604a60922d0028776106300003efce9fb94ff19c900bf0e780078001b00000c2200fa3c0000
  -- 数据字段的解析完整版
  function data_decode(data_str)
      -- Chip
      chip = {
          imei = data_str:sub(1, 15),
          -- imei = (data_str:byte(36) == 0 and 1 or -1),
          imsi = data_str:sub(16, 30),
          iccid = data_str:sub(31, 50),
          version = data_str:sub(51, 52)
      }
      
      -- TODO:
      local rssi_c = data_str:sub(60) == 0 and 1 or -1 -- 判断首是否为0，这个就是三元运算符
      local rsrp_c = data_str:sub(32) == 0 and 1 or -1
      local rsrq_c = data_str:sub(34) == 0 and 1 or -1
      local snr_c = data_str:sub(36) == 0 and 1 or -1 -- snr 应该没符号
  
      local nets = {
          pwrl_plevel = tonumber(data_str:sub(53, 54), 10),
          cel = str_toNumb(data_str:sub(57, 58)),
          rssi = str_toNumb(data_str:sub(61, 62)) * rssi_c,
          rsrp = str_toNumb(data_str:sub(65, 66)) * rsrp_c,
          rsrq = str_toNumb(data_str:sub(69, 70)) * rsrq_c,
          snr = str_toNumb(data_str:sub(73, 74)) * snr_c
      }
      local way = {way = str_toNumb(data_str:sub(107, 108))}
      return chip, nets, info, way
       -- 截取的部分代码，明白它的意思就行了   
  
  
  end
  ```




### NB-iot 部分

> 关于休眠部分，一块电池要用6年，因此处理各项省电措施很重要

- 设备上锁 /释放锁

  `cm_pm_work_lock();`  // 给线程 上锁，防止设备进入休眠

  `  cm_pm_work_unlock();`  //  线程退出后然后马上进入睡眠模式 

   ` osThreadExit();`    //当这个线程退出的时候即将进入休眠



- T3412 定时器

  psm_cfg.periodic_tau = (uint8_t)max_updTime; // 开启T3412 定时器，定时器超时就退出睡眠  设置2小时休眠

  中国移动的官方文档说明：

  ![image-20230612103401326](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161740355.png)

  - 针对单位转化写了一个函数

    ```c
    /** 用途：设定设备的最大超时时间，如果要更改10080 也行
     * @author: DearL
     * @brief:  psm_cfg.periodic_tau 转化接口
                1-5bit 为定时时间，6-8bit 为定时单位
                目前来说如果设定时间单位为10 小时高bit 位就定义为10
                000--10分钟  001--1小时  010--10小时
                011--2秒     100--30秒    101--1分钟  110--320小时
     * @param:  uint16_t *sour_maxTime 待转化时间
     * @return: 0 false   1 true;
    */
    uint8_t tau_timeConvert(uint32_t *sour_maxTime)
    {
        uint8_t base = 0b11111;
        uint32_t max_time = 320 * 60 * base; // 分钟
        uint16_t high_bit = 0b101;
        uint16_t low_bit = 0;
        // high_bit = (*sour_maxTime >> 5);  // 截取高位
        low_bit = (*sour_maxTime % base); // 截取低位
        if (*sour_maxTime <= 0 || *sour_maxTime >= max_time)
        {
            return 0; // 输入时间错误
        }
    
        /* 不考虑秒钟的情况*/
        if (*sour_maxTime >= 1 && *sour_maxTime < 1 * base) // 大于1分钟小于10分钟
        {
            high_bit = 0b101;
            low_bit = (*sour_maxTime / 1); // 截取低位
        }
        else if (*sour_maxTime >= 10 && *sour_maxTime < 10 * base) // 10 分钟到 以10为基底的分钟数
        {
            high_bit = 0b000;
            low_bit = (*sour_maxTime / 10); // 截取低位
        }
        else if (*sour_maxTime >= 60 && *sour_maxTime < 60 * base) // 1小时 起步（其实是310 分钟）
        {
            high_bit = 0b001;
            low_bit = (*sour_maxTime / 60); // 截取低位
        }
        else if (*sour_maxTime >= 10 * 60 && *sour_maxTime < 10 * 60 * base)
        {
            high_bit = 0b010;
            low_bit = (*sour_maxTime / (10 * 60)); // 截取低位
        }
        else if (*sour_maxTime >= 320 * 60 && *sour_maxTime < max_time) // 320 小时 到320 小时为基底的分钟数
        {
            high_bit = 0b110;
            low_bit = (*sour_maxTime / (320 * 60)); // 截取低位
        }
        *sour_maxTime = (high_bit << 5) + low_bit + 1; // +1 刚好囊括 所需的最大数
        return 1;
    }
    
    ```

    

- T3324 定时器

  psm_cfg.active_time = 0x05;          // T3324 定时器设置为10秒钟 // 这个决定设备睡醒之后工作多久

  ![image-20230612103417618](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161740356.png)

  - 关于定时器说明，前5bit 是定时时间， 6-8bit 位表明是时间的单位（倍数）。



- PSM 模式、DRX模式、eDRx模式

  ![img](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161739337.jpg)

  PSM 省电模式，适合周期上报。

  DRX 适合共享单车那种物联网，即时联网，快速响应

  ```react
  eDRX是一种机制，它允许NB-IoT设备在无需接收数据时进入睡眠状态。eDRX通过延长设备在无数据传输期间的监听窗口周期来实现节能。设备将在指定的eDRX周期内进入睡眠，并且只在指定的监听窗口期间唤醒以接收来自基站的传输。
  
  PSM则是一种更为节能的机制，它允许NB-IoT设备在一段时间内完全断开与网络的连接。在PSM模式下，设备与基站之间的连接被完全释放，并且设备进入深度睡眠状态。在预定的时间间隔后，设备会唤醒并重新建立与基站的连接，以接收来自网络的数据。
  ```



- 睡眠唤醒机制

  给Wake UP 引脚高电平实现设备唤醒。项目中采用的磁吸开关/角度开关。

  通过监听GPIO 引脚电平来确定设备唤醒的方式



### RTOS、Socket编程、数据处理部分

#### RTOS 部分

- 设备重启 的处理（除了注册部分以外，别的情况尽量不要让设备重启（避免出现死循环））

  原因：

  - 我定义了一个循环获取 sim 卡状态信息，如果循环超时还没获取到就重启。然而如果因为运营商的问题，导致我的设备一直费电，这是不允许的。因此如果要使用重启，我给它添加了一个重启次数判定( 写入flash )，用来防止因为外部原因导致设备一直重启。

  结果：

  - **关于 sim卡 状态获取异常**： 

    1. 重启并不能再次拿到sim 卡，因为我的理解 reboot() 软件重启 并不会让sim 重新初始化( 除非断电那种重启 )。正因如此导致我设备一直处于重启状态( 一直费电也不能跳出 )。

    2. 对于reboot() 重启办法，我选择写入flash 重启次数，重启超时之后，break 跳出循环 一条路走到休眠。

    3. 最终方案：

       break 出去之后休眠了这次数据没法上报。后面了解到 sim 卡拿不到可以通过开关飞行模式解决。

       ```c
       // 中移物联网 给的函数
       cm_modem_set_cfun(0);  // 关闭模组  理解为飞行模式开
       osDelay(2000);
       cm_modem_set_cfun(1);  // 打开模组
       osDelay(2000);
       ```

       同时，也给它放入循环 并设定次数，超过次数 设置休眠时间为`1`,睡醒了再试。

       ```c
       extern int sim_ready;
       if (!sim_ready)
       {
           uploadTime = 1;
       }
       ```

       



- uart 中断服务函数 关于 `信号量`  的死磕

  - 中断 和信号量 chatGPT 给的回复

    系统中断：

    - 硬件或操作系统通过中断机制通知事件的发生，如定时器到期、设备准备好等。
    - 中断通常由硬件设备或操作系统内核触发，是异步的，即事件发生时会立即中断正在执行的程序。
    - 中断通常用于处理实时性要求高的事件，例如实时数据采集、设备响应等。
    - 中断处理程序通常是短暂的，用于快速响应和处理事件。

    信号量：

    - 信号量是一种同步机制，用于控制对共享资源的访问。
    - 信号量通常由软件线程之间使用，用于实现互斥、同步和资源管理等。
    - 信号量提供了对临界区的保护，确保同一时间只有一个线程可以访问共享资源。
    - 信号量可以用于解决并发编程中的竞争条件、死锁和资源争用等问题。

    &emsp;&emsp;虽然系统中断可以用于通知事件的发生，但它并不提供同步和互斥的能力。**信号量用于线程间的同步和资源管理，确保多个线程在访问共享资源时的正确性和互斥性**。

    &emsp;&emsp;在实际应用中，系统中断和信号量经常一起使用。例如，在一个多线程的嵌入式系统中，**中断可以触发某些事件的处理程序，而处理程序在访问共享资源之前可能需要获取相应的信号量来确保资源的正确使用**。

    &emsp;&emsp;综上所述，系统中断和信号量是不同的机制，各自有其适用的场景和目的。系统中断用于异步事件的通知，而信号量用于线程间的同步和资源管理。

  - OpenCPU 系统中uart 实例：

    在uart 接收处理函数中：（伪代码，部分变量由于篇幅问题不写出定义）

    默认信号量给出阻塞，防止其在函数中一直循环

    ```c
    static void *g_opencpu_recv_tsk_handle = NULL;
    static void *g_uart_sem = NULL;
    ...
    
    static void uart_recv_task_loop(void *arg)
    {
        (void)arg;
        cm_user_msg_t user_msg = {0};
        cm_uart_cmd_recv_t *pstUartCmdRecv = &gstUartCmdRecv;
        CM_DEMO_UART_LOG("uart recv task start\r\n");
    
        while (1)
        {
            if (g_uart_sem != NULL)
            {
                osSemaphoreAcquire(g_uart_sem, osWaitForever); // 阻塞，如果有信号量过来就放行
            }
             if (uart1_rx_size != 0)
            {
                cm_uart_write(CM_UART_DEV_1, (const void *)uart1_rx_buf, uart1_rx_size, 500);
                CM_LOG("uart1 recv and write ok, recv len %d,data:%s\r\n", uart1_rx_size, uart1_rx_buf);
                memset((void *)uart1_rx_buf, 0, sizeof(uart1_rx_buf));
                uart1_rx_size = 0;
            } 
            else if(uart2_rx_size != 0)
            {
                 ...
            }
        }
        osThreadExit();
    }
    
    /**
     *  \brief 串口1接收中断回调函数
     *
     *  \details 此函数只作为串口接收，请勿添加大量代码。
     */
    static void user_uart1_callback(char *rcv_data, uint32_t len)
    {
        if (uart1_rx_size < UART_RX_BUF_SIZE)
        {
            len = (UART_RX_BUF_SIZE - uart1_rx_size) > len ? len : (UART_RX_BUF_SIZE - uart1_rx_size);
            memcpy((void *)&uart1_rx_buf[uart1_rx_size], (void *)rcv_data, len);
            uart1_rx_size += len;
        }
    
        if (g_uart_sem != NULL)
        {
            //Release a Semaphore token up to the initial maximum count.
            osSemaphoreRelease(g_uart_sem);   // uart 中断回调函数，释放信号量，就是给g_uart_sem 信号量添加到最大
        }
    }
    
    void func()
    {
        
        CM_LOG("uart1 open ok\n");
        uart_event.event_entry = (void *)user_uart1_callback;
        cm_uart_register_event(CM_UART_DEV_1, &uart_event);
        
        
        if (g_opencpu_recv_tsk_handle == NULL)
        {
            osThreadAttr_t attr = {
                .name = "OpenCPU-UART-DEMO",
                .priority = osPriorityNormal2,
                .stack_size = 1024,
            };
    
            g_opencpu_recv_tsk_handle = osThreadNew(uart_recv_task_loop, NULL, (const osThreadAttr_t *)&attr);
        }
    }
    ```

    



- 定时器运用

  原由：

  1. 在特定情况下创建定时器来执行特定时间的任务，在定时器超时的时候释放资源。

  2. 设备如果因为代码或者别的异常，安排一个定时器来确保设备不会死循环( 类似于软件看门狗 )。
  3. 循环定时器，定义一定循环来确保能读取某GPIO 电平或其它。

  运用：

  1. 电池防钝化放电。在开启定时器之前将GPIO 拉高放电，在定时器超时的回调函数 执行拉低。定时器设定的时间就是电池拉高放钝化的时间

     ```c
     /**
      *  @brief 电池放电定时器回调函数
      */
     static void batterTimer_callback(void *arg)
     {
         (void)arg;
         cm_gpio_set_level(CM_GPIO_NUM_6, CM_GPIO_LEVEL_LOW);
         battery_Status = 0;
         cm_demo_printf("Battery_Active Success\n");
     }
     
     /**
      *  @brief 电池放电定时器
      */
     static void *s_timer_id = NULL;
     int battery_timer(uint32_t time)
     {
         osTimerAttr_t timer_attr = {.name = "Battery_Active"};
         s_timer_id = osTimerNew(batterTimer_callback, osTimerOnce, NULL, &timer_attr);
     
         if (s_timer_id == NULL)
         {
             cm_demo_printf("Battery_Active Timer create fail\n");
             return -1;
         }
         cm_demo_printf("Battery_Active Timer create success\n");
         cm_gpio_set_level(CM_GPIO_NUM_6, CM_GPIO_LEVEL_HIGH);
         battery_Status = 1;
         osTimerStart(s_timer_id, time * 1000);
         return 0;
     }
     ```

  

  2. 软件监控，防止程序进入死循环不能重启或睡眠

     ```c
     static void rtc_timer_callback(void *arg);
     static osTimerId_t timer_handle = NULL;
     
     static void monitor_timer_callback(void *arg) // 监控 定时器的回调函数
     {
         if (upload_suc)
         {
             osTimerDelete(timer_handle); // 删除定时器 并置空
             timer_handle = NULL;
         }
         else
         {
             cm_demo_printf("upload timeOut monitor_timer_callback excute reboooooooooot now!!!! \n");
             osDelay(200);
             cm_pm_reboot(0); // 异常处理，没执行到上报成功就重启（正常情况下，不会等这么久，）
         }
     }
     
     void fun(){
         if (timer_handle == NULL)  // 防止循环这里再添加新的定时器
         {
             osTimerAttr_t timer_attr = {.name = "test_timer"};
             timer_handle = osTimerNew(monitor_timer_callback, osTimerPeriodic, NULL, &timer_attr);
         }
         osTimerStart(timer_handle, 90000); // 启动定时器
     }
     
     ```

     

  3. 创建循环定时器，`way_timer_handle = osTimerNew(way_timer_callback, osTimerPeriodic, NULL, &timer_attr);`

     osTimerPeriodic 定义为按照周期定时，直到读取到想要的值，或者 数据已经上报，则删除定时器并置空( 退出循环 )。

     ```c
     static void *way_timer_handle = NULL;
     cm_gpio_level_e ret_level = CM_GPIO_LEVEL_LOW;
     /**
      *30s 或者上报周期之前判断 gpio7拉高与否
      */
     static void way_timer_callback(void *arg)
     {
         (void)arg;
         if (cm_gpio_get_level(CM_GPIO_NUM_7, &ret_level) < 0)
         {
             cm_demo_printf("error !!0\n");
         }
     
         if (ret_level == 1 || upload_suc) // 获取到数据 或者 上报成功
         {
             run_way = ret_level;
             cm_demo_printf("way_timer_callback run_way = %d \n", ret_level);
             osTimerStop(way_timer_handle);   // 删除定时器 并置空
             osTimerDelete(way_timer_handle); // 删除定时器 并置空
             way_timer_handle = NULL;
             cm_gpio_deinit(CM_GPIO_NUM_7);
         }
     }
     
     /**
      * @param void
      * @return 0/1 GPIO7 level , -1 error
      * @brief 测量启动方式，如果低电平就是异常启动，磁吸/角度开关
      *
      */
     void DevRun_way()
     {
         cm_gpio_cfg_t init_GPIO7 =
             {
                 .mode = CM_GPIO_MODE_INPUT,
                 .pull = CM_GPIO_PULL_UP,
             };
         gpio_Cfg(&init_GPIO7, CM_GPIO_NUM_7);
     
         // 使用定时器来判断启动方式，在上报数据结束之前不断读取gpio 电平，只要有一次为1，就上报1
         if (way_timer_handle == NULL)
         {
             osTimerAttr_t timer_attr = {.name = "way_timer"};
             way_timer_handle = osTimerNew(way_timer_callback, osTimerPeriodic, NULL, &timer_attr);
         }
         osTimerStart(way_timer_handle, 100); // 100 ms不断循环的定时器
     }
     ```

     

- 后续使用了 **事件** 来进行线程间通信( 如果使用多个互斥量会好一点 )

  



#### Socket部分



- socket 的建立流程

  ```c
  typedef struct
  {
      char ip[16];
      int16_t port;
      char prod_id[16];
      char spt_name[16];
  } net_conf;
  
  net_conf s_conf = { // 建立socket 连接时 需要添加端口
      "183.230.40.40",
      1811,         // 服务器端口号
      "596096",     // production_ID 产品ID
      "newScript"	  // 第四个参数 是服务器上数据解析脚本的文件名
  }; 
  
  // 建立长连接
  struct sockaddr_in server_addr;
  memset(&server_addr, 0, sizeof(server_addr));
  server_addr.sin_len = sizeof(server_addr);
  server_addr.sin_family = AF_INET;
  server_addr.sin_port = htons(s_conf.port);
  server_addr.sin_addr.s_addr = inet_addr(s_conf.ip); // 这里获取到IP 地址和端口，可以建立连接
  
  ret = connect(socketid, (const struct sockaddr *)&server_addr, sizeof(server_addr)); // 先建立连接，之后再发起请求 上方有提到IP地址和端口
  if (ret == -1)
  {
      cm_demo_printf("tcp connect error\n");
      close(socketid);
      osEventFlagsSet(conet_task_flag, EVNT_CONET_FAIL);
      osThreadExit();
  }
  
  // 设置TCP 超时时间5s
  struct timeval tv_out;
  tv_out.tv_sec = 5;
  tv_out.tv_usec = 0;
  // int nNetTimeout = 5000; // 20秒，
  setsockopt(socketid, SOL_SOCKET, SO_RCVTIMEO, (char *)&tv_out, sizeof(tv_out));
  // 超时5s 之后执行下一函数
  ```
  
  

- send( ) / recv( ) 函数的阻塞

  发送函数一半不用定义发送超时，能执行到这里网络是没有问题的，发送回阻塞在这里，默认阻塞1分钟，超时就继续往下走。（send 函数也不能写在循环里(因为没必要) ）

  ```c
   // 设置TCP 超时时间5s
      struct timeval tv_out;
      tv_out.tv_sec = 5;
      tv_out.tv_usec = 0;
      // int nNetTimeout = 5000; // 20秒，
      setsockopt(socketid, SOL_SOCKET, SO_RCVTIMEO, (char *)&tv_out, sizeof(tv_out)); // 接收超时 ，定义5s
      // setsockopt(socketid, SOL_SOCKET, SO_SNDTIMEO, (char *)&tv_out, sizeof(tv_out)); // 发送超时
      // 超时5s 之后执行下一函数
  ```

  伪代码，多次循环接收服务端回传的数据， recv 阻塞为5s

  ```c
  		uint8_t waitCont_Count = 5;
          while (waitCont_Count--)
          {
              memset(test_rx_buf, 0, sizeof(test_rx_buf));
              ret = recv(socketid, test_rx_buf, sizeof(test_rx_buf), 0);
              if (ret > 0)
              {
                  cm_demo_printf("tcp recv data,len:%d, [%s]\n", ret, test_rx_buf);
                  if (strstr(test_rx_buf, "succ") != NULL)
                  {
                      // 避免最后一轮循环拿到succ 但是没执行。要么把waitCount_Count 放在循环后面--，而不是把--放在循环条件中
                      waitCont_Count = 1;
                      login_succ = 1;
                      break;
                  }
              }
              else
              {
                  cm_demo_printf("tcp establish error retry now !\n");
                  // ret = 1; // 因为 send() 之后的函数都是1，所以避免被recv 影响
                  continue;
              }
          }
          if (waitCont_Count <= 0)
          {
              cm_demo_printf("connect to Service error return and sleep now!! \n");
              close(socketid);
              return;
          }
  ```
  
  

- socketID 在不同的任务(线程) 中进行调度

  原由：

  1. 之前 socket数据接收recv( )函数 和upload( ) 函数是放在两个不同的线程中跑的，recv( ) 中用的到upload()里面创建的线程id，用来做数据接收
  2. 可以在不同的线程中使用recv，一边接收数据一边执行。

  结果：

  最终还是舍弃了多开一个线程。还是放在一个 `Task ` 里面阻塞5 秒后再写入函数。毕竟要上报数据，要使得在当此的uploadTime 能显示正确的值( 也不差这5s 的用电量 )。

  

  

#### 数据处理部分

- 读取和写入flash 的问题。

  原由：

  写入flash 如果是写数字进去，那么读的时候很大可能会乱码，（I guest）写入数据之前我没给那一块flash 清空置0，导致读出来的东西不对劲。

  结果：

  1. 为了简单起见，写入flash 使用`字符串`方式( **采用** `sprintf(res_jsonStr, "%d", res->valueint);` **来实现对数据的格式化**为string)。 
  2. 读flash 的时候读string 类型，再使用   `int32_t uploadTime = atoi(sleepTime);`   转化为number 类型。
  3. 其实读和写flash 也能写数字进去，只是一定要记得初始化，因为你不知道之前这块flash 地址里面写了什么鬼东西。



- 数据交互 JSON 数据格式的验证

  原由：

  1. 需要限定用户往设备发送了哪种数据类型，规定发布的交互信息要如下格式。

     ```react
     - 配置设备参数(配置后立即生效, 但验证需要下一周期)
       > {"ac":"set","upd":100}  upd 为 INT 分钟数
     - 放电防钝化(立即生效, 单位为 s)
       > {"ac":"ele","tm":10}
     - 升级指令(升级指令下发后立即生效)
       > {"ac":"update"}
     ```

  2. 需要限定用户发送的`ac` 一定是string 类型 且内容是`set`、 `ele`、 `update`， `upd`、`tm` 一定是number 类型

  3. 且上报时间 / 电池防钝化时间 一定是一个整数，从而作此限定。

  

  结果：

  1. 引入 JSON 代码库（并非 移动公司写的库）实现对交互信息的 json 解析，对于`ac` 使用指针访问 `valuestring`  参数，对于`upd` 、`tm` 使用指针访问`valueint` 参数。

  2. 部分伪代码如下

     ```c
                 json = cJSON_Parse(test_rx_buf);
                 ac = cJSON_GetObjectItem(json, "ac"); // 判断是set ele 或者 updata 指令
                                                       // 需要在接收云平台下发的数据这里判断，，倘若在读取flash 里面判断的话，
                                                       // 那么云平台下发的烂数据，也写进去，下次启动依旧报错，直到发送正确的数据为止
                 // 如果不存在ac 就continue
                 if (ac == NULL || ac->type == cJSON_NULL) // 发包key 必须是ac
                 {
                     cm_demo_printf(" test_rx_buf don't include \"ac \" !!,recv MSG type error !\n ");
                     continue;
                 }
     
                 sprintf(ac_json, "%s", ac->valuestring);
                 cm_demo_printf("ac->valuestring = %s \n", ac->valuestring);
     
                 if (strstr(ac_json, "set") != NULL)
                 {
                     res = cJSON_GetObjectItem(json, "upd");
                     // 如 ac 的字段是set 的情况，它的子key 如果不是upd 就continue; 否则在判断upd 的数据atoi 是否合法
                     if (res == NULL || res->type == cJSON_NULL)
                     {
                         cm_demo_printf(" set type don't include \"upd \" !!,recv MSG type error !\n ");
                         continue;
                     }
                     else if ((res->valueint) <= 0)
                     {
                         cm_demo_printf(" set  \"upd \" value <= 0 !!,recv MSG type error !\n ");
                         continue;
                     }
     
                     sprintf(res_jsonStr, "%d", res->valueint);
                     cm_demo_printf("res_jsonStr .len () = %d \n", strlen(res_jsonStr));
                     cm_demo_printf(res_jsonStr);
     
                     /*
                     如果 nptr不能转换成 int 或者 nptr为空字符串，那么将返回 0  。
                     特别注意，该函数要求被转换的字符串是按十进制数理解的。
                     atoi输入的字符串对应数字存在大小限制（与int类型大小有关），若其过大可能报错-1。
                     */
                     judge = atoi(res_jsonStr);                          // 判断  upd 的对象是否合法，合法的话就写入flash ，否则continue;
                     if (judge == -1 || !judge || res_jsonStr[0] == '0') // atoi 不能转化就 返回0 ，用于判断传入数据的合法性,res_jsonStr[0] == '0' 用来判断 接收是否为string ，string 就为true
                     {
                         cm_demo_printf("judge == -1 || !judge || res_jsonStr[0] == '0' message type error!!\n");
                         continue;
                     }
     
                     cm_demo_printf("UPLOAD_TIME_ADDR here!!!  %s \n", res_jsonStr);
     
                     if (cm_flash_write(UPLOAD_TIME_ADDR, res_jsonStr, 128) == -1) // 直接写128字节进去
                     {
                         cm_demo_printf("cJSON_UPD_write error \n");
                     }
                     cm_flash_read(UPLOAD_TIME_ADDR, &read_test_jsonStr, sizeof(read_test_jsonStr));
                     cm_demo_printf(" read = %s \n", read_test_jsonStr);
                 }
     
     
     ```

     



### 总结

- 将来可能会多开线程，比如交互量比较大的情况。
- 硬件初始化，网络部分，数据组包部分 单独开Thread，
- 各个部分做各个部分的事情，增强代码的可读性 + 把部分可公用的代码做成接口。之后的项目复制粘贴就好



### 后期优化

&emsp;&emsp;使用多个线程进行执行，数据组包线程 不必等待网络线程结束，只要设备一初始化完成，就组包+网络建立连接 同时进行，最后数据的上报则需要等待组包和网络建立连接，可以嵌套 if ( ) 进行判断。

注意：

1. **获取锁的线程要放在前面，不然会出现 意想不到的BUG。**( 磁吸多次才能唤醒 )

2. 事件 接收的时候 OpenCPU 这个RTOS 只用了 `osEventFlagsWait(init_task_flag, EVNT_INIT_SUCC, osFlagsNoClear, 30000);`   **osFlagsNoClear** 这一个事件处理。因为osFlagsWaitAll 和 osFlagsWaitAny 都会在事件发生之后清空事件标志。

3. 其次在OpenCPU 中 osFlagsWaitAll 和 osFlagsWaitAny 时间组别的事件发生后，它们都会等待超时才会停止阻塞。（不知道是不是BUG 还是OpenCPU 故意设计的）

4. **一定要释放资源**，我有一个 循环定时器 在监听 某个GPIO ，当数据上报和GPIO 拉高的时候定时器就会删除并置空`osTimerStop(way_timer_handle);` &emsp; `osTimerDelete(way_timer_handle);`。 但是因为异常导致程序跑飞( 或逻辑问题 导致 程序所有线程都退出掉 ) ，**如若资源没释放** 那么 下面这个回调函数就不会执行删除定时器。（**因为 OS 都没了，线程都没了，它调用不了 cm_os.h 的函数**）

   ```c
   /**
    * @param void
    * @return 0/1 GPIO7 level , -1 error
    * @brief 测量启动方式，如果低电平就是异常启动，磁吸/角度开关
    *
    */
   void *way_timer_handle = NULL;
   cm_gpio_level_e ret_level = CM_GPIO_LEVEL_LOW;
   void DevRun_way()
   {
       cm_gpio_cfg_t init_GPIO7 =
           {
               .mode = CM_GPIO_MODE_INPUT,
               .pull = CM_GPIO_PULL_UP,
           };
       gpio_Cfg(&init_GPIO7, CM_GPIO_NUM_7);
   
       // 使用定时器来判断启动方式，在上报数据结束之前不断读取gpio 电平，只要有一次为1，就上报1
       if (way_timer_handle == NULL)
       {
           osTimerAttr_t timer_attr = {.name = "way_timer"};
           way_timer_handle = osTimerNew(way_timer_callback, osTimerPeriodic, NULL, &timer_attr);
       }
       osTimerStart(way_timer_handle, 150); // 100 ms不断循环的定时器
   }
   
   /**
    *30s 或者上报周期之前判断 gpio7拉高与否
    */
   static void way_timer_callback(void *arg)
   {
       (void)arg;
       if (cm_gpio_get_level(CM_GPIO_NUM_7, &ret_level) < 0)
       {
           cm_demo_printf("runway --cm_gpio_get_level FUNC() error !!\n");
       }
   
       if (ret_level == 1 || upload_suc) // 获取到数据 或者 上报成功
       {
           run_way = ret_level;
           cm_demo_printf("way_timer_callback run_way = %d \n", ret_level);
           cm_gpio_deinit(CM_GPIO_NUM_7);
           osTimerStop(way_timer_handle);   // 删除定时器 并置空
           osTimerDelete(way_timer_handle); // 删除定时器 并置空
           way_timer_handle = NULL;
       }
   }
   ```
   
5. 等待进入休眠时间过长：

   找原因：1.释放其它硬件资源，结果还是一样，休眠要27s 进入。 2. 创建新的测试文件，初次进休眠所需时间有点久可能要20-30s, 但是之后就倒头就睡。3. 就考虑是不是网络的原因了。

   **解决方案：在退出之前开飞行模式，一测试，欸？ 直接倒头就睡！**









