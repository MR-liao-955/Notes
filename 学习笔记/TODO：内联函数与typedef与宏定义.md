### 内联函数 && typedef && macos

[toc]

#### inline 内联函数

> 原理

编译器在调用内联函数时，它会直接将函数体复制过去，而不是生成函数调用命令。因此可以减少函数调用的开销，但是会增大函数的体积。

> 何时何地使用？

- 频繁调用的函数

- 函数体比较小

  如果函数体内容较多，它会无形地导致代码膨胀，增加缓存的占用。

- 尽量写在头文件，且函数体也写进头文件，更直观。

  **关键字 `inline` 必须与函数定义体放在一起才能使函数成为内联，仅将 `inline` 放在函数声明前面不起任何作用。**

  如下风格的函数 Foo 不能成为内联函数：

  ```c++
      inline void Foo(int x, int y); // inline 仅与函数声明放在一起
      void Foo(int x, int y){}
  ```

  而如下风格的函数 Foo 则成为内联函数：

  ```c++
      void Foo(int x, int y);
  
      inline void Foo(int x, int y) {} // inline 与函数定义体放在一起
  ```

> 使用案例

- betaflight 源码中会经常调用的且对实时性要求较高的函数。

  ![image-20231229103408388](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202401021758258.png)

  

- 内联函数在 C++ 中的应用。

  我们在定义类中，往往会把成员变量定义成私有的`private` , 这样在读写的时候会给定一个成员接口。

  此时把 **读写成员函数定义成 内联函数** 的话，它的执行效率会好很多。

  ```c++
  class Person
  {
  private:
      int age;
  public:
      inline void setAge(){}
      inline int getAge()
      {
          return age;
      }
  }
  ```



> 注意事项

- 内联函数内不允许使用 循环语句、switch语句，( 这些语句可能不会被执行 )。

- 有些函数即使声明为内联函数也不一定会被编译器内联。

  比如: 虚函数、递归函数就不会被正常内联。

- inline 仅仅是对编译器的一个建议，最后是否要内联还得看编译器的想法，如果能在调用点展开，则会形成真正的内联。

- 内联函数一定要放在函数体的地方声明 (也没必要在引用的时候声明，用户**不关心这个函数接口是否为内联的**)

  ```c
  // 正确的有效声明
  inline int test(){
      return 0;
  }
  
  // 错误的声明 (编译器不认)
  inline int test();
  ```

  


>  内联函数和宏定义的区别：

- 内联函数的**本质还是函数，内联只是它的一个属性而已。**
- 内联函数会检查其参数类型、返回值之类的，用法更安全。
- 内联函数会在内存中生成实体，宏定义操作的则是 token，可以进行 token 的替换和连接的操作



##### `__containerof(encoder, rmt_dshot_esc_encoder_t, base); `

![image-20240227103951029](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080957120.png)

![image-20240227102657973](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080957121.png)

- 该知识点出处

  1. 结构体声明部分

     ```c
     // 结构体声明部分
     typedef struct
     {
         rmt_encoder_t base;   // 下方函数形参中 rmt_encoder_t *encoder 指明了该函数的基准
         rmt_encoder_t *bytes_encoder;
         rmt_encoder_t *copy_encoder;
         rmt_symbol_word_t dshot_delay_symbol;
         int state;
     } rmt_dshot_esc_encoder_t;
     
     ```

  2. __containerof( ) 宏定义函数调用部分

     ```c
     static size_t rmt_encode_dshot_esc(rmt_encoder_t *encoder, rmt_channel_handle_t channel,
                                        const void *primary_data, size_t data_size, rmt_encode_state_t *ret_state)
     {
         /*
         这行代码的目的是通过 base 成员的指针 encoder 获取整个 rmt_dshot_esc_encoder_t 结构体的指针。
         这种技巧常常用在数据结构设计中，特别是在嵌套结构体的情况下，以方便访问整个数据结构。
         */
         rmt_dshot_esc_encoder_t *dshot_encoder = __containerof(encoder, rmt_dshot_esc_encoder_t, base); // 获取到 rmt_dshot_esc_encoder_t 结构体指针
     }
     ```

  3. __containerof() 宏定义声明部分

     ```c
     /******* offsetof() ********/
     #ifndef offsetof
     #define offsetof(type, member) ((long) &((type *) 0)->member)
     #endif
     
     /******* __containerof() ********/
     #if CONFIG_IDF_TARGET_LINUX && !defined(__containerof)
     #define __containerof(ptr, type, member) ({         \
         const typeof( ((type *)0)->member ) *__mptr = (ptr); \
         (type *)( (char *)__mptr - offsetof(type,member) );})
     #endif
     ```

  

- 说明

  1. __containerof( ) 并不是 C 语言库中给定的函数，而是一种常见的技巧
  2. 原理：通过内层结构体某个 **内层成员指针**、**外层结构体类型( 名字 ) **和 **内层结构体成员** ，来获取外层结构体指针。
  3. 根据结构体定义，从偏移量来计算出外层结构体成员的寄存器地址。

- 分析

  个人理解：

  1. `rmt_encoder_t *encoder` &emsp;encoder 为 rmt_encoder_t 结构体指针，它本来就是 堆 或者 栈 中的对象，而 `rmt_dshot_esc_encoder_t` 与 `base` 仅仅是 结构体名称 和结构体成员，他们并没有在实际的 堆、栈、全局区 的内存中。
  2. 目的是使用 encoder 这个对象来 **获取 / 转化** 成为 `rmt_dshot_esc_encoder_t` 结构体对象。

  

##### `__typeof__()`

- [参考地址1](https://www.cnblogs.com/wjw-blog/p/8722183.html)、[参考博客2](https://hackmd.io/@sysprog/c-trick)
- `__typeof()` 、`__typeof__()` 是C语言的编译器特定扩展，因为标准C语言是不含这样的运算符的。标准C要求编译器用双下划线前缀扩展语言。（这也是为什么你不应该为自己的函数，变量加双下划线的原因）
- 灵活的获取参数类型，在程序员不知道类型的情况下，来定义新的类型。



#### typedef

[参考博客](https://www.cnblogs.com/pam-sh/p/15232940.html)

作用：给变量起别名、和struct 配合便于创建结构体、定义与平台无关的类型

- 和宏定义有点类似，但是存在一定的区别。

  1. typedef 会对定义的部分进行初步计算，然后再带入到调用的地方。

     ```c
     // #define用法例子：
     #define f(x) x*x
     int main()
     {
         int a=6, b=2, c;
         c=f(a) / f(b);
         printf("%d\n", c);
         return 0;
     }
     // 计算结果是 c = 36
     ```

  2. #define 宏定义不做类型检查，仅仅是机械的字符串替换，而 typedef 会稍微处理一下

  3. typedef 经常和 struct 搭配。

- typedef 配合 struct 使得创建结构体更加方便。

  1. 使用 typedef

     ```c
     typedef struct{
         int a;
         double b;
     }my_struct;
     my_struct struct1;  // 使用typedef 关键字定义结构体，之后创建结构体对象时可以省略 struct 关键字。
     ```

  2. 不使用 typedef

     ```c
     struct my_struct2{
       	int a;
         double b;
     };
     struct my_sturct2 struct2;  // 要多写一个 struct
     ```

- 注意：typedef 在语法上是一个存储类的关键字 ( auto, extern, static, register 等 )。使用时不要再额外添加存储类的关键字了，否则编译会报错。

  ~~`typedef static int INT2;`~~ 会导致编译失败， 提示: " 指定了一个以上的存储类 "

  ![image-20240229173624225](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080957122.png)

  上图就是用相同的名字定义新的类型名。可能是为了实现接口函数之类

#### macos 宏定义

[参考博客](https://www.jb51.net/article/276400.htm)

目前有内联函数之后，宏定义函数出现较少



> 宏定义的返回值  [宏定义返回值和可变参数参考博客](https://cstriker1407.info/blog/macro-return-values-and-variable-argument-macros/)

![image-20240229161029483](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202404080957123.png)

- 写法 `#define func(a) ({ a = a+5;a;})`

- 作用：。。。用到再说，我个人很少写宏定义函数的。



> 宏定义展开

- 可以配合 ``#ifndef MACOS_H` 、`#endif`来防止宏定义重名





> 宏定义使用 # ## 连接



> do{}while(0);







#### extern 关键字

- extern "C" 和 extern "C++" 函数声明