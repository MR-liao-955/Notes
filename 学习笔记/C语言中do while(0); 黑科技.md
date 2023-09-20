### do{ }while(0); 的黑科技

> 前言

&emsp;&emsp;对于循环来说，do while 循环仿佛一直用得比较少，除了必要的执行一遍循环体中的语句之外，我个人一般很少使用到该循环的语法。但是无意间在知乎看到了do while(0) 这个黑科技玩法。因此记录一下。



#### 宏定义

```c
// 一般使用宏定义的时候 一切正常
#define a 10
printf("a = %d",a);

// 当宏定义比较复杂的时候
#define log printf("hello \n");printf("world\n");
if(0)
    log
printf("-------");
system("pause");
// 此时会打印 
/* 
world
-------
*/

```

- 根据上面的情况，你也许会想到 给宏定义加一个括号{ }

  ```c
  // 加一个括号的情形1
  #define log {printf("hello \n");printf("world\n");}
  if(0)
      log
  printf("-------");
  system("pause");
  
  ```

  没错，这样确实能正常判断。但是我的if 后面加一个else 阁下应当怎么处理？

  ```c
  // 如果判断的是if else 
  #define log {printf("hello \n");printf("world\n");};
  if(0)
  {
      log
  }
  else
  {
      printf("-------");
  }
  
  
  system("pause");
  
  ```







#### 替代 goto:





#### TODO://

1. 页面的各类符号需要修改正确。

2. 开屏的等待交界面需要搞好。

3. 土壤温度这部分值为何还有，待解决。 -- 因为传参的时候导致

4. 设备不联网的时候会导致I2C 读温湿度失败

   ```bash
   # 情景描述：
   1. 如果未插卡，未联网的情况下，必须要在 下方中包含sys.wait() 函数才能正常拿到I2C 数据
       lvgl.obj_align(env1_Info, env1_QR, nil, 65, 0)                    -- 相对位置 -- 居中
       icon_create()
       sys.wait(4000)
       icon_flush()
       sys.publish("SCR_INIT_DONE")
   2. 如果插卡，联网的情况下，可以直接拿到I2C 数据，且不用sys.wait() 都可以拿到。
   
   # 猜测:
   1. 可能是 联网部分会抢占系统时钟之类的。
   2. 可能是 task 之间发布消息，但是温湿度那边正好被触发中断
       
   # 确定原因:
   中断热的话，sensor_cb， 该中断是设备插入识别的中断。
   再刨根问底: 就是 i2c_pwr() 写反了。应当是低电平能读取到数据
   
   
   ```

   

   

