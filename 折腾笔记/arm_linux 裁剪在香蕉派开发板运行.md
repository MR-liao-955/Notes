### 手动裁剪一个Linux 系统，步骤记录。

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
4. 熟悉xmake 或者 cmake 或者 arm-gcc-gnu-eabi 的使用。



- 涉及我之前不太熟悉的 Linux 命令

  ```sh
  // curl 命令
  curl -L https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.4.7.tar.xz -o ./test.tar.xz
  
  // rz sz 命令，快速发送/接收文件
  
  // yum
  
  
  ```
  
  





#### 准备工作

- 资料下载

  ```shell
  // 内核文件的下载  这里我下载的 linux-5.1.2.tar.gz 版本
  https://cdn.kernel.org/pub/linux/kernel/v5.x/
  curl https://mirrors.ustc.edu.cn/kernel.org/linux/kernel/v5.x/linux-5.1.2.tar.gz | tar xvf -
  
  // 查看文件大小
  du -h linux-5.1.2.tar.gz
  
  ```
  
  



- 这里使用X86 的ubuntu 系统作为编译环境

  ```shell
  // 使用xshell 连接ubuntu 服务器。
  // 使用WinSCP 或者rz命令 传输文件，地址 `/home/kernal_workspace`
  tar -xvf linux-5.1.2.tar.gz
  
  
  
  
  ```

  







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

&emsp;&emsp;这个问题太基础，只是因为跟手动跑系统有关，所以稍微提一下。kernel编译后的成果是zImage，uImage只是添加了一个长度为0x40的头，用来记录给uboot的相关信息。虽然uImage是专门给uboot用的，但uboot的启动并不一定要uImage，它既可以接受uImage也可以接受zImage。可以说：uImage的头信息本质上是一种启动参数的传递形式。
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







#### 1.修改内核的配置文件，以特定的模板为基准

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
  
  ##如果有需求需要清理.config 文件时请使用 make mrproper
  
  
  # 使用make xxxx  选择特定开发板配置框架来作为内核处理文件
  ## chatGPT 给的回复，sunxi_defconfig 的内核配置文件是和 香蕉派M3 最相似的
  make sunxi_defconfig 
  
  # 修改内核所需要以及不需要的文件。
  
  # 编译内核
  make -j15   # 使用15个CPU 核心进行编译
  
  # 在 arch/arm/boot/ 中能找到编译好的内核文件
  
  # 在 arch/arm/boot/dts 中可以找到 sun8i-a83t-bananapi-m3.dts 的设备树文件
  
  
  
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
  
  
  
  
  
  
  ```

  

- 香蕉派M3 官网  [香蕉派镜像官网](https://wiki.banana-pi.org/Banana_Pi_BPI-M3#Image_Release)

  ![image-20230801141832075](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202308011418460.png)





#### 根文件系统，系统命令存放地

&emsp;&emsp;构建根文件系统可以使用 buildroot  或者busybox，











#### uboot 引导









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

  































































