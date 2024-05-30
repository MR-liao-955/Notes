### Risc-V 架构下 Linux SDK 学习笔记

[toc]

> 前言：

1. 由于 Linux 部分内容太多了，因此该文档只是一个框架文档。具体细分还会写别的文档。
2. 移植 openssh && sftp 请查阅别的文档。





#### 一、Luban SDK

[官方文档地址](http://artinchip.com/knowledge/topics/luban-sdk-overview-luban.html)

Luban SDK 是通过 buildroot 编译框架进行裁剪生成的。



#### 二、编译环境准备

- Linux 环境

  1. 更换软件源

  2. 拉取工程文件

     `git clone https://gitee.com/artinchip/d211.git`

  3. 一键安装编译环境

     ```bash
     # 进入下载好的工程文件目录
     cd d211
     # 一键安装
     ./tools/scripts/oneclick.sh quiet
     ```
  
  

- 编译 & 烧录方法

  - 使用官方 demo_ 调试板子

  - 添加自己的板子进行修改

    `make add_board`



- Windows 环境

  - Vscode

  - adb 调试

  - 串口调试

     



#### 三、risc-v 交叉编译环境搭建

> 前言

1. 由于我们需要编译自己的工程文件，因此有必要搭建自己的交叉编译环境。
2. 从 A7608E 移植到 D213ECV 中，A7608E 构建工具用的 makefile。

##### - 参考官方文档的 'LVGL' 的编译环境搭建教程 TODO://





##### - 在网上搜到的 Risc-V 交叉编译环境搭建教程 todo://备做参考。

[知乎参考文档](https://zhuanlan.zhihu.com/p/72862396)

pay attention: 克隆 toolchain 仓库速度特别慢！ 目前暂不使用该知乎参考文档。



##### - 构建编译环境。

目前使用到的工具链为: `riscv64-unknown-elf-` . 

[参考博客教程](https://decaf-lang.github.io/minidecaf-tutorial/docs/step0/riscv_env.html)

1. 如果单纯设置用户变量, 直接写入到 `bashrc` 就好 `vim ~/.bashrc` 用来指定你当前的用户变量。

   随后 `source ~/.bashrc` 应用配置。

2. 如果设置为系统变量，就将下载好的编译工具( 以及子文件 )移动到 `/usr` 路径下。

   `cp ./riscv64-toolchain/* /usr/ -r`

检查是否成功, 输入 `riscv` 后按 `tab` 键，看看是否能打印出编译工具名字。

![image-20240415183353456](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404191600210.png)





##### - 遇到的坑

- (小) 官方文档烧录的某个镜像 demo 会报错。而换了另外的 defconfig 就没事。

- (中) `CMakeLists.txt` 文件编写中，设置源文件路径和头文件 路径需要先包含，一定要有一个顺序问题。

  否则编译报错会莫名其妙。 (后续不采用 CMake, 对于小工程，还是采用 Makefile 来写)

- (微小) size_t 改为为 unsigned char 或者 uint8_t。



- (巨坑) 编译器问题？头文件声明之后，别的文件引了头文件会导致多重定义报错。

  头文件中不能声明变量。**注意声明和定义的关系！！！！**





- 解决 make 编译报错问题: 多重定义，但是我的 源码中那个文件中并没有定义那个变量。

  方法：该编译器不支持头文件中存放定义( 声明 )。将头文件的声明全部注释，并在需要调用的地方 extern

  > 缺点：这样各类全局变量看着就十分混乱。
  >
  > 优点：能简单解决问题，测试时候就这样快速是解决就好。
  >
  > 改进：专门拉一个头文件进来，需要修改的时候就使用 extern，需要引用的时候就 extern const





#### 四、编译三方程序

> 引言

&emsp;&emsp;由于之前使用到 A7608E 模块写过 Linux 驱动代码。因此当时考虑的是直接构建 Makefile 文件或者其它的方式构建此工程。但，，，存在很多问题: 系统官方的库找不到 suach: mosquitto、json/json-c。

&emsp;&emsp;目前而言，不使用该模块上网，因此 mosquitto 库可以暂时不考虑。需要早点实现 Gpio、Uart 的驱动，再研究如何根据 Uart 和模块进行网络通信 ( 拨号、各类网络方面的业务 )。

> 问题点

&emsp;&emsp;官方并未给出编译第三方程序的文档。我这边自己写的 CMake 存在连接方面的报错 (编译源文件通过，但缺失 json-c 库的源文件，需要引入它的c，或者做成动态/静态链接库)。动态链接库: 程序运行时链接 (.dll、.so)；静态链接库: 编译时链接 (.a、.lib)；

&emsp;&emsp;在 ./d211/source/artinchip/test-gpio 中包含了 Gpio 的 CMake 示例工程，但是它无法正常地通过 CMake 编译，因为它报错缺失各类头文件，。以及找不到头文件的宏定义 (头文件选择错误)。

##### - 自己编写的 CMakeLists.txt 来编译工程  TODO://



##### - 编译官方的 gpio 例程报错解决

- 编译流程 ( 后面有时间了再使用自己的 CMakeLists.txt 运行 )

  ```bash
  # 1. 搭建编译环境，根据文档上方的构建编译环境。
  	#这里采用用户变量来选择编译工具。 
  	vim ~/.bashrc
  	#添加行
  	PATH=$PATH:/root/RISC-V/linux-sdk/test/riscv64-linux-x86_64-20210512/bin
  	
  # 2. 创建编译目录
  	#创建cmake工程目录
  	mkdir build
  	apt-get install cmake
  	cmake ..
  
  # 3. 编译随后出现下图的错误。
  	make
  ```

  ![image-20240418100048431](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404191600212.png)



- 图示报错解决处理步骤

  1. 查看芯片官方 CMakeLists.txt 文件

     ```shell
     cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
       
     project(test-gpio LANGUAGES C)
     
     # Suppress cmake unused warning
     set(ignore ${BUILD_DOC} ${BUILD_DOCS} ${BUILD_EXAMPLE} ${BUILD_EXAMPLES}
             ${BUILD_SHARED_LIBS}${BUILD_TEST}${BUILD_TESTING}${BUILD_TESTS})
     
     add_compile_options(-Wall)
     
     add_executable(test_gpio test_gpio.c)
     add_executable(test_gpio_output test_gpio_output.c)
     
     # Install
     # install directories
     if(NOT CMAKE_INSTALL_PREFIX)
             message(FATAL_ERROR "ERROR: CMAKE_INSTALL_PREFIX is not defined.")
     endif()
     include(GNUInstallDirs)
     message(STATUS "GNUInstallDirs=" ${GNUInstallDirs})  #后续打印 头文件路径。
     
     if(DEFINED CMAKE_INSTALL_FULL_LIBDIR)
             install(TARGETS test_gpio RUNTIME DESTINATION "${CMAKE_INSTALL_FULL_BINDIR}")
             install(TARGETS test_gpio_output RUNTIME DESTINATION "${CMAKE_INSTALL_FULL_BINDIR}")
     endif() # CMAKE_INSTALL_FULL_LIBDIR
     ```

     可以发现: `include(GNUInstallDirs)` 存在问题，因为 `cmake ..` 时未能打印出其路径

     ![image-20240418100432099](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404191600213.png)

     随后考虑该路径是不是某种定义或者其它。根据其命名，认定它为编译工具所指向的路径？

  2. 查看根文件下的 Makefile, 以及 根Makefile 中 `include package/Makefile.sdk` 的 Makefile.sdk

     并未找到

  3. CMake 后续再研究 TODO： 目前先解决工程问题，是使用 SDK 而不是开发 SDK

  

  还是在 SDK 根路径下使用 make test-gpio 才成功解决的。
  
  TODO:// 下午询问芯片厂家 单独编译 gpio、uart 模块的方法。/而不是编译整个 Linux 系统。





##### - 使用 Makefile 替代 CMake 来实现此工程的编译

> 使用 make VERBOSE=1 > log.txt 打印出所有日志并分析。

1. 在 log.txt 查找 `gpio.c`  这部分的日志，单独提出来可以看出使用的编译器以及 `头文件/源文件` 路径，以及编译选项和宏定义之类。

   ```bash
   /root/RISC-V/d211/output/d211_demo88_nand/host/bin/riscv64-unknown-linux-gnu-gcc --sysroot=/root/RISC-V/d211/output/d211_demo88_nand/host/riscv64-linux-gnu/sysroot   -Os -g0  -DNDEBUG   -Wall -o CMakeFiles/test_gpio.dir/test_gpio.c.o   -c /root/RISC-V/d211/source/artinchip/test-gpio/test_gpio.c
   ```

2. 根据询问厂家得知单独编译 `test-gpio` 的例程是在跟路径执行 `make test-gpio` 。

   ```bash
   # 同样，将日志打印出来，来看是如何编译出来的
   执行: make test-gpio VERBOSE=1 > log.txt
   # 查看到有用的信息和 make 编译整个工程一致。因此将此编译信息修改为 makefile 的形式。
   ```

   

> 自己手动写 Makefile ( 搞定 )

具体参考笔记参考 A7608E 的 makefile，然后再自己重新写。





##### - Uart 驱动

> 问题点

&emsp;&emsp;使用了 A7608E 的 uart 测试代码，刷的 `d211_demo100_nand_defconfig` 固件，发现只有 debug 口可以和电脑的 USB 转 TTL 通信。别的 Uart 口则无反应。

###### -- 官方 defconfig 的 uart demo100 无法收发数据，修改设备树解决。

- 在默认 defconfig 中 `d211_demo128_nand_defconfig` 能正常运行开发板上所有的 uart。而 demo100 只有debug 的串口有消息。

  ![image-20240428180803433](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404291823507.png)

- 询问厂家技术支持，得到模棱两可的回复。说的修改 pinmux ，实际上根本无从下手，后来 **对照了一下 demo100 和 demo128 的 .dts 设备树** 后发现了不同点。将 demo100 的修改成 demo128 的引脚映射 就能正常收发 uart 数据了。

  ```shell
  # demo128 的路径
  ./source/target/d211/fountainhead_demo128_defconfig/board.dts
  # demo100 的路径
  ./source/target/d211/fountainhead_nand100_defconfig/board.dts
  ```

  ![image-20240428181718867](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404291823508.png)

  

- 



###### -- Uart 官方







##### - GPIO 驱动

> 匠心创的方案：在 linux 内核 4.8 之后支持 GPIO 使用字符型接口
>
> 参考文档: http://www.pedestrian.com.cn/user/gpio/index.html

&emsp;&emsp;采用 `/dev/gpiochipx ` 来实现 GPIO 控制。并采用 ioctrl( ) 函数来控制。

###### -- 遇到的问题: 官方 test_gpio_output 的 demo 跑不通

- 现象如下：

  ![image-20240508105755466](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405081146161.png)



- 原因猜测：

  1. 和 uart 一样，匠芯创未专门适配此开发板，并未对 demo 进行调整。由于我使用的是开发板 (PF15) 的一个按键充当 GPIO，因此考虑是 **设备树的问题** 引脚的问题。

  2. 之前 Uart 问匠芯创时，他们说可能是 pinmux 的问题，但是并没给具体的问题解决思路... 但是我在 GPIO 这里的 './d211/target/d211/common/d211-pinctrl.dtsi' 文件下发现有 **PF15 的复用功能配置**

     <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405081146162.png" alt="image-20240508111632313" style="zoom: 50%;" />

     <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405081146163.png" alt="image-20240508111707566" style="zoom:50%;" />

     <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405081146164.png" alt="image-20240508111737126" style="zoom:50%;" />

     考虑与上述引脚复用有关系.

  3. 根据串口工具提示的 `pin PF15 already requested by 18610000.codec; cannot claim for PF:95` 报错 , 而且在 './d211/target/d211/common/d211.dtsi' 文件下发现此路径对应的 codec 设备树点。

     <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405081146165.png" alt="image-20240508112103791" style="zoom:50%;" />

  4. 因此去 './d211/target/d211/fountainhead_demo128/board.dts' 中查找 codec 节点并注释它。

     <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405081146166.png" alt="image-20240508113522029" style="zoom:80%;" />

  5. 重新编译 Linux 镜像并烧录，打开 PF15 官方 demo 就不报错了！！正常运行，且隔 1s 电平反转一次。

     ![image-20240508113811897](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405081146167.png)



###### -- GPIO 驱动流程

> TODO: 修改输出电平能力





> GPIO 流程
>
> Linux下include/uapi/linux/gpio.h 库官方API文档 https://docs.kernel.org/userspace-api/gpio/chardev.html

1. 使用常规的 open( ) 函数获取到 gpio 的 fd。注意 `gpiochipx`  仅仅表示 gpio 组，例如 PF15 为 `gpiochip5`，offset 为 15。PD4为 `gpiochip4`，offset 为 4.

   ```c
   // 我这里用到循环遍历所有的 gpiochip。。因此这里是数组。
   int fd_gopen[gpiochips];
   fd_gopen[i] = open(base_gname, O_RDWR); // base_gname 其实是 "/dev/gpiochipx"  x=1,2,3...
   ```

2. 配置 `struct gpio_v2_line_request req;` I/O 口的请求的结构体，用来获取设备的信息

   <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405141600020.png" alt="image-20240508165628876" style="zoom: 80%;" />

   ```c
   // 创建结构体对象
   struct gpio_v2_line_request req;
   
   // 设置你想要配置的选项 以及偏移量 (你所需要控制的IO)
   memset(&req, 0, sizeof(req));
   req.config.flags |= GPIO_V2_LINE_FLAG_OUTPUT;  // 配置 GPIO 方向
   req.num_lines = 1; //请求的GPIO数量，一次可以请求多个，以简化管理。
   req.offsets[0] = 15; // 设置偏移量，我们配置的 PF15，因此这里设置为15 
   strcpy(req.consumer, "gpio output pin"); // 用户配置的描述信息，(随便写都行)
   
   ```

   

3. 使用 ioctl( ) 函数获取到设备描述符的信息。

   目的: 将 req 的地址传入，用来获取对 GPIO 的 `lfd` 操作描述符 ( 下一个步骤会提到 )。

   注意: ioctl 函数属于比较底层的函数，有比他更高级的函数就尽量使用高级一点的。

   ```c
   // 我这里用到遍历所有的 gpiochip 因此 fd_gopen 是一个数组。。。
   ret = ioctl(fd_gopen[i], GPIO_V2_GET_LINE_IOCTL, &req); 
   ```

   

4. 使用 ioctl( ) 函数针对具体的 IO 进行操作 ( 拉高 / 拉低 )，通过 `req` 获取到的设备信息，得到具体 GPIO 的描述符 `lfd`

   方法：通过 **`struct gpio_v2_line_values value`**  此结构体对象来进行对 IO 的操作

   ioctl( ) 中的命令参数为 `GPIO_V2_LINE_SET_VALUES_IOCTL`

   ```c
   int lfd[gpiochips];
   
   // 创建控制命令的描述符
   struct gpio_v2_line_values value;
   value.mask = 1; // 掩码
   value.bits = 1;	// 设置 GPIO 输出电平的高低
   
   // 获取具体的 GPIO 描述符。这里我用的遍历，所以 lfd 是数组
   lfd[i] = req.fd;
   // 使用控制命令 GPIO_V2_LINE_SET_VALUES_IOCTL 对 GPIO 的描述符进行写入
   ret = ioctl(lfd[i], GPIO_V2_LINE_SET_VALUES_IOCTL, &value);
   
   sleep(3);
   // 对电平进行反转
   value.bits ^= 1;
   ret = ioctl(lfd[i], GPIO_V2_LINE_SET_VALUES_IOCTL, &value);
   ```



##### - I2C 驱动

> 吐槽：

修改设备树引脚后Linux必须重新编译。。。uboot 或者 内核都要重新编译。费事



> 目前 i2c0 可以正常驱动。但是 i2c2 无法识别 ( 盲猜还是设备树的问题 )
>
> 设备树引脚选择没问题。可能是电源驱动的问题

破案了，外部上拉的问题，如果外接传感器，传感器有上拉电源也能驱动。



- 代码部分解释

  ```c
  
  
  
  ```

  







##### - USB 作为 Device 连接电脑显示串口 (虚拟多串口)

1. [串口驱动方式 CDC-ACM、VCP、HID 对比](https://bbs.21ic.com/icview-3179084-1-1.html)



> Linux 内核启用 Gadget 后接电脑，用来显示串口，并与之通信。

这一部分跟着官方文档上走，注意要屏蔽掉 ADB 调试功能才能虚拟成 USB device





> usb 虚拟多串口 TODO:// 暂时不做，网上资料很少。 先搞SSH



> SSH 搞完了，回来理清思路

1. 如果开启 g_serial 模块

   ```bash
   -> Device Drivers
   --> USB support
   ---> USB Gadget Support
   ----> USB Gadget precomposed configurations
   		<*>Serial Gadget (with CDC ACM and CDC OBEX support)
   
   ```

   设备启动时会自动加载 usb device，电脑上就能看到串口 ，应该 Linux 内核自动识别的 usb dev。

   此时： 执行该部分会提示 busy。（为什么要加载 g_serial 模块？因为不加载就无法创建 `mkdir -p /sys/kernel/config/usb_gadget/g1/functions/gser.gs0` ）

   ```
   [aic@g1] # echo `ls /sys/class/udc` > UDC                                                                                                    
   sh: write error: Device or resource busy   
   ```

2. 虚拟 VCP https://community.renesas.com/the_vault/f/archive-forum/8468/multiple-com-ports-using-one-usb-peripheral

   可行性验证: http://janaxelson.com/usb_virtual_com_port.htm :monkey::laughing::angel:



###### -- TODO:// Windows 编写 USB 驱动。(HARD)

> 方案: :monkey_face: 修改

1. 使用 Visual Studio 下载安装 Windows kit 之类的 SDK 工具。
2. 参考文档：chatGPT、微软官方文档
3. 



> 实施: :laughing: 使用



```bash
# TODO: 目前认为该配置没问题，但是可能内核没开启 CONFIG_USB_ACM 
# 但内核开启了之后 执行 echo 10200000.udc > UDC 依旧报错
# zcat /proc/config.gz | grep CONFIG_USB_ACM # 查看内核是否开启ACM

mount -t configfs none /sys/kernel/config
cd /sys/kernel/config/usb_gadget

mkdir g2
cd g2

echo "0x04e8" > idVendor
echo "0x2d01" > idProduct

mkdir configs/c.1

mkdir functions/acm.GS0
mkdir functions/acm.GS1

mkdir strings/0x409
mkdir configs/c.1/strings/0x409

echo "0123456789" > strings/0x409/serialnumber
echo "Samsung Inc." > strings/0x409/manufacturer
echo "dearl usb" > strings/0x409/product

echo "Conf 1" > configs/c.1/strings/0x409/configuration
echo 120 > configs/c.1/MaxPower

// SourceSink：驱动 set configuration 会选取 第一个 configuration

ln -s functions/acm.GS0 configs/c.1
ln -s functions/acm.GS1 configs/c.1

echo 10200000.udc > UDC
```

















匠芯创 D21X

1. RJ45 网络接口，I2C、GPIO 等外设已验证，开发板插入 USB 调试口可以修改设备名。

2. 开发板插入 SD 卡后经过几个小时的 6500 个文件长时间读写测试，SD 卡会生成预期的文件个数和内容。

3. D213 芯片 USBH0 可以虚拟成单个 USB 设备端，与PC主机端通信。

后续将一个物理 USB 口虚拟为多个 USB 设备端，并尝试修改虚拟口的设备名，移植 SSH / SFTP 功能到开发板。







##### - 遇到的坑

- 在编译自己的代码 app 时，引入头文件报错 :disappointed_relieved:

  1. 问题描述:

     &emsp;&emsp;我 main.c 中引入了 config.h 头文件，而且 Makefile 的编译 main.o 的路径理论上是包含了 config.h 的路径的，但是编译依旧报错。![image-20240419153150808](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404191600214.png)

  2. 临时解决方法。

     路径问题~！！！薛定谔的路径 makefile 中宏定义变量使用 makefile 所在的文件夹的相对路径，

  3. 问题原因：暂未去查找。时间紧，任务重。







#### 五、RNDIS 移植 && mosquitto 移植 && json库移植

---

##### - 添加板子 make add_board && 启用 RNDIS 模块 && USB 接外部 4G 模块发送 AT 指令

###### -- 添加新板子

- 在 SDK 根目录下使用 `make add_board` 选择默认的 `demo128_nand_defconfig`

  ![image-20240428092924095](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037607.png)

  TIP: 经过后期测试，发现 demo128 才能驱动除 debug 以外的 Uart 设备。

  猜测: 可能是设备树之类，亦或者 pinmux 引脚复用未配置。

  后续结论: demo128 和 demo100 和 demo88 表示不同芯片引脚的开发板，它们的设备树映射也不同，因此需要修改成正确的 pinmux 引脚。

  

###### -- 内核中启用 RNDIS 模块

[参考文档->4G 模块 LINUX 集成用户手册 ( 域格 )](https://www.docin.com/p-2082456085.html)

[上述参考文档 -- 域格文档下载链接](https://www.yuge-info.com/uploads/soft/190912/ShanghaiYUGE4GModuleLINUXIntegratedUserManual.pdf)

> 内核配置部分

1. 使用 make km ( make kernel-menuconfig ) 进入图形配置界面

   ![image-20240428095753626](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037609.png)

2. 进入如下路径并开启内核功能

   ```bash
   Device Drivers
   - [*]Network device support
   -- <*>USB Network Adapters
   --- <M>Multi-purpose USB Networking Framework
   		<M>Host for RNDIS and ActiveSync devices #这里用 M 或者 * 都行。如果用*就不生成 rndis_host.ko 内核模块，而是直接编译进内核
   ```

   Ask: 使用 M 之后编译好的内核模块也能执行，而 chatGPT 提示需要加载内核才能运行。。。?



- 根据上方操作插入 usb 已经可以看到新添加的 usb 设备了，

  ![image-20240428101652454](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037610.png)

  > 将 EMC 网卡自动获取 DHCP 的 ip 之类的信息 （如果没有则重新插拔 usb）
  >
  > `udhcpc -i eth0`

  &emsp;&emsp; 获取DHCP 之后 ipaddr 查看当前 EMC 网卡信息

  ![image-20240428102046694](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037611.png)

  &emsp;&emsp;测试 `ping www.baidu.com`

  ![image-20240428102458736](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037612.png)

  

---

###### -- Linux 开发板外接 4G 模块，可以查看到 3 个 USB 设备，并向它发送 AT 指令设置 APN

> 需求：

&emsp;&emsp;开发板中 USB HOST 就类似于 Windows 电脑主机，USB 接入模块会显示 3 个串口



- 内核中使能 GSM 和 CDMA 模块功能。

  > > CDMA: ( 码分多址 Code Division Multiple Access )
  >
  > &emsp;&emsp;作用：用于无线通信的多址技术，主要允许多个用户在同一频谱上进行通信，且互不干扰。常用于移动通信、卫星通信、无线局域网等。

  1. 使用在 d211/ 根目录下运行 'make km' 进行内核选项的编辑。( 如果是其它 linux 内核源码，大部分采用 make menuconfig 来进行配置 )

  2. 配置选项

     Device Drivers ---> 

     &emsp;&emsp;USB support --->

     &emsp;&emsp;&emsp;&emsp;<*> USB Serial Converter support --->

     &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;<*> USB driver for GSM and CDMA modems

  

- Linux 开发板中使用 `lsusb` 查看 VID PID

  下图红色方框内就是合宙模块接入 Linux 开发板 USB Host 后新增的 usb 设备。 VID = 0x1782、PID = 0x4e00

  ![image-20240528103526768](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405281433358.png)



- 修改 Linux 内核 usb 驱动，添加目标模块 VID 和 PID

  1. 进入路径 ./source/linux-5.10/drivers/usb/serial/option.c 的源码中进行修改

  2. 添加模块的 PID 和 VID 做成宏定义

     ![image-20240528142154469](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405281433359.png)‘

  3. 添加添加 USB 设备到 option_ids[ ] 结构体数组

     ![image-20240528142502453](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202405281433360.png)

  



- 重新编译 Linux 内核，烧录后将模块加载到系统中( 这一部分看情况是否需要做，因为不做这一步依旧正常使用，我反正没看出区别 )。

  ```bash
  modprobe usbserial vendor=0x1782 product=0x4e00 
  ```



- 上述操作参考的 域格 的模块文档

<img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037613.png" alt="image-20240428102603465" style="zoom: 80%;" />

<img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037614.png" alt="image-20240428102626999" style="zoom: 80%;" />



---

##### - 移植 mosquitto 库

[编译 mosquitto 库参考文档](http://velep.com/archives/1423.html)

> 为什么要移植该库？

&emsp;&emsp;因为之前的 A7608E 的代码中使用到了该库，但是 D21x 这个板子的工程中并未包含此库，考虑把它交叉编译成 动态/静态 链接库。

> 移植流程：Eclipse 的 mosquitto 官网: https://mosquitto.org/

1. 官网下载库文件并上传到 linux 交叉编译环境中 下载地址： https://mosquitto.org/files/source/

   `curl -l https://mosquitto.org/files/source/mosquitto-2.0.18.tar.gz -o mosquitto.sdk.tar.gz`

   ![image-20240428104341869](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281130217.png)

2. 复制并解压 `tar -xvf mosquitto.sdk.tar.gz `

   ```shell
   tar -xvf your_tar_file.tar -C /your/target/path #带路径的解压方式
   ```

   ![image-20240428104803092](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281130218.png)

3. 在 SDK 根路径下修改 config.mk

   - 在文件开始的地方增加 如下交叉编译工具链和生成路径

     ```makefile
     CC=riscv64-unknown-linux-gnu-gcc
     CXX=riscv64-unknown-linux-gnu-g++
     CPP=riscv64-unknown-linux-gnu-g++
     AR=riscv64-unknown-linux-gnu-ar
     LD=riscv64-unknown-linux-gnu-ld
     DESTDIR=install_out  #设置目标生成路径，但是好像没啥用。。。
     ```

     <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281130219.png" alt="image-20240428105243300" style="zoom: 67%;" />

   - 添加 strip(剥夺、精简、条) 工具 ( 生产部署环境中去除目标文件的符号或者其它调试信息，减少文件大小 )

     ```shell
     STRIP?=riscv64-unknown-linux-gnu-strip
     ```

     ![image-20240428105621096](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281130220.png)

   - 修改 prefix 定义

     ```shell
     prefix=out
     ```

   - 去除不需要的部分 SSL 之类的

     ```shell
     WITH_TLS:=no
     
     WITH_TLS_PSK:=no
     
     WITH_SRV:=no
     
     WITH_UUID:=no
     ```

   - 编译 **make && make install** 并查看生成的目标文件

     并在 `./lib/install_out/lib/` 查看到 `libmosquitto.so` 的动态链接库文件。

     `./lib/install_out/include/` 查看到 `mosquitto.h` 的头文件。

   - 将 libmosquitto.so 移动至开发板，并运行测试程序。

   

- 编译通过后 adb push 推送到 Linux 系统下之后 **报错缺少动态链接库。**

  ![image-20240428111541027](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281130221.png)

  将动态链接库放入到 Linux 下，**需要设置环境变量来让程序找到它。**
  
  `export LD_LIBRARY_PATH=/path/to/libmosquitto:$LD_LIBRARY_PATH`
  
  Ask: 这个路径可以不必是它的真实路径。。它依旧找得到。。。奇了怪了。但是不写又报错。



##### - 移植 cJSON 库

[参考文档](https://www.cnblogs.com/20180211lijunxin/articles/14859898.html)

- 拉取 `git clone https://github.com/json-c/json-c.git `库并在 **json-c 同级目录下** 创建 `toolChain_json.cmake` 文件

  ![image-20240428112059889](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281130222.png)

  ```shell
  ## toolChain_json.cmake 
  set(CMAKE_SYSTEM_NAME Linux)
    
  SET(TOOLCHAIN_DIR "/root/RISC-V/linux-sdk/env_toolchain/riscv64-linux-x86_64-20210512")
  set(CMAKE_C_COMPILER ${TOOLCHAIN_DIR}/bin/riscv64-unknown-linux-gnu-gcc)
  set(CMAKE_CXX_COMPILER ${TOOLCHAIN_DIR}/bin/riscv64-unknown-linux-gnu-g++)
  
  link_libraries(m)
  
  set(CMAKE_FIND_ROOT_PATH "/root/RISC-V/linux-sdk/port_lib/json-c")
  set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
  set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
  ```

  

- 进入 json-c 中执行

  ```bash
  cmake -DCMAKE_INSTALL_PREFIX=/home/json-c/install -DCMAKE_TOOLCHAIN_FILE=../toolChain_json.cmake .
  ```

- 编译并查看生成的目标文件 `/home/json-c/install`

  ```shell
  make
  make install
  ```

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281130224.png" alt="image-20240428112730851" style="zoom:80%;" />

- 如果需要 `export LD_LIBRARY_PATH=/path/to/libjson-c:$LD_LIBRARY_PATH`



##### - 多线程写入文件，使用互斥锁

> 部分写入文件的笔记，请看我文档的 A7608E 部分

测试写入文件这部分，我们使用的互斥锁，来保护单次写入文件。









##### - 遇到的坑

---

###### -- 1.使用 demo128_defconfig 的默认配置编译的镜像无法使用 rndis (并非编译错误，而是使用上失误)

> 短期解决办法 

1. 先在内核中选中 rndis，随后在**编译路径中找到内核编译的模块**

   ```bash
   # 编译的时候重定向输出日志，查找到该模块的位置。
   make VERBOSE=1 > log.txt
   ```

   ![image-20240425165101175](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037615.png)

   ![image-20240425165622707](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037616.png)

   

2. 从 demo100_defconfig 中编译出内核模块。在编译环境中把此模块提出来。

   ```bash
   # 在路径中找到 rndis_host.ko
   /root/RISC-V/linux-sdk/d211/output/d211_demo100_nand/build/linux-5.10/drivers/net/usb
   ```

3. 使用 `adb push` 或者其它方法将其推到开发板中。

   ```bash
   # 将 rndis_host.ko 移动至内核中。
   /lib/modules/5.10.44/kernel/drivers/net/usb/rndis_host.ko
   ```

4. 随后就能正常使用 rndis 了

> 本质原因：

尴尬: 我内核配置文件中中选择的是 * 表示直接编译至内核中，所以没有 rndis_host.ko，而且为什么之前没连上网？因为 usb 没插拔导致不识别。

---



#### 六、将程序制作成 img 的 Linux 镜像。(硬件加密？)

// TODO.





#### 七、其它

- 设备树

  [设备树语法参考文档](https://www.cnblogs.com/xiaojiang1025/p/6131381.html)



- 移植 openssh & sftp 请查阅移植 openssh 文档部分。













#### 总结

&emsp;&emsp;思路方面的问题: 要找准目标，明白目前项目的需求，就 d21x 而言目前是实现芯片和 模块进行通信，**应当把精力花在业务部分上面。而不是折腾环境。**目的是使用 SDK，而不是开发 SDK。这部分也是我的缺点，办事情之前有必要理清思路，大局观的方向先找准，细枝末节是后话。考虑每次做项目时做好思维导图把握方向。







#### 方案与工作内容

- 方向: **先调通 gpio 和 uart 外设。后续 uart 和 合宙模块通信。该芯片不实现上网的功能**。但是 MQTT 解析类或者各种业务代码 拨号之类的需要在该芯片上完成。
- 类比: 将合宙模块理解为 usb 接电脑，设备管理器会多出一个网卡，且使用该网卡上网。
- 困难点: uart 如何与模块进行网络信息的交换。可能需要移植网卡驱动？
- 备选: 最后才考虑使用 AT 指令和模块通信，因为 AT 指令字段解析很复杂，但凡涉及字符串的拼接，需要判断的层面就很多。。。 除非做一个中间层，AT在芯片内部消化掉，自动适配不同的上网模块( 工程量巨大 )。











# TODO: 2024.5.24

1. 测试 RNDIS 稳定性。

2. D213X 外挂 4G 模块 ( 7608 or 724)，通过 AT 口向模块发送 AT 指令。

   原因: 因为模块接入到不同的网卡需要配置 APN ，内网卡和外网卡需要重新配置 APN。能发送 AT 指令即可。

3. 使用 C or C# 写 windows USB 驱动，大概率是和 Linux 开发板上配置的 VID 和 PID 进行绑定，这部分我考虑使用 C++ 来编写。

















1. 测试 RNDIS 长时间收发问题。

   ```bash
   1-13 15:16
   1-14  5:43
   
   
   9+5 = 14.30  6.30 + 14.30 == 9:00结束
   
   # 使用下方命令行 查看当前文件夹的文件个数
   ls -l | grep "^-" | wc -l
   
   ```

   

todo: 修改 usb 口，当插入电脑时，识别到 usb 口的名称改为 fountainhead 这种。

// 完成，搜全局设备名，改为 fountainhead。然后插入电脑，如果还是显示之前的，就卸载设备再拔插



// 移植参考教程

https://zhuanlan.zhihu.com/p/387939051



https://www.openssl.org/source/

```shell

// 编译openssl 时的配置选项
./Configure linux64-riscv64  no-asm shared no-async --prefix=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir
    
// 执行下方的配置选项就可以生成 lib库了
./Configure linux64-riscv64 --prefix=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir

./Configure linux-generic32 no-asm shared no-async --prefix=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir
    
vim Makefile



```





https://www.openssh.com/portable.html

https://mirrors.aliyun.com/pub/OpenBSD/OpenSSH/portable/

```shell
# 编译openssh
/root/RISC-V/linux-sdk/port_lib/ssh/zlib-1.3
    
./configure --host=riscv64-linux --with-libs --with-zlib=/root/RISC-V/linux-sdk/port_lib/ssh/zlib-1.3/install_dir --with-ssl-dir=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir  --disable-etc-default-login 
    
    
# 不禁用默认登录
./configure --host=riscv64-linux --with-libs --with-zlib=/root/RISC-V/linux-sdk/port_lib/ssh/zlib-1.3/install_dir --with-ssl-dir=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir 

./Configure --host=riscv64-linux --with-libs --with-zlib=/root/RISC-V/linux-sdk/port_lib/ssh-oldversion/zlib-1.2.9/install_dir --with-ssl-dir=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir --disable-etc-default-login 

--without-openssl-header-check
```

```bash
# 临时存放
.c.o:
        $(CC) $(CFLAGS) $(CPPFLAGS) -I /root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/include  -c $< -o $@
        
        
# 
ssh$(EXEEXT): $(LIBCOMPAT) libssh.a $(SSHOBJS)
        $(LD) -o $@ $(SSHOBJS) $(LDFLAGS) -lssh -lopenbsd-compat -L /root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/crypto $(LIBS) $(GSSLIBS) $(CHANNELLIBS)



# 可执行的配置命令
./configure --host=riscv64-linux --with-libs --with-zlib=/root/RISC-V/linux-sdk/port_lib/ssh/zlib-1.3/install_dir --with-ssl-dir=/root/RISC-V/linux-sdk/port_lib/ssh/openssl-3.2.1/install_dir --disable-etcdefault-login  CC=riscv64-unknown-linux-gnu-gcc AR=riscv64-unknown-linux-gnu-ar


#测试用
./configure --host=riscv64-linux --with-libs --with-zlib=/root/RISC-V/linux-sdk/port_lib/ssh-oldversion/zlib-1.2.9/install_dir --with-ssl-dir=/root/RISC-V/linux-sdk/port_lib/ssh-oldversion/openssl-3.0.12/install_dir --disable-etcdefault-login  CC=riscv64-unknown-linux-gnu-gcc AR=riscv64-unknown-linux-gnu-ar

```







```bash
#编译 zlib-1.2.8 流程
cd /home/ssh-code/zlib-1.2.8
mkdir install_dir                                              #创建安装目录
./configure --prefix=/root/RISC-V/linux-sdk/port_lib/ssh-test/zlib-1.2.8/install_dir   #执行之后会生成Makefile
vim Makefile                                                  #修改Makfile 将其中gcc、g++都修改为交叉编译器的名称。
## 原来代码
#  19: CC=gcc
#  ...
#  30: LDSHARED=gcc -shared -Wl,-soname,libz.so.1,--version-script,zlib.map
#  31: CPP=gcc -E
## 修改如下
#  19: CC=arm-linux-gcc
#  ...
#  30: LDSHARED=arm-linux-gcc -shared -Wl,-soname,libz.so.1,--version-script,zlib.map
#  31: CPP=arm-linux-gcc -E
make
make install


./configure --prefix=/root/RISC-V/linux-sdk/port_lib/ssh-oldversion/zlib-1.2.9/install_dir 


```



```bash
mount -t configfs none /sys/kernel/config
cd /sys/kernel/config/usb_gadget
mkdir g0
cd g0
echo "0x1d6b" > idVendor
echo "0x0104" > idProduct
mkdir strings/0x409
ls strings/0x409/
echo "0123456789" > strings/0x409/serialnumber
echo "AIC Inc." > strings/0x409/manufacturer
echo "Bar Gadget" > strings/0x409/product
mkdir functions/acm.GS0
mkdir configs/c.1
ls configs/c.1
mkdir configs/c.1/strings/0x409
ls configs/c.1/strings/0x409/
echo "ACM" > configs/c.1/strings/0x409/configuration
ln -s functions/acm.GS0 configs/c.1
echo `ls /sys/class/udc` > UDC
```







