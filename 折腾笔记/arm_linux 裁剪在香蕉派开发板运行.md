### 手动裁剪一个Linux 系统，步骤记录。

**写作目的：**

1. 为我的开发板进行减负，自定义一个Linux 系统，并在其中运行wordpress 服务 。
2. 学习并回顾之前的 qemu 交叉编译的过程。
3. 了解Linux设备的底层构建。
4. 为后续的开发做技术储备。



**框架：**

- 内核裁剪：make menuconfig 或者别的工具进行，( makefile配置 )交叉编译并生成 uimage 等内核镜像
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
  // curl 
  
  
  
  
  ```

  





#### 准备工作

- 资料下载

  ```shell
  // 内核文件的下载  这里我下载的 linux-5.1.2.tar.gz 版本
  https://cdn.kernel.org/pub/linux/kernel/v5.x/
  curl https://mirrors.ustc.edu.cn/kernel.org/linux/kernel/v5.x/linux-5.1.2.tar.gz | tar xvf -
  
  
  
  ```

  



- 这里使用X86 的ubuntu 系统作为编译环境

  ```shell
  // 使用xshell 连接ubuntu 服务器。
  // 使用WinSCP 传输文件，地址 `/home/kernal_workspace`
  tar -xvf linux-5.1.2.tar.gz
  
  
  
  
  ```

  







#### 关于一些概念

全志A83T 芯片

- A83T 基于armv7l 指令，即为32位，因此编译系统时请选择32位（armv8 为arm64）可以使用浮点运算`arm-linux-gnueabihf-`
- Cortex-A 系列架构。 



**内核文件讲解：**

&emsp;&emsp;在Linux内核源代码中，有许多主要文件和目录承担着不同的角色和功能。以下是一些常见的主要文件和目录以及它们的作用：

![image-20230619161443514](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306201648932.png)

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







#### 1.修改内核的配置文件，以特定的模板为基准

```
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
  1. 在官网下载 arm-linux-gcc-4.4.3.tar.gz  然后配置环境变量。
  2. 直接使用ubuntu 安装 gcc-arm-linux-gnueabi 且安装g++arm-linux-gnueabi
      - gcc（GNU Compiler Collection）是通用的编译器集合，可以编译多种编程语言，包括C、C++、Objective-C、Fortran等。它是C语言的编译器前端。
      - g++ 特定针对C++ 语言
  	sudo apt-get install gcc-arm-linux-gnueabi
  	sudo apt-get install g++-arm-linux-gnueabi 
  	
  3. 修改环境变量
      export ARCH=arm // 说明是arm体系 
      export CROSS_COMPILE=arm-linux-gnueabi- // 使用的交叉编译器
  
  
  ```



- 环境配置报错 

  1. `apt-get install flex`

  ![image-20230619172038163](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306201648933.png)

  2. `apt-get install bison`

     ![image-20230619174700562](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306201648934.png)

  3. 修改Makefile 的目标编译架构 为arm 

     ```bash
     vim Makefile
     /ARCH       # 使用n 或者N 翻页  必须和 arch 文件夹下的arm 完全一致
     # 将ARCH        ?= $(SUBARCH)  改为 ARCH        ?= arm
     ```

     ![image-20230619174549301](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306201648935.png)

     

- 配置make 裁剪环境报错  `make menuconfig`

  ```bash
  # 通过面板图形化界面来设置需要(添加/删除)的内核
  sudo apt-get install ncurses-dev
  
  ```

  ![image-20230619175448422](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/202306201648936.png)



#### 使用make menuconfig 裁剪内核相应功能

- 在linux-5.1.2 文件夹

  ```bash
  # 使用make clean 清理.config 配置文件
  
  
  # 使用make xxxx  选择特定开发板配置框架来作为内核处理文件
  
  
  # 
  
  
  
  ```

  



- 设备树 `linux-5.1.2/arch/arm/boot/dts `

  ```bash
  # 设备树用于描述
  
  
  
  
  
  
  ```

  

- 香蕉派M3 官网  [香蕉派镜像官网](https://wiki.banana-pi.org/Banana_Pi_BPI-M3#Image_Release)

  ![image-20230621105030786](https://raw.githubusercontent.com/MR-liao-955/Notes/main/img/image-20230621105030786.png)





#### 根文件系统，系统命令存放地

&emsp;&emsp;构建根文件系统可以使用 buildroot  或者busybox，











#### uboot 引导









































































