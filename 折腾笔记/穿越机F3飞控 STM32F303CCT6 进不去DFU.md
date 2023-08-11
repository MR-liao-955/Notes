### stm32F303CCT6 进不去DFU

> 问题描述

1. 重新折腾穿越机，但是新的飞机我在调参的时候手贱，把UART 口给改了，导致betaflight 进不去地面站。
2. 根据betaflight 给的解决方法，需要重新刷写固件，但按住U-boot 给板子上电，并进不去DFU模式，也就无从谈起刷固件。
3. 抛开betaflight configure 软件，使用STM 的官方DFU_Demo 程序也无法识别按住u-boot 的飞控。



#### 寻求解决办法

- 使用STM32 官方的DfuSe 工具

  https://www.st.com/en/development-tools/stsw-stm32080.html



- 使用SWD 强制烧录固件



- 























