### C++ 中类自定义拷贝构造会导致编译器不会提供默认构造引发的思考

[toc]

**为何会想到这个问题？**

&emsp;&emsp;起因，这段时间又在继续学剩下的部分 C++ ( 最新的学习笔记先不整理，还在施工之中，而且剩下的 C++ 教程也不多了，目前进度是函数对象 ( 仿函数 )  )，同时翻了翻之前的笔记，看到了类这部分，然后读了读拷贝构造函数，因此思考到这个问题。

#### - 自定义拷贝构造函数，如果成员没被拷贝，也没提供默认无参构造，如果访问它会如何？

C++ 中类成员会默认提供一个无参构造，析构，拷贝构造，拷贝构造会

当拷贝构造函数并没有对所有值都进行拷贝，而没被拷贝的值没进行初始化之后，如果再外部没有调用它？那么会报错吗？还是有默认值？



回答: 栈中并不会执行数据清零，如果自定义了拷贝构造，而类的元素中没有默认赋值且拷贝构造也没给它拷贝，如果在外部调用的时候，可能会出现BUG，也可能不会，取决于之前那块内存中保存的什么。待验证！！



#### - 根据上方问题，如果显式定义了一个空无参构造，如果访问它会如何？

那么如果自定义了拷贝构造，然后再显式地定义一个空白无参构造函数，会出现什么有趣的现象？那么再访问那个未拷贝且未赋值的元素会出现什么情况？





#### - 阅读 SX1280 Lora 的部分代码遇到的语法问题，以前没见过，解释？

> 阅读代码时的节选

```c++
class SX12xxDriverCommon
{
public:
    typedef uint8_t rx_status;
    enum
    {
        SX12XX_RX_OK             = 0,
        SX12XX_RX_CRC_FAIL       = 1 << 0,
        SX12XX_RX_TIMEOUT        = 1 << 1,
        SX12XX_RX_SYNCWORD_ERROR = 1 << 2,
    };

    SX12xxDriverCommon():    /**  此处的 ':' 这个语法是什么含义？  **/
        RXdoneCallback(nullCallbackRx),
        TXdoneCallback(nullCallbackTx) {}

    static bool ICACHE_RAM_ATTR nullCallbackRx(rx_status) {return false;}
    static void ICACHE_RAM_ATTR nullCallbackTx() {}
    
    bool (*RXdoneCallback)(rx_status crcFail); //function pointer for callback
    void (*TXdoneCallback)(); //function pointer for callback
	/*
	...................... 省略后面函数 ......................
    */
}
```

```c++
class SX127xDriver: public SX12xxDriverCommon
{

public:
    static SX127xDriver *instance;

    ///////////Radio Variables////////
    bool headerExplMode;
    bool crcEnabled;

    //// Parameters ////
    uint16_t timeoutSymbols;
    ///////////////////////////////////

    ////////////////Configuration Functions/////////////
    SX127xDriver();
    /*
	...................... 省略后面函数 ......................
    */
}

SX127xDriver::SX127xDriver(): SX12xxDriverCommon()  /** 这部分貌似是拷贝函数的继承，那么它的机制是什么呢？从父类的拷贝函数中继承给它？ **/
{
  instance = this;
  // default values from datasheet
  currSyncWord = SX127X_SYNC_WORD;
  currBW =SX127x_BW_125_00_KHZ;
  currSF = SX127x_SF_7;
  currCR = SX127x_CR_4_5;
  currOpmode = SX127x_OPMODE_SLEEP;
  ModFSKorLoRa = SX127x_OPMODE_LORA;
  // Dummy default values which are overwritten during setup
  currPreambleLen = 0;
  PayloadLength = 8;
  currFreq = 0;
  headerExplMode = false;
  crcEnabled = false;
  lowFrequencyMode = SX1278_HIGH_FREQ;
  lastSuccessfulPacketRadio = SX12XX_Radio_1;
}

```

- 该问题是语法问题，chatGPT 给出解答如下：

  ```c++
  class SX12xxDriverCommon
  {
  public:
      SX12xxDriverCommon():    /**  此处的 ':' 解释  **/
          RXdoneCallback(nullCallbackRx),
          TXdoneCallback(nullCallbackTx) {}
  }
  ```

  1. 该部分 C++ 类中 `类名( ):` 是用来初始化类的对象的，对于 `RXdoneCallback(nullCallbackRx),TXdoneCallback(nullCallbackTx) {}` 该函数为回调函数指针。

  2. 先找到定义它的地方

     `bool (*RXdoneCallback)(rx_status crcFail);void (*TXdoneCallback)();`

     这种类型的定义是定义的函数指针 // function pointer for callback

  3. 语法说明: 

     `类名():` 这种语法和 `类名( ){}`相似，

     不同点：

     -  `类名():`  
       1. 只能进行成员变量的初始化。
       2. 语法：`MyClass() : x(0), y(0.0) {}` 每个成员之间使用 `,` 隔开。 
     - `类名( ){}`，可以在函数体内进行额外的处理。
     - 性能方面的注意事项，`类名():` 在性能方面上更有优势。

     ```c++
     class MyClass {  // chatGPT 给的解释
     public:
         int x;
         double y;
     /**
     说明: 下方两个 MyClass() 无参构造，只是为了比较说明用，实际不能写两个相同的函数
     */
         // 使用初始化列表初始化成员变量
         MyClass() : x(0), y(0.0) {
             // 这里只能进行成员变量的初始化
         }
     
         // 使用构造函数主体进行初始化
         MyClass() {
             x = 0;        // 初始化成员变量
             y = 0.0;
             // 在构造函数主体内可以执行更多操作
             if (x > 10) {
                 // 做一些额外处理
             }
         }
     };
     
     ```

- `SX127xDriver::SX127xDriver(): SX12xxDriverCommon(){ ... } ` 的解释

  &emsp;&emsp;改行代码指的是函数的继承，`SX127xDriver` 类中的 `SX127xDriver( )` 构造函数继承于 `SX12xxDriverCommon()` 的构造函数，而父类构造是一个指针。。这部分代码较为复杂，没仔细阅读。



- 在 RGB  灯带驱动库文件，再次遇到该语法

  ```c++
  /*
  Adafruit_NeoPixel 类中的有参构造函数Adafruit_NeoPixel(...)
  ':' 后面的是类成员的初始化。
  
  */
  Adafruit_NeoPixel::Adafruit_NeoPixel(uint16_t n, int16_t p, neoPixelType t)
      : begun(false), brightness(0), pixels(NULL), endTime(0) {
    updateType(t);
    updateLength(n);
    setPin(p);
  #if defined(ARDUINO_ARCH_RP2040)
    // Find a free SM on one of the PIO's
    sm = pio_claim_unused_sm(pio, false); // don't panic
    // Try pio1 if SM not found
    if (sm < 0) {
      pio = pio1;
      sm = pio_claim_unused_sm(pio, true); // panic if no SM is free
    }
    init = true;
  #endif
  }
  ```

  



#### - 虚析构 & 纯虚析构解决父类指针释放子类对象的问题，那么进行验证？

问题灵感：[自建博客，后续可能访问不了。](https://dearl.top/2022/11/22/%e5%a4%9a%e6%80%81/)

- 纯虚析构用于解决父类指针释放子类对象问题

  如果子类中没有堆中的数据，则不需要释放，就不需要写纯虚析构。
  
- 根据之前写的笔记部分，当父类纯虚虚构的时候，为什么一定要在外部实现父类的纯虚析构呢？如果没实现它为何会编译报错？



- 脑残问题: 在继承或者多态中，**父类类名**创建的**父类对象**有什么作用？

  能创建好之后修改指向吗？( 因为类是引用类型， )



- 虚函数指针 vfprt、虚函数表 vftable、虚基指针 vbptr





24.47

4.116  4.114  4.129   4.135  3.898  4.101



24.47

4.119   4.094    4.093    4.150    3.819    4.154



























