启动默认低电平IO 口整理

- 不可用GPIO 

  ```shell
  # uart占用
  20\21\23\28\
  
  # 按键占用
  24\25\26
  
  #LCD 和I2C 供电占用
  1\4\14\15\
  
  # LED 占用
  9\10\11\12\22\
  
  #I2C插入识别
  5\17
  
  #LCD 占用
  0\2\3
  ```

- 空闲GPIO 

  ```shell
  # 未被占用
   [22 -> 该GPIO 用于背光控制(但会出现白屏)]  、7、13、18、19、27、29、30、31、
  
  
  ```

  ![image-20231008103133914](%E5%90%AF%E5%8A%A8%E9%BB%98%E8%AE%A4%E4%BD%8E%E7%94%B5%E5%B9%B3IO%20%E5%8F%A3%E6%95%B4%E7%90%86.assets/image-20231008103133914.png)

  ![image-20231008102957955](%E5%90%AF%E5%8A%A8%E9%BB%98%E8%AE%A4%E4%BD%8E%E7%94%B5%E5%B9%B3IO%20%E5%8F%A3%E6%95%B4%E7%90%86.assets/image-20231008102957955.png)

  ![image-20231008103115180](%E5%90%AF%E5%8A%A8%E9%BB%98%E8%AE%A4%E4%BD%8E%E7%94%B5%E5%B9%B3IO%20%E5%8F%A3%E6%95%B4%E7%90%86.assets/image-20231008103115180.png)

  ![image-20231008103314874](%E5%90%AF%E5%8A%A8%E9%BB%98%E8%AE%A4%E4%BD%8E%E7%94%B5%E5%B9%B3IO%20%E5%8F%A3%E6%95%B4%E7%90%86.assets/image-20231008103314874.png)

  ![image-20231008103347244](%E5%90%AF%E5%8A%A8%E9%BB%98%E8%AE%A4%E4%BD%8E%E7%94%B5%E5%B9%B3IO%20%E5%8F%A3%E6%95%B4%E7%90%86.assets/image-20231008103347244.png)

  ![image-20231008103404697](%E5%90%AF%E5%8A%A8%E9%BB%98%E8%AE%A4%E4%BD%8E%E7%94%B5%E5%B9%B3IO%20%E5%8F%A3%E6%95%B4%E7%90%86.assets/image-20231008103404697.png)

  ![image-20231008103426729](%E5%90%AF%E5%8A%A8%E9%BB%98%E8%AE%A4%E4%BD%8E%E7%94%B5%E5%B9%B3IO%20%E5%8F%A3%E6%95%B4%E7%90%86.assets/image-20231008103426729.png)



- 可用GPIO

  - 空闲不可用GPIO 

    ```shell
    GPIO[7]
    GPIO[27]
    ```

  - 空闲可用GPIO

    ```shell
    13、18、19、29、
    
    
    ```

  - 有疑问 ，

    ```shell
    30、31  # 文档提示 724UG-MFM 和MFC 已经内置贴片SIM卡，因此不用做GPIO ，但我们是外置的
    ```

    

  























