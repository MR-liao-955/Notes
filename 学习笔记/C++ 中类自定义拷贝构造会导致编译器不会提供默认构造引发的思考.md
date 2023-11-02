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



































