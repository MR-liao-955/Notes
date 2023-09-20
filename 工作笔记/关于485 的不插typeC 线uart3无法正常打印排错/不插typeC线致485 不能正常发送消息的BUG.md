### 关于485 在typeC 拔出之后只有一个485口能正常收发串口数据，另外一个会出异常的排错

#### 产生背景:

&emsp;&emsp;4G 开关刷入测试代码时遇到的BUG，测试时，插着typeC 看日志，就没关心断掉typeC 的情况时串口数据能否正常收发。

&emsp;&emsp;现象: 当插着typeC 时一切正常，拔掉typeC 之后uart1 能正常收发数据，uart3 就会出现莫名其妙的数据乱码，或者数据不完整等问题。起初我还以为是硬件部分的问题。

&emsp;&emsp;比对: 和官方的uart demo 进行对照，官方的demo 在拔掉typeC 之后所有的uart 依旧能正常的工作。而我写的测试代码中有一个uart_485 有问题。所以这是软件部分的问题。

![image-20230920162634380](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309201626080.png)

#### 排查过程:

- 考虑是底层固件有问题？

  刷入相同固件，结果依旧没任何变化

- 考虑是引入的不同脚本有问题？

  注释掉` require ` 多余的脚本，依旧一致

- 考虑是串口接收代码的不同导致的问题？

  使用接收中断代码，结果还是一样。而且现象也包括发送的乱码。

- 考虑是三体人入侵地球了？

  emmmm，三体人还不至于对一个普通本科生下手。

- 发现有一行代码叫：pm.wake("testUart")

  emmmm, 好像就是这行代码，导致这个奇怪的问题。



#### 确定原因:

> pm.wake("testUart")

[官方文档的解释](https://doc.openluat.com/wiki/21?wiki_page_id=1960)

[合宙pm.wake(tag) 函数接口文档](https://doc.openluat.com/wiki/31?wiki_page_id=3961)

该函数用于让模块一直保持唤醒，如果不执行该函数就会导致那个BUG。

因此在uart 执行之前保持`pm.wake(tag)` 处于一直唤醒状态。



- 下方为验证是pm.wake() 函数问题点。

  间隔15s ，15s 内是正常收发，15s 后复现问题点，如此往复。

  ```lua
  -- 伪代码
  uart_id = 3
  sys.taskInit(function()
      local wakeup = 0
      local count = 0
      while true do
          sys.wait(1000)
  		-- 发送测试
          uart.write(uart_id, "uart_" .. uart_id .. "send")
          log.info("uart_" .. uart_id .. " sendMSG ")
  
          if count == 15 then
              count = 0
              if wakeup == 0 then
                  wakeup = 1
                  uart.write(3,"uart_3 will sleep 15s")
                  pm.sleep("testUart")
              else
                  wakeup = 0
                  pm.wake("testUart")
                  uart.write(3,"uart_3 wake 15s")
              end
          end
      end
  end)
  
  ```

  

#### 总结 && 发牢骚:

- 总结: 

  &emsp;&emsp;合宙 air_724 芯片一般情况下不使用低功耗的功能，因此每次在uart 部分最好打开pm.wake，保证uart 串口或者别的外设 不出现bug，该次问题排查还是有点离谱，主要开始没注意到休眠这部分的问题。



- 发牢骚:

  简直就离谱，为毛pm.wake() 没打开只会导致一个uart_485 出问题，而另外一个完全没问题呢？？为何又偏偏是拔掉typeC (日志/ 刷固件) 的usb 才会出现这个现象呢？？ 没找到原因之前我还以为三体人入侵地球了。

