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
// 此时会打印 -- printf("world\n"); 该行代码希望放在判断里面，因此这里未能实现想要的操作
/* 
world
-------
*/

```

- 根据上面的情况，你也许会想到 给**宏定义加一个括号{ }**

  ```c
  // 加一个括号的情形1
  #define log() {printf("hello \n");printf("world\n");}
  if(0)
      log()
  printf("-------");
  system("pause");
  
  ```

  没错，这样确实能正常判断。**但是我的if 后面加一个else** 阁下应当怎么处理？

  ![image-20230921113056603](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309211200981.png)
  
  ```c
  // 如果判断的是if else 
  #define log() {printf("hello \n");printf("world\n");}
  if(0)
  {
      log();  // 这里会报错，展开因为一半写函数都会在后面添加; 因此这类容易报错
      /*
      展开为:
      if(0)
      {
          {
          printf("hello \n");
          printf("world\n");
          };  -- 这里不能有分号
      }
      */
  }
  else
  {
      printf("-------");
  }
  system("pause");
  ```
  
  去掉分号，就能正常编译，但是不符合代码阅读规范
  
  ![image-20230921113242257](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309211200982.png)

- 解决办法: `do{}while(0)`  黑科技。

  ![image-20230921113841851](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202309211200983.png)



#### 替代 goto

- 使用goto 的代码

  ```c++
  void fun(int a)
  {
     if(1 == a)
     {
         ...//todo
         goto exit;
     }
     if(2 == a)
     {
       ...//todo
       goto exit;
     }
  exit:
     ...//todo
     printf("a is error"\n);
  }
  ```

- do{...}while(0) 替代goto:   -- break 跳出循环，就类似于goto到需要执行的地方

  ```c++
  int fun(int a)
  {
     do{
         if(1 == a)
         {
           ...//todo
           break;
         }
         if(2 == a)
         {
           ...//todo
           break;
         }
     }while(0);
     ...//todo
     printf("a is error"\n);
  }
  ```

  

#### 定义单独函数块(可以避免变量重名冲突...)

```c++
int fun(int a)
{
   do{
       if(1 == a)
       {
         ...//todo
         break;
       }
       if(2 == a)
       {
         ...//todo
         break;
       }
   }while(0);
   ...//todo
   printf("a is error"\n);
}
```



#### 避免空宏的 warning

&emsp;&emsp;有的时候，程序为了不同的平台移植或者不同架构的限制，很多时候会先定义空宏，后续再根据实际的需要看是否定义具体内容。但是在编译的时候，这些空宏可能会给出warning，为了避免这样的warning，我们可以使用do{...}while(0)来定义空宏，这种情况不太常见，因为有很多编译器已经支持空宏。

```C++
1 //空宏
2 #define EMPTY_FUN
3 //增加do{...}while(0)来定义空宏
4 #define EMPTY_FUN do{}while(0) //避免了可能的编译warning
```



> reference

[参考地址](https://www.cnblogs.com/Sharemaker/p/17142670.html)

