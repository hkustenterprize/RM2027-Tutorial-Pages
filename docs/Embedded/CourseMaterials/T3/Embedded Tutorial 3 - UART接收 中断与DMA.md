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

##### 什么是中断?
```text
main() 函数 正在运行
      ↓ UART收到数据，产生中断
UART Callback 函数
      ↓ 处理完成
回到 main() 之前断掉位置，继续运行
```

> **中断（Interrupt）**：硬件事件发生时，CPU 暂停当前工作，转去执行对应的中断处理程序；处理完成后，再回到原来的位置继续运行。

---
## 示例：中断接收 8 字节数据

```c
uint8_t rx_buffer[8];
volatile uint8_t rx_it_ready = 0;
```

进入 `while (1)` 前启动第一次接收：
```c
HAL_UART_Receive_IT(&huart2, rx_buffer, sizeof(rx_buffer));
```
`HAL_UART_Receive_IT()` 启动一次接收后立即返回；收满指定字节数后，HAL 调用接收完成回调。

回调函数的实现：
```c
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if (huart->Instance == USART2)
    {
        rx_it_ready = 1;
    }
}
```

主循环处理数据，再启动下一次接收：
```c
while (1)
{
    if (rx_it_ready)
    {
        rx_it_ready = 0;
        
        HAL_UART_Receive_IT(&huart2, rx_buffer, sizeof(rx_buffer));
    }

    RunOtherTasks();
}
```
> 中断接收是“一次性预约”：完成后必须再次调用 `HAL_UART_Receive_IT()`，否则后续数据不会进入缓冲区。

---

中断中应当只做：**保存数据、设置标志、尽快退出**。
不要在中断中使用 `HAL_Delay()`、长循环、大量 `printf()` 或复杂控制算法。

```c
// 被主程序和中断共同访问、且可能随时变化的简单变量通常声明为 volatile。
volatile uint8_t data_ready = 0; 

void SomeInterruptCallback(void)
{
    data_ready = 1; // 相当于一个信号 通知主循环
}

while (1)
{
    if (data_ready) // 得知进了一次中断
    {
        data_ready = 0; // 立即复位
        ProcessData();
    }
}
```

---
## 实践：中断接收 8 字节数据

#### CubeMX 启用 USART 中断

以 `USART2` 为例：
1. 将 USART2 设置为 `Asynchronous`
2. 保持上一讲的 `115200 8N1`
3. 在 `NVIC Settings` 中勾选 **USART2 global interrupt**
4. Generate Code

CubeMX 会在 `stm32f1xx_it.c` 中生成真正的硬件中断入口：
```c
void USART2_IRQHandler(void)
{
    HAL_UART_IRQHandler(&huart2);
}
```

`HAL_UART_IRQHandler()` 检查中断原因，并在合适时调用我们编写的 HAL Callback。通常不要删除这个 IRQ Handler。

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

可以把一段连续到达、随后出现空闲的数据视为一个**数据块**。

关键函数：
```c
HAL_UARTEx_ReceiveToIdle_IT(&huart2, rx_idle_it, sizeof(rx_idle_it));
```

---
## 示例：空闲中断接收不定长数据

```c
#define IDLE_IT_BUFFER_SIZE 64

uint8_t rx_buf[IDLE_IT_BUFFER_SIZE];
volatile uint16_t rx_idle_it_length = 0;
volatile uint8_t rx_idle_it_ready = 0;
```

进入主循环前启动：
```c
HAL_UARTEx_ReceiveToIdle_IT(&huart2, rx_buf, sizeof(rx_buf));
```

线路空闲或缓冲区填满时，HAL 调用以下回调。`Size` 是本次实际收到的字节数：
```c
void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *huart, uint16_t Size)
{
    if (huart->Instance == USART2)
    {
        rx_idle_it_length = Size;
        rx_idle_it_ready = 1;
    }
}
```

```c
while (1)
{
    if (rx_idle_it_ready)
    {
        uint16_t length = rx_idle_it_length;
        rx_idle_it_ready = 0;

        ProcessReceivedData(rx_idle_it, length);

        HAL_UARTEx_ReceiveToIdle_IT(&huart2, rx_idle_it, sizeof(rx_idle_it));
    }

    RunOtherTasks();
}
```

这个入门示例在主循环处理期间暂停接收。更高要求的项目需要双缓冲或软件环形缓冲区。

如果需要区分回调原因：
```c
HAL_UART_RxEventTypeTypeDef event;
event = HAL_UARTEx_GetRxEventType(huart);

if (event == HAL_UART_RXEVENT_IDLE) { /* 线路空闲 */ }
if (event == HAL_UART_RXEVENT_TC)   { /* 缓冲区已满 */ }
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
UART收到字节 -> 中断CPU -> CPU读取UART外设寄存器 -> CPU写入RAM整合数据
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
UART收到字节 -> DMA自动写入RAM整合数据

DMA在一批数据完成后中断CPU -> CPU在RAM中处理数据
```

DMA外设不负责解析命令、帧头或校验，它只负责**搬运**。
我们只需要配置搬运的**源头和目的地**，DMA外设就可以 **独立于CPU** 搬运数据。

（DMA挂的原理正是如此：数据不流经CPU而直接通过DMA送到另一台计算机进行分析，故也不经过操作系统，所以软件反作弊难查出DMA作弊。）

---
## 示例：DMA+空闲中断，Normal 模式

进入主循环前启动：

```c
#define DMA_RX_BUFFER_SIZE 128

uint8_t rx_dma[DMA_RX_BUFFER_SIZE];
volatile uint16_t rx_dma_length = 0;
volatile uint8_t rx_dma_ready = 0;

HAL_UARTEx_ReceiveToIdle_DMA(&huart2,
                             rx_dma,
                             sizeof(rx_dma));

/* 本例不使用缓冲区半满事件 */
__HAL_DMA_DISABLE_IT(huart2.hdmarx, DMA_IT_HT);
```

```c
void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *huart,
                                uint16_t Size)
{
    if (huart->Instance == USART2)
    {
        rx_dma_length = Size;
        rx_dma_ready = 1;
    }
}
```

```c
while (1)
{
    if (rx_dma_ready)
    {
        uint16_t length = rx_dma_length;
        rx_dma_ready = 0;

        ProcessReceivedData(rx_dma, length);

        if (HAL_UARTEx_ReceiveToIdle_DMA(&huart2,
                                         rx_dma,
                                         sizeof(rx_dma)) == HAL_OK)
        {
            __HAL_DMA_DISABLE_IT(huart2.hdmarx, DMA_IT_HT);
        }
    }

    RunOtherTasks();
}
```

Normal 模式在 IDLE 或缓冲区满后停止 DMA，必须重新启动。处理与重启之间存在空窗，适合“发一包、停一下、等待响应”的通信。

Circular DMA 连续高速数据可能一直不产生 IDLE

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
- Circular 模式可以持续接收

缺点：
- Normal 模式重启时有空窗
- 缓冲区过小或处理不及时仍会丢数据

适用场景：**不定长、高速、大数据量或连续到达**的数据。

---
## 四种接收方式对比

| 方式             | 搬运者    | 结束条件           | 适合场景           |
| -------------- | ------ | -------------- | -------------- |
| **阻塞接收**       | CPU主循环 | 收满 Size / 超时   | 简单（RM不建议用）     |
| **普通中断**       | CPU中断  | 收满 Size        | 短小固定长度数据       |
| **空闲中断**       | CPU中断  | 收满 Size / IDLE | 不定长、成段数据（有间隔）  |
| **DMA + 空闲中断** | DMA    | 收满 Size / IDLE | **高速、大量、连续数据** |
|                |        |                |                |

应面向不同场景选择满足需求的最合适的方案。

**优化你的通讯流程**让他能使用**DMA**收发。（队里只用DMA）

---
## 举一反三：DMA在嵌入式中的其他应用场景？

显然不止UART接收

发送可不可以用DMA？

点对点大量连续数据搬运

---
## 应用层协议：IDLE 不等于真正的数据包边界

IDLE 只能帮我们获得**数据块**，不能代替应用层协议：

```text
| Header 2B | Payload nB | CRC 2B |
```

```c
void ProcessReceivedData(uint8_t *data, uint16_t length)
{
    if (length < 5) return;
    if (data[0] != 0xAA || data[1] != 0x55) return;

    uint8_t payload_length = data[2];
    if (length != 2 + 1 + payload_length + 2) return;
    if (!CheckCRC(data, length)) return;

    HandlePayload(&data[3], payload_length);
}
```

也可以用**位域**！非常方便！

现实世界，场景复杂。
一条应用层数据包甚至可能被两次 IDLE 分开，需要解析器跨数据块保存状态。
可靠通信仍然需要**帧头、帧尾、校验和**。

---
## 应用层协议的解析（Decode）方法 ：位操作与位域

#### 什么是位域 Bit-field？

C 语言允许规定一个结构体成员只占用若干个 bit，这种成员称为**位域**。
```c
typedef struct
{
    uint8_t motor_enable : 1;
    uint8_t led_enable   : 1;
    uint8_t error        : 1;
    uint8_t mode         : 3;
    uint8_t reserved     : 2;
} StatusBits_t;
```

使用起来像普通结构体成员：
```c
StatusBits_t status = {0};  

status.motor_enable = 1;
status.mode = 5;

if (status.error)
{
    HandleError();
}
```

每个冒号后的数字表示成员占用的位数：
```c
uint8_t mode : 3;  // 只能保存 0～7
```

位域的优点：
- 表达多个开关状态时可读性较好
- 可以紧凑地保存小范围整数
- 访问形式接近普通结构体成员

---
## 实践：作业1

使用 DMA RxToIdle 接收FS遥控器的数据，并解码。

Tuto阶段成绩占比：
DDL：

