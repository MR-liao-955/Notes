### C 中结构体定义的函数指针，通过结构体成员来实现函数的调用

> 起因：

&emsp;&emsp;我在阅读 ESP32 中 RMT 外设驱动 Dshot 电调协议的时候遇到的 C 语言语法问题，这个结构体的用法未曾见过，它是在结构体中定义了一个 `函数指针`，指针后面括号了形参，这种写法之前没遇到过特此记录。

```c
struct rmt_encoder_t {
     */ // 下方是一个函数，返回类型为 size_t
    size_t (*encode)(rmt_encoder_t *encoder, rmt_channel_handle_t tx_channel, const void *primary_data, size_t data_size, rmt_encode_state_t *ret_state);
}
```



> 目录:

[toc]

#### 结构体中定义函数指针，并在外部调用时去实现。

作用：类似于 C++ 中的接口，提前在结构体中定义函数与其形参个数。

##### struct 同名结构体抽象层

- `typedef struct rmt_encoder_t rmt_encoder_t;` 结构体定义，使用 `rmt_encoder_t` 新的结构体对象创建 `rmt_encoder_t `。该方法的作用是类似于接口的定义。更抽象。。

  <img src="https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311171659428.png" alt="image-20231117111025251"  />

  ```c
  /**
   * @brief Interface of RMT encoder
   */
  struct rmt_encoder_t {
      /**
       * @brief Encode the user data into RMT symbols and write into RMT memory
       *
       * @note The encoding function will also be called from an ISR context, thus the function must not call any blocking API.
       * @note It's recommended to put this function implementation in the IRAM, to achieve a high performance and less interrupt latency.
       *
       * @param[in] encoder Encoder handle  //编码
       * @param[in] tx_channel RMT TX channel handle, returned from `rmt_new_tx_channel()`
       * @param[in] primary_data App data to be encoded into RMT symbols
       * @param[in] data_size Size of primary_data, in bytes
       * @param[out] ret_state Returned current encoder's state
       * @return Number of RMT symbols that the primary data has been encoded into
       */ // 下方是一个函数，返回类型为 size_t
      size_t (*encode)(rmt_encoder_t *encoder, rmt_channel_handle_t tx_channel, const void *primary_data, size_t data_size, rmt_encode_state_t *ret_state);
  
      /**
       * @brief Reset encoding state
       *
       * @param[in] encoder Encoder handle
       * @return
       *      - ESP_OK: reset encoder successfully
       *      - ESP_FAIL: reset encoder failed
       */
      esp_err_t (*reset)(rmt_encoder_t *encoder);
  
      /**
       * @brief Delete encoder object
       *
       * @param[in] encoder Encoder handle
       * @return
       *      - ESP_OK: delete encoder successfully
       *      - ESP_FAIL: delete encoder failed
       */
      esp_err_t (*del)(rmt_encoder_t *encoder);
  };
  
  /***************************** 抽象层 *************************************/
  /** @cond */
  // 左边新的类型名，右边是已有类型名，目的是为了让该结构体更抽象 (确实挺抽象的)
  typedef struct rmt_encoder_t rmt_encoder_t;
  /** @endcond */
  ```



##### ' size_t (*encode)(.....); '用于抽象结构体函数体。类似于函数的重写( 面向对象子类重写父类方法 )。

> 说明：

&emsp;&emsp;该抽象函数的结构体声明在上方的结构体定义中。该部分的解释未必完全正确，在这里我们称该函数为接口。如果有类似的叫法都为同一个意思。

- 根据 chatGPT 的描述，该成员具体的类型是一个 `encode`  的函数指针，调用方法

  > 节选的daima

  ```c
  static size_t IRAM_ATTR rmt_encode_bytes(rmt_encoder_t *encoder, rmt_channel_handle_t channel, const void *primary_data, size_t data_size, rmt_encode_state_t *ret_state)
  {
  	// 重写该函数的函数体
  }
  
  static esp_err_t rmt_del_bytes_encoder(rmt_encoder_t *encoder)
  {
      // 函数体
  }
  
  static esp_err_t rmt_del_copy_encoder(rmt_encoder_t *encoder)
  {
      // 函数体
  }
  
  // 接口层, ' rmt_encoder_t base; '该成员的类型就是接口抽象层中定义的抽象函数
  typedef struct rmt_bytes_encoder_t {
      rmt_encoder_t base;     // encoder base class  // 存放 rmt 外设的基本属性
      size_t last_bit_index;  // index of the encoding bit position in the encoding byte
      size_t last_byte_index; // index of the encoding byte in the primary stream
      rmt_symbol_word_t bit0; // bit zero representing
      rmt_symbol_word_t bit1; // bit one representing
      struct {
          uint32_t msb_first: 1; // encode MSB firstly
      } flags;
  } rmt_bytes_encoder_t;
  
  // 定义 rmt_bytes_encoder_t *encoder ，用来存放并实现结构体的函数接口
  esp_err_t rmt_new_bytes_encoder(const rmt_bytes_encoder_config_t *config, rmt_encoder_handle_t *ret_encoder)
  {
      esp_err_t ret = ESP_OK;
      ESP_GOTO_ON_FALSE(config && ret_encoder, ESP_ERR_INVALID_ARG, err, TAG, "invalid argument");
      rmt_bytes_encoder_t *encoder = heap_caps_calloc(1, sizeof(rmt_bytes_encoder_t), RMT_MEM_ALLOC_CAPS);
      ESP_GOTO_ON_FALSE(encoder, ESP_ERR_NO_MEM, err, TAG, "no mem for bytes encoder");
      encoder->base.encode = rmt_encode_bytes; // 函数的重写。在此重写该函数，
      encoder->base.del = rmt_del_bytes_encoder; // 函数重写。重写 del 函数
      encoder->base.reset = rmt_bytes_encoder_reset;  //函数重写。重写 reset 函数
      encoder->bit0 = config->bit0;
      encoder->bit1 = config->bit1;
      encoder->flags.msb_first = config->flags.msb_first;
      // return general encoder handle
      *ret_encoder = &encoder->base;  // 将初始化完毕的接口 传递给 传入的结构体地址。(拷贝赋值)
      ESP_LOGD(TAG, "new bytes encoder @%p", encoder);
  err:
      return ret;
  }
  ```

  



#### __containerof( encoder, rmt_led_strip_encoder_t, base ) 函数，其实该函数是一个宏。

作用：通过结构体对象获取整个结构体的指针。

![image-20231117154557431](https://dearliao.oss-cn-shenzhen.aliyuncs.com/Note/picture/202311171659430.png)

```c
/*************** 代码出处 *****************/
static esp_err_t rmt_del_dshot_encoder(rmt_encoder_t *encoder)
{
    rmt_dshot_esc_encoder_t *dshot_encoder = __containerof(encoder, rmt_dshot_esc_encoder_t, base);    /* 该函数为了获取整个函数的指针 */
    rmt_del_encoder(dshot_encoder->bytes_encoder);
    rmt_del_encoder(dshot_encoder->copy_encoder);
    free(dshot_encoder);
    return ESP_OK;
}

```

在 Linux 内核代码部分也有给该函数，但是目前没深入到那么底层。因此，该函数暂时不作解释

