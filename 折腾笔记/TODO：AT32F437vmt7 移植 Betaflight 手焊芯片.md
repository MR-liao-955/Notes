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



#### 固件烧写+硬件的验证+连通 BF 地面站



#### Watching out bumps ( 踩过的坑 )



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







##### 不同的语法

- Betaflgiht 单元测试代码 C++ 编写 ( .cc 文件 ) 

  1. `motor_output_unittest.cc` 电机输出测试文件，这里关注 ``TEST( )` 这个宏定义。![image-20231219110551335](TODO%EF%BC%9AAT32F437vmt7%20%E7%A7%BB%E6%A4%8D%20Betaflight%20%E6%89%8B%E7%84%8A%E8%8A%AF%E7%89%87.assets/image-20231219110551335.png)

  2. `gtets.h` 谷歌测试框架头文件

     ![image-20231219111103770](TODO%EF%BC%9AAT32F437vmt7%20%E7%A7%BB%E6%A4%8D%20Betaflight%20%E6%89%8B%E7%84%8A%E8%8A%AF%E7%89%87.assets/image-20231219111103770.png)

  3. 谷歌单元测试参考博客

     https://www.51cto.com/article/285290.html 

     https://gjgjh.github.io/gtest.html#/%E4%BD%BF%E7%94%A8gtest

  



##### 之前看的 ESP32 部分 Dshot 协议驱动电机代码记录





##### Betaflight 正式代码阅读

