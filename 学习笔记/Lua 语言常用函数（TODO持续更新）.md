### Lua 语言常用函数（TODO:持续更新）

官方Lua API 库：[http://www.lua.org/manual/5.3/](http://www.lua.org/manual/5.3/)

合宙 API 库：[https://doc.openluat.com/wiki/31?wiki_page_id=3920#packpack_format_val1_val2_valn__73](https://doc.openluat.com/wiki/31?wiki_page_id=3920#packpack_format_val1_val2_valn__73)



#### string 库函数

- string.byte( )

  ```lua
  
  
  
  
  ```

- string.char( )

  ```lua
  
  
  
  
  ```

- string.format( )

  ```lua
  -- 用于格式化数据
  
  
  
  ```

- string.find( )



- string.gsub( )





#### package

- pack.pack( )

  ```lua
  
  
  
  ```

- pack.unpack( )

  ```lua
  -- 参考SHT30 通过I2C 发送命令之后 的recv() 接收到的函数。 此处的data 为string 类型
  local data = i2c.recv(i2c_id, SHT30_ADDR, 6)
  local _, tp, crcTp, rh, crcRh = pack.unpack(data, ">HhHh")
  -- 上方的 ">HhHh" 对应 大端(不使用所以用_ 表示) ,tp为unsigned short,crcTp为short,...
  --[[
  pack.unpack( string, format, init)
  format 为string类型,
  pack.unpack( string, format, init)
  格式化符号 ‘<’:设为小端编码 ‘>’:设为大端编码 ‘=’:大小端遵循本地设置 
  ‘z’:空字符串 ‘p’:byte字符串 ‘P’:word字符串 ‘a’:size_t字符串 ‘A’:指定长度字符串 
  ‘f’:float ‘d’:double ‘n’:Lua number ‘c’:char ‘b’:byte = unsigned char 
  ‘h’:short ‘H’:unsigned short ‘i’:int ‘I’:unsigned int ‘l’:long ‘L’:unsigned long
  ]]
  
  返回类型 
  1.int 第一个为字符串标记位置
  2.可能有N个返回值，根据 format 来定，如上方的">HhHh" 就有5个返回值
  
  local _,a = pack.unpack(x,">h") --解包成short (2字节)
  ```

  







#### 校验函数





#### MQTT 函数







#### 外设( PWM 、GPIO、UART、I2C、 )

















