### SIMCOM-Linux 开发学习记录

[toc]

&emsp;&emsp;开发板采用的是 A7608E 芯片，

#### Linux 编译环境搭建

&emsp;&emsp;由于该 Linux 部分环境交叉编译工具链厂商都做好脚本，因此，只需要在 Linux 下执行 `source ./simcom_crosscompile/simcom-crosscompile-env-init` 即可，且该工具链并不会影响到 Linux 系统的原有交叉编译环境。

- 解压 SDK 

  `tar -xvf A59C4B01V05A76xxMX_Linux_SDK_240131_open_sdk`

  并进入文件夹

- 编译所有

  make

- 单独编译内核

  1. 执行 `make kernel_menuconfig` 进行内核的配置
  2. 单独编译内核模块：`make kernel_module`

- 单独编译文件系统 `make rootfs`

- 单独编译 boot 引导程序 `make aboot`

- 清理编译生成文件 `make clean`

  注意: 并不会清理掉 kernel 部分的 `.config` 配置。

  进入 simcom_kernel 执行 `make mrproper` 则可以清理掉内核配置文件。
  
  Tip : 可以在 `A59C4B01V05A76xxMX_Linux_SDK_240131_open_sdk/simcom_kernel/arch/arm/configs` 找到各个厂商的参考内核配置，在kernel 的根目录可以执行 `make xxx_defconfig` 来选取参考内核配置 ( 不是arm32 架构的也可以在别的架构中配置内核，或者自己使用 make menuconfig 来自己设置 )



> 如果需要编译新的程序，使用 SIMCOM 交叉编译工具链时，需要修改 Makefile



#### Windows驱动程序安装

- 解压 `SIMCom_USB_Drivers`
- 插入开发板并找到未安装驱动的设备，并手动安装。

- `CATStudio_V3_0_9_84` 该工具是日志抓取工具。



模块自身信息

GPIO外设之类，I2C、SPI、ADC、按键、串口

开机->上网流程，入网之类。

参考 B-iot 模块当时的设备信息查询。



#### 固件烧录

&emsp;&emsp;使用软件 `A76XX_A79XX_MADL V1.21` 该软件通过 BLF 文件进行烧录，只烧录底层，上层业务代码通过 Linux 下 ADB 进行测试。

注意：烧录的时候选择 elf 文件请选择 `1803_Nand.blf`，而不是 `1803_Nand_Facotry_only_Erase_All.blf`。

烧录工厂模式的程序进去会出现问题。

烧录自己编译的内核、bootloader 和文件系统之类的，直接替换如下文件到烧录工程上去

![image-20240306161632441](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720135.png)

<img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720137.png" alt="image-20240306164434794" style="zoom: 67%;" />



##### - 误烧录 Factory 的 blf 文件时可能会导致无法进入 adb 调试

- 烧录好程序之后设备要重启。

- 如果发现无法进入 adb 调试时，执行下方操作

  ```bash
  #用串口工具的AT口(如果有moden口就用moden口)发送一下这条AT命令：
  AT*PROD?
  #如果回复的是1，那么就再发送一下：
  AT*PROD=0
  #可能暂时无法将PROD改为0，再次查询的时候可能还是存在异常，执行下方操作。
  at+cusbadb=1
  at+creset
  #####随后试着将串口关闭后再试着进入 adb shell #########
  ```



#### 编译环境部分学习

##### - 修改编译目录

- 目的: 将编译的用户代码文件和demo文件分开

- 方法

  1. 创建新的变量 example 中源文件的路径

     ```makefile
     MAIN_DIR = .
     TARGET_OBJ_DIR = $(MAIN_DIR)/obj
     
     TESTC_DIR = $(MAIN_DIR)/src
     ###添加 example 目录变量
     EXAMPLE_DIR = $(MAIN_DIR)/example/src
     ```

  2. 修改 example 头文件路径

     ```makefile
     ALL_PATHS += ./inc
     ## 添加 example 头文件路径,并移动example头文件刀 ./example/inc 中
     ALL_PATHS += ./example/inc
     ALL_PATHS += $(LIB_PATH)/usr/include
     ALL_PATHS += $(LIB_PATH)/usr/include/sdk_api
     ALL_PATHS += $(LIB_PATH)/usr/include/sdk_api/common
     ALL_PATHS += $(LIB_PATH)/usr/include/glib-2.0
     ```

  3. 设置编译方式---- EXAMPLE_DIR => $(TARGET_OBJ_DIR)/%.o

     ```makefile
     #新的 example 路径，依旧在 TARGET_OBJ_DIR 中生成 .o 文件
     $(TARGET_OBJ_DIR)/%.o:$(EXAMPLE_DIR)/%.c
     	echo ---------------------------------------------------------
     	#echo $(ALL_PATHS)
     	#echo $(ALL_INCLUDES)
     	echo Build OBJECT $(@) from SOURCE $<
     	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
     	echo ---------------------------------------------------------
     
     $(TARGET_OBJ_DIR)/%.o:$(TESTC_DIR)/%.c
     	echo ---------------------------------------------------------
     	#echo $(ALL_PATHS)
     	#echo $(ALL_INCLUDES)
     	echo Build OBJECT $(@) from SOURCE $<
     	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
     	echo ---------------------------------------------------------
     
     $(TARGET_OBJ_DIR)/%.o:$(TESTC_DIR)/%.cpp
     	#echo $(ALL_PATHS)
     	#echo $(ALL_INCLUDES)
     	echo Build OBJECT $(@) from SOURCE $<
     	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
     	echo ---------------------------------------------------------
     ```

  4. TIP: 由于我在 for 循环中定义了变量。因此编译条件要设置为 c99/gnu99 但是 c99 编译器会报错

     问题点如上方

     

##### - 编译时遇到的问题

- makefile 中注释如下

  ```makefile
  CFLAGS = -c -O2
  
  ROOTFS_DIR ?=../../simcom_rootfs
  
  LIBS = -lubox -lubus -lmtel -lprop2uci -luci -llog
  
  MAIN_DIR = .    #MAIN_DIR = . 表示当前的 main 所在目录也就是 ./simcom_apps/helloworld/ 这个目录
  
  TARGET_BIN_DIR = $(MAIN_DIR)/bin	#生成二进制文件
  TARGET_OBJ_DIR = $(MAIN_DIR)/obj	#obj中间文件
  
  BIN_NAME  = helloworld
  
  TEST_DIR = $(MAIN_DIR)
  TESTC_DIR = $(MAIN_DIR)
  
  SRC_DIR = $(TEST_DIR)
  
  INCLUDE_PREFIX = -I
  
  LINK_PREFIX = -L
  
  ALL_INCLUDES_PATHS = $(SRC_DIR)
  ALL_INCLUDES_PATHS += $(LIB_PATH)/usr/include
  
  ALL_LIB_PATHS = $(LIB_PATH)/usr/lib
  
  ALL_INCLUDES = $(addprefix $(INCLUDE_PREFIX), $(ALL_INCLUDES_PATHS))
  
  ALL_LINKS = $(addprefix $(LINK_PREFIX), $(ALL_LIB_PATHS))
  
  OBJ_CMD = -o
  
  LD_CMD = -o
  
  TEST_OBJS = $(TARGET_OBJ_DIR)/helloworld.o \
  			$(TARGET_OBJ_DIR)/LedControl.o \
  			$(TARGET_OBJ_DIR)/GpioControl.o \
  			$(TARGET_OBJ_DIR)/gpio.o
  
  BIN_OBJS = $(TEST_OBJS)
  
  $(TARGET_OBJ_DIR)/%.o:$(TESTC_DIR)/%.c
  	echo ---------------------------------------------------------
  	#echo $(ALL_PATHS)
  	#echo $(ALL_INCLUDES)
  	echo Build OBJECT $(@) from SOURCE $<
  	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
  	echo ---------------------------------------------------------
  
  $(TARGET_OBJ_DIR)/%.o:$(TESTC_DIR)/%.cpp
  	#echo $(ALL_PATHS)
  	#echo $(ALL_INCLUDES)
  	echo Build OBJECT $(@) from SOURCE $<
  	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
  	echo ---------------------------------------------------------
  
  .PHONY: all clean
  all:prep bin  install
  
  prep:
  	@if test ! -d $(TARGET_BIN_DIR); then mkdir $(TARGET_BIN_DIR); fi
  	@if test ! -d $(TARGET_OBJ_DIR); then mkdir $(TARGET_OBJ_DIR); fi
  
  bin:$(BIN_OBJS)
  	@echo Create bin file $(BIN_NAME)
  	$(CC) $(ALL_LINKS) $(BUILD_LDFLAGS) $(STD_LIB) -o $(TARGET_BIN_DIR)/$(BIN_NAME) $^ $(LIBS)
  	@echo ---------------------------------------------------------
  
  install:
  	cp $(TARGET_BIN_DIR)/$(BIN_NAME)  $(ROOTFS_DIR)/usr/bin
  	cp ./files/helloworld.init  $(ROOTFS_DIR)/etc/init.d/helloworld
  
  clean:
  	@rm -fr $(TARGET_OBJ_DIR) $(TARGET_BIN_DIR)
  
  ```

- 编译报错在 for 循环中定义局部变量

  ![image-20240311100343437](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720138.png)

  ```c
         for (uint8_t i = 0; i < gpio_max; i++)  // 在循环中进行初始化，需要指定编译器版本为C99
          {
              gpio_set_value(12, 0);
          }
  
  /**
  	解决办法：makefile 中修改
  	# CFLAGS = -c -O2
      # 需要添加编译选项
      CFLAGS = -c -O2 -std=c99
  */
  ```


##### - 设备树所在的位置，且名称为 a7600sl.dts

- 查找 ./Makefile

![image-20240311152630934](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720139.png)

![image-20240311152716313](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720140.png)

- 在内核的 boot 驱动部分能找到设备树的源文件

![image-20240311152552466](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720141.png)







#### makefile 部分

##### - 参考 makefile

- 了解makefile基本写法，以及如何实现文件的链接的。便于后续添加库文件之类

  ```makefile
  ## project.mk 文件中只放了 2 个宏定义,定义了芯片类型
  include ./../../project.mk
  CFLAGS = -c -O2 -std=gnu99
  # CFLAGS = -c -O2
  
  MAIN_DIR = .
  
  TARGET_BIN_DIR = $(MAIN_DIR)/bin
  TARGET_OBJ_DIR = $(MAIN_DIR)/obj
  
  BIN_NAME  = demo_app
  
  TESTC_DIR = $(MAIN_DIR)/src
  ###example目录
  EXAMPLE_DIR = $(MAIN_DIR)/example/src
  
  INCLUDE_PREFIX = -I
  
  LINK_PREFIX = -L
  
  ALL_PATHS += ./inc
  ## 添加 example 头文件路径
  ALL_PATHS += ./example/inc
  # 由于下方 ALL_INCLUDES 已经单独添加了路径，因此这里就不添加了
  # ALL_PATHS += ./src/cJSON
  ALL_PATHS += $(LIB_PATH)/usr/include
  ALL_PATHS += $(LIB_PATH)/usr/include/sdk_api
  ALL_PATHS += $(LIB_PATH)/usr/include/sdk_api/common
  ALL_PATHS += $(LIB_PATH)/usr/include/glib-2.0
  
  ifeq ($(BLUETOOTH_FUNC),1)
  	ALL_PATHS += $(LIB_PATH)/usr/include/bluez-5.30
  	ALL_PATHS += ./bluetooth/inc
  endif
  
  # 将cJSON 也添加进 INCLUDE中
  # ALL_INCLUDES = $(addprefix $(INCLUDE_PREFIX), $(ALL_PATHS))
  ALL_INCLUDES = $(addprefix $(INCLUDE_PREFIX), $(ALL_PATHS), $(MAIN_DIR)/src/cJSON)
  
  LIB_DIRS += $(LIB_PATH)/usr/lib
  
  ifeq ($(BLUETOOTH_FUNC),1)
  	LIB_DIRS += ./bluetooth/lib
  endif
  
  ALL_LINKS += $(addprefix $(LINK_PREFIX), $(LIB_DIRS))
  
  OBJ_CMD = -o
  
  LD_CMD = -o
  
  TEST_OBJS = $(TARGET_OBJ_DIR)/GpioControl.o \
  			$(TARGET_OBJ_DIR)/mqtt_s.o \
  			$(TARGET_OBJ_DIR)/CMD_mqtt.o \
  			$(TARGET_OBJ_DIR)/CMD_uart.o \
  			$(TARGET_OBJ_DIR)/util.o \
  			$(TARGET_OBJ_DIR)/cJSON.o \
  			$(TARGET_OBJ_DIR)/main.o
  
  ifeq ($(BLUETOOTH_FUNC),1)
  	TEST_OBJS += $(TARGET_OBJ_DIR)/BTControl.o
  endif
  
  SIMCOM_CFLAGS := -DFEATURE_DEMO_DATACALL \
  	-DFEATURE_DEMO_AT \
  	-DFEATURE_DEMO_VCALL \
  	-DFEATURE_DEMO_NAS \
  	-DFEATURE_DEMO_DMS \
  	-DFEATURE_DEMO_UIM \
  	-DFEATURE_DEMO_SMS \
  	-DFEATURE_DEMO_WIFI \
  	-DFEATURE_DEMO_GPIO \
  	-DFEATURE_DEMO_GPS \
  	-DFEATURE_DEMO_ADC \
  	-DFEATURE_DEMO_I2C \
  	-DFEATURE_DEMO_UART \
  	-DFEATURE_DEMO_SPI \
  	-DFEATURE_DEMO_FTP \
  	-DFEATURE_DEMO_SSL \
  	-DFEATURE_DEMO_HTTPS \
  	-DFEATURE_DEMO_FTPS \
  	-DFEATURE_DEMO_LBS \
  	-DFEATURE_DEMO_SYSTEM \
  	-DFEATURE_DEMO_ECALL \
  	-DFEATURE_DEMO_FOTA
  
  ifeq ($(BLUETOOTH_FUNC),1)
  	CFLAGS += -DFEATURE_DEMO_BT
  endif
  
  LFLAGS +=  -lm -lsdk -lcurl -lcrypto -lssl -lubus -lubox -lrilutil -ljson-c -luci -lpolarssl -llog -lprop2uci
  
  ifeq ($(BLUETOOTH_FUNC),1)
  	LIBS += -l:libbluetooth.so -lglib-2.0 -lreadline -lncurses -lgatt -lshared
  endif
  
  ifeq ($(PROJECT),a7600c-lxbd)
  	CFLAGS += -D__SIMCOM_PROJ_A7600C_LXBD__
  else
  	CFLAGS += -D__SIMCOM_PROJ_A7605CR2_LXBD__
  endif
  
  CFLAGS += $(SIMCOM_CFLAGS)
  
  BIN_OBJS = $(TEST_OBJS)
  
  # 单独添加 cJSON 文件进去
  $(TARGET_OBJ_DIR)/%.o:$(MAIN_DIR)/src/cJSON/%.c
  # $(TARGET_OBJ_DIR)/%.o:$(TESTC_DIR)/cJSON/%.c
  	echo OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
  	echo Build OBJECT $(@) from SOURCE $<
  	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
  	echo OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
  
  $(TARGET_OBJ_DIR)/%.o:$(EXAMPLE_DIR)/%.c
  	echo OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
  	echo Build OBJECT $(@) from SOURCE $<
  	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
  	echo OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
  
  $(TARGET_OBJ_DIR)/%.o:$(TESTC_DIR)/%.c
  	echo ---------------------------------------------------------
  	#echo $(ALL_PATHS)
  	#echo $(ALL_INCLUDES)
  	echo Build OBJECT $(@) from SOURCE $<
  	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
  	echo ---------------------------------------------------------
  
  $(TARGET_OBJ_DIR)/%.o:$(TESTC_DIR)/%.cpp
  	#echo $(ALL_PATHS)
  	#echo $(ALL_INCLUDES)
  	echo Build OBJECT $(@) from SOURCE $<
  	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
  	echo ---------------------------------------------------------
  
  .PHONY: all clean
  
  all:prep bin
  
  prep:
  ifeq ($(BLUETOOTH_FUNC),1)
  	@cp -a ./bluetooth/src ./src
  endif
  	@if test ! -d $(TARGET_BIN_DIR); then mkdir $(TARGET_BIN_DIR); fi
  	@if test ! -d $(TARGET_OBJ_DIR); then mkdir $(TARGET_OBJ_DIR); fi
  
  bin:$(BIN_OBJS)
  	@echo Create bin file $(BIN_NAME)
  	@$(CXX) $(ALL_LINKS) $(BUILD_LDFLAGS) -o $(TARGET_BIN_DIR)/$(BIN_NAME) -lmosquitto $^ $(LFLAGS)
  	@echo ---------------------------------------------------------
  
  clean:
  	@rm -fr $(TARGET_OBJ_DIR) $(TARGET_BIN_DIR)
  ```

  

##### - 改动部分解析

###### -- 添加头文件路径 & 定义一个目录

```makefile
#原始 Makefile 的目录
TESTC_DIR = $(MAIN_DIR)/src
###example目录
EXAMPLE_DIR = $(MAIN_DIR)/example/src

## 添加 example 头文件路径
ALL_PATHS += ./example/inc
# 由于下方 ALL_INCLUDES 已经单独添加了路径，因此这里就不添加了
# ALL_PATHS += ./src/cJSON
ALL_PATHS += $(LIB_PATH)/usr/include
ALL_PATHS += $(LIB_PATH)/usr/include/sdk_api
ALL_PATHS += $(LIB_PATH)/usr/include/sdk_api/common
ALL_PATHS += $(LIB_PATH)/usr/include/glib-2.0

# 将cJSON 也添加进 INCLUDE中
# ALL_INCLUDES = $(addprefix $(INCLUDE_PREFIX), $(ALL_PATHS))
ALL_INCLUDES = $(addprefix $(INCLUDE_PREFIX), $(ALL_PATHS), $(MAIN_DIR)/src/cJSON)
```

> 该部分没什么好讲的，

- addprefix 作用可以理解为为将后续的路径全部格式化一遍，中间时由空格分开的 ( 下方 chatGPT 回复 )

  ![image-20240314170440986](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720142.png)

  

###### -- 将 .c 文件编译成 .o 文件

```makefile
$(TARGET_OBJ_DIR)/%.o:$(MAIN_DIR)/src/cJSON/%.c
# $(TARGET_OBJ_DIR)/%.o:$(TESTC_DIR)/cJSON/%.c
	echo OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
    #echo $(ALL_PATHS)
	#echo $(ALL_INCLUDES)
	echo Build OBJECT $(@) from SOURCE $<
	$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<
	echo OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
```

作用：根据前面添加的头文件地址来编译 .c 文件。最后在 `TARGET_OBJ_DIR` 目录生成 .o。

字段解释: echo 只是打印输出字符串到终端的。

![image-20240314164140791](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720143.png)

1. 其中 `$(@)` 表示 `$(TARGET_OBJ_DIR)/%.o` 中的原始文件，`$<` 表示源文件。
2. `$(CC) $(CFLAGS) $(ALL_INCLUDES) $(OBJ_CMD) $@ $<` 其中 
   - CC 表示使用交叉编译工具编译
   - CFLAGS 是**宏定义**，**作为编译选项使用**，可能用在各类库文件( 动态链接库之类 )，也可能用在你所编译的 C 源文件中，就类似于 stm32 在 Keil 中配置的宏定义。( #ifdef XXXX     #endif )
   - ALL_INCLUDE 是前方定义的 **各类头文件路径**
   - OBJ_CMD 表示目标命令。这个工程的 Makefile OBJ_CMD 是 `-o` 表示连接



###### -- 生成目标文件

```makefile
bin:$(BIN_OBJS)
	@echo Create bin file $(BIN_NAME)
	@$(CXX) $(ALL_LINKS) $(BUILD_LDFLAGS) -o $(TARGET_BIN_DIR)/$(BIN_NAME) -lmosquitto $^ $(LFLAGS)
	@echo ---------------------------------------------------------
```

> 编译和连接部分就不提了，主要关注我自己加的

1. `@`  符号表示 防止执行结果输出到终端上。但为什么要加 echo 呢。。这不矛盾吗

2. `$^` 是一个自动化变量，表示规则中所有的依赖项列表，它会传给编译器 `$(CXX)` 用来链接生成最终可执行文件

3. `-lmosquitto` 是我使用 `mqtt_s.c` 文件时直接编译会报错，因此引了一个 mqtt 库进来，如果不引此库会报如下错误。

   <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720144.png" alt="image-20240314165231576" style="zoom: 67%;" />





#### SIMCOM_A7608 Linux SDK 各类外设使用

##### - GPIO

- 



##### - Uart

Linux 下使用串口收发数据的集中方式 [参考博客](https://www.eet-china.com/mp/a230001.html)

###### -select 函数使用

```c
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

<img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720145.png" alt="img" style="zoom: 50%;" />

- 在 linux 中，使用 select( ) 函数实现对 I/O 口的复用，传递 select 函数的参数会告诉内核:

  1. 我们要检查的文件描述符数量
  2. 对于每个描述符，我们关心的状态 ( 想从一个文件描述符中读或者写，还是关注一个描述符中是否出现异常 )
  3. 要等待多长时间，设置为 NULL 表示一直等待，不为NULL 的结构体表示等待最大时间。

  返回值: 表示有多少文件描述符处于可读、可写或者异常状态

- select() 函数相关常见宏定义

  ```c
  ```

- 注意事项：select( ) 函数阻塞的时候要确保前面的 fcntl( ) 函数没阻塞。否则会出现 `I/O possbile` 错误。



>  fopen( ) 和 open( ) 函数的区别。



###### -信号处理中断

- 设置串口描述符

  ```c
      // TEMP: 该部分代码还未整理，后续整理成笔记
  	struct sigaction sa;
      int flags = fcntl(UART_fd, F_GETFL, 0); // 先获取当前配置, 下面只更改O_ASYNC标志
      /* 将串口文件描述符设置为非阻塞模式，从而允许该文件描述符异步地接收数据和信号。*/
      fcntl(UART_fd, F_SETFL, flags | O_ASYNC);
  
      // 设置 SIGIO 信号的处理函数
      sa.sa_handler = sigio_handler;
      sigemptyset(&sa.sa_mask);
      sa.sa_flags = 0;
      /* 设置了 SIGIO 信号的处理函数为 sigio_handler，从而在该信号被触发时读取串口数据并进行处理。*/
      sigaction(SIGIO, &sa, NULL);
  ```

- 异常现象: 我在 while 循环中 放了一个打印输出的 printf，以及一个 write 函数。

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720146.png" alt="image-20240312184454964" style="zoom:50%;" />

  当我发送一个 aa中断数据时，它会触发两次中断

  ![image-20240312184655804](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720147.png)

  中断处理函数如下

  ```c
  int count = 0;
  void sigio_handler()
  {
      count += 1;
      printf("sigio_handler running! count = %d \n", count);
  }
  ```

  > 分析

  write 也会触发中断。当接受到中断时，触发 printf( ) 本来就睡眠的函数会被立即唤醒？进入到下一个循环？

  - 先是唤醒后执行了 `printf("Send MSG ret = %d \n", ret);` 之后再触发中断？
  - 根据现象分析：当我在 com9 发送数据时，

  预测 `write()` 函数会触发中断

  > 结论：

  1. 先触发中断，然后直接 sleep 结束 ( 直接唤醒 )，当 write时会触发中断。
  2. write 中断触发没有特别的快，而是先紧跟着write() 后面的一个函数执行了再进入中断 ( 当然，取决于执行的函数 )，

  > 解决方案:  ( 后续改为了使用 select() 函数来执行 uart 中断部分的操作 ) ==> 使用select 之后神清气爽，就没再出现过这个问题了

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720148.png" alt="image-20240313182109200" style="zoom: 80%;" />



##### - json 库

###### - `#include <json/json.h>`    

其实就是 json-c.c 这个库

Tip: `my_json_obj 不必每次都创建，可以创建一个然后复用它。`

> 创建单层 json [参考文档](https://linuxprograms.wordpress.com/2010/08/19/json_object_new_string/)

`The json object created: { "action": "getprodinfo" }`

```c
    json_object *jobj = json_object_new_object();
    json_object *my_json_obj = json_object_new_string("getprodinfo");
    json_object_object_add(jobj, "action", my_json_obj);
	printf("The json object created: %s\n", json_object_to_json_string(jobj));
```

> 多层 json 嵌套

`The json object created: { "action": "getprodinfo", "value": "3" }`

```c
    // 单层部分
	json_object *jobj = json_object_new_object();
    json_object *my_json_obj = json_object_new_string("getprodinfo");
    json_object_object_add(jobj, "action", my_json_obj);
    printf("The json object created: %s\n", json_object_to_json_string(jobj));

	// 第二层
    json_object *my_json_obj2 = json_object_new_object();
    my_json_obj2 = json_object_new_string("3");
    json_object_object_add(jobj, "value", my_json_obj2);
    // 打印输出: The json object created: { "action": "getprodinfo", "value": "3" }
    printf("The json object created: %s\n", json_object_to_json_string(jobj));
```

> 基本数据类型 json

`int object create: { "int_val": 10 }`

```c
    // 基本数据类型int
    json_object *m_int_obj = json_object_new_object();
    json_object *m_new_int = json_object_new_int(m_int);  // 创建int 的json类型
    json_object_object_add(m_int_obj,"int_val",m_new_int);
    printf("int object create: %s\n",json_object_to_json_string(m_int_obj));
```

> 数组类型 json  [参考博客](https://linuxprograms.wordpress.com/2010/08/19/json_object_new_array/)

TIP: 其中的 string 和 int 可以混用。

`array object create: { "test_arr": [ "11", 12, "13" ] }`

```c
    // 参考文档： https://linuxprograms.wordpress.com/2010/08/19/json_object_new_array/
    // 数组类型
    json_object *m_arr_obj = json_object_new_object();
    json_object *m_new_arr = json_object_new_array();
    json_object *str1 = json_object_new_string("11");
    json_object *int2 = json_object_new_int(12);
    json_object *str3 = json_object_new_string("13");
    json_object_array_add(m_new_arr,str1);
    json_object_array_add(m_new_arr,int2);
    json_object_array_add(m_new_arr,str3);
    json_object_object_add(m_arr_obj,"test_arr",m_new_arr);
    printf("array object create: %s\n", json_object_to_json_string(m_arr_obj));
```

> 错开对象 (添加数组索引) [参考博客](https://linuxprograms.wordpress.com/2010/08/19/json_object_array_put_idx/)

- 函数内容 `json_object_array_put_idx()`

- 作用：有时候前面的数组我们并不需要保存值，或者等待值传进之前需要 json

  ```c
      json_object *m_array_put_index = json_object_new_object();
  
      json_object *jarray = json_object_new_array();
  
      json_object *arr_index[3];
      arr_index[0] = json_object_new_string("index1");
      arr_index[1] = json_object_new_string("index2");
      arr_index[2] = json_object_new_string("index3");
      for (size_t i = 0; i < 3; i++)
      {
          json_object_array_put_idx(jarray, i + 2, arr_index[i]);
      }
  
      json_object_object_add(m_array_put_index, "my_arr_index", jarray);
      printf("array object create: %s\n", json_object_to_json_string(m_array_put_index));
  ```

- `struct json_object* json_tokener_parse(const char *str);` 该函数用于将 string 字符串生成 json_object 对象



- `json_object_is_type( )` 判断value的类型。

  ```c
    /*
    		json_type_object, “名称/值”对的集合 Json 对象的值类型
          json_type_boolean,
          json_type_double,
          json_type_int,
          json_type_array,  “值”的集合
           json_type_string
    */
  ```

  

  [参考地址](https://linuxprograms.wordpress.com/2010/08/19/json_object_is_type/)

- 





- 延申思考：

  对于字符型数组 ( C 中字符串 ) 而言，数组的每个元素的长度不确定，那么它是如何分配内存的呢？

  把它考虑成二维数组？







- [总览参考文档](https://blog.51cto.com/xzx951753/1707186)

> 清理创建的对象

<img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720149.png" alt="image-20240318165459286" style="zoom: 80%;" />

- 个人理解：当一个对象被 add 到两外 2 个(多个) 对象上时，对父对象进行释放，会导致子对象被释放 2次(多次)。因此使用 `json_object_put( );` 可以确保不被多次释放 (C++ 中好像释放不存在的对象会报空指针异常 )。

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720150.png" alt="image-20240319102930901" style="zoom:50%;" />

```c
// - 1
void json_object_object_del(struct json_object* jso, const char *key);
/* 从jso对象中删除键值为key的子对象，并释放该子对象及键值所占的内存。
（注：可能有文档说json_object_object_del只是删除而不释放内存，但实际上这是错误的）。 */

// - 2
json_object_put(baz_obj);
/*
	用于清理创建的 json 对象，释放资源
*/
```

> 其它暂未用到的函数( 先挖坑，后面再填 )

```c
// 1 排序用，排序方式可以自己定义
static int sort_fn(const void *j1, const void *j2) // 示例代码
{
    json_object *const *jso1, *const *jso2;
    int i1, i2;

    jso1 = (json_object *const *)j1;
    jso2 = (json_object *const *)j2;
    if (!*jso1 && !*jso2)
        return 0;
    if (!*jso1)
        return -1;
    if (!*jso2)
        return 1;

    i1 = json_object_get_int(*jso1);
    i2 = json_object_get_int(*jso2);

    return i1 - i2;
}
void json_object_array_sort(struct json_object *jso, int(*sort_fn)(const void *, const void *)); // sort_fn 是一个回调函数

// 2 获取数组长度
int json_object_array_length(struct json_object *obj);

// 3
struct json_object *json_object_array_get_idx(struct json_object *obj, int idx);

// 4 获取数据
int json_object_get_int(json_object *my_int);
```



> 总结

`json/json.h` 该库函数相对 cJSON 更简洁。只是看起来这些函数名字都好长。



###### - cJSON (该部分库笔记在 trm 多点测温的项目笔记)

##### - wait() 、sleep() 函数

- wait() 函数  [参考地址](https://www.cnblogs.com/linux-sir/archive/2012/01/27/2330014.html)

  ```bash
  #include <sys/types.h>
  
  #include <wait.h>
  
  int wait(int *status)
  
  函数功能是：父进程一旦调用了wait就立即阻塞自己，由wait自动分析是否当前进程的某个子进程已经退出，如果让它找到了这样一个已经变成僵尸的子进程，wait就会收集这个子进程的信息，并把它彻底销毁后返回；如果没有找到这样一个子进程，wait就会一直阻塞在这里，直到有一个出现为止。
  
  注：
  
  　　当父进程忘了用wait()函数等待已终止的子进程时,子进程就会进入一种无父进程的状态,此时子进程就是僵尸进程.
  
  　　wait()要与fork()配套出现,如果在使用fork()之前调用wait(),wait()的返回值则为-1,正常情况下wait()的返回值为子进程的PID.
  
  　　如果先终止父进程,子进程将继续正常进行，只是它将由init进程(PID 1)继承,当子进程终止时,init进程捕获这个状态.
  
  　　参数status用来保存被收集进程退出时的一些状态，它是一个指向int类型的指针。但如果我们对这个子进程是如何死掉毫不在意，只想把这个僵尸进程消灭掉，（事实上绝大多数情况下，我们都会这样想），我们就可以设定这个参数为NULL，就像下面这样：
  
  pid = wait(NULL);
  
  如果成功，wait会返回被收集的子进程的进程ID，如果调用进程没有子进程，调用就会失败，此时wait返回-1，同时errno被置为ECHILD。
  
  　　如果参数status的值不是NULL，wait就会把子进程退出时的状态取出并存入其中， 这是一个整数值（int），指出了子进程是正常退出还是被非正常结束的，以及正常结束时的返回值，或被哪一个信号结束的等信息。由于这些信息 被存放在一个整数的不同二进制位中，所以用常规的方法读取会非常麻烦，人们就设计了一套专门的宏（macro）来完成这项工作，下面我们来学习一下其中最常用的两个：
  
  1，WIFEXITED(status) 这个宏用来指出子进程是否为正常退出的，如果是，它会返回一个非零值。
  
  （请注意，虽然名字一样，这里的参数status并不同于wait唯一的参数–指向整数的指针status，而是那个指针所指向的整数，切记不要搞混了。）
  
  2， WEXITSTATUS(status) 当WIFEXITED返回非零值时，我们可以用这个宏来提取子进程的返回值，如果子进程调用exit(5)退出，WEXITSTATUS(status) 就会返回5；如果子进程调用exit(7)，WEXITSTATUS(status)就会返回7。请注意，如果进程不是正常退出的，也就是说， WIFEXITED返回0，这个值就毫无意义。
  
  
  ```

  

##### - select() 函数

> 与之类似的函数为 poll( )





#### Linux 部分

##### - 进程间通信 管道 (pipe)

> 还未达到使用进程间通信的级别。后续有需求再补(未必是这个文档)。



##### - 线程等待

- 需求: 

  &emsp;&emsp;使用由于使用 `pthread_join( )` 函数会导致线程阻塞，比如第一个线程开启了`pthread_join`之后，后续的线程会等第一个线程执行完之后还会继续运行。

  &emsp;&emsp;后来撤销掉 `pthread_join( )` ，转而配置线程属性。线程属性设置 **分离式线程** 确保线程运行完之后可以正确退出释放资源。用到的函数: `pthread_attr_init(&attr)`、`pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED)`、`pthread_attr_destroy(&attr)`。用到的结构体: `pthread_attr_t`。

  配置分离式线程后，不可以再使用 `pthread_join()`。

  <span style="color:Purple;">**问题点:** </span>

  &emsp;&emsp;之前配置的是在最后一个线程使用 `pthread_join( )` 函数，不规范，而且担心如果最后一个线程崩了，问题就很大，因此考虑使用线程间的信号阻塞来确定线程是否退出完。

  1. 线程崩溃之后可能不会执行判断标志位，导致主线程永远不会知道这个线程已经结束。
  2. 

- 



##### - 信号量







#### 程序开机自启

- 添加守护进程

  做成 systemctl ..... 这类启动文件。

- 写入进程到 `/etc/rc.common` 尾部

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720151.png" alt="image-20240305184808133" style="zoom: 67%;" />

  将该进程修改为 sh 脚本，执行的时候使用 sh 来执行。

- 





#### 新的东西

- `app_timer.h` 头文件中`typedef void (*timer_ind_cb_fcn)(union sigval v);` 写法

- `int strcmp(const char *str1, const char *str2);` 函数如果传入了 NULL ，那么程序会直接崩溃！。  只要触发了调用它所在的函数，就会崩溃，我这里是中断回调函数调用的它。

  



TODO:// 

1. 开发板烧录最新的 SDK 编译好的底层 img 镜像，

   img 镜像打包之类。

2. 向厂家要求提供烧录文档

3. adb 指令下缺少动态链接库，烧录新镜像进去查看是否能启动

4. 思考是否可以将必启动的程序项烧录到内核中。



### ASK: 

#### 1. 关于信号量，Linux中的信号量可以





### 总结

#### 踩到的坑

##### 1. (大) select() 函数所配置的uart_fd 必须配置为非阻塞，不然报错 'I/O possible'

> 现象描述：
>
> &emsp;&emsp;当uart 配置的 select( ) 函数，监听 uart 的 read，当使用串口工具向设备一发送串口消息时就报如下图的错误。

![image-20240322164558214](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720152.png)

- 代码部分

  ```c
  /**** 注意 flags 必须把 fcntl() 函数设置为非阻塞模式 ****/
      flags = fcntl(uart_fd, F_GETFL, 0);
      flags |= O_NONBLOCK;   // 这行代码就是设置非阻塞模式的
      fcntl(uart_fd, F_SETFL, flags);
  /**** 注意 flags 必须把 fcntl() 函数设置为非阻塞模式 END ****/
  ```

  ```c
      uart_fd = open(COM9, O_RDWR);
      set_opt(uart_fd, BAUD_RATE, 8, 'N', 1, 0);
      /****************Linux Uart 中断******************/
      flags = fcntl(uart_fd, F_GETFL, 0); // 先获取当前配置, 下面只更改O_ASYNC标志
      /* 将串口文件描述符设置为非阻塞模式，从而允许该文件描述符异步地接收数据和信号。*/
      flags |= O_NONBLOCK; // 如果要使用select() 函数，则必须设置flags为非阻塞，否则会报错 I/O error
      /*
              原因分析:flags 是阻塞 uart 设备描述符的。
              1. 下方 select() 函数设置的串口收到消息就会立即读取 I/O
              2. 当select 读 I/O 的时候发现 发数据的一瞬间
                  这个串口又又又又阻塞了，所以可能就报错 'I/O possible'
      */
      fcntl(uart_fd, F_SETFL, flags);
  
      while (1)
      {
              // 使用 select 函数检测串口是否可读
              fd_set rfds;
              FD_ZERO(&rfds); // 使用select()函数检测数据是否可读写
              FD_SET(uart_fd, &rfds);
  
              // struct timeval timeout;
              // timeout.tv_sec = 10;
              // timeout.tv_usec = 0;
              // int retval = select(uart_fd + 1, &rfds, NULL, NULL, &timeout);
              int retval = select(uart_fd + 1, &rfds, NULL, NULL, NULL);
              int n = read(uart_fd, buffer, sizeof(buffer));
              if (n > 0)
              {
              }
      }
  ```

  

##### 2. (大) 数据处理函数中 , strcmp( ) 函数形参有一个是 NULL 会导致调用这个函数时系统崩溃。

> `strcmp(json_object_get_string(v_obj), "getprodinfo")` 该函数



##### 3. (中) MQTT publish 发送客户端无法发送正常的 MQTT 数据，但是它可以正常接收。

如图： `mqtt res = 0` 表示正常时候的MQTT res。当出现异常时它会返回错误码。

<img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720153.png" alt="image-20240322192402713" style="zoom: 67%;" />

![image-20240322191336863](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202403251720154.png)

现象描述: 设备隔一小段时间就无法发送 MQTT 数据到 PC 端( 但是可以正常接收，此时 mqtt res 还是为 0 表正常 )，再过一段时间后 mqtt res 才等于 14。也就是错误码 MOSQ_ERR_ERRNO，再使用 perrno 打印出来的结论为Broken pipe。

> 问题解决：

&emsp;&emsp;是mosquitto_loop_start( ) 函数未使用，导致隔了一段时间 pub_mosq 这个对象和 broker 失去了连接。因此出现那种奇怪的现象。

&emsp;&emsp;警示: 了解一款芯片或者开发架构的时候请不要理所当然的按照自己的想法去做，应当先参照官方 demo, 然后再尝试添加自己的想法。



##### 4. (小) 注意字节大小，uint16_t 不能存放波特率，因为无符号 2 字节的范围是0~65535。以及 fopen() 要用fclose() 关闭

波特率的存放请注意数据范围。以及文件打开方式 fopen() 之后请使用 fclose() 关闭，open() 后不能使用 fclose() 关。否则会出现意料之外的报错。

注意: `FILE *check_file = fopen("config.bin", "r");` 只有文件存在时才会被打开，如果不存在，那么你用 ` fclose() ` 关闭会报错。



##### 5. (巨坑) ！注意使用fwrite() 和 fread() 的时候，写入的结构体顺序必须和读取的结构体顺序相同！否则数据会出问题。以及: fwrite() 形参的 size 和 数据个数的问题。

[关于fwrite() 函数和 fread()函数的解释! ](https://cloud.tencent.com/developer/article/1823693)

- 结构体传参

> 写入结构体时: 

<img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404111017965.png" alt="image-20240326163325862" style="zoom:50%;" />

> 读取结构体时：

<img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404111017967.png" alt="image-20240326163443156" style="zoom: 50%;" />

&emsp;&emsp;读取结构体时和 `fwrite( )` 函数写入结构体时的顺序千万不能错。我的理解: **因为他只是在文件中查找，而不是在数据库中找对象。指针没给到它正确的长度会导致数据错误。**这就我当时考虑如何存放全局变量以及配置时的一样，在思考如何根据索引来找到相应的值。。结果这是数据库该干的事情，但是这里只是文件，它没那么强大。

&emsp;&emsp;想要使用 `fread( )` 读取某个数据时，它前面的所有的数据都要读取，然后指针才会指到你想要的数据位置。

- 数据长度和个数问题

  ```c
  /** @func:  fwrite
  *   @brief: 向文件写入
  *   @para:  [buffer]:指向数据块的指针
  *           [size]:每个数据的大小，单位为Byte(例如：sizeof(int)就是4)
  *           [count]:数据个数
  *           [stream]:文件指针，如fp
  *   @return:实际写入的个数
  */
  size_t fwrite(const void* buffer, size_t size, size_t count, FILE* stream);
  ```

  **fwrite的返回值随着调用格式的不同而不同：** ( fread() 同理 )

  > **调用格式1**：`fwrite(buf,sizeof(buf),1,fp);`，将整个buf数据作为1个数据写入，则写入个数是1 成功写入返回值为1 
  >
  > **调用格式2**：`fwrite(buf,1,sizeof(buf),fp);`，将1Byte作为1个数据写入，则写入个数是sizeof(buf) 成功写入则返回实际写入的数据个数(单位为Byte)

  也就是格式1 表示把结构体打包写入，就写一个数据个数，记得读的时候也一个读。

  格式2 表示把结构体分开写入，可以一个( 多个 )字节一个( 多个 )字节来读写。

  

##### 6. (小) 结构体创建对象时不能放在头文件。否则在多次引用的时候，编译器会报错( 重复定义 )

```bash
#    在头文件中创建结构体对象通常是不推荐的，
# 因为头文件会被多个源文件包含，这样会导致
# 每个包含该头文件的源文件都会创建一个结构体
# 对象的副本，从而可能会导致重复定义的错误。
```



##### 7. (大) 字符串数组指针在添加赋值的时候请主动分配内存，否则程序崩溃。

问题点来源: 为了查找开发板其它的 uart，需要赋值，并遍历打开所有的串口，一个一个查看有哪些uart设备。

当然这么多 uart 设备符不可能一个一个敲，因此用循环来给定打开设备的路径。

当使用 `char *uart_dev[64] = {};` 时，要注意**后面的字符串指针不会自动初始化，如果直接强行赋值，会导致程序崩溃**

衍生提问: `uart_dev[64]` 中的索引为 0-4 的是定义的时候就指定了字符串常量，因此它内存分配到栈上( 如果是全局变量，它就在全局区 )，因此不用`free`, 而后面 4-64 是通过 `malloc ` 出来的，因此要 free，不然长时间运行的程序内存泄露会导致不可预见的错误。

```c
    char *uart_dev[64] = {  // 定义64个字符数组的指针，并初始化前4个字符串指针。
        // 64个
        "/dev/ttyS0",
        "/dev/ttyS1",
        "/dev/ttyS2",
        "/dev/tty",
    };

	/***** 给未初始化的 uart_dev 后续的赋值 *****/
    char s[20];
    char base[256] = "/dev/tty";
    for (size_t i = 4; i < 64; i++)
    {
        sprintf(s, "%d", i);

        strcpy(base, "/dev/tty");
        strcat(base, s);
        printf("base == %s\n", base);

        uart_dev[i] = malloc(strlen(base) + 1); // 注意使用数组指针的时候记得分配内存地址。否则会崩溃
        strcpy(uart_dev[i], base);
        printf("****** uart_dev[i] ==%s \n", uart_dev[i]);
    }

    /********* 用完请释放分配的堆的内存空间  **********/
    for (size_t i = 4; i < 64; i++)
    {
        free(uart_dev[i]);
    }
```



##### 8. (中) 定时器问题，。

1. timer_create() 创建的定时器中不能使用sleep( )，别问我为什么，因为我代码中sleep() 不执行。



##### 9. (巨坑) 使用 sigaction() 函数配置回调函数时，回调函数使用了 `阻塞型 read()` 函数，导致当前线程后面的 sleep() printf() 函数不起作用，而且 MQTT 线程无法正确接收数据。

- 现象说明:

  ```bash
  #  sa.sa_handler = sigio_handler; 中 sigio_handler 函数中使用了阻塞型 read() 函数。
  1. 后续while(1){} 的函数体不会被执行
  2. task_MQTT 中接收 MQTT数据的函数亦无法执行。
  
  # sa_hanlder(信号处理函数) 就和中断服务函数一样，它不能执行太长或者阻塞的代码，否则会造成不可预知的异常。
  ```

   现象1: MQTT自从串口中断触发，执行到信号处理函数中的 阻塞型 `read()`, 设备就无法收到 MQTT 数据了。

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404111017968.png" alt="image-20240408091737623" style="zoom:50%;" />

  现象2: 在信号处理函数使用阻塞型 `read()` 后, 串口第一条数据设备无法收到，后续才能正常接收。

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404111017969.png" alt="image-20240408094110801" style="zoom:67%;" />

  现象3: 该信号处理函数使用阻塞型 `read()` 后, 不仅仅在 MQTT线程异常，别的线程也会去世。普天之下只有这个 Uart 阻塞信号处理函数在执行。。

  

- 部分代码

  ```c
  void sigio_handler()
  {
      memset(rcv_msg, 0, sizeof(rcv_msg));
      int n = read(uart_fd, rcv_msg, sizeof(rcv_msg));
      if (n > 0)
      {
          printf("****** uart receive sigio_handler *****\n");
          printf("uart rcv_msg = %s , uart_fd = %d\n", rcv_msg, uart_fd);
          // 测试write() 一下
          write(uart_fd, rcv_msg, strlen(rcv_msg));
          cmd_process(rcv_msg); // 处理接收到的字符串  // 使用信号来处理此函数会不会好一点。
          // todo:使用信号来让cmd执行处理函数
      }
  }
  
  void uart_thread_func(){
  	fcntl(uart_fd, F_SETOWN, getpid());
      int flags = fcntl(uart_fd, F_GETFL, 0); // 先获取当前配置, 下面只更改O_ASYNC标志
      /* 将串口文件描述符设置为非阻塞模式，从而允许该文件描述符异步地接收数据和信号。*/
      fcntl(uart_fd, F_SETFL, flags | O_ASYNC | O_NONBLOCK); //!!!!!!!!!这里要设置位O_NONBLOCK，否则会出问题
      // 设置 SIGIO 信号的处理函数
      sa.sa_handler = sigio_handler;
      sigemptyset(&sa.sa_mask); // 初始化信号集，使其包含所有信号
      sa.sa_flags = 0;
      /* 设置了 SIGIO 信号的处理函数为 sigio_handler，从而在该信号被触发时读取串口数据并进行处理。*/
      sigaction(SIGIO, &sa, NULL); // 表示当前所属的进程。触发了IO的信号
  
  	while (1)
      {
          sleep(5);
          printf("````````thread_uart sleep 5 seconds Loop \n");
      }
  }
  ```

- 问题说明: `fcntl(uart_fd, F_SETFL, flags | O_ASYNC | O_NONBLOCK);` **这部分必须设置 `O_NONBLOCK`**, 否则当信号处理函数执行时会出现很多异常

- 原因分析: 

  可能信号处理函数执行







































