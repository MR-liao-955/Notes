case1:
	DCU 未收到 MSN、USN 和制造商代码的库存详细信息时，DCU 应丢弃表的加密密钥并在 ACK 文件中共享标志位 "03"
case2:
	DCU 已收到 MSN、USN 和制造商代码的库存详细信息时( 说人话就是数据库中有 密钥之类的东西 )。
	DCU 就会和仪表进行通信，就会生成对应的 ACK 文件，其中有 "状态码" 来表明是否能连接到表