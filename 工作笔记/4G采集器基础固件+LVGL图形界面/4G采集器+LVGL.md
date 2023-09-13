###  4G 采集器+LVGL 

[toc]

**概览**

- LVGL (Light and Versatile Graphics Library 轻量级通用图形库) 是广泛用于嵌入式等小型设备的一个图形库
- 相比于之前使用Lua 的给屏幕打点的方式来制作图形界面，使用LVGL 更简单，且移植性更好。
- LVGL 部分在本次项目中全部自己搭建，采集器部分只需要适配之前采集器的代码就行。



#### LVGL 部分

##### - 目标：

- 制作出如下图的界面效果，并且界面的文字可以更改，方便后续做出调整

  ![image-20230908112948021](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811916.png)

  ![image-20230908112953241](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811917.png)

  ![3](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811918.jpg)

  

  ![image-20230908113002576](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811919.png)

- 实际效果( 部分页面底图还是值得优化 )

  ![3](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811920.png)

  ![image-20230908113438556](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811921.png)

  ![image-20230908113450972](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811922.png)

  ![image-20230908113531931](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811923.png)



- 整体思路

  &emsp;&emsp;由于之前使用过java 的Swing GUI 框架，所以了解一部分做屏幕的大体思路(虽然一个是程序，一个是屏幕)，只要有需要实现的效果，就一定会找到对应的解决办法。

  - 以图片作为底图，在图片上方添加相应的控件，主要区分一种层次感。其中包含一部分面向对象的思维。
  - 翻页的具体实现可采用控件的删除，或者控件的隐藏( 覆盖 )。
  - 页面切换通过控件的属性，或者外部中断来触发。

  

##### - 整体困难:

- LVGL 官方给的库和合宙的LVGL 库是不一样的，很多官方LVGL 库函数，合宙并没有移植，导致使用上存在困难。
- 思路部分问题，图标使用贴图或者 直接ps 到底图，这一部分走了一些弯路。
- 合宙LVGL 将UI 设计部分教程下架了，只有根据我师傅以前在 **VScode 插件中使用UI 模拟器 **来进行参考，但它不是万能的，因为很多自动生成的函数不能( ~~直接~~) 使用，需要一点一点排查。

- 刚开始的 label 标签控件的对齐，以及img 控件的导入显示，控件的位置关系都存在理解上的问题
- 官方固件没有bar 控件的函数，需要定制相应固件，而只包含LVGL 库的固件没有浮点库( 而不能使用4G 采集器的部分函数，**定制固件中需要包含 LVGL + Float库** )。



##### - 框架层次:

![image-20230908120202194](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811924.png)

- page_scr_bottom.lua 

  &emsp;&emsp;该文件包含页面底图，底图放置在cont 容器控件中，方便整体管理全局的属性。该文件中的控件可以理解为所有页面的公共部分。

  - 设置横屏 + cont 容器控件存放底图，确保，方便统一修改属性

    img_bottom 控件作为全局变量，上层容器创建时调用该控件作为页面加载。(类似于父类、子类的关系)

    ```lua
        -- 设置横屏
        lvgl.disp_set_rotation(nil, lvgl.DISP_ROT_270)
    
        -- 加载全局字体
        local my_font = lvgl.font_load("/lua/opposans_b_20.bin")
        my_style = lvgl.style_t()
        lvgl.style_init(my_style)
        lvgl.style_set_text_font(my_style, lvgl.STATE_DEFAULT, my_font)
    
        -- 图像控件的父类控件  -- 容器控件
        img_bottom_cont = lvgl.cont_create(lvgl.scr_act(), nil)
        lvgl.obj_set_auto_realign(img_bottom_cont, true)
        lvgl.obj_align(img_bottom_cont, nil, lvgl.ALIGN_CENTER, 0, 0)
        -- 自动居中
        lvgl.cont_set_fit(img_bottom_cont, lvgl.FIT_TIGHT)
        lvgl.cont_set_layout(img_bottom_cont, lvgl.LAYOUT_COLUMN_MID)
        -- 全局修改字体，因为所有的style 都是依附于该容器的
        lvgl.obj_add_style(img_bottom_cont, lvgl.LABEL_PART_MAIN, my_style)
    
        -- 创建图像控件
        img_bottom = lvgl.img_create(img_bottom_cont, nil)
        lvgl.img_set_src(img_bottom, "/lua/src_bottom_page.png")
    
    ```

    

  - 控件包含屏幕左上角的信号等图标( img 控件)，png 图片自带透明通道，不必担心其周围部分。

    ![image-20230908120540528](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811925.png)

    ```lua
        -- 创建信号值的图片
        img_sign = lvgl.img_create(page_scr_bottom.img_bottom, nil)
        lvgl.obj_set_hidden(img_sign, true)
    
        lvgl.obj_set_click(img_sign, true) -- 设置可点击
        lvgl.obj_set_click(img_4G, true)
        lvgl.obj_set_click(img_net, true)
    
    	-- 测试信号值，1,5 实际使用时再采用rssi rsrp 等表示信号强弱
        for i = 1, 5 do
            if i == 1 then
                -- lvgl.img_set_src(img_sign, "/lua/_signal5.png")
                lvgl.img_set_src(img_sign, "/lua/_signal1.png")
    
                -- 如果i == 1 就不显示联网的图标
            elseif i == 2 then
                lvgl.img_set_src(img_sign, "/lua/_signal2.png")
            elseif i == 3 then
                lvgl.img_set_src(img_sign, "/lua/_signal3.png")
            elseif i == 4 then
                lvgl.img_set_src(img_sign, "/lua/_signal4.png")
            elseif i == 5 then
                lvgl.img_set_src(img_sign, "/lua/_signal5.png")
                -- 表示无信号
            end
            lvgl.obj_align(img_sign, nil, lvgl.ALIGN_IN_TOP_LEFT, 10, 13)
            sys.wait(500)
            -- lvgl.obj_set_pos(img_sign, 20, 20)
        end
    
    ```

  - label 标签控件和顶部bar 进度条控件。

    ![image-20230908141210898](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811926.png)

    当使用传感器的环境监测界面作为初始页面，应当把光标停留在该处。

    ```lua
        -- 上方页面栏
        env1_Monitor = lvgl.label_create(img_bottom, nil)
        env1_QR = lvgl.label_create(img_bottom, nil)
        env1_Info = lvgl.label_create(img_bottom, nil)
    
        -- 重新着色
        lvgl.label_set_recolor(env1_Monitor, true)
        lvgl.label_set_recolor(env1_QR, true)
        lvgl.label_set_recolor(env1_Info, true)
    
        -- 修改label 文本
        lvgl.label_set_text(env1_Monitor, "#ACADAF 环境监测#")
        lvgl.label_set_text(env1_QR, "#ACADAF 管理#")
        lvgl.label_set_text(env1_Info, "#ACADAF 版权#")
    
        -- 上方提示的进度条 (默认在环境监测页面)
        top_bar = lvgl.bar_create(page_scr_bottom.img_bottom, nil)
        lvgl.obj_set_hidden(top_bar, true) -- 隐藏下标控件(初始完毕之后再开启)
        lvgl.obj_set_size(top_bar, 66, 4);
        lvgl.obj_align(top_bar, nil, lvgl.ALIGN_IN_TOP_LEFT, 100, 29)
        lvgl.bar_set_range(top_bar, 0, 100) -- 进度条取值范围
        lvgl.bar_set_value(top_bar, 100, lvgl.ANIM_OFF)    -- 进度条拉满。
    
        lvgl.obj_align(env1_Monitor, nil, lvgl.ALIGN_IN_TOP_LEFT, 90, 10) --初始位置
        lvgl.obj_align(env1_QR, env1_Monitor, nil, 80, 0)    -- 相对位置 
        lvgl.obj_align(env1_Info, env1_QR, nil, 65, 0)   -- 相对位置
    
    ```

  

- page_env1.lua

  &emsp;&emsp;此页面为环境监测页面，具体逻辑：创建多个 label 控件作为文本框，bar 控件依附于label 控件的相对位置以下，从而之后修改页面时只需要移动基准的对象即可。

  &emsp;&emsp;由于第二页的传感器页面和该页面类似，因此创建对象的时候也把第二页的同时创建了

  页面的步骤：

  1. 创建容器( 切屏时隐藏，要显示的时候打开 )。，该容器父类是`page_scr_bottom.img_bottom`
  2. 设置容器背景+ 边框全为透明。
  3. 该页面所有控件都以 该页面为父类对象，使得隐藏父类容器，子类对象全部隐藏。

  

  ![image-20230908142145025](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811927.png)

  ```lua
  
  -- 所需要创建的对象都在这里 (就不完全粘贴了，)
  nv1_elem_t = {
      {
          valve_open,
          "阀门开度",
          sensor_val.sensor_val.valve_open, -- 获取全局变量的阀门开度值
      },
      {
          air_tp,
          "空气温度",
          sensor_val.sensor_val.air_tp,
      },
      。。。。
  }
  
  -- 进度条
  -- 形参1 是底图的控件，形参2 则是作为基准对象(其实也可以只传位置)，形参3 是比率
  local function envInfo_bar_create(obj_parent, obj, rate)
      local bar = lvgl.bar_create(obj_parent, nil)
      lvgl.obj_set_size(bar, 130, 7)
      lvgl.obj_set_pos(bar, 90, 150)
  
      lvgl.obj_align(bar, obj, nil, -10, 14)
      -- 设置进度条的最大值和最小值  -- 设置该值为图像选择页面
      -- 范围0-100
      lvgl.bar_set_range(bar, 0, 100)
  
      -- 设置加载完成时间
      lvgl.bar_set_anim_time(bar, 1500)
      -- 设置加载到的值  该函数可以设置直接显示或者有动态效果
      -- lvgl.ANIM_OFF lvgl.ANIM_ON 这两个值来决定是否显示设置值的一个中间效果
      lvgl.bar_set_value(bar, rate, lvgl.ANIM_ON)
  
      return bar
  end
  
  -- 创建label 文本信息
  function add_label(obj_parent, env1Info_t, pos)
      -- 设置比率，因为进度条显示的环境温度范围未必是 0-100 ℃
      local min, max, val
      if pos == 1 then
          -- 阀门开度1 - 100
          max = 100
          min = 0
      elseif pos == 2 then
          -- 空气温度 0-45
          max = 40
          min = -10
      elseif pos == 3 then
          -- 瞬时流量
          max = 1000
          min = 0
      elseif pos == 4 then
          -- 土壤温度
          max = 45
          min = -10
      elseif pos == 5 then
          -- 水温
          max = 100
          min = -4
      elseif pos == 6 then
          -- 湿度
          max = 100
          min = 0
      elseif pos == 7 then
          -- 总流量
          max = 100
          min = 0
      elseif pos == 8 then
          -- 土壤湿度
          max = 50
          min = 10
      elseif pos == 9 then
          -- 土壤电离
          max = 100
          min = 0
      elseif pos == 10 then
          -- 土壤EC
          max = 100
          min = 0
      elseif pos == 11 then
          -- 水压1
          max = 1000
          min = 0
      elseif pos == 12 then
          -- VOC
          max = 100
          min = 0
      elseif pos == 13 then
          -- 氮磷钾含量
          max = 50
          min = 0
      elseif pos == 14 then
          --土壤PH
          max = 9
          min = 5
      elseif pos == 15 then
          -- 水压2
          max = 1000
          min = 0
      elseif pos == 16 then
          -- CO2浓度
          max = 40
          min = 0
      end
  
      -- 根据table 命名添加对象
      env1Info_t[1] = lvgl.label_create(obj_parent, nil)
  
      lvgl.label_set_recolor(env1Info_t[1], true)
  
      -- =============设置文本左对齐===============
      -- --设置文本左方向对齐。(便于后续以首行为基准)
      lvgl.label_set_long_mode(env1Info_t[1], lvgl.LABEL_LONG_BREAK) -- 必须得设置长文本才能自动对齐
      lvgl.label_set_align(env1Info_t[1], lvgl.LABEL_ALIGN_LEFT)
      lvgl.obj_set_width(env1Info_t[1], 150)
      lvgl.obj_set_height(env1Info_t[1], 50)
      -- =========================================
  
      local ins_text = "#ACADAF " .. env1Info_t[2] .. "# #FFFFFF " .. env1Info_t[3] .. " %#"
      lvgl.label_set_text(env1Info_t[1], ins_text)
  
      if pos ~= 1 and pos ~= 5 and pos ~= 9 and pos ~= 13 then
          pos = pos - 1
          lvgl.obj_align(env1Info_t[1], env1_elem_t[pos][1], nil, 0, 35)
      elseif pos == 1 or pos == 9 then
          -- 显示具体数值
          lvgl.obj_set_pos(env1Info_t[1], 21, 85) -- 屏幕改变之后修改该绝对位置的值
      elseif pos == 5 or pos == 13 then           -- 如果是水温部分，则向后平移
          lvgl.obj_set_pos(env1Info_t[1], 171, 85)
      end
      
      -- 计算传感器数值的占比 换算成百分比进去
      val = env1Info_t[3]
      local rate = (100 * (val - min)) / (max - min)
      envInfo_bar_create(obj_parent, env1Info_t[1], rate) -- 添加进度条控件
  end
  
  ```

  ```lua
  -- 环境监测页面1 的入口函数
  function page_env1()
      log.info("========= page_env1 =========")
  
      -- 创建容器
      -- 创建容器统一管理页面的元素，目的：同时隐藏所有元素，切屏会更丝滑
      env1_cont = lvgl.cont_create(page_scr_bottom.img_bottom, nil); -- 使用容器作为对象
      lvgl.obj_set_hidden(env1_cont, false)                          -- 默认情况隐藏容器
      lvgl.obj_set_auto_realign(env1_cont, true);
  
      -- 设置屏幕的分辨率
      lvgl.obj_set_width(env1_cont, 320)
      lvgl.obj_set_height(env1_cont, 280)
      -- 设置容器位置
      -- 容器上层部分需要添加bar 来进行管理用于移动光标
  
      local s = lvgl.style_t()
      lvgl.style_init(s)
      lvgl.style_set_bg_opa(s, lvgl.STATE_DEFAULT, 0)        -- 修改容器的背景为透明
      lvgl.style_set_border_opa(s, lvgl.STATE_DEFAULT, 0)    -- 消除容器的边框效果
      lvgl.obj_add_style(env1_cont, lvgl.LABEL_PART_MAIN, s) -- 添加style
  
      -- 第一个形参是父类的形参
      add_label(env1_cont, env1_elem_t[1], 1)
      add_label(env1_cont, env1_elem_t[2], 2)
      add_label(env1_cont, env1_elem_t[3], 3)
      add_label(env1_cont, env1_elem_t[4], 4)
      add_label(env1_cont, env1_elem_t[5], 5)
      add_label(env1_cont, env1_elem_t[6], 6)
      add_label(env1_cont, env1_elem_t[7], 7)
      add_label(env1_cont, env1_elem_t[8], 8)
  
      log.info("========= page_env1 END=========")
  end
  
  ```

- page_QR.lua 

  &emsp;&emsp;该页面主要是二维码和 几行换行的字组成。( 子容器和上述过程一样 )

  问题点: 整体长度重新着色的字符是不会自动换行的，如果分段着色，需要设置特定文本长度`lvgl.obj_set_width(tips_text1, 100)`，但是会出现其它意料之外的问题，因此还是采用了 3个label 控件，放在3行中

  ```lua
  -- 文本
      -- 2. 文本控件添加提示信息  ---由于着色之后需要设置换行存在问题，出此下策(后续再改进)
      local tips_text1 = lvgl.label_create(qr_cont)
      local tips_text2 = lvgl.label_create(qr_cont)
      local tips_text3 = lvgl.label_create(qr_cont)
  
      lvgl.label_set_long_mode(tips_text1, lvgl.LABEL_LONG_BREAK) -- 可对齐的文本
      lvgl.label_set_align(tips_text1, lvgl.LABEL_ALIGN_LEFT)
      lvgl.label_set_recolor(tips_text1, true)                    -- 重新着色
      lvgl.label_set_text(tips_text1, "#FFFFFF 使用微信扫描#")
      lvgl.obj_set_width(tips_text1, 100)
      lvgl.obj_align(tips_text1, nil, lvgl.ALIGN_IN_TOP_LEFT, 40, 120) -- 相对位置
  
      lvgl.label_set_long_mode(tips_text2, lvgl.LABEL_LONG_BREAK)      -- 可对齐的文本
      lvgl.label_set_align(tips_text2, lvgl.LABEL_ALIGN_LEFT)
      lvgl.label_set_recolor(tips_text2, true)                         -- 重新着色
      lvgl.label_set_text(tips_text2, "#FFFFFF 右侧二维码以#")
      lvgl.obj_set_width(tips_text2, 100)
      lvgl.obj_align(tips_text2, tips_text1, nil, 0, 18)
  
      lvgl.label_set_long_mode(tips_text3, lvgl.LABEL_LONG_BREAK) -- 可对齐的文本
      lvgl.label_set_align(tips_text3, lvgl.LABEL_ALIGN_LEFT)
      lvgl.label_set_recolor(tips_text3, true)                    -- 重新着色
      lvgl.label_set_text(tips_text3, "#FFFFFF 管理设备#")
      lvgl.obj_set_width(tips_text3, 100)
      lvgl.obj_align(tips_text3, tips_text2, nil, 0, 18)
  
  -- 二维码控件
      -- 3. 二维码控件，生成二维码图片
      qrcode = lvgl.qrcode_create(qr_cont, nil)
      lvgl.qrcode_set_txt(qrcode, "https://dearl.top")
      lvgl.obj_set_size(qrcode, 120, 120)
      lvgl.obj_align(qrcode, nil, lvgl.ALIGN_IN_TOP_LEFT, 180, 100)
  ```

  

- 控制页面( 页面切换，隐藏控件 )： `eventTouch.lua`

  作用: 按键触发或者屏幕点击触发的页面

  ```lua
  -- 页面切换定位
  local page_loc = 1
  local function page_x(x)
      if x == 1 then
          lvgl.label_set_text(page_scr_bottom.env1_Monitor, "#FFFFFF 环境监测#")
          lvgl.label_set_text(page_scr_bottom.env1_QR, "#ACADAF 管理#")
          lvgl.label_set_text(page_scr_bottom.env1_Info, "#ACADAF 版权#")
  
          lvgl.obj_set_size(page_scr_bottom.top_bar, 64, 4);
          lvgl.obj_align(page_scr_bottom.top_bar, page_scr_bottom.env1_Monitor, nil, 0, 12)
  
          lvgl.obj_set_hidden(page_env1.env1_cont, false)
          lvgl.obj_set_hidden(page_env2.env2_cont, true)
          lvgl.obj_set_hidden(page_QR.qr_cont, true)
          lvgl.obj_set_hidden(page_info.info_cont, true)
      elseif x == 2 then
          lvgl.label_set_text(page_scr_bottom.env1_Monitor, "#FFFFFF 环境监测#")
          lvgl.label_set_text(page_scr_bottom.env1_QR, "#ACADAF 管理#")
          lvgl.label_set_text(page_scr_bottom.env1_Info, "#ACADAF 版权#")
          -- ----
          -- 添加页面上方的进度条控件
          -- 进度条修改位置和大小
          lvgl.obj_set_size(page_scr_bottom.top_bar, 64, 4);
          lvgl.obj_align(page_scr_bottom.top_bar, page_scr_bottom.env1_Monitor, nil, 0, 12)
  
          lvgl.obj_set_hidden(page_env1.env1_cont, true)
          lvgl.obj_set_hidden(page_env2.env2_cont, false)
          lvgl.obj_set_hidden(page_QR.qr_cont, true)
          lvgl.obj_set_hidden(page_info.info_cont, true)
      elseif x == 3 then
          -- 1. 重新设定 page_scr_bottom 的文本颜色
          lvgl.label_set_text(page_scr_bottom.env1_Monitor, "#ACADAF 环境监测#")
          lvgl.label_set_text(page_scr_bottom.env1_QR, "#FFFFFF 管理#")
          lvgl.label_set_text(page_scr_bottom.env1_Info, "#ACADAF 版权#")
  
          -- 上方提示的进度条
          lvgl.obj_set_size(page_scr_bottom.top_bar, 33, 4);
          lvgl.obj_align(page_scr_bottom.top_bar, page_scr_bottom.env1_QR, nil, 0, 12)
  
          -- ----------- 设置动画效果//TODO: -------------
          -- 代码报错
          -- anim = lvgl.anim_t()
          -- lvgl.anim_set_values(anim, -lvgl.obj_get_height(btn), lvgl.obj_get_y(btn), lvgl.ANIM_PATH_OVERSHOOT)
          -- lvgl.anim_set_time(anim, 300, -2000)
          -- lvgl.anim_set_repeat(anim, 500)
          -- lvgl.anim_set_playback(anim, 500)
          -- lvgl.anim_set_exec_cb(anim, btn, set_y)
          -- lvgl.anim_create(anim)
          -- --------- 设置动画效果 END -------------
  
          lvgl.obj_set_hidden(page_env1.env1_cont, true)
          lvgl.obj_set_hidden(page_env2.env2_cont, true)
          lvgl.obj_set_hidden(page_QR.qr_cont, false)
          lvgl.obj_set_hidden(page_info.info_cont, true)
      elseif x == 4 then
          lvgl.label_set_text(page_scr_bottom.env1_Monitor, "#ACADAF 环境监测#")
          lvgl.label_set_text(page_scr_bottom.env1_QR, "#ACADAF 管理#")
          lvgl.label_set_text(page_scr_bottom.env1_Info, "#FFFFFF 版权#")
  
          -- 上方提示的进度条
          lvgl.obj_set_size(page_scr_bottom.top_bar, 33, 4);
          lvgl.obj_align(page_scr_bottom.top_bar, page_scr_bottom.env1_Info, nil, 0, 12)
          lvgl.obj_set_hidden(page_env1.env1_cont, true)
          lvgl.obj_set_hidden(page_env2.env2_cont, true)
          lvgl.obj_set_hidden(page_QR.qr_cont, true)
          lvgl.obj_set_hidden(page_info.info_cont, false)
      end
  end
  
  -- 设置GPIO 按键来控制页面的切换
  -- param: event=0 时向左翻页，event=1 时向右翻页，
  function eventTouch(event)
      if event == 0 then
          page_loc = page_loc - 1
      elseif event == 1 then
          page_loc = page_loc + 1
      end
      if page_loc == 0 then
          page_loc = 5
      elseif page_loc == 5 then
          page_loc = 1
      end
      sys.timerStart(page_x, 20, page_loc)
  end
  
  pio.pin.setdebounce(20) -- 按键消抖20ms
  sys.taskInit(function()
      while true do       -- 等待屏幕页面初始化完毕
          sys.wait(200)
          if page_scr_bottom.scr_status == 1 then break end
      end
      local page_next = pins.setup(pio.P0_26, function(msg)
          if msg == cpu.INT_GPIO_POSEDGE then -- 上升沿中断
              log.info("Mbus_IO26上升沿中断！,向后翻页")
              eventTouch(1)
          end
      end, pio.PULLUP)
      local page_before = pins.setup(pio.P0_24, function(msg)
          if msg == cpu.INT_GPIO_POSEDGE then -- 上升沿中断
              log.info("Mbus_IO24上升沿中断！,向前翻页")
              eventTouch(0)
          end
      end, pio.PULLUP)
  end)
  
  ```

- LVGL 屏幕初始化部分: `DispMag.lua`

  ```lua
  module(..., package.seeall)
  require "sys"
  require "misc"
  require "sim"
  require "pins"
  require "common"
  require "utils"
  -- 屏幕驱动
  require "color_lcd_spi_st7789"
  -- -- -- 按键处理页面( 跳转用 )
  require "eventTouch"
  
  -- 页面底图
  require "page_scr_bottom"
  -- -- 传感器数据页面
  require "page_env1"
  require "page_env2"
  -- -- 二维码页面部分
  require "page_QR"
  -- -- 公司信息页面
  require "page_info"
  
  -- 屏幕电源
  local DispCtrPin = pins.setup(11, 0)
  DispCtrPin(1)
  
  -- 加载主界面模块
  sys.taskInit(function()
      sys.wait(3000)
      lvgl.init()
      log.warn("Disp", "start")
  
      -- 设置整体样式
      page_scr_bottom.img_bottom_fnc()                    -- 加载底图
      lvgl.obj_set_hidden(page_scr_bottom.top_bar, false) -- 取消隐藏的bar 控件
      page_env1.page_env1()
      page_env2.page_env2()
      page_QR.page_QR()
      page_info.page_info()
  end)
  
  ```

  

##### - 遇到的问题点，与解决过程

> 图片作为底图，传递给lvgl

- 应该使用"/lua/src_img.png" 同时，将src_img.png 添加到合宙烧录工具的路径中

  1. 当时我才用绝对路径，或者相对路径的形式，但是发现屏幕中并不能显示图像。

     ![image-20230908151433882](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811928.png)

  2. 后来转念一想，固件是烧录到模块中的，但是你的图片还是在电脑上，应当把图片 和固件一起烧录到模块，然后使用lua 专门的路径来获取图像信息。

- 层级关系，字体出现周围白色边框，当文本先加载之后再加载底图，就会出现下面情况。

  ```lua
  sys.timerStart(function()
      -- lvgl初始化放在整体上面
      lvgl.init()
      page_scr_bottom.img_bottom_fnc()  -- 页面切换
  end, 5000)   -- 5s后开启图像页面
  
  sys.timerStart(page_env1,8000)  -- 8s后，开启文本再添加到img 控件上，
  
  --[[
  	这里如果是8s 就会导致图片出现下图1的情况。
  	如果改为7s 或者更低 就不会出现问题，就如图2的正常现象 (但时间要保证先执行img 生成图片控件)
  	sys.timerStart(page_env1,7000)
  ]]
  ```

  ![image-20230908154136374](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811929.png)

  ![image-20230908154146873](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811930.png)

  

> 位置相关函数

- 有至少三种类型的函数来确定位置。

  1. 对象级别 `lvgl.obj_set_pos(img_sign, 20, 20)`

     但是使用该位置函数会存在弊端，有时候屏幕更换了底图，或者有其它动作的时候就会导致位置发生偏移，又要重新修改位置。

  2. 对象级别相对位置 `lvgl.obj_align(img_sign, nil, lvgl.ALIGN_IN_TOP_LEFT, 10, 13)`

     该案例函数，是以屏幕左上方为坐标起点移动的。 

     当然 **形参2 也可以是父类对象**，根据父类对象来确定位置，例：`lvgl.obj_align(env1_QR, env1_Monitor, nil, 80, 0) `

  3. 控件级别 ``  但是不常用，肯定有这个函数，但是我忘了函数名了，当时踩了好大一个坑。

  

> 看门狗重启问题：1. 驱动文件启动部分需要调整为LVGL。2. lvgl.init( ) 该函数要在你调用任何lvgl 库函数之前执行

- 驱动文件部分 

  该部分需要调整总线接口为 `bus = lvgl.BUS_SPI4LINE`，而不是disp 函数接口。(猜测这部分和底层封装的SPI 协议做过lvgl 的适配 )

  ![image-20230908154657402](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811931.png)

  ![image-20230908154718198](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811932.png)

- lvgl.init( ) 该函数要在你调用任何lvgl 库函数之前执行( 最好上电即执行 )。否则会报错。

  ![image-20230908155311666](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811933.png)

  

> 屏幕触摸模拟事件回调函数异常。

&emsp;&emsp;现象：由于屏幕没有触摸屏功能，因此使用模拟按下的函数，每隔一段时间就执行模拟函数，但是屏幕切换，切过去之后又切回来。。。

- 查找原因为：删除控件也会导致回调函数的启动。后续改为隐藏控件来实现。

  ```lua
  -- 下方这个删除控件的函数会导致 obj_set_event_cb(toPage1_btn, event_handler) 的event_handler 回调函数被触发
      if page_env1.toPage2_btn then
          lvgl.obj_clean(page_env1.toPage2_btn)
          lvgl.obj_del(page_env1.toPage2_btn)
      end
  --[[
  	经过排查：发现是lvgl.obj_del(page_env1.toPage2_btn) 函数触发了事件回调
  ]]
  
  -- 但是该控件要么设置隐藏，要么删除。但是创建必须最多创建一次
  ```

  

> 控件全部添加到table 中，切换页面的时候遍历table 所有控件并删除 -> 改为 cont 容器管理，切换页面只是隐藏cont 容器( 所有子控件都会被隐藏 )

- 使用隐藏的方式更便于 容器的管理。下面浅谈其优劣

  ```bash
  # 删除控件法
     1.创建一个全局的table 变量，在每一个页面中都将其对象放到table 中，切换页面时遍历并删除所有控件。
     2.但是该方案会导致图像切换时控件删除不统一 (虽然也看不出来)。
     3.而且会导致不同页面难以管理
     4.有点是节约控件。
  # 容器隐藏法
     1.创建容器，该容器的父类为 page_scr_bottom.img_bottom 控件。
     2.将所有该页面的控件都设置其父类控件为 本页面的容器，且除了开屏第一个页面以外，其余所有页面容器默认隐藏。
     3.在按键触发脚本文件中，切换到某个页面就打开它的容器，并隐藏其它所有页面的容器。
  ```

  

TODO:// 定时传感器数据刷新。 这部分暂时还没有具体的要求，因为后续接传感器，只要收到传感器数据就执行一个刷新的操作

目前控制方案，采用按键翻页。



##### - 新东西

好像都是新东西，又get 到一项新技能，后续我试着在stm32F407 开发板上移植C 语言的LVGL 。



#### 4G 采集器适配新设备



##### - RS485 三线控制

问题点: 485 控制引脚设置之后 依旧只能要么收要么发，

现象: 

1. 第一版的硬件485 无法收发的原因是MOS 管焊接反了，后来飞线之后设置`uart.set_rs485_oe(2, pio.P0_22, 1, 1, 1) `，之后可以正常类似全双工一样收发。

2. 后来第二版出来只能半双工了，代码应当没问题，控制引脚拉高就是接收，拉低就为发送( 但是不能使用`uart.set_rs485_oe(2, pio.P0_22, 1, 1, 1)`,实现自动控制 )。

3. 接收数据的时候只有每次在**数据发送的** 时候才 **有可能收得到串口数据**。对照如下：

   i. 当间隔时间很长的时候，接收到数据的概率就很低。![image-20230911171728621](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811934.png)

   ii. 当间隔变短，接收数据概率就很高。

   ![image-20230911171858319](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811935.png)

4. 示波器抓取波形显示：

   解释：**每500 ms 采集器通过485向串口发送一次数据**，也就是`uart.write(2,"uart2send")`,**示波器显示每500ms 时三线485 控制引脚电平拉高到`2.2V` 一次**，意味着电脑要给设备发送消息，必须踩到这一点才能发出！！！

   ASK: 一般除了`uart.write( )`函数以外，别的时候都是低电平为什么不发。。。因为uart.write( ) 被执行的时候电平才拉高。。 你问我为什么不再代码修改低电平接收，那么他为什么能接收呢？

   ![image-20230911172758777](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811936.png)

   ![image-20230911172828785](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811937.png)



解决:

- 定位原因：

  1. `uart.on(2, "receive", function()   ... end)` 中函数我放了一个`u_send( )`

     该原因只是表层的send 发送导致 read 不到。实际上跟它没关系，但是它帮助我找到最终原因

  2. 后来发现是`uart.set_rs485_oe(2, pio.P0_28, 0, 1, 1)` 函数 **第三个参数** 是配置高低电平发送与接收！！！！

- 解决:

  ```lua
  --多看文档!!!! 多看文档！！！！ 多看文档❗❗❗❗
  
  uart.set_rs485_oe(id, io[, level] [, timeUs] [, mode])
  --[[485转向控制
  
  参数	类型	释义	取值
  id	number	串口号	1(UART1),2(UART2),3(UART3),0x81(USB)
  io	number	GPIO值	pio.Pxx
  level	number	输出使能电平有效值，默认1，配置为1时表示高电平发送，配置为0时表示低电平发送	1/0
  timeUs	number	485 oe转向延迟时间,单位US，缺省时为0延迟5个当前波特率的时钟时间	
  mode	number	485 oe转向模式选择，默认0。 配置为1时表示使用定时器控制，可以使转向时间小于100us(timeUs最小值为1)	1/0
    ]]
  
  ```

最后实现效果: 能发，也能收

![image-20230911180605705](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309111811938.png)

- 思考: 合宙是封装过三线485 的收发机制的，那么如果是stm32 的芯片呢？不确定中断是否能来得及将电平转向。



##### - MBUS 总线收发数据调试

- 对于MBUS 总线的理解
  1. MBUS 为2根线，标准电压差为36V/24V 但可以使用其它的电压，，疑似在信号传输的时候不分正反。但是分主从。
  2. 可以将MBUS 理解为一种特殊的普通串口，类似于485 使用差分进行数据收发，他们都为半双工，
  3. MBUS 和modbus 不一样，MBUS 我理解为作用于物理层和传输层的一种物理层协议，modbus 则是应用层对数据经行处理的协议。**MBUS 是上层协议的物理上的一种载体**，
- MBUS 必须分主从，Master 携带电压。可以为非电表类的其它表进行供电和数据传输

- feature: 

  [参考地址](https://wiki.dzsc.com/8349.html)

  　　1. 高速稳定的通讯速率，在4.8kb/s 的通讯速率时可达到2.4Km的可靠通讯距离；

  　　2. 在4.8kb/s、2.4Km的可靠通讯距离时，可有多达500个接点的容量；

  　　3. 允许串型、星型、交叉等任意接线分支的布线方式；

  　　4. 极低的静态功耗，低达200uA；

  　　5. 使用普通双绞线，无极性二线制安装接线；

  　　6. 隔离通讯设备可保证在遭雷击时可靠工作，

  　　7. 恒流的电流环通讯方式，抗干扰性强；

  　　8. 具有设备自动登录等功能，可容纳多种设备，预留多种通讯协议，扩展方便；

  　　9. 通过总线可向从设备提供200mA电流；



##### - I2C 耳机口插入识别检测的问题

电压不能默认拉高。。。。( 所以不要懒，该动手写的测试代码一行都不要少 )





##### - 看旧版采集器代码的理解

- 对于整体代码框架

  ![image-20230912093314572](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309131145242.png)

  如上图，之前设备使用的Lora 和MBUS/485 对应的Uart 的收发。层次结构: MQTT  或者OneNet_MQTT 收到服务器下发的命令时调用Cmd.lua 中`RevCMD()` 函数来对命令进行解析，对于定义的**不同的数据格式**，对于接收到的MQTT 回调的`payload` 字段可以区分是 `UPDATE`、`GETPRODINFO`、`SET`、`RESTART`.... 等命令，然后再解析成具体传感器(或是从传感器发送数据给MQTT  )

- MQTT 服务器连接，使用了2个MQTT 服务器。一个移动的OneNet 平台，另一个则是公司自建的服务器。
- MQTT 的$dp 订阅，`$dp` 是一个特殊的主题，通常用于设备与云端进行通信。通常情况，设备会发布或者订阅 `$dp` 主题下的消息，以便特定执行设备的配置、控制或数据交换操作



#### 总结

&emsp;&emsp;对于LVGL 虽然一开始有一种层次结构的想法，但是在具体实施过程中存在一部分问题，其中包括函数库不熟悉，底层应该使用图片控件？或者屏幕、容器控件？最终选择了容器控件，图片控件放于底图会导致各类属性不好统一管理，而且最初的做法是切换页面采用控件删除的方式，也就是创建全局的table ( C语言称之为数组，但是比数组高级 )，并在每次创建控件时将其放入table ，每次切换别的页面时删除所有控件。后续将每一个页面都创建另外一个子容器，切换页面仅仅 隐藏/显示 容器，方便管理。

&emsp;&emsp;LVGL 部分页面切换按键的回调函数问题， 当删除带有回调部分的控件时会触发回调的事件，产生现象: 切换过去后马上切回来。后续处理，将带回调的控件隐藏即可，但是目前使用的方案按键切换。





- temp





