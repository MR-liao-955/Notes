## Windows USB 驱动学习记录

> 参考文档：微软的 USB 部分文档。(困难模式，文档还是机翻的，而且其它博客文档很少)

### 相关概念

思路理清：Linux 开发板作为 Device，Windows USB 作为 Host，当 Linux 作为 Device 时，需要调用内核的 Gadget 模块将 USB Device 虚拟为特定设备。

Windows 开发驱动是为了开发设备管理器里面的未识别的驱动，安装驱动后虚拟多个串口，与串口名称。

要了解 Windows WDF 框架，以及 KMDF 框架进行开发。









TODO://  Windows 驱动暂时不编写，这部分难度较大。

1. 使用 D21x 做成采集器, 功能要和采集器一致，对接 oneNet 云平台， 屏幕使用 C 版本的 LVGL，LVGL 采用官方的 LVGL。时限: 10天 :togo: -> 6月10日端午节要提交成果。
2. 如果 485 无法验证就改用虚拟串口。
3. 验证 SPI，PWM外设。
4. SNMP 移植、Sqlite 移植。( 做完采集器之后再移植 )
5. 后续小网关的项目参与协作 ( 暂时不关心，后续才考虑 )。





























