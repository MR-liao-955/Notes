### ESP32 驱动 RGB 灯带

[toc]

#### 环境搭建

- python 环境，电脑已经安装了python 3.9.13 因此跳过这一步

  ![image-20231023172119370](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420427.png)

- [环境搭建环境教程文档](https://aijishu.com/a/1060000000295342)

- 在 VSCode 中安装 'ESP-IDF EXPLORER' 插件，并使用 VS code 终端 PowerShell 编译。

  ![image-20231027114351647](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420428.png)

- `idf.py build` 进行编译,  

  `idf.py -p COM124 flash`  进行烧录固件

> 编译时环境变量报错指南

- 使用 VScode 插件进行编译和烧录

  ![image-20231031115259412](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420429.png)

- 使用 powershell 时执行命令  'idf.py build' 报错

  ![image-20231031113457789](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420430.png)

  

  解决办法:

  添加环境变量 在如下图的路径下  `.\export.bat`

  ![image-20231031115208987](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420431.png)

  ![image-20231031113759790](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420432.png)

  

   





#### RGB 控制

[参考控制原理教程](https://blog.csdn.net/gengyuchao/article/details/93239317)

- 接收芯片采用类似PWM 编码的形式

- 根据灯珠的芯片手册，发现 PWM、GPIO 等方式并不能满足 时序翻转所需的 3us 因此，根据网上教程，选用 SPI 进行模拟 PWM 收发。

- 控制原理

  根据输入的高低电平持续时间的占比来决定数字是 0 还是 1，可以根据固定的时钟，来对应 SPI 持续发送字节来表示输入的是高电平还是低电平

  ![image-20231023183820806](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420433.png)

  ![image-20231027115043784](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420434.png)

  注:  该灯珠文档有误，24bit 的数据结构标反，应该时 GRB 的顺序结构。

- 根据上图，可以明白，RGB 分别是 24bit 控制一个灯，但，我们采用的是 SPI 模拟 PWM 信号，而**由多个 SPI 的信号才能表示单个 1 or 0**。根据码型时间和 SPI 时钟频率，我这边选用的 **1字节(  8bit ) 来表示 24 bit 中的一个bit**。



- ( 待优化 ) 频率选择，由于我们想要 0.3 微秒，因此 esp_flash_speed_s 需要 10MHz 来表示我们确切的时序( **后来我并没有这么做，依旧采用的 8MHz 来发送高低电平** )。

  (3bit->1 , 6bit->0) 代表低电平。

  ![image-20231023190916403](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420435.png)





#### ESP32 控制 RGB 灯的代码实现

> 参考教程

[API 中文 SPI Master 部分文档 ( 参考代码 )](https://juejin.cn/post/7108542432051986439)

[SPI 模拟 PWM 点亮 RGB 灯原理文档](https://blog.csdn.net/gengyuchao/article/details/93239317)

- ESP32 引脚描述

  ![引脚图](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420436.png)

![image-20231024091405830](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420437.png)



> 实现部分代码 ( 讲解在后面 )

- 初代版本代码 ( 点亮多少个灯珠就发送多长的结构体，如果发送 50个 甚至 200 个灯珠，内存占用就太大了)，栈溢出警告 ( stack over flower )  

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/spi_master.h"
#include "driver/gpio.h"

#include "sdkconfig.h"
#include "esp_log.h"

#define DMA_CHAN 2
#define PIN_NUM_MISO 12
#define PIN_NUM_MOSI 13
#define PIN_NUM_CLK 14
#define PIN_NUM_CS 15

static const char TAG[] = "main";

esp_err_t spi_write(spi_device_handle_t spi, uint8_t *data, uint8_t len);

// 使用结构体来存放数据
typedef struct
{
    uint16_t R[8];
    uint16_t G[8];
    uint16_t B[8];
} RGB_color_t;

// /**
//  * @brief: 取反，将16进制低位在前，高位在后，返回亦是1字节 16进制数
//  * @parm: HEX
//  * @return: ~HEX 取反后的16进制数
//  */
// uint8_t color_dec(uint8_t data)
// {
//     uint8_t ref = 0x00;
//     uint8_t i = 7;
//     // 循环7次，反向短除法。(最后结果反写)
//     while (i)
//     {
//         uint8_t ret = 0;
//         ret = data % 2; // 取余
//         data = data / 2;
//         printf("ret == %d, ", ret);
//         ref = ref | (ret << i); // 向16进制末尾插入余数
//         if (i == 1)             // 短除法最后一位
//         {
//             ref = ref | ((data % 2) << 0);
//             printf("ret == %d, ", ret);
//         }
//         i--;
//     }
//     ESP_LOGI(TAG, "ref =  %d", ref);
//     return ref;
// }

/**
 * @brief: 不取反  emmmm
 * @parm: HEX
 * @return: ~HEX 取反后的16进制数
 */
uint8_t color_dec(uint8_t data)
{
    return data;
}

/**
 * @brief 将数据膨胀 8 字节，高位在前，这就是将要发给 RGB 灯带的数据
 * @parm: data: 原始数据， arr: 将要转化成的数组
 * @return: void
 */
void value_expand(uint8_t data, uint16_t *arr)
{
    for (size_t i = 7; i > 0; i--)
    {
        uint8_t res = 0;
        res = data % 2;
        data = data / 2;
        if (res == 1)
        {
            arr[i] = 0xFC;
            // arr[i] = 0xE0;
        }
        else
        {
            arr[i] = 0xC0;
            // arr[i] = 0x07;
        }
        if (i == 1)
        {
            if (data % 2 == 1)
            {
                arr[0] = 0xFC;
                // arr[0] = 0xE0;
            }
            else
            {
                arr[0] = 0xC0;
                // arr[0] = 0x07;
            }
        }
    }
}

void dec_spisend_data(uint8_t *color, RGB_color_t *sendDATA)
{
    // R
    ESP_LOGI(TAG, "=============== 1 =================");
    uint8_t R = color_dec(color[0]); // 先转化为理论要发送的数据，随后转化为电平高低持续时间
    ESP_LOGI(TAG, "=============== 2 =================");
    uint8_t G = color_dec(color[1]);
    ESP_LOGI(TAG, "=============== 3 =================");
    uint8_t B = color_dec(color[2]);
    ESP_LOGI(TAG, "============== END =================");

    // 扩容处理
    /**
     * 1个字节扩容至8字节
     * 每一个数据位提出，然后打包成结构体，等待发送。
     */

    uint16_t R_arr[8] = {};
    uint16_t G_arr[8] = {};
    uint16_t B_arr[8] = {};

    value_expand(R, R_arr);
    value_expand(G, G_arr);
    value_expand(B, B_arr);
    // 将数据传递给结构体，作为要发送的数据
    memcpy(sendDATA->R, R_arr, sizeof(R_arr));
    memcpy(sendDATA->G, G_arr, sizeof(G_arr));
    memcpy(sendDATA->B, B_arr, sizeof(B_arr));
}

esp_err_t spi_write(spi_device_handle_t spi, uint8_t *data, uint8_t len)
{
    esp_err_t ret;
    spi_transaction_t t;
    if (len == 0)
        return;               // no need to send anything
    memset(&t, 0, sizeof(t)); // Zero out the transaction

    gpio_set_level(PIN_NUM_CS, 0);

    t.length = len * 8;                         // Len is in bytes, transaction length is in bits.
    t.tx_buffer = data;                         // Data
    t.user = (void *)1;                         // D/C needs to be set to 1
    ret = spi_device_polling_transmit(spi, &t); // Transmit!
    assert(ret == ESP_OK);                      // Should have had no issues.

    gpio_set_level(PIN_NUM_CS, 1);
    return ret;
}

esp_err_t spi_read(spi_device_handle_t spi, uint8_t *data)
{
    spi_transaction_t t;

    gpio_set_level(PIN_NUM_CS, 0);

    memset(&t, 0, sizeof(t));
    t.length = 8;
    t.flags = SPI_TRANS_USE_RXDATA;
    t.user = (void *)1;
    esp_err_t ret = spi_device_polling_transmit(spi, &t);
    assert(ret == ESP_OK);
    *data = t.rx_data[0];

    gpio_set_level(PIN_NUM_CS, 1);

    return ret;
}

void app_main(void)
{
    esp_err_t ret;
    spi_device_handle_t spi;
    ESP_LOGI(TAG, "Initializing bus SPI%d...", SPI2_HOST + 1);

    spi_bus_config_t buscfg = {
        .miso_io_num = PIN_NUM_MISO, // MISO信号线
        .mosi_io_num = PIN_NUM_MOSI, // MOSI信号线
        .sclk_io_num = PIN_NUM_CLK,  // SCLK信号线
        .quadwp_io_num = -1,         // WP信号线，专用于QSPI的D2
        .quadhd_io_num = -1,         // HD信号线，专用于QSPI的D3
        .max_transfer_sz = 64 * 8,   // 最大传输数据大小
    };

    spi_device_interface_config_t devcfg = {
        // .clock_speed_hz = SPI_MASTER_FREQ_8M, // Clock out at 10 MHz,
        .clock_speed_hz = SPI_MASTER_FREQ_8M, // Clock out at 10 MHz,
        .mode = 0,                            // SPI mode 0
        /*
         * The timing requirements to read the busy signal from the EEPROM cannot be easily emulated
         * by SPI transactions. We need to control CS pin by SW to check the busy signal manually.
         */
        .spics_io_num = -1,
        .queue_size = 15, // 传输队列大小，决定了等待传输数据的数量
    };
    // Initialize the SPI bus
    ret = spi_bus_initialize(SPI2_HOST, &buscfg, DMA_CHAN);
    ESP_ERROR_CHECK(ret);

    ret = spi_bus_add_device(SPI2_HOST, &devcfg, &spi);
    ESP_ERROR_CHECK(ret);

    gpio_pad_select_gpio(PIN_NUM_CS);                 // 选择一个GPIO
    gpio_set_direction(PIN_NUM_CS, GPIO_MODE_OUTPUT); // 把这个GPIO作为输出

    uint8_t red[3] = {16, 16, 16}; // G R B
    RGB_color_t red_t;
    RGB_color_t send_rgb_arr[25] = {}; // 该数组只是存放结构体对象，前期并不赋值
    for (size_t i = 0; i < 25; i++)
    {
        dec_spisend_data(&red, &send_rgb_arr[i]);
    }

    while (1)
    {
        for (size_t i = 0; i < 5; i++)
        {
            /* code */
            ret = spi_write(spi, &send_rgb_arr[i], sizeof(red_t));
        }

        ESP_ERROR_CHECK(ret);
        // ESP_LOGI(TAG, "spi_write res =  %d", ret);
        vTaskDelay(10);
    }
}

```

- 合宙使用该灯带的参考代码

  ```lua
  --[[
  Description: 烟雾净化器
  Version: 1.0
  Author: yeele
  Date: 2022-04-14 11:36:21
  LastEditTime: 2022-04-14 11:36:22
  LastEditors: yeele
  --]] --
  PROJECT = "Cleaner"
  VERSION = "1.0.1"
  
  log.info("main", PROJECT, VERSION)
  
  -- sys库是标配
  _G.sys = require("sys")
  
  if wdt then
      wdt.init(15000)
      sys.timerLoopStart(wdt.feed, 10000)
  end
  
  -- 初始化spi
  local spi_device = spi.deviceSetup(0, nil, 0, 0, 8, 8000000, spi.MSB, 1, 1)
  
  -- hex 转二进制字符串
  local function dec_short_bin(num)
      if num == 0 then return "00000000" end
      if num == 1 then return "00000001" end
      if num == 2 then return "00000010" end
      -- 转16位二进制(反码形式)
      local bin = ""
      local rem = 0 -- 余数
      while true do
          rem = num % 2 -- 获取余数
          bin = bin .. rem
          num = math.ceil((num - rem) / 2)
          if num == 1 then break end
      end
      bin = string.reverse(bin .. "1") -- 翻转结果
      for i = 1, 8 - #bin do bin = "0" .. bin end
      return bin
  end
  
  -- 数据转码
  -- 0 0xC0
  -- 1 0xFC
  local function bin_decode(x)
      local xbin = ""
      for i = 1, #x do
          if (x:sub(i, i) == "1") then
              xbin = xbin .. "FC"
          else
              xbin = xbin .. "C0"
          end
      end
      return xbin
  end
  
  -- 编码
  local function color_convert(r, g, b)
      local rtbl = bin_decode(dec_short_bin(r))
      local gtbl = bin_decode(dec_short_bin(g))
      local btbl = bin_decode(dec_short_bin(b))
      local data, _ = string.fromHex(gtbl .. rtbl .. btbl)
      return data
  end
  
  -- 常用颜色映射表
  local ColorMap = {
      Black = color_convert(0, 0, 0),
      White = color_convert(0xff, 0xff, 0xff),
      Red = color_convert(0xff, 0, 0),
      Green = color_convert(0, 0xff, 0),
      Blue = color_convert(0, 0, 0xff),
      Yellow = color_convert(0xff, 0xff, 0),
      -- 自主调光
      Simple = color_convert(50, 50, 50)
  }
  
  --
  sys.taskInit(function()
      local all = ColorMap.White .. ColorMap.Red .. ColorMap.Yellow ..
                      ColorMap.Green .. ColorMap.Blue .. ColorMap.Green ..
                      ColorMap.Yellow .. ColorMap.Red .. ColorMap.White ..
                      ColorMap.Red .. ColorMap.Yellow .. ColorMap.Green ..
                      ColorMap.Blue .. ColorMap.Green .. ColorMap.Yellow ..
                      ColorMap.Red
  
      local smp = ""
      while true do
          for i = 1, 16 do smp = smp .. ColorMap.White end
          spi_device:send(smp)
          sys.wait(5000)
          smp = ""
          for i = 1, 16 do smp = smp .. ColorMap.Red end
          spi_device:send(smp)
          sys.wait(5000)
          smp = ""
          for i = 1, 16 do smp = smp .. ColorMap.Green end
          spi_device:send(smp)
          sys.wait(5000)
          smp = ""
          for i = 1, 16 do smp = smp .. ColorMap.Blue end
          spi_device:send(smp)
          sys.wait(5000)
          smp = ""
          for i = 1, 16 do smp = smp .. ColorMap.Yellow end
          spi_device:send(smp)
          sys.wait(5000)
      end
  end)
  
  -- 
  sys.run()
  
  ```



#### 数据处理函数

- 8 字节高位和低位呼唤位置

  ```c
  /**
   * @author: DearL
   * @brief: 取反，将16进制低位在前，高位在后，返回亦是1字节 16进制数
   * @parm: HEX
   * @return: ~HEX 取反后的16进制数
   */
  uint8_t color_dec(uint8_t data)
  {
      uint8_t ref = 0x00;
      uint8_t i = 7;
      // 循环7次，反向短除法。(最后结果反写)
      while (i)
      {
          uint8_t ret = 0;
          ret = data % 2; // 取余
          data = data / 2;
          printf("ret == %d, ", ret);
          ref = ref | (ret << i); // 向16进制末尾插入余数
          if (i == 1)             // 短除法最后一位
          {
              ref = ref | ((data % 2) << 0);
              printf("ret == %d, ", ret);
          }
          i--;
      }
      ESP_LOGI(TAG, "ref =  %d", ref);  // ESP32 打印输出
      return ref;
  }
  ```



#### 代码改进

> 对于数据分段，点亮较长的灯带，SPI 通过分段发送，让灯变量

- 改进部分:

  ```c
  void queue_write(spi_device_handle_t spi, RGB_color_t *rgb_t, uint8_t len)
  {
      uint8_t times = len / 10;
      uint8_t residue = len % 10; // 剩余SPI 需要发送的长度
      // 发送次数
      // uint8_t cycle_count = 10;
      for (size_t i = 0; i < times; i++)
      {
          for (size_t j = 0; j < 10; j++)
          {
              spi_write(spi, &rgb_t[j], sizeof(RGB_color_t));
          }
      }
      for (size_t i = 0; i < residue; i++)
      {
          spi_write(spi, &rgb_t[i], sizeof(RGB_color_t));
      }
  }
  
  // 伪代码，截取部分
  void test()
  {
      uint8_t red[3] = {255, 255, 255}; // G R B
      RGB_color_t send_rgb_arr_test[10] = {}; // TODO: 先以10 个灯珠为缓存，依次发送
      for (size_t i = 0; i < 10; i++)
      {
          dec_spisend_data(&red, &send_rgb_arr_test[i]);
      }
      while (1)
      {
          queue_write(spi, &send_rgb_arr_test, 130);
      }
  }
  
  ```

- 待改进部分: 发送不同颜色的灯的处理，显然上方代码将一个很长的灯带分成多份 10 个长度的灯，但 10 个灯只能亮同一种颜色 ( 如果去发送数据的代码改，那么就会增加耦合度 )

  ```c
  //TODO: 
  ```

  



#### 注意事项 && 碰到的问题

- 给设备发送数据时的现象

  > 在测试 SPI 发送数据阶段，使用示波器测试现象

  1. 如果采用 string 的方式发送时，比如 "1111" 等代码，示波器显示的波纹，可以看出，示波器时从左向右扫描，发送的数据  (char ) '1' = 0x31 = 0b00110001 因此波纹如下。

     ![image-20231027155541953](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311021420438.png)

  2. 后续测试不同的报文，示波器左边代表设备先接收到的数据，而且是字符串第一个字符。

  3. 发送 "1111" 和发送 {0x31,0x31,0x31,0x31} 效果是一样的。因此可以 hex 发送

  > 得出结论

  ESP32 发送出去的数据，数据组包高位 ( 最左边 ) 最先从 SPI 引脚发出。而且单个字节的高位 ( 左边 ) 先发。

  

- 数据编码时是否取反的问题

  &emsp;&emsp;由上方结论，由于 RGB 灯珠芯片的特性，这边使用结构体来存放灯珠的颜色，使用  `RGB_color_t send_rgb_arr[25] = {};` 结构体数组来使灯珠发光。

  > 关于编码时是否取反的问题

  为何提出这个问题？

  答：因为参考的 Lua 代码中就有编码取反的操作。随后我也跟着用 C 语言重构了，但是发现取反之后会存在问题 ( 不取反就完全正常 )。

  后续的采用的方法，

  答：当然是不取反，直接对 RGB 原始数据进行膨胀，编码

- RGB 原始数据进行编码的详解

  1. 0xFC == 0b11111100 , 0x03 == 0b0000 0011

     根据 Hex 所表示的二进制数，这边理解为一种占空比的形式？

     ( 制作灯珠的厂家文档并不规范，因此同一种 RGB 编码能驱动多种类型灯珠 )

- 由于有多个灯珠，而如果创建相同灯珠数量长度的结构体数组，就会导致内存占用完之后栈溢出

  当时对该问题的思考：

  1. 设置一个缓存队列，固定缓存队列的长度，这样就能控制栈的大小。但我没考虑发送数组组包的时候也得使用内存空间。
  2. 使用 malloc 来分配堆空间以组包数据，但是 malloc 依旧会占用特别大内存 ( 根据灯长度决定 )。
  3. 临时考虑采用 SPI 分段发送，使用长度为 10 的结构体数组作为缓存，如果亮 66 个灯，就发送7次。

  采用的方案: 

  &emsp;&emsp;采用的上述第三个思考的办法，先临时能让模块驱动很长的 RGB 灯，后续再想办法做成别的效果。

  不足之处: 

  &emsp;&emsp;采用这个办法会导致一组灯只能亮 同一种颜色，如果要改动亮灯的不同效果，还得在代码中继续更改。



- //TODO:

  内存空间利用率的问题，需要设置缓存将数据发送之后清除缓存区并添加后续的数据。

  这个如果用队列那会方便很多 ( 如果使用 STL  的 queue 容器就方便太多了 )。

  这里考虑使用 C 中写一个队列缓存

  

  

```c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/spi_master.h"
#include "driver/gpio.h"

#include "sdkconfig.h"
#include "esp_log.h"

#define DMA_CHAN 2
#define PIN_NUM_MISO 12
#define PIN_NUM_MOSI 13
#define PIN_NUM_CLK 14
#define PIN_NUM_CS 15

#define LED_NUM 33

static const char TAG[] = "main";

esp_err_t spi_write(spi_device_handle_t spi, uint8_t *data, uint8_t len);
// 使用结构体来存放数据
typedef struct
{
    uint16_t R[8];
    uint16_t G[8];
    uint16_t B[8];
} RGB_color_t;

/**
 * @author: DearL
 * @brief: 取反，将16进制低位在前，高位在后，返回亦是1字节 16进制数(根据短除法的逻辑写的代码)
 * @parm: HEX
 * @return: ~HEX 取反后的16进制数
 */
// uint8_t color_dec(uint8_t data)
// {
//     uint8_t ref = 0x00;
//     uint8_t i = 7;
//     // 循环7次，反向短除法。(最后结果反写)
//     while (i)
//     {
//         uint8_t ret = 0;
//         ret = data % 2; // 取余
//         data = data / 2;
//         printf("ret == %d, ", ret);
//         ref = ref | (ret << i); // 向16进制末尾插入余数
//         if (i == 1)             // 短除法最后一位
//         {
//             ref = ref | ((data % 2) << 0);
//             printf("ret == %d, ", ret);
//         }
//         i--;
//     }
//     ESP_LOGI(TAG, "ref =  %d", ref);
//     return ref;
// }

/**
 * @brief: 不取反  emmmm
 * @parm: HEX
 * @return: ~HEX 取反后的16进制数
 */
// uint8_t color_dec(uint8_t data)
// {
//     ESP_LOGI(TAG, "data =  %d", data);
//     return data;
// }

/**
 * @brief 将数据膨胀 8 字节，高位在前，这就是将要发给 RGB 灯带的数据
 * @parm: data: 原始数据， arr: 将要转化成的数组
 * @return: void
 */
uint8_t res = 0;
void value_expand(uint8_t data, uint16_t *arr)
{
    for (size_t i = 7; i > 0; i--)
    {
        res = data % 2;
        data = data / 2;
        if (res == 1)
        {
            arr[i] = 0xFC;
            // arr[i] = 0xE0;
        }
        else
        {
            arr[i] = 0x03;
            // arr[i] = 0xC0;
            // arr[i] = 0x07;
        }
        if (i == 1)
        {
            if (data % 2 == 1)
            {
                arr[0] = 0xFC;
                // arr[0] = 0xE0;
            }
            else
            {
                arr[0] = 0x03;
                // arr[0] = 0xC0;
                // arr[0] = 0x07;
            }
        }
    }
}

uint16_t R_arr[8] = {};
uint16_t G_arr[8] = {};
uint16_t B_arr[8] = {};
void dec_spisend_data(const uint8_t *color, RGB_color_t *sendDATA)
{
    // 扩容处理
    /**
     * 1个字节扩容至8字节
     * 每一个数据位提出，然后打包成结构体，等待发送。
     */
    // 数据大端的情况不使用color_dec() 函数
    // value_expand(color_dec(color[0]), &R_arr);
    // value_expand(color_dec(color[1]), &G_arr);
    // value_expand(color_dec(color[2]), &B_arr);
    value_expand(color[0], &R_arr);
    value_expand(color[1], &G_arr);
    value_expand(color[2], &B_arr);
    // 将数据传递给结构体，作为要发送的数据
    memcpy(sendDATA->R, R_arr, sizeof(R_arr));
    memcpy(sendDATA->G, G_arr, sizeof(G_arr));
    memcpy(sendDATA->B, B_arr, sizeof(B_arr));
}

/**
 * @brief: 根据彩灯的算法，该函数用于呼吸灯 && 流光灯等具体算法实现
 * @note: 为了避免过于暗淡，因此有一个85 的限位，防止亮度过低
 * @param: WheelPos 轮子位置，
 * @copyright: 复制于网上的算法
 */
void Wheel(uint8_t *sendArr, uint8_t WheelPos, uint8_t brightness)
{
    WheelPos = 255 - WheelPos;
    if (WheelPos < 85)
    {
        sendArr[0] = (255 - WheelPos * 3) * brightness / 255;
        sendArr[1] = 0;
        sendArr[2] = (WheelPos * 3) * brightness / 255;
        return;
    }
    if (WheelPos < 170)
    {
        WheelPos -= 85;
        sendArr[0] = 0;
        sendArr[1] = (WheelPos * 3) * brightness / 255;
        sendArr[2] = (255 - WheelPos * 3) * brightness / 255;
        return;
    }
    WheelPos -= 170;
    sendArr[0] = (WheelPos * 3) * brightness / 255;
    sendArr[1] = (255 - WheelPos * 3) * brightness / 255;
    sendArr[2] = 0;
    return;
}

// Slightly different, this makes the rainbow equally distributed throughout
// 渐变流光灯效果
/*
    TODO: 将发送数据做成缓存，20个为一组，发多少清空多少，避免内存占用太多
    待考虑的地方：结构体的装载速度要够快，在发送间隔之内要做到装载的结构体完成。
    如果 20 个为一组不能完成装载，那就在发送数据的时候装载。
*/

void rainbowCycle(spi_device_handle_t spi) // 发送数据
{
    uint16_t i, j;
    RGB_color_t send_color_data_t[LED_NUM] = {};
    uint8_t sendArr[3] = {};
    memset(&sendArr, 0, sizeof(sendArr));
    // for (j = 0; j < 10; j++)
    for (j = 0; j < 256 * 5; j++)
    {                                 // 5 cycles of all colors on wheel
        for (i = 0; i < LED_NUM; i++) // 根据长度来确定
        {
            Wheel(&sendArr, ((i * 256 / LED_NUM) + j), 64);
            dec_spisend_data(&sendArr, &send_color_data_t[i]);
        }
        ESP_LOGI(TAG, "-----------------------------------\n");
        for (size_t i = 0; i < LED_NUM; i++)
            spi_write(spi, &send_color_data_t[i], sizeof(RGB_color_t));
        vTaskDelay(1);
    }
}

// 渐变流光灯
void rainbow(spi_device_handle_t spi)
{
    uint16_t i, j;
    RGB_color_t send_color_data_t[LED_NUM] = {};
    uint8_t sendArr[3] = {};
    memset(&sendArr, 0, sizeof(sendArr));
    for (j = 0; j < 256; j++)
    {
        for (i = 0; i < LED_NUM; i++) // 循环130次， 130 为灯的个数
        {
            Wheel(&sendArr, (i + j), 255);
            dec_spisend_data(&sendArr, &send_color_data_t[i]);
        }
        for (size_t i = 0; i < LED_NUM; i++)
            spi_write(spi, &send_color_data_t[i], sizeof(RGB_color_t));
        vTaskDelay(1);
    }
}

// 剧场追逐样式  //TODO:未完成
//  uint32_t 表示颜色
void theaterChase(spi_device_handle_t spi, const uint8_t *sendArr)
{
    //{255, 255, 0}
    // uint8_t sendArr[3] = {255, 255, 0};
    RGB_color_t send_color_data_t[LED_NUM] = {};
    // memset(&sendArr, 0, sizeof(sendArr));  //请勿使用该函数memset 形参
    uint16_t nocolor[8] = {};

    for (int j = 0; j < 10; j++)
    { // do 10 cycles of chasing
        for (int q = 0; q < 3; q++)
        {
            value_expand(sendArr[0], &R_arr);
            value_expand(sendArr[1], &G_arr);
            value_expand(sendArr[2], &B_arr);
            value_expand(0, &nocolor);
            for (uint16_t i = 0; i < LED_NUM; i = i + 3)
            {
                /*
                    产生现象: 只亮2个灯
                    猜测原因: 因为只隔3个才填充数据导致,中部缺失的数据没有传入,
                            超过间隔时间之后导致后面的灯不亮.
                */
                // 装载进结构体
                if ((i + q) >= LED_NUM)
                    break;
                // 将数据传递给结构体，作为要发送的数据
                memcpy(send_color_data_t[i + q - 2].R, nocolor, sizeof(R_arr));
                memcpy(send_color_data_t[i + q - 2].G, nocolor, sizeof(G_arr));
                memcpy(send_color_data_t[i + q - 2].B, nocolor, sizeof(B_arr));
                memcpy(send_color_data_t[i + q - 1].R, nocolor, sizeof(R_arr));
                memcpy(send_color_data_t[i + q - 1].G, nocolor, sizeof(G_arr));
                memcpy(send_color_data_t[i + q - 1].B, nocolor, sizeof(B_arr));
                memcpy(send_color_data_t[i + q].R, R_arr, sizeof(R_arr));
                memcpy(send_color_data_t[i + q].G, G_arr, sizeof(G_arr));
                memcpy(send_color_data_t[i + q].B, B_arr, sizeof(B_arr));

                // strip.setPixelColor(i + q, c); // turn every third pixel on
                // spi_write(spi, &send_color_data_t[i + q], sizeof(RGB_color_t));
            }
            // 发送具体数据编码
            for (size_t i = 0; i < LED_NUM; i++)
                spi_write(spi, &send_color_data_t[i], sizeof(RGB_color_t));
            vTaskDelay(50);
            for (uint16_t i = 0; i < LED_NUM; i = i + 3)
            {
                if ((i + q) >= LED_NUM)
                    break;
                memset(&send_color_data_t[i + q], nocolor, sizeof(send_color_data_t[i + q]));
                memset(&send_color_data_t[i + q - 1], nocolor, sizeof(send_color_data_t[i + q]));
                memset(&send_color_data_t[i + q - 2], nocolor, sizeof(send_color_data_t[i + q]));
                // strip.setPixelColor(i + q, 0); // turn every third pixel off
            }
        }
    }
}

// TODO:未完成,算法实现依旧有点怪异 如果要置0,不能直接传0 进去
void theaterChaseRainbow(spi_device_handle_t spi)
{
    RGB_color_t send_color_data_t[LED_NUM] = {};
    uint8_t sendArr[3] = {};
    for (int j = 0; j < 256; j++)
    { // cycle all 256 colors in the wheel
        for (int q = 0; q < 3; q++)
        {
            for (uint16_t i = 0; i < LED_NUM; i = i + 3)
            {
                if ((i + q) >= LED_NUM)
                    break;
                Wheel(&sendArr, (i + j) % 255, 128);
                dec_spisend_data(&sendArr, &send_color_data_t[i + q]);
                memset(&sendArr, 0, sizeof(sendArr));
                dec_spisend_data(&sendArr, &send_color_data_t[i + q - 1]);
                dec_spisend_data(&sendArr, &send_color_data_t[i + q - 2]);
                // strip.setPixelColor(i + q, Wheel((i + j) % 255)); // turn every third pixel on
            }

            for (size_t i = 0; i < LED_NUM; i++)
                spi_write(spi, &send_color_data_t[i], sizeof(RGB_color_t));

            vTaskDelay(10);

            for (uint16_t i = 0; i < LED_NUM; i = i + 3)
            {
                if ((i + q) >= LED_NUM)
                    break;
                // i+q 置为0, 为何不 i+q-1,i+q-2 也置为0? 因为上面已经做了这件事
                dec_spisend_data(&sendArr, &send_color_data_t[i + q]);
                // strip.setPixelColor(i + q, 0); // turn every third pixel off
            }
        }
    }
}

// 流星
/**
 * @note: 将实现效果比作贪吃蛇
 *
 */
void meteor(spi_device_handle_t spi, uint8_t red, uint8_t green, uint8_t blue)
{
    uint8_t sendArr[3] = {};
    RGB_color_t send_color_data_t[LED_NUM] = {};

    uint8_t noclolor[3] = {};
    noclolor[0] = 0;
    noclolor[1] = 0;
    noclolor[2] = 0;
    // ---------
    const uint8_t num = 3;
    uint8_t max_color = red;
    if (green > max_color) // 选举单色光中最高光强
        max_color = green;
    if (blue > max_color)
        max_color = blue;
    uint8_t instance = (max_color - 200) / num;  // 55/15 == 3
    for (uint16_t i = 0; i < LED_NUM + num; i++) // 灯珠个数+队列最后一位
    {
        for (uint8_t j = 0; j < num; j++) // 蛇头
        {
            if (i - j >= 0 && i - j < LED_NUM) // 渐变 变暗  instance为变暗因子
            {
                int red_after = red - (instance * j);
                int green_after = green - (instance * j);
                int blue_after = blue - (instance * j);

                if (j >= 1) // 蛇身,亮度比蛇头暗很多
                {
                    red_after -= 200;
                    green_after -= 200;
                    blue_after -= 200;
                }
                sendArr[0] = red_after >= 0 ? red_after : 0;
                sendArr[1] = green_after >= 0 ? green_after : 0;
                sendArr[2] = blue_after >= 0 ? blue_after : 0;
                dec_spisend_data(&sendArr, &send_color_data_t[i - j]);
            }
        }
        if (i - num >= 0 && i - num < LED_NUM) // 删掉队尾
        {
            dec_spisend_data(&noclolor, &send_color_data_t[i - num]);
        }

        for (size_t i = 0; i < LED_NUM; i++)
            spi_write(spi, &send_color_data_t[i], sizeof(RGB_color_t));
        vTaskDelay(8);
    }
}

// void meteor_overturn(uint8_t red, uint8_t green, uint8_t blue, uint8_t wait)
// {

//   const uint8_t num = 15;
//   uint8_t max_color = red;
//   if (green > max_color)
//     max_color = green;
//   if (blue > max_color)
//     max_color = blue;
//   uint8_t instance = (max_color - 200) / num;
//   for (int i = strip.numPixels() - 1; i >= -num; i--)
//   {
//     for (uint8_t j = 0; j < num; j++)
//     {
//       if (i + j >= 0 && i + j < strip.numPixels())
//       {
//         int red_after = red - instance * j;
//         int green_after = green - instance * j;
//         int blue_after = blue - instance * j;
//         if (j >= 1)
//         {
//           red_after -= 200;
//           green_after -= 200;
//           blue_after -= 200;
//         }
//         strip.setPixelColor(i + j, strip.Color(red_after >= 0 ? red_after : 0, green_after >= 0 ? green_after : 0, blue_after >= 0 ? blue_after : 0));
//       }
//     }
//     if (i + num >= 0 && i + num < strip.numPixels())
//       strip.setPixelColor(i + num, 0);

//     strip.show();
//     delay(wait);
//   }
// }

esp_err_t spi_write(spi_device_handle_t spi, uint8_t *data, uint8_t len)
{
    esp_err_t ret;
    spi_transaction_t t;
    if (len == 0)
        return;               // no need to send anything
    memset(&t, 0, sizeof(t)); // Zero out the transaction

    gpio_set_level(PIN_NUM_CS, 0);

    t.length = len * 8;                         // Len is in bytes, transaction length is in bits.
    t.tx_buffer = data;                         // Data
    t.user = (void *)1;                         // D/C needs to be set to 1
    ret = spi_device_polling_transmit(spi, &t); // Transmit!
    assert(ret == ESP_OK);                      // Should have had no issues.

    gpio_set_level(PIN_NUM_CS, 1);
    return ret;
}

esp_err_t spi_read(spi_device_handle_t spi, uint8_t *data)
{
    spi_transaction_t t;

    gpio_set_level(PIN_NUM_CS, 0);

    memset(&t, 0, sizeof(t));
    t.length = 8;
    t.flags = SPI_TRANS_USE_RXDATA;
    t.user = (void *)1;
    esp_err_t ret = spi_device_polling_transmit(spi, &t); // SPI 设备轮询
    assert(ret == ESP_OK);
    *data = t.rx_data[0];

    gpio_set_level(PIN_NUM_CS, 1);

    return ret;
}

void queue_write(spi_device_handle_t spi, RGB_color_t *rgb_t, uint8_t len)
{
    uint8_t times = len / 10;
    uint8_t residue = len % 10; // 剩余SPI 需要发送的长度
    // 发送次数
    // uint8_t cycle_count = 10;
    for (size_t i = 0; i < times; i++)
    {
        for (size_t j = 0; j < 10; j++)
        {
            spi_write(spi, &rgb_t[j], sizeof(RGB_color_t));
        }
    }
    for (size_t i = 0; i < residue; i++)
    {
        spi_write(spi, &rgb_t[i], sizeof(RGB_color_t));
    }
}

void app_main(void)
{
    esp_err_t ret;
    spi_device_handle_t spi;
    ESP_LOGI(TAG, "Initializing bus SPI%d...", SPI2_HOST + 1);

    spi_bus_config_t buscfg = {
        .miso_io_num = PIN_NUM_MISO, // MISO信号线
        .mosi_io_num = PIN_NUM_MOSI, // MOSI信号线
        .sclk_io_num = PIN_NUM_CLK,  // SCLK信号线
        .quadwp_io_num = -1,         // WP信号线，专用于QSPI的D2
        .quadhd_io_num = -1,         // HD信号线，专用于QSPI的D3
        .max_transfer_sz = 64 * 8,   // 最大传输数据大小
    };

    spi_device_interface_config_t devcfg = {
        // .clock_speed_hz = SPI_MASTER_FREQ_8M, // Clock out at 10 MHz,
        .clock_speed_hz = SPI_MASTER_FREQ_8M, // Clock out at 10 MHz,
        .mode = 0,                            // SPI mode 0
        /*
         * The timing requirements to read the busy signal from the EEPROM cannot be easily emulated
         * by SPI transactions. We need to control CS pin by SW to check the busy signal manually.
         */
        .spics_io_num = -1,
        .queue_size = 15, // 传输队列大小，决定了等待传输数据的数量
    };
    // Initialize the SPI bus
    ret = spi_bus_initialize(SPI2_HOST, &buscfg, DMA_CHAN); // SPI 总线的初始化 总线包含 MISO,MOSI,CLK,默认空闲高低电平,传输数据大小
    ESP_ERROR_CHECK(ret);

    ret = spi_bus_add_device(SPI2_HOST, &devcfg, &spi); // spi 设备初始化 设备包含: 时钟、模式、硬件片选IO、传输队列
    ESP_ERROR_CHECK(ret);

    gpio_pad_select_gpio(PIN_NUM_CS);                 // 选择一个GPIO
    gpio_set_direction(PIN_NUM_CS, GPIO_MODE_OUTPUT); // 把这个GPIO作为输出

    // uint8_t red[3] = {4, 13, 4}; // G R B
    // uint8_t red[3] = {40, 130, 40}; // G R B
    // uint8_t red[3] = {80, 255, 80}; // G R B
    // uint8_t red[3] = {128, 128, 128}; // G R B
    // uint8_t red[3] = {0, 0, 255};   // G R B
    uint8_t red[3] = {255, 255, 0};            // G R B  // 黄灯
    uint8_t test_theaterChase[3] = {32, 0, 0}; // G R B  // 黄灯

    // RGB_color_t send_rgb_arr_test[10] = {}; // TODO: 先以10 个灯珠为例，依次发送
    // // RGB_color_t *send_rgb_arr_test2 = (RGB_color_t *)malloc(sizeof(RGB_color_t) * 10); // 在堆中开辟数组
    // for (size_t i = 0; i < 10; i++)
    // {
    //     dec_spisend_data(&red, &send_rgb_arr_test[i]);
    // }
    while (1)
    {
        // queue_write(spi, &send_rgb_arr_test, 130); // 数据连续发送
        // rainbowCycle(spi);
        // rainbow(spi);
        // theaterChase(spi, test_theaterChase); // 取名为觥筹交错,挺难看的.效果
        // theaterChaseRainbow(spi);
        meteor(spi, 255, 0, 255);

        ESP_ERROR_CHECK(ret);
        ESP_LOGI(TAG, "spi_write res =  %d", ret);
        // vTaskDelay(20);
    }
}


```







