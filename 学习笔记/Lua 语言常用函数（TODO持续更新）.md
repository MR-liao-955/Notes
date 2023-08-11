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

  函数原型： `string.format(fmStr,...)`

  解释：`fmStr` 是格式化参数，`...`表示待格式化的数据

  &emsp;&emsp;一个指示由字符`%`加上一个字母组成，这些字母指定了如何格式化参数，例如`d`用于十进制数、`x`用于十六进制数、`o`用于八进制数、`f`用于浮点数、`s`用于字符串等。

  ```lua
  print(string.format("%.4f", 3.1415926))     -- 保留4位小数  
  print(string.format("%d %x %o", 31, 31, 31))-- 十进制数31转换成不同进制  
  d,m,y = 29,7,2015  
  print(string.format("%s %02d/%02d/%d", "today is:", d, m, y))  
  --控制输出2位数字，并在前面补0  
  
  -->输出  
  -- 3.1416  
  -- 31 1f 37  
  -- today is: 29/07/2015  
  
  
  ```

- string.tonumber( )

  函数原型：

  ```lua
  -- 这个函数感觉就是我自己写的 str_toNumb 函数
  
  
  ```

  









- string.find( )



- string.gsub( )



- recvBuf:toHex( )

  函数原型&ensp;`string.toHex(str,separator)`

  解释：将Lua 字符串转化成HEX 字符串：

  ```lua
  -- 如"123abc" 转化为 "313233616263"    根据ASCII 码来进行转化
  --[[
  	string.toHex("\1\2\3") -> "010203" 3
      string.toHex("123abc") -> "313233616263" 6
  ]]
  
  -- 形参：
  --	str, 输入字符串
  --	separator, 分割参数,默认"" 输出的 16进制字符串分隔符
  
  -- 返回值：
  --	hexstring, 16进制组成的串
  --	len, 输入的字符串长度！！
  ```

  



- x:fromHex( )     该函数在合宙的 utils 工具库函数中

  函数原型 &ensp;`string.fromHex(hex)`&ensp;

  解释：将HEX 字符串转化为Lua 字符串：

  ```lua
  -- 如"313233616263"  转化为 "123abc" 
  --[[
      string.fromHex("010203")       ->  "\1\2\3"
      string.fromHex("313233616263:) ->  "123abc"
  ]]
  
  -- 形参：
  --	hexstring, 16进制组成的串
  
  -- 返回值：
  --	charstring, 字符串组成的串
  --	len, 输出的字符串长度
      
  ```

  



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

  

#### sys 库函数

- sys.subscrib





- sys.publish





- 







#### 校验函数

- 关于取反

  > 归纳：
  >
  > &emsp;&emsp;对于16进制数而言，取反就是1111 1111 减去待处理数。
  >
  > &emsp;&emsp;也就是 ( 2^16 - 1 ) - number 

  ```lua
  -- 取反函数
  local function bnot(number, numBits)
      local maxVal = 2 ^ numBits - 1
      return maxVal - number
  end
  
  -- 对于取反的理解。以2^4 为例。
  --[[
  	案例1：
  	- 5 = 0101 
  	- 取反： 1010 = 10 = 1111 - 0101 = 15 - 5
  
  	案例2：
  	- 9 = 1001
  	- 取反： 0110 = 6 = 1111 - 1001 = 15 - 6
  ]]
  
  ```

- 





#### MQTT 函数







#### 外设( PWM 、GPIO、UART、I2C、 )

















