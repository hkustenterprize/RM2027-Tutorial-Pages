# Embedded Tutorial 3 - UART接收 中断与DMA

---

## 回顾

上一讲 我们讲解了什么是 UART

并完成了一次 STM32 与 电脑 间的双向UART通讯

本节，我们详细讲解：STM32接收UART数据的方式，代码怎么写

---

## 方式一：阻塞接收

> **阻塞**：操作未完成时，程序停在**函数内部**等待，不能继续执行主循环中的其他任务。

```c
HAL_StatusTypeDef HAL_UART_Receive(
    UART_HandleTypeDef *huart, // 使用哪个UART
    uint8_t *pData,           // 接收缓冲区
    uint16_t Size,            // 必须接收的字节数
    uint32_t Timeout          // 最长等待时间，单位ms
);
```

> `Size` 是“必须收满多少字节”。

示例：固定接收 4 字节，成功后原样发回。

```c
uint8_t rx_buffer[4];

while (1)
{
    HAL_StatusTypeDef status;

    status = HAL_UART_Receive(&huart2, rx_buffer, sizeof(rx_buffer), HAL_MAX_DELAY);

    if (status == HAL_OK)
    {
        HAL_UART_Transmit(&huart2, rx_buffer, sizeof(rx_buffer), 1000);
    }
}
```

如果电脑只发送 `ABC` 三字节，函数会继续等待第 4 字节，直到超时。

---

## 阻塞接收的缺点

阻塞接收：

- CPU主动等待每一个字节的到来，并处理。

缺点：

- 等待期间 CPU 无法处理其他任务，大量CPU资源被浪费在了等待这种无意义的事情上。
- 超时时间太短容易截断（大包），太长会卡住程序
- 不适合需要同时控制电机、读取传感器的主循环（有其他高频任务）

---

## 有没有一种更好的方式？

等待信件有两种方式：

- 一直站在门口看，等信件来了就立刻收下。
- 门上安装个门铃：**做事**，信件到达时，**门铃响**，(放下手头的事去取信。收完信)，**继续做事**。

---

## 方式二：中断接收  ??什么是中断??

### 什么是中断?

```text
main() 函数 正在运行
      ↓ UART收到数据，产生中断
UART Callback 函数
      ↓ 处理完成
回到 main() 之前断掉位置，继续运行
```

> **中断（Interrupt）**：硬件事件发生时，CPU 暂停当前工作，转去执行对应的中断处理程序；处理完成后，再回到原来的位置继续运行。

---

## 开启中断的函数

```c
/**
  * @brief  Receives an amount of data in non blocking mode.
  * @note   When UART parity is not enabled (PCE = 0), and Word Length is configured to 9 bits (M1-M0 = 01),
  *         the received data is handled as a set of u16. In this case, Size must indicate the number
  *         of u16 available through pData.
  * @param  huart Pointer to a UART_HandleTypeDef structure that contains
  *               the configuration information for the specified UART module.
  * @param  pData Pointer to data buffer (u8 or u16 data elements).
  * @param  Size  Amount of data elements (u8 or u16) to be received.
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_UART_Receive_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size)
```

## 注册Callback的函数

```c
#if (USE_HAL_UART_REGISTER_CALLBACKS == 1)
/**
  * @brief  Register a User UART Callback
  *         To be used instead of the weak predefined callback
  * @note   The HAL_UART_RegisterCallback() may be called before HAL_UART_Init(), HAL_HalfDuplex_Init(), HAL_LIN_Init(),
  *         HAL_MultiProcessor_Init() to register callbacks for HAL_UART_MSPINIT_CB_ID and HAL_UART_MSPDEINIT_CB_ID
  * @param  huart uart handle
  * @param  CallbackID ID of the callback to be registered
  *         This parameter can be one of the following values:
  *           @arg @ref HAL_UART_TX_HALFCOMPLETE_CB_ID Tx Half Complete Callback ID
  *           @arg @ref HAL_UART_TX_COMPLETE_CB_ID Tx Complete Callback ID
  *           @arg @ref HAL_UART_RX_HALFCOMPLETE_CB_ID Rx Half Complete Callback ID
  *           @arg @ref HAL_UART_RX_COMPLETE_CB_ID Rx Complete Callback ID
  *           @arg @ref HAL_UART_ERROR_CB_ID Error Callback ID
  *           @arg @ref HAL_UART_ABORT_COMPLETE_CB_ID Abort Complete Callback ID
  *           @arg @ref HAL_UART_ABORT_TRANSMIT_COMPLETE_CB_ID Abort Transmit Complete Callback ID
  *           @arg @ref HAL_UART_ABORT_RECEIVE_COMPLETE_CB_ID Abort Receive Complete Callback ID
  *           @arg @ref HAL_UART_MSPINIT_CB_ID MspInit Callback ID
  *           @arg @ref HAL_UART_MSPDEINIT_CB_ID MspDeInit Callback ID
  * @param  pCallback pointer to the Callback function
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_UART_RegisterCallback(UART_HandleTypeDef *huart, HAL_UART_CallbackIDTypeDef CallbackID,
                                            pUART_CallbackTypeDef pCallback)
```

## 示例：中断接收数据

```c
uint8_t rxBuffer[4];
```

进入 `while (1)` 前启动第一次接收：

```c
HAL_UART_RegisterCallback(&huart2, HAL_UART_RX_COMPLETE_CB_ID, uart2RxCallback);
HAL_UART_Receive_IT(&huart2, rxBuffer, 4);
```

`HAL_UART_Receive_IT()` 启动一次接收后立即返回；收满指定字节数后，HAL 调用接收完成回调。

回调函数的实现：

```c
void uart2RxCallback(UART_HandleTypeDef *huart)
{
    HAL_UART_Receive_IT(&huart2, rxBuffer, 4);
}
```

主循环运行其他代码：

```c
while (1)
{
    // 运行其他代码
    HAL_Delay(1);
}
```

> 中断接收是“一次性预约”：完成后必须再次调用 `HAL_UART_Receive_IT()`，否则后续数据不会进入缓冲区。

---

## 实践：中断接收 8 字节数据

### CubeMX 启用 USART 中断

以 `USART2` 为例：

1. 将 USART2 设置为 `Asynchronous`
2. 保持上一讲的 `115200 8N1`
3. 在 `NVIC Settings` 中勾选 **USART2 global interrupt**
4. 将中断优先级设置为9
5. Generate Code

---

## 普通中断接收：优缺点和适用场景

优点：

- 数据没到时不阻塞主循环，没有无意义的等待。
- 秒杀阻塞这种安卓思维。

缺点？（后续会讲）

适用场景：固定 n 字节控制指令、短传感器数据、低频命令等。

问题：如果使用：

```c
HAL_UART_Receive_IT(&huart2, rx_buffer, 64);
```

电脑只发送 5 字节的 `start`，就不会触发接收完成回调。如何接收这种**不定长数据**？

---

## 方式三：UART 空闲中断

UART 没有发送数据时，RX 保持高电平，即 **Idle** 状态。

当 UART 已经收到数据，随后线路持续一个数据帧左右的时间没有新数据，UART 外设会产生 **IDLE Line Event（空闲线事件）**。

```text
[byte 1][byte 2][byte 3]-------- idle --------
                                  ^ 触发IDLE
```

可以把一段连续到达、随后出现空闲的数据视为一个**数据块**。会触发空闲中断。

开启中断的函数：

```c
/**
  * @brief Receive an amount of data in interrupt mode till either the expected number of data is received or an IDLE event occurs.
  * @note   Reception is initiated by this function call. Further progress of reception is achieved thanks
  *         to UART interrupts raised by RXNE and IDLE events. Callback is called at end of reception indicating
  *         number of received data elements.
  * @note   When UART parity is not enabled (PCE = 0), and Word Length is configured to 9 bits (M = 01),
  *         the received data is handled as a set of uint16_t. In this case, Size must indicate the number
  *         of uint16_t available through pData.
  * @param huart UART handle.
  * @param pData Pointer to data buffer (uint8_t or uint16_t data elements).
  * @param Size  Amount of data elements (uint8_t or uint16_t) to be received.
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_UARTEx_ReceiveToIdle_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size)
```

注册Callback的函数

```c
/**
  * @brief  Register a User UART Rx Event Callback
  *         To be used instead of the weak predefined callback
  * @param  huart     Uart handle
  * @param  pCallback Pointer to the Rx Event Callback function
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_UART_RegisterRxEventCallback(UART_HandleTypeDef *huart, pUART_RxEventCallbackTypeDef pCallback)
```

---

## 示例：空闲中断接收不定长数据

```c
uint8_t rxBuffer[64];
```

进入主循环前启动：

```c
HAL_UART_RegisterRxEventCallback(&huart2, uart2RxToIdleCallback);
HAL_UARTEx_ReceiveToIdle_IT(&huart2, rxBuffer, 64);
```

线路空闲或收到数据大小达到64时，HAL 调用以下回调。`Size` 是本次实际收到的字节数：

```c
void uart2RxToIdleCallback(UART_HandleTypeDef *huart, uint16_t size)
{
  HAL_UARTEx_ReceiveToIdle_IT(&huart2, rxBuffer, 64);
}
```

主循环运行其他代码

```c
while (1)
{
    // 运行其他代码
    HAL_Delay(1);
}
```

---

## 空闲中断接收：优缺点和适用场景

优点：

- 不需要预先知道精确长度
- 数据停下来后，可以得到本次实际接收长度
- 继承中断接收的优点

缺点：继承中断接收的缺点（到底是啥啊？？）

适用场景：

- **不定长、成段到达**的数据
- 电脑串口助手发送的命令
- “发送一段后短暂停顿”的设备

---

## 中断接收不为人知的秘密 .\_?

普通中断接收的数据路径：

```text
UART收到一个字节 -> 中断CPU -> CPU读取UART外设寄存器 -> CPU把数据从寄存器搬运到内存
      ⬆                                                 ↓
      ------------------------<-------------------------
当：一批数据接收完成 -> CPU在RAM中处理数据
```

接收每个字节时仍需要 CPU 中断**搬运** 只是从UART外设搬运到内存而已！

高频、连续数据会带来较大的中断负担

想象一下 你收128字节的打包，会断128次！

非常之不优雅。

那怎么办？那你能帮帮我么？

---

## 帮！DMA来帮

（对，你没听错，就是你以前听到过的那个DMA！）

---

## 方式四：DMA？! 对的对的 !

> **DMA（Direct Memory Access，直接存储器访问）**：在外设和内存之间自动搬运数据的硬件模块（**也是一个外设**）。**不经过CPU直接操作内存，极大提升效率。**

DMA接收流程：

```text
UART收到字节 -> DMA自动把数据搬运到内存

DMA在一批数据完成后中断CPU -> CPU执行Callback
```

DMA外设只负责**搬运**。

我们只需要配置搬运的**源头和目的地**，DMA外设就可以 **独立于CPU** 搬运数据。

（DMA挂的原理正是如此：数据不流经CPU而直接通过DMA送到另一台计算机进行分析，故也不经过操作系统，所以软件反作弊难查出DMA作弊。）

---

## 示例：DMA+空闲中断，Normal 模式

开启DMA空闲中断接收的函数

```c
/**
  * @brief Receive an amount of data in DMA mode till either the expected number of data is received or an IDLE event occurs.
  * @note   Reception is initiated by this function call. Further progress of reception is achieved thanks
  *         to DMA services, transferring automatically received data elements in user reception buffer and
  *         calling registered callbacks at half/end of reception. UART IDLE events are also used to consider
  *         reception phase as ended. In all cases, callback execution will indicate number of received data elements.
  * @note   When the UART parity is enabled (PCE = 1), the received data contain
  *         the parity bit (MSB position).
  * @note   When UART parity is not enabled (PCE = 0), and Word Length is configured to 9 bits (M = 01),
  *         the received data is handled as a set of uint16_t. In this case, Size must indicate the number
  *         of uint16_t available through pData.
  * @param huart UART handle.
  * @param pData Pointer to data buffer (uint8_t or uint16_t data elements).
  * @param Size  Amount of data elements (uint8_t or uint16_t) to be received.
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_UARTEx_ReceiveToIdle_DMA(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size)
```

注册Callback的函数

```c
/**
  * @brief  Register a User UART Rx Event Callback
  *         To be used instead of the weak predefined callback
  * @param  huart     Uart handle
  * @param  pCallback Pointer to the Rx Event Callback function
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_UART_RegisterRxEventCallback(UART_HandleTypeDef *huart, pUART_RxEventCallbackTypeDef pCallback)
```

自定义的Callback函数

```c
void uart2RxToIdleDMACallback(UART_HandleTypeDef *huart, uint16_t size)
{
  HAL_UARTEx_ReceiveToIdle_DMA(&huart2, rxBuffer, 64);
  __HAL_DMA_DISABLE_IT(huart2.hdmarx, DMA_IT_HT);
}
```

进入主循环前启动：

```c
HAL_UART_RegisterRxEventCallback(&huart2, uart2RxToIdleDMACallback);
HAL_UARTEx_ReceiveToIdle_DMA(&huart2, rxBuffer, 64);
__HAL_DMA_DISABLE_IT(huart2.hdmarx, DMA_IT_HT);
```

```c
while (1)
{
    // 运行其他代码
    HAL_Delay(1);
}
```

---

## 实践：DMA+空闲中断 接收

CubeMX 配置 `USART2_RX` DMA：

1. `DMA Settings` 中添加 RX 请求
2. Direction：**Peripheral To Memory**
3. Peripheral Increment：**Disable**
4. Memory Increment：**Enable**
5. Data Width：**Byte**
6. 入门示例选择 **Normal** 模式
7. NVIC 中启用 USART2 和对应 DMA Channel 中断

---

## DMA+空闲中断：优缺点和适用场景

优点：

- DMA 自动搬运，CPU 不必逐字节响应
- 适合高波特率和大数据量
- Receive-to-Idle 能获得不定长数据

适用场景：**不定长、高速、大数据量或连续到达**的数据。

---

## 四种接收方式对比

| 方式             | 搬运者    | 结束条件           | 适合场景           |
| -------------- | ------ | -------------- | -------------- |
| **阻塞接收**       | CPU主循环 | 收满 Size / 超时   | 简单（RM不建议用）     |
| **普通中断**       | CPU中断  | 收满 Size        | 短小固定长度数据       |
| **空闲中断**       | CPU中断  | 收满 Size / IDLE | 不定长、成段数据（有间隔）  |
| **DMA + 空闲中断** | DMA    | 收满 Size / IDLE | **高速、大量、连续数据** |

应面向不同场景选择满足需求的最合适的方案。

也可以用**位域**！非常方便！

## 数据解码

现实世界，场景复杂。

一个数字可能被uart拆成两个包发送，我们需要从两个包里还原出来原始数据

---

## 应用层协议的解析（Decode）方法 ：位域

### 什么是位域 Bit-field？

C 语言允许规定一个结构体成员只占用若干个 bit，这种成员称为**位域**。

```c
typedef struct {
    uint16_t HP  : 13;
    uint16_t MP  : 13;
    uint16_t STR : 10;
    uint16_t DEF : 10;
    uint16_t INT : 10;
} __attribute__((packed, aligned(1))) ReceivedBitValue;
```

每个冒号后的数字表示成员占用的位数：

```c
uint8_t mode : 3;  // 只能保存 0～7
```

位域解码示例

```c
#include <string.h>
#include <stdint.h>
typedef struct {
    uint16_t HP  : 13;
    uint16_t MP  : 13;
    uint16_t STR : 10;
    uint16_t DEF : 10;
    uint16_t INT : 10;
} __attribute__((packed, aligned(1))) ReceivedBitValue;

typedef struct {
  uint16_t HP;
  uint16_t MP;
  uint16_t STR;
  uint16_t DEF;
  uint16_t INT;
}ReceivedValue;

static uint8_t data[7] = {0xFA, 0x02, 0x5F, 0xBE, 0x70, 0xCB, 0xE7};
ReceivedBitValue decoded_data_in_bit_field;
ReceivedValue decoded_data;
```

解码代码

```c
memcpy(&decoded_data_in_bit_field, data, sizeof(data)); // 复制内存
decoded_data.HP = decoded_data_in_bit_field.HP;
decoded_data.MP = decoded_data_in_bit_field.MP;
decoded_data.STR = decoded_data_in_bit_field.STR;
decoded_data.DEF = decoded_data_in_bit_field.DEF;
decoded_data.INT = decoded_data_in_bit_field.INT;
```

---

## 实践：作业1

使用 DMA RxToIdle 接收FS遥控器的数据，并解码。

Tuto阶段成绩占比：

DDL：
