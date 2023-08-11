### 手动裁剪一个Linux 系统，步骤记录。(未完成)

[toc]

#### 概览

**写作目的：**

1. 为我的开发板进行减负，自定义一个Linux 系统，并在其中运行wordpress 服务 。
2. 学习并回顾之前的 qemu 交叉编译的过程。
3. 了解Linux设备的底层构建。
4. 为后续的开发做技术储备。



**框架：**

- 内核裁剪：make menuconfig 或者别的工具进行，( makefile配置 )交叉编译并生成 uImage 等内核镜像
- 制作跟文件系统
- u-boot 启动文件
- 交叉编译软件 && 移植( 软件移植 / 系统移植到开发板 )



**裁剪保留功能：**

1. 保留网络部分驱动，
2. 保留usb 驱动，以及sata 硬盘之类的驱动。
3. 保留各类命令( 之前使用QEMU 进行裁剪的时候，做出来的成品少了很多命令，只保留了最基础的的命令之类 )，保留curl ,apt-get update 之类的指令。
4. 要能兼容 国内源 ，并且下载的包能正常运行。( 如果不能搞定源的问题，那么就解决软件编译的问题，软件手动编译 或者编译docker 放在容器内运行 )
5. //TODO: 设备树的编译和保留。( 第一版不做此要求，裁剪成功并能正确运行之后再做后续操作 )



**能学到的东西：**

1. 加深对Linux 的理解，
2. 裁剪合适的功能，未来开发会更有帮助。
3. 对编译不再害怕，哪怕跨平台的编译。
4. 熟悉xmake 或者 cmake 或者 交叉编译环境 的使用。



#### 关于一些概念

全志A83T 芯片

- A83T 基于armv7l 指令，即为32位，因此编译系统时请选择32位（armv8 为arm64）可以使用浮点运算`arm-linux-gnueabihf-`
- Cortex-A 系列架构。 



**内核文件讲解：**

&emsp;&emsp;在Linux内核源代码中，有许多主要文件和目录承担着不同的角色和功能。以下是一些常见的主要文件和目录以及它们的作用：

![image-20230801141759589](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308011418455.png)

1. arch/：该目录包含了不同的体系结构架构目录，如x86、arm、powerpc等。每个目录下都包含与特定体系结构相关的代码和配置文件。
2. block/：该目录包含了块设备相关的代码，用于处理和管理块设备，如硬盘和闪存等。
3. drivers/：该目录包含了各种设备驱动程序，用于支持不同的硬件设备。它包括网络设备驱动、存储设备驱动、输入设备驱动等。
4. fs/：该目录包含了文件系统相关的代码，用于支持不同的文件系统类型，如ext4、NTFS、FAT等。每个文件系统类型都有相应的子目录。
5. include/：该目录包含了内核头文件，用于定义各种数据结构、函数和常量。它提供了其他源代码文件中所需的声明和定义。
6. init/：该目录包含了内核初始化相关的代码。它包括启动和初始化内核所需的代码，如启动参数解析、设备初始化等。
7. kernel/：该目录包含了内核的核心代码。它包括调度器、进程管理、中断处理、内存管理等关键组件的实现。
8. mm/：该目录包含了内存管理相关的代码。它包括内存分配、页面管理、虚拟内存管理等。
9. net/：该目录包含了网络协议栈的实现。它包括各种网络协议的实现，如TCP/IP、UDP、ICMP等。
10. scripts/：该目录包含了一些构建和配置脚本，用于编译和构建内核。

这只是内核源代码中一些主要文件和目录的简要介绍，实际上内核源代码非常庞大复杂。不同的目录和文件在内核中承担着不同的功能和角色，负责不同的子系统和模块的实现和管理。详细了解和理解内核源代码的不同部分需要深入研究和学习。



##### uImage 和zImage 的讲解

[参考博客地址](http://conanwhf.github.io/2017/06/12/bootup-4-kernel/)

&emsp;&emsp;kernel编译后的成果是zImage，uImage只是添加了一个长度为0x40的头，用来记录给uboot的相关信息。虽然uImage是专门给uboot用的，但uboot的启动并不一定要uImage，它既可以接受uImage也可以接受zImage。可以说：uImage的头信息本质上是一种启动参数的传递形式。
&emsp;&emsp;制作uImage的工具是**mkimage**，它是来自于uboot。为了避免各种可能的版本问题，建议直接使用uboot编译后的`uboot/tools/mkimage`。而uImage的制作脚本则包含在kernel的编译脚本中，使用命令：
`make uImage LOADADDR=0x80008000`
即可编译获得uImage，前提是你已经将mkimage放入了编译环境的/bin/文件夹下。LOADADDR是kernel的启动地址（**注意，这不是真正的kernel运行地址**），uBoot会将kernel拷贝到此地址后（实际中也可能不拷贝）执行。关于uboot使用的几个内存地址的具体讲解也很多，这么些年也没什么变化，需要了解的可以自行去搜索。



##### .s 和.S 文件的区别

```bash
  在汇编语言中，.s 和 .S 是两种常见的文件扩展名，它们在大多数情况下用于表示汇编源代码文件。它们的区别在于对预处理器的处理方式。

  .s 文件：以小写字母 .s 结尾的文件是汇编源代码文件，通常是直接汇编的源文件。这意味着文件中的代码会被原样汇编，不经过预处理器的处理。任何在该文件中的宏或条件编译指令都不会被展开或处理。

  .S 文件：以大写字母 .S 结尾的文件也是汇编源代码文件，但是与 .s 文件不同的是，它会经过预处理器的处理。预处理器会展开宏、处理条件编译指令，并对代码进行预处理。因此，.S 文件中可以包含 C 预处理器的指令，如宏定义、条件编译等，这使得代码更加灵活和可读性更好。

  使用 .S 文件的好处是你可以在汇编代码中使用更多的预处理功能，这样可以使代码更加简洁和模块化。另外，你可以在 .S 文件中嵌入 C 风格的宏，这样可以提高代码的可读性和可维护性。

  总之，.s 文件是未经预处理的汇编源代码文件，而 .S 文件是经过预处理的汇编源代码文件。根据你的需求和代码的复杂性，你可以选择使用合适的文件类型来编写汇编代码。
```



##### 编译Linux 的uImage 内核镜像时对于LOADADDR 扩展的思考

- 起因：我想编译uImage 内核，并使用u-boot 引导，但是uImage 比zImage 头部多一个LOADADDR 的内存地址，但是我翻芯片手册没翻到。
  1. u-boot 初始化硬件设备CPU、内存控制器、外设等
  2. u-boot 从存储介质( Flash,SD卡，网络等 ) 方式加载Linux 内核镜像和设备树到RAM 中
  3. 再然后加载设备树，( 我的理解：在主控芯片之外的 开发板的信息保存到RAM 中，让芯片知道 '电脑' 的配置情况 )。然后将设备树的地址传递给内核
  4. 加载内核，u-boot 会跳转到u-boot 内核的入口，并将设备树的地址和其它启动参数传递给内核。
  5. Linux 内核的初始化，即为编译过内核的驱动或者别的，u-boot  的使命完成剩下的交给Linux 了。



- 关于RAM 的地址映射 和内存大小的思考

  ![image-20230803112957125](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037008.png)

  &emsp;&emsp;上图为STM32F407 的Memory mapping，从地址-x4000 0000 到0xA000 0FFF 的地址区间，通过计算，可以转化为。**0xA000 0FFF - 0x4000 0000 = 0x6000 0FFF = 1610616831 (byte) = 1536 (MB) ≈ 1.5(GB)** 

  &emsp;&emsp;显然作为一款单片机目前不可能有1.5GB 的内存，**而这些地址范围只是代表处理器的地址空间，并非实际的物理内存的大小。**

  &emsp;&emsp;STM32F407 ( Flash memory up to 1 Mbyte, up to 192 Kbytes of SRAM ) ，但是上图这个逻辑地址表明STM32F407 支持最大的内存，也就是外挂的内存。实际上面的最高地址 0x0 -> 0xFFFF FFFF 可转化为4GB



- 关于扩展的RAM 和芯片内部的RAM 的理解

  > 对于CortexM4 架构和 CortexA7 架构的初略认识。

  1. 以STM32F407ZGT6 为例架构为CortexM4，它的Memory mapping 分为APB1,APB2,AHB1,AHB2,AHB3,Reserved,CORTEX-M4 internal perpherals 等区域。

  2. CortexA7 根据chatGPT 回答

     AXI 总线：高性能外设和高速 RAM（如 DDR RAM）通常连接到 AXI 总线上，这样可以实现较高的数据传输速率。

     AHB 总线：低速 RAM、SRAM、Flash 存储器以及其他较低性能的外设通常连接到 AHB 总线上。

     APB 总线：较低速度的外设（如串口、GPIO 控制器等）通常连接到 APB 总线上。

     Cortex-A7 系列的芯片的内存分布是更为灵活和复杂的，因为它们通常用于更多复杂的应用，有更高的性能要求。

  > BananaPi M3 拥有2GB ram、8GB emmc 的理解
  >
  > [参考地址](https://www.yiboard.com/thread-725-1-1.html)

  &emsp;&emsp;对于高性能的芯片，作为用户级的处理器，应当支持多样性的定制，类似于树莓派4B 那种，2G、4G、8G内存的定制，同时也可以把emmc 这类存储和内存的设计原理关联起来。

  &emsp;&emsp;要使用LOADADDR 需要找一个没有被用过的内存空间( 绕开APB AHB... 系统外设占用的RAM )，

  

- 关于LOADADDR 的一部分理解

  理论上说只要LOADADDR 设置的地址和芯片各类 总线/寄存器 的地址不冲突，LOADADDR的地址可以随意。具体看芯片手册，但是。。全志A83T 的芯片像是传家宝一样，根本找不到有用的手册





#### 修改内核的配置文件，以特定的模板为基准

```bash
ASK: arm 内核文件中 路径arch/arm/configs 的各类_defconfig 是什么?
Answer:
在ARM架构的Linux内核源代码中，路径arch/arm/configs下的各类_defconfig文件是一些默认配置文件，用于定义不同ARM架构的硬件平台的内核配置选项。

这些_defconfig文件包含了一系列的配置选项，用于启用或禁用不同的内核功能和驱动程序，以及设置硬件平台特定的参数。这些文件提供了内核配置的基础，可以根据具体的硬件平台需求进行自定义配置。

每个_defconfig文件都对应于一个特定的ARM硬件平台或开发板。它们通常以硬件平台的名称或开发板的名称命名，例如imx6q_defconfig（i.MX6 Quad开发板）或bcm2835_defconfig（树莓派开发板）等。

这些文件可以作为内核配置的起点，可以直接使用它们作为基础配置，也可以根据需要进行修改和自定义。通过选择适当的_defconfig文件，开发人员可以快速配置适合特定硬件平台的内核。

使用make <defconfig>命令，例如make imx6q_defconfig，可以基于对应的_defconfig文件生成内核的默认配置。开发人员可以在此基础上进行进一步的配置和定制，以满足特定硬件平台的需求。
```

[参考地址](https://anriku.top/2018/12/24/%E6%B2%A1%E5%BC%80%E5%8F%91%E6%9D%BF%E5%81%9ALinux%E5%B5%8C%E5%85%A5%E5%BC%8F%E5%BC%80%E5%8F%91-%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%90%9E%E5%AE%9A%E4%B8%80%E5%88%87/)

- 低版本Linux 内核镜像需要配置镜像描述，高版本则需要` make dtbs `&emsp;来编译设备树



- 安装arm 交叉编译环境

  [Linux 内核各个版本下载地址](https://cdn.kernel.org/pub/linux/kernel/)
  
  [交叉编译工具链下载地址 Linaro](https://releases.linaro.org/components/toolchain/binaries/)
  
  ```bash
  # arm 交叉编译环境有两种，
  1. 在非官网下载 arm-linux-gcc-4.4.3.tar.gz  然后配置环境变量。
  	地址: http://www.friendlyelec.com.cn/download.asp  (不可使用该编译器，因为它是32位的)
  '如果使用教程的arm-linux-gcc-4.4.3 编译器，也许在QEMU 的环境下能运行，但是这次我们不用QEMU'
  X86交叉编译工具下载地址： https://releases.linaro.org/components/toolchain/binaries/
  下载版本号： gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf.tar.xz
  
  ## 添加环境变量
  sudo vi ~/.bashrc
  ###添加如下内容
  PATH=/root/arm-gcc-4.4.3/arm-linaro-linux-gnueabihf/bin:$PATH
  
  ## 让配置立即生效
  source ~/.bashrc
  	
  ## 测试是否生效
  输入 arm-none-  然后按TAB 键看看是否有命令补全
  
  ## 修改Makefile 的属性配置
  vim /root/linux-kernel/linux-5.1.2/Makefile
  
  ###修改为如下内容
  ARCH            ?= arm
  CROSS_COMPILE   ?= arm-none-linux-gnueabihf-
  
  
  ################################# 分割线 ###################################
  	
  2. 直接使用ubuntu 安装 gcc-arm-linux-gnueabi 且安装g++arm-linux-gnueabi
      - gcc（GNU Compiler Collection）是通用的编译器集合，可以编译多种编程语言，包括C、C++、Objective-C、Fortran等。它是C语言的编译器前端。
      - g++ 特定针对C++ 语言
  	sudo apt-get install gcc-arm-linux-gnueabi
  	sudo apt-get install g++-arm-linux-gnueabi 
  	
  ## 修改环境变量
      export ARCH=arm // 说明是arm体系 
      export CROSS_COMPILE=arm-linux-gnueabi- // 使用的交叉编译器
  
  ```
  
  apt-get install make -y
  
  > 我采用的是 gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf 交叉编译工具
  
  ```bash
  # 按照上面的交叉编译工具链安装好之后，进行编译会报错
  ## 倘若遇到
  /*
      In file included from /root/arm-gcc/gcc-linaro-arm-linux-gnueabi/bin/../lib/gcc/arm-linux-gnueabi/6.1.1/plugin/include/gcc-plugin.h:28:0,
                       from scripts/gcc-plugins/gcc-common.h:7,
                       from scripts/gcc-plugins/arm_ssp_per_task_plugin.c:3:
      /root/arm-gcc/gcc-linaro-arm-linux-gnueabi/bin/../lib/gcc/arm-linux-gnueabi/6.1.1/plugin/include/system.h:681:10: fatal error: gmp.h: No such file or directory
       #include <gmp.h>
                ^~~~~~~
      compilation terminated.
      scripts/gcc-plugins/Makefile:54: recipe for target 'scripts/gcc-plugins/arm_ssp_per_task_plugin.so' failed
      make[2]: *** [scripts/gcc-plugins/arm_ssp_per_task_plugin.so] Error 1
      scripts/Makefile.build:500: recipe for target 'scripts/gcc-plugins' failed
      make[1]: *** [scripts/gcc-plugins] Error 2
      make[1]: *** Waiting for unfinished jobs....
      Makefile:1273: recipe for target 'scripts' failed
      make: *** [scripts] Error 2
  */
  # 执行
  sudo apt-get install libgmp-dev
  
  ## 倘若遇到
  /*
      HOSTCXX scripts/gcc-plugins/arm_ssp_per_task_plugin.so
      In file included from scripts/gcc-plugins/gcc-common.h:95:0,
                       from scripts/gcc-plugins/arm_ssp_per_task_plugin.c:3:
      /root/arm-gcc/gcc-linaro-arm-linux-gnueabi/bin/../lib/gcc/arm-linux-gnueabi/6.1.1/plugin/include/builtins.h:23:10: fatal error: mpc.h: No such file or directory
       #include <mpc.h>
                ^~~~~~~
      compilation terminated.
      scripts/gcc-plugins/Makefile:54: recipe for target 'scripts/gcc-plugins/arm_ssp_per_task_plugin.so' failed
      make[2]: *** [scripts/gcc-plugins/arm_ssp_per_task_plugin.so] Error 1
      scripts/Makefile.build:500: recipe for target 'scripts/gcc-plugins' failed
      make[1]: *** [scripts/gcc-plugins] Error 2
      Makefile:1273: recipe for target 'scripts' failed
      make: *** [scripts] Error 2
  */
  # 执行
  sudo apt-get install libmpc-dev
  
  ```



- 环境配置报错 

  1. `apt-get install flex`

  ![image-20230801141810611](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308011418456.png)

  2. `apt-get install bison`

     ![image-20230801141815810](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308011418457.png)

  3. 修改Makefile 的目标编译架构 为arm 

     > 注意make sunxi_defconfig 需要在内核的根路径执行。也就是 ...../linux-5.1.2/
     
     ```bash
     vim Makefile
     /ARCH       # 使用n 或者N 翻页  必须和 arch 文件夹下的arm 完全一致
     # 将ARCH        ?= $(SUBARCH)  改为 ARCH        ?= arm
     ```

     ![image-20230801141821524](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308021729284.png)
     
     

- 配置make 裁剪环境报错  `make menuconfig`

  ```bash
  # 通过面板图形化界面来设置需要(添加/删除)的内核
  sudo apt-get install ncurses-dev
  
  ```

  ![image-20230801141826294](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308011418459.png)



#### 使用make menuconfig 裁剪内核相应功能

- 在linux-5.1.2 文件夹

  ```bash
  # 使用make clean 清理编译过的.o .d .s .i zImage等编译过程/结果文件
  make clean
  
  ##如果有需求需要清理.config 文件时请使用 \
  make mrproper
  make distclean
  
  # 使用make xxxx  选择特定开发板配置框架来作为内核处理文件
  ## chatGPT 给的回复，sunxi_defconfig 的内核配置文件是和 香蕉派M3 最相似的
  make sunxi_defconfig 
  
  # 修改内核所需要以及不需要的文件。
  make menuconfig
  
  # 编译内核
  make -j15   # 使用15个CPU 核心进行编译
  如果要编译uImage 就需要指定 LOADADDR
  参考地址： https://www.cnblogs.com/bigsissy/p/11093789.html
  
  # 在 arch/arm/boot/ 中能找到编译好的内核文件
  
  # 在 arch/arm/boot/dts 中可以找到 sun8i-a83t-bananapi-m3.dts 的设备树文件
  
  ```
  
  > 如果遇到如下报错
  
  ![image-20230804182558792](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037009.png)
  
  ```bash
  # 添加环境变量
  export ARCH=arm  # 这行命令可以没有，因为在Makefile 文件里修改了ARCH ?= arm 所以没问题
  export CROSS_COMPILE=arm-linux-gnueabi-
  # make -j15  编译之后遇到提问
  GCC plugins (GCC_PLUGINS) [N/y/?] (NEW) 
  # 我选的N
  ```
  
  

- 编译uImage 文件

  ![image-20230804095826371](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037010.png)

  ```bash
  # 上图使用了，但是编译报错
  make uImage -j15 LOADADDR=0x40008000
  # 第一次编译会报错，提示没有找到mkimage工具，需要手动安装u-boot-tools即可；
  sudo apt-get install u-boot-tools
  
  ```



- 设备树 `linux-5.1.2/arch/arm/boot/dts `

  ```bash
  
  ##DTS（Device Tree Source）:
     DTS 是设备树的源文件，通常以 .dts 或 .dtsi 为扩展名。它是一种用于描述硬件平台和设备的中立、可读性较好的文本文件。DTS 文件使用一种特定的语法来描述设备树的结构，其中包含了硬件设备的属性、寄存器配置、中断信息等。DTS 文件通过编译器（如 dtc）转换为 DTB 文件。
  
  ##DTB（Device Tree Blob）:
     DTB 是设备树的二进制文件，通常以 .dtb 为扩展名。它是 DTS 文件编译后得到的二进制形式，用于在运行时被 Linux 内核加载和解析。在启动过程中，Bootloader 会将 DTB 文件加载到内存中，并传递给 Linux 内核。Linux 内核会使用 DTB 文件中的信息来配置和初始化硬件设备，以及在运行时进行设备的管理。
  
  # 区别：
     DTS 是设备树的源文件，使用文本形式描述硬件设备和平台信息，可读性较好，易于编辑和理解。
  DTB 是经过编译器转换的二进制文件，包含了从 DTS 文件中提取出的设备树信息。它是在运行时由内核加载和解析的，不可直接编辑或读取。
     在 Linux 内核的嵌入式系统中，设备树的使用允许将硬件描述与内核代码分离，从而实现硬件平台的独立性和可移植性。设备树允许在同一内核镜像上支持多种不同的硬件平台，而无需为每个硬件平台单独编译内核。因此，设备树在嵌入式系统中起到了非常重要的作用。
  ```

  

  ```bash
  # 设备树用于描述
  
  - 作为初学者，我就不自己修改设备树了。
  - 在 /linux-6.1.42/arch/arm/boot/dts 中就有各厂商的设备树文件
  - 而且刚好有我需要的 sun8i-a83t-bananapi-m3.dts 设备树源文件。
  
  # 在 ./linux-6.1.42 目录下执行 编译dts 生成dtb 文件
  make dtbs
  
  ```

  

- 香蕉派M3 官网  [香蕉派镜像官网](https://wiki.banana-pi.org/Banana_Pi_BPI-M3#Image_Release)

  ![image-20230801141832075](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308011418460.png)





#### 根文件系统，系统命令存放地

&emsp;&emsp;构建根文件系统可以使用 buildroot  或者busybox，

[buildroot 教程](https://www.lxlinux.net/9319.html)

[官网&&下载地址](https://buildroot.org/)



```bash
# 使用buildroot 会下载很多资源包。因为buildroot 本体比较小，如果还要使用它提供的编译器的话，就会下载很频繁
# 由于没有刚好适配香蕉派M3 的官方配置文件 因此用m2 代替
make bananapi_m2_ultra_defconfig
```

> 修改Target options 选项， 不使用浮点计算

![image-20230804184458101](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037011.png)

>修改Toolchain 选项，交叉编译工具链使用我们自己下载的工具链，并配置地址。

请注意 toolchain kernel headers series( )  我用的 linaro-gcc-arm-linux-6.1.1 版本的，这里选的4.6.x 否则报错。。我也不知道为什么。

![image-20230804185245290](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037012.png)

![image-20230803175825789](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037013.png)



> 修改System configuration 

![image-20230804185603093](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037014.png)



> 配置Filesystem images 设置文件系统格式

![image-20230804185709632](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037015.png)



> 配置Kernel 选项  禁止编译Linux 内核 和U-boot，内核+uboot 我们自己来( 除非我编译的uboot 跑不起来 )

![image-20230804185845248](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037016.png)

![image-20230804185953511](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037017.png)



> 提示要安装unzip

![image-20230807090600086](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037018.png)



> 编译工具报错- 原因：我是用外部交叉编译工具，应当在Toolchain prefix 指定我用什么类型的( gnueabi/gnueabihf.... )

![image-20230807095413186](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037019.png)

![image-20230807095632512](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037020.png)

- 可以开始编译



#### uboot 引导

[u-boot 下载地址](https://ftp.denx.de/pub/u-boot/)

- 概览( 报错解决办法在下面的 '报错指南' )

  ```bash
  # 在configs 文件夹中存放了很多开发板的默认官方u-boot 配置文件
  ## 由于没有Bananapi_m3 的，因此我们用稍微相似的 bananapi_m2_berry_defconfig
  make bananapi_m2_berry_defconfig
  
  # 进入make menuconfig 配置界面 修改部分。
  ARM architecture  --->
    [*] Sunxi SoC Variant (sun8i (Allwinner A83T))  --->
      (X) sun8i (Allwinner A83T)
  
  export CROSS_COMPILE=arm-linux-gnueabi-
  make -j15   # 使用15核 进行编译
  ```

  TIP: 由于属于初学者，我的目的是为了自己编译内核和u-boot ，根文件系统 放入Linux 开发板让它跑起来，

  因此修改部分能少改动就最好，避免跑不起来。

- 报错指南

  1. 使用在配置好make menuconfig 之后使用 make 编译报错

     ![image-20230804163214538](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037021.png)

     ```bash
     # make 工具不知道用什么编译器来编译这类文件，因此要export 指定编译类型,
     export CROSS_COMPILE=arm-linux-gnueabi-
     
     ## 对于不同的编译器具体看你的选择
     
     ### 注意： 该方法在你退出终端之后这个环境变量就会消失，还会继续报错，你如果还要继续编译你应该再次输入export XXXXX 指令
     ### 如果你想每次不用这么麻烦，那么也可以将它写入到 vim ~/.bashrc 的用户变量中。
     
     # 上面方法等同于把 CROSS_COMPILE=arm-linux-gnueabi- 写入到make 命令中
     make CROSS_COMPILE=arm-linux-gnueabi- -j15
     ```

  2. 好了，make 编译工具你已经选择好了，然后又有如下报错了

     ![image-20230804163744118](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037022.png)

     ```bash
     # 这部分报错是python 环境的问题
     ## ModuleNotFoundError: No module named 'distutils.core' 报错信息
     
     # 1. 检查是否有安装 distutils.core 模块
     python3 -m distutils.core
     ## 我这里提示 /usr/bin/python3: No module named distutils.core
     # 2. 安装 distutils.core
     python3 -m pip install --upgrade pip
     python3 -m pip install --upgrade setuptools
     
     
     ### 然后我这里又提示/usr/bin/python3: No module named pip  
     ### 说明没安装python 的pip
     # 3. 安装pip
     sudo apt-get install python3-pip    # 如果是python3 就安装它，别的版本另外谷歌
     
     ## 再次安装 distutils.core
     python3 -m pip install --upgrade pip
     python3 -m pip install --upgrade setuptools
     
     # 这个报错就搞定了！
     
     
     ```

  3. 解决上一个报错问题之后又来，swig 库的问题

     ![image-20230804164809442](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037023.png)

     ```bash
     # 安装swig 组件
     apt-get install swig
     ```

  4. 之后报错一片红，因为缺少库的原因( 图片还没截完 )

     ![image-20230804165012669](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308071037024.png)

     ```bash
     # 该错误说明u-boot 编译的时候缺少 OpenSSL库的头文件
     apt-get install libssl-dev 
     
     
     # 此时理论上编译就能生成 u-boot,如果是make menuconfig 设置修改错误，也会导致编译报错。
     # 建议先使用一个相似的 官方配置文件 编译通过之后再进行部分的修改
     make bananapi_m2_berry_defconfig
     make menuconfig
     make -j15
     ```

     

#### 修改Linux 登录界面

- 文本编辑器 修改 vim /etc/motd

  ```bash
  # 修改登录提示界面
  vim /etc/motd
  
  # 添加以下信息
              #############################  
              #                           #
              #  道路千万条，安全第一条。 #
              #  删库一时爽，亲人两行泪！ #
              #                           #
              #############################
  
  |\_/|     *****************************    (\_/)
  / @ @ \    *                           *   (='.'=) 
  ( > º < )   *    莫删库，删库必被抓！   *   (")_(")
  `>>x<<     *                           *
  /  O  \    *****************************
  
  
  
  ```

  































































