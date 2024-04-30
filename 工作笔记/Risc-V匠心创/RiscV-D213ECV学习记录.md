### Risc-V 架构下 Linux SDK 学习笔记

[toc]

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

  否则编译报错会莫名其妙

- (微小) size_t 改为为 unsigned char 或者 uint8_t。



- (巨坑) 编译器问题？头文件声明之后，别的文件引了头文件会导致多重定义报错。

  头文件中不能声明变量。**注意声明和定义的关系！！！！**





TODO:// 解决 make 编译报错问题: 多重定义，但是我的 源码中那个文件中并没有定义那个变量。

该编译器不支持头文件中存放定义( 声明 )。









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

- 询问厂家技术支持，得到模棱两可的回复。说的修改 pinmux ，实际上根本无从下手，后来对照了一下 demo100 和 demo128 的 .dts 设备树后发现了不同点。将 demo100 的修改成 demo128 的引脚映射 就能正常收发 uart 数据了。

  ```shell
  # demo128 的路径
  ./source/target/d211/fountainhead_demo128_defconfig/board.dts
  # demo100 的路径
  ./source/target/d211/fountainhead_nand100_defconfig/board.dts
  ```

  ![image-20240428181718867](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404291823508.png)

  

- 吐槽: 匠心创在没有订单的时候技术支持又慢又不到位。





##### - 遇到的坑

- 引入头文件报错

  1. 问题描述:

     我 main.c 中引入了 config.h 头文件，而且 Makefile 的编译 main.o 的路径理论上是包含了 config.h 的路径的，但是编译依旧报错。![image-20240419153150808](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404191600214.png)

  2. 临时解决方法。

     路径问题~！！！薛定谔的路径 makefile 中宏定义变量使用 makefile 所在的文件夹的相对路径，

  3. 问题猜测。



##### - GPIO 驱动

> 匠心创的方案：在 linux 内核 4.8 之后支持 GPIO 使用字符型接口
>
> 参考文档: http://www.pedestrian.com.cn/user/gpio/index.html

&emsp;&emsp;采用 `/dev/gpiochipx ` 来实现 GPIO 控制。并采用 ioctrl( ) 函数来控制。



问题点: 使用 gpiochipx 如何控制 GPIO？

```shell
#temp 后续删，下方的映射不一定对
gpiochip6 -> PU
# 用户层的GPIO 映射
gpiochip3 -> PC
gpiochip1 -> PD

# 选择正确的 gpiochip* 
可能需要查看


```









#### 五、RNDIS 移植 && mosquitto 移植

> 前言: D213ECV 的目的是实现 RNDIS 功能，它与上网模块连接，实现拨号功能。



##### - 添加板子 make add_board && 启用 RNDIS 模块

- 在 SDK 根目录下使用 `make add_board` 选择默认的 `demo128_nand_defconfig`

  ![image-20240428092924095](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037607.png)

  TIP: 经过后期测试，发现 demo128 才能驱动除 debug 以外的 Uart 设备。

  猜测: 可能是设备树之类，亦或者 pinmux 引脚复用未配置。

  

- 启用 RNDIS 模块

  [参考文档->4G 模块 LINUX 集成用户手册 ( 域格 )](https://www.docin.com/p-2082456085.html)

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

  

  > 但，教程中提示如果设备不支持的话需要修改内核设备源码 ( 如果需要的话 )
  >
  > 使用 `lsusb` 查看 VID PID

  - usb 内核在 `linux-5.10/drivers/usb/serial/option.c` 添加设备驱动，添加好驱动之后就可以执行 `modprobe usbserial vendor=1782 product=4e00` 来加载设备了。（不确定这一步骤的作用）

    但是: 存在根据 `域格` 的文档，并未在 `/dev/ttyU*` 中找到设备。也许这个模块是合宙的，存在差异。

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037613.png" alt="image-20240428102603465" style="zoom: 67%;" />

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404281037614.png" alt="image-20240428102626999" style="zoom:67%;" />

  

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

- 拉取库并在 **json-c 同级目录下** 创建 `toolChain_json.cmake` 文件

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



##### - 移植 A7608E 当时写的将配置信息写入到配置文件的函数

> 不必修改，直接使用。具体请看我文档的 A7608E 部分



##### - 遇到的坑

---

###### -- 1.使用 demo128_defconfig 的默认配置编译的镜像无法使用 rndis (不是坑，是我的问题。下方忽略。)

> 短期解决办法 ( 笨办法 )

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





#### 六、将程序制作成 img 的 Linux 镜像。



#### 总结

&emsp;&emsp;思路方面的问题: 要找准目标，明白目前项目的需求，就 d21x 而言目前是实现芯片和 模块进行通信，**应当把精力花在业务部分上面。而不是折腾环境。**目的是使用 SDK，而不是开发 SDK。这部分也是我的缺点，办事情之前有必要理清思路，大局观的方向先找准，细枝末节是后话。考虑每次做项目时做好思维导图把握方向。



- 编译头文件找不到的问题: 可以查阅 Makefile 中的 -i 看它包含了哪些头文件/路径
- 根目录的 Makefile 中 include(Makefile.sdk) 之后，Makefile.sdk 的所有头文件路径都会被包含，生命周期就是这个 make 的周期。
- Linux 中无后缀
- such as : <linux/gpio.h> 这个目录下，要找准目录，因为在 source/test_gpio 的例程中，编译会报错，宏定义的错误 ( 去 gpio.h 的头文件找这个宏定义)，因此要去 gpio.h 中查找这个宏定义，查看其是否真的包含它。
- make --debug=a > log.txt 将详细日志打印到 log.txt中



- 目前遇到的困难: TODO:// gpio 头文件的包含，头文件个数太多了，要单独编译 软件，而不是整个系统。
- 待解决: // 根据 log.txt 日志查看是包含的是哪些头文件。



#### 方案与工作内容

- 方向: **先调通 gpio 和 uart 外设。后续 uart 和 合宙模块通信。该芯片不实现上网的功能**。但是 MQTT 解析类或者各种业务代码 拨号之类的需要在该芯片上完成。
- 类比: 将合宙模块理解为 usb 接电脑，设备管理器会多出一个网卡，且使用该网卡上网。
- 困难点: uart 如何与模块进行网络信息的交换。可能需要移植网卡驱动？
- 备选: 最后才考虑使用 AT 指令和模块通信，因为 AT 指令字段解析很复杂，但凡涉及字符串的拼接，需要判断的层面就很多。。。 除非做一个中间层，AT在芯片内部消化掉，自动适配不同的上网模块( 工程量巨大 )。
- 











# TODO:

#### 五一回来：测试 SDK 稳定性。



#### 假期有空学一下: DTS 设备树以及其它的移植方面的东西.

内核部分也稍微学一下,避免遇到 BUG 无法解决.















