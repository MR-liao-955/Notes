### 重新阅读 I2C 总线协议引起的思考



> 起因：

阅读 AT32F437VMT7 手册时看到 I2C 部分，就回想起之前做的 I2C 笔记，虽然我也经常是使用 I2C 总线，但是，对于 I2C 的地址又使得我重新考量。

#### 一个 MCU 接多个相同品牌相同型号的 传感器地址是否会冲突？

- 不可，除非同一个型号的传感器的地址是可配置的。
- 如果接 2个相同 I2C 地址的传感器，它们会根据线的长度来就近选择吗？



#### MBUS 总线





























































