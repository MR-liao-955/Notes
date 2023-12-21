### C中常用库函数 （TODO: 持续更新）

[toc]

- string 数据处理

  1.  strstr( ) 查找子串

     ```c
     /*
     匹配test_rx_buf 中是否有 "upload",如有就返回其索引
     函数返回值是一个指向第一次出现子字符串的指针，如果未找到子字符串，则返回NULL
     */
     if (strstr(test_rx_buf, "upload") != NULL)
     {
         upload_suc = 1;
         break;
     }
     ```

  2. atoi( ) 字符串转数组( 后面自建函数我也自己写了一个仿atoi( ) 函数 )

     ```c
     // atoi 将字符串转化为数字
     int main() {
         const char *str = "12345";
         int value = atoi(str);
     
         printf("String: %s\n", str);
         printf("Integer value: %d\n", value);
     
         return 0;
     }
     // str_toNumb(recv_FLASH, strlen(recv_FLASH));  // 后面我也自建了函数
     ```

  3. itoa( ) 函数与 sprintf( ) 函数。

     ```c
     //sprintf() 函数，将格式化数据输出到字符串中。
     /*
     	玩法多样：可以转char为string 也可以转化为int为string，float 也能转
     */
         char devreg[64] = {0};
         sprintf(devreg, "{\"title\":\"%s\",\"sn\":\"%s\"}", devsn, devsn);
     	sprintf(res_jsonStr, "%d", res->valueint);
     
     
     // itoa() 不是标准C库函数，但是某些编译器提供了这个函数。
     /* 
     	它接收一个整数和一个字符数组，并将整数转化为字符串
     */
     void test2() {  
         int number = 12345;
         char str[20];
     
         itoa(number, str, 10);
     
         printf("Number: %d\n", number);
         printf("String: %s\n", str);
     }
     
     ```

  4. strlen( ) 函数，输出 字符数组的长度

     ```c
     // 以socket 的send() 函数为例
     ret = send(socketid, args, strlen(args), 0);
     ```

  5. strcat( ) 函数，将字符串添加到 字符串尾部

     ```c
     // 这一块字符串拼接，拼接到sendMESSAGE
     char sendMESSAGE[109] = {0};
     strcat(sendMESSAGE, uploadDATA.sim_Info.imei);
     strcat(sendMESSAGE, uploadDATA.sim_Info.imsi);
     strcat(sendMESSAGE, uploadDATA.sim_Info.iccid);
     strcat(sendMESSAGE, temp);
     ```

  6. strncpy( ) 函数，

     ```c
     // 字符串拷贝 最后一个参数是要复制的字符数
     strncpy(OutputStr, strBuff, BuffLen); // 将strBuff复制到OutputStr
     ```

  7. strncmp( ) 函数，

     ```c
     // 用于对比 str1 和str2 的前5个字符是否相等
     /*
     	完全相等返回0，
     	如果str1前面某个数 字典顺序小于 str2 返回负数，
     	如果str1前面某个数 字典顺序大于 str2 返回正数。
     */
     
     const char *str1 = "Hello";
     const char *str2 = "Hello, World!";
     int result = strncmp(str1, str2, 5);
     ```

     



- 网络编程 && socket 之类的函数

  

  1. setsockopt( ) 函数，定义发送/接收 阻塞超时时间。[参考地址，详解](https://blog.csdn.net/A493203176/article/details/65438182)
  
     ```c
     // socket 或者 tcp 发送/接收请求(阻塞)定义超时
     struct timeval tv_out;
     tv_out.tv_sec = 5;
     tv_out.tv_usec = 0;
     setsockopt(socketid, SOL_SOCKET, SO_RCVTIMEO, (char *)&tv_out, sizeof(tv_out));
     ```
  
  2. socket( ) 函数，本地生成socket
  
     ```c
     socketid = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP); // 本地生成socketid 进程号
     ```
  
  3. connect( ) 函数，与服务器建立TCP 长链接
  
     ```c
     // tcp 建立连接所需的ip、端口、服务器端口号、脚本名。
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
         "newScript"   // 服务器上脚本的文件名
     }; 
     
     struct sockaddr_in server_addr;
     memset(&server_addr, 0, sizeof(server_addr));
     server_addr.sin_len = sizeof(server_addr);
     server_addr.sin_family = AF_INET;
     server_addr.sin_port = htons(s_conf.port);
     server_addr.sin_addr.s_addr = inet_addr(s_conf.ip); // 这里获取到IP 地址和端口，可以建立连接
     
     ret = connect(socketid, (const struct sockaddr *)&server_addr, sizeof(server_addr));
     ```
  
  4. send( ) 函数，阻塞状态,直到发送完毕,与设备建立连接、向设备发送数据。(  本质都是一样的，都是发送数据。因为连接已经由connect() 函数建立 )
  
     ```c
     // 发送报文，与设备建立连接
     ret = send(socketid, args, strlen(args), 0);
     ```
  
  5. recv( ) 函数，阻塞状态,直到收到服务器发过来的数据。(可以通过setsockopt( ) 定义阻塞超时时间 )
  
     ```c
     // 接收服务器回传信息，传入char 型数组地址，用于存放服务器下发数据
     ret = recv(socketid, test_rx_buf, sizeof(test_rx_buf), 0);
     ```
  
     
  
  

- 内存管理函数

  1. memset( ) 函数，为特定的内存设置初值（通常和malloc 一起使用malloc 为它开辟堆空间）

     ```c
     // uart0_rx_buf 开辟的内存空间进行初始化。
     static uint8_t uart0_rx_buf[128] = {0};
     memset(uart0_rx_buf, 0, sizeof(uart0_rx_buf));
     ```

  2. memcpy( ) 函数，

     ```c
     uint8_t toArray[29] = {0};
     int uint_SIZE = sizeof(uploadDATA.dev) + sizeof(uploadDATA.sensor);
     
     uint8_t *pStruct = &uploadDATA.dev.ver; // 创建结构体指针，
     memcpy(toArray, pStruct, uint_SIZE);   //通过指针进行内存的拷贝，数据大小为uint8_t 需要转化为string 的长度
     ```

  3. malloc( ) 函数，分配动态内存

     ```c
     
     // 下方使用malloc 开辟动态内存空间， 以下就是用malloc 模仿动态数组
     int uint_SIZE = sizeof(uploadDATA.dev) + sizeof(uploadDATA.sensor);
     uint8_t *const toArray = (uint8_t *)malloc(uint_SIZE); // 本质就是数组（指针常量，指针不可变，即为引用&）
     ```

     

  4. free( ) 函数，释放分配的内存空间

     ```c
     // 释放之前malloc 开辟的堆
     free(toArray);
     ```

- CJSON 

  1. 结构体定义

     ```c
     cJSON *json;
     cJSON *ac;
     cJSON *res;
     ```

  2. cJSON_Parse( ) 函数，

     ```c
     // 将uart、TCP 接收的数据转化为JSON "结构体"
     // 解析之后应当还能用子json 来解析
     json = cJSON_Parse(test_rx_buf);
     ```

  3. cJSON_GetObjectItem( ) 函数，

     ```c
     // 在json 结构体中查找"ac" 对象。如果没有就返回NULL，或者ac->type 就变成cJSON_NULL 这个宏定义
     ac = cJSON_GetObjectItem(json, "ac"); 
     
     if (ac == NULL || ac->type == cJSON_NULL) // 发包key 必须是ac
     {
         cm_demo_printf(" test_rx_buf don't include \"ac \" !!,recv MSG type error !\n ");
         continue;
     }
     ```

     



- 其它常用函数

  ```c
  // TODO: 待整理
  
  
  ```

  



### 自建工具函数

##### 1. 16进制的 string 转化为 int类型（查表法 lua/C 实现）

```lua
--[[
    @auther: DearL
    @param : 原始string
    @brief : 将16进制的string 转化为int，string必须小写，
    @date : 2023.5.11
]]
function str_toNumb(str)
    local tb_num = {
        '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e',
        'f'
    }
    local ret_num = 0;
    str_len = str:len() -- 长度，思路：根据长度来计算出string 直接转化为number 的数
    for i = 1, str_len do
        -- print("str:byte(i) = ",str:sub(i,i))
        for j = 1, 15 do -- 用于匹配 tb_num 查表
            if (tb_num[j] == str:sub(i, i)) then
                local mutply = 1 -- 倍数
                for k = 1, str_len - i do mutply = mutply * 16 end
                ret_num = ret_num + mutply * j -- 这里的j 代表基数，索引为i 时应该对应的值
                break
            end
        end
    end
    return ret_num;
end
```

```c
/**
 *
 * @auther: DearL
 * @param : 原始string , 字符串长度
 * @brief : 将16进制的string 转化为int，string必须小写，
 * @date : 2023.5.13
 * @return : 经过转化的数
 *
 */
uint32_t str_toNumb(char *str, uint16_t str_len)
{
    char char_Base[] = {'1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e',
                        'f'};
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



##### 2. 数组转化为string 

```c
/**
 *  数组转化为string
 *  传入 unsigned char ArrayHex[16] = { 0x1a, 0x2b, 0x3c, 0x4d, 0x5e, 0x6f, 0x7b, 0x8d, 0x9e, 0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16 };
 *  输出 printf("after ArrayToStr :%s\n", str);
 *控制台：after ArrayToStr :1a2b3c4d5e6f7b8d9e10111213141516
 *
 *
 */
int ArrayToStr(unsigned char *Buff, unsigned int BuffLen, char *OutputStr)
{
    int i = 0;
    char TempBuff[128] = {0};
    char strBuff[256] = {0};

    for (i = 0; i < BuffLen; i++)
    {
        sprintf(TempBuff, "%02x", Buff[i]);      // 以十六进制格式输出到TempBuff，宽度为2
        strncat(strBuff, TempBuff, BuffLen * 2); // 将TempBuff追加到strBuff结尾
    }
    strncpy(OutputStr, strBuff, BuffLen * 2); // 将strBuff复制到OutputStr
    return BuffLen * 2;
}
```



##### 3.二分法查找 ,以NTC 热敏电阻 为例

有些情况下，使用ADC 测温，硬件工程师通常会给出一个公式，通过公式计算ADC 转化为电阻 再转化成温度。但是这个项目公式法只适用于温度 >0 ℃ 的情况，而且准确性不如查表（可能因为ln 函数的问题）

- 使用公式法

```c
/**
 * 从math.h 中写的ln(x) 公式。用于计算ln值（如果导表就太浪费资源了）
 */
double myln(double a)
{
    int N = 15; // 我们取了前15+1项来估算
    int k, nk;
    double x, xx, y;
    x = (a - 1) / (a + 1);
    xx = x * x;
    nk = 2 * N + 1;
    y = 1.0 / nk;
    for (k = N; k > 0; k--)
    {
        nk = nk - 2;
        y = 1.0 / nk + xx * y;
    }
    return 2.0 * x * y;
}

/**
 * @param Rntc 获取到的电阻值
 * @param temperature 温度值的float指针用于直接获取温度
 *
 */
void Get_Kelvin_Temperature(float Rntc, float *temperature)
{
    float N1, N2, N3;
    N1 = (myln(R25) - myln(Rntc)) / B;
    N2 = 1 / T25 - N1;
    N3 = 1 / N2;
    *temperature = N3 - 273.15;  // 开尔文温度转化为 摄氏度
}
```



- 使用查表法

```c
/**
 * @author DearL
 * @brief ADC电压转温度  再查表计算
 * @param ADC_voltage
 * @return int temperarture
 *
 */
uint16_t ADC_convert_temp(const uint16_t voltage)
{
    // uint8_t temp;
    // !! voltage 值没传进来
    // TODO: voltage 直接转化为10进制数，而不是float 类型
    /*
        NTC电阻值 R = 10 * ADCval / (0xFFF -ADCval  )
        ADCval 为采集的NTC值，十六进制
    */

    float temp_R;
    temp_R = (voltage * 10) / (4095 - voltage);  // 由硬件工程师给计算公式
    // cm_demo_printf("voltage = %04x \n", voltage);
    // cm_demo_printf("voltage = %d \n", voltage);
    // cm_demo_printf("temp_R = %f \n", temp_R);
    int16_t temp = judge_tp_NTC(temp_R);
    if (temp < 0)
        return (uint16_t)(-temp + (1 << 12));

    return temp;
}
```

```c

/**
 * 查表
 */
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
 * @author DearL
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

```



##### 4.随机数的产生(MN316版本)

通常情况下随机数种子从 "time.h" 中拿就行了，但是MN316中，可能是time.h 文件的time(NULL) 函数不能拿到时间，导致每次获取的随机数 都一样（刷固件也一样的结果）。因此，我们采用MN316 "cm_rtc.h"库函数的获取的UTC 时间充当随机数种子。

```c
/**
 * @brief 获取随机数
 * @author DearL
 * @date 2023.5.17
 * @return 0~9 的随机数
 */
#include "stdlib.h"
//#include "time.h"  这里不使用time.h 
int get_randomNumb()
{
    srand((unsigned int)cm_rtc_get_time());
    // 随机数种子:
    // srand((unsigned int)time(NULL));  // 不能用C库的time ，可能是MN316对C库的随机数支持不到位
    return rand() % 10;
}

/*
说明：
一般情况要使用  time.h   stdlib.h 头文件，但这里就不使用time.h了。
因为在 cm_rtc.h 引用过 time.h

*/

```



##### 5.两字节 大端小端转换

往结构体中放数据，待解析时会出现2字节及以上的字节翻转，高低位交换。因此在处理数据的时候我们选择将大小端进行转换。以下是一个2字节的转换案例，如果需要更多字节，可以更新一下接口函数

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



##### 6. 一字节 ( 8bit ) 高位和低位的位置互换 + 数据替换

应用场景: 目前写这个函数的时候是为了驱动 RGB 灯带。

> 高低位互换

```c
/**
 * @author: DearL
 * @brief: 取反，将16进制低位在前，高位在后，返回亦是1字节 16进制数
 * @parm: HEX
 * @return: ~HEX 取反后的16进制数
 * @date: 2023.10.26
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

> 数据替换 ( 数据膨胀 )

用途：写该函数时，是使用 SPI 模拟类似 PWM 信号，根据时序计算出发送多少个 1，表示 MOSI 高位占用时间，也可以用模拟占空比。0xFC = 0b1111 1100 ，0x03 = 0b0000 0011 , 当占空比较大时 表示为 1。

```c
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
        }
        else
        {
            arr[i] = 0x03;
        }
        if (i == 1)
        {
            if (data % 2 == 1)
            {
                arr[0] = 0xFC;
            }
            else
            {
                arr[0] = 0x03;
            }
        }
    }
}
```



##### 7. 检查数组是否有重复值(算法)

设计原则,  创建临时数组，并设置标志位。

```c
/**
 	array[i]    					---> 待检查的数组
	valuesAsIndexes[array[i]]		---> 临时保存用的数组
*/
bool invalid = false;
int valuesAsIndexes[size]; // 初始化数组
for (unsigned i = 0; i < size; i++)
{
    valuesAsIndexes[i] = -1;
}

// 检查是否有重复值[算法]
if (!invalid)  // 检查是否有重复值[算法]。值得学
{
    for (unsigned i = 0; i < size; i++)
    {
        /*
                    假设array[4] = {2, 1, 3, 2}
                    val[2] = 2
                    val[1] = 1
                    val[3] = 3
                    val[2] != -1  ==> invalid = true ==> 数据重构
                */
        if (-1 != valuesAsIndexes[array[i]])  // 如果array[i]，有负数，则判断标志位非法
        {
            invalid = true;  // 如果没有重复值就设置标志位
            break;
        }
        valuesAsIndexes[array[i]] = array[i]; // 重复值
    }
}
```

代码来源: 查看 AT32 的 Betaflight 源码中 dshot.c 中的函数，查看到找数组重复值的方法。

```c
void validateAndfixMotorOutputReordering(uint8_t *array, const unsigned size)
{
    bool invalid = false;
    for (unsigned i = 0; i < size; i++)  // 确保数据不会超出范围才能使用下方的是否重复判断
    {
        if (array[i] >= size)
        { // 数组值不能大于数组长度，否则标志位判断为非法
            invalid = true;
            break;
        }
    }
    int valuesAsIndexes[size]; 
    for (unsigned i = 0; i < size; i++)
    {
        valuesAsIndexes[i] = -1;
    }
    if (!invalid)  // 检查是否有重复值[算法]。值得学
    {
        for (unsigned i = 0; i < size; i++)
        {
            /*
                假设array[4] = {2, 1, 3, 2}
                val[2] = 2
                val[1] = 1
                val[3] = 3
                val[2] != -1  ==> invalid = true ==> 数据重构
            */
            /*
                假设array[4] = {2, 1, 7, 2}
                val[2] = 2
                val[1] = 1
                val[7] = 7  // 显然没有 val[7]
                val[2] != -1  ==> invalid = true ==> 数据重构
            */
            if (-1 != valuesAsIndexes[array[i]])  // 如果array[i]，有负数，则判断标志位非法
            {
                invalid = true;
                break;
            }
            valuesAsIndexes[array[i]] = array[i];
        }
    }
    if (invalid)
    { // 如果数组值中但凡有一个大于数组长度，数组就重置为 {0,1,2,....,7}
        for (unsigned i = 0; i < size; i++)
        {
            array[i] = i;
        }
    }
}
```







