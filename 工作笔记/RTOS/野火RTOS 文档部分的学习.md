## 野火RTOS 文档部分的学习

[toc]



### 实际运用( OpenCPU 的rtos )



#### EVENT

- osEventFlagsWait(init_task_flag, EVNT_INIT_FAIL, osFlagsWaitAll, 70000);

  当`option` 设置为`osFlagsWaitAll` 时，该函数只要有任意条件

- ASK:   osFlagsWaitAll  和 osFlagsWaitAny 一样要等到阻塞超时才能执行后续的操作，而且它们会清除标志位的信息( 经过测试，感觉这一块是移动移植的时候的问题 )



### 实际运用中产生的BUG

1. 事件标志位的问题

2. lock 持有锁的实时性，应当持有锁的线程放前面，

   否则会出现唤醒之后马上进入休眠的情况。

3. task 线程退出之后记得释放资源。

   否则所有线程都退出之后，系统跑飞，回调函数中不能引用os 的资源，但是回调函数会一直跑，导致创建的定时器删不掉

4. 事件标志位使用的时候，目前我用OpenCPU 的RTOS 只用了osFlagsNoClear 作为阻塞，拿取事件状态我还是使用的Get 方法。  如果直接用osEventFlagsWait(  ); 会出现意想不到的惊喜( 惊吓 )

   osEventFlagsGet(conet_task_flag)





### 野火 FreeROTS 教程

[野火官方文档地址](https://doc.embedfire.com/linux/stm32mp1/freertos/zh/latest/application/task_notification.html)

>**在FreeRTOS,系统调度， 最终也是产生PendSV中断，**在PendSV Handler里面实现任务的切换，所以还是可以归结为中断。 既然这样，FreeRTOS对临界段的保护最终还是回到对中断的开和关的控制。

- 关于sysTick 滴答定时器，和PendSV (可挂起的系统调用) 中断

  [SVC 和pendSV 参考地址](https://zhuanlan.zhihu.com/p/513133828)

  [有了Systick 中断为什么还要PendSV 中断](https://blog.csdn.net/wcc243588569/article/details/117792602)

  系统有256个中断，在 0-15 的区间是系统内核异常中断，剩下的是外部中断

  case1： 需要进行任务切换时，如果SysTick 中断在最高，让它执行中断服务函数就会导致RTOS 系统的时钟出问题。

  case2：如果PendSV 优先级最高，中断来时，如果系统正在打开uart 串口，打开到一半去执行 PendSV 的中断服务函数了（不会影响 rtos 时钟），执行完函数 系统内核的指针就指向下一个步骤 ，就跳过 uart 

  default：总结：设置Systick 中断优先级最高，PendSV 优先级最低，让Systick 中断服务函数触发PendSV 中断。这样又能保证PendSV 中断服务函数在系统初始化结束之后执行，又能保证Systick 一定会触发该中断且不影响系统时钟。

  - chatgpt 回复

    ![image-20230529095230019](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161741468.png)





#### 裸机系统

- 轮询系统

  先执行硬件初始化工作，然后程序放入 main 中的大循环中跑。

  优点: 代码简洁，逻辑清晰，MCU 只执行一件事。

  缺点: 外部事件到来时系统还在执行任务，切换不到外部事件的中断( 比如按键按下，手抬起来时还没执行完函数 )。

  ```c
  // 参考流程
  int main(void)
  {
      /* 硬件相关初始化 */
          HardWareInit();
      /* 无限循环 */
      for (;;) {
          /* 处理事情1 */
          DoSomething1();
          /* 处理事情2 */
          DoSomething2();
          /* 处理事情3 */
          DoSomething3();
          }
  }
  ```

- 前后台系统

  就是在轮询系统中加入了**中断**。其实就是单片机用到的内容。感觉 MCU 使用一部分性能一直在查找中断寄存器，如果发生中断，就在汇编底层执行中断处理函数。

  ```c
  // 模拟流程。
  int flag1 = 0;
  int flag2 = 0;
  int flag3 = 0;
  int main(void)
  {
      /* 硬件相关初始化 */
          HardWareInit();
      /* 无限循环 */
      for (;;)
      {
          if (flag1)
          {
              /* 处理事情1 */
              DoSomething1();
          }
          if (flag2)
          {
              /* 处理事情2 */
              DoSomething2();
          }
      }
  }
  void ISR1(void)
  {
      /* 置位标志位 */
          flag1 = 1;
      /* 如果事件处理时间很短，则在中断里面处理
      如果事件处理时间比较长，在回到前台处理 */
      //DoSomething1();
  }
  void ISR2(void)
  {
      /* 置位标志位 */
          flag2 = 1;
  
      /* 如果事件处理时间很短，则在中断里面处理
      如果事件处理时间比较长，在回到前台处理 */
      DoSomething2();
  }
  ```

#### 多任务系统

就是创建 RTOS 中的创建 task ，task 独立且包含优先级。多个任务同步执行( 其实是在单片机内核中不断切换，从而让用户觉得无感 )。

在每个 Task 中，task 不能返回，且无限循环。RTOS 比裸机系统占用更多一点 RAM 与 Flash。但是现在的 MCU完全足以抵消掉那点占用。

| 模型       | 事件响应 | 事件处理 | 特点                       |
| ---------- | -------- | -------- | -------------------------- |
| 轮询系统   | 主程序   | 主程序   | 轮询响应事件，轮询处理事件 |
| 前后台系统 | 中断     | 主程序   | 实时响应事件，轮询处理事件 |
| 多任务系统 | 中断     | 任务     | 实时响应事件，实时处理事件 |



##### - FreeRTOS 启动流程

> 何时创建 Task 就不细说了，这部分在 MN316 的 NB-iot 模块中已使用过，不再赘述

- osKernelInitialize 函数

  CMSIS-RTOS 使用 osKernelInitialize 函数来初始化 RTOS 内核状态，该函数定义如下

  ```c
   osStatus_t osKernelInitialize (void) {
   	osStatus_t stat;
       if (IS_IRQ()) {			// 判断当前运行环境是否在中断中
           stat = osErrorISR; 
       }
       else {  			
           // 不在中断中时，判断当前内核状态，内核未激活时调用本函数才生效，否则返回 osError
           if (KernelState == osKernelInactive) {	    
               // FreeRTOS 使用 5 种不同的动态内存分配策略，使用 HEAP_5 时才使用如下的  vPort...函数初始化
               #if defined(USE_FREERTOS_HEAP_5) && (HEAP_5_REGION_SETUP == 1)
                   vPortDefineHeapRegions (configHEAP_5_REGIONS);
               #endif
               KernelState = osKernelReady;   // 设置 RTOS 内核状态为就绪态
               stat = osOK;
           } else {
          	 stat = osError;
           }
       }
       return (stat);
   }
  ```

- **osThreadAttr_t** 结构体，作为 Task 线程的属性定义

  ```c
  /********* cmsis_os2.h *********/
  //描述线程属性的结构体
   typedef struct {
       const char                   *name;   ///< 线程名
       uint32_t                 attr_bits;   ///< 在FreeRTOS内核中没有使用
       void                      *cb_mem;    ///< 控制块内存
       uint32_t                   cb_size;   ///< 控制块大小
       void                   *stack_mem;    ///< 栈地址
       uint32_t                stack_size;   ///< 栈大小
       osPriority_t              priority;   ///< 最初的线程优先级 (默认值: osPriorityNormal)
       TZ_ModuleId_t            tz_module;   ///< 在FreeRTOS内核中没有使用
       uint32_t                  reserved;   ///< 保留 (必须为0)
   } osThreadAttr_t;
  ```

  **name** ：由于描述线程名。默认为NULL

  **attr_bits** ：用于描述线程的类型，在FreeRTOS内核中并没有使用

  **cb_mem**、**cb_size** ：用于静态创建线程时内存控制块需手动指定。

  **stack_mem** ：用于静态创建线程需手动指定。

  **stack_size** ：栈大小。默认栈大小为 128 字节

  **priority** ：描述线程优先级。默认任务优先级为 osPriorityNormal（24）。

  **tz_module** ：在FreeRTOS内核中没有使用

  **reserved** ：保留必须为0。



- **osThreadNew 函数**

  CMSIS-RTOS 使用 osThreadNew 函数用来创建线程

  **osThreadFunc_t func** 该参数表示线程的回调函数，将函数名传递给 osThreadNew 用来创建线程

  argument 为函数参数

  attr 作为创建的线程的属性而传入

  ```c
  /************** cmsis_os2.h ***************/
  // 创建新线程  传入参数为：线程函数、线程参数、线程控制块
  osThreadId_t osThreadNew(osThreadFunc_t func, void *argument, const osThreadAttr_t *attr)
  {
      const char *name;
      uint32_t stack;
      TaskHandle_t hTask;
      UBaseType_t prio;
      int32_t mem; // 标记创建线程方式(动态、静态)
  
      hTask = NULL;
  
      // 判断是否在中断中使用该函数以及线程函数是否为空
      if (!IS_IRQ() && (func != NULL))
      {
          stack = configMINIMAL_STACK_SIZE;     // 默认线程栈大小为128
          prio = (UBaseType_t)osPriorityNormal; // 默认线程优先级为24
  
          name = NULL; // 默认线程名为空
          mem = -1;
  
          // 线程控制块不为空
          if (attr != NULL)
          {
  
              // 设置线程名
              if (attr->name != NULL)
              {
                  name = attr->name;
              }
  
              // 设置线程优先级
              if (attr->priority != osPriorityNone)
              {
                  prio = (UBaseType_t)attr->priority;
              }
  
              // 线程优先级出错判断
              if ((prio < osPriorityIdle) || (prio > osPriorityISR) || ((attr->attr_bits & osThreadJoinable) == osThreadJoinable))
              {
                  return (NULL);
              }
  
              // 设置栈大小
              if (attr->stack_size > 0U)
              {
                  /* In FreeRTOS stack is not in bytes, but in sizeof(StackType_t) which is 4 on ARM ports.       */
                  /* Stack size should be therefore 4 byte aligned in order to avoid division caused side effects */
                  stack = attr->stack_size / sizeof(StackType_t);
              }
  
              // 判断线程控制块创建线程的方式
              if ((attr->cb_mem != NULL) && (attr->cb_size >= sizeof(StaticTask_t)) &&
                  (attr->stack_mem != NULL) && (attr->stack_size > 0U))
              {
                  mem = 1;
              }
              else
              {
                  if ((attr->cb_mem == NULL) && (attr->cb_size == 0U) && (attr->stack_mem == NULL))
                  {
                      mem = 0;
                  }
              }
          }
          else
          {
              mem = 0;
          }
          // 使用静态创建线程
          if (mem == 1)
          {
              hTask = xTaskCreateStatic((TaskFunction_t)func, name, stack, argument, prio, (StackType_t *)attr->stack_mem,
                                        (StaticTask_t *)attr->cb_mem);
          }
          // 使用动态创建线程
          else
          {
              if (mem == 0)
              {
                  if (xTaskCreate((TaskFunction_t)func, name, (uint16_t)stack, argument, prio, &hTask) != pdPASS)
                  {
                      hTask = NULL;
                  }
              }
          }
      }
      // 将使用FreeRTOS创建线程得到的线程id返回
      return ((osThreadId_t)hTask);
  }
  ```

- osKernelStart 函数

  作用: 用于启动任务调度器，不能在中断中使用。在创建完任务时，仅仅是把任务添加到系统中，并没有真正调度，空闲任务也没实现 ( 空闲任务是保证 RTOS 中一定有一个任务保持运行，空闲任务应设置 **最低优先级** )。**这些无需用户实现**，FreeRTOS 全部帮我们搞定。

  ```c
  osStatus_t osKernelStart(void)
  {
      osStatus_t stat;
      // 判读是否在中断中执行osKernelStart函数
      if (IS_IRQ())
      {
          stat = osErrorISR;
      }
      else
      {
          if (KernelState == osKernelReady)
          {
              /* Ensure SVC priority is at the reset value */
              SVC_Setup();
              /* Change state to enable IRQ masking check */
              KernelState = osKernelRunning; // 设置内核状态为运行态
              /* Start the kernel scheduler */
              vTaskStartScheduler(); // FreeRTOS启动任务调度器函数
              stat = osOK;
          }
          else
          {
              stat = osError;
          }
      }
      return (stat);
  }
  ```

##### - Cotex-M 架构，FreeRTOS 启动任务和任务切换

FreeRTOS 为了启动任务和任务切换使用了三个异常: SVC、PendSV、SysTick

1. SVC ( 系统服务调用 ) 用于任务启动，**有些操作系统不允许直接访问硬件**，而是通过提供一些系统服务函数，用户程序使用 SVC 发出对系统服务函数的呼叫请求，以这种方法调用它们来直接访问硬件，它就会产生 SVC 异常

2. SysTick 用于产生系统时钟，提供一个时间片，如果多个任务共享同一个优先级，则每次 SysTick 中断，下一个任务将获得一个时间片。

3. PendSV ( 可挂起系统调用 ) 用于任务切换，它可以被高优先级中断打断。一般设置PendSV 为低优先级

   ![image-20231128153326675](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404081002082.png)



##### - FreeRTOS 启动示例

该部分不做讲解，类似之前做的 MN316 模块的 RTOS。

```c
 #include "FreeRTOS.h"
 #include "task.h"
 #include "main.h"
 #include "cmsis_os.h"

 osThreadId_t LED1TaskHandle;
 const osThreadAttr_t LED1Task_attributes = {
     .name = "LED1Task",
     .priority = (osPriority_t) osPriorityNormal,
     .stack_size = 128 * 4
 };

 osThreadId_t LED2TaskHandle;
 const osThreadAttr_t LED2Task_attributes = {
     .name = "LED2Task",
     .priority = (osPriority_t) osPriorityLow,
     .stack_size = 128 * 4
 };

 void LED1_Task(void *argument);
 void LED2_Task(void *argument);

 void MX_FREERTOS_Init(void); /* (MISRA C 2004 rule 8.1) */

 void MX_FREERTOS_Init(void) {

     LED1TaskHandle = osThreadNew(LED1_Task, NULL, &LED1Task_attributes);
     LED2TaskHandle = osThreadNew(LED2_Task, NULL, &LED2Task_attributes);
 }

 void LED1_Task(void *argument)
 {
     for(;;)
     {
         HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);
         osDelay(200);
     }
 }

 void LED2_Task(void *argument)
 {
     for(;;)
     {
         HAL_GPIO_TogglePin(LED2_GPIO_Port, LED2_Pin);
         osDelay(500);
     }
 }
```



##### - 临界段的保护

> 临界段: 在执行的时候不能被中断的代码段。

最常出现的是 **对全局变量的操作**，全局变量谁都可以对他进行操作，但是同一时间只能一个 " 人 " 操作。



> 什么情况下临界段会被打断呢？ 

1. 系统调度: 

   其实也是中断，在 FreeRTOS 系统调度最终也是产生 PendSV 中断，在 PendSV Handler 里面实现任务的切换。

2. 外部中断



###### - Cortex-M 内核快速关中断指令

为了快速地开关中断，Cortex-M 内核专门设置了一条 CPS 指令。

```shell
CPSID I ;PRIMASK=1     ;关中断
CPSIE I ;PRIMASK=0     ;开中断
CPSID F ;FAULTMASK=1   ;关异常
CPSIE F ;FAULTMASK=0   ;开异常
```

上面列出 PRIMASK、FAULTMASK 这 2 个中断的屏蔽寄存器，其实还有一个 BASEPRI 中断屏蔽寄存器

| 名字      | 功能描述                                                     |
| --------- | ------------------------------------------------------------ |
| PRIMASK   | 这是个只有单一比特的寄存器。在它被置1后，就关掉所有可屏蔽的异常，只剩下NMI和硬FAULT可以响应。 它的缺省值是0，表示没有关中断。 |
| FAULTMASK | 这是个只有1 个位的寄存器。当它置1 时，只有NMI才能响应，所有其他的异常，甚至是硬FAULT，也通通闭嘴。 它的缺省值也是0，表示没有关异常。 |
| BASEPRI   | 这个寄存器最多有9位（由表达优先级的位数决定）。它定义了被屏蔽优先级的阈值。当它被设成 某个值后，所有优先级号大于等于此值的中断都被关（优先级号越大，优先级越低）。但若被设成0， 则不关闭任何中断，0也是缺省值。 |

· 在 FreeRTOS 中，对中断的开与关是通过操作 BASEPRI 寄存器来实现，大于等于 BASEPRI 的值的中断都会被屏蔽，小于它的则不会被屏蔽。

· 可以设置 BASEPRI 来为一些紧急的高优先级中断留一条后路

###### - 关中断

- 关中断函数

  (1). 不带返回值的关中断函数，不能嵌套，不能在中断里使用，

  (2). 带返回值的关中断函数，可以嵌套，可以在中断里使用，在更新完 BASEPRI 之后会将原来的值返回，返回的值作为形参传入开中断函数

  ```c
  /* 不带返回值的关中断函数，不能嵌套，不能在中断里面使用 */ //(1)
  #define portDISABLE_INTERRUPTS() vPortRaiseBASEPRI()
  
  void vPortRaiseBASEPRI( void )
  {
  uint32_t ulNewBASEPRI = configMAX_SYSCALL_INTERRUPT_PRIORITY; 
      //configMAXxxxxx是一个在 FreeRTOSConfig.h 中定义的宏，即要写入到 BASEPRI 寄存器的值，该宏默认为 191，
      //高四位有效，即 0xb0,也就是11，中断优先级值大于等于11 的中断都会被屏蔽
      __asm
      {
          msr basepri, ulNewBASEPRI 	//将configMAXxxxx 的值写入到 BASEPRI 寄存器，实现关中断
          dsb
          isb
      }
  }
  
  /* 带返回值的关中断函数，可以嵌套，可以在中断里面使用 */ //(2)
  #define portSET_INTERRUPT_MASK_FROM_ISR() ulPortRaiseBASEPRI()
  ulPortRaiseBASEPRI( void )
  {
      uint32_t ulReturn, ulNewBASEPRI = configMAX_SYSCALL_INTERRUPT_PRIORITY; //(2)-1
      __asm
      {
          mrs ulReturn, basepri     		// 保存 BASEPRI 的值，记录当前哪些中断被关闭
          msr basepri, ulNewBASEPRI		// 更新 BASEPRI 的值
          dsb
          isb
      }
      return ulReturn; 					// 返回原来的 BASEPRI 的值
  }
  ```

  

###### - 开中断

- 开中断函数

  (1). 开中断函数，形参为要更新的 BASEPRI 寄存器值，根据形参的不通过，分为中断保护与非保护版本。

  (2). 不带中断保护的开中断函数，直接将 BASEPRI 的值设置为 0，与 portDISABLE_INTERRUPTS() 成对使用

  (3). 带中断保护的开中断函数，将上一次中断时保存的 BASEPRI 值作为形参。

  ```c
  /* 不带中断保护的开中断函数 */
  #define portENABLE_INTERRUPTS() vPortSetBASEPRI( 0 )  //(2)
  
  /* 带中断保护的开中断函数 */
  #define portCLEAR_INTERRUPT_MASK_FROM_ISR(x) vPortSetBASEPRI(x)  //(3)
  
  void vPortSetBASEPRI( uint32_t ulBASEPRI )  //(1)
  {
      __asm
      {
          msr basepri, ulBASEPRI
      }
  }
  ```

  

###### - 进入/退出临界段

- 进入/退出临界段的宏

  可以看出，该宏定义和前文中的

  ```c
  #define taskENTER_CRITICAL()                portENTER_CRITICAL()
  #define taskENTER_CRITICAL_FROM_ISR() portSET_INTERRUPT_MASK_FROM_ISR()
  
  #define taskEXIT_CRITICAL()         portEXIT_CRITICAL()
  #define taskEXIT_CRITICAL_FROM_ISR( x ) portCLEAR_INTERRUPT_MASK_FROM_ISR( x )
  ```

- 进入和退出临界段

  文档请参考野火部分

  [进入/退出临界段](https://doc.embedfire.com/linux/stm32mp1/freertos/zh/latest/application/critical_protect.html#id11)



##### - 任务管理

每个任务在自己的环境中运行，在任何时刻，只有一个任务得到运行，RTOS 决定运行哪个任务，调度器会不断的启动、停止每一个任务，用户视角上看相当于所有任务同时执行。任务切出时，它的执行环境会被保存在该任务的栈空间中，当任务再次运行时，就能从栈中正确恢复上次的运行环境。

*是不是有点像 PendSV 中断？*

> 任务调度器

&emsp;&emsp;FreeRTOS 提供的任务调度器是基于优先级的全抢占式的调度: 系统中除了 **中断处理函数**、**调度器上锁部分的代码**  和 **禁止中断的代码** 是不可抢占外，系统的其它部分都是可以抢占的。

&emsp;&emsp;优先级范围 0~N, 优先级数字越大，优先级就越高。一般 0 为最低优先级，分配给空闲任务用，不建议用户使用该优先级。

- 任务状态迁移图

  ![image-20231129143337702](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404081002083.png)

  1. 创建任务 -> 就绪态 ( Ready ) : 任务创建完成后进入就绪态，表明任务已准备就绪，随时可以运行，只等待调度器进行调度。
  2. 就绪态 -> 运行态 ( Running ) : 发生任务切换时，就序列表中最高优先级的任务被执行，从而进入运行态。
  3. 运行态 -> 就绪态 : 有更高优先级任务创建或者恢复后，会发生任务调度，此时高优先级任务变为运行态，原先运行的任务变为就绪态，且在等高优先级执行完毕后再运行原来的任务。
  4. 运行态 -> 阻塞态 ( Blocked ) : 正在运行的任务发生阻塞 ( 挂起、延时、读信号量等待 ) 时，该任务会从就绪列表中删除，任务状态由运行态变成阻塞态，然后发生任务切换，在就绪列表中找最高优先级运行。
  5. 阻塞态 -> 就绪态 : 阻塞的任务被恢复后 ( 任务回复、延时时间超时、读信号量超时或者读到信号量等 )，任务就会加入到就绪列表中，如果该任务优先级高于正在运行的优先级，就会发生切换。
  6. 就绪态、阻塞态、运行态 -> 挂起态 ( Suspended ) : 被挂起的任务得不到 CPU 的使用权，也不会参与调度，除非它从挂起态中解除。
  7. 挂起态 -> 就绪态

  

- 常用任务函数讲解

  1. `osThreadGetId(void)` 返回当前线程 ID 函数

     ```c
     osThreadId_t osThreadGetId (void) {
     osThreadId_t id;
     
        id = (osThreadId_t)xTaskGetCurrentTaskHandle();
     
        return (id);
     }
     ```

  2. `osThreadSuspend(osThread_t thread_id)` 任务挂起函数

     ```c
     /* cmsis_os2.c */
     osStatus_t osThreadSuspend (osThreadId_t thread_id) {
     TaskHandle_t hTask = (TaskHandle_t)thread_id;
     osStatus_t stat;
     
     //并不能在中断中使用挂起函数
     if (IS_IRQ()) {
        stat = osErrorISR;
     }
     //判断任务句柄的有效性
     else if (hTask == NULL) {
        stat = osErrorParameter;
     }
     //使用vTaskSuspend挂起线程
     else {
        stat = osOK;
        vTaskSuspend (hTask);
     }
     
     return (stat);
     }
     ```

     

  3. `osThreadResume(osThread_t thread_id)` 任务恢复函数

     ```c
     /* cmsis_os2.c */
     osStatus_t osThreadResume (osThreadId_t thread_id) {
        TaskHandle_t hTask = (TaskHandle_t)thread_id;
        osStatus_t stat;
     
        //并不能在中断中使用恢复函数
        if (IS_IRQ()) {
           stat = osErrorISR;
        }
        //判断任务句柄的有效性
        else if (hTask == NULL) {
           stat = osErrorParameter;
        }
        //使用vTaskResume函数恢复线程
        else {
           stat = osOK;
           vTaskResume (hTask);
        }
        return (stat);
     }
     ```

     

  4. 任务删除函数

     > osThreadExit ()
     >
     > ```c
     > // 该函数用于终止当前运行线程
     > __NO_RETURN void osThreadExit (void) {
     > #ifndef USE_FreeRTOS_HEAP_1
     >    vTaskDelete (NULL);
     > #endif
     >    for (;;);
     > }
     > ```

     > osThreadTerminate ( )
     >
     > ```c
     > /********* cmsis_os2.c **********/
     > // 该函数可以用于删除别的线程  // thread_id ：线程ID，可由osThreadNew或者osThreadGetId得到。
     > osStatus_t osThreadTerminate (osThreadId_t thread_id) {
     >    TaskHandle_t hTask = (TaskHandle_t)thread_id;
     >    osStatus_t stat;
     > #ifndef USE_FreeRTOS_HEAP_1
     >    eTaskState tstate;
     > 
     >    //并不能在中断中使用线程结束函数
     >    if (IS_IRQ()) {
     >       stat = osErrorISR;
     >    }
     >    //判断任务句柄的有效性
     >    else if (hTask == NULL) {
     >       stat = osErrorParameter;
     >    }
     >    else {
     >       //获取线程状态
     >       tstate = eTaskGetState (hTask);
     > 
     >       if (tstate != eDeleted) {
     >          stat = osOK;
     >          vTaskDelete (hTask);  //使用vTaskDelete删除线程
     >       } else {
     >          stat = osErrorResource;
     >       }
     >    }
     > #else
     >    stat = osError;
     > #endif
     > 
     >    return (stat);
     > }
     > ```

  5. osDelay 延时函数

     用于阻塞延时，调用该函数时，任务将进入阻塞态。任务会让出 MCU 资源。形参为时钟节拍 ( 如果时钟节拍为 1 ms，那么形参为 100 时就为 100ms ) 

     ```c
     osStatus_t osDelay (uint32_t ticks) {
        osStatus_t stat;
        //不能在中断中使用延时函数
        if (IS_IRQ()) {
           stat = osErrorISR;
        }
        else {
           stat = osOK;
           if (ticks != 0U) {
              vTaskDelay(ticks); //调用vTaskDelay进行延时
           }
        }
        return (stat);
     }
     ```

     



1. 中断服务函数
2. 任务
3. 空闲任务
4. 任务执行时间





##### - 消息队列

##### - 信号量

##### - 互斥量

##### - 事件

##### - 软件定时器

##### - 任务通知

##### - 内存管理

##### - 中断管理

##### - CPU使用统计





















#### TODO: END

- 1. 

- 消息队列

  

- 中断 和信号量

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





- 互斥量

  &emsp;&emsp;互斥量是一种特殊的二值信号量，更多的是用于保护资源，作为锁的特点。互斥量不能在中断服务函数中使用，因为其特有的优先级继承机制只在任务起作用，在中断的上下文环境毫无意义。

  - 用于互锁的互斥量可以充当保护资源的令牌

  - 明确一个定义 ： **优先级翻转**

    ![image-20230530163727272](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161741469.png)

    正常情况下，设备资源被保护了， 当L 在执行任务时，H想要使用该资源，但是由于保护的问题，L不会释放资源，H就被阻塞， 当L还没执行完时，M任务想执行，但是M被L阻塞，**当L执行完，M就会先执行M ，然后再执行L。** 这就是优先级翻转。

    图1**(1)**：L任务正在使用某临界资源， H任务被唤醒，执行H任务。但L任务并未执行完毕，此时临界资源还未释放。

    图1**(2)**：这个时刻H任务也要对该临界资源进行访问，但 L任务还未释放资源，由于保护机制，H任务进入阻塞态， L任务得以继续运行，此时已经发生了优先级翻转现象。

    图1**(3)**：某个时刻M任务被唤醒，由于M任务的优先级高于L任务， M任务抢占了CPU的使用权，M任务开始运行， 此时L任务尚未执行完，临界资源还没被释放。

    图1**(4)**：M任务运行结束，归还CPU使用权，L任务继续运行。

    图1**(5)**：L任务运行结束，释放临界资源，H任务得以对资源进行访问，H任务开始运行。

    > &emsp;**&emsp;优先级继承**：  在H任务申请该资源的时候，由于申请不到资源会进入阻塞态， 那么系统就会把当前正在使用资源的L任务的优先级临时提高到与H任务优先级相同，此时M任务被唤醒了， 因为它的优先级比H任务低，所以无法打断L任务，因为此时L任务的优先级被临时提升到H，所以当L任务使用完该资源了， 进行释放，那么此时H任务优先级最高，将接着抢占CPU的使用权， H任务的阻塞时间仅仅是L任务的执行时间， 此时的优先级的危害降到了最低，看！这就是优先级继承的优势。
    >
    > ![image-20230530164342287](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161741470.png)
    >
    > 图2**(1)**：L任务正在使用某临界资源，L任务正在使用某临界资源， H任务被唤醒，执行H任务。 但L任务并未执行完毕，此时临界资源还未释放。
    >
    > 图2**(2)**：某一时刻H任务也要对该资源进行访问，由于保护机制，H任务进入阻塞态。 此时发生优先级继承，系统将L任务的优先级暂时提升到与H任务优先级相同，L任务继续执行。
    >
    > 图2**(3)**：在某一时刻M任务被唤醒，由于此时M任务的优先级暂时低于L任务，所以M任务仅在就绪态，而无法获得CPU使用权。
    >
    > 图2**(4)**：L任务运行完毕，H任务获得对资源的访问权，H任务从阻塞态变成运行态，此时L任务的优先级会变回原来的优先级。
    >
    > 图2**(5)**：当H任务运行完毕，M任务得到CPU使用权，开始执行。
    >
    > 图2**(6)**：系统正常运行，按照设定好的优先级运行。(将危害降低至最小)
    >
    > TIP：FreeRTOS的优先级继承机制不能解决优先级反转，只能将这种情况的影响降低到最小， 硬实时系统在一开始设计时就要避免优先级反转发生。

    

    我的理解就是，互斥量是一个含有`优先级继承`  的二值信号量。

    > &emsp;&emsp;用互斥量处理不同任务对临界资源的同步访问时，任务想要获得互斥量才能进行资源访问， 如果一旦有任务成功获得了互斥量，则互斥量立即变为闭锁状态，此时其他任务会因为获取不到互斥量而不能访问这个资源， 任务会根据用户自定义的等待时间进行等待，直到互斥量被持有的任务释放后，其他任务才能获取互斥量从而得以访问该临界资源， 此时互斥量再次上锁，如此一来就可以确保每个时刻只有一个任务正在访问这个临界资源，保证了临界资源操作的安全性。
    >
    > ![image-20230530172146276](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306161741471.png)

    

    

    

- 事件

  **无数据传输，但可以用于 任务与任务间 、 中断与任务间的同步。**

  对于需要判断某硬件（或其它）处于什么状态，可以使用 全局变量定义一个标志位( 就像MN316 项目中的判断设备是否数据成功上报 或者 设备初始化联网是否成功 )，那么问题就来了？

  - 如何对全局变量进行保护呢，如何处理多任务同时对它进行访问？
  - 如何让内核对事件进行有效管理呢？使用全局变量的话，就需要在任务中轮询查看事件是否发送，这简直就是在浪费CPU资源啊， 还有等待超时机制，使用全局变量的话需要用户自己去实现。

  > &emsp;&emsp;在某些场合，可能需要多个时间发生了才能进行下一步操作，比如一些危险机器的启动，需要检查各项指标， 当指标不达标的时候，无法启动，但是检查各个指标的时候，不能一下子检测完毕啊，所以，需要事件来做统一的等待， 当所有的事件都完成了，那么机器才允许启动，这只是事件的其中一个应用。









































































































