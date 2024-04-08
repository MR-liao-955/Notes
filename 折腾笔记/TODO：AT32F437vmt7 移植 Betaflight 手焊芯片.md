### AT32F437vmt7 移植 Betaflight 飞控固件 (一)

[toc]

> 前言

1. 之前一直想自制 F4 飞控，但是由于各种外部因素，一直没去做。后来看到二木大佬发帖，使用 AT32F437 移植 Betaflight , 因此产生了尝试移植的想法。正好工作中的开发部分忙完，是时候开始之前的学习移植计划了。
2. 由于根据 Bom 表去采购元器件时，当时以为价格不算贵，结果各类元件算下来，成本快接近买成品了。。3 套元器件，以及各类配件，算下来接近 300￥，何况移植的成功性无法保证，相当于交学费了。。
3. 目前能力不足以支撑移植 Betaflight 飞控固件，目前计划先焊接好，烧录固件并调通硬件试飞，**根据 `二木山人` 和 `辉光管` 以及其他大佬们的成品来进行验证。**后续再更改气压计、陀螺仪、时钟频率、以及其它外设，以提升自身的技术水平。
4. 目前 Betaflight 的 AT32 分支代码仓库在 https://github.com/flightng/atbetaflight 地址，瞻仰大佬的脚步，以及各类开源精神，希望总有一天实力水平上来之后，我也能参与到开源项目之中。



#### Setting new goal

- PCB 打板 + 购买硬件。
- 熬夜焊接 + 连通性测试。
- 烧录固件，以及连接 Betaflight 地面站。
- 尝试编译 AT32 Betaflight 源码，并理解它的结构。
- 粗读代码 (理解框架)。
- 细读代码 ( 理解外设驱动代码 )。( 暂定目标理解驱动各类芯片，暂时不考虑飞控的飞行算法，以及 PID 算法之类 )
- 尝试修改驱动。



#### 硬件购买+焊接

> 提个意见，焊接尽量自己动手，锻炼动手能力

#### 固件烧写+硬件的验证+连通 BF 地面站

固件烧写采用的 CMSIS-Daplink 的四个引脚烧录。但是存在问题点：

1. 按住 BOOT 之后飞控上电无法进入 DFU 模式。同时雅特力烧写工具无法识别 AT32 芯片
2. 根据问题点1，在淘宝购买的成品 AT32 飞控就无此问题。
3. 烧录完成固件之后，发现无法识别气压计，气压计芯片没焊好。

固件烧录步骤



#### Watching out bumps ( 踩过的坑 )

- The biggest bump is ' don't let other person do it, who can't shut up his mouth '



#### 代码阅读（仅仅是阅读，对阅读代码部分做笔记）

&emsp;&emsp;Betaflight 移植部分的单元测试代码由 C++ 实现 , 对于 C++ 而言，基本语法和面向对象目前不成问题，但是它的新特性有点多，和之前学的还是存在一点差异。特此记录。

###### main() 函数

> 在 Betaflight 源码中，main 函数有许多个，暂时先拿最像main 的函数来开始阅读 `路径: src\main\main.c`

```c
void run(void);

int main(void)
{
    init();
    run();
    return 0;
}

void FAST_CODE run(void)
{
    while (true) {
        scheduler();
#ifdef SIMULATOR_BUILD
        delayMicroseconds_real(50); // max rate 20kHz
#endif
    }
}

```



###### 传感器设计思路

[参考文档](https://blog.csdn.net/lida2003/article/details/127179263)



###### scheduler( ) 函数阅读以及学习

[参考大神博客](https://blog.csdn.net/lida2003/article/details/130604471)

[代码&&调度器框架讲解](https://blog.csdn.net/lida2003/article/details/124876862)

Betaflight 是在 bare-metal ( 裸机 ) loop 基础上增加对业务过程耗时统计+基于时间优先级调度一个算法逻辑 scheduler。

- 任务时序
- 调度器切换任务
- 调度任务的时间更新

调度框架

```bash
scheduler
 ├──> <陀螺仪使能>
 │   ├──> <修正调度异常剩余时间> //其他耗时任务导致任务调度时间不足，比如USB任务
 │   ├──> <修正调度触发阈值时间> //调度代码耗散时间，控制接近边界 schedLoopStartMinCycles，调整粒度schedLoopStartDeltaUpCycles
 │   └──> <触发调度> // schedLoopRemainingCycles < schedLoopStartCycles
 │       ├──> <修正调度触发阈值时间> //控制接近边界 schedLoopStartMinCycles，调整粒度schedLoopStartDeltaDownCycles
 │       ├──> [精准对齐调度触发时间]
 │       ├──> schedulerExecuteTask(gyroTask, currentTimeUs); //执行陀螺仪传感任务
 │       ├──> schedulerExecuteTask(getTask(TASK_FILTER), currentTimeUs); //按比例执行过滤任务
 │       ├──> schedulerExecuteTask(getTask(TASK_PID), currentTimeUs); //按比例执行PID任务
 │       ├──> rxFrameCheck //检查接收机数据
 │       ├──> [故障保护模式检查]
 │       └──> <使用陀螺仪外部中断模式锁定调度器时间>
 ├──> [更新调度器剩余时间，schedLoopRemainingCycles]
 └──> <陀螺仪不使能 或者 调度剩余时间大于非实时任务检查时间>
     ├──> [更新所有非实时任务的任务动态优先级]
     │   ├──> <事件驱动任务>
	 │   │   ├──> <执行任务checkFunc，并更新任务动态优先级>
	 │   │   └──> <已执行checkFunc，未调度执行任务，继续提升任务动态优先级>
	 │   ├──> <时间驱动任务>
	 │   │   └──> [更新基于上次调度与当前时间间隔的任务动态优先级]
	 │   └──> <挑选最高优先级任务，并计算剩余执行时间>  //TASK_SERIAL只要优先级够，就直接执行不进行阻塞
     ├──> [更新checkCycles，所有非实时任务最优任务选择本次计算时间]
	 └──> <有高优先级任务选中>
         ├──> [更新taskRequiredTimeCycles和schedLoopRemainingCycles数据]
         ├──> <陀螺仪不使能 或者  (taskRequiredTimeCycles < schedLoopRemainingCycles)>
	     │   ├──> schedulerExecuteTask(selectedTask, currentTimeUs); //执行选中高优先级任务
		 │   ├──> [TASK_OSD/TASK_RX, skippedRxAttempts/skippedOSDAttempts清零]
		 │   └──> [修正taskGuardCycles，以便下次更精确的预估执行时间]
		 └──> <TASK_OSD/TASK_RX 或者 (taskAgePeriods > TASK_AGE_EXPEDITE_COUNT)>
	         └──> [能找TASK_AGE_EXPEDITE_SCALE比率缩短任务预计时间]

```



- 调度器执行 `schedulerExecuteTask(getTask(TASK_FILTER), currentTimeUs);` 函数。

  该函数为 task 正式启动的函数，根据当前函数 scheduler 的局部变量 `task_t *selectedTask = NULL;` 来确定执行什么函数，

  当然，在schedulerExecuteTask( ) 函数执行之前会有优先级判断，来确定这一轮执行哪个 task





- 如果调度器在执行任务的期间，来了一个硬件中断，哪么它是如何切换到中断中执行的呢？







###### 驱动 Led 函数

- 在 Betaflight 中，所有的外设驱动的执行函数都是在一个结构体数组中的回调函数接口中。

  下方 Betaflight 部分代码节选

  ```c
  // 宏定义
  #define DEFINE_TASK(taskNameParam, subTaskNameParam, checkFuncParam, taskFuncParam, desiredPeriodParam, staticPriorityParam) \
      {        								 \
          .taskName = taskNameParam,   		 \
          .subTaskName = subTaskNameParam,     \
          .checkFunc = checkFuncParam,         \
          .taskFunc = taskFuncParam,    		 \
          .desiredPeriodUs = desiredPeriodParam,  \
          .staticPriority = staticPriorityParam   \
      }
  
  // Task info in .bss (unitialised data)
  task_t tasks[TASK_COUNT];
  
  // Task ID data in .data (initialised data)
  task_attribute_t task_attributes[TASK_COUNT] = {
      [TASK_SYSTEM] = DEFINE_TASK("SYSTEM", "LOAD", NULL, taskSystemLoad, TASK_PERIOD_HZ(10), TASK_PRIORITY_MEDIUM_HIGH),
      [TASK_MAIN] = DEFINE_TASK("SYSTEM", "UPDATE", NULL, taskMain, TASK_PERIOD_HZ(1000), TASK_PRIORITY_MEDIUM_HIGH),
      [TASK_SERIAL] = DEFINE_TASK("SERIAL", NULL, NULL, taskHandleSerial, TASK_PERIOD_HZ(100), TASK_PRIORITY_LOW), // 100 Hz should be enough to flush up to 115 bytes @ 115200 baud
      [TASK_BATTERY_ALERTS] = DEFINE_TASK("BATTERY_ALERTS", NULL, NULL, taskBatteryAlerts, TASK_PERIOD_HZ(5), TASK_PRIORITY_MEDIUM),
      [TASK_BATTERY_VOLTAGE] = DEFINE_TASK("BATTERY_VOLTAGE", NULL, NULL, batteryUpdateVoltage, TASK_PERIOD_HZ(SLOW_VOLTAGE_TASK_FREQ_HZ), TASK_PRIORITY_MEDIUM), // Freq may be updated in tasksInit
      [TASK_BATTERY_CURRENT] = DEFINE_TASK("BATTERY_CURRENT", NULL, NULL, batteryUpdateCurrentMeter, TASK_PERIOD_HZ(50), TASK_PRIORITY_MEDIUM),
      /* 后续内容就不完全复制了，内容太多 */
  }
  ```

  - 使用枚举来存放各类 task 数组下标

    ![image-20240103145339015](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954905.png)

  - 创建结构体数组后使用枚举来进行结构体的初始化

    ```c
    /*  示例如下  */
    test_t test[10] = {
        [ENUM_1] = { }, // ENUM_1 为结构体数组的索引， { } 中才是结构体初始化所实现的赋值
        [ENUM_2] = { },
        [ENUM_3] = { }
    }
    ```

    ![image-20240103145555107](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954906.png)





###### Rx 接收机函数

- 



###### bmp280 气压计的驱动

- 草稿

  1. BMP280 是气压计，它的 `static void taskUpdateBaro(timeUs_t currentTimeUs)`为 task 中的启动函数。实现了上述结构体的接口

     > task.c

     ![image-20240103155422479](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954907.png)

     

  2. baroUpdate 才是具体函数的实现，因此可以认为 taskUpdateBaro 属于中间层 ( 和调度器框架接口解耦，和具体 baro 选型的传感器没有硬关联 )。

     > main/sensors/arometer.c

     ![image-20240103155756690](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954908.png)

     

  3. 注意看上方函数执行接口是调用的结构体的函数指针来执行

     ![image-20240103155959855](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954909.png)

     

  4. 此时是无法直到该气压计函数在在哪里调用的，因为 ( 不同的种类飞控使用的传感器不一致，因此会有一个 target.h 来定义你使用的哪个飞控。 ) 同时，makefile 文件会决定你编译哪个文件，这里以 bmp280 为例。

     > main/drivers/barometer/barometer_bmp280.c
     >
     > 其中的 bmp280Detect( baroDev_t *baro ) 函数会为结构体 baro 选定具体实现的函数。
  
     ![image-20240103163056652](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954910.png)
  
     
  
  5. bmp280Detect() 函数被 baroDetect() 调用。baroDetect( ) 又被 sensorsAutodetect() 函数调用。其中 sensorsAutodetect() 函数直接调用者为 init( ) 。。。。
  
     > 这下关系就明了了吧。函数的层级关系如次。后续做一个 思考。为何设计者要如此设计？

- 草稿 END

  

- 思想: init 和函数执行是分开的，如果 target.h 头文件使用了 `USE_BARO` 这个宏定义之后，baro 的部分代码才会被 init 执行，调用的时候，是调度器遍历 task 然后再执行这部分的函数实现。

  但是，任务调度器是和飞控设备执行代码耦合的，和普通的 RTOS 不一样。

- 













###### bmi270 与 bmi088 (陀螺仪&加速度计)

[BMI088 官网](https://www.bosch-sensortec.com/products/motion-sensors/imus/bmi088/)

- bmi270 驱动函数位于 './src/main/drivers/accgyroaccgyro_spi_bmi270_init.c && accgyro_spi_bmi270.c' 文件中。



- 注意：Betaflight 中各类总线 ( BUS ) 是 Betaflight 实现的驱动，而不同的 MCU 芯片驱动要适配到 Betaflight。



- 关于传感器驱动，部分初始化代码是传感器官网提供的驱动函数



> 驱动接口联合体部分。







###### 硬件访问层



##### 滤波模块(代码部分) (放弃)

- 单独提出该模块，因为这部分算法是在调度器中着重强调的。而且滤波方法可以用在别的方面。因此着重了解一下



- 滤波部分也类似于函数接口一样，写了结构体接口



> 算法

- 原理

  

- 公式

- 函数接口形式 ( 结构体定义函数接口，然后再外部强制转换进行实现 )

  ![image-20240104114402698](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954911.png)

  函数接口的具体实现 ( 将自定义)

  ![image-20240104114429688](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954912.png)

  ![image-20240104114449989](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954913.png)

- 疑问点: 不同的滤波函数实现方法，调用位置未找到。

  回答: 在 gyro_init.c 中 `gyroInitLowpassFilterLpf( )` 函数中存在函数接口的指定实现 

  ![image-20240105093210794](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080954914.png)

  

> slewFilter 回转滤波器

- 作用: 当 input 参数超过 threshold 时，过滤掉变化量不大于 slewLimit 的 input。

- 看代码的心得：( 把这个结构体想象成一维数轴，正负区间为 threshold ) 。
  1. filter->state 是上次的有效数据，而且 filter 是全局变量，存放在全局区，程序运行时不会被释放。
  2. input 传入的传感器数据 ( 传入参数名: gyroADCf ) ，如果上次 state 在 threshold 的正负区间内，都认为此次 input 有效。
  3. 如果 上次有效的 input 在区间外，他么这次会比较上回的 回旋限制的差值

- 代码实现

  ```c
  FAST_CODE float slewFilterApply(slewFilter_t *filter, float input)
  {
      if (filter->state >= filter->threshold)
      { // 状态值大于等于临界值时 (理解 state 为上次输入的有效值)
          if (input >= filter->state - filter->slewLimit)
          // 且输入值大于等于 状态值和回旋限制的差值，就修改状态值为输入值 //说人话就是输入值大于变化值
          {
              filter->state = input;
          }
      }
      else if (filter->state <= -filter->threshold) // 数轴上来看，就是负数部分。。
      {                                             // 状态值小于等于负的临界值，
          // 如何确认input 为正负值？因为不考虑 Input 为正值的情况，如果为正值，会被上面判断拿走，如果为中间值，他就会被下面拿走
          if (input <= filter->state + filter->slewLimit)
          { // 如果输入值 <= 状态值+临界值
              filter->state = input;
          }
      }
      else
      {
          filter->state = input; // 上次输入有效值在 threshold(临界值) 内，无论这次值有多变态，都会认为这次输入有效
      }
      return filter->state; // filter 是否会留着下一次使用？应该会，因为形参传入的是一个全局变量的指针
  }
  
  ```



> simpleLowpassFilter 简单低通滤波器

- 原理



- 公式
- 







##### 不同的语法

- 结构体定义时指针函数的写法

  注意下方代码的 ` bool (*checkFunc)(timeUs_t currentTimeUs, timeDelta_t currentDeltaTimeUs); ` 写法, 是不是很像之前看到 C++ 中的 Dshot 协议中的写法？

  ```c
  typedef struct {
      // Configuration
      const char * taskName;
      const char * subTaskName;
      bool (*checkFunc)(timeUs_t currentTimeUs, timeDelta_t currentDeltaTimeUs);
      void (*taskFunc)(timeUs_t currentTimeUs);
      timeDelta_t desiredPeriodUs;        // target period of execution
      const int8_t staticPriority;        // dynamicPriority grows in steps of this size
  } task_attribute_t;
  ```

  

- Betaflgiht 单元测试代码 C++ 编写 ( .cc 文件 ) 

  1. `motor_output_unittest.cc` 电机输出测试文件，这里关注 ``TEST( )` 这个宏定义。![image-20231219110551335](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202401021758793.png)

  2. `gtets.h` 谷歌测试框架头文件

     ![image-20231219111103770](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202401021758795.png)

  3. 谷歌单元测试参考博客

     https://www.51cto.com/article/285290.html 

     https://gjgjh.github.io/gtest.html#/%E4%BD%BF%E7%94%A8gtest

- 



##### 宏定义 \#define PG_DECLARE(_type, _name) 作为接口

- 代码出处

  ```c
  // Declare system config
  #define PG_DECLARE(_type, _name)                                        \
      extern _type _name ## _System;                                      \
      extern _type _name ## _Copy;                                        \
      static inline const _type* _name(void) { return &_name ## _System; }\   // 内联函数
      static inline _type* _name ## Mutable(void) { return &_name ## _System; }\ //
      struct _dummy                                                       \
      /**/
  ```

- [C语言中“##”的独特用法](https://blog.51cto.com/u_15091053/2616782)

- 初步理解：外联的 '_name_System' 的函数，获取它的引用地址，但是并未找到 _name_System 这部分的定义。(理论上说应该有外部 gyroConfig_System 函数)

- 

  



##### 之前看的 ESP32 部分 Dshot 协议驱动电机代码记录

- 这部分 Dshot 代码和 betaflight 的传感器代码有异曲同工之妙 ( 指的是结构体定义函数指针，并在外部实现它)



##### Betaflight 正式代码阅读



##### temp

- 构造函数和析构的顺序

  ```bash
  # 如果通过父类创建子类对象，构造时先调用父类构造函数，在调用子类构造。
  # 析构时先析构子类，然后再析构父类对象。
  ```

  ```bash
  
  
  
  ```

  

  







