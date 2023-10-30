### ESP32 驱动 RGB 灯带

[toc]

#### 环境搭建

- python 环境，电脑已经安装了python 3.9.13 因此跳过这一步

  ![image-20231023172119370](ESP32%20%E9%A9%B1%E5%8A%A8%20RGB%20%E7%81%AF%E5%B8%A6.assets/image-20231023172119370.png)

- [环境搭建环境教程文档](https://aijishu.com/a/1060000000295342)

- 在 VSCode 中安装 'ESP-IDF EXPLORER' 插件，并使用 VS code 终端 PowerShell 编译。

  ![image-20231027114351647](ESP32%20%E9%A9%B1%E5%8A%A8%20RGB%20%E7%81%AF%E5%B8%A6.assets/image-20231027114351647.png)

- `idf.py build` 进行编译,  

  `idf.py -p COM124 flash`  进行烧录固件





#### RGB 控制

[参考控制原理教程](https://blog.csdn.net/gengyuchao/article/details/93239317)

- 接收芯片采用类似PWM 编码的形式

- 根据灯珠的芯片手册，发现 PWM、GPIO 等方式并不能满足 时序翻转所需的 3us 因此，根据网上教程，选用 SPI 进行模拟 PWM 收发。

- 控制原理

  根据输入的高低电平持续时间的占比来决定数字是 0 还是 1，可以根据固定的时钟，来对应 SPI 持续发送字节来表示输入的是高电平还是低电平

  ![image-20231023183820806](ESP32%20%E9%A9%B1%E5%8A%A8%20RGB%20%E7%81%AF%E5%B8%A6.assets/image-20231023183820806.png)

  ![image-20231027115043784](ESP32%20%E9%A9%B1%E5%8A%A8%20RGB%20%E7%81%AF%E5%B8%A6.assets/image-20231027115043784.png)

  注:  该灯珠文档有误，24bit 的数据结构标反，应该时 GRB 的顺序结构。

- 根据上图，可以明白，RGB 分别是 24bit 控制一个灯，但，我们采用的是 SPI 模拟 PWM 信号，而**由多个 SPI 的信号才能表示单个 1 or 0**。根据码型时间和 SPI 时钟频率，我这边选用的 **1字节(  8bit ) 来表示 24 bit 中的一个bit**。



- ( 待优化 ) 频率选择，由于我们想要 0.3 微秒，因此 esp_flash_speed_s 需要 10MHz 来表示我们确切的时序( **后来我并没有这么做，依旧采用的 8MHz 来发送高低电平** )。

  (3bit->1 , 6bit->0) 代表低电平。

  ![image-20231023190916403](ESP32%20%E9%A9%B1%E5%8A%A8%20RGB%20%E7%81%AF%E5%B8%A6.assets/image-20231023190916403.png)





#### ESP32 控制 RGB 灯的代码实现

> 参考教程

[API 中文 SPI Master 部分文档 ( 参考代码 )](https://juejin.cn/post/7108542432051986439)

[SPI 模拟 PWM 点亮 RGB 灯原理文档](https://blog.csdn.net/gengyuchao/article/details/93239317)

- ESP32 引脚描述

  ![引脚图](ESP32%20%E9%A9%B1%E5%8A%A8%20RGB%20%E7%81%AF%E5%B8%A6.assets/%E5%BC%95%E8%84%9A%E5%9B%BE.png)

![image-20231024091405830](ESP32%20%E9%A9%B1%E5%8A%A8%20RGB%20%E7%81%AF%E5%B8%A6.assets/image-20231024091405830.png)



> 实现部分代码 ( 讲解在后面 )

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

  

#### 注意事项 && 碰到的问题

- 给设备发送数据时的现象

  > 在测试 SPI 发送数据阶段，使用示波器测试现象

  1. 如果采用 string 的方式发送时，比如 "1111" 等代码，示波器显示的波纹，可以看出，示波器时从左向右扫描，发送的数据  (char ) '1' = 0x31 = 0b00110001 因此波纹如下。

     ![image-20231027155541953](ESP32%20%E9%A9%B1%E5%8A%A8%20RGB%20%E7%81%AF%E5%B8%A6.assets/image-20231027155541953.png)

  2. 后续测试不同的报文，示波器左边代表设备先接收到的数据，而且是字符串第一个字符。

  3. 发送 "1111" 和发送 {0x31,0x31,0x31,0x31} 效果是一样的。因此可以 hex 发送

  > 得出结论

  ESP32 发送出去的数据，高位 ( 最左边 ) 最先从 SPI 引脚发出。

  

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





- 









