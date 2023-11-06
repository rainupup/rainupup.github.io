# 串口

## 通信接口

- 通信的目的：将一个设备的数据传送到另一个设备，扩展硬件系统
- 通信协议：制定通信的规则，通信双方按照协议规则进行数据收发

| **名称** |       **引脚**       | **双工** | **时钟** | **电平** | **设备** |
| :------: | :------------------: | :------: | :------: | :------: | :------: |
|  USART   |        TX、RX        |  全双工  |   异步   |   单端   |  点对点  |
|   I2C    |       SCL、SDA       |  半双工  |   同步   |   单端   |  多设备  |
|   SPI    | SCLK、MOSI、MISO、CS |  全双工  |   同步   |   单端   |  多设备  |
|   CAN    |     CAN_H、CAN_L     |  半双工  |   异步   |   差分   |  多设备  |
|   USB    |        DP、DM        |  半双工  |   异步   |   差分   |  点对点  |



**同步通讯与异步通讯**

- 同步通讯：收发设备双方会使用一根信号线表示时钟信号，在`时钟信号`的驱动下双方进行协调，同步数据，通讯中通常双方会统一规定在时钟信号的上升沿或下降沿对数据线进行采样，对应时钟极性与时钟相位。
- 异步通讯：不需要**时钟信号**进行数据同步，它们直接在数据信号中**穿插一些同步用的信号位**，或者把**主体数据进行打包**，以数据帧（串口：起始位 数据 校验位(可以没有) 停止位）的格式传输数据，某些通讯中还需要双方约定数据的传输速率（波特率），以便更好地同步。

**STM32串口简介**

- **USART**-通用同步异步收发器(Universal Synchronous Asynchronous Receiver and Transmitter)是一个串行通信设备，可以灵活地与外部设备进行全双工数据交换。
- 有别于 USART 还有一个**UART**(Universal Asynchronous Receiver and Transmitter)，它是在 USART 基础上裁剪掉了同步通信功能（时钟同步），只有异步通信。简单区分同步和异步就是看通信时需不需要对外提供时钟输出，我们平时用的串口通信基本都是 UART。
- 串行通信一般是以**帧格式**传输数据，即是一帧一帧的传输，每帧包含有**起始信号、数据信息、校验信息(由我们自己设置)、停止信号**。
- 串口通信使用TTL逻辑：Transistor-transistor-logic

**波特率**

- 比特率：每秒钟传送的比特数，单位bit/s
- 波特率：每秒钟传送的码元数，单位Baud(每秒钟的高低电平数量)
- 比特率 = 波特率 * log2 M ，M表示每个码元承载的信息量
- 二进制系统中，波特率数值上等于比特率



**串口的几个重要的参数:**

1. 波特率：串口通信的速率
2. 起始位：标志一个数据帧的开始，固定为**低电平**，当数据开始发送时，产生一个**下降沿**。(空闲–>起始位)
3. 数据位：数据帧的有效载荷，1为高电平，0为低电平，**低位先行**；比如 发送数据帧0x0F(00001111) 在数据帧里就是低位线性 即 1111 0000
4. 校验位：用于数据验证，根据数据位计算得来。分为奇校验，偶校验和无校验。
5. 停止位：用于**数据帧间隔**，固定为**高电平**。数据帧发送完成后，产生一个**上升沿**。(数据传输–>停止位)

| F103xx串口号 | TXD  | RXD  |
| :----------: | :--: | :--: |
|      1       | PA9  | PA10 |
|      2       | PA2  | PA3  |
|      3       | PB10 | PB11 |
|      4       | PC10 | PC11 |
|      5       | PC12 | PD2  |



## USART简介

1. USART（Universal Synchronous/Asynchronous Receiver/Transmitter）通用同步/异步收发器
2. USART是STM32内部集成的硬件外设，可根据数据寄存器的一个字节数据自动生成数据帧时序，从TX引脚发送出去，也可自动接收RX引脚的数据帧时序，拼接为一个字节数据，存放在数据寄存器里
3. 自带波特率发生器，最高达4.5Mbits/s
4. 可配置数据位长度（8/9）、停止位长度（0.5/1/1.5/2）
5. 可选校验位（无校验/奇校验/偶校验）
6. 支持同步模式、硬件流控制、DMA、智能卡、IrDA、LIN
7. STM32F103C8T6 USART资源： USART1、 USART2、 USART3

![image-20230421124841178](STM32卷二.assets/image-20230421124841178.png)

![image-20230421124848542](STM32卷二.assets/image-20230421124848542.png)



**USART基本结构**

![image-20230421124929513](STM32卷二.assets/image-20230421124929513.png)

![image-20231006104936523](STM32卷二.assets/image-20231006104936523.png)

注意：在F1/F4芯片中，我们使用DR寄存器间接操作RDR/TDR寄存器







**波特比率寄存器（BRR）**

![image-20230802154942461](STM32卷二.assets/image-20230802154942461.png)把USARTDIV的整数部分写入位[15:4]， USARTDIV的小数部分写入[3:0]

USARTDIV = DIV_Mantissa + （DIV_Fraction/16）;DIV_Mantissa为整数部分，DIV_Fraction为小数部分



**波特率发生器**

- 发送器和接收器的波特率由波特率寄存器BRR里的DIV确定
- 计算公式：$Baud=\frac{fck}{(16∗USARTDIV)}$
- 其中fck是串口的时钟，如：USART1的时钟是PCLK2,其他串口都是PCLK1

![image-20230802155257321](STM32卷二.assets/image-20230802155257321.png)

![image-20230802155403856](STM32卷二.assets/image-20230802155403856.png)





**硬件电路**

- 简单双向串口通信有两根通信线（发送端TX和接收端RX）

- **TX与RX要交叉连接**

- 当只需单向的数据传输时，可以只接一根通信线

- 当电平标准不一致时，需要加电平转换芯片

  ![image-20230421130720312](STM32卷二.assets/image-20230421130720312.png)



### USART框图

USART是STM32内部集成的硬件外设，可以根据数据寄存器的一个字节数据自动生成数据帧时序，从TX引脚发送出去，也可以自动接收RX引脚的数据帧时序，拼接成一个字节数据，存放在数据寄存器里。

当配置好USART的电路之后，直接读取数据寄存器，就可以自动发送数据和接收数据了。在发送和接收的模块有4个重要的寄存器

1. 发送数据寄存器TDR
2. 发送移位寄存器，把一个字节的数据一位一位的移出去
3. 接收数据寄存器RDR
4. 接收移位寄存器，把一个字节的数据



![image-20230421124905122](STM32卷二.assets/image-20230421124905122.png)



### 寄存器

状态寄存器(USART_SR) ：主要使用

- TC：发送完成 (Transmission complete) 
  - 0：发送还未完成；
  - 1：发送完成。
- RXNE：读数据寄存器非空 (Read data register not empty) 
  - 0：数据没有收到；
  - 1：收到数据，可以读出。

数据寄存器(USART_DR)：包含了发送或接收的数据。



控制寄存器 USART_CR1、USART_CR2、USART_CR3：设置控制模式



### 串口发送

在配置串口的各个参数时，可以选择发送数据帧的数据位的大小，可选8位或9位。

串口发送数据实际上就是对发送数据寄存器TDR进行写操作。

1. 当串口发送数据时，会检测发送移位寄存器是不是有数据正在移位，如果没有移位，那么这个数据就会立刻转移到发送移位寄存器里。准备发送。
2. 当数据移动到移位寄存器时，会产生一个TXE发送寄存器空标志位，该位描述如下。当TXE被置1，那么就可以在TDR写入下一个数据了。即发送下一个数据。

### 串口接收

1. 数据从RX引脚通向接收移位寄存器，在接收控制的控制下，一位一位的读取RX的电平，把第一位放在最高位，然后右移，移位八次之后就可以接收一个字节了。
2. 当一个字节数据移位完成之后，这一个字节的数据就会整体的移到接收数据寄存器RDR里来。
3. 在转移时会置RXNE接收标志位，即RDR寄存器非空， 下方为该位的描述。当被置1后，就说明数据可以被读出

### 校验

利用串口传输数据时，近距离传输还好，远距离传输由于线路长度影响，可能会使信号在传输过程中出现不可预知的错误，为了达到通信的稳定性，在远距离通信时一般要引入一种校验方式来去除干扰。奇校验ODD，偶校验EVEN，累加和校验，CRC循环码冗余码校验



**奇校验（odd parity）**：让传输的数据（包含校验位）中1的个数为奇数。

即：如果传输字节中1的个数是偶数，则校验位为“1”，奇数相反；即最终使1的个数为奇数

以发送字符：10101010为例

**偶校验（even parity）**：让传输的数据（包含校验位）中1的个数为偶数。

即：如果传输字节中1的个数是偶数，则校验位为“0”，奇数相反；即最终使1的个数为偶数

![image-20231006112915983](STM32卷二.assets/image-20231006112915983.png)

数据和校验位发送给接受方后，接收方再次对数据中1的个数进行计算，如果为奇数则校验通过，表示此次传输过程未发生错误。如果不是奇数，则表示有错误发生，此时接收方可以向发送方发送请求，要求重新发送一遍数据。

**优缺点：**

- 奇偶校验的检错率只有**50%**，因为只有奇数个数据位发生变化能检测到，如果偶数个数据位发生变化则无能为力了╮(╯﹏╰）╭
- 奇偶校验每传输一个字节都需要加一位校验位，对传输效率影响很大。
- 奇偶校验只能发现错误，但不能纠正错误，也就是说它只能告诉你出错了，但不能告诉你怎么出错了，一旦发现错误，只好重发。
- 虽然奇偶校验有很多缺点，但因为其使用起来十分简单，故目前仍被广泛使用。

**累加和校验**

所谓的累加和校验有很多种，最常见的一种是在每次通信数据包最后都加一个字节的校验数据，这个校验字节里的数据是通信数据包里所有数据的**不进位累加和**。例如：

![img](STM32卷二.assets/v2-5f52da10f710724ad67b9ea6c50f983c_720w.png)

接收方接收到数据后同样对一个数据包的数据进行不进位累加和计算，如果累加出的结果与校验位相同的话就认为传输的数据没有错误。

**优缺点：**

- 实现起来方便简单，被广泛运用。
- 检错率一般，例如一个字节多1，一个字节少1，则会出现误判。
- 和奇偶校验一样，只能发现错误，但不能纠正错误。





## HAL库

### 句柄结构体

以UART为例

stm32f1xx_hal_uart.h

stm32f1xx_hal_uart.c

**UART_HandleTypeDef**：串口的句柄

```
typedef struct __UART_HandleTypeDef{
  USART_TypeDef                 *Instance;        /* UART 寄存器基地址 */
  UART_InitTypeDef              Init;             /* UART 通信参数 */
  uint8_t                       *pTxBuffPtr;      /* 指向 UART 发送缓冲区 */
  uint16_t                      TxXferSize;       /* UART 发送数据的大小 */
  __IO uint16_t                 TxXferCount;      /* UART 发送数据的个数 */
  uint8_t                       *pRxBuffPtr;      /* 指向 UART 接收缓冲区 */
  uint16_t                      RxXferSize;       /* UART 接收数据大小 */
  __IO uint16_t                 RxXferCount;      /* UART 接收数据的个数 */
  DMA_HandleTypeDef             *hdmatx;          /* UART 发送参数设置（DMA） */
  DMA_HandleTypeDef             *hdmarx;          /* UART 接收参数设置（DMA） */
  HAL_LockTypeDef               Lock;             /* 锁定对象 */
  __IO HAL_UART_StateTypeDef    gState;           /* UART 发送状态结构体 */
  __IO HAL_UART_StateTypeDef    RxState;          /* UART 接收状态结构体 */
  __IO uint32_t                 ErrorCode;        /* UART 操作错误信息 */
} UART_HandleTypeDef;
```

1）Instance：指向 UART 寄存器基地址。实际上这个基地址 HAL 库已经定义好了，可以选择范围：USART1~ USART3、UART4、UART5。 2）Init：UART 初始化结构体，用于配置通讯参数，如波特率、数据位数、停止位等等。下面我们再详细讲解这个结构体。 3）pTxBuffPtr，TxXferSize，TxXferCount：分别是指向发送数据缓冲区的指针，发送数据的大小，发送数据的个数。 4）pRxBuffPtr，RxXferSize，RxXferCount：分别是指向接收数据缓冲区的指针，接受数据的大小，接收数据的个数； 5）hdmatx，hdmarx：配置串口发送接收数据的 DMA 具体参数。 6）Lock：对资源操作增加操作锁保护功能，可选 HAL_UNLOCKED 或者 HAL_LOCKED 两个参数。如果 gState 的值等于 HAL_UART_STATE_RESET，则可认为串口未被初始化，此时，分配锁资源，并且调用 HAL_UART_MspInit 函数来对串口的 GPIO 和时钟进行初始化。

7）gState，RxState：分别是 UART 的发送状态、工作状态的结构体和 UART 接受状态的结构体。HAL_UART_StateTypeDef 是一个枚举类型，列出串口在工作过程中的状态值，有些值只适用于 gState，如 HAL_UART_STATE_BUSY。 8）ErrorCode：串口错误操作信息。主要用于存放串口操作的错误信息。



**UART_InitTypeDef**：串口的结构体

```
typedef struct{
  uint32_t BaudRate;                  /* 波特率 */
  uint32_t WordLength;                /* 字长 */
  uint32_t StopBits;                  /* 停止位 */
  uint32_t Parity;                    /* 校验位 */
  uint32_t Mode;                      /* UART 模式 */
  uint32_t HwFlowCtl;                 /* 硬件流设置 */
  uint32_t OverSampling;              /* 过采样设置 */
} UART_InitTypeDef;
```

1）BaudRate：波特率设置。一般设置为 2400、9600、19200、115200。 2）WordLength：数据帧字长，可选 8 位或 9 位。这里我们设置为 8 位字长数据格式。 3）StopBits：停止位设置，可选 0.5 个、1 个、1.5 个和 2 个停止位，一般我们选择 1 个停止位。 4）Parity：奇偶校验控制选择，我们设定为无奇偶校验位。 5）Mode：UART 模式选择，可以设置为只收模式，只发模式，或者收发模式。这里我们设置为全双工收发模式。 6）HwFlowCtl：硬件流控制选择，我们设置为无硬件流控制。 7）OverSampling：过采样选择，选择 8 倍过采样或者 16 过采样，一般选择 16 过采样。

### **常用函数**

```c
/**初始化函数
*参数：串口句柄
*返回值：HAL_StatusTypeDef枚举类型，有4个，分别是HAL_OK表示成功，HAL_ERROR表示错误，HAL_BUSY表示忙碌HAL_TIMEOUT超时。
**/
HAL_StatusTypeDef HAL_UART_Init(UART_HandleTypeDef *huart);

/**用于开启以中断的方式接收指定字节。数据接收在中断处理函数里面实现。
* 形参 1 是 UART_HandleTypeDef 结构体指针类型的串口句柄。
* 形参 2 是要接收的数据地址。
* 形参 3 是要接收的数据大小，以字节为单位。
**/
HAL_StatusTypeDef HAL_UART_Receive_IT(UART_HandleTypeDef *huart,uint8_t *pData, uint16_t Size);

/**以阻塞的方式发送指定字节的数据
* 形参 1 ：UART_HandleTypeDef 结构体类型指针变量
* 形参 2：指向要发送的数据地址，地址所在的内容可以有多个数据，不一定是一个8位数据
* 形参 3：要发送的数据大小，以字节为单位
* 形参 4：设置的超时时间，以ms单位
**/
HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);

//HAL 库中断处理公共函数
void HAL_UART_IRQHandler(UART_HandleTypeDef *huart);
//回调函数
__weak void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_TxHalfCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_RxHalfCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_ErrorCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_AbortCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_AbortTransmitCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_AbortReceiveCpltCallback(UART_HandleTypeDef *huart)
    
    
// 常用的宏    
/** 
  * @param  __HANDLE__ specifies the UART Handle.
  * @param  __FLAG__ specifies the flag to check.
  *            @arg UART_FLAG_CTS:  CTS Change flag (not available for UART4 and UART5)
  *            @arg UART_FLAG_LBD:  LIN Break detection flag
  *            @arg UART_FLAG_TXE:  Transmit data register empty flag
  *            @arg UART_FLAG_TC:   Transmission Complete flag
  *            @arg UART_FLAG_RXNE: Receive data register not empty flag
  *            @arg UART_FLAG_IDLE: Idle Line detection flag
  *            @arg UART_FLAG_ORE:  Overrun Error flag
  *            @arg UART_FLAG_NE:   Noise Error flag
  *            @arg UART_FLAG_FE:   Framing Error flag
  *            @arg UART_FLAG_PE:   Parity Error flag
  */
#define __HAL_UART_GET_FLAG(__HANDLE__, __FLAG__) (((__HANDLE__)->Instance->SR & (__FLAG__)) == (__FLAG__))
#define __HAL_UART_CLEAR_FLAG(__HANDLE__, __FLAG__) ((__HANDLE__)->Instance->SR = ~(__FLAG__))
```

### **案例**

 将上位机发送给单片机的数据，再发回来

```c
UART_HandleTypeDef UART_Handle;	               // 串口句柄
uint8_t g_rx_buffer[1];                        // 接收的内容
uint8_t g_rx_flag = 0;                         // 接收标志位

void UART_Init(void){
    UART_Handle.Instance = USART1;	                     // 使用USART1
    
    UART_Handle.Init.BaudRate = 115200;           
    UART_Handle.Init.Mode =  UART_MODE_TX_RX;            // 读写
    UART_Handle.Init.Parity = UART_PARITY_NONE;          // 不使用校验
    UART_Handle.Init.StopBits = UART_STOPBITS_1;         // 停止位1
    UART_Handle.Init.WordLength = UART_WORDLENGTH_8B;    // 数据位8
    UART_Handle.Init.HwFlowCtl = UART_HWCONTROL_NONE ;
    
    HAL_UART_Init(&UART_Handle);                         // 初始化
    
    HAL_UART_Receive_IT(&UART_Handle, g_rx_buffer, 1);   // 接收1个字节的内容存入g_rx_buffer
}
//初始化回调函数，可以在内部完成GPIO、USART、NVIC的初始化
void HAL_UART_MspInit(UART_HandleTypeDef *huart){
	// 判断是不是USART1调用的回调函数
    if(huart->Instance == USART1){
        __HAL_RCC_GPIOA_CLK_ENABLE();
        __HAL_RCC_USART1_CLK_ENABLE();
        
        GPIO_InitTypeDef GPIO_InitStructure;
        GPIO_InitStructure.Mode = GPIO_MODE_AF_PP;
        GPIO_InitStructure.Pin = GPIO_PIN_9;
        GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;
        HAL_GPIO_Init(GPIOA,&GPIO_InitStructure);
        
        GPIO_InitStructure.Mode = GPIO_MODE_INPUT;
        GPIO_InitStructure.Pin = GPIO_PIN_10;
        GPIO_InitStructure.Pull = GPIO_PULLUP;
        GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;
        HAL_GPIO_Init(GPIOA,&GPIO_InitStructure);
        
        HAL_NVIC_SetPriorityGrouping(NVIC_PRIORITYGROUP_2);
        HAL_NVIC_SetPriority(USART1_IRQn,2,2);
        HAL_NVIC_EnableIRQ(USART1_IRQn);
    }
}
// 中断服务函数
void USART1_IRQHandler(void){
    HAL_UART_IRQHandler(&UART_Handle);                     // 调用HAL中断服务函数，在内部调用一些回调函数
    HAL_UART_Receive_IT(&UART_Handle, g_rx_buffer, 1);     // 从新开启接收中断
}
// 接收完成回调函数
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart){
    if(huart->Instance == USART1){
        g_rx_flag = 1;                // 接收完成
    }
}
int main(void){
    HAL_Init();                             
    sys_stm32_clock_init(RCC_PLL_MUL9);       
    delay_init(72);                           
    UART_Init();
   
    while(1){
        if(g_rx_flag == 1){       // 当接收完成，再发送给上位机
            HAL_UART_Transmit(&UART_Handle,g_rx_buffer,1,10);           // 将1个字节大小的g_rx_buffer发出
            while(__HAL_UART_GET_FLAG(&UART_Handle,UART_FLAG_TC) != 1); // 等待发送完成标志位
            g_rx_flag = 0;        // 从置接收完成标志位
        }
    }
}
```



## 标准库

### 函数

stm32f10_usart.h

**USART初始化结构体**

```
typedef struct{
  uint32_t USART_BaudRate;            //串口通信使用的波特率 一般是9600或者是115200
  uint16_t USART_WordLength;          //数据位 有8位和9位可以选择
  uint16_t USART_StopBits;            //停止位 有1、0.5、2位
  uint16_t USART_Parity;              //校验位，可以选择奇偶校验和不校验。没有需求就直接无校验
  uint16_t USART_Mode;                //串口的模式，发送模式(USART_Mode_Tx)还是接收模式(USART_Mode_Rx)，还是两者都有    
  uint16_t USART_HardwareFlowControl; //是否选择硬件流触发，一般这个不选，所以选择无硬件流触发。
} USART_InitTypeDef;
void USART_DeInit(USART_TypeDef* USARTx);                                     //初始化
void USART_Init(USART_TypeDef* USARTx, USART_InitTypeDef* USART_InitStruct);
void USART_StructInit(USART_InitTypeDef* USART_InitStruct);

void USART_Cmd(USART_TypeDef* USARTx, FunctionalState NewState);
void USART_ITConfig(USART_TypeDef* USARTx, uint16_t USART_IT, FunctionalState NewState);
void USART_SendData(USART_TypeDef* USARTx, uint16_t Data);                    //发送数据
uint16_t USART_ReceiveData(USART_TypeDef* USARTx);                            //接收数据
//查看状态
FlagStatus USART_GetFlagStatus(USART_TypeDef* USARTx, uint16_t USART_FLAG);
void USART_ClearFlag(USART_TypeDef* USARTx, uint16_t USART_FLAG);
ITStatus USART_GetITStatus(USART_TypeDef* USARTx, uint16_t USART_IT);
void USART_ClearITPendingBit(USART_TypeDef* USARTx, uint16_t USART_IT);
```

### 在STM32中的配置

#### 数据发送

**1.RCC开启USART、串口TX/RX所对应的GPIO口**

```
RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);         //开启USART1的时钟
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);          //开启GPIOA的时钟(查了端口复用表)
```



**2. 初始化GPIO口**

根据自己的需求来配置GPIO口，发送和接收是都需要还是只需要其中一个。然后对应的根据引脚定义表来初始化对应的GPIO口。

| 端口 | 复用      |
| ---- | --------- |
| PA9  | USART1_TX |
| PA10 | USART1_RX |

```c
GPIO_InitTypeDef GPIO_InitStructure;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;            //复用推挽输出
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;                  //USART1对应的TX端为GPIOA9
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_Init(GPIOA, &GPIO_InitStructure);
```



**3.串口初始化**

```c
USART_InitTypeDef USART_InitStructure;
USART_InitStructure.USART_BaudRate = 9600;
USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;    //不使用硬件流触发
USART_InitStructure.USART_Mode = USART_Mode_Tx;                                    //TX 发送模式
USART_InitStructure.USART_Parity = USART_Parity_No;                                //不选择校验
USART_InitStructure.USART_StopBits = USART_StopBits_1;                             //停止位1位
USART_InitStructure.USART_WordLength = USART_WordLength_8b;                        //数据位8位
USART_Init(USART1, &USART_InitStructure);

USART_Cmd(USART1, ENABLE);                                                         //使能串口
```

**4. 串口发送数据**

```
void Serial_SendByte(uint8_t Byte){
    USART_SendData(USART1, Byte);                                //使用USART_SendData函数，将数据输出到USART1
    
    //0 表示数据还未转移到移位寄存器，循环等待
    //1 数据已经被转移到了移位寄存器可以发送数据
    while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
    //不需要手动清零 再次写入TDR时会自动清零 
}
```



**程序**

```C
#include "stm32f10x.h"                  // Device header
#include <stdio.h>
#include <stdarg.h>
void Serial_Init(void){
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    USART_InitTypeDef USART_InitStructure;
    USART_InitStructure.USART_BaudRate = 9600;
    USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
    USART_InitStructure.USART_Mode = USART_Mode_Tx;
    USART_InitStructure.USART_Parity = USART_Parity_No;
    USART_InitStructure.USART_StopBits = USART_StopBits_1;
    USART_InitStructure.USART_WordLength = USART_WordLength_8b;
    USART_Init(USART1, &USART_InitStructure);
    
    USART_Cmd(USART1, ENABLE);
}

void Serial_SendByte(uint8_t Byte){
    USART_SendData(USART1, Byte);
    while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
}

//发送数组
void Serial_SendArray(uint8_t *Array, uint16_t Length){
    uint16_t i;
    for (i = 0; i < Length; i ++){
        Serial_SendByte(Array[i]);
    }
}
//发送字符串
void Serial_SendString(char *String){
    uint8_t i;
    for (i = 0; String[i] != '\0'; i ++)  //String[i] != '\0;char类型会在最后一位+0，作为标志位{
        Serial_SendByte(String[i]);
    }
}

uint32_t Serial_Pow(uint32_t X, uint32_t Y){
    uint32_t Result = 1;
    while (Y --){
        Result *= X;
    }
    return Result;
}

void Serial_SendNumber(uint32_t Number, uint8_t Length){
    uint8_t i;
    for (i = 0; i < Length; i ++){
        Serial_SendByte(Number / Serial_Pow(10, Length - i - 1) % 10 + '0');
    }
}
//重写print函数
int fputc(int ch, FILE *f)                   {
    Serial_SendByte(ch);
    return ch;
}
//可变量print函数，需要#include <stdarg.h>
void Serial_Printf(char *format, ...){
    char String[100];
    va_list arg;
    va_start(arg, format);
    vsprintf(String, format, arg);
    va_end(arg);
    Serial_SendString(String);
}
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Serial.h"
#include <stdio.h>

int main(void){
    OLED_Init();
    Serial_Init();
    Serial_SendByte(0x41);
    uint8_t MyArray[] = {0x42, 0x43, 0x44, 0x45};
    Serial_SendArray(MyArray, 4);
    Serial_SendString("\r\nNum1=");
    Serial_SendNumber(111, 3);
    printf("\r\nNum2=%d", 222);        //需要#include <stdio.h>
    char String[100];
    sprintf(String, "\r\nNum3=%d", 333);
    Serial_SendString(String);
    Serial_Printf("\r\nNum4=%d", 444);
    Serial_Printf("\r\n");
    while (1){  
    }
}
```

#### 数据接收

**两种方式**

- 查询方式就是通过不断的查询RXNE标志位，通过判断RXNE位的状态来确定数据是否接收。
- 中断方式就是通过配置接收输出控制通道，配置NVIC，在中断服务子函数里进行数据的接收。

#### 查询RXNE标志位

```c
void Serial_Init(void){
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;                //TX
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;               //RX
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    USART_InitTypeDef USART_InitStructure;
    USART_InitStructure.USART_BaudRate = 9600;
    USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
    USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;       //TX RX
    USART_InitStructure.USART_Parity = USART_Parity_No;
    USART_InitStructure.USART_StopBits = USART_StopBits_1;
    USART_InitStructure.USART_WordLength = USART_WordLength_8b;
    USART_Init(USART1, &USART_InitStructure);
    
    USART_Cmd(USART1, ENABLE);
}
```

为0时数据没有收到，为1时收到了数据，数据可以从RDR里读出

```c
uint8_t RX_Data;
int main(){ 
    Serial_Init();
    Serial_SendByte(0x16);
    while(1){
        if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE)==SET)   //0 循环等待 1 可以接收数据{
            RX_Data=USART_ReceiveData(USART1);           
            Serial_SendByte(RX_Data);
        }
    }
}
```

#### 使用中断

**初始化** 

```
void Serial_Init(void){
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;                //TX
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;               //RX
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    USART_InitTypeDef USART_InitStructure;
    USART_InitStructure.USART_BaudRate = 9600;
    USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
    USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;       //TX RX
    USART_InitStructure.USART_Parity = USART_Parity_No;
    USART_InitStructure.USART_StopBits = USART_StopBits_1;
    USART_InitStructure.USART_WordLength = USART_WordLength_8b;
    USART_Init(USART1, &USART_InitStructure);
    
    USART_ITConfig(USART1, USART_IT_RXNE, ENABLE);            //开启中断输出控制
    //配置NVIC
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    NVIC_InitTypeDef NVIC_InitStructure;
    NVIC_InitStructure.NVIC_IRQChannel = USART1_IRQn;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
    NVIC_Init(&NVIC_InitStructure);
    
    USART_Cmd(USART1, ENABLE);
}   
```

**中断服务子函数**

```c
uint8_t Serial_RxData;
uint8_t Serial_RxFlag;
uint8_t Serial_GetRxFlag(void){
	if (Serial_RxFlag == 1){
		Serial_RxFlag = 0;
		return 1;
	}
	return 0;
}
uint8_t Serial_GetRxData(void){
	return Serial_RxData;
}
//上面两个函数 仅仅是封装，一个良好的习惯，一般不在中断中写太长的程序
//中断服务子函数
void USART1_IRQHandler(void){
	if (USART_GetITStatus(USART1, USART_IT_RXNE) == SET){    //RXNE 标志位为1 表示可以接收数据
		Serial_RxData = USART_ReceiveData(USART1);           //接收数据
		Serial_RxFlag = 1;
		USART_ClearITPendingBit(USART1, USART_IT_RXNE);      //清除RXNE标志位
	}
}
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Serial.h"
uint8_t RxData;
int main(void){
	OLED_Init();
	OLED_ShowString(1, 1, "RxData:");
	Serial_Init();
	while (1){
		if (Serial_GetRxFlag() == 1){
			RxData = Serial_GetRxData();
			Serial_SendByte(RxData);
			OLED_ShowHexNum(1, 8, RxData, 2);
		}
	}
}
```

## RS485

### RS485简介

- 在串口的基础上，RS-485仅是一个电气标准，描述了接口的物理层，像协议、时序、串行或并行数据以及链路全部由设计者或更高层协议定义。
- RS485 是美国电子工业协会（Electronic Industries Association，EIA）于1983年发布的串行通信接口标准，经通讯工业协会（TIA）修订后命名为 TIA/EIA-485-A。
- RS485 是一种工业控制环境中常用的通讯协议，其中RS 是 Recommended Standard 的缩写。
- RS485 是 半双工异步 串行通信。
- RS485是串行通信标准，使用差分信号传输，**抗干扰能力强**，常用于工控领域。
- RS485具有强大的组网功能，在串口基础协议之上还制定MODBUS协议。
- 串口基础协议：仅指封装了基本数据包格式的协议（基于数据位）
- MODBUS协议：使用基本数据包组合成通讯帧格式的高层应用协议（基于数据包或字节）

| 通信接口 | 通信方式   | 信号线    | 电平标准                              | 拓扑结构     | 通信距离 | 通讯速率 | 抗干扰能力 |
| -------- | ---------- | --------- | ------------------------------------- | ------------ | -------- | -------- | ---------- |
| TTL      | 全双工     | TX/RX/GND | 逻辑1 : 2.4~5 V  逻辑0 : 0~0.4 V      | 点对点       | 1米      | 100kbps  | 弱         |
| RS232    | 全双工     | TX/RX/GND | 逻辑1 : -(15~3) V  逻辑0 : +(3~15)  V | 点对点       | 100米    | 20kbps   | 较弱       |
| RS485    | **半双工** | 差分线AB  | 逻辑1 : +(2~6)V  逻辑0 : -(2~6)V      | **多点双向** | 1200米   | 100kbps  | 强         |

**特点**

- **支持多节点**：一般最大支持 32 个节点。
- 传输距离远：最远通讯距离可达1200米。
- 抗干扰能力强：差分信号传输。
- 连接简单：只需要两根信号线（A+和B-）就可以进行正常的通信。

**差分信号传输** RS485 通信采用差分信号传输，通常情况下只需要两根信号线就可以进行正常的通信。 在差分信号中，逻辑0和逻辑1是用两根信号线（A+和B-）的电压差来表示。

- 逻辑 1：两根信号线（A+和B-）的电压差在 +2V～+6V 之间。
- 逻辑 0：两根信号线（A+和B-）的电压差在 -2V～-6V 之间。 
- ![1691819585927](STM32卷二.assets/1691819585927.png)





### RS485总线连接图

![image-20230812134525182](STM32卷二.assets/image-20230812134525182.png)

**连接方式**

- 在 RS485 通信网络中，通常会使用 485 收发器来转换 TTL 电平和 RS485 电平。
- 节点中的串口控制器使用 RX 与 TX 信号线连接到 485 收发器上，而收发器通过**差分线**连接到网络总线。
- 串口控制器与收发器之间一般使用 TTL 信号传输，收发器与总线则使用差分信号来传输。
- 发送数据时，串口控制器的 TX 信号经过收发器转换成差分信号传输到总线上。
- 接收数据时，收发器把总线上的差分信号转化成 TTL 信号通过 RX 引脚传输到串口控制器中。
- **通常在这些节点中只能有一个主机，剩下的全为从机**。
- 在总线的起止端分别加了一个 120 欧的匹配电阻。

### TP8485

单片机串口通信一般是**TTL电平**，如果需要**RS485 通信**，就需要**RS485芯片**在中间转换一下。

TP8485E/SP3485 可作为 RS485 的收发器，该芯片支持 3.3V~5.5V 供电，最大传输速度可达 250Kbps，支持多达 256 个节点(单位负载为 1/8 的条件下)，并且支持输出短路保护。

![image-20230812134845661](STM32卷二.assets/image-20230812134845661.png)



图中 A、B 总线接口，用于连接 485 总线。RO 是接收输出端，DI 是发送数据收入端，RE是接收使能信号（低电平有效），DE 是发送使能信号（高电平有效）。

| 引脚 | 名称 | 功能                              |
| ---- | ---- | --------------------------------- |
| 1    | RO   | 接收器输出----接RX                |
| 2    | RE   | 接收器输出使能（低电平-接收使能） |
| 3    | DE   | 驱动器输出使能（高电平-发送使能） |
| 4    | DI   | 驱动器输入----接TX                |
| 5    | GND  | 接地                              |
| 6    | A    | 驱动器输出/接收器输入（同相）     |
| 7    | B    | 驱动器输出/接收器输入（反相）     |
| 8    | VCC  | 芯片供电+3.3V                     |

RS485 普通收发电路图原理：

- **RS485_EN 为高电平**，逻辑为1，发送使能，接收禁止。
- **RS485_EN 为低电平**，逻辑为0，发送禁止，接收使能

在编写驱动程序时：

- 在**发送数据前**，给RS485_EN 置高电平。
- 在**发送数据后**，给RS485_EN 置低电平。





# I2C

## 简介

- I2C总线是Philips公司在八十年代初推出的一种串行、半双工的总线，主要用于近距离、低速的芯片之间的通信；I2C总线有两根双向的信号线，一根数据线SDA用于收发数据，一根时钟线SCL用于通信双方时钟的同步；I2C总线硬件结构简单，简化了PCB布线，降低了系统成本，提高了系统可靠性，因此在各个领域得到了广泛应用。
- I2C总线是一种**多主机总线**，连接在 I2C总线上的器件分为主机和从机。主机有权发起和结束一次通信，从机只能被动呼叫；当总线上有多个主机同时启用总线时，I2C也具备冲突检测和仲裁的功能来防止错误产生；每个连接到I2C总线上的器件都有一个唯一的地址（7bit），且**每个器件都可以作为主机也可以作为从机（同一时刻只能有一个主机）**，总线上的器件增加和删除不影响其他器件正常工作；I2C总线在通信时总线上发送数据的器件为发送器，接收数据的器件为接收器。
- I2C总线可以通过外部连线进行在线检测，便于系统故障诊断和调试，故障可以立即被寻址，软件也有利于标准化和模块化，缩短开发时间。
- I2C总线上可挂接的设备数量受总线的最大电容400pF限制。
- 串行的8位双向数据传输速率在标准模式下可达100Kbit/s，快速模式下可达400Kbit/s，高速模式下可达3.4Mbit/s。
- 总线具有极低的电流消耗，抗噪声干扰能力强，增加总线驱动器可以使总线电容扩大10倍，传输距离达到15m；兼容不同电压等级的器件，工作温度范围宽。

### 特点

- 只需要SDA、SCL两条总线；
- 没有严格的波特率要求；
- 所有组件之间都存在简单的主/从关系，连接到总线的每个设备均可通过唯一地址进行软件寻址；
- I2C是真正的多主设备总线，可提供仲裁和冲突检测；
- 传输速度分为四种模式：
  - 标准模式（Standard Mode）：100 Kbps
  - 快速模式（Fast Mode）：400 Kbps
  - 高速模式（High speed mode）：3.4 Mbps
  - 超快速模式（Ultra fast mode）：5 Mbps
- 最大主设备数：无限制；
- 最大从机数：理论上，1008个从节点，寻址模式的最大节点数为**2的7次方**或**2的10次方**，但有16个地址保留用于特殊用途。



### 硬件电路

- 所有I2C设备的SCL连在一起，SDA连在一起
- 设备的SCL和SDA均要配置成**开漏输出模式**
- SCL和SDA各添加一个**上拉电阻**，阻值一般为4.7KΩ左右

![image-20230525090650587](STM32卷二.assets/image-20230525090650587.png)

![image-20230525090711934](STM32卷二.assets/image-20230525090711934.png)

### 物理特性

I2C 总线使用连接设备的 "SDA"（ 串行数据总线）和"SCL"（ 串行时钟总线 ） 来传送信息。

![1684976656602](STM32卷二.assets/1684976656602.png)



### 同步数据信号

I2C总线在进行数据传送时，时钟线SCL为低电平期间发送器向数据线上发送一位数据，在此期间数据线上的信号允许发生变化，时钟线SCL为高电平期间接收器从数据线上读取一位数据，在此期间数据线上的信号不允许发生变化，必须保持稳定。

### 时钟同步与仲裁

**（1）时钟同步**

时钟同步是通过I2C总线上的SCL之间的**线“与”（wire-AND）**来完成的，即如果有多个主机同时产生时钟，那么只有所有master都发送高电平时，SCL上才表现为高电平，否则SCL都表现为低电平。

线“与”特性由开漏电路实现。如果控制开漏输出INT为0，低电平，则VGS >0，N-MOS管导通，使输出接地，若控制开漏输出INT为1 (它无法直接输出高电平) 时，则N-MOS 管关闭，所以引脚既不输出高电平，也不输出低电平，为高阻态。正常使用时必须外部接上拉电阻。也就是说，**若有很多个开漏模式引脚（C1、C2....）连接到一起时，只有当所有引脚都输出高阻态，才由上拉电阻提供高电平，此高电平的电压为外部上拉电阻所接的电源的电压。若其中一个引脚为低电平，那线路就相当于短路接地，使得整条线路都为低电平，0 伏。**

**（2）仲裁** 

总线仲裁与时钟同步类似，当所有主机在SDA上都写1时，SDA的数据才是1，只要有一个主机写0，那此时SDA上的数据就是0.

一个主机每发送一个bit数据，在SCL为高电平时，就检查SDA的电平是否和发送的数据一致，如果不一致，这个主机便知道自己输掉了仲裁，然后停止向SDA写数据。也就是说，如果主机一致检查到总线上数据和自己发送的数据一致，则继续传输，这样在仲裁过程中就保证了赢得仲裁的master不会丢失数据。

输掉仲裁的主机在检测到自己输了之后也就不再产生时钟脉冲，并且要在总线空闲时才能重新传输。

仲裁的过程可能要经过多个bit的发送和检查，实际上两个主机如果发送的时序和数据完全一样，则两个主机都能正常完成整个数据传输。 **注意：**多个主机仲裁时，因为线“与”特性，**谁为低电平谁能强制SDA为低，也就是跟自己匹配，所以先高（高电平1）的那个就会仲裁失败。****

**总结：**当输出1时属于释放，即高低电平由外部上拉电阻决定。而输出0时，会将整个线路短路，显示低电平；假设此时芯片1输出0，芯片2输出1，输出0的芯片会将整个线路短路，这两个芯片最终输出的都是0，芯片2实际发送的和想发送的不一致，最终被仲裁掉。





### IIC的高阻态

漏极开路（Open Drain）即高阻状态，适用于输入/输出，其可独立输入/输出低电平和高阻状态，若需要产生高电平，则需使用外部上拉电阻

高阻状态：高阻状态是三态门电路的一种状态。逻辑门的输出除有高、低电平两种状态外，还有第三种状态——高阻状态的门电路。**电路分析时高阻态可做开路理解。**

我们知道IIC的所有设备是接在一根总线上的，那么我们进行通信的时候往往只是几个设备进行通信，那么这时候其余的空闲设备可能会受到总线干扰，或者干扰到总线，怎么办呢？

**为了避免总线信号的混乱，IIC的空闲状态只能有外部上拉， 而此时空闲设备被拉到了高阻态，也就是相当于断路， 整个IIC总线只有开启了的设备才会正常进行通信，而不会干扰到其他设备。**



**为什么IIC总线SDA建议用开漏模式？**

IIC的SDA脚即要作为输出，又要作为输入，用开漏输出模式，很好实现输出输入共用，避免IO模式频繁切换带来的麻烦。

输出时：主机（MCU）输出0，可以拉低信号，来实现低电平发送，主机输出1（实际不起作用），由外部上拉电阻上拉，实现高电平发送。

输入时：主机（MCU）设置输出1状态，此时因为MCU无法输出1，相当于释放了SDA脚，此时外部器件可以主动拉低SDA脚/释放SDA脚（同样由上拉电阻提供“输出1的功能”），实现SDA脚的高低电平变化。



**开漏 和 推挽 对比**

对于推挽输出而言：推挽输出是强输出电流模式，在此模式下的输出通道上的推挽结构MOS管，属于强上拉和强下拉的，这会影响读取IDR时的值，强上拉意味着会将来自外部的低电平输入强制置高，强下拉意味着会将来自外部的高电平输入强制置低

在开漏模式下，实现了虚拟的I/O端口双向通信：只要CPU输出逻辑“1”，由于N-MOS管处于关闭状态，I/O端口的电平将完全由外部电路决定，因此，CPU可以在“输入数据寄存器”读到外部电路的信号，而不是它自己输出的逻辑“1”。



使用推挽时：必须在发送完数据后，将输出模式 改为输入模式



当主机发送数据完毕，这时候要接受从机的ack。如果SDA是推挽，数据最后一个脉冲期间SDA要保持稳定，也就是主机在脉冲结束后还会把SDA保持一段时间，才能释放SDA。从机需要在主机释放之后才能拉低SDA（因为如果在主机未释放之前拉低SDA，主机又是输出高，也就是导线两端电平一高一低，炸了）。但是从机并不知道主机是何时释放的SDA，也就是说从机要尽量延后拉低。又但是从机必须在下个脉冲到来之前拉低SDA，由于脉冲并不受从机控制，如果数据脉冲和ack脉冲紧紧相邻，那么主机只能在数据脉冲结束瞬间拉低。又要尽量晚又要尽早，所以是矛盾的。而SDA开漏，从机可以立马拉低SDA，因为这时候导线两端电平不会出现一高一低的情况。主机和从机都只能拉低和浮空，外接电阻拉高。两者都浮空或拉低，没问题。一浮空一拉低，就是低嘛。

## 通讯特性

通常情况下，一个完整的I2C通信过程包括以下 4 部分：

- 开始条件
- 地址传送
- 数据传送
- 停止条件 主机在 SCL 线上输出串行时钟信号，数据在 SDA 线上进行传输，每传输一个字节（最高位 MSB 开始传输）后面跟随一个应答位，一个 SCL 时钟脉冲传输一个数据位。

**通信过程**

1. 主机发送起始信号启用总线
2. 主机发送一个字节数据指明从机地址和后续字节的传送方向
3. 被寻址的从机发送应答信号回应主机
4. 发送器发送一个字节数据
5. 接收器发送应答信号回应发送器
6. ........ （循环步骤4、5）
7. 通信完成后主机发送停止信号释放总线

第4步和第5步用的是发送器和接收器，不是主机和从机，这是由第一个字节的最后一位决定主给从发，还是从给主发。

也就是说，第一个字节和最后的停止信号一定是主机发给从机，但中间就不一定了。

**发送数据过程中不允许改变发送方向**（除非重启一次通信）。



**这是一帧标准的写数据帧**

![image-20230525091256417](STM32卷二.assets/image-20230525091256417.png)

### 开始和停止

当总线上的主机都不驱动总线，总线进入空闲状态， SCL 和 SDA 都为高电平。总线空闲状态下总线上设备都可以通过发送开始条件启动通信。 

当 **SCL 线为高时**，SDA 线上出现由高到低的信号，表明总线上产生了起始信号。 

当 **SCL 线为高时**，SDA 线上出现由低到高的信号，表明总线上产生了停止信号，如下图所示： ![1684976976590](STM32卷二.assets/1684976976590.png)

当两个起始信号之间没有停止信号时，即产生了重复起始信号。主机采用这种方法与另一个从机或相同的从机以不同传输方向进行通信（例如：从写入设备到从设备读出）而不释放总线。



停止情况有两种：

1. 主机不想发了，就发送停止信号；
2. 从机不想接了，不应答，主机就发送停止信号结束此次通信。

### 设备地址传送

- 开始条件或者重新开始条件后面的帧是地址帧（一个字节），用于指定主机通信的对象地址，在发送停止条件之前，指定的从机一直有效。
- I2C通讯支持：7 位寻址和10 位寻址两种模式。
  - 7 位寻址模式，地址帧（8bit）的高 7 位为从机地址，地址帧第 8 位来决定数据帧传送的方向：7 位从机地址 + 1位 读/写位，读/写位控制从机的数据传输方向（0：写； 1：读） 
  - 10 位寻址模式，主机发送帧，第一帧 发送头序列（11110XX0，其中 XX 表示 10 位地址的高 两位），然后第二帧发送低八位从机地址。 主机接收帧 ，第一帧发送头序列（11110XX0，其中 XX 表示 10 位地址的高两位），然后第二帧发送低八位从机地址。接下来会发送一个重新开始条件，然后再发送一帧头序列（11110XX1 ，其中 XX 表示 10 位地址的高两位）帧格式

![1684977452645](STM32卷二.assets/1684977452645.png)

**例如：**下图为设备地址码为1101000，也就是要和地址为1101000的设备通讯



![1684977272085](STM32卷二.assets/1684977272085.png)



### 数据传送

![1684977096307](STM32卷二.assets/1684977096307.png)

- SCL低电平期间，主机将数据位依次放到SDA线上（高位先行），然后释放SCL，从机将在SCL高电平期间读取数据位，所以SCL高电平期间SDA不允许有数据变化，依次循环上述过程8次，即可发送一个字节
- I2C总线通信时每个字节为8位长度，数据传送时，先传送最高位，后传送低位，发送器发送完一个字节数据后接收器必须发送1位应答位来回应发送器，即一帧共有9位。I2C每次发送数据必须是8位。



### 读写位

![1684977871176](STM32卷二.assets/1684977871176.png)

- 0：写数据
- 1：读数据



### 应答位

![1684977999946](STM32卷二.assets/1684977999946.png)

- 上拉电阻影响下SDA默认为高，而从机拉低SDA就是确认收到数据即ACK，否则NACK
- 0：有应答
- 1：没有应答到或者读取完成



## 读写简介

### 写出

这是一帧标准的写数据帧

![image-20230525091256417](STM32卷二.assets/image-20230525091256417.png)

**指定地址写：**对于指定设备（Slave Address），在指定地址（Reg Address）下，写入指定数据（Data）

![image-20230525092915276](STM32卷二.assets/image-20230525092915276.png)

单片机对编号为1101000的设备，进行写入操作，并把数据0xaa写入到0x19寄存器下

解析如下：

- S ：表示开始条件；
- SLA ：表示从机地址；
- R/W#：表示发送和接收的方向。当 R/W# 为“1” 时，将数据从从机发送到主机；当 R/W#为“0” 时，将数据从主机发送到从机；
- Sr ：表示重新开始条件；
- DATA ：表示发送和接收的数据；
- P ：表示停止条件。

### 读入

这是一帧标准的读数据帧

![image-20230525093220716](STM32卷二.assets/image-20230525093220716.png)

1. 它也是首先写入设备地址，然后是写数据0，接下来写的是寄存器的地址
2. 在收到从机的应答信号之后
3. 主机需要再发送一个起始信号，然后需要再发送一遍设备的地址
4. 然后才能发送读数据1
5. 接下来，存储器就会把寄存器里面的数据发送给单片机
6. 这样就完成了一帧数据的读取（最后的应答信号为1，是由主机发给从机）

**指定地址读**

对于指定设备（Slave Address），在指定地址（Reg Address）下，读取从机数据（Data）

![image-20230525093308671](STM32卷二.assets/image-20230525093308671.png)





## I2C外设简介

- STM32内部集成了硬件I2C收发电路，可以由硬件自动执行时钟生成、起始终止条件生成、应答位收发、数据收发等功能，减轻CPU的负担
- 支持多主机模型
- 支持7位/10位地址模式
- 支持不同的通讯速度，**标准速度(高达100 kHz)，快速(高达400 kHz)**
- 支持DMA
- 兼容SMBus协议
- STM32F103C8T6 硬件I2C资源：I2C1、I2C2



## I2C硬件框图



![image-20230709152636654](STM32卷二.assets/image-20230709152636654.png)



### 1.通讯引脚

STM32芯片有多个IIC外设，它们的IIC通讯信号引出到不同的GPIO引脚上，使用时必须配置这些指定的引脚。

| 引脚 |   I2C1   |     I2C2     |   I2C3   |
| :--: | :------: | :----------: | :------: |
| SCL  | PB6/PB10 | PH4/PF1/PB10 | PH7/PA10 |
| SDA  | PB7/PB9  | PH5/PF0/PB11 | PH8/PC9  |

### 2.时钟控制逻辑

- SCL线的时钟信号，由IIC接口根据时钟控制寄存器（CCR）控制，控制的参数主要位时钟频率。
- 可选择IIC通讯的“标准/快速”模式，这两个模式分别对应100/400Kbits/s的通讯速率。
- **在快速模式下可选择SCL时钟的占空比，可选T(low)/T(high) = 2或T(low)/T(high)=16/9模式。**
- CCR寄存器中12位的配置因子CCR，它与IIC外设的输入时钟源共用作用，产生SCL时钟。STM32的IIC外设输入时钟源位PCKL1。

**计算时钟频率**

- 标准模式：T high = CCR T pckl1 T low= CCRTpclk1
- 快速模式中 Tlow/Tlow =2时：Thigh = CCRTpckl1 T low = 2low*Tpckl1
- 快速模式中 Tlow/Tlow =16/9时：Thigh = 9CCRTpckl1 T low = 16lowTpckl1



例：PCLK1 = 36MHz,想要配置400Kbits/s 方法 PCLK时钟周期：TPCLK1 = 1/36 000 000 目标SCL时钟周期：TSCL = 1/400 000 SCL时钟周期内的高电平时间：Thigh = TSCL/3 SCL时钟周期内的低电平时间：Tlow = 2*TSCL/3 计算CCR的值：CCR = THIGH/TPCLK1 = 30 计算出来的值写入到寄存器即可

### 3.数据控制逻辑

- IIC的SDA信号主要连接到数据移位寄存器上，数据移位寄存器的数据来源及目标是数据寄存器(DR)、地址寄存器(OAR)、PEC寄存器以及SDA数据线。
- 当向外发送数据的时候，数据移位寄存器以“数据寄存器”为数据源，把数据一位一位地通过SDA信号线发送出去。
- 当从外部接收数据的时候，数据移位寄存器把SDA信号线采样到的数据一位一位地存储到”数据寄存器”中。

### 4.整体控制逻辑

- 包含控制寄存器CR1 CR2和状态寄存器SR1 SR2。
- 整体控制逻辑负责协调整个I2C外设，控制逻辑的工作模式根据我们配置的“控制寄存器(CR1/CR2)”的参数而改变。其中，CR1寄存器控制各种起始、结束的使能，CR2寄存器管理中断。



**状态寄存器 1(I2C_SR1)** 

常用状态

- **TxE**：数据寄存器为空(发送时) (Data register empty (transmitters)) 

  - 0：数据寄存器非空；

  - 1：数据寄存器空。

    – 在发送数据时，数据寄存器为空时该位被置’1’，在发送地址阶段不设置该位。 – 软件写数据到DR寄存器可清除该位；或在发生一个起始或停止条件后，或当PE=0时由硬件自动清除。 如果收到一个NACK，或下一个要发送的字节是PEC(PEC=1)，该位不被置位。 注：在写入第1个要发送的数据后，或设置了BTF时写入数据，都不能清除TxE位，这是因为数据寄存器仍然为空。

- **RxNE**：数据寄存器非空(接收时) (Data register not empty (receivers)) 

  - 0：数据寄存器为空；
  - 1：数据寄存器非空。 – 在接收时，当数据寄存器不为空，该位被置’1’。在接收地址阶段，该位不被置位。 – 软件对数据寄存器的读写操作清除该位，或当PE=0时由硬件清除。 在发生ARLO事件时，RxNE不被置位。 注：当设置了BTF时，读取数据不能清除RxNE位，因为数据寄存器仍然为满。

### I2C基本结构

![image-20230709154438430](STM32卷二.assets/image-20230709154438430.png)

注意：只有一个移位寄存器





## 通信过程

### 主机发送

![image-20230709154731106](STM32卷二.assets/image-20230709154731106.png)

1. 生成起始信号：CR1里的START 位置 1 后，接口会在 SR2的BUSY 位清零后生成一个起始位并切换到主模式。生成起始信号成功后SR1的SB 位会由硬件置 1 ，如果使能了事件中断(CR2的 ITEVFEN 位置 1 )则生成一个中断。这个中断在数据手册里官方命名为EV5。即为事件5中断。至于为什么从5开始，因为事件1到4在从机部分用了。接下来主设备会等待软件对 SR1 执行读操作，然后把从设备地址写入 DR 寄存器。只有这样才能清零SR1的SB位。
2. 从地址传输，接下来从地址会通过内部移位寄存器发送到 SDA 线。stm32支持10位和7位地址。大多数使用7位。在 7 位寻址模式下，会发送一个地址字节。地址字节被发出后，SR1的ADDR 位会由硬件置 1 如果使能了事件中断(CR2的 ITEVFEN 位置 1 )则生成一个中断EV6。接下来主设备会等待对 SR1 寄存器执行读操作，然后对 SR2 寄存器执行读操作，只有这样才能清零SR1的ADDR 位。
3. 主发送器，在发送出地址并将 ADDR 清零后，主设备会通过内部移位寄存器将 DR 寄存器中的字节发送到 SDA 线。在向数据寄存器写数据前如果开启了事件中断会进入中断EV8,接收到应答脉冲后，TxE 位会由硬件置 1 并在 ITEVFEN 和 ITBUFEN 位均置 1 时生成一个中断EV8。如果在上一次数据传输结束之前 TxE 位已置 1 但数据字节尚未写入 DR 寄存器，则 BTF 位会置 1，而接口会一直延长 SCL 低电平，等待I2C_DR 寄存器被写入，以将 BTF 清零。结束通信当最后一个字节写入 DR 寄存器后，软件会将 STOP 位置 1 以生成一个停止位EV8_2。接口会自动返回从模式（M/SL 位清零）。
4. 结束通信:主设备会针对自从设备接收的最后一个字节发送 NACK。在接收到此 NACK 之后，从设备会释放对 SCL 和 SDA 线的控制。随后，主设备可发送一个停止位/重复起始位。
   1. 为了在最后一个接收数据字节后生成非应答脉冲，必须在读取倒数第二个数据字节后（倒数第二个 RxNE 事件之后）立即将 ACK 位清零。
   2. 要生成停止位/重复起始位，软件必须在读取倒数第二个数据字节后（倒数第二个 RxNE事件之后）将 STOP/START 位置 1。
   3. 在只接收单个字节的情况下，会在 EV6 期间（在 ADDR 标志清零之前）禁止应答并在EV6 之后生成停止位。
   4. 生成停止位后，接口会自动返回从模式（M/SL 位清零）。

### 主机接收

![image-20230709155740844](STM32卷二.assets/image-20230709155740844.png)



## HAL库

### 案例

AT24C02读写数据，使用引脚PB6 PB7；单次读/写，连续读/写

**IIC程序**

```C
#define IIC_PORT    GPIOB
#define IIC_SCL_PIN GPIO_PIN_6  
#define IIC_SDA_PIN GPIO_PIN_7
#define IIC_SCL(x)     do{x ? HAL_GPIO_WritePin(IIC_PORT,IIC_SCL_PIN,GPIO_PIN_SET) : HAL_GPIO_WritePin(IIC_PORT,IIC_SCL_PIN,GPIO_PIN_RESET);}while(0)
#define IIC_SDA(x)     do{x ? HAL_GPIO_WritePin(IIC_PORT,IIC_SDA_PIN,GPIO_PIN_SET) : HAL_GPIO_WritePin(IIC_PORT,IIC_SDA_PIN,GPIO_PIN_RESET);}while(0)
#define IIC_READ_SDA   HAL_GPIO_ReadPin(IIC_PORT,IIC_SDA_PIN)

void IIC_Init(void){
    __HAL_RCC_GPIOB_CLK_ENABLE();
    
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.Mode = GPIO_MODE_OUTPUT_PP;    /* SCL 推挽输出*/
    GPIO_InitStructure.Pull = GPIO_PULLUP; 
    GPIO_InitStructure.Pin = IIC_SCL_PIN;
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH; 
    HAL_GPIO_Init(IIC_PORT,&GPIO_InitStructure);
    
    GPIO_InitStructure.Mode = GPIO_MODE_OUTPUT_OD;    /* SDA 开漏输出*/
    GPIO_InitStructure.Pin = IIC_SDA_PIN;
    HAL_GPIO_Init(IIC_PORT,&GPIO_InitStructure);
}
void IIC_delay(void){
    delay_us(2);
}
// 起始信号
void IIC_start(void){
    /* SCL为高电平期间, SDA从高电平往低电平跳变*/
    IIC_SDA(1);
    IIC_SCL(1); 
    IIC_delay();  
    IIC_SDA(0);
    IIC_delay(); 
    IIC_SCL(0); 
    IIC_delay();    /* 钳住总线, 准备发送/接收数据 */
}
// 停止信号
void IIC_Stop(void){
    /* SCL为高电平期间, SDA从低电平往高电平跳变*/
    IIC_SDA(0);
    IIC_delay();
    IIC_SCL(1);
    IIC_delay();
    IIC_SDA(1);
    IIC_delay();
}
/** 等待应答位
*  返回1：没有应答；返回0：有应答 
**/
uint8_t IIC_WaitAck(void){
    IIC_SDA(1); /* 释放SDA为高电平,让外设下拉为低电平 */
    IIC_delay();    
    IIC_SCL(1); /* 开始读数据 */
    IIC_delay();
    
    /* 等待外设拉低电平 */
    if(IIC_READ_SDA){   /* 读取SDA线上的高低电平，为高电平时代表没有应答 */
        IIC_Stop();     /* 没应答信号，直接停止 */      
        return 1;
    }
    IIC_SCL(0);         /* SCL低电平表示结束ACK检查 */ 
    IIC_delay(); 
    return 0;
}
/**
*   发送应答位
**/
void IIC_Ack(void){
    IIC_SCL(0);
    IIC_delay();
    IIC_SDA(0);  /* 数据线为低电平，表示应答 */
    IIC_delay();
    IIC_SCL(1);
    IIC_delay();
}
/**
*   发送非应答位
**/
void IIC_NAck(void){
    IIC_SCL(0);
    IIC_delay();
    IIC_SDA(1);  /* 数据线为高电平,表示非应答 */
    IIC_delay();
    IIC_SCL(1);
    IIC_delay();
}
// 发送1字节数据
void IIC_SendByte(uint8_t data){
    for(uint8_t i = 0;i < 8;i++){
        IIC_SDA( (data & 0x80) >> 7 );   /* 发送最高位 */
        IIC_delay();
        IIC_SCL(1);     /* 读取数据 */
        IIC_delay();
        IIC_SCL(0);     /* 准备下次读 */    
        data <<= 1;     /* 将下一位变为高位 */
    }
    IIC_SDA(1); /* 发送完成,主机释放SDA线 */ 
}
/** 接收数据
*   参数：1发送应答位，0不发送应答位
**/
uint8_t IIC_ReadByte(uint8_t ack){
    uint8_t receive = 0; 
    for(uint8_t i = 0;i < 8;i++){
        receive <<= 1;  
        IIC_SCL(1);
        IIC_delay();
        if(IIC_READ_SDA) receive++; /* 如果SDA为高电平则最低为置1 */
        IIC_SCL(0);  
        IIC_delay();
    }
    if(!ack) IIC_NAck();
    else    IIC_Ack();
    return receive;
}
```

**AT24C02程序**

```c
#include "./BSP/AT23XX/at23xx.h"
#include "./BSP/IIC/iic.h"
#include "./SYSTEM/delay/delay.h"
void AT24XX_Init(void){
    IIC_Init();
}
uint8_t AT24_ReadData(uint8_t ADDR){
    uint8_t receive;
    IIC_start();   
    IIC_SendByte(0XA0); /*设备号A，0代表写入*/
    IIC_WaitAck();
    IIC_SendByte(ADDR); /* 写入地址 */
    IIC_WaitAck();
    
    IIC_start();        /* 重新发起开始信号 */
    IIC_SendByte(0XA1); /*设备号A，1代表读*/
    IIC_WaitAck();  
    receive  = IIC_ReadByte(0);
    IIC_Stop();   
    return receive;
}
void AT24_SentData(uint8_t ADDR,uint8_t DATA){
    IIC_start();
    IIC_SendByte(0XA0); /*设备号A，0代表写入*/   
    IIC_WaitAck();
    IIC_SendByte(ADDR); /* 写入地址 */
    IIC_WaitAck();
    IIC_SendByte(DATA); /* 写入数据 */
    IIC_WaitAck();
    IIC_Stop();
    delay_ms(10);  		/* 等待写入，AT24需要等待5ms*/
}
/** 连续读
*   参数1：地址
*   参数2：存放数据的数组
*   参数3：读取的长度
**/
void AT24_ContinueRead(uint8_t ADDR,uint8_t* pDATA,uint8_t Lenght){
    while(Lenght--){
        *pDATA = AT24_ReadData(ADDR);
        pDATA++;
        ADDR++;
    }
}
/** 连续写
*   参数1：地址
*   参数2：存放数据的数组
*   参数3：读取的长度
**/
void AT24_ContinueWrite(uint8_t ADDR,uint8_t* pDATA,uint8_t Lenght){
    while(Lenght--){
        AT24_SentData(ADDR,*pDATA);
        pDATA++;
        ADDR++;
    }
}

uint8_t Data[] = {"WWW.RAINUPUP.CN"};
#define DataSize sizeof(Data) 
uint8_t datatemp[DataSize];
int main(void){
    HAL_Init();                         /* 初始化HAL库 */
    sys_stm32_clock_init(RCC_PLL_MUL9); /* 设置时钟, 72Mhz */
    delay_init(72);                     /* 延时初始化 */
    usart_init(115200);                 /* 串口初始化为115200 */
    led_init();                         /* 初始化LED */
    lcd_init();                         /* 初始化LCD */
    key_init();
    
    lcd_show_string(30, 120, 200, 16, 16, "SentData:", RED);
    lcd_show_string(30, 140, 200, 16, 16, "ReadData:", RED);
    AT24XX_Init();
    uint8_t key;
    while (1) {
        key = key_scan(0);
        
        if(key == 1){
            AT24_ContinueWrite(0,Data,DataSize);
            lcd_show_string(132,120, 200, 16, 16, (char*)Data, RED);
        }
        if(key == 2){
            AT24_ContinueRead(0,datatemp,DataSize);
            lcd_show_string(132,140, 200, 16, 16, (char*)datatemp, RED);
        }
        
        if(key == 3){
            AT24_SentData(100,13);				/* 往地址100写入数据 13*/
            lcd_show_num(132,120,13,2,16,BLUE);
        }
        if(key == 4){
            uint8_t re = AT24_ReadData(100);	/* 在地址100处读数据 */	
            lcd_show_num(132,140,re,2,16,BLUE);
        }
    }
}
```

## 标准库

### 常用函数

```c
// 初始化相关
void I2C_DeInit(I2C_TypeDef* I2Cx);
void I2C_Init(I2C_TypeDef* I2Cx, I2C_InitTypeDef* I2C_InitStruct);
void I2C_StructInit(I2C_InitTypeDef* I2C_InitStruct);
// 使能相关
void I2C_Cmd(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_DMACmd(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_DMALastTransferCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);
// 产生一个开始、结束信号，硬件IIC使用
void I2C_GenerateSTART(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_GenerateSTOP(I2C_TypeDef* I2Cx, FunctionalState NewState);
// 接收一个数据后，是否给从机一个应答，硬件IIC使用；在I2C结构体中也可以设置
void I2C_AcknowledgeConfig(I2C_TypeDef* I2Cx, FunctionalState NewState);
// 作为从机时，配置自身设备地址2
void I2C_OwnAddress2Config(I2C_TypeDef* I2Cx, uint8_t Address);
void I2C_DualAddressCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_GeneralCallCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_ITConfig(I2C_TypeDef* I2Cx, uint16_t I2C_IT, FunctionalState NewState);
// 发送数据、接收数据
void I2C_SendData(I2C_TypeDef* I2Cx, uint8_t Data);
uint8_t I2C_ReceiveData(I2C_TypeDef* I2Cx);

/* 发送一个7位地址，也可以使用上方I2C_SendData()函数进行发送地址
参数3:
	@arg I2C_Direction_Transmitter: Transmitter mode	发送模式
	@arg I2C_Direction_Receiver: Receiver mode			接收模式
*/
void I2C_Send7bitAddress(I2C_TypeDef* I2Cx, uint8_t Address, uint8_t I2C_Direction);
uint16_t I2C_ReadRegister(I2C_TypeDef* I2Cx, uint8_t I2C_Register);
void I2C_SoftwareResetCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_NACKPositionConfig(I2C_TypeDef* I2Cx, uint16_t I2C_NACKPosition);
void I2C_SMBusAlertConfig(I2C_TypeDef* I2Cx, uint16_t I2C_SMBusAlert);
void I2C_TransmitPEC(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_PECPositionConfig(I2C_TypeDef* I2Cx, uint16_t I2C_PECPosition);
void I2C_CalculatePEC(I2C_TypeDef* I2Cx, FunctionalState NewState);
uint8_t I2C_GetPEC(I2C_TypeDef* I2Cx);
void I2C_ARPCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_StretchClockCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);
void I2C_FastModeDutyCycleConfig(I2C_TypeDef* I2Cx, uint16_t I2C_DutyCycle);

//由于发送或接收数据时，会产生多个标志位的变化，所以需要查看多个标志位
/**		同时查询1个或多个标志位，重要
  *     @arg I2C_EVENT_SLAVE_TRANSMITTER_ADDRESS_MATCHED           : EV1
  *     @arg I2C_EVENT_SLAVE_RECEIVER_ADDRESS_MATCHED              : EV1
  *     @arg I2C_EVENT_SLAVE_TRANSMITTER_SECONDADDRESS_MATCHED     : EV1
  *     @arg I2C_EVENT_SLAVE_RECEIVER_SECONDADDRESS_MATCHED        : EV1
  *     @arg I2C_EVENT_SLAVE_GENERALCALLADDRESS_MATCHED            : EV1
  *     @arg I2C_EVENT_SLAVE_BYTE_RECEIVED                         : EV2
  *     @arg (I2C_EVENT_SLAVE_BYTE_RECEIVED | I2C_FLAG_DUALF)      : EV2
  *     @arg (I2C_EVENT_SLAVE_BYTE_RECEIVED | I2C_FLAG_GENCALL)    : EV2
  *     @arg I2C_EVENT_SLAVE_BYTE_TRANSMITTED                      : EV3
  *     @arg (I2C_EVENT_SLAVE_BYTE_TRANSMITTED | I2C_FLAG_DUALF)   : EV3
  *     @arg (I2C_EVENT_SLAVE_BYTE_TRANSMITTED | I2C_FLAG_GENCALL) : EV3
  *     @arg I2C_EVENT_SLAVE_ACK_FAILURE                           : EV3_2
  *     @arg I2C_EVENT_SLAVE_STOP_DETECTED                         : EV4
  *     @arg I2C_EVENT_MASTER_MODE_SELECT                          : EV5	
  *     @arg I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED            : EV6	注意有两个事件6     
  *     @arg I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED               : EV6
  *     @arg I2C_EVENT_MASTER_BYTE_RECEIVED                        : EV7
  *     @arg I2C_EVENT_MASTER_BYTE_TRANSMITTING                    : EV8
  *     @arg I2C_EVENT_MASTER_BYTE_TRANSMITTED                     : EV8_2
  *     @arg I2C_EVENT_MASTER_MODE_ADDRESS10                       : EV9
  *		…………
  *		返回值：
  * 		- SUCCESS: 指定事件发生
  * 		- ERROR: 指定事件未发生
  */
ErrorStatus I2C_CheckEvent(I2C_TypeDef* I2Cx, uint32_t I2C_EVENT);
// 同时查询2个标志位
uint32_t I2C_GetLastEvent(I2C_TypeDef* I2Cx);
// 查询一个标准位
FlagStatus I2C_GetFlagStatus(I2C_TypeDef* I2Cx, uint32_t I2C_FLAG);

void I2C_ClearFlag(I2C_TypeDef* I2Cx, uint32_t I2C_FLAG);
ITStatus I2C_GetITStatus(I2C_TypeDef* I2Cx, uint32_t I2C_IT);
void I2C_ClearITPendingBit(I2C_TypeDef* I2Cx, uint32_t I2C_IT);
typedef struct
{
  uint32_t I2C_ClockSpeed;                //时钟速度,0~100KHMz为标准状态，100~400KHMz为快速状态  
  uint16_t I2C_Mode;                      //工作模式,选择I2C还是SMBus模式
  uint16_t I2C_DutyCycle;                 //时钟信号低电平/高电平的占空比,只有在I2C_ClockSpeed设为快速状态时生效 
  uint16_t I2C_OwnAddress1;               //自身器件地址，从机时使用
  uint16_t I2C_Ack;                       //ACK应答使能，I2C_AcknowledgeConfig()函数可以设置  
  uint16_t I2C_AcknowledgedAddress;       //I2C寻址模式,7位、10位
}I2C_InitTypeDef;
//I2C_Mode 工作模式
#define I2C_Mode_I2C                    ((uint16_t)0x0000)
#define I2C_Mode_SMBusDevice            ((uint16_t)0x0002)  
#define I2C_Mode_SMBusHost              ((uint16_t)0x000A)
//I2C_DutyCycle 高低电平占空比，由于高速模式，SDA线切换数据较快，可能来不及更改，所以增加SCL低电平时间
#define I2C_DutyCycle_16_9              ((uint16_t)0x4000) 
#define I2C_DutyCycle_2                 ((uint16_t)0xBFFF)
//I2C_Ack ACK应答使能 ，
#define I2C_Ack_Enable                  ((uint16_t)0x0400)
#define I2C_Ack_Disable                 ((uint16_t)0x0000)
//I2C_AcknowledgedAddress  I2C寻址模式,7位、10位
#define I2C_AcknowledgedAddress_7bit    ((uint16_t)0x4000)
#define I2C_AcknowledgedAddress_10bit   ((uint16_t)0xC000)
```

### 案例

硬件IIC

根据 发送接收序列图进行编写

```c
#define MPU6050_ADDRESS		0xD0
/*超时函数，在IIC发送或接收时，需要等待众多事件*/
void MPU6050_WaitEvent(I2C_TypeDef* I2Cx, uint32_t I2C_EVENT){	
	uint32_t Timeout;
	Timeout = 10000;
	while (I2C_CheckEvent(I2Cx, I2C_EVENT) != SUCCESS){
		Timeout --;
		if (Timeout == 0){
			break;
		}
	}
}
/* 硬件IIC 读*/
void MPU6050_WriteReg(uint8_t RegAddress, uint8_t Data){
	I2C_GenerateSTART(I2C2, ENABLE);	// 产生一个起始位
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_MODE_SELECT);	// 调用上方编写的超时函数，等待EV5事件
	
	I2C_Send7bitAddress(I2C2, MPU6050_ADDRESS, I2C_Direction_Transmitter);	// 发送设备地址，发送模式
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED);	// 等待EV6事件，注意是TRANSMITTER
	
	I2C_SendData(I2C2, RegAddress);											// 发送要写的地址
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_BYTE_TRANSMITTING);			// 等待EV8事件
	
	I2C_SendData(I2C2, Data);												// 发送数据
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_BYTE_TRANSMITTED);				// 等待EV8_2事件
	
	I2C_GenerateSTOP(I2C2, ENABLE);						// 停止位
}
/* 硬件IIC 写*/
uint8_t MPU6050_ReadReg(uint8_t RegAddress){
	uint8_t Data;
	
	I2C_GenerateSTART(I2C2, ENABLE);										// 产生一个起始位
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_MODE_SELECT);					// 等待EV5事件
	
	I2C_Send7bitAddress(I2C2, MPU6050_ADDRESS, I2C_Direction_Transmitter);	// 发送设备地址，发送模式
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED);	// 等待EV6事件
	
	I2C_SendData(I2C2, RegAddress);											// 发送要写的地址
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_BYTE_TRANSMITTED);				// 等待EV8事件
	
	I2C_GenerateSTART(I2C2, ENABLE);										// 重新产生起始条件
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_MODE_SELECT);					// 等待EV5事件
	
	I2C_Send7bitAddress(I2C2, MPU6050_ADDRESS, I2C_Direction_Receiver);		// 发送设备地址，接收模式
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED);
	
	I2C_AcknowledgeConfig(I2C2, DISABLE);									// 先不发送 响应
	I2C_GenerateSTOP(I2C2, ENABLE);											// 停止位
	
	MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_BYTE_RECEIVED);				// 等待EV7
	Data = I2C_ReceiveData(I2C2);											// 读取数据
	
	I2C_AcknowledgeConfig(I2C2, ENABLE);									// 启动响应位
	
	return Data;
}

void MPU6050_Init(void){
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C2, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10 | GPIO_Pin_11;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	
	I2C_InitTypeDef I2C_InitStructure;
	I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;				// IIC模式
	I2C_InitStructure.I2C_ClockSpeed = 50000;				// 50Khz 低速模式
	I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;		// 占空比 2：1
	I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;				// 发送响应位
	I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;	// 7位寻址模式
	I2C_InitStructure.I2C_OwnAddress1 = 0x00;				// 做从机时的地址
	I2C_Init(I2C2, &I2C_InitStructure);
	
	I2C_Cmd(I2C2, ENABLE);
	
	MPU6050_WriteReg(MPU6050_PWR_MGMT_1, 0x01);
	MPU6050_WriteReg(MPU6050_PWR_MGMT_2, 0x00);
	MPU6050_WriteReg(MPU6050_SMPLRT_DIV, 0x09);
	MPU6050_WriteReg(MPU6050_CONFIG, 0x06);
	MPU6050_WriteReg(MPU6050_GYRO_CONFIG, 0x18);
	MPU6050_WriteReg(MPU6050_ACCEL_CONFIG, 0x18);
}

uint8_t MPU6050_GetID(void){
	return MPU6050_ReadReg(MPU6050_WHO_AM_I);
}

void MPU6050_GetData(int16_t *AccX, int16_t *AccY, int16_t *AccZ, 
						int16_t *GyroX, int16_t *GyroY, int16_t *GyroZ)
{
	uint8_t DataH, DataL;
	
	DataH = MPU6050_ReadReg(MPU6050_ACCEL_XOUT_H);
	DataL = MPU6050_ReadReg(MPU6050_ACCEL_XOUT_L);
	*AccX = (DataH << 8) | DataL;
	
	DataH = MPU6050_ReadReg(MPU6050_ACCEL_YOUT_H);
	DataL = MPU6050_ReadReg(MPU6050_ACCEL_YOUT_L);
	*AccY = (DataH << 8) | DataL;
	
	DataH = MPU6050_ReadReg(MPU6050_ACCEL_ZOUT_H);
	DataL = MPU6050_ReadReg(MPU6050_ACCEL_ZOUT_L);
	*AccZ = (DataH << 8) | DataL;
	
	DataH = MPU6050_ReadReg(MPU6050_GYRO_XOUT_H);
	DataL = MPU6050_ReadReg(MPU6050_GYRO_XOUT_L);
	*GyroX = (DataH << 8) | DataL;
	
	DataH = MPU6050_ReadReg(MPU6050_GYRO_YOUT_H);
	DataL = MPU6050_ReadReg(MPU6050_GYRO_YOUT_L);
	*GyroY = (DataH << 8) | DataL;
	
	DataH = MPU6050_ReadReg(MPU6050_GYRO_ZOUT_H);
	DataL = MPU6050_ReadReg(MPU6050_GYRO_ZOUT_L);
	*GyroZ = (DataH << 8) | DataL;
}
```





# SPI

## SPI简介

- SPI（Serial Peripheral Interface 串行外围设备接口）是由Motorola公司开发的一种通用数据总线
- 四根通信线：SCK（Serial Clock）、MOSI（Master Output Slave Input）、MISO（Master Input Slave Output）、SS（Slave Select）
- 同步，全双工
- 支持总线挂载多设备（**一主多从**）
- 时钟是一个振荡信号，它告诉接收端在确切的时机对数据线上的信号进行采样。
- **产生时钟的一侧称为主机**，另一侧称为从机。总是只有一个主机（一般来说可以是微控制器/MCU），但是可以有多个从机（后面详细介绍）；
- 数据的采集时机可能是时钟信号的**上升沿（从低到高）或下降沿（从高到低）**。



整体的传输大概可以分为以下几个过程：

- 主机先将SS信号拉低，这样保证开始接收数据；
- 当接收端检测到时钟的边沿信号时，它将立即读取数据线上的信号，这样就得到了一位数据（1bit）;
- 由于时钟是随数据一起发送的，因此指定数据的传输速度并不重要，尽管设备将具有可以运行的最高速度（稍后我们将讨论选择合适的时钟边沿和速度）。
- 主机发送到从机时：主机产生相应的时钟信号，然后数据一位一位地将从MOSI信号线上进行发送到从机；
- 主机接收从机数据：如果从机需要将数据发送回主机，则主机将继续生成预定数量的时钟信号，并且从机会将数据通过MISO信号线发送；



**SPI分类**

对于FLASH来说：速度至上

| SPI分类              | 方向   | 描述                                                         |
| -------------------- | ------ | ------------------------------------------------------------ |
| Standard SPI 标准SPI | 全双工 | 标准SPI，通信线包含：片选CS、时钟线CLK、输入DI、输出DO。驱动SPI FLASH还需要写保护WR以及维持HOLD引脚 |
| Dual SPI 双线SPI     | 半双工 | 对标准SPI改进，DO和DI改成IO0和IO1，变为双向IO口。 一个周期内，通过数据线，传输2位数据 |
| Qual SPI 四线SPI     | 半双工 | 对Dual SPI改进，写保护WR和维持HOLD复用为数据IO口，为IO2和IO3。 一个周期内，通过数据线，传输4位数据 |



## 硬件电路

- 所有SPI设备的SCK、MOSI、MISO分别连在一起
- 主机另外引出多条SS控制线，分别接到各从机的SS引脚
- 输出引脚配置为**推挽输出**，输入引脚配置为**浮空或上拉输入**



SPI总线包括4条逻辑线，定义如下：

- MISO：Master input slave output 主机输入，从机输出（数据来自从机）；
- MOSI：Master output slave input 主机输出，从机输入（数据来自主机）；
- SCLK ： Serial Clock 串行时钟信号，由主机产生发送给从机；
- SS：Slave Select 片选信号，由主机发送，以控制与哪个从机通信，通常是低电平有效信号。

其他制造商可能会遵循其他命名规则，但是最终他们指的相同的含义。以下是一些常用术语；

- MISO也可以是SIMO，DOUT，DO，SDO或SO（在主机端）;
- MOSI也可以是SOMI，DIN，DI，SDI或SI（在主机端）;
- NSS也可以是CE，CS或SSEL;
- SCLK也可以是SCK;

![image-20230708122453286](STM32卷二.assets/image-20230708122453286.png)



当主机要发送数据到从机或接收从机数据时，要先将对应的SS置为低电平



## 移位示意图

![image-20230708123226045](STM32卷二.assets/image-20230708123226045.png)



![image-20230708124655185](STM32卷二.assets/image-20230708124655185.png)

1. 主机的波特率发生器产生时钟，这个时钟信号发送到从机，主机和从机根据对应的时钟信号发送接收数据
2. 当时钟信号发生第一次变化时，主机的移位寄存器将最左侧数据放到MOSI中，从机的移位寄存器将最左侧的数据放到MISO中
3. 当时钟信号再次发生变化时，主机将从机放到MISO中的数据添加到移位寄存器的最右侧，从机将主机放到MOSI中的数据添加到移位寄存器的最右侧
4. 循环往复

注意：只有一个移位寄存器

没有读和写的说法，因为实质上每次SPI是主从设备在交换数据。也就是说，你发一个数据必然会收到一个数据；你要收一个数据必须也要先发一个数据。

在只发送或只接收时，会存在浪费，

- 当只发送时，实际上也在接收，只是接收的数据不存放,一般接收(由发送端发出)的无用数据都是0x00或0xFF
- 当只接收时，实际上也在发送，只是发送的数据接收端不存放,一般发送无用数据(0x00或0xFF)

**数据发送**

![image-20230811182454198](STM32卷二.assets/image-20230811182454198.png)



**数据接收**

![image-20230811182513304](STM32卷二.assets/image-20230811182513304.png)







## SPI框图

### 硬件介绍

- STM32内部集成了硬件SPI收发电路，可以由硬件自动执行时钟生成、数据收发等功能，减轻CPU的负担
- **可配置8位/16位数据帧、高位先行/低位先行**
- 时钟频率： fPCLK / (2, 4, 8, 16, 32, 64, 128, 256)
- 支持多主机模型、主或从操作
- **可精简为半双工/单工通信**，可配置为使用MOSI和MISO进行双路读取
- 支持DMA
- 兼容I2S协议：数字音频信号协议
- STM32F103C8T6 硬件SPI资源：SPI1、SPI2
  - SPI1由APB2控制：72MHz
  - SPI2由APB1控制：36MHz

![image-20230708175647681](STM32卷二.assets/image-20230708175647681.png)

### 1.通讯引脚

SPI 一共有 4 个引脚，不同 SPI 对应的引脚也不同：

**注意**：硬件SPI的引脚是固定的，而软件SPI引脚可以随意，一般NSS引脚都是由软件模拟，所以不需要一定按引脚接线，其他引脚需要

| 引脚 | SPI1 | SPI2 |       SPI3       |
| :--: | :--: | :--: | :--------------: |
| NSS  | PA4  | PB12 | PA15下载口的TDI  |
| CLK  | PA5  | PB13 |  PB3下载口的TDO  |
| MISO | PA6  | PB14 | PB4下载口的NTRST |
| MOSI | PA7  | PB15 |       PB5        |

实际应用中，一般不使用 SPI 外设的标准 NSS 信号线，而是更简单地使用普通的 GPIO，软件控制它的电平输出，从而产生通讯起始和停止信号。

### 2.数据控制逻辑

- 这部分主要控制数据的接收和发送以及数据帧格式和MSB/LSB先行，和串口通讯类似，**SPI的收发数据也是通过缓冲区和移位寄存器来实现的**。MOSI和MISO都与移位寄存器相连以便传输数据，下面具体说明一下主机发送数据和接收数据的流程。
- 当发送完一帧数据的时候，“状态寄存器 SR”中的“TXE 标志位”会被置 1，表示传输完一帧，发送缓冲区已空；类似地，当接收完一帧数据的时候，“ RXNE 标志位”会被置 1，表示传输完一帧，接收缓冲区非空；
- 发送数据：地址和数据总线会在相应的地址上取到要发送的数据，将数据放入发送缓冲区，当向外发送数据时，移位寄存器会以发送缓冲区为数据源，一位一位的将数据发送出去。
- 接收数据：接收数据时，移位寄存器把数据线采样到的数据一位一位的传到接收缓冲区，再由总线读取接收缓冲区中的数据。
- **数据帧格式**：通过配置“控制寄存器CR1”的“DFF为”可以控制数据帧格式为8位还是16位，即一次接收或发送数据的大小。(16位就是一次性发2个8位)
- **先行位**：通过配置“控制寄存器CR1”的“LSBFIRST 位”可选择 MSB（最高有效位） 先行还是 LSB（最低有效位） 先行。Most/Least Significant Bit

**发送缓冲器空闲标志(TXE)** 

- 此标志为’1’时表明发送缓冲器为空，可以写下一个待发送的数据进入缓冲器中。当写入SPI_DR时，TXE标志被**自动清除**。

**接收缓冲器非空(RXNE)** 

- 此标志为’1’时表明在接收缓冲器中包含有效的接收数据。读SPI数据寄存器可以清除此标志。

**2.x** ：主要配置的是多主多从，MOSI与MISO进行交换

### 3.时钟控制逻辑

这一块的内容主要是配置SCK的时钟频率和SPI的通讯模式（CPOL和CPHA）。 时钟频率的配置：波特率发生器通过控制“控制寄存器CR1”中的BR[2:0]三个位来配置fpclk的分频因子，对fpclk分频后的频率就是SCK的时钟频率，具体配

| BR[0:2] | 分频结果(SCK 频率) |
| :-----: | :----------------: |
|   000   |      fpclk/2       |
|   001   |      fpclk/4       |
|   010   |      fpclk/8       |
|   011   |      fpclk/16      |
|   100   |      fpclk/32      |
|   101   |      fpclk/64      |
|   110   |     fpclk/128      |
|   111   |     fpclk/256      |

### 4.整体逻辑控制

在外设工作时，控制逻辑会根据外设的工作状态修改“状态寄存器(SR)”，我们只要读取状态寄存器相关的寄存器位，就可以了解 SPI 的工作状态了。除此之外，控制逻辑还根据要求，负责控制产生 SPI 中断信号、DMA 请求及控制NSS 信号线，不过NSS信号线我们时一般是连接GPIO口，通过软件来控制电平输出，从而产生起始信号和停止信号。



### SPI基本结构

![image-20230708180740267](STM32卷二.assets/image-20230708180740267.png)





## 工作机制

### 起始条件

- 起始条件：SS从高电平切换到低电平(片选)
- 终止条件：SS从低电平切换到高电平
- 即选中从机时为开启，不选中从机时为关闭

### **相关缩写**

SPI的极性Polarity和相位Phase，最常见的写法是CPOL和CPHA，不过也有一些其他写法，简单总结如下：

1. CKPOL (Clock Polarity) = CPOL = POL = Polarity = 时钟极性
2. CKPHA (Clock Phase) = CPHA = PHA = Phase = 时钟相位
3. SCK = SCLK = SPI的时钟
4. Edge = 边沿，即时钟电平变化的时刻，即上升沿(rising edge)或者下降沿(falling edge)
   - Leading edge=前一个边沿=第一个边沿，对于开始电压是1，那么就是1变成0的时候，对于开始电压是0，那么就是0变成1的时候；
   - Trailing edge=后一个边沿=第二个边沿，对于开始电压是1，那么就是0变成1的时候（即在第一次1变成0之后，才可能有后面的0变成1），对于开始电压是0，那么就是1变成0的时候；

### 时钟极性CPOL

先说什么是SCLK时钟的空闲时刻，其就是当SCLK在数发送8个bit比特数据之前和之后的状态，于此对应的，SCLK在发送数据的时候，就是正常的工作的时候，有效active的时刻了。

先说英文，其精简解释为：Clock Polarity = IDLE state of SCK。

再用中文详解：

- SPI的CPOL，表示当SCLK空闲idle的时候，其电平的值是低电平0还是高电平1：
  - **CPOL=0**，表示当SCLK=0时处于空闲态，所以有效状态就是SCLK处于高电平时
  - **CPOL=1**，表示当SCLK=1时处于空闲态，所以有效状态就是SCLK处于低电平时

### 时钟相位CPHA

首先说明一点，capture strobe = latch = read = sample，都是表示数据采样，数据有效的时刻。相位，对应着数据采样是在第几个边沿（edge），是第一个边沿还是第二个边沿，0对应着第一个边沿，1对应着第二个边沿。

- **CPHA=0**，在时钟的第一个跳变沿（上升沿或下降沿）进行数据采样(接收)。在第2个边沿发送数据
- **CPHA=1**，在时钟的第二个跳变沿（上升沿或下降沿）进行数据采样(接收)。在第1个边沿发送数据

### 模式

| SPI工作模式 | CPOL | CPHA | SCL空闲状态 | 采样边沿 | 采样时刻 |
| ----------- | ---- | ---- | ----------- | -------- | -------- |
| 0           | 0    | 0    | 低电平      | 上升沿   | 奇数边沿 |
| 1           | 0    | 1    | 低电平      | 下降沿   | 偶数边沿 |
| 2           | 1    | 0    | 高电平      | 下降沿   | 奇数边沿 |
| 3           | 1    | 1    | 高电平      | 上升沿   | 偶数边沿 |

工作站基本采用模式0 或3，但是模式1方便理解

**Mode0**：

- 交换一个字节（模式0）
- CPOL=0：空闲状态时，SCK为低电平
- CPHA=0：SCK第一个边沿移入数据（将MOSI和MISI中的值放入寄存器），第二个边沿移出数据（将寄存器中的值放入MOSI和MISI中）

![image-20230708130014868](STM32卷二.assets/image-20230708130014868.png)

当检测到第一个边沿之前，先将数据放到MOSI和MISI中，当检测到第一个边沿时(上升沿),将MOSI和MISI中的数据放入寄存器，当检测到第二个边沿时(下降沿),将数据放入MOSI和MISI中



**Mode1**：

- 交换一个字节（模式1）
- CPOL=0：空闲状态时，SCK为低电平
- CPHA=1：SCK第一个边沿移出数据，第二个边沿移入数据

![image-20230708130043716](STM32卷二.assets/image-20230708130043716.png)

当检测到第一个边沿时(上升沿),将数据放入MOSI和MISI中，当检测到第二个边沿时(下降沿),将MOSI和MISI中的数据放入寄存器





**Mode2**：

- 交换一个字节（模式2）
- CPOL=1：空闲状态时，SCK为高电平
- CPHA=0：SCK第一个边沿移入数据，第二个边沿移出数据

![image-20230708130115675](STM32卷二.assets/image-20230708130115675.png)

**Mode3**：

- 交换一个字节（模式3）
- CPOL=1：空闲状态时，SCK为高电平
- CPHA=1：SCK第一个边沿移出数据，第二个边沿移入数据

![image-20230708130140911](STM32卷二.assets/image-20230708130140911.png)

### SPI时序

**发送指令**

向SS指定的设备，发送指令（0x06）

![image-20230708130832748](STM32卷二.assets/image-20230708130832748.png)

**指定地址写**

向SS指定的设备，发送写指令（0x02），

随后在指定地址（Address[23:0]）下，写入指定数据（Data）

![image-20230708130927229](STM32卷二.assets/image-20230708130927229.png)

由于地址为24为，而每次只能传8为，所以地址需要3次传送

**指定地址读**

向SS指定的设备，发送读指令（0x03），

随后在指定地址（Address[23:0]）下，读取从机数据（Data）

![image-20230708130949329](STM32卷二.assets/image-20230708130949329.png)



## 通信过程

### **主模式全双工连续传输**

![image-20230708180959653](STM32卷二.assets/image-20230708180959653.png)

首先需要知道一个大概的过程：主机输出的数据都要通过缓冲区（对应数据寄存器`SPI_DR`）才能进入移位寄存器，最后再发送；主机读入的数据都要通过移位寄存器，再移入缓冲区。因此，**缓冲区和移位寄存器中的数据在同一时刻是不一样的，它们正好相差了一次通信时间**。



**从主机发送数据到从机的详细过程（以 CPHA=1、CPOL=1 为例）**

- 往发送缓冲区（即数据寄存器SPI_DR）写入一个数据（比如图中的 0XF1），此时触发 SCK 信号进入忙碌状态。此时状态寄存器SPI_SR的BSY（忙标志，Busy flag）位为 1，表示 SPI 开始通信；TXE（发送缓冲为空，Transmit buffer empty）位为 0，表示发送缓冲非空。
- 由于移位寄存器还没有数据，因此发送缓冲区很快将数据 0xF1 转移到移位寄存器中。此时TXE位为 1，表示发送缓冲为空。移位寄存器将数据一位一位传到 MOSI 线。
- 等待至TXE位为 1后，软件往发送缓冲区写入第二个数据（比如 0xF2），TXE位变为 0，表示发送缓冲非空。由于移位寄存器还没将前一个数据 0xF1 全部发送出去，因此新数据 0xF2 暂时存到缓冲区。
- 前一个数据发送完毕后，新数据 0xF2 转移到移位寄存器中，此时TXE位为 1，表示发送缓冲为空。移位寄存器将数据一位一位传到 MOSI 线。
- 如此往复：等待TXE位由0变为 1，软件往发送缓冲区写入新数据，前一个数据发送完，新数据就往移位寄存器补，并将其一位一位发出去。发送出去的数据总比新写入缓冲区的数据少一次发送周期。



**从从机接收数据到主机的详细过程（以 CPHA=1、CPOL=1 为例）**

- 必须先使 SPI 启动（即让 SCK 处于忙碌状态），才能接收数据。因此，需要先往发送缓冲随意写入一个数据，以触发 SCK 信号进入忙碌状态。写入数据后，状态寄存器SPI_SR的BSY位为 1，表示 SPI 开始通信；RXNE（接收缓冲非空，Receive buffer not empty）位为 0，表示接收缓冲为空。
- 移位寄存器一位一位接收到从 MISO 来的数据 0xA1，将数据转移到接收缓冲区中，然后进行下一次数据接收；此时RXNE位为 1，表示接收缓冲非空，表明软件可以开始读取数据 0xA1。
- 待数据读取完后，RXNE位为 0，表示接收缓冲为空。此时移位寄存器将新数据 0xA2 转移到接收缓冲区中，RXNE位为 1，表示接收缓冲非空，表明软件可以开始读取数据 0xA2
- 如此往复：移位寄存器一位位接收数据，然后将数据转移到接收缓冲区，然后继续接收下一次数据；软件等待RXNE位由 0 变为 1 后，从接收缓冲区读取数据。接收到的数据总比新读取缓冲区的数据多一个发送周期。

### 非连续传输

![image-20230708181741408](STM32卷二.assets/image-20230708181741408.png)



1. 先等待 发送缓冲器空闲标志(TXE) 为空(1)，即缓冲器SPI_DR中为空
   - TXE当为0时，一直等待
   - TXE当为1时 --> 写入数据到缓冲器SPI_DR，此时缓冲器SPI_DR有数据 --> TXE的值变为0
2. 移位寄存器再从SPI_DR取值，取完后缓冲器SPI_DR为空(TXE为1)，但是现在不着急将数据再次放到缓冲器SPI_DR中
   - 发送的同时，也会接收，发送完成也就代表接收完成，当接收完一个字节后 接收缓冲器非空(RXNE) 为1

**注意：** 要想接收，必须先发送，因为只有发送，才能触发时序的生成

## 相关寄存器

| **寄存器** | **名称**       | **作用**                                            |
| ---------- | -------------- | --------------------------------------------------- |
| SPI_CR1    | SPI控制寄存器1 | 用于配置SPI工作参数                                 |
| SPI_SR     | SPI状态寄存器  | 用于查询当前SPI传输状态（TXE、RXNE）                |
| SPI_DR     | SPI数据寄存器  | 用于存放待发送数据或接收数据，有两个缓冲区（TX/RX） |

**SPI 控制寄存器 1（SPI_CR1）**

![image-20230811182907829](STM32卷二.assets/image-20230811182907829.png)

该寄存器控制着 SPI 很多相关信息，包括主设备模式选择，传输方向，数据格式，时钟极性、时钟相位和使能等。



**SPI 状态寄存器（SPI_SR）**

![image-20230811183020434](STM32卷二.assets/image-20230811183020434.png)该寄存器是查询当前 SPI 的状态的，我们在实验中用到的是 TXE 位和 RXNE 位，即发送完成和接收完成是否的标记。

**SPI 数据寄存器（SPI_DR）**

![image-20230811183044676](STM32卷二.assets/image-20230811183044676.png)

该寄存器是 SPI 数据寄存器，是一个双寄存器，包括了发送缓存和接收缓存。当向该寄存器写数据的时候，SPI 就会自动发送，当收到数据的时候，也是存在该寄存器内。

## HAL库

|           驱动函数           |  关联寄存器   |     功能描述      |
| :--------------------------: | :-----------: | :---------------: |
| __HAL_RCC_SPIx_CLK_ENABLE(…) |  RCC_APB2ENR  |   使能SPIx时钟    |
|       HAL_SPI_Init(…)        |    SPI_CR1    |     初始化SPI     |
|      HAL_SPI_MspInit(…)      |  初始化回调   | 初始化SPI相关引脚 |
|     HAL_SPI_Transmit(…)      | SPI_DR/SPI_SR |      SPI发送      |
|      HAL_SPI_Receive(…)      | SPI_DR/SPI_SR |      SPI接收      |
|  HAL_SPI_TransmitReceive(…)  | SPI_DR/SPI_SR |    SPI接收发送    |
|     __HAL_SPI_ENABLE(…)      | SPI_CR1(SPE)  |    使能SPI外设    |
|     __HAL_SPI_DISABLE(…)     | SPI_CR1(SPE)  |    失能SPI外设    |

```
typedef struct __SPI_HandleTypeDef{
    SPI_TypeDef                *Instance;
    SPI_InitTypeDef            Init;
    DMA_HandleTypeDef          *hdmatx;
    DMA_HandleTypeDef          *hdmarx;
} SPI_HandleTypeDef;
typedef struct{
    uint32_t Mode               /* SPI模式 */
    uint32_t Direction          /* 工作方式 */
    uint32_t DataSize           /* 帧格式 */
    uint32_t CLKPolarity        /* 时钟极性 */
    uint32_t CLKPhase           /* 时钟相位 */
    uint32_t NSS                /* SS控制方式 */
    uint32_t BaudRatePrescaler  /* SPI波特率预分频值 */
    uint32_t FirstBit           /* 数据传输顺序 */
    uint32_t TIMode             /* 帧格式：Motorola / TI  */
    uint32_t CRCCalculation     /* 设置硬件CRC校验 */
    uint32_t CRCPolynomial      /* 设置CRC校验多项式，1~65535 默认为7 */
    …
} SPI_InitTypeDef;
/* SPI模式 */
#define SPI_MODE_SLAVE      /* 从机模式 */
#define SPI_MODE_MASTER     /* 主机模式 */
/* 工作方式 */
#define SPI_DIRECTION_2LINES            /* 双线全双工 */
#define SPI_DIRECTION_2LINES_RXONLY
#define SPI_DIRECTION_1LINE
/* 帧格式 */
#define SPI_DATASIZE_8BIT
#define SPI_DATASIZE_16BIT
/* 时钟极性 */
#define SPI_POLARITY_LOW        /* 空闲低电平 */
#define SPI_POLARITY_HIGH       /* 空闲高电平 */
/* 时钟相位 */
#define SPI_PHASE_1EDGE         /* 第一个边沿 */
#define SPI_PHASE_2EDGE         /* 第二个边沿 */

/* SS控制方式 */
#define SPI_NSS_SOFT            /* 软件片选 */
#define SPI_NSS_HARD_INPUT
#define SPI_NSS_HARD_OUTPUT
/* SPI波特率预分频值 */    
#define SPI_BAUDRATEPRESCALER_2
#define SPI_BAUDRATEPRESCALER_4
#define SPI_BAUDRATEPRESCALER_8
#define SPI_BAUDRATEPRESCALER_16
#define SPI_BAUDRATEPRESCALER_32
#define SPI_BAUDRATEPRESCALER_64
#define SPI_BAUDRATEPRESCALER_128
#define SPI_BAUDRATEPRESCALER_25
/* 数据传输顺序 */      
#define SPI_FIRSTBIT_MSB        /* 高位先行 */
#define SPI_FIRSTBIT_LSB        /* 低位先行*/

/* 帧格式：Motorola / TI  默认摩托罗拉*/
#define SPI_TIMODE_DISABLE

/* 设置硬件CRC校验 */      
#define SPI_CRCCALCULATION_DISABLE
#define SPI_CRCCALCULATION_ENABLE
```

### 软件读写

读写WQ25QXX

![11-1 软件SPI读写W25Q64](STM32卷二.assets/11-1 软件SPI读写W25Q64.jpg)

SPI模块

```
#define SPI_GPIO_PORT       GPIOA
#define SPI_GPIO_PIN_CS     GPIO_PIN_4
#define SPI_GPIO_PIN_CLK    GPIO_PIN_5
#define SPI_GPIO_PIN_MISO   GPIO_PIN_6
#define SPI_GPIO_PIN_MOSI   GPIO_PIN_7

#define SPI_CS(X)  do{ X ? HAL_GPIO_WritePin(GPIOA,SPI_GPIO_PIN_CS ,GPIO_PIN_SET) : HAL_GPIO_WritePin(GPIOA,SPI_GPIO_PIN_CS ,GPIO_PIN_RESET); }while(0)

#define SPI_CLK(X)  do{ X ? HAL_GPIO_WritePin(GPIOA,SPI_GPIO_PIN_CLK,GPIO_PIN_SET) : HAL_GPIO_WritePin(GPIOA,SPI_GPIO_PIN_CLK,GPIO_PIN_RESET); }while(0)

#define SPI_W_MOSI(X) do{ X ? HAL_GPIO_WritePin(GPIOA,SPI_GPIO_PIN_MOSI,GPIO_PIN_SET) : HAL_GPIO_WritePin(GPIOA,SPI_GPIO_PIN_MOSI,GPIO_PIN_RESET); }while(0)

#define SPI_R_MISO()  HAL_GPIO_ReadPin(GPIOA,SPI_GPIO_PIN_MISO)


void SPI_Init(void){   
    __HAL_RCC_GPIOA_CLK_ENABLE();
    // 输出引脚配置为推挽输出，输入引脚配置为浮空或上拉输入
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStructure.Pin = SPI_GPIO_PIN_CS | SPI_GPIO_PIN_CLK |SPI_GPIO_PIN_MOSI;
    GPIO_InitStructure.Pull = GPIO_NOPULL;
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;
    HAL_GPIO_Init(SPI_GPIO_PORT,&GPIO_InitStructure);
    
    GPIO_InitStructure.Mode = GPIO_MODE_INPUT;
    GPIO_InitStructure.Pin = SPI_GPIO_PIN_MISO;
    
    HAL_GPIO_Init(SPI_GPIO_PORT,&GPIO_InitStructure);
    
    SPI_CS(1);      // 初始化SS线为高电平，表示不选中从机
    SPI_CLK(0);     // 空闲转态
}
/* 开启SPI */
void SPI_Start(void){
    SPI_CS(0);
}
/* 停止SPI */
void SPI_Stop(void){
    SPI_CS(1);
}
/**
*   SPI输出和读入，由于读的时候一定有写，写的时候一定有读，所以读写在一起
*   使用移位寄存器方式
**/
uint8_t SPI_SendReadByte(uint8_t data){  
    for(uint8_t i = 0;i < 8;i++){
        SPI_W_MOSI( ( data & 0x80) >> 7);   // 将最高为输出
        SPI_CLK(1);
        data <<= 1;                         // 移除最高位
        if(SPI_R_MISO() == 1) data++;       // 如果读入为1，则将1加入到最低位
        SPI_CLK(0);   
    }  
    return data;
}
```

W25QXX模块

```
void W25QXX_Init(void){
    SPI_Init();
}
void W25QXX_ReadID(uint8_t *MID,uint16_t *DID){ // 读取厂商ID，和设备ID
    SPI_Start();
    SPI_SendReadByte(0X9F);                     // 向W25Q64发送0x9f，查询厂商ID，和设备ID

    /*
    * 当发送0x9f后，W25Q64会发出1个8位的厂商ID，再通过两次发出一个16为的设备ID
    * 但是SPI没有单独接收或发送的情况，只有发送和接收同时进行，所以主机要想接收到厂商ID和设备ID
    * 就必须连续发送3个8为数据，从而接收3个8位数据
    */
    *MID = SPI_SendReadByte(0XFF);              // 通过发送0xff这种无效数据，接收到厂商ID
    *DID = SPI_SendReadByte(0XFF);              // 通过发送0xff这种无效数据，接收到设备ID的高八位
    *DID <<= 8;                                 // 左移8位
    *DID |= SPI_SendReadByte(0XFF);             // 合并低八位
    SPI_Stop();
}
// 写使能
void W25QXX_WriteEnable(void){
    SPI_Start();
    SPI_SendReadByte(0X06);                     // 06h为写使能，W2W25Q64不会再发送数据到主机
    SPI_Stop();
}
// 读状态寄存器，主要是判断芯片是否在忙转态
void W25QXX_ReadStatue(void){
    uint32_t Timeout = 1000000;
    
    SPI_Start();
    SPI_SendReadByte(0X05);                     // 05h为读状态寄存器，W2W25Q64会再发送1个数据到主机，读取最低位即可
    while((SPI_SendReadByte(0XFF) & 0x01) == 0x01){         // 如果读取到的数据为1，则表示芯片正忙
        if(--Timeout == 0) break;                           // 最多等1000000自减
    }
    SPI_Stop();
    
}
// 指定页写入
void W25QXX_PageWrite(uint32_t ADDR,uint8_t *pData,uint16_t lenght){
    W25QXX_WriteEnable();                       // 写操作需要先使能
    SPI_Start();
    //发送0x02 表示指定页写入，后面还需要24为的地址，再之后是写入的内容
    SPI_SendReadByte(0X02);         // 02h为写入页，W2W25Q64会再发送1个数据到主机，读取最低位即可 
    
    SPI_SendReadByte(ADDR >> 16);   // 右移16位，就是23~16位，注意不是Address >>= 16，>> 仅仅为右移不赋值
    SPI_SendReadByte(ADDR >> 8);    
    SPI_SendReadByte(ADDR);         // 强制类型转换，SPI_SendReadByte(uint8_t data) uint32_t -> uint8_t会保留下低八位
    
    for(uint8_t i = 0;i < lenght;i++){
        SPI_SendReadByte(pData[i]);
    }
    
    SPI_Stop();
    W25QXX_ReadStatue();            // 等待芯片为空闲状态时再放行
}
// 扇区擦除
void W25QXX_SectorErase(uint32_t ADDR){
    W25QXX_WriteEnable();           // 需要先使能
    SPI_Start();    
    SPI_SendReadByte(0X20);         //发送0x20表示扇区擦除，需要24为地址
    
    SPI_SendReadByte(ADDR >> 16);
    SPI_SendReadByte(ADDR >> 8);
    SPI_SendReadByte(ADDR);
    
    SPI_Stop();
    W25QXX_ReadStatue();
}
// 读数据
void W25QXX_ReadData(uint32_t ADDR,uint8_t *pData,uint16_t lenght){
//    W25QXX_WriteEnable();         // 读数据不需要使能
    SPI_Start();
    SPI_SendReadByte(0X03); 
    
    SPI_SendReadByte(ADDR >> 16);
    SPI_SendReadByte(ADDR >> 8);
    SPI_SendReadByte(ADDR);
    
    for(uint8_t i = 0;i < lenght;i++){
        pData[i] = SPI_SendReadByte(0XFF);
    } 
    SPI_Stop();
}
```

main

```
uint8_t SendData[4] = {1,2,3,4};
#define SendSize    sizeof(SendData)
uint8_t ReadData[SendSize];
int main(void)
{
    HAL_Init();                         /* 初始化HAL库 */
    sys_stm32_clock_init(RCC_PLL_MUL9); /* 设置时钟, 72Mhz */
    delay_init(72);                     /* 延时初始化 */
    usart_init(115200);                 /* 串口初始化为115200 */
    led_init();                         /* 初始化LED */
    lcd_init();                         /* 初始化LCD */
    key_init();                         /* 初始化key */
    
    lcd_show_string(30, 110, 200, 16, 16, "Mid:", RED);
    lcd_show_string(30, 130, 200, 16, 16, "DID:", RED);
    
    lcd_show_string(30, 150, 200, 16, 16, "Send Data:", RED);
    lcd_show_string(30, 180, 200, 16, 16, "Read Data:", RED);
    W25QXX_Init();
    uint8_t mid=0;
    uint16_t did=0;
    while (1){  
        W25QXX_ReadID(&mid,&did);                  // 读取设备ID
        lcd_show_num(132, 110,mid,4,16,BLUE);
        lcd_show_num(132, 130,did,8,16,BLUE);  
        
        for(int i = 0;i < SendSize;i++){
            lcd_show_num(132 + i * 17, 150,SendData[i],1,16,BLUE);
        }      
        //对0x00000000地址的扇区进行擦除，即填充0xff
        W25QXX_SectorErase(0x00000000); 
        
        W25QXX_PageWrite(0x00000000,SendData,4);    // 指定页写
        W25QXX_ReadData(0x00000000,ReadData,4);     // 将数据读出
        for(int i = 0;i < SendSize;i++){
            lcd_show_num(132 + i * 17, 180,ReadData[i],1,16,BLUE);
        }
        
        for(int i = 0;i < SendSize;i++){
            SendData[i]++;
        }
    }
}
```

### 硬件读写

与软件读写相比，只需要更改SPI模块

```c
#define SPI_GPIO_PORT       GPIOA
#define SPI_GPIO_PIN_CS     GPIO_PIN_4
#define SPI_GPIO_PIN_CLK    GPIO_PIN_5
#define SPI_GPIO_PIN_MISO   GPIO_PIN_6
#define SPI_GPIO_PIN_MOSI   GPIO_PIN_7

#define SPI_CS(X)  do{ X ? HAL_GPIO_WritePin(GPIOA,SPI_GPIO_PIN_CS ,GPIO_PIN_SET) : HAL_GPIO_WritePin(GPIOA,SPI_GPIO_PIN_CS ,GPIO_PIN_RESET); }while(0)

SPI_HandleTypeDef SPI_HandleStructure;

void SPI_Init(void){
    SPI_HandleStructure.Instance = SPI1;				
    SPI_HandleStructure.Init.Mode = SPI_MODE_MASTER;				/* 设置SPI模式（主机模式） */
    SPI_HandleStructure.Init.Direction =  SPI_DIRECTION_2LINES;		/* 设置SPI工作方式（全双工） */
    SPI_HandleStructure.Init.DataSize = SPI_DATASIZE_8BIT;			/* 设置数据格式（8bit长度） */
    SPI_HandleStructure.Init.CLKPolarity = SPI_POLARITY_LOW;		/* 设置时钟极性（CPOL = 0） */
    SPI_HandleStructure.Init.CLKPhase = SPI_PHASE_1EDGE;			/* 设置时钟相位（CPHA = 0）第一个边沿 */
    SPI_HandleStructure.Init.NSS = SPI_NSS_SOFT;					/* 设置片选方式（软件片选，自定义GPIO） */
    SPI_HandleStructure.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_256;	/* 设置SPI时钟波特率分频（256分频） */
    SPI_HandleStructure.Init.FirstBit = SPI_FIRSTBIT_MSB;			/* 设置大小端模式（MSB高位在前） */
    SPI_HandleStructure.Init.TIMode = SPI_TIMODE_DISABLE;			/* 设置帧格式（关闭TI模式） */
    SPI_HandleStructure.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;	/* 设置CRC校验（关闭CRC校验） */
    SPI_HandleStructure.Init.CRCPolynomial = 7;								/* 设置CRC校验多项式（范围：1~65535） */

    HAL_SPI_Init(&SPI_HandleStructure);

    SPI_CS(1); /* 不选择CS */
}
void HAL_SPI_MspInit(SPI_HandleTypeDef *hspi){
    __HAL_RCC_SPI1_CLK_ENABLE();
    if(hspi->Instance == SPI1){
        __HAL_RCC_GPIOA_CLK_ENABLE();
            
        GPIO_InitTypeDef GPIO_InitStructure;
         
        GPIO_InitStructure.Mode = GPIO_MODE_OUTPUT_PP;
        GPIO_InitStructure.Pin = SPI_GPIO_PIN_CS;
        GPIO_InitStructure.Pull = GPIO_NOPULL;
        GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;
        HAL_GPIO_Init(SPI_GPIO_PORT,&GPIO_InitStructure);
        
        GPIO_InitStructure.Mode = GPIO_MODE_AF_PP;
        GPIO_InitStructure.Pin = SPI_GPIO_PIN_CLK | SPI_GPIO_PIN_MOSI;
        HAL_GPIO_Init(SPI_GPIO_PORT,&GPIO_InitStructure);
        
        GPIO_InitStructure.Mode = GPIO_MODE_INPUT;
        GPIO_InitStructure.Pin = SPI_GPIO_PIN_MISO;
        
        HAL_GPIO_Init(SPI_GPIO_PORT,&GPIO_InitStructure); 
        
//        __HAL_SPI_ENABLE(&g_spi2_handler);
    }
}
/* 开启SPI */
void SPI_Start(void){
    SPI_CS(0);
}
/* 停止SPI */
void SPI_Stop(void){
    SPI_CS(1);
}
uint8_t SPI_SendReadByte(uint8_t data){  
    uint8_t receive;
    HAL_SPI_TransmitReceive(&SPI_HandleStructure,&data,&receive,1,1000);
    return receive;
}
```



## 标准库

### 常用函数

```
//初始化，I2S为数字音频信号协议
void SPI_Init(SPI_TypeDef* SPIx, SPI_InitTypeDef* SPI_InitStruct);
void I2S_Init(SPI_TypeDef* SPIx, I2S_InitTypeDef* I2S_InitStruct);
void SPI_StructInit(SPI_InitTypeDef* SPI_InitStruct);
void I2S_StructInit(I2S_InitTypeDef* I2S_InitStruct);
//使能
void SPI_Cmd(SPI_TypeDef* SPIx, FunctionalState NewState);
void I2S_Cmd(SPI_TypeDef* SPIx, FunctionalState NewState);
//中断
void SPI_I2S_ITConfig(SPI_TypeDef* SPIx, uint8_t SPI_I2S_IT, FunctionalState NewState);
//使能DMA，与DMA配合
void SPI_I2S_DMACmd(SPI_TypeDef* SPIx, uint16_t SPI_I2S_DMAReq, FunctionalState NewState);

//接收、发送数据
void SPI_I2S_SendData(SPI_TypeDef* SPIx, uint16_t Data);
uint16_t SPI_I2S_ReceiveData(SPI_TypeDef* SPIx);

void SPI_NSSInternalSoftwareConfig(SPI_TypeDef* SPIx, uint16_t SPI_NSSInternalSoft);
void SPI_SSOutputCmd(SPI_TypeDef* SPIx, FunctionalState NewState);
void SPI_DataSizeConfig(SPI_TypeDef* SPIx, uint16_t SPI_DataSize);
void SPI_TransmitCRC(SPI_TypeDef* SPIx);
void SPI_CalculateCRC(SPI_TypeDef* SPIx, FunctionalState NewState);
uint16_t SPI_GetCRC(SPI_TypeDef* SPIx, uint8_t SPI_CRC);
uint16_t SPI_GetCRCPolynomial(SPI_TypeDef* SPIx);
void SPI_BiDirectionalLineConfig(SPI_TypeDef* SPIx, uint16_t SPI_Direction);
// 与标志位相关：
/**
  *     @arg SPI_I2S_FLAG_TXE: Transmit buffer empty flag.
  *     @arg SPI_I2S_FLAG_RXNE: Receive buffer not empty flag.
  *     …………
**/
FlagStatus SPI_I2S_GetFlagStatus(SPI_TypeDef* SPIx, uint16_t SPI_I2S_FLAG);
void SPI_I2S_ClearFlag(SPI_TypeDef* SPIx, uint16_t SPI_I2S_FLAG);

// 与中断标志位相关：
ITStatus SPI_I2S_GetITStatus(SPI_TypeDef* SPIx, uint8_t SPI_I2S_IT);
void SPI_I2S_ClearITPendingBit(SPI_TypeDef* SPIx, uint8_t SPI_I2S_IT);
typedef struct
{
 uint16_t SPI_Direction;         /*设置 SPI 的单双向模式 */
 uint16_t SPI_Mode;              /*设置 SPI 的主/从机端模式 */
 uint16_t SPI_DataSize;          /*设置 SPI 的数据帧长度，可选 8/16 位 */
 uint16_t SPI_CPOL;              /*设置时钟极性 CPOL，可选高/低电平*/
 uint16_t SPI_CPHA;              /*设置时钟相位，可选奇/偶数边沿采样 */
 uint16_t SPI_NSS;               /*设置 NSS 引脚由 SPI 硬件控制还是软件控制*/
 uint16_t SPI_BaudRatePrescaler; /*设置时钟分频因子， fpclk/分频数=fSCK */
 uint16_t SPI_FirstBit;          /*设置 MSB/LSB 先行 */
 uint16_t SPI_CRCPolynomial;     /*设置 CRC 校验的表达式 这是 SPI 的 CRC 校验中的多项式，若我们使用 CRC 校验时，就使用这个成员的参数(多项式)，来计算 CRC 的值。*/
} SPI_InitTypeDef;
// uint16_t SPI_BaudRatePrescaler 本成员设置波特率分频因子，分频后的时钟即为 SPI 的 SCK 信号线的时钟频率。这个成员参数可设置为 fpclk 的 2、 4、 6、 8、 16、 32、 64、 128、 256 分频。
#define SPI_BaudRatePrescaler_2         ((uint16_t)0x0000)
#define SPI_BaudRatePrescaler_4         ((uint16_t)0x0008)
#define SPI_BaudRatePrescaler_8         ((uint16_t)0x0010)
#define SPI_BaudRatePrescaler_16        ((uint16_t)0x0018)
#define SPI_BaudRatePrescaler_32        ((uint16_t)0x0020)
#define SPI_BaudRatePrescaler_64        ((uint16_t)0x0028)
#define SPI_BaudRatePrescaler_128       ((uint16_t)0x0030)
#define SPI_BaudRatePrescaler_256       ((uint16_t)0x0038)

//SPI_CPHA时钟相位
#define SPI_CPHA_1Edge                  ((uint16_t)0x0000)
#define SPI_CPHA_2Edge                  ((uint16_t)0x0001)
//SPI_CPOL时钟极性
#define SPI_CPOL_Low                    ((uint16_t)0x0000)
#define SPI_CPOL_High                   ((uint16_t)0x0002)

//SPI_CRCPolynomial

//SPI_DataSize本成员可以选择 SPI 通讯的数据帧大小是为 8 位(SPI_DataSize_8b)还是 16 位
#define SPI_DataSize_16b                ((uint16_t)0x0800)
#define SPI_DataSize_8b                 ((uint16_t)0x0000)

//SPI_Direction 本成员设置 SPI 的通讯方向
#define SPI_Direction_2Lines_FullDuplex ((uint16_t)0x0000)    //双线全双工
#define SPI_Direction_2Lines_RxOnly     ((uint16_t)0x0400)    //双线只接
#define SPI_Direction_1Line_Rx          ((uint16_t)0x8000)    //单线只接收
#define SPI_Direction_1Line_Tx          ((uint16_t)0xC000)    //单线只发送

//SPI_Mode本成员设置 SPI 工作在主机模式(SPI_Mode_Master)或从机模式(SPI_Mode_Slave )，这两个模式的最大区别为 SPI 的 SCK 信号线的时序， SCK 的时序是由通讯中的主机产生的。若被配置为从机模式， STM32 的 SPI 外设将接受外来的 SCK 信号。
#define SPI_Mode_Master                 ((uint16_t)0x0104)
#define SPI_Mode_Slave                  ((uint16_t)0x0000)
//SPI_FirstBit设置高位先行还是低位先行
#define SPI_FirstBit_MSB                ((uint16_t)0x0000)    //高位数据在前
#define SPI_FirstBit_LSB                ((uint16_t)0x0080)    //低位数据在前
//SPI_NSS本成员配置 NSS 引脚的使用模式，在硬件模式中的 SPI 片选信号由 SPI 硬件自动产生，而软件模式则需要我们亲自把相应的 GPIO 端口拉高或置低产生非片选和片选信号。实际中软件模式应用比较多。
#define SPI_NSS_Soft                    ((uint16_t)0x0200)    //软件模式
#define SPI_NSS_Hard                    ((uint16_t)0x0000)    //硬件模式
```

### 软件读写

MySPI模块：完成通用的读写交换

```c
#include "stm32f10x.h"                  // Device header

//注意BitAction类型，它表示只要不是0，就为1
void MySPI_W_SS(uint8_t BitValue){
	GPIO_WriteBit(GPIOA,GPIO_Pin_4,(BitAction)BitValue);       // 选择从设备
}
void MySPI_W_SCK(uint8_t BitValue){
	GPIO_WriteBit(GPIOA, GPIO_Pin_5, (BitAction)BitValue);
}
void MySPI_W_MOSI(uint8_t BitValue){
	GPIO_WriteBit(GPIOA, GPIO_Pin_7, (BitAction)BitValue);
}
uint8_t MySPI_R_MISO(void){
	return GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_6);
}
void MySPI_Init(void){
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	// 输出引脚配置为推挽输出，输入引脚配置为浮空或上拉输入
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4 | GPIO_Pin_5 | GPIO_Pin_7;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	MySPI_W_SS(1);               // 初始化SS线为高电平，表示不选中从机
	MySPI_W_SCK(0);              // 空闲转态，要想切换模式，更改初始SCK 和 MySPI_W_SCK(0)、MySPI_W_SCK(1)的位置即可
}
void MySPI_Start(void){          // 开始传输，选中从机
	MySPI_W_SS(0);  
}
void MySPI_Stop(void){           // 停止传输，选中从机
	MySPI_W_SS(1);  
}
// 使用掩码方式
uint8_t MySPI_ReadWrite(uint8_t ByteSend){
	uint8_t ByteReceive = 0x00;
	/*
	* 如要发送1000 0000
	* 第一次应发送1：1000 0000 & 1000 0000 = 1000 0000  第一次发送第八位
	* 左移一位  0100 0000   
	* 第二次应发送0：1000 0000 & 0100 0000 = 0000 0000  第二次发送第七位
	* 接收类似
	*/
	for(int i = 0;i < 8; i++){
		MySPI_W_MOSI(ByteSend & (0x80 >> i));             // 将数据放入MOSI
		MySPI_W_SCK(1);                                   // 将数据移入
		if(MySPI_R_MISO() == 1)ByteReceive |= (0x80 >> i);// 读取MISO中的值，为1，则说明从机发送了数据1，经过0x80 >> i，放入ByteReceive
		MySPI_W_SCK(0);                                   // 将数据移出
	}
	return ByteReceive;
}
// 使用移位寄存器方式
uint8_t MySPI_ReadWrite2(uint8_t ByteSend){
	for(int i = 0;i < 8; i++){
		MySPI_W_MOSI(ByteSend & 0x80);          // 每一次都发送最高位 第八位
		ByteSend <<= 1;                         // 移除最高位
		MySPI_W_SCK(1);                         
		if(MySPI_R_MISO() == 1)ByteSend |= 0x01;// 如果为1，则将1加入到最低位
		MySPI_W_SCK(0);
	}
	return ByteSend;
}
```

W25Q64模块：完成对W25Q64的读写、擦除、查询设备ID、使能等操作

```c
#include "stm32f10x.h"                  // Device header
#include "MySPI.h"              

void W25Q64_Init(){
	MySPI_Init();
}
void W25Q64_ReadID(uint8_t *MID,uint16_t *DID){         // 读取厂商ID，和设备ID
	MySPI_Start();                                      // 开启
	MySPI_ReadWrite(0x9f);                              // 向W25Q64发送0x9f，查询厂商ID，和设备ID
	/*
	* 当发送0x9f后，W25Q64会发出1个8位的厂商ID，再通过两次发出一个16为的设备ID
	* 但是SPI没有单独接收或发送的情况，只有发送和接收同时进行，所以主机要想接收到厂商ID和设备ID
	* 就必须连续发送3个8为数据，从而接收3个8位数据
	*/
	*MID = MySPI_ReadWrite(0xff);                       // 通过发送0xff这种无效数据，接收到厂商ID
	*DID = MySPI_ReadWrite(0xff);                       // 通过发送0xff这种无效数据，接收到设备ID的高八位
	*DID <<= 8;                                         // 左移8位
	*DID |= MySPI_ReadWrite(0xff);                      // 合并低八位
	MySPI_Stop();                                       // 停止
}
// 写使能
void W2W25Q64_WriteEnable(void){
	MySPI_Start();   
	MySPI_ReadWrite(0x06);   // 06h为写使能，W2W25Q64不会再发送数据到主机
	MySPI_Stop(); 
}
// 读状态寄存器，主要是判断芯片是否在忙转态
void W2W25Q64_ReadStatusReg(void){
	
	uint32_t TimeOut = 10000000;
	
	MySPI_Start();  
	MySPI_ReadWrite(0x05);   // 05h为读状态寄存器，W2W25Q64会再发送1个数据到主机，读取最低位即可
	
	while((MySPI_ReadWrite(0xFF) & 0x01) == 0x01){     // 如果读取到的数据为1，则表示芯片正忙
		if(--TimeOut == 0) break;                      // 最多等10000000自减
	}
	
	MySPI_Stop(); 
}
// 指定页写入
void W2W25Q64_PageWrite(uint32_t Address,uint8_t *Array,uint16_t count){
	
	W2W25Q64_WriteEnable();                      // 写操作需要先使能
	
	MySPI_Start(); 
	//发送0x02 表示指定页写入，后面还需要24为的地址，再之后是写入的内容
	MySPI_ReadWrite(0x02);  // 02h为写入页，W2W25Q64会再发送1个数据到主机，读取最低位即可  

	MySPI_ReadWrite(Address >> 16);  // 右移16位，就是23~16位，注意不是Address >>= 16，>> 仅仅为右移不赋值
	MySPI_ReadWrite(Address >> 8);
	MySPI_ReadWrite(Address);        // uint8_t MySPI_ReadWrite2(uint8_t ByteSend) uint8_t会保留下低八位
	
	for(int i = 0; i < count; i++){
		MySPI_ReadWrite(Array[i]);
	}
	MySPI_Stop(); 
	
	W2W25Q64_ReadStatusReg();        // 等待芯片为空闲状态时再放行
}

// 扇区擦除
void W2W25Q64_SectorErase(uint32_t Address){
	W2W25Q64_WriteEnable();                      // 需要先使能
	
	MySPI_Start(); 
	//发送0x20表示扇区擦除，需要24为地址
	MySPI_ReadWrite(0x20);
	MySPI_ReadWrite(Address >> 16);
	MySPI_ReadWrite(Address >> 8);
	MySPI_ReadWrite(Address);
	MySPI_Stop(); 
	
	W2W25Q64_ReadStatusReg();        // 等待芯片为空闲状态时再放行
}
// 读数据
void W2W25Q64_ReadData(uint32_t Address,uint8_t *Array,uint32_t count){
	MySPI_Start(); 
	
	MySPI_ReadWrite(0x03);
	MySPI_ReadWrite(Address >> 16);
	MySPI_ReadWrite(Address >> 8);
	MySPI_ReadWrite(Address);

	for(int i = 0; i < count; i++){
		Array[i] = MySPI_ReadWrite(0xff);
	}
	MySPI_Stop(); 
}
```

main模块：将一个数组写入到W2W25Q64中，并从W2W25Q64中读取

```c
uint8_t MID;
uint16_t DID;
uint8_t writeArray[4] = {0x01,0x02,0x03,0x04};
uint8_t readArray[4] = {0x00,0x00,0x00,0x00};
int main(void)
{	
	OLED_Init();
	W25Q64_Init();
	
	OLED_ShowString(1,1,"MID:");
	OLED_ShowString(1,1,"MID:");
	W25Q64_ReadID(&MID,&DID);
	OLED_ShowHexNum(1, 5, MID, 2);
	OLED_ShowHexNum(1, 12, DID, 4);
	
	OLED_ShowString(2,1,"W:");
	OLED_ShowString(3,1,"R:");
	while(1){
		OLED_ShowHexNum(2, 3, writeArray[0], 2);
		OLED_ShowHexNum(2, 6, writeArray[1], 2);
		OLED_ShowHexNum(2, 9, writeArray[2], 2);
		OLED_ShowHexNum(2, 12, writeArray[3], 2);
		
		Delay_ms(1000);
        //对0x00000000地址的扇区进行擦除，即填充0xff
		W2W25Q64_SectorErase(0x00000000);                 //要想写入，必须先擦除，否则会出现乱码，因为W2W25Q64只能从1变为0
		W2W25Q64_PageWrite(0x00000000,writeArray,4);
		
		writeArray[0]++;
		writeArray[1]++;
		writeArray[2]++;
		writeArray[3]++;
		
		W2W25Q64_ReadData(0x00000000,readArray,4);
		OLED_ShowHexNum(3, 3, readArray[0], 2);
		OLED_ShowHexNum(3, 6, readArray[1], 2);
		OLED_ShowHexNum(3, 9, readArray[2], 2);
		OLED_ShowHexNum(3, 12, readArray[3], 2);
	}
}
```

### 硬件读写

与软件读写相比只需要修改MySPI模块

```c
#include "stm32f10x.h"                  // Device header

void MySPI_W_SS(uint8_t BitValue){
	GPIO_WriteBit(GPIOA,GPIO_Pin_4,(BitAction)BitValue); //虽然SS选择可用有硬件的NSS来决定，但是工作中一般还是使用软件模拟SS
}
void MySPI_Init(void){
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_SPI1,ENABLE);  //SPI1使用APB2
	
	// 输出引脚配置为推挽输出，输入引脚配置为浮空或上拉输入
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;     //GPIO_Pin_4为SS选择
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4;     
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;     //时钟信号的发出和数据的发出使用复用推挽输出
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5 | GPIO_Pin_7;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;       //数据的接收使用上拉输入
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	SPI_InitTypeDef SPI_InitStructure;                  //初始化SPI
	SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_128;   //128分频
	SPI_InitStructure.SPI_CPHA = SPI_CPHA_1Edge;        //第一个边沿作为输出
	SPI_InitStructure.SPI_CPOL = SPI_CPOL_Low;          //空闲为低电平
	SPI_InitStructure.SPI_CRCPolynomial = 7;            //不知道
	SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b;   //一次发送8位
	SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex;     //全双工
	SPI_InitStructure.SPI_FirstBit = SPI_FirstBit_MSB;  //高位先行
	SPI_InitStructure.SPI_Mode = SPI_Mode_Master;       //作为主机模式
	SPI_InitStructure.SPI_NSS = SPI_NSS_Soft;           //由软件模拟SS
	SPI_Init(SPI1,&SPI_InitStructure);
	
	SPI_Cmd(SPI1,ENABLE); 
	
	MySPI_W_SS(1);               // 初始化SS线为高电平，表示不选中从机
}
void MySPI_Start(void){          // 开始传输，选中从机
	MySPI_W_SS(0);  
}
void MySPI_Stop(void){           // 停止传输，选中从机
	MySPI_W_SS(1);  
}
uint8_t MySPI_ReadWrite(uint8_t ByteSend){
	while(SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_TXE) != SET);   //1先等待，TXE为1，也就是数据缓冲器SPI_DR为空
	SPI_I2S_SendData(SPI1, ByteSend);                              //2写出数据，并自动将TXE设为0
	
	while(SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_RXNE) != SET);  //3等待，RXNE为1，也就是接收缓冲器SPI_DR非空
	return SPI_I2S_ReceiveData(SPI1);                              //4读入数据，并自动将RXNE设为0
}
```

# CAN

## 简介

CAN 是 Controller Area Network 的缩写（以下称为 CAN），是 ISO 国际标准化的串行通信协议。“通过多个 LAN，进行大量数据的高速通信”的需要，1986 年德国电气商博世公司开发出面向汽车的 CAN 通信协议。此后，CAN 通过 ISO11898 及 ISO11519 进行了标准化



**CAN 协议特点：**

1. 多主控制。在总线空闲时，所有单元都可以发送消息（多主控制），而两个以上的单元同时开始发送消息时，根据标识符（Identifier以下称为ID）决定优先级。**ID并不是表示发送的目的地址，而是表示访问总线的消息的优先级**。两个以上的单元同时开始发送消息时，对各消息ID的每个位进行逐个仲裁比较。仲裁获胜（被判定为优先级最高）的单元可继续发送消息，仲裁失利的单元则立刻停止发送而进行接收工作。
2. 系统的柔软性。与总线相连的单元没有类似于“地址”的信息。因此在总线上增加单元时，连接在总线上的其它单元的软硬件及应用层都不需要改变。
3. 通信速度较快，通信距离远。最高1Mbps（距离小于40M），最远可达10KM（速率低于5Kbps）。
4. 具有错误检测、错误通知和错误恢复功能。所有单元都可以检测错误（错误检测功能），检测出错误的单元会立即同时通知其他所有单元（错误通知功能），正在发送消息的单元一旦检测出错误，会强制结束当前的发送。强制结束发送的单元会不断反复地重新发送此消息直到成功发送为止（错误恢复功能）。
5. 故障封闭功能。CAN可以判断出错误的类型是总线上暂时的数据错误（如外部噪声等）还是持续的数据错误（如单元内部故障、驱动器故障、断线等）。由此功能，当总线上发生持续数据错误时，可将引起此故障的单元从总线上隔离出去。
6. 连接节点多。CAN总线是可同时连接多个单元的总线。可连接的单元总数理论上是没有限制的。但实际上可连接的单元数受总线上的时间延迟及电气负载的限制。降低通信速度，可连接的单元数增加；提高通信速度，则可连接的单元数减少。正是因为 CAN 协议的这些特点，使得 CAN 特别适合工业过程监控设备的互连，因此，越来越受到工业界的重视，并已公认为最有前途的现场总线之一。CAN 协议经过 ISO 标准化后有两个标准：ISO11898 标准（高速 CAN）和 ISO11519-2 标准（低速CAN）。其中 ISO11898 是针对通信速率为125Kbps~1Mbps 的高速通信标准，而 ISO11519-2 是针对通信速率为 125Kbps 以下的低速通信标准。

![image-20230814141018456](STM32卷二.assets/image-20230814141018456.png)

**CAN FD** ：了解即可

CAN FD 是CAN with Flexible Data rate的缩写。也可以简单的认为是传统CAN的升级版。CAN FD是为了适应新能源和智能化汽车发展，由CAN2.0演变而来的新的CAN通信协议。 对比传统CAN总线技术，CAN FD有两方面的升级：

- 支持可变速率—> 最大5Mbit/s；
- 支持更长数据长度–> 最长64 bytes数据。

CAN和CAN FD区别：可以理解成CAN协议的升级版，只升级了协议，物理层未改变。

- 传输速率不同
  - CAN：最大传输速率1Mbps。
  - CAN FD：速率可变，仲裁比特率最高1Mbps（与CAN相同），数据比特率最高8Mbps,据调研目前应用的都是5Mbps。
- 数据长度不同
  - CAN：一帧数据最长8字节
  - CAN FD：一帧数据最长64字节。
- 帧格式不同：CanFD新增了FDF、BRS、ESI位。
- ID长度不同
  - CAN标准帧ID长度最长11bit
  - CANFD标准帧ID长度可扩展到12bit。
- CAN-FD和CAN主要的区别有两点：
  - 可变速率：CAN-FD采用了两种位速率：从控制场中的BRS位到ACK场之前（含CRC分界符）为可变速率，其余部分为原CAN总线用的速率。两种速率各有一套位时间定义寄存器，它们除了采用不同的位时间单位TQ外，位时间各段的分配比例也可不同。
  - 新的数据场长度：CAN-FD对数据场的长度作了很大的扩充，DLC最大支持64个字节，在DLC小于等于8时与原CAN总线是一样的，大于8时有一个非线性的增长，所以最大的数据场长度可达64字节。



### CAN物理层

与 I2C、SPI 等具有时钟信号的同步通讯方式不同，CAN 通讯并不是以时钟信号来进行同步的，它是一种异步通讯，只具有 CAN_High 和 CAN_Low 两条信号线，CAN 类似 RS485 也是通过差分信号传输数据。根据 CAN 总线上两根线的电位差来判断总线电平。总线电平分为显性电平和隐性电平，二者必居其一。



CAN 物理层的形式主要有两种

- 遵循 ISO11898 标准的高速、短距离“闭环网络”，它的总线最大长度为 40m，通信速度最高为 1Mbps，总线的两端各要求有一个“120 欧”的电阻。
- 遵循 ISO11519-2 标准的低速、远距离“开环网络”，它的最大传输距离为 1km，最高通讯速率为 125kbps，两根总线是独立的、不形成闭环，要求每根总线上 各串联有一个“2.2 千欧”的电阻。
- 终端电阻，用于阻抗匹配，以减少回波反射

#### **闭环总线网络**

![image-20230814141430011](STM32卷二.assets/image-20230814141430011.png)

#### **开环总线网络**

![image-20230814141459161](STM32卷二.assets/image-20230814141459161.png)

#### **通信节点**

从 CAN 通讯网络图可了解到，CAN 总线上可以挂载多个通讯节点，节点之间的信号经过总线传输，实现节点间通讯。由于 CAN 通讯协议不对节点进行地址编码，而是**对数据内容进行编码**的，所以网络中的节点个数理论上不受限制，只要总线的负载足够即可，可以通过中继器增强负载。



CAN 通讯节点由一个 CAN 控制器及 CAN 收发器组成，控制器与收发器之间通过CAN_Tx 及 CAN_Rx 信号线相连，收发器与 CAN 总线之间使用 CAN_High 及 CAN_Low信号线相连。其中 CAN_Tx 及 CAN_Rx 使用普通的类似 TTL 逻辑信号，而 CAN_High 及CAN_Low 是一对差分信号线，使用比较特别的差分信号。当 CAN 节点需要发送数据时，控制器把要发送的二进制编码通过 CAN_Tx 线发送到收发器，然后由收发器把这个普通的逻辑电平信号转化成差分信号，通过差分线 CAN_High 和 CAN_Low 线输出到 CAN 总线网络。而通过收发器接收总线上的数据到控制器时，则是相反的过程，收发器把总线上收到的 CAN_High 及 CAN_Low 信号转化成普通的逻辑电平信号，通过 CAN_Rx 输出到控制器中。



例如，STM32 的 CAN 片上外设就是通讯节点中的控制器，为了构成完整的节点，还要给它外接一个CAN收发器，CAN 控制器与 CAN 收发器的关系如同 TTL 串口与 MAX3232 电平转换芯片的关系，MAX3232 芯片把 TTL 电平的串口信号转换成 RS-232 电平的串口信号，CAN 收发器的作用则是把 CAN 控制器的 TTL 电平信号转换成差分信号(或者相反)。



目前有以下CAN电平转换芯片（不全）

![1691994350639](STM32卷二.assets/1691994350639.png)

![image-20230814143025865](STM32卷二.assets/image-20230814143025865.png)



#### **差分信号**

差分信号又称差模信号，与传统使用单根信号线电压表示逻辑的方式有区别，使用差分信号传输时，需要两根信号线，这两个信号线的振幅相等，相位相反，通过两根信号线的电压差值来表示逻辑 0 和逻辑 1。

![1691994068114](STM32卷二.assets/1691994068114.png)

相对于单信号线传输的方式，使用差分信号传输具有如下优点：

- **抗干扰能力强**，当外界存在噪声干扰时，几乎会同时耦合到两条信号线上，而接收端只关心两个信号的差值，所以外界的共模噪声可以被完全抵消。
- **能有效抑制它对外部的电磁干扰**，同样的道理，由于两根信号的极性相反，他们对外辐射的电磁场可以相互抵消，耦合的越紧密，泄放到外界的电磁能量越少。
- 时序定位精确，由于差分信号的开关变化是位于两个信号的交点，而不像普通单端信号依靠高低两个阈值电压判断，因而受工艺，温度的影响小，能降低时序上的误差，同时也更适合于低幅度信号的电路。

#### **CAN 协议中的差分信号**

CAN 协议中对它使用的 CAN_High 及 CAN_Low 表示的差分信号做了规定。

**显性电平具有优先权**。发送方通过使总线电平发生变化，将消息发送给接收方。

以高速 CAN 协议为例，当表示**逻辑 1 时(隐性电平)**，CAN_High 和 CAN_Low线上的电压均为 2.5v，即它们的电压差 VH-VL=0V；而表示**逻辑 0 时(显性电平)**，CAN_High 的电平为 3.5V，CAN_Low 线的电平为 1.5V，即它们的电压差为 VH-VL=2V。

例如，当 CAN 收发器从 CAN_Tx 线接收到来自 CAN 控制器的低电平信号时(逻辑 0)，它会使 CAN_High 输出 3.5V，同时 CAN_Low 输出 1.5V，从而输出显性电平表示逻辑 0。



![1691994159807](STM32卷二.assets/1691994159807.png)

![image-20230814142414228](STM32卷二.assets/image-20230814142414228.png)



在 CAN 总线中，必须使它处于隐性电平(逻辑 1)或显性电平(逻辑 0)中的其中一个状态。假如有两个 CAN 通讯节点，在同一时间，一个输出隐性电平，另一个输出显性电平，类似 I2C 总线的“线与”特性将使它处于显性电平状态，显性电平的名字就是这样来的，即可以认为显性具有优先的意味。



由于 CAN 总线协议的物理层只有 1 对差分线，在一个时刻只能表示一个信号，所以对通讯节点来说，**CAN 通讯是半双工**的，收发数据需要分时进行。在 CAN 的通讯网络中，因为共用总线，在整个网络中同一时刻只能有一个通讯节点发送信号，其余的节点在该时刻都只能接收。



### CAN协议层

#### **报文(帧)**

在 SPI 通讯中，片选、时钟信号、数据输入及数据输出这 4 个信号都有单独的信号线，I2C 协议包含有时钟信号及数据信号 2 条信号线，异步串口包含接收与发送 2 条信号线，这些协议包含的信号都比 CAN 协议要丰富，它们能轻易进行数据同步或区分数据传输方向。而 CAN 使用的是两条差分信号线，只能表达一个信号，简洁的物理层决定了 CAN 必然要配上一套更复杂的协议，如何用一个信号通道实现同样、甚至更强大的功能呢？CAN协议给出的解决方案是**对数据、操作命令(如读/写)以及同步信号进行打包**，**打包后的这些内容称为报文**。



**报文的种类**

在原始数据段的前面加上传输起始标签、片选(识别)标签和控制标签，在数据的尾段加上 CRC 校验标签、应答标签和传输结束标签，把这些内容按特定的格式打包好，就可以用一个通道表达各种信号了，各种各样的标签就如同 SPI 中各种通道上的信号，起到了协同传输的作用。当整个数据包被传输到其它设备时，只要这些设备按格式去解读，就能还原出原始数据，这样的报文就被称为 CAN 的“数据帧”

| 帧类型                      | 帧作用                                         |
| --------------------------- | ---------------------------------------------- |
| 数据帧（Data Frame）        | 用于发送单元向接收单元传输数据的帧             |
| 遥控帧（Remote Frame）      | 用于接收单元向具有相同ID的发送单元请求数据的帧 |
| 错误帧（Error Frame）       | 用于当检测出错误时向其他单元通知错误的帧       |
| 过载帧（Overload Frame）    | 用于接收单元通知其尚未做好接收准备的帧         |
| 间隔帧（Inter Frame Space） | 用于将数据帧  及遥控帧与前面的帧分离开来的帧   |

#### 数据帧结构

![image-20230814144002249](STM32卷二.assets/image-20230814144002249.png)



**数据帧以一个显性位(逻辑 0)开始**，**以 7 个连续的隐性位(逻辑 1)结束**，在它们之间，分别有**仲裁段、控制段、数据段、CRC 段和 ACK 段**。

**帧起始：**SOF 段(Start Of Frame)，译为帧起始，帧起始信号只有一个数据位，是一个显性电平，它用于通知各个节点将有数据传输，其它节点通过帧起始信号的电平跳变沿来进行硬同步。



**仲裁段：**当同时有两个报文被发送时，总线会根据仲裁段的内容决定哪个数据包能被传输，这也是它名称的由来。 仲裁段的内容主要为本数据帧的 ID 信息(标识符)，数据帧具有标准格式和扩展格式两种，区别就在于 ID 信息的长度，**标准格式的 ID 为 11 位**，扩展格式的 ID 为 29 位，它在标准 ID 的基础上多出 18 位。在 CAN 协议中，ID 起着重要的作用，它决定着数据帧发送的优先级，也决定着其它节点是否会接收这个数据帧。CAN 协议不对挂载在它之上的节点分配优先级和地址，对总线的占有权是由信息的重要性决定的，即对于重要的信息，我们会给它打包上一个优先级高的 ID，使它能够及时地发送出去。也正因为它这样的优先级分配原则，使得 CAN 的扩展性大大加强，在总线上增加或减少节点并不影响其它设备。报文的优先级，是通过对 ID 的仲裁来确定的。根据前面对物理层的分析我们知道如果总线上同时出现显性电平和隐性电平，总线的状态会被置为显性电平，CAN 正是利用这个特性进行仲裁。

若两个节点同时竞争 CAN 总线的占有权，当它们发送报文时，若**首先出现隐性电平，则会失去对总线的占有权**，进入接收状态。

两个设备发送的电平一样，所以它们一直继续发送数据。到了图中箭头所指的时序处，节点单元 1 发送的为隐性电平，而此时节点单元 2 发送的为显性电平，由于总线的“**线与**”特性使它表达出显示电平，因此单元 2 竞争总线成功，这个报文得以被继续发送出去。

![565a47d66a4ff3f278a211a435f431a](STM32卷二.assets/565a47d66a4ff3f278a211a435f431a.png)

仲裁段 ID 的优先级也影响着接收设备对报文的反应。因为在 CAN 总线上数据是以**广播**的形式发送的，**所有连接在 CAN 总线的节点都会收到所有其它节点发出的有效数据，因而我们的 CAN 控制器大多具有根据 ID 过滤报文的功能，它可以控制自己只接收某些 ID的报文。**



RTR 位(Remote Transmission Request Bit)，译作远程传输请求位，它是用于区分数据帧和遥控帧的（0，数据帧；1，远程帧），当它为显性电平时表示数据帧，隐性电平时表示遥控帧。

IDE 位(Identifier Extension Bit)，译作标识符扩展位，它是用于区分标准格式与扩展格式，当它为显性电平时表示标准格式，隐性电平时表示扩展格式。

SRR 位(Substitute Remote Request Bit)，只存在于扩展格式，它用于替代标准格式中的RTR 位。由于扩展帧中的 SRR 位为隐性位，RTR 在数据帧为显性位，所以在两个 ID相同的标准格式报文与扩展格式报文中，标准格式的优先级较高。



**控制段：**在控制段中的 r1 和 r0 为保留位，默认设置为显性位。它最主要的是 DLC 段(Data Length Code)，译为数据长度码，它由 4 个数据位组成，用于表示本报文中的数据段含有多少个字节，DLC 段表示的数字为 0~8。



**数据段：**数据段为数据帧的核心内容，它是节点要发送的原始信息，由 0~8 个字节组成，**MSB先行**。



**CRC 段：**为了保证报文的正确传输，CAN 的报文包含了一段 15 位的 CRC 校验码，一旦接收节点算出的 CRC 码跟接收到的 CRC 码不同，则它会向发送节点反馈出错信息，利用错误帧请求它重新发送。CRC 部分的计算一般由 CAN 控制器硬件完成，出错时的处理则由软件控制最大重发数。在 CRC 校验码之后，有一个 CRC 界定符，它为隐性位，主要作用是把 CRC 校验码与后面的 ACK 段间隔起来。

**ACK 段：**ACK 段包括一个 ACK 槽位，和 ACK 界定符位。类似 I2C 总线，在 ACK 槽位中，发送节点发送的是隐性位，而接收节点则在这一位中发送显性位以示应答。在 ACK 槽和帧结束之间由 ACK 界定符间隔开。

**帧结束：**EOF 段(End Of Frame)，译为帧结束，帧结束段由发送节点发送的 7 个隐性位表示结束。

#### 其他帧结构

![image-20230814145618849](STM32卷二.assets/image-20230814145618849.png)

#### CAN 波特率及位同步

由于 CAN 属于异步通讯，没有时钟信号线，连接在同一个总线网络中的各个节点会像串口异步通讯那样，节点间使用约定好的波特率进行通讯，特别地，CAN 还会使用“**位同步**”的方式来抗干扰、吸收误差，实现对总线电平信号进行正确的采样，确保通讯正常。

##### **位时序**

为了实现位同步，CAN 协议把每一个数据位的时序分解成 SS 段、PTS 段、PBS1 段、PBS2 段，这**四段的长度加起来即为一个 CAN 数据位的长度**。分解后最 小的时间单位是 Tq(Time Quantum)，而一个完整的位由 8~25 个 Tq 组成。为方便表示，图中的高低电平直接代表信号逻辑 0 或逻辑 1(不是差分信号)。

![image-20230814151259822](STM32卷二.assets/image-20230814151259822.png)





![1691996381783](STM32卷二.assets/1691996381783.png)

​	该图中表示的 CAN 通讯信号每一个数据位的长度为 19Tq，其中 SS 段占 1Tq，PTS 段占 6Tq，PBS1 段占 5Tq，PBS2 段占 7Tq。**信号的采样点位于 PBS1 段与 PBS2 段之间**，通过控制各段的长度，可以对采样点的位置进行偏移，以便准确地采样。

- SS 段(SYNC SEG)：SS 译为同步段，**若通讯节点检测到总线上信号的跳变沿被包含在 SS 段的范围之内，则表示节点与总线的时序是同步的**，当节点与总线同步时，采样点采集到的总线电平即可被确定为该位的电平。SS 段的大小固定为 1Tq。
- PTS 段(PROP SEG)：PTS 译为传播时间段，这个时间段是用于补偿网络的物理延时时间。是总线上输入比较器延时和输出驱动器延时总和的两倍。PTS 段的大小可以为 1~8Tq。
- PBS1 段(PHASE SEG1)：PBS1 译为相位缓冲段，主要用来**补偿边沿阶段的误差**，它的时间长度在重新同步的时候**可以加长**。PBS1 段的初始大小可以为 1~8Tq。
- PBS2 段(PHASE SEG2)：PBS2 这是另一个相位缓冲段，也是用来补偿边沿阶段误差的，它的时间长度在重新同步时**可以缩短**。PBS2 段的初始大小可以为 2~8Tq。

##### **通讯的波特率**

总线上的各个通讯节点只要约定好 1 个 Tq 的时间长度以及每一个数据位占据多少个Tq，就可以确定 CAN 通讯的波特率。 例如，假设上图中的 1Tq=1us，而每个数据位由 19 个 Tq 组成，则传输一位数据需要时间 T1bit =19us，从而每秒可以传输的数据位个数为：$1000000ns/19us = 52631.6bps$

这个每秒可传输的数据位的个数即为通讯中的波特率。

````
波特率计算方法：时钟主频 / 分频 / (tq1 + tq2 + swj)
````



##### **同步过程分析**

波特率只是约定了每个数据位的长度，数据同步还涉及到相位的细节，这个时候就需要用到数据位内的 SS、PTS、PBS1 及 PBS2 段了。根据对段的应用方式差异，CAN 的数据同步分为**硬同步和重新同步**。其中**硬同步只是当存在“帧起始信号”时起作用**，无法确保后续一连串的位时序都是同步的，而重新同步方式可解决该问题，这两种方式具体介绍如下：



**硬件同步：** 节点通过CAN总线发送数据，一开始发送帧起始信号。总线上其他节点会检测帧起始信号在不在位数据的SS段内，判断内部时序与总线是否同步。 假如不在SS段内，这种情况下，采样点获得的电平状态是不正确的。所以，节点会使用硬件同步方式调整， 把自己的SS段平移到检测到边沿的地方，获得同步，同步情况下，采样点获得的电平状态才是正确的。

![image-20230814150759954](STM32卷二.assets/image-20230814150759954.png)

**重新同步：**前面的硬同步只是当存在帧起始信号时才起作用，如果在一帧很长的数据内，节点信号与总线信号相位有偏移时，这种同步方式就无能为力了。因而需要引入重新同步方式，它利用普通数据位的**高至低电平的跳变沿来同步**(帧起始信号是特殊的跳变沿)。重新同步与硬同步方式相似的地方是它们都使用 SS 段来进行检测，同步的目的都是使节点内的 **SS段把跳变沿包含起来**。



重新同步的方式分为**超前**和**滞后**两种情况，以总线跳变沿与 SS 段的相对位置进行区分。第一种相位超前的情况如图，节点从总线的边沿跳变中，检测到它内部的时序比总线的时序相对超前 2Tq，这时控制器在下一个位时序中的 PBS1 段增加 2Tq 的时间长度，使得节点与总线时序重新同步。

![image-20230814150948873](STM32卷二.assets/image-20230814150948873.png)

第二种相位滞后的情况如图，节点从总线的边沿跳变中，检测到它的时序比总线的时序相对滞后 2Tq，这时控制器在**前一个位时序**中的 PBS2 段减少 2Tq 的时间长度，获得同步。

![image-20230814151036161](STM32卷二.assets/image-20230814151036161.png)

在重新同步的时候，PBS1 和 PBS2 中增加或减少的这段时间长度被定义为“**重新同步补偿宽度 SJW (reSynchronization Jump Width)**”。一般来说 CAN 控制器会限定 SJW 的最大值，如限定了最大 SJW=3Tq 时，单次同步调整的时候不能增加或减少超过 3Tq 的时间长度，若有需要，控制器会通过**多次小幅度**调整来实现同步。当控制器设置的 SJW 极限值较大时，可以吸收的误差加大，但通讯的速度会下降。



## STM32 CAN 外设

STM32 的芯片中具有 bxCAN 控制器 (Basic Extended CAN)，它支持 CAN 协议 2.0A 和2.0B 标准。该 CAN 控制器支持最高的通讯速率为 1Mb/s；可以自动地接收和发送 CAN 报文，支持使用标准 ID 和扩展 ID 的报文；外设中具有 3 个发送邮箱，发送报文的优先级可以使用软件控制，还可以记录发送的时间；具有 2 个 3 级深度的接收 FIFO，可使用过滤功能只接收或不接收某些 ID 号的报文；可变的过滤器组（最多 28 个）;可配置成自动重发；不支持使用 DMA 进行数据收发。

CAN2 无法直接访问存储区域，所以使用 CAN2 的时候必须使能 CAN1 外设的时钟。在 STM32 互联型产品中，带有 2 个 CAN 控制器，而我们使用的STM32F103ZET6 属于增强型，不是互联型，只有 1 个 CAN 控制器。



![image-20230814163715560](STM32卷二.assets/image-20230814163715560.png)

### CAN 控制内核

CAN 控制内核包含了各种控制寄存器及状态寄存器

**CAN主控制寄存器 (CAN_MCR)** 

![1691999486091](STM32卷二.assets/1691999486091.png)

(1) DBF 调试冻结功能 DBF(Debug freeze)调试冻结，使用它可设置 CAN 处于工作状态或禁止收发的状态，禁止收发时仍可访问接收 FIFO 中的数据。这两种状态是当 STM32 芯片处于程序调试模式时才使用的，平时使用并不影响。 (2) TTCM 时间触发模式 TTCM(Time triggered communication mode)时间触发模式，它用于配置 CAN 的时间触发通信模式，在此模式下，CAN 使用它内部定时器产生时间戳，并把它保存在CAN_RDTxR、CAN_TDTxR 寄存器中。内部定时器在每个 CAN 位时间累加，在接收和发送的帧起始位被采样，并生成时间戳。利用它可以实现 ISO 11898-4 CAN 标准的分时同步通信功能。 (3) ABOM 自动离线管理 ABOM(Automatic bus-off management) 自动离线管理，它用于设置是否使用自动离线管理功能。当节点检测到它发送错误或接收错误超过一定值时，会自动进入离线状态，在离线状态中，CAN 不能接收或发送报文。处于离线状态的时候，可以软件控制恢复或者直接使用这个自动离线管理功能，它会在适当的时候自动恢复。 (4) AWUM 自动唤醒 AWUM(Automatic bus-off management)，自动唤醒功能，CAN 外设可以使用软件进入低功耗的睡眠模式，如果使能了这个自动唤醒功能，当 CAN 检测到总线活动的时候，会自动唤醒。 (5) NART 自动重传 NART(No automatic retransmission)报文自动重传功能，设置这个功能后，当报文发送失败时会自动重传至成功为止。若不使用这个功能，无论发送结果如何，消息只发送一次。 (6) RFLM 锁定模式 RFLM(Receive FIFO locked mode)FIFO 锁定模式，该功能用于锁定接收 FIFO。锁定后，当接收 FIFO 溢出时，会丢弃下一个接收的报文。若不锁定，则下一个接收到的报文会覆盖原报文。 (7) TXFP 报文发送优先级的判定方法 TXFP(Transmit FIFO priority)报文发送优先级的判定方法，当 CAN 外设的发送邮箱中有多个待发送报文时，本功能可以控制它是根据报文的 ID 优先级还是报文存进邮箱的顺序来发送。



**位时序寄存器(CAN_BTR)及波特率**

CAN 外设中的位时序寄存器 CAN_BTR 用于配置测试模式、波特率以及各种位内的段参数。

![image-20230814155932409](STM32卷二.assets/image-20230814155932409.png)



(1) CAN控制器模式：为方便调试，STM32 的 CAN 提供了测试模式，配置位时序寄存器 CAN_BTR 的 SILM及 LBKM 寄存器位可以控制使用正常模式、静默模式、回环模式及静默回环模式

![image-20230814155832926](STM32卷二.assets/image-20230814155832926.png)

(2) 位时序及波特率：STM32 外设定义的位时序与前面的 CAN 标准时序有一点区别

![image-20230814160118418](STM32卷二.assets/image-20230814160118418.png)

STM32 的 CAN 外设位时序中只包含 3 段，分别是同步段 SYNC_SEG、位段 BS1 及位段 BS2，采样点位于 BS1 及 BS2 段的交界处。其中 SYNC_SEG 段固定长度为 1Tq，而BS1 及 BS2 段可以在位时序寄存器 CAN_BTR 设置它们的时间长度，它们可以在重新同步期间增长或缩短，该长度 SJW 也可在位时序寄存器中配置。理解 STM32 的 CAN 外设的位时序时，可以把它的 BS1 段理解为是由前面介绍的CAN 标准协议中 **PTS 段与 PBS1 段合在一起的**，而 BS2 段就相当于 PBS2 段。

通过配置位时序寄存器 CAN_BTR 的TS1[3:0]及 TS2[2:0]寄存器位设定 BS1 及 BS2 段的长度后，我们就可以确定每个 CAN 数据位的时间：

BS1 段时间：$T_{S1}=Tq*(TS1[3:0] + 1)$

BS2 段时间：$T_{S2}= Tq * (TS2[2:0] + 1)$

一个数据位的时间：$T_{1bit} =1Tq+T_{S1}+T_{S2} =1+ (TS1[3:0] + 1)+ (TS2[2:0] + 1)= N Tq$



其中单个时间片的长度 Tq 与 CAN 外设的所挂载的时钟总线及分频器配置有关，**CAN1 和 CAN2 外设都是挂载在 APB1 总线上**的，而位时序寄存器 CAN_BTR 中的 BRP[9:0]寄存器位可以设置 CAN 外设时钟的分频值 ，所以：$Tq = (BRP[9:0]+1) * T_{PCLK}$   其中的 PCLK 指 APB1 时钟，默认值为 36MHz。

![1692000860928](STM32卷二.assets/1692000860928.png)

### CAN 发送邮箱

一共有 3个发送邮箱，即最多可以缓存 3 个待发送的报文。

每个发送邮箱中包含有标识符寄存器 CAN_TIxR、数据长度控制寄存器 CAN_TDTxR及 2 个数据寄存器 CAN_TDLxR、CAN_TDHxR

**发送邮箱标识符寄存器 (CAN_TIxR) (x=0..2)**

![1692001112259](STM32卷二.assets/1692001112259.png)

CAN 的标准标识符的总位数为 11 位，而扩展标识符的总位数为 29 位。当报文使用扩展标识符的时候，标识符寄存器 CAN_TIxR 中的 STDID[10:0]等效于 EXTID[18:28]位，它与 EXTID[17:0]共同组成完整的 29 位扩展标识符。

该寄存器主要用来设置标识符（包括扩展标识符），另外还可以设置帧类型，通过 TXRQ 置1，来请求邮箱发送。因为有 3 个发送邮箱，所以寄存器 CAN_TIxR 有 3 个。

**CAN 发送邮箱数据长度和时间戳寄存器(CAN_TDTxR)**

![1692001195073](STM32卷二.assets/1692001195073.png)

**CAN 发送邮箱低字节数据寄存器(CAN_TDLxR)**

![1692001231122](STM32卷二.assets/1692001231122.png)

该寄存器用来存储将要发送的数据，这里只能存储低 4 个字节，另外还有一个寄存器CAN_TDHxR，该寄存器用来存储高 4 个字节，这样总共就可以存储 8 个字节。

### CAN 接收 FIFO

它一共有 2 个接收 FIFO，每个 FIFO 中有 3 个邮箱，即最多可以缓存 6 个接收到的报文。当接收到报文时，FIFO 的报文计数器会自增，而 STM32 内部读取 FIFO 数据之后，报文计数器会自减，我们通过状态寄存器可获知报文计数器的值，而通过前面主控制寄存器的 RFLM 位，可设置锁定模式，锁定模式下 FIFO 溢出时会丢弃新报文，非锁定模式下 FIFO 溢出时新报文会覆盖旧报文。



跟发送邮箱类似，每个接收 FIFO 中包含有标识符寄存器 CAN_RIxR、数据长度控制寄存器 CAN_RDTxR 及 2 个数据寄存器 CAN_RDLxR、CAN_RDHxR

通过中断或状态寄存器知道接收 FIFO 有数据后，我们再读取这些寄存器的值即可把接收到的报文加载到 STM32 的内存中。

### 验收筛选器

设置接收哪些信息

一共有 28 个筛选器组，每个筛选器组有 2 个寄存器，CAN1 和 CAN2 共用的筛选器的。其中STM32F103 系列芯片仅有 14 个筛选器组：0-13 号。 在 CAN 协议中，消息的标识符与节点地址无关，但与消息内容有关。因此，发送节点将报文广播给所有接收器时，接收节点会根据报文标识符的值来确定软件是否需要该消息，为了简化软件的工作，STM32 的 CAN 外设接收报文前会先使用验收筛选器检查，只接收需要的报文到 FIFO 中。



筛选器工作的时候，可以调整筛选 ID 的长度及过滤模式。根据筛选 ID 长度来分类有有以下两种：

| 过滤器组Reg | 32位                               | 16位（寄存器由两部分组成）          |
| ----------- | ---------------------------------- | ----------------------------------- |
| CAN_FxR1    | STDID[10:0]、EXTID[17:0]、IDE、RTR | STDID[10:0]、EXTID[17:15]、IDE、RTR |
| CAN_FxR2    | STDID[10:0]、EXTID[17:0]、IDE、RTR | STDID[10:0]、EXTID[17:15]、IDE、RTR |

通过配置筛选尺度寄存器 CAN_FS1R 的 FSCx 位可以设置筛选器工作在哪个尺度。

而根据过滤的方法分为以下两种模式：

- **标识符列表模式**，它把要接收报文的 ID 列成一个表，要求报文 ID 与列表中的某一个标识符完全相同才可以接收，可以理解为白名单管理。

- **掩码模式**，它把可接收报文 ID 的某几位作为列表，这几位被称为掩码，可以把它理解成关键字搜索，只要掩码(关键字)相同，就符合要求，报文就会被保存到接收 FIFO。

  通过配置筛选尺度寄存器 CAN_FS1R 的 FSCx 位可以设置筛选器工作在哪个尺度。

![image-20230814163223313](STM32卷二.assets/image-20230814163223313.png)

每组筛选器包含 2 个 32 位的寄存器，分别为 CAN_FxR1 和 CAN_FxR2，它们用来存储要筛选的 ID 或掩码

![1692001985198](STM32卷二.assets/1692001985198.png)

例如下面的表格所示，在掩码模式时，第一个寄存器存储要筛选的 ID，第二个寄存器存储掩码，掩码为 1 的部分表示该位必须与 ID 中的内容一致，筛选的结果为表中第三行的ID 值，它是一组包含多个的 ID 值，其中 x 表示该位可以为 1 可以为 0。

![1692002024149](STM32卷二.assets/1692002024149.png)

而工作在标识符模式时，2 个寄存器存储的都是要筛选的 ID，它只包含 2 个要筛选的ID 值(32 位模式时)。如果使能了筛选器，且报文的 ID 与所有筛选器的配置都不匹配，CAN 外设会丢弃该报文，不存入接收 FIFO。



**CAN 过滤器模式寄存器（CAN_FM1R）**

![1692001647954](STM32卷二.assets/1692001647954.png)

该寄存器用于设置各过滤器组的工作模式，对 28 个过滤器组的工作模式，都可以通过该寄存器设置，不过该寄存器必须在过滤器处于初始化模式下（CAN_FMR 的 FINIT 位=1），才可以进行设置。对 STM32F103ZET6 来说，只有[13:0]这 14 个位有效。

**CAN 过滤器位宽寄存器(CAN_FS1R)**

![image-20230814162811105](STM32卷二.assets/image-20230814162811105.png)

**CAN 过滤器 FIFO 关联寄存器（CAN_FFA1R）**

![image-20230814162835754](STM32卷二.assets/image-20230814162835754.png)

该寄存器设置报文通过过滤器组之后，被存入的 FIFO，如果对应位为 0，则存放到 FIFO0；如果为 1，则存放到 FIFO1。该寄存器也只能在过滤器处于初始化模式下配置。

**CAN 过滤器激活寄存器（CAN_FA1R）**

对对应位置 1，即开启对应的过滤器组；置 0 则关闭该过滤器组。

**CAN 的过滤器组 i 的寄存器 x（CAN_FiRx）**

![image-20230814162939657](STM32卷二.assets/image-20230814162939657.png)

每个过滤器组的 CAN_FiRx 都由 2 个 32 位寄存器构成，即：CAN_FiR1 和 CAN_FiR2。根据过滤器位宽和模式的不同设置，这两个寄存器的功能也不尽相同。关



## HAL库

### 常用函数

|              驱动函数              |         关联寄存器         |             功能描述              |
| :--------------------------------: | :------------------------: | :-------------------------------: |
|    __HAL_RCC_CANx_CLK_ENABLE(…)    |                            |            使能CAN时钟            |
|          HAL_CAN_Init(…)           |         MCR / BTR          |             初始化CAN             |
|      HAL_CAN_ConfigFilter(…)       |        过滤器寄存器        |         配置CAN接收过滤器         |
|          HAL_CAN_Start(…)          |         MCR / MSR          |            启动CAN设备            |
|  HAL_CAN_ActivateNotification(…)   |            IER             |             使能中断              |
|       __HAL_CAN_ENABLE_IT(…)       |            IER             |          使能CAN中断允许          |
|      HAL_CAN_AddTxMessage(…)       | TSR/TIxR/TDTxR/TDLxR/TDHxR |             发送消息              |
| HAL_CAN_GetTxMailboxesFreeLevel(…) |            TSR             | 等待发送完成 ,返回值为3时发送完成 |
|   HAL_CAN_GetRxFifoFillLevel(…)    |         RF0R/RF1R          |           等待接收完成            |
|      HAL_CAN_GetRxMessage(…)       |   RF0R/RF1R/RDLxR/RDHxR    |             接收消息              |

```c
/**	发送数据
*	参数1：CAN句柄
*	参数2：CAN TX句柄
*	参数3：输出的数据
*	参数4：使用的发送邮箱：CAN_TX_MAILBOX0、CAN_TX_MAILBOX1、CAN_TX_MAILBOX2
**/
HAL_StatusTypeDef HAL_CAN_AddTxMessage(CAN_HandleTypeDef *hcan, CAN_TxHeaderTypeDef *pHeader, uint8_t aData[], uint32_t *pTxMailbox);
/**	等待接收完成
*	参数1：CAN句柄
*	参数2：接收用的FIFO:CAN_RX_FIFO0、CAN_RX_FIFO1
**/
uint32_t HAL_CAN_GetRxFifoFillLevel(CAN_HandleTypeDef *hcan, uint32_t RxFifo);
/**	接收数据
*	参数1：CAN句柄
*	参数2：接收用的FIFO:CAN_RX_FIFO0、CAN_RX_FIFO1
*	参数3：CAN RX句柄
*	参数4：接收到的数据
**/
HAL_StatusTypeDef HAL_CAN_GetRxMessage(CAN_HandleTypeDef *hcan, uint32_t RxFifo, CAN_RxHeaderTypeDef *pHeader, uint8_t aData[]); 
```

**初始化**

```c
typedef struct __CAN_HandleTypeDef{
    CAN_TypeDef                 *Instance;			/* CAN1/CAN2 */
    CAN_InitTypeDef             Init;
} CAN_HandleTypeDef;
typedef struct{
    uint32_t Prescaler				/* 预分频 范围: 1~1024*/
    uint32_t Mode					/* 配置 CAN 的工作模式，回环或正常模式 */
    uint32_t SyncJumpWidth			/* 配置 SJW 极限值 */
    uint32_t TimeSeg1				/* 时间段1(BS1)长度 */
    uint32_t TimeSeg2				/* 时间段1(BS1)长度 */
    uint32_t TimeTriggeredMode		/* 是否使能 TTCM 时间触发功能 */
    uint32_t AutoBusOff				/* 是否使能 ABOM 自动离线管理功能 */
    uint32_t AutoWakeUp				/* 是否使能 AWUM 自动唤醒功能 */
    uint32_t AutoRetransmission 	/* 是否使能 NART 自动重传功能 */
    uint32_t ReceiveFifoLocked		/* 是否使能 RFLM 锁定 FIFO 功能 */
    uint32_t TransmitFifoPriority	/*  传输FIFO优先级 */
} CAN_InitTypeDef;
```

(1) Prescaler

本成员设置 CAN 外设的时钟分频，它可控制时间片 Tq 的时间长度，这里设置的值最终会减 1 后再写入 BRP 寄存器位，即前面介绍的 Tq 计算公式：

Tq = (BRP[9:0]+1) x TPCLK等效于：Tq = CAN_Prescaler x TPCLK

(2) Mode

本成员设置 CAN 的工作模式，可设置为正常模式 (CAN_MODE_NORMAL)、回环模式 (CAN_MODE_LOOPBACK)、静默模式 (CAN_MODE_SILENT) 以及回环静默模式(CAN_MODE_SILENT_LOOPBACK)。

(3) SyncJumpWidth

本成员可以配置 SJW 的极限长度，即 CAN 重新同步时单次可增加或缩短的最大长度，它可以被配置为 1-4Tq(CAN_SJW_1/2/3/4tq)。

(4) TimeSeg1

本成员用于设置 CAN 位时序中的 BS1 段的长度，它可以被配置为 1-16 个 Tq 长度(CAN_BS1_1/2/3…16tq)。

(5) TimeSeg2

本成员用于设置 CAN 位时序中的 BS2 段的长度，它可以被配置为 1-8 个 Tq 长度(CAN_BS2_1/2/3…8tq)。SYNC_SEG、 BS1 段及 BS2 段的长度加起来即一个数据位的长度，即前面介绍的原来

计算公式：T1bit =1Tq+TS1+TS2=1+ (TS1[3:0] + 1)+ (TS2[2:0] + 1)

等效于：T1bit= 1Tq+CAN_BS1+CAN_BS2

(6) TimeTriggeredMode

本成员用于设置是否使用时间触发功能 (ENABLE/DISABLE)，时间触发功能在某些CAN 标准中会使用到。

(7) AutoBusOff

本成员用于设置是否使用自动离线管理 (ENABLE/DISABLE)，使用自动离线管理可以在节点出错离线后适时自动恢复，不需要软件干预。

(8) AutoWakeUp

本成员用于设置是否使用自动唤醒功能 (ENABLE/DISABLE)，使能自动唤醒功能后它会在监测到总线活动后自动唤醒。

(9) AutoWakeUp

本成员用于设置是否使用自动离线管理功能 (ENABLE/DISABLE)，使用自动离线管理可以在出错时离线后适时自动恢复，不需要软件干预。

(10) AutoRetransmission

本成员用于设置是否使用自动重传功能 (ENABLE/DISABLE)，使用自动重传功能时，会一直发送报文直到成功为止。

(11) ReceiveFifoLocked

本成员用于设置是否使用锁定接收 FIFO(ENABLE/DISABLE)，锁定接收 FIFO 后，若FIFO 溢出时会丢弃新数据，否则在 FIFO 溢出时以新数据覆盖旧数据。

(12) TransmitFifoPriority

本成员用于设置发送报文的优先级判定方法 (ENABLE/DISABLE)，使能时，以报文存入发送邮箱的先后顺序来发送，否则按照报文 ID 的优先级来发送。配置完这些结构体成员后，我们调用库函数 HAL_CAN_Init 即可把这些参数写入到 CAN 控制寄存器中，实现 CAN 的初始化



**CAN 发送及接收结构体**

```
typedef struct{
    uint32_t StdId; /* 存储报文的标准标识符 11 位，0-0x7FF. */
    uint32_t ExtId; /* 存储报文的扩展标识符 29 位，0-0x1FFFFFFF. */
    uint32_t IDE;   /* 存储 IDE 扩展标志 */
    uint32_t RTR    /* 存储 RTR 远程帧标志 */
    uint32_t DLC;   /* 存储报文数据段的长度，0-8 */
    FunctionalState TransmitGlobalTime; 
} CAN_TxHeaderTypeDef;
typedef struct{
    uint32_t StdId; /* 存储报文的标准标识符 11 位，0-0x7FF. */
    uint32_t ExtId; /* 存储报文的扩展标识符 29 位，0-0x1FFFFFFF. */
    uint32_t IDE;   /* 存储 IDE 扩展标志 */
    uint32_t RTR;   /* 存储 RTR 远程帧标志 */
    uint32_t DLC;   /* 存储报文数据段的长度，0-8 */
    uint32_t Timestamp; 
    uint32_t FilterMatchIndex; 
} CAN_RxHeaderTypeDef;
```

(1) StdId

本成员存储的是报文的 11 位标准标识符，范围是 0-0x7FF。

(2) ExtId

本成员存储的是报文的 29 位扩展标识符，范围是 0-0x1FFFFFFF。ExtId 与 StdId 这两个成员根据下面的 IDE 位配置，只有一个是有效的。

(3) IDE

本成员存储的是扩展标志 IDE 位，当它的值为宏 CAN_ID_STD 时表示本报文是标准帧，使用 StdId 成员存储报文 ID；当它的值为宏 CAN_ID_EXT 时表示本报文是扩展帧，使用 ExtId 成员存储报文 ID。

(4) RTR

本成员存储的是报文类型标志 RTR 位，当它的值为宏 CAN_RTR_Data 时表示本报文是数据帧；当它的值为宏 CAN_RTR_Remote 时表示本报文是遥控帧，由于遥控帧没有数据段，所以当报文是遥控帧时，数据是无效的

(5) DLC

本成员存储的是数据帧数据段的长度，它的值的范围是 0-8，当报文是遥控帧时 DLC值为 0。

**CAN 筛选器结构体**

```c
typedef struct{
    uint32_t FilterIdHigh           /* ID高字节 */
    uint32_t FilterIdLow            /* ID低字节 */
    uint32_t FilterMaskIdHigh       /* 掩码高字节 */
    uint32_t FilterMaskIdLow        /* 掩码低字节 */
    uint32_t FilterFIFOAssignment   /* 过滤器关联FIFO */
    uint32_t FilterBank             /* 选择过滤器组 */
    uint32_t FilterMode             /* 过滤器模式*/
    uint32_t FilterScale            /* 过滤器位宽 */
    uint32_t FilterActivation       /* 过滤器使能 */
    Uint32_t SlaveStartFilterBank   /* 从CAN选择启动过滤器组 单CAN没有意义*/
} CAN_FilterTypeDef;
```

(1) FilterIdHigh

FilterIdHigh 成员用于存储要筛选的 ID，若筛选器工作在 32 位模式，它存储的是所筛选 ID 的高 16 位；若筛选器工作在 16 位模式，它存储的就是一个完整的要筛选的 ID。

(2) FilterIdLow

类似地，FilterIdLow 成员也是用于存储要筛选的 ID，若筛选器工作在 32 位模式，它存储的是所筛选 ID 的低 16 位；若筛选器工作在 16 位模式，它存储的就是一个完整的要筛选的 ID。

(3) FilterMaskIdHigh

FilterMaskIdHigh 存储的内容分两种情况，当筛选器工作在标识符列表模式时，它的功能与 FilterIdHigh 相同，都是存储要筛选的 ID；而当筛选器工作在掩码模式时，它存储的是 FilterIdHigh 成员对应的掩码，与 FilterIdLow 组成一组筛选器。

(4) FilterMaskIdLow

类似地， FilterMaskIdLow 存储的内容也分两种情况，当筛选器工作在标识符列表模式时，它的功能与 FilterIdLow 相同，都是存储要筛选的 ID；而当筛选器工作在掩码模式时，它存储的是 FilterIdLow 成员对应的掩码，与 FilterIdLow 组成一组筛选器。

|   过滤器配置模式   | CAN_FxR1[31:16] | CAN_FxR1[15:0] | CAN_FxR2[31:16]  | CAN_FxR2[15:0]  |
| :----------------: | :-------------: | :------------: | :--------------: | :-------------: |
| 32位标识符屏蔽模式 |  FilterIdHigh   |  FilterIdLow   | FilterMaskIdHigh | FilterMaskIdLow |
| 32位标识符列表模式 |  FilterIdHigh   |  FilterIdLow   | FilterMaskIdHigh | FilterMaskIdLow |
| 16位标识符屏蔽模式 | FilterMaskIdLow |  FilterIdLow   | FilterMaskIdHigh |  FilterIdHigh   |
| 16位标识符列表模式 | FilterMaskIdLow |  FilterIdLow   | FilterMaskIdHigh |  FilterIdHigh   |

(5) CAN_FilterFIFOAssignment 本成员用于设置当报文通过筛选器的匹配后，该报文会被存储到哪一个接收 FIFO，它的可选值为 FIFO0 或 FIFO1(宏 CAN_Filter_FIFO0/1)。 (6) CAN_FilterNumber 本成员用于设置筛选器的编号，即本过滤器结构体配置的是哪一组筛选器，CAN 一共有 28 个筛选器，所以它的可输入参数范围为 0-27(STM32F103 系列芯片的可输入参数为 0-13)。

(7) CAN_FilterMode 本成员用于设置筛选器的工作模式，可以设置为列表模式(宏 CAN_FilterMode_IdList)及掩码模式(宏 CAN_FilterMode_IdMask)。 (8) CAN_FilterScale 本成员用于设置筛选器的尺度，可以设置为 32 位长(宏 CAN_FilterScale_32bit)及 16 位长(宏 CAN_FilterScale_16bit)。 (9) CAN_FilterActivation 本成员用于设置是否激活这个筛选器(宏 ENABLE/DISABLE)。

### 案例

使用CAN1，PA11PA12,进行回环传输数据

```c
#define CAN_RX_GPIO_PORT                GPIOA
#define CAN_RX_GPIO_PIN                 GPIO_PIN_11
#define CAN_RX_GPIO_CLK_ENABLE()        do{ __HAL_RCC_GPIOA_CLK_ENABLE(); }while(0)   /* PA口时钟使能 */
#define CAN_TX_GPIO_PORT                GPIOA
#define CAN_TX_GPIO_PIN                 GPIO_PIN_12
#define CAN_TX_GPIO_CLK_ENABLE()        do{ __HAL_RCC_GPIOA_CLK_ENABLE(); }while(0)   /* PA口时钟使能 */
/* CAN接收RX0中断使能 */
#define CAN_RX0_INT_ENABLE      0               /* 0,不使能; 1,使能; */


CAN_HandleTypeDef   g_canx_handler;     /* CANx句柄 */
CAN_TxHeaderTypeDef g_canx_txheader;    /* 发送参数句柄 */
CAN_RxHeaderTypeDef g_canx_rxheader;    /* 接收参数句柄 */
/**
 * @brief       CAN初始化
 * @param       tsjw    : 重新同步跳跃时间单元.范围: 1~3;
 * @param       tbs2    : 时间段2的时间单元.范围: 1~8;
 * @param       tbs1    : 时间段1的时间单元.范围: 1~16;
 * @param       brp     : 波特率分频器.范围: 1~1024;
 *   @note      以上4个参数, 在函数内部会减1, 所以, 任何一个参数都不能等于0
 *              CAN挂在APB1上面, 其输入时钟频率为 Fpclk1 = PCLK1 = 36Mhz
 *              tq     = brp * tpclk1;
 *              波特率 = Fpclk1 / ((tbs1 + tbs2 + 1) * brp);
 *              我们设置 can_init(1, 8, 9, 4, 1), 则CAN波特率为:
 *              36M / ((8 + 9 + 1) * 4) = 500Kbps
 *
 * @param       mode    : CAN_MODE_NORMAL,  普通模式;
                          CAN_MODE_LOOPBACK,回环模式;
 * @retval      0,  初始化成功; 其他, 初始化失败;
 */
uint8_t can_init(uint32_t tsjw, uint32_t tbs2, uint32_t tbs1, uint16_t brp, uint32_t mode){
  g_canx_handler.Instance = CAN1;
  g_canx_handler.Init.Prescaler = brp;                /* 分频系数(Fdiv)为brp+1 */
  g_canx_handler.Init.Mode = mode;                    /* 模式设置 */
  g_canx_handler.Init.SyncJumpWidth = tsjw;           /* 重新同步跳跃宽度(Tsjw)为tsjw+1个时间单位 CAN_SJW_1TQ~CAN_SJW_4TQ */
  g_canx_handler.Init.TimeSeg1 = tbs1;                /* tbs1范围CAN_BS1_1TQ~CAN_BS1_16TQ */
  g_canx_handler.Init.TimeSeg2 = tbs2;                /* tbs2范围CAN_BS2_1TQ~CAN_BS2_8TQ */
  g_canx_handler.Init.TimeTriggeredMode = DISABLE;    /* 非时间触发通信模式 */
  g_canx_handler.Init.AutoBusOff = DISABLE;           /* 软件自动离线管理 */
  g_canx_handler.Init.AutoWakeUp = DISABLE;           /* 睡眠模式通过软件唤醒(清除CAN->MCR的SLEEP位) */
  g_canx_handler.Init.AutoRetransmission = ENABLE;    /* 禁止报文自动传送 */
  g_canx_handler.Init.ReceiveFifoLocked = DISABLE;    /* 报文不锁定,新的覆盖旧的 */
  g_canx_handler.Init.TransmitFifoPriority = DISABLE; /* 优先级由报文标识符决定 */
  if (HAL_CAN_Init(&g_canx_handler) != HAL_OK){
    return 1;
  }
#if CAN_RX0_INT_ENABLE
  /* 使用中断接收 */
  __HAL_CAN_ENABLE_IT(&g_canx_handler, CAN_IT_RX_FIFO0_MSG_PENDING); /* FIFO0消息挂号中断允许 */
  HAL_NVIC_EnableIRQ(USB_LP_CAN1_RX0_IRQn);                          /* 使能CAN中断 */
  HAL_NVIC_SetPriority(USB_LP_CAN1_RX0_IRQn, 1, 0);                  /* 抢占优先级1，子优先级0 */
#endif

  CAN_FilterTypeDef sFilterConfig;
  /* 配置CAN过滤器 不使用任何过滤*/
  sFilterConfig.FilterBank = 0;                             /* 过滤器0 */
  sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;
  sFilterConfig.FilterScale = CAN_FILTERSCALE_32BIT;
  sFilterConfig.FilterIdHigh = 0x0000;                      /* 32位ID */
  sFilterConfig.FilterIdLow = 0x0000;
  sFilterConfig.FilterMaskIdHigh = 0x0000;                  /* 32位MASK */
  sFilterConfig.FilterMaskIdLow = 0x0000;
  sFilterConfig.FilterFIFOAssignment = CAN_FILTER_FIFO0;    /* 过滤器0关联到FIFO0 */
  sFilterConfig.FilterActivation = CAN_FILTER_ENABLE;       /* 激活滤波器0 */
  sFilterConfig.SlaveStartFilterBank = 14;
  /* 过滤器配置 */
  if (HAL_CAN_ConfigFilter(&g_canx_handler, &sFilterConfig) != HAL_OK){
    return 2;
  }
  /* 启动CAN外围设备 */
  if (HAL_CAN_Start(&g_canx_handler) != HAL_OK){
    return 3;
  }
  return 0;
}

void HAL_CAN_MspInit(CAN_HandleTypeDef *hcan){
  if (CAN1 == hcan->Instance){
    CAN_RX_GPIO_CLK_ENABLE();       /* CAN_RX脚时钟使能 */
    CAN_TX_GPIO_CLK_ENABLE();       /* CAN_TX脚时钟使能 */
    __HAL_RCC_CAN1_CLK_ENABLE();    /* 使能CAN1时钟 */

    GPIO_InitTypeDef gpio_initure;

    gpio_initure.Pin = CAN_TX_GPIO_PIN;
    gpio_initure.Mode = GPIO_MODE_AF_PP;
    gpio_initure.Pull = GPIO_PULLUP;
    gpio_initure.Speed = GPIO_SPEED_FREQ_HIGH;
    HAL_GPIO_Init(CAN_TX_GPIO_PORT, &gpio_initure); /* CAN_TX脚 模式设置 */

    gpio_initure.Pin = CAN_RX_GPIO_PIN;
    gpio_initure.Mode = GPIO_MODE_AF_INPUT;
    HAL_GPIO_Init(CAN_RX_GPIO_PORT, &gpio_initure); /* CAN_RX脚 必须设置成输入模式 */
  }
}
#if CAN_RX0_INT_ENABLE /* 使能RX0中断 */
/**
 * @brief       CAN RX0 中断服务函数
 *   @note      处理CAN FIFO0的接收中断
 * @param       无
 * @retval      无
 */
void USB_LP_CAN1_RX0_IRQHandler(void)
{
  uint8_t rxbuf[8];
  uint32_t id;
  can_receive_msg(id, rxbuf);
  printf("id:%d\r\n", g_canx_rxheader.StdId);
  //……
}
#endif
/**
 * @brief       CAN 发送一组数据
 * @note        发送格式固定为: 标准ID, 数据帧
 * @param       id          : 标准ID(11位)
 */
uint8_t can_send_msg(uint32_t id, uint8_t *msg, uint8_t len){
  uint32_t TxMailbox = CAN_TX_MAILBOX0;     /* 使用邮箱1 */
  g_canx_txheader.StdId = id;         /* 标准标识符 */
  //g_canx_txheader.ExtId = id;       /* 扩展标识符(29位) */
  g_canx_txheader.IDE = CAN_ID_STD;   /* 使用标准帧 */
  g_canx_txheader.RTR = CAN_RTR_DATA; /* 数据帧 */
  g_canx_txheader.DLC = len;
  if (HAL_CAN_AddTxMessage(&g_canx_handler, &g_canx_txheader, msg, &TxMailbox) != HAL_OK) { /* 发送消息 */
    return 1;
  }
  while (HAL_CAN_GetTxMailboxesFreeLevel(&g_canx_handler) != 3); /* 等待发送完成,所有邮箱为空 */
  return 0;
}

/**
 * @brief       CAN 接收数据查询
 * @note        接收数据格式固定为: 标准ID, 数据帧
 * @param       id      : 要查询的 标准ID(11位)
 * @param       buf     : 数据缓存区
 * @retval      接收结果
 *   @arg       0   , 无数据被接收到;
 *   @arg       其他, 接收的数据长度
 */
uint8_t can_receive_msg(uint32_t id, uint8_t *buf){
  if (HAL_CAN_GetRxFifoFillLevel(&g_canx_handler, CAN_RX_FIFO0) == 0){     /* 没有接收到数据 */
    return 0;
  }
  if (HAL_CAN_GetRxMessage(&g_canx_handler, CAN_RX_FIFO0, &g_canx_rxheader, buf) != HAL_OK){  /* 读取数据 */
    return 0;
  } 
  /* 软件判断，接收到的ID不对 / 不是标准帧 / 不是数据帧 */
  if (g_canx_rxheader.StdId!= id || g_canx_rxheader.IDE != CAN_ID_STD || g_canx_rxheader.RTR != CAN_RTR_DATA){     
    return 0;
  }
  return g_canx_rxheader.DLC;
}
int main(void){
    uint8_t key;
    uint8_t i = 0, t = 0;
    uint8_t cnt = 0;
    uint8_t canbuf[8];
    uint8_t rxlen = 0;
    uint8_t res;
    uint8_t mode = 1; /* CAN工作模式: 0,普通模式; 1,环回模式 */

    HAL_Init();                                                            /* 初始化HAL库 */
    sys_stm32_clock_init(RCC_PLL_MUL9);                                    /* 设置时钟, 72Mhz */
    delay_init(72);                                                        /* 延时初始化 */
    usart_init(115200);                                                    /* 串口初始化为115200 */
    usmart_dev.init(72);                                                   /* 初始化USMART */
    led_init();                                                            /* 初始化LED */
    lcd_init();                                                            /* 初始化LCD */
    key_init();                                                            /* 初始化按键 */
    can_init(CAN_SJW_1TQ, CAN_BS2_8TQ, CAN_BS1_9TQ, 4, CAN_MODE_LOOPBACK); /* CAN初始化, 环回模式, 波特率500Kbps */

    lcd_show_string(30, 110, 200, 16, 16, "LoopBack Mode", RED);
    lcd_show_string(30, 130, 200, 16, 16, "KEY0:Send KEK_UP:Mode", RED); /* 显示提示信息 */

    lcd_show_string(30, 150, 200, 16, 16, "Count:", RED);        /* 显示当前计数值 */
    lcd_show_string(30, 170, 200, 16, 16, "Send Data:", RED);    /* 提示发送的数据 */
    lcd_show_string(30, 230, 200, 16, 16, "Receive Data:", RED); /* 提示接收到的数据 */

    while (1) {
        key = key_scan(0);
        if (key == KEY0_PRES){
            for (i = 0; i < 8; i++){
                canbuf[i] = cnt + i; /* 填充发送缓冲区 */
                if (i < 4){
                    lcd_show_xnum(30 + i * 32, 190, canbuf[i], 3, 16, 0X80, BLUE); /* 显示数据 */
                }else{
                    lcd_show_xnum(30 + (i - 4) * 32, 210, canbuf[i], 3, 16, 0X80, BLUE); /* 显示数据 */
                }
            }
            res = can_send_msg(0X12, canbuf, 8); /* ID = 0X12, 发送8个字节 */
            if (res){
                lcd_show_string(30 + 80, 170, 200, 16, 16, "Failed", BLUE); /* 提示发送失败 */
            }else{
                lcd_show_string(30 + 80, 170, 200, 16, 16, "OK    ", BLUE); /* 提示发送成功 */
            }
        }
        else if (key == WKUP_PRES){ /* WK_UP按下，改变CAN的工作模式 */
            mode = !mode;
            if (mode == 0){  /* 普通模式，需要2个开发板 */
                /* CAN普通模式初始化, 普通模式, 波特率500Kbps */
                can_init(CAN_SJW_1TQ, CAN_BS2_8TQ, CAN_BS1_9TQ, 4, CAN_MODE_NORMAL);    
                lcd_show_string(30, 110, 200, 16, 16, "Nnormal Mode ", RED);
            }
            else{            /* 回环模式,一个开发板就可以测试了. */
                /* CAN普通模式初始化, 回环模式, 波特率500Kbps */
                can_init(CAN_SJW_1TQ, CAN_BS2_8TQ, CAN_BS1_9TQ, 4, CAN_MODE_LOOPBACK);  
                lcd_show_string(30, 110, 200, 16, 16, "LoopBack Mode", RED);
            }
        }

        rxlen = can_receive_msg(0X12, canbuf); /* CAN ID = 0X12, 接收数据查询 */
        if (rxlen){ /* 接收到有数据 */
            lcd_fill(30, 270, 130, 310, WHITE); /* 清除之前的显示 */
            for (i = 0; i < rxlen; i++){
                if (i < 4){
                    lcd_show_xnum(30 + i * 32, 250, canbuf[i], 3, 16, 0X80, BLUE); /* 显示数据 */
                }else{
                    lcd_show_xnum(30 + (i - 4) * 32, 270, canbuf[i], 3, 16, 0X80, BLUE); /* 显示数据 */
                }
            }
        }
    }
}
```



## 标准库

### 常用函数

```c
/* Initialization and Configuration functions *********************************/ 
uint8_t CAN_Init(CAN_TypeDef* CANx, CAN_InitTypeDef* CAN_InitStruct);
void CAN_FilterInit(CAN_FilterInitTypeDef* CAN_FilterInitStruct);//初始化CAN的过滤器
void CAN_StructInit(CAN_InitTypeDef* CAN_InitStruct);//初始化CAN的控制器
void CAN_SlaveStartBank(uint8_t CAN_BankNumber); //启动CAN从机过滤器
void CAN_DBGFreeze(CAN_TypeDef* CANx, FunctionalState NewState);//控制CAN控制器的调试模式
void CAN_TTComModeCmd(CAN_TypeDef* CANx, FunctionalState NewState);//控制CAN控制器的TT-COM模式
 
/* Transmit functions *********************************************************/
//用于CAN消息的发送,参数1：CAN1 or CAN2
uint8_t CAN_Transmit(CAN_TypeDef* CANx, CanTxMsg* TxMessage);
uint8_t CAN_TransmitStatus(CAN_TypeDef* CANx, uint8_t TransmitMailbox);//用于获取CAN消息发送状态
void CAN_CancelTransmit(CAN_TypeDef* CANx, uint8_t Mailbox);//取消发送
 
/* Receive functions **********************************************************/
//CAN消息接收，参数2:CAN_FIFO0 or CAN_FIFO1
void CAN_Receive(CAN_TypeDef* CANx, uint8_t FIFONumber, CanRxMsg* RxMessage);
void CAN_FIFORelease(CAN_TypeDef* CANx, uint8_t FIFONumber);//清空缓冲区
uint8_t CAN_MessagePending(CAN_TypeDef* CANx, uint8_t FIFONumber);//获取CAN接收FIFO中待处理的消息数量
 
/* Operation modes functions **************************************************/
uint8_t CAN_OperatingModeRequest(CAN_TypeDef* CANx, uint8_t CAN_OperatingMode);
uint8_t CAN_Sleep(CAN_TypeDef* CANx);
uint8_t CAN_WakeUp(CAN_TypeDef* CANx);
 
/* Error management functions *************************************************/
uint8_t CAN_GetLastErrorCode(CAN_TypeDef* CANx);
uint8_t CAN_GetReceiveErrorCounter(CAN_TypeDef* CANx);
uint8_t CAN_GetLSBTransmitErrorCounter(CAN_TypeDef* CANx);
 
/* Interrupts and flags management functions **********************************/
void CAN_ITConfig(CAN_TypeDef* CANx, uint32_t CAN_IT, FunctionalState NewState);
FlagStatus CAN_GetFlagStatus(CAN_TypeDef* CANx, uint32_t CAN_FLAG);
void CAN_ClearFlag(CAN_TypeDef* CANx, uint32_t CAN_FLAG);
ITStatus CAN_GetITStatus(CAN_TypeDef* CANx, uint32_t CAN_IT);
void CAN_ClearITPendingBit(CAN_TypeDef* CANx, uint32_t CAN_IT);
```

**CAN初始化结构体**

```c
typedef struct{
  uint16_t CAN_Prescaler;   /* 预分频 范围: 1~1024*/ 
  uint8_t CAN_Mode;         /* 配置 CAN 的工作模式，回环或正常模式 */
  uint8_t CAN_SJW;          /* 配置 SJW 极限值，也就是SS段发生偏移的最大值*/
  uint8_t CAN_BS1;          /* 时间段1(BS1)长度 */
  uint8_t CAN_BS2;          /* 时间段1(BS1)长度 */
  FunctionalState CAN_TTCM; /* 是否使能 TTCM 时间触发功能 */
  FunctionalState CAN_ABOM;  /* 是否使能 ABOM 自动离线管理功能 */
  FunctionalState CAN_AWUM;  /* 是否使能 AWUM 自动唤醒功能 */
  FunctionalState CAN_NART;  /* 是否使能 NART 自动重传功能 */
  FunctionalState CAN_RFLM;  /* 是否使能 RFLM 锁定 FIFO 功能 */
  FunctionalState CAN_TXFP;  /* 传输FIFO优先级 */
} CAN_InitTypeDef;

// CAN_Prescaler：本成员设置CAN外设的时钟分频，它可以控制时间片Tq的时间长度。也就是前面提及的计算波特率的公式。

// CAN_Mode：工作模式
#define CAN_Mode_Normal             ((uint8_t)0x00)  
#define CAN_Mode_LoopBack           ((uint8_t)0x01) 
#define CAN_Mode_Silent             ((uint8_t)0x02)
#define CAN_Mode_Silent_LoopBack    ((uint8_t)0x03)
    
// CAN_SJW：本成员可以配置SJW的极限长度，即CAN重新同步时单次可增加或缩短的最大长度，可以被配置为1-4Tq（CAN_SJW_1/2/3/4tq）
#define CAN_SJW_1tq                 ((uint8_t)0x00)
#define CAN_SJW_2tq                 ((uint8_t)0x01)
#define CAN_SJW_3tq                 ((uint8_t)0x02)
#define CAN_SJW_4tq                 ((uint8_t)0x03)
    
// CAN_BS1：本成员用于设置CAN时序中BS1的长度，可以配置为（CAN_BS1_1-16tq）
#define CAN_BS1_1tq                 ((uint8_t)0x00)  /*!< 1 time quantum */
#define CAN_BS1_16tq                ((uint8_t)0x0F)  /*!< 16 time quantum */

// CAN_BS2：本成员用于设置CAN时序中BS2的长度，可以配置为（CAN_BS2_1-8tq）
#define CAN_BS2_1tq                 ((uint8_t)0x00)  /*!< 1 time quantum */
#define CAN_BS2_8tq                 ((uint8_t)0x07)  /*!< 8 time quantum */

// CAN_TTCM：是否使能TTCM时间触发功能
// CAN_ABOM：是否使能ABOM自动离线管理功能
// CAN_AWUM：是否使能AWUM自动唤醒功能
// CAN_NART：是否使能NART自动重传功能
// CAN_RFLM：是否使能RFLM锁定FIFO功能
// CAN_TXFP：配置TXFP报文优先级的判定方法 （以上是否使能的这些诸多结构体成员，都是ENABLE或者DISABLE，用到该功能时使能即可）
```

**CAN过滤结构体**

```c
typedef struct{
  uint16_t CAN_FilterIdHigh;			/* ID高字节 */
  uint16_t CAN_FilterIdLow;				/* ID低字节 */
  uint16_t CAN_FilterMaskIdHigh;		/* 掩码高字节 */
  uint16_t CAN_FilterMaskIdLow;			/* 掩码低字节 */
  uint16_t CAN_FilterFIFOAssignment;	/* 设置经过筛选后的数据存储到哪个接收FIFO */
  uint8_t CAN_FilterNumber;				/* 选择过滤器组 0 to 13 */
  uint8_t CAN_FilterMode;				/* 过滤器模式*/
  uint8_t CAN_FilterScale;				/* 设置筛选器尺度16 or 32*/
  FunctionalState CAN_FilterActivation; /* 过滤器使能 */
} CAN_FilterInitTypeDef;

// CAN_FilterFIFOAssignment
#define CAN_Filter_FIFO0             ((uint8_t)0x00)  /*!< Filter FIFO 0 assignment for filter x */
#define CAN_Filter_FIFO1             ((uint8_t)0x01)  /*!< Filter FIFO 1 assignment for filter x */
// CAN_FilterMode
#define CAN_FilterMode_IdMask       ((uint8_t)0x00)  /*!< identifier/mask mode */
#define CAN_FilterMode_IdList       ((uint8_t)0x01)  /*!< identifier list mode */
// CAN_FilterScale 
#define CAN_FilterScale_16bit       ((uint8_t)0x00) /*!< Two 16-bit filters */
#define CAN_FilterScale_32bit       ((uint8_t)0x01) /*!< One 32-bit filter */
```

**CAN发送接收结构体**

```c
typedef struct {
    uint32_t StdId;  /* 存储报文的标准标识符11位，0-0x7FF. */
    uint32_t ExtId;  /* 存储报文的扩展标识符29位，0-0x1FFFFFFF. */
                     /* ExtId与StdId这两个成员根据IDE位配置，只有一个是有效的。*/
    uint8_t IDE;     /* 存储IDE扩展标志 */
                     /* 当它的值为宏CAN_ID_STD时表示本报文是标准帧，使用StdId成员存储报文ID； 
                        当它的值为宏CAN_ID_EXT时表示本报文是扩展帧，使用ExtId成员存储报文ID。*/
    uint8_t RTR;     /* 存储RTR远程帧标志*/
                     /* 当它的值为宏CAN_RTR_Data时表示本报文是数据帧；
                        当它的值为宏CAN_RTR_Remote时表示本报文是遥控帧， 
                        由于遥控帧没有数据段，所以当报文是遥控帧时，下面的Data[8]成员的内容是无效的。*/
    uint8_t DLC;     /* 存储报文数据段的长度，0-8, 当报文是遥控帧时DLC值为0。 */
    uint8_t Data[8]; /* 存储报文数据段的内容 */
} CanTxMsg;
typedef struct {
    uint32_t StdId;  /* 存储了报文的标准标识符11位，0-0x7FF. */
    uint32_t ExtId;  /* 存储了报文的扩展标识符29位，0-0x1FFFFFFF. */
                     /* ExtId与StdId这两个成员根据IDE位配置，只有一个是有效的。*/
    uint8_t IDE;     /* 存储了IDE扩展标志 */
                     /* 当它的值为宏CAN_ID_STD时表示本报文是标准帧，使用StdId成员存储报文ID； 
                        当它的值为宏CAN_ID_EXT时表示本报文是扩展帧，使用ExtId成员存储报文ID。*/
    uint8_t RTR;     /* 存储了RTR远程帧标志*/
                     /* 当它的值为宏CAN_RTR_Data时表示本报文是数据帧；
                        当它的值为宏CAN_RTR_Remote时表示本报文是遥控帧， 
                        由于遥控帧没有数据段，所以当报文是遥控帧时，下面的Data[8]成员的内容是无效的。*/
    uint8_t DLC;     /* 存储报文数据段的长度，0-8, 当报文是遥控帧时DLC值为0。 */
    uint8_t Data[8]; /* 存储了报文数据段的内容 */
    uint8_t FMI;     /* 存储了筛选器的编号，表示本报文是经过哪个筛选器存储进接收FIFO的，可以用它简化软件处理，0-0xFF */
} CanRxMsg;
```





# 看门狗

## 独立看门狗

### **IWDG简介** 

IWDG的全称：Independent watchdog，即独立看门狗

IWDG的本质：能产生系统复位信号的计数器，对系统进行复位

IWDG的特性：递减的计数器，时钟由独立的RC振荡器提供（可在待机和停止模式下运行），看门狗被激活后，当递减计数器计数到0x000时产生复位

喂狗：在计数器计数到0之前，重装载计数器的值，防止复位

时钟源：来自内部低速时钟



**产出复位的方式**

1. NRST引脚上的低电平(外部复位) 
2. 窗口看门狗计数终止(WWDG复位) 
3. 独立看门狗计数终止(IWDG复位) 
4. 软件复位(SW复位) 
5. 低功耗管理复位



**作用**：当产生异常（无法喂狗）,看门狗会对系统进行复位

1. 异常：外界电磁干扰或者自身系统（硬件或软件）异常，造成程序跑飞，如：陷入某个不正常的死循环，打断正常的程序运行
2. 作用：主要用于检测外界电磁干扰，或硬件异常导致的程序跑飞问题
3. 应用：在一些需要高稳定性的产品中，并且对时间精度要求较低的场合
4. 独立看门狗是异常处理的最后手段，不可依赖，应在设计时尽量避免异常的发生！



**工作原理**

![image-20230802184028015](STM32卷二.assets/image-20230802184028015.png)

**IWDG框图**

![image-20230802184101729](STM32卷二.assets/image-20230802184101729.png)

启用IWDG后，LSI时钟会自动开启；LSI时钟频率并不精确，F1用40kHz，F4/F7/H7用32kHz进行计算即可



**IWDG寄存器**

键寄存器（IWDG_KR）：用于喂狗，解除PR和RLR寄存器写访问保护，以及启动看门狗工作

![image-20230802184208373](STM32卷二.assets/image-20230802184208373.png)

预分频器寄存器(IWDG_PR)

![image-20230802184225914](STM32卷二.assets/image-20230802184225914.png)

重装载寄存器(IWDG_RLR)：用于存放重装载值，低12位有效，即最大值为4096

![image-20230802184247706](STM32卷二.assets/image-20230802184247706.png)

状态寄存器(IWDG_SR)：用于判断预分频值和重装载值是否已经被更新

![image-20230802184307092](STM32卷二.assets/image-20230802184307092.png)

寄存器配置操作步骤

1. 通过在键寄存器 (IWDG_KR) 中写入 0xCCCC 来使能 IWDG。
2. 通过在键寄存器 (IWDG_KR) 中写入 0x5555 来使能寄存器访问。
3. 通过将预分频器寄存器 (IWDG_PR) 编程为 0~7 中的数值来配置预分频器。
4. 对重载寄存器 (IWDG_RLR) 进行写操作。
5. 等待寄存器更新 (IWDG_SR = 0x0000 0000)。
6. 刷新计数器值为 IWDG_RLR 的值 (IWDG_KR = 0xAAAA)。



**IWDG溢出时间计算**

![image-20230802184448289](STM32卷二.assets/image-20230802184448289.png)



### IWDG配置步骤

1. 取消PR/RLR寄存器写保护，设置IWDG预分频系数和重装载值，启动IWDG：HAL_IWDG_Init()
2. 及时喂狗，即写入0xAAAA 到IWDG_KR：HAL_IWDG_Refresh()

|       函数       |  主要寄存器   |                主要功能                |
| :--------------: | :-----------: | :------------------------------------: |
|  HAL_IWDG_Init   | IWDG_PR/RL/KR |  使能IWDG，设置预分频系数和重装载值等  |
| HAL_IWDG_Refresh |    IWDG_KR    | 把重装载寄存器的值重载到计数器中，喂狗 |

**句柄结构体**

```c
typedef struct { 
   IWDG_TypeDef *Instance; /* IWDG 寄存器基地址 */
   IWDG_InitTypeDef Init;  /* IWDG 初始化参数 */
}IWDG_HandleTypeDef;
typedef struct{ 
    uint32_t Prescaler;    /* 预分频系数 */ 
    uint32_t Reload;       /* 重装载值 */ 
} IWDG_InitTypeDef;
```

### 案例

![image-20230802191435833](STM32卷二.assets/image-20230802191435833.png)



```c
IWDG_HandleTypeDef g_iwdg_handle;
/* IWDG初始化函数 */
void iwdg_init(uint8_t prer, uint16_t rlr){
    g_iwdg_handle.Instance = IWDG;
    g_iwdg_handle.Init.Prescaler = prer;
    g_iwdg_handle.Init.Reload = rlr;
    HAL_IWDG_Init(&g_iwdg_handle);
}

/* 喂狗函数 */
void iwdg_feed(void){
    HAL_IWDG_Refresh(&g_iwdg_handle);
}
int main(void){
    HAL_Init();                             /* 初始化HAL库 */
    sys_stm32_clock_init(RCC_PLL_MUL9);     /* 设置时钟为72Mhz */
    delay_init(72);                         /* 延时初始化 */
    usart_init(115200);                     /* 串口初始化为115200 */
    
    printf("您还没喂狗，请及时喂狗！！！\r\n");
    iwdg_init(IWDG_PRESCALER_32, 1250);     /* 预分频系数为32，重装载值为1250，溢出时间约为1s 32*1250/40000 */
    while (1) {
        delay_ms(999);                      /* 串口显示 已经喂狗 */
        //delay_ms(1050);                   /* 没有及时喂狗，系统复位串口显示 您还没喂狗，请及时喂狗！！！ */
        iwdg_feed();
        printf("已经喂狗\r\n");
    }
}
```



## 窗口看门狗

### WWDG简介

WWDG的全称：Window watchdog，即窗口看门狗

WWDG的本质：能产生系统复位信号和**提前唤醒中断**的计数器

WWDG的特性：

- **递减**的计数器
- 当递减计数器值从 **0x40减到0x3F**时复位（64到63，01000000 -->00111111,即T6位跳变到0）
- 计数器的值大于W[6:0]值时喂狗会复位
- **提前唤醒中断** (EWI)：当递减计数器等于 0x40 时可产生

喂狗：在窗口期内重装载计数器的值，防止复位

时钟源：来自PCLK1（APB1 34MHz）



**作用：**用于监测单片机程序运行时效是否精准，主要检测**软件**异常

**应用：**需要**精准**检测程序运行时间的场合





**工作原理**

(T[6:0])最大值为127; W[6:0]最小为64

![image-20230802194001804](STM32卷二.assets/image-20230802194001804.png)

**WWDG框图**

![image-20230802194323341](STM32卷二.assets/image-20230802194323341.png)

可以看出：复位的两种激活路线：WDGA激活位必须要为1

- 路线1：WWDG_CR寄存器的T6变为0，也就是 0x40减到0x3F时（64到63，01000000 -->00111111,即T6位跳变到0）
- 路线2：当T6:0 >W6:0时，进行写WWDG_CR (喂狗)



**WWDG寄存器**

控制寄存器(WWDG_CR)：用于使能窗口看门狗工作，以及重装载计数器值（即喂狗）

![image-20230802194916591](STM32卷二.assets/image-20230802194916591.png)

配置寄存器(WWDG_CFR)：用于使能窗口看门狗提前唤醒中断，设置预分频系数，设置窗口上限值

![image-20230802194843404](STM32卷二.assets/image-20230802194843404.png)

状态寄存器(WWDG_SR)：用于判断是否发生了WWDG提前唤醒中断

![image-20230802194856046](STM32卷二.assets/image-20230802194856046.png)

**WWDG超时时间计算**

![image-20230802194956330](STM32卷二.assets/image-20230802194956330.png)



### WWDG配置步骤

1. WWDG工作参数初始化：HAL_WWDG_Init()
2. WWDG Msp初始化：HAL_WWDG_MspInit()  配置NVIC、CLOCK等
3. 设置优先级，使能中断：HAL_NVIC_SetPriority()、 HAL_NVIC_EnableIRQ()
4. 编写中断服务函数：WWDG_IRQHandler() --> HAL_WWDG_IRQHandler
5. 重定义提前唤醒回调函数：HAL_WWDG_EarlyWakeupCallback()
6. 在窗口期内喂狗：HAL_WWDG_Refresh()

**常用函数**

|       函数       |    主要寄存器    |              主要功能              |
| :--------------: | :--------------: | :--------------------------------: |
|  HAL_WWDG_Init   | WWDG_CR/WWDG_CFR | 使能WWDG，设置预分频系数和窗口值等 |
| HAL_WWDG_Refresh |     WWDG_CR      |        重装载计数器，即喂狗        |

```c
__HAL_RCC_WWDG_CLK_ENABLE();      // 开启WWDG时钟
void HAL_WWDG_EarlyWakeupCallback(WWDG_HandleTypeDef *hwwdg);       // 提前回调函数
```

**句柄结构体**

```c
typedef struct {
  WWDG_TypeDef *Instance; 	/* WWDG 寄存器基地址 */ 
  WWDG_InitTypeDef Init;    /* WWDG 初始化参数 */
}WWDG_HandleTypeDef;
typedef struct {
  uint32_t Prescaler;       /* 预分频系数 */
  uint32_t Window;          /* 窗口值M */
  uint32_t Counter;         /* 计数器值T */
  uint32_t EWIMode;         /* 提前唤醒中断使能 */ 
}WWDG_InitTypeDef;
```

### 案例

![image-20230802203349420](STM32卷二.assets/image-20230802203349420.png)

```
WWDG_HandleTypeDef g_wwdg_handle;
void WWDG_Init(uint8_t tr, uint8_t wr, uint32_t fprer){
    g_wwdg_handle.Instance = WWDG;                // 基地址
    g_wwdg_handle.Init.Prescaler = fprer;         // 分频
    g_wwdg_handle.Init.Counter = tr;              // T值
    g_wwdg_handle.Init.Window = wr;               // W值
    g_wwdg_handle.Init.EWIMode = WWDG_EWI_ENABLE; // 提前中断使能
    HAL_WWDG_Init(&g_wwdg_handle);
}
// WWDG初始化回调函数
void HAL_WWDG_MspInit(WWDG_HandleTypeDef *hwwdg){
    __HAL_RCC_WWDG_CLK_ENABLE();                  // 开启WWDG时钟
    HAL_NVIC_SetPriority(WWDG_IRQn, 2, 3);
    HAL_NVIC_EnableIRQ(WWDG_IRQn);
}
// 中断服务函数
void WWDG_IRQHandler(void){
    HAL_WWDG_IRQHandler(&g_wwdg_handle);  // HAL中断公共函数
}
// 提前中断函数回调函数
void HAL_WWDG_EarlyWakeupCallback(WWDG_HandleTypeDef *hwwdg){
    LED1_TOGGLE();
}

int main(void){
    HAL_Init();                                 /* 初始化HAL库 */
    sys_stm32_clock_init(RCC_PLL_MUL9);         /* 设置时钟,72M */
    delay_init(72);                             /* 初始化延时函数 */
    UART_Init();
    LED_init();   
    
    if(__HAL_RCC_GET_FLAG(RCC_FLAG_WWDGRST) != RESET){  // 这个函数可以查看是谁复位的系统
        printf("窗口看门狗复位\r\n");
        __HAL_RCC_CLEAR_RESET_FLAGS();
    }else{
        printf("外部复位\r\n");
    }
   
    WWDG_Init(0x7f, 0x5f, WWDG_PRESCALER_8);  //T值127 W值95 8分频
    
    printf("请在窗口期内喂狗\r\n\r\n");
    
    while(1){
        delay_ms(55);                       // 串口显示 已经喂狗！！！！
        //delay_ms(10);                     // 在T W区间喂狗，触发服务，串口显示 窗口看门狗复位
        HAL_WWDG_Refresh(&g_wwdg_handle);
        printf("已经喂狗！！！！\r\n");
        LED0_TOGGLE();
    }
}
```

## 区别

|     对比点     |          独立看门狗          |            窗口看门狗            |
| :------------: | :--------------------------: | :------------------------------: |
|     时钟源     |      LSI(40KHz或32KHz)       |           PCLK1或PCLK3           |
|    复位条件    |         递减计数到0          | 计数值大于W[6:0]值喂狗或减到0x3F |
|      中断      |           没有中断           |     计数值减到0x40可产生中断     |
| 递减计数器位数 | 12位（最大计数范围：4096~0） |   7位（最大计数范围：127~63）    |
|    应用场合    |  防止程序跑飞，死循环，死机  |    检测程序时效，防止软件异常    |





# FSMC

## **FSMC简介**

- FSMC( Flexible static memory controller)全称“灵活的静态存储器控制器”，是 STM32中一个很有特色的外设，通过 FSMC，STM32可以通过FSMC与SRAM、ROM、PSRAM、Nor Flash和NandFlash存储器的引脚相连，从而进行数据的交换。
- 要注意的是，FSMC 只能扩展静态的内存，即名称里面的 S：static，不能是动态的内存，比如 SDRAM 就不能扩展。
- 配置好FSMC，**定义一个指向这些地址的指针，通过对指针操作就可以直接修改存储单元的内容**，FSMC自动完成读写命令和数据访问操作，不需要程序去实现时序。
- F1/ F4(407)系列大容量型号，且引脚数目在100脚以上的芯片都有FSMC接口，F4/F7/H7系列就是FMC接口

```c
#define      FSMC_Addr_DATA        ( ( uint32_t ) 0x60020000 )       // 定义一个地址
void SRAM_Write_Data ( uint16_t usData ){
	* ( __IO uint16_t * ) ( FSMC_Addr_DATA ) = usData;               // 传输数据，直接往这个地址写数据即可
}
uint16_t SRAM_Read_Data ( void ){
	return ( * ( __IO uint16_t * ) ( FSMC_Addr_DATA ) );             // 读数据，直接返回这个地址上的数据即可
}
```

**FSMC和FMC区别**

- F1 和 F407 使用的是FSMC（Flexible static memory controller）“静态存储器控制器” 是Cortex-M3内核的芯片专属的,STM32可以通过FSMC与静态SRAM、ROM、PSRAM、Nor Flash和NandFlash存储器的引脚相连，从而进行数据的交换。
- 在Cortex-M4内核的F429和Cortex-M7内核的F7,H7系列中，变成了FMC(Flexible Memory Controller) 灵活存储控制器,支持了动态SDRAM 等设备，
- 其中最大的区别是什么呢？
  - FMC是在FSMC(Flexible Static Memory Controller)的基础上扩展了SDRAM的总线控制器,使用 FMC 可以动态刷新 SDRAM
- 静态RAM跟动态RAM
  - SRAM，静态的随机存取存储器，又被称为静态RAM，利用双稳态电路进行存储。即使有干扰对稳态电路也没影响，所以有双稳态性，“静态”是指只要不掉电，存储在SRAM中的数据就可以一直保存，只要有电，SRAM中的数据就不会有变化。加电情况下，不需要一直刷新，数据不会丢失
  - DRAM，动态随机存取存储器，需要不断的刷新，才能保存数据。主要的作用原理是利用电容内存储电荷的多寡来代表一个二进制比特（bit）是1还是0。由于在现实中电容会有漏电的现象，导致电位差不足而使记忆消失，因此除非电容经常周期性地充电，否则无法确保记忆长存。DRAM读取具有破坏性，也就是说，在读操作中会破坏内存单元行中的数据。因此，必需在该行上的读或写操作结束时，把行数据写回到同一行中。这一操作称为预充电，是行上的最后一项操作。必须完成这一操作之后，才能访问新的行，这一操作称为关闭打开的行。
  - SDRAM（Synchronous Dynamic Random Access Memory），同步动态随机存储器。同步的DRAM
  - 同步和异步的区别是同步方式需要一个专门的片外时钟CLK信号引脚，所有的读写操作都是跟着这个时钟信号走的



**框图**

![image-20230808143603534](STM32卷二.assets/image-20230808143603534.png)



在框图的右侧是FSMC外设相关的控制引脚，控制不同类型存储器的时候会有一些不同的引脚，其中地址线FSMC_A和数据线FSMC_D是所有控制器都共用的

用于连接硬件设备的引脚，控制不同类型的存储器会用不同的引脚。（“N”表明低电平有效信号）  

|   **FSMC信号**   | **信号方向** |               **功能**                |
| :--------------: | :----------: | :-----------------------------------: |
|  **FSMC_NE[x]**  |     输出     | 片选引脚，x=1…4，每个对应不同的内存块 |
|   **FSMC_CLK**   |     输出     |       时钟（同步突发模式使用）        |
| **FSMC_A[25:0]** |     输出     |               地址总线                |
| **FSMC_D[15:0]** |  输出/输入   |             双向数据总线              |
|   **FSMC_NOE**   |     输出     |               输出使能                |
|   **FSMC_NWE**   |     输出     |               写入使能                |
|  **FSMC_NWAIT**  |     输入     |       NOR闪存要求FSMC等待的信号       |
|  **FSMC_NADV**   |     输出     |     地址、数据线复用时作锁存信号      |

假如我们控制SRAM，则只需要**FSMC_NE[x]**  、  **FSMC_A[25:0]**  、  **FSMC_D[15:0]**  、**FSMC_NOE**  、 **FSMC_NWE**  



**FSMC地址映射**

FSMC 连接好外部的存储器并初始化后，就可以直接通过访问地址来读写数据 其中这部分在内存中有着固定的存储地址，存储单元是映射到 STM32 的内部寻址空间的；在程序里，定义一个指向这些地址的指针，然后就可以通过指针直接修改该存储单元的内容，FSMC 外设会自动完成数据访问过程，读写命令之类的操作不需要程序控制，具体如下：

![image-20230808144111840](STM32卷二.assets/image-20230808144111840.png)

**也就是从0x6000 0000 到0x9FFF FFFF 这1.0GB大小的空间被作为FSMC的地址**



![STM32F407读写SRAM_AHappySpringA的博客-CSDN博客_stm32sram读写](STM32卷二.assets/format,png.png)

如果从内核看的话中，左侧的是 Cortex-M3 内核的存储空间分配，右侧是 STM32 FSMC 外设的地址映射。可以看到 FSMC 的 NOR/PSRAM/SRAM/NAND FLASH 以及 PC 卡的地址都在 External RAM 地址空间内。正是因为存在这样的地址映射，使得访问 FSMC 控制的存储器时，就跟访问 STM32 的片上外设寄存器一样(片上外设的地址映射即图中左侧的“Peripheral”区域)

也就是访问FSMC其实就是访问固定的地址，跟寄存器操作异曲同工。





FSMC 把整个 存储区域分成了 4 个 Bank 区域，NOR 及 SRAM 存储器只能使用 Bank1 的地址，在每个 Bank 的内部又分成了 4 个小块，每个小块有相应的控制引脚用于连接片选信号FSMC_NE1/2/3/4



**拿存储块1为例**：FSMC存储块1被分为4个区，每个区管理64M字节空间

| **BANK1选区** | **片选信号** | **地址范围**              |
| ------------- | ------------ | ------------------------- |
| 第1区         | FSMC_NE1     | 0x6000 0000 ~ 0x63FF FFFF |
| 第2区         | FSMC_NE2     | 0x6400 0000 ~ 0x67FF FFFF |
| 第3区         | FSMC_NE3     | 0x6800 0000 ~ 0x6BFF FFFF |
| 第4区         | FSMC_NE4     | 0x6C00 0000 ~ 0x6FFF FFFF |



**HADDR**

HADDR[27:26]位用于选择四个存储块之一：

![1691477543017](STM32卷二.assets/1691477543017.png)

HADDR总线是转换到外部存储器的内部AHB地址线。

简单来说，从CPU通过AHB总线到外部信号线之间的关系。

HADDR[25:0]包含外部存储器地址。HHADDR是字节地址，而存储器访问不都是按字节访问，接到存储器的地址线与其数据宽度相关。（比如LCD使用16位数据线）

![image-20230808145658816](STM32卷二.assets/image-20230808145658816.png)

> 注意：数据宽度为16位时，地址存在偏移

![image-20230808145756483](STM32卷二.assets/image-20230808145756483.png)

Bank1的256M字节空间由`28根地址线`（ADDR[27:0]）寻址。这里ADDR 是内部AHB地址总线，其中`ADDR[25:0]`对应外部存储器地址`FSMC_A[25:0]`，而HADDR[26:27]对4个区进行寻址。为什么[25:0]这26位为地址：64M Byte = 2^26 Byte

四个区对应二进制：

- Bank1：0110-0000 0000-0000 0000-0000 0000-0000 ，即 60 00 00 00；
- Bank2：0110-0100 0000-0000 0000-0000 0000-0000 ，即 64 00 00 00；
- Bank3：0110-1000 0000-0000 0000-0000 0000-0000 ，即 68 00 00 00；
- Bank4：0110-1100 0000-0000 0000-0000 0000-0000 ，即 6c 00 00 00；

**数据宽度**：

- 这里要注意的是FSMC中每一个地址，对应的是存储器中的一个字节
- 假设我们用BANK1的区1，那么0x6000 0000-0x63ff ffff (共64M个字节byte地址) 对应的就是存储器的地址就是64Mx8=512M地址
- 也就是FSMC的64M个bit地址映射着存储器64M个字节byte地址
- 那也就是默认情况下存储器地址数据为8位，可以正常读取，但是如果存储器地址数据为16位，32位，就是两个字节一个地址，四个字节一个地址，那么控制16位，32位宽度的存储设备，且不支持单字节访问就比较麻烦了。比如16位宽度的NOR Flash，写入仅支持16位或者32位（通过写入两次16位），写入8位数据是不支持的 。
- 这个时候，为了方便操作，FMC在硬件设计上就提供了一种解决办法，将内部数据总线ADDR[25:0]措位后接到FMC_A地址引脚上，“丢弃”地址的bit-0，从bit-1开始对地址计数（相当于只计算偶数地址，总共有32M个地址）
- FSMC在实际输出地址时，将地址的值右移一位（相当于除以2，变成了偶数地址），输出到实际的地址线上。

## FSMC时钟

FSMC 外设挂载在 AHB 总线上，时钟信号来自于 HCLK(默认 72MHz)，控制器的同步时钟输出就是由它分频得到。 它的时钟频率可通过 FSMC_BTR 寄存器的 CLKDIV 位配置，HCLK 与 FSMC_CLK 的分频系数(CLKDIV)，可以为 2～16 分频

- 它可用于与**同步类型**的 NOR FLASH 芯片通过FSMC_CLK 引脚输出进行**同步通讯**，
- 对于**异步类型**的存储器，不使用同步时钟信号，所以时钟分频配置不起作用

FSMC 外设支持输出多种不同的时序以便于控制不同的存储器, 综合了 SRAM／ROM、PSRAM 和 NOR Flash 产品的信号特点，定义了 ABCD 四种不同的异步时序模型。选用不同的时序模型时，需要设置不同的时序参数

NOR/PSRAM控制器产生的异步时序就有5种，总体分为两类：一类是模式1，其他为拓展模式。拓展模式相对模式1来说读写时序时间参数设置可以不同，满足存储器读写时序不一样需求。

| **访问模式** | **对应的外部存储器** | **时序特性**                                                 | **时间参数**           |
| ------------ | -------------------- | ------------------------------------------------------------ | ---------------------- |
| 模式1        | SRAM/CRAM            | OE在读时序片选过程不翻转，有NBL信号，无NADV信号              | DATAST、ADDSET         |
| 模式A        | SRAM/PSRAM(CRAM)     | OE在读时序片选过程翻转，有NBL信号，无NADV信号                | DATAST、ADDSET         |
| 模式B/2      | NOR FLASH            | OE在读时序片选过程不翻转，无NBL信号，有NADV信号              | DATAST、ADDSET         |
| 模式C        | NOR FLASH            | OE在读时序片选过程翻转，无NBL信号，有NADV信号                | DATAST、ADDSET         |
| 模式D        | 带地址扩展的异步操作 | OE在读时序片选过程翻转，无NBL信号，有NADV信号，存在地址保存时间 | DATAST、ADDSET、ADDHLK |
| 同步突发     |                      | 根据同步时钟FSMC_CK读取多个顺序单元的数据                    | CLKDIV、DATLAT         |

## 常用函数

FSMC 相关的库函数分布在 stm32f1xx_II_fsmc.c 文件和头文件 stm32f1xx_II_fsmc.h 中。sram在stm32f1xx_hal_sram.c中

```
SRAM_HandleTypeDef
    
FSMC_NORSRAMInitTypeDef类型
FSMC_NORSRAMTimingInitTypeDef类型

FSMC_NANDInitTypeDef类型  
FSMC_NAND_PCCARDTimingInitTypeDef类型
FSMC_PCCARDInitTypeDef类型
/** SRAM 初始化函数
*   参数1：SRAM句柄
*   参数2：设置NORSRAM存储器的读时序
*   参数3：设置NORSRAM存储器的读时序，读写不同步用到
**/
HAL_StatusTypeDef HAL_SRAM_Init(SRAM_HandleTypeDef *hsram, FSMC_NORSRAM_TimingTypeDef *Timing,
                                FSMC_NORSRAM_TimingTypeDef *ExtTiming); 
__weak void HAL_SRAM_MspInit(SRAM_HandleTypeDef *hsram)

HAL_SDRAM_Init(); //SDRAM 初始化函数,省略入口参数
HAL_NOR_Init();   //NOR 初始化函数,省略入口参数
HAL_NAND_Init();  //NAND 初始化函数,省略入口参数
typedef struct{
    FSMC_NORSRAM_TypeDef *Instance;             /* 寄存器基地址,寄存器基地址，如果读写时序一样则配置基地址就行 */    
    FSMC_NORSRAM_EXTENDED_TypeDef *Extended;    /* 扩展模式寄存器基地址,也就是读写时序不同的时候， 指定写操作时序寄存器地址 */ 
    FSMC_NORSRAM_InitTypeDef Init;              /* SRAM初始化结构体 真正用来设置 SRAM 控制接口参数的*/ 
    HAL_LockTypeDef Lock;                       /* SRAM锁对象结构体 */ 
    __IO HAL_SRAM_StateTypeDef State;           /* SRAM设备访问状态 */ 
    DMA_HandleTypeDef *hdma;                    /* DMA结构体 使用 DMA 时候才使用*/ 
} SRAM_HandleTypeDef;
typedef struct{
	uint32_t NSBank; 				/* 存储区块号 */ 
	uint32_t DataAddressMux; 		/* 设置地址总线与数据总线是否复用,该变量仅对 NOR/PSRAM有效  */ 	
	uint32_t MemoryType; 			/* 存储器类型 */ 	
	uint32_t MemoryDataWidth; 		/* 存储器数据宽度，可选 8 位还是 16 位*/
	uint32_t BurstAccessMode;		/* 设置是否支持突发访问模式，只支持同步类型的存储器 */
	uint32_t WaitSignalPolarity;	/* 设置等待信号的极性 */
	uint32_t WrapMode; 				/* 突发模式下存储器传输使能 */
	uint32_t WaitSignalActive; 		/* 等待信号在等待状态之前或等待状态期间有效 */ 
	uint32_t WriteOperation; 		/* 存储器写使能,也就是是否允许写入 */ 
	uint32_t WaitSignal; 			/* 是否使能等待状态插入 */ 
	uint32_t ExtendedMode; 			/* 使能或者禁止使能扩展模式,也就是是否允许读写使用不同时序 */ 
	uint32_t AsynchronousWait; 		/* 用于异步传输期间，使能或者禁止等待信号 */ 
	uint32_t WriteBurst; 			/* 用于使能或者禁止异步的写突发操作 */ 
	uint32_t PageSize; 				/* 设置页大小 */ 
} FSMC_NORSRAM_InitTypeDef;
```



参数 WriteBurst，BurstAccessMode，WaitSignalPolarity，WrapMode，WaitSignalActive，WaitSignal，AsynchronousWait 等是用在**突发访问和同步时序**情况下。

```
typedef struct{
    uint32_t AddressSetupTime;          /* 地址建立时间 */ 
    uint32_t AddressHoldTime;           /* 地址保持时间 */    
    uint32_t DataSetupTime;             /* 数据建立时间 */    
    uint32_t BusTurnAroundDuration;     /* 总线周转阶段的持续时间 */
    uint32_t CLKDivision;               /* CLK时钟输出信号的周期 */
    uint32_t DataLatency;               /* 同步突发NOR FLASH的数据延迟 */
    uint32_t AccessMode;                /* 异步模式配置 */
}FSMC_NORSRAM_TimingTypeDef;
```

## FSMC驱动LCD

### 8080协议

**8080时序简介**

8080协议是一种并行、异步、[半双工](https://so.csdn.net/so/search?q=半双工&spm=1001.2101.3001.7020)通信协议，可用于单片机控制器与LCD驱动芯片之间的通信，由Intel提出，也叫英特尔总线。8080的通信端包括：

|  **信号**   | **名称**  | **控制状态** |                **作用**                |
| :---------: | :-------: | :----------: | :------------------------------------: |
|   **CS**    |   片选    |    低电平    |  选中器件，低电平有效，先选中，后操作  |
|   **WR**    |    写     |      ↑       | 写信号，上升沿有效，用于数据/命令写入  |
|   **RD**    |    读     |      ↑       | 读信号，上升沿有效，用于数据/命令读取  |
|   **RS**    | 数据/命令 |  0=命/1=数   | 表示当前是读写数据还是命令，也叫DC信号 |
| **D[15:0]** |  数据线   |      无      |  双向数据线，可以写入/读取驱动IC数据   |
|   **RES**   |   复位    |              |          复位信号，低电平有效          |

**注意：数据线是16位，也就是说16根线**

**8080写时序**

CS低电平选择外设，WR设置写入信号，RS设置写入命令还是数据(数据1/命令0在WR的上升沿),D表示写入的内容，RD保存高电平

![image-20230808132239395](STM32卷二.assets/image-20230808132239395.png)

**8080读时序**

数据（RS=1）/命令（RS=0）在RD的上升沿，读取到MCU，WR保持高电平

![image-20230808132509926](STM32卷二.assets/image-20230808132509926.png)

**案例**：读写LCD

```
void lcd_wr_data(uint16_t data){
    LCD_RS(1);      /* 操作数据 */
    LCD_CS(0);      /* 选中 */
    LCD_DATA_OUT(data); /* 数据 */
    LCD_WR(0);      /* WR低电平 */
    LCD_WR(1);      /* WR高电平 */
    LCD_CS(1);      /* 释放片选 */
}
uint16_t  lcd_rd_data(void){
    uint16_t ram;           /* 定义变量 */
    LCD_RS(1);                      /* 操作数据 */
    LCD_CS(0);          /* 选中 */
    LCD_RD(0);          /* RD低电平 */
    ram = LCD_DATA_IN;      /* 读取数据 */
    LCD_RD(1);          /* RD高电平 */
    LCD_CS(1);          /* 释放片选 */
    return ram；         /* 返回读数 */
}
```

### LCD

**LCD驱动芯片简介**

用于控制LCD的各种显示功能和效果，整体功能较复杂。常见型号：ILI9341/ST7789等

一般我们只需要：6条指令即可完成对LCD的基本使用（以9341为例）

| **指令(HEX)** | **名称** |             **作用**              |
| :-----------: | :------: | :-------------------------------: |
|   **0XD3**    |   读ID   | 用于读取LCD控制器的ID，区分型号用 |
|   **0X36**    | 访问控制 |  设置GRAM读写方向，控制显示方向   |
|   **0X2A**    |  列地址  |         一般用于设置X坐标         |
|   **0X2B**    |  页地址  |         一般用于设置Y坐标         |
|   **0X2C**    |  写GRAM  |        用于往LCD写GRAM数据        |
|   **0X2E**    |  读GRAM  |       用于读取LCD的GRAM数据       |



### FSMC驱动LCD

LCD使用的是类似异步、地址与数据线独立的SRAM控制方式

![image-20230808152201175](STM32卷二.assets/image-20230808152201175.png)

使用FSMC模式A与8080对比

![image-20230808152301498](STM32卷二.assets/image-20230808152301498.png)

**时序**

![image-20230808152445720](STM32卷二.assets/image-20230808152445720.png)

**LCD的RS信号线与地址线关系**

![image-20230808152713225](STM32卷二.assets/image-20230808152713225.png)

### 程序

```C
SRAM_HandleTypeDef g_sram_handle;    /* SRAM句柄(用于控制LCD) */
void lcd_init(void){
    FSMC_NORSRAM_TimingTypeDef fsmc_read_handle;
    FSMC_NORSRAM_TimingTypeDef fsmc_write_handle;
    
    /* FSMC控制的对象 */
    g_sram_handle.Instance = FSMC_NORSRAM_DEVICE;            // NORSRAM设备
    g_sram_handle.Extended = FSMC_NORSRAM_EXTENDED_DEVICE;   // NORSRAM扩展设备
    
    g_sram_handle.Init.NSBank               = FSMC_NORSRAM_BANK4;               /* NOR/PSRAM存储块中的BANK4 */
    g_sram_handle.Init.DataAddressMux       = FSMC_DATA_ADDRESS_MUX_DISABLE;    /* 数据线和地址线不复用 */
    g_sram_handle.Init.MemoryType           = FSMC_MEMORY_TYPE_SRAM;            /* 存储器类型 SRAM */
    g_sram_handle.Init.MemoryDataWidth      = FSMC_NORSRAM_MEM_BUS_WIDTH_16;    /* 存储器数据宽度 16位 */
    g_sram_handle.Init.BurstAccessMode      = FSMC_BURST_ACCESS_MODE_DISABLE;   /* 是否使能突发访问，仅对同步突发存储器有效 此处未用到 */
    g_sram_handle.Init.WaitSignalPolarity   = FSMC_WAIT_SIGNAL_POLARITY_LOW;    /* 等待信号的极性,仅在突发模式访问下有用 */
    g_sram_handle.Init.WaitSignalActive     = FSMC_WAIT_TIMING_BEFORE_WS;       /* 存储器是在等待周期之前的一个时钟周期还是等待周期期间使能NWAIT */
    g_sram_handle.Init.WriteOperation       = FSMC_WRITE_OPERATION_ENABLE;      /* 存储器写使能 */
    g_sram_handle.Init.WaitSignal           = FSMC_WAIT_SIGNAL_DISABLE;         /* 等待使能位,此处未用到 */
    g_sram_handle.Init.ExtendedMode         = FSMC_EXTENDED_MODE_ENABLE;        /* 读写使用不同的时序 */
    g_sram_handle.Init.AsynchronousWait     = FSMC_ASYNCHRONOUS_WAIT_DISABLE;   /* 是否使能同步传输模式下的等待信号,此处未用到 */
    g_sram_handle.Init.WriteBurst           = FSMC_WRITE_BURST_DISABLE;         /* 禁止突发写 */

    /* FSMC读时序控制寄存器 */
    fsmc_read_handle.AccessMode         = FSMC_ACCESS_MODE_A;    /* 模式A */
    fsmc_read_handle.AddressSetupTime   = 1;        /* 地址建立时间(ADDSET)为2个HCLK 1/72M = 13.9ns * 2 = 27.8ns (实际 > 200ns) */
    fsmc_read_handle.AddressHoldTime    = 0;        /* 地址保持时间(ADDHLD) 模式A是没有用到 */
    /* 因为液晶驱动IC的读数据的时候，速度不能太快,尤其是个别奇葩芯片 */
    fsmc_read_handle.DataSetupTime      = 15;       /* 数据保存时间(DATAST)为16个HCLK = 13.9 * 16 = 222.4ns */

    /* FSMC写时序控制寄存器 波形一致 */
    fsmc_write_handle.AccessMode        = FSMC_ACCESS_MODE_A;   /* 模式A */
    fsmc_write_handle.AddressSetupTime  = 1;        /* 地址建立时间(ADDSET)为2个HCLK = 27.8 ns */
    fsmc_write_handle.AddressHoldTime   = 0;        /* 地址保持时间(ADDHLD) 模式A是没有用到 */
    /* 某些液晶驱动IC的写信号脉宽，最少也得50ns。 */
    fsmc_write_handle.DataSetupTime     = 3;        /* 数据保存时间(DATAST)为4个HCLK = 13.9 * 4 = 55.6ns (实际 > 200ns) */
    HAL_SRAM_Init(&g_sram_handle, &fsmc_read_handle, &fsmc_write_handle);
    /**……**/
}
void HAL_SRAM_MspInit(SRAM_HandleTypeDef *hsram){
    __HAL_RCC_FSMC_CLK_ENABLE();            /* 使能FSMC时钟 */
    /**……**/
}
```



## FSMC相关寄存器

对于NOR_FLASH/PSRAM控制器(存储块1)配置工作，通过FSMC_BCRx、FSMC_BTRx和FSMC_BWTRx寄存器设置（其中x=1~4，对应4个区）。

| **寄存器** | **名称**       | **作用**                                    |
| ---------- | -------------- | ------------------------------------------- |
| FSMC_BCR4  | 片选控制寄存器 | 包含存储器块的信息（存储器类型/数据宽度等） |
| FSMC_BTR4  | 片选时序寄存器 | 设置读操作时序参数（ADDSET/DATAST）         |
| FSMC_BWTR4 | 写时序寄存器   | 设置写操作时序参数（ADDSET/DATAST）         |

**SRAM/NOR闪存片选控制寄存器（FSMC_BCRx）**

![image-20230808153058368](STM32卷二.assets/image-20230808153058368.png)

- EXTMOD：扩展模式使能位，控制是否允许读写不同的时序。
- WREN：写使能位。
- MWID[1:0]：存储器数据总线宽度。00，表示8位数据模式；01表示16位数据模式；10和11保留。
- MTYP[1:0]：存储器类型。00表示SRAM、ROM；01表示PSRAM；10表示NOR FLASH；11保留。
- MBKEN：存储块使能位。

以驱动LCD为例：

- EXTMOD：1,    LCD采用读和写用不同的时序
- WREN：1，向TFTLCD写入数据
- MWID：01，表示16位数据模式
- MTYP：00，表示SRAM
- MBKEN：1，使能块

**SRAM/NOR闪存片选时序寄存器（FSMC_BTRx）**：如果未设置FSMC_BCRx的EXTMOD位，则读写共用这个时序寄存器！

![image-20230808153409620](STM32卷二.assets/image-20230808153409620.png)

- ACCMOD[1:0]：访问模式。00：模式A；01：模式B；10：模式C；11：模式D。
- DATAST[7:0]：数据保持时间，等于DATAST(+1)个HCLK时钟周期，DATAST最大为255。
- ADDSET[3:0]：地址建立时间。表示ADDSET(+1)个HCLK时钟周期，ADDSET最大为15。



以驱动LCD为例：

- DATAST[7:0]
  - 对于ILI9341来说，其实就是RD低电平持续时间，最小为355ns。
  - 对于F1，一个HCLK = 13.9ns(1/72M)，设置为15         (STM32F1的FSMC性能存在问题)
  - 对于F4，一个HCLK = 6ns(1/168M)，设置为60
- ADDSET[3:0]
  - 对于ILI9341来说，相当于RD高电平持续时间，为90ns。
  - F1即使设置为0，RD也有超过90ns的高电平，这里设置为1。F4对该位设置为15。



**SRAM/NOR闪存写时序寄存器（FSMC_BWTRx）**：读写不同步时才用到这个寄存器

![image-20230808153733692](STM32卷二.assets/image-20230808153733692.png)

- ACCMOD[1:0]：访问模式。00：模式A；01：模式B；10：模式C；11：模式D。
- DATAST[7:0]：数据保持时间，等于DATAST(+1)个HCLK时钟周期，DATAST最大为255。
- ADDSET[3:0]：地址建立时间。表示ADDSET(+1)个HCLK时钟周期，ADDSET最大为15。

以驱动LCD为例：

- DATAST[7:0]
  - 对于ILI9341来说，其实就是WR低电平持续时间，最小为15ns。
  - 对于F1，一个HCLK = 13.9ns，设置为3
  - 对于F4，一个HCLK = 6ns，设置为9
- ADDSET[3:0]
  - 对于ILI9341来说，相当于WR高电平持续时间，为15ns。
  - F1即使设置为1，WR也有超过15ns的高电平，这里设置为1。F4对该位设置为8。

**FSMC寄存器组合说明**

在ST官方提供的寄存器定义里面，并没有定义FSMC_BCRx、FSMC_BTRx、FSMC_BWTRx等这个单独的寄存器，而是将他们进行了一些组合，规则如下：

FSMC_BCRx和FSMC_BTRx，组合成BTCR[8]寄存器组，他们的对应关系如下：

- BTCR[0]对应FSMC_BCR1，BTCR[1]对应FSMC_BTR1
- BTCR[2]对应FSMC_BCR2，BTCR[3]对应FSMC_BTR2
- BTCR[4]对应FSMC_BCR3，BTCR[5]对应FSMC_BTR3
- BTCR[6]对应FSMC_BCR4，BTCR[7]对应FSMC_BTR4

FSMC_BWTRx则组合成BWTR[7]寄存器组，他们的对应关系如下：

- BWTR[0]对应FSMC_BWTR1，BWTR[2]对应FSMC_BWTR2，
- BWTR[4]对应FSMC_BWTR3，BWTR[6]对应FSMC_BWTR4，
- BWTR[1]、BWTR[3]和BWTR[5]保留，没有用到





# SDIO

## SD卡

很多单片机系统都需要大容量存储设备，以存储数据。目前常用的有 U 盘，FLASH 芯片，SD 卡等。最适合单片机系统的莫过于 SD 卡了，它不仅容量可以做到很大（32GB 以上），支持 SPI/SDIO 驱动，而且有多种体积的尺寸可供选择（标准的 SD 卡尺寸及 Micor SD 卡尺寸等）

SD 卡主要有 SD、Mini SD 和 microSD(原名 TF 卡，2004 年正式更名为 Micro SD Card）

SD卡存储容量等级分为四个

| **SD卡类型** | **协议规范** | **文件系统** | **容量等级** |
| :----------: | :----------: | :----------: | :----------: |
|     SDSC     |    SD1.0     |   FAT12,16   |  上限至2GB   |
|     SDHC     |    SD2.0     |    FAT32     |  2GB至32GB   |
|     SDXC     |    SD3.0     |    exFAT     |  32GB至2TB   |
|     SDUC     |    SD7.0     |    exFAT     |  2TB至128TB  |

SD卡经常被用在Window操作系统上存取数据，就得使用操作系统支持的FAT或exFAT文件系统。（EEPROM或者NOR FLASH操作一样，读写数据并验证数据的正确性，不需要FAT文件系统。）



### **SD卡驱动方式**

可以通过SPI与SD卡进行通信(非常复杂),使用SDIO与HAL库配合简单快捷

SD 卡 与 microSD卡的引脚不同，SD卡有9个引脚，microSD卡有8个引脚，只比 SD 卡少了一个电源引脚 VSS2。SD 卡和 micorSD 只有引脚和形状大小不同，内部结构类似，操作时序完全相同，可以使用完全相同的代码驱动

SD 卡允许了不同的接口来访问它的内部存储单元。最常见的是 **SDIO 模式和 SPI 模式**

下面以microSD卡为例

| **引脚编号** | **引脚名称** | **功能（SDIO模式）** | **功能（SP模式）** |
| :----------: | :----------: | :------------------: | :----------------: |
|    Pin  1    |     DAT2     |       数据线2        |        保留        |
|    Pin  2    |   DAT3/CS    |       数据线3        |      片选信号      |
|    Pin  3    |   CMD/MOSI   |        命令线        | 主机输出，从机输入 |
|    Pin  4    |     VDD      |         电源         |        电源        |
|    Pin  5    |     CLK      |         时钟         |        时钟        |
|    Pin  6    |     VSS      |        电源地        |       电源地       |
|    Pin  7    |  DAT0/MISO   |       数据线0        | 主机输入，从机输出 |
|    Pin  8    |     DAT1     |       数据线1        |        保留        |

SDIO接口通信线：CLK/CMD/DAT0~3

- CLK：时钟线，由SDIO主机产生，由STM32微控制器SDIO外设输出
- CMD：命令线，SDIO主机通过该线发送命令控制SD卡，若命令要求SD卡响应，SD卡也是通过该线传输响应信息。
- DAT0~3：数据线，用于接收或发送数据；SD卡可将DAT0拉低表示处于忙状态

### SD卡寄存器

SD 卡有自己的寄存器，但它不能直接进行读写操作，需要通过命令来控制，SDIO 协议定义了一些命令用于实现某一特定功能，SD 卡根据收到的命令要求对内部寄存器进行修改。

| **名称** | **宽度(bit)** |                           **描述**                           |
| :------: | :-----------: | :----------------------------------------------------------: |
|   CID    |      128      | 卡标识寄存器，提供制造商ID、OEM/应用ID、产品名称、版本、序列号、制造日期等信息（每个卡都是唯一的） |
|   RCA    |      16       | 相对卡地址(Relative card address)寄存器，提供本地系统中卡的地址，动态由卡建议，在主机初始化的时候确定  注意：仅SDIO模式下有，SPI模式下无RCA |
|   CSD    |      128      |       卡特定数据寄存器，提供SD卡操作条件相关信息和数据       |
|   SCR    |      64       |             SD配置寄存器，提供SD卡一些特定的数据             |
|   OCR    |      32       |          操作条件寄存器，主要是SD卡的操作电压等信息          |

OCR寄存器：0~23位表示支持的电压范围(SD卡支持范围中，相应位置1)，30为表示支持卡的类别(1：V2.0 SDHC卡，0：V2.0 SDSC卡)

### 命令和响应

SDIO与SPI模式有区别，以SDIO模式为例

一个完整的 SD 卡操作过程是：主机(单片机等)发起“命令”，SD 卡根据命令的内容决定是否发送响应信息及数据等，如果是数据读/写操作，主机还需要发送停止读/写数据的命令来结束本次操作，这意味着主机发起命令指令后，SD 卡可以没有响应、数据等过程，这取决于命令的含义。

SD 卡有多种命令和响应，它们的格式定义及含义在《SD 卡协议 V2.0》的第三和第四章有详细介绍，发送命令时主机只能通过 CMD 引脚发送给 SD 卡，串行逐位发送时先发送最高位(MSB)，然后是次高位这样类推

![1692247336002](STM32卷二.assets/1692247336002.png)

命令：应用相关命令(ACMD)和通用命令(CMD)，通过命令线CMD传输，固定长度48位

响应：SD卡接收到命令，会有一个响应，用来反应SD卡状态。有2种响应类型：短响应(48位，格式与命令一样)和长响应(136位)。

数据：主机发送的数据 / SD发送的数据。SD数据是以块(Block)形式传输，SDHC卡数据块长度一般为512字节。数据块需要CRC保证数据传输成功。



**SD卡命令格式**

SD卡的命令格式由6个字节组成，发送数据时高位在前，SD卡的写入命令格式如下：

![image-20230817124345479](STM32卷二.assets/image-20230817124345479.png)

字节1：最高 2 位固定为 01，低 6 位为命令号（比如 CMD16，为 10000B 即 16 进制的 0X10，完整的 CMD16，第一个字节为 01010000， 即 0X10+0X40）

字节2~5：命令参数，有些命令参数是保留位，没有定义参数的内容，保留位应设置为0

字节6：用于校验命令传输内容正确性，前7位为CRC(循环冗余校验)校验位，最后一位为停止位1



SD 卡的命令总共有 12 类，分为 Class0~Class11，介绍几个比较重要的命令

基本命令 Class 0（CMD0/CMD8/CMD9/CMD10/CMD12/CMD2/CMD3）、面向块读取命令 Class 2（CMD16/CMD17/CMD18）、面向块写入命令 Class 4（CMD16/CMD24/CMD25）、擦除命令 Class 5、加锁命令 Class 7、特定于应用命令 Class 8（CMD55）、面向块写保护命令 Class 6、I/O模式命令 Class 9、SD卡特定应用命令（ ACMD41 / ACMD6 / ACMD51）、Switch功能命令



**SD卡常用命令**

|     命令     |        参数        | 响应 |                            描述                            |
| :----------: | :----------------: | :--: | :--------------------------------------------------------: |
|  CMD0(0x0)   |        NONE        |  无  |                          复位SD卡                          |
|  CMD8(0X08)  | VHS+Check  pattern |  R7  |                    主机发送接口状态命令                    |
| ACMD41(0X29) |    HCS+VDD电压     |  R3  |          主机发送容量支持信息HCS  和OCR寄存器内容          |
|  CMD2(0X2)   |        NONE        |  R2  |                   读取SD卡的CID寄存器值                    |
|  CMD3(0X3)   |        NONE        |  R6  |                  要求SD卡发布新的相对地址                  |
|  CMD9(0X9)   |        RCA         |  R2  |                 获得选定卡的CSD寄存器内容                  |
|  CMD7(0X7)   |        RCA         | R1b  |                          选中SD卡                          |
| CMD16(0x10)  |    block length    |  R1  |                   设置SD卡的块大小(字节)                   |
| CMD17(0X11)  |        地址        |  R1  |                      读取一个块的数据                      |
| CMD18(0X12)  |        地址        |  R1  |                    连续读取多个块的数据                    |
| CMD12(0X0C)  |        NONE        | R1b  |                   多块读取强制卡停止传输                   |
| CMD13(0x0D)  |        RCA         |  R1  |                    被选中的卡返回其状态                    |
| ACMD23(0x97) |  Number of block   |  R1  | 设置需要预擦除的数据块数，提高SD卡多数据块写入性能（速度） |
| CMD24(0X18)  |        地址        |  R1  |                      写入一个块的数据                      |
| CMD25(0X19)  |        地址        |  R1  |                    连续写入多个块的数据                    |
| CMD55(0X37)  |        NONE        |  R1  |               通知SD卡，接下来发送是应用命令               |

**SD卡响应**

- SD卡和单片机的通信采用发送应答机制。
- 每发送一个命令，SD卡都会给出一个应答，以告知主机该命令的执行情况，或者返回主机需要获取的数据。使用SDIO接口时，响应通过CMD线传输。
- SD卡响应因使用接口不同，格式也不同。响应具体有R1、R1b、R2、R3、R7。
- 响应内容大小可以分为短响应48bit和长响应136bit。

| 响应 | 长度(SDIO) |                    描述                     |
| :--: | :--------: | :-----------------------------------------: |
|  R1  |   48bit    |                正常响应命令                 |
| R1b  |   48bit    | 格式与R1相同，可选增加忙信号(数据线0上传输) |
|  R2  |   136bit   |     SDIO:根据命令可返回CID/CSD寄存器值      |
|  R3  |   48bit    |   SDIO:ACMD41命令的响应，返回OCR寄存器值    |
|  R6  |   48bit    |               已发布的RCA响应               |
|  R7  |   48bit    |       CMD8命令的响应(卡支持电压信息)        |

SD 卡的响应因使用接口不同，比如 SDIO 和 SPI 接口，它们的响应种类以及响应格式也是不同。这里以 SDIO 接口下的 R1 响应为例

![image-20230817125108931](STM32卷二.assets/image-20230817125108931.png)

### 操作模式

- SD 卡系统(包括主机和 SD 卡)定义了 SD 卡的工作模式，在每个操作模式下，SD 卡都有几种状态，状态之间通过命令控制实现卡状态的切换。
- 在SD卡系统(主机和SD卡)定义了两种操作模式：卡识别模式和数据传输模式。
- 系统复位后，主机和SD卡都处于卡识别模式，主机在总线上找设备；当SD卡被主机识别后，SD卡进入到数据传输模式，而主机在总线上所有卡都被识别后也进入数据传输模式。

![image-20230817125308749](STM32卷二.assets/image-20230817125308749.png)

**卡识别模式状态转换**

![image-20230817125418993](STM32卷二.assets/image-20230817125418993.png)

**数据传输模式卡状态转换**

![image-20230817125459766](STM32卷二.assets/image-20230817125459766.png)



## SDIO接口简介

SDIO，全称 Secure Digital Input and Output，即安全数字输入输出接口。多媒体卡（MMC 卡）、SD 存储卡、SDI/O 卡和 CE￾ATA 设备都有 SDIO 接口。

![image-20230817125754440](STM32卷二.assets/image-20230817125754440.png)

STM32F10x系列和F4xx系列控制器只支持SD卡规范版本2.0，即只支持标准容量SD卡和高容量SDHC卡，不支持超大容量SDXC标准卡，所以可以支持最高卡容量是32G

支持三种不同的数据总线模式：1 位(默认)、4 位和 8 位。



### **SDIO框图**

STM32F1 的 SDIO 控制器功能框图，F4系列不同

![image-20230817130000345](STM32卷二.assets/image-20230817130000345.png)

复位后默认情况下 SDIO_D0 用于数据传输。初始化后主机可以改变数据总线的宽度（通过ACMD6 命令设置）。 如果一个多媒体卡接到了总线上，则 SDIO_D0、SDIO_D[3:0]或 SDIO_D[7:0]可以用于数据传输。MMC 版本 V3.31 和之前版本的协议只支持 1 位数据线，所以只能用 SDIO_D0（为了通用性考虑，在程序里面我们只要检测到是 MMC 卡就设置为 1 位总线数据）。 如果一个 SD 或 SDI/O 卡接到了总线上，可以通过主机配置数据传输使用 SDIO_D0 或SDIO_D[3:0]。所有的数据线都工作在推挽模式。 SDIO_CMD 有两种操作模式：

- 用于初始化时的开路模式(仅用于 MMC 版本 V3.31 或之前版本)
- 用于命令传输的推挽模式(SD/SD I/O 卡和 MMCV4.2 在初始化时也使用推挽驱动)



### **SDIO 的时钟**

SDIO 总共有 3 个时钟，分别是

- 卡时钟（SDIO_CK）：每个时钟周期在命令和数据线上传输 1 位命令或数据。对于多媒体卡 V3.31 协议，时钟频率可以在 0MHz 至 20MHz 间变化；对于多媒体卡 V4.0/4.2 协议，时钟频率可以在 0MHz 至 48MHz 间变化；对于 SD 或 SDI/O 卡，时钟频率可以在 0MHz 至 25MHz间变化。
- SDIO 适配器时钟（SDIOCLK）：该时钟用于驱动 SDIO 适配器，其频率等于 AHB 总线频率（HCLK 48MHz），并用于产生 SDIO_CK 时钟。
- AHB 总线接口时钟（HCLK/2）：该时钟用于驱动 SDIO 的 AHB 总线接口，其频率为HCLK/2。AHB总线接口时钟(HCLK/2)，36MHz   

SDIO_CK 与 SDIOCLK 的关系为：SDIO_CK = SDIOCLK / (2 + CLKDIV)

注意：SD卡初始化时，SDIO_CK不可超过400KHz；初始化完成后，可设为最大(不可超过SD卡最大操作频率25MHz)并可更改数据总线宽度(默认只用SDIO_D0进行初始化)



### **SDIO适配器**

提供SD卡特有的功能：产生时钟、发送命令、接收应答、双向传输数据

![image-20230817131301100](STM32卷二.assets/image-20230817131301100.png)

**控制单元**：电源管理和时钟管理功能

**命令通道**

命令通道单元向卡发送命令并从卡接收响应。

![image-20230817131249551](STM32卷二.assets/image-20230817131249551.png)

命令通道状态机(CPSM) 

当写入命令寄存器并设置了使能位，开始发送命令。命令发送完成时，命令通道状态机(CPSM)设置状态标志并在不需要响应时进入空闲状态(见下图)。当收到响应后，接收到的CRC码将会与内部产生的CRC码比较，然后设置相应的状态标志。

![image-20230817131139139](STM32卷二.assets/image-20230817131139139.png)

**数据通道**

数据通道子单元在主机与卡之间传输数据。

![image-20230817131235637](STM32卷二.assets/image-20230817131235637.png)

在时钟控制寄存器中可以配置卡的数据总线宽度。如果选择了4位总线模式，则每个时钟周期四条数据信号线(SDIO_D[3:0])上将传输4位数据；如果选择了8位总线模式，则每个时钟周期八条数据信号线(SDIO_D[7:0])上将传输8位数据；如果没有选择宽总线模式，则每个时钟周期只在SDIO_D0上传输1位数据。

根据传输的方向(发送或接收)，使能时数据通道状态机(DPSM)将进入Wait_S或Wait_R状态：

- 发送：DPSM进入Wait_S状态。如果发送FIFO中有数据，则DPSM进入发送状态，同时数据通道子单元开始向卡发送数据。
- 接收：DPSM进入Wait_R状态并等待开始位；当收到开始位时，DPSM进入接收状态，同时数据通道子单元开始从卡接收数据。

数据通道状态机(DPSM) 

![image-20230817131431589](STM32卷二.assets/image-20230817131431589.png)

**数据FIFO** 

数据FIFO(先进先出)子单元是一个具有发送和接收单元的数据缓冲区。

包括32个字的数据缓冲，和发送与接收电路

TXACT和RXACT标志分别表明当前处于发送还是接收状态，由数据通道子单元对这两个标志互斥的置位



发送FIFO

- SDIO发送数据时，由数据通道单元置位TXACT，表示数据传输正在进行
- 当TXACT位为1，通过APB2接口(F4) / AHB接口(F1)将数据写入到发送FIFO
- 发送FIFO相关状态：TXFIFOF,TXFIFOE,TXFIFOHE,TXDAVL,TXUNDERRUN

| 标志     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| TXFIFOF  | 当所有32个发送FIFO字都有有效的数据时，该标志为高。           |
| TXFIFOE  | 当所有32个发送FIFO字都没有有效的数据时，该标志为高。         |
| TXFIFOHE | 当8个或更多发送FIFO字为空时，该标志为高。该标志可以作为DMA请求。 |
| TXDAVL   | 当发送FIFO包含有效数据时，该标志为高。该标志的意思刚好与TXFIFOE相反。 |
| TXUNDERR | 当发生下溢错误时，该标志为高。写入SDIO清除寄存器时清除该标志。读空FIFO后继续读则导致下溢 |

接收FIFO

- SDIO接收数据时，由数据通道单元置位RXACT
- 当RXACT位为1，通过接收FIFO存放从数据通道接收到的数据
- 接收FIFO的相关状态： RXFIFOF,RXFIFOE,RXFIFOHF,RXDAVL,RXOVERR

| 标志     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| RXFIFOF  | 当所有32个接收FIFO字都有有效的数据时，该标志为高。           |
| RXFIFOE  | 当所有32个接收FIFO字都没有有效的数据时，该标志为高。         |
| RXFIFOHF | 当8个或更多接收FIFO字有有效的数据时，该标志为高。该标志可以作为DMA请求。 |
| RXDAVL   | 当接收FIFO包含有效数据时，该标志为高。该标志的意思刚好与RTXFIFOE相反。 |
| RXOVERR  | 当发生上溢错误时，该标志为高。写入SDIO清除寄存器时清除该标志。写满FIFO后继续写则导致上溢 |

 通过硬件流控：可避免发送模式下溢、可避免接收模式上溢

 

### **命令格式**

命令是用于开始一项操作。命令在CMD线上串行发送，属于半双工模式，命令和响应可分别发送和接收。命令固定长度为48位。

1位 start bit + 1位 transmission bit + 6位 command index + 32位argument + 7位CRC + 1位end bit



SDIO支持2种响应类型，48位短响应和136位长响应，2种类型都有CRC错误检测

![image-20230817132502515](STM32卷二.assets/image-20230817132502515.png)

其中命令索引是寄存器SDIO_CMD 和 参数是寄存器SDIO_ARG

![image-20230817132611928](STM32卷二.assets/image-20230817132611928.png)



### **SDIO块数据传输**

SDIO和SD卡通信一般以数据块的形式进行传输（还有一种是流的方式）



**块读**

![image-20230817133301123](STM32卷二.assets/image-20230817133301123.png)

SD卡在收到主机相关命令后，开始发送数据块到主机，所有数据块都带CRC校验(由硬件自动处理)；单块数据块读时，在收到1个数据块就停止，不需要发送停止命令。

多块数据块读时，SD卡将一直发送数据给主机，直到接到主机发送的STOP命令。



**块写**

![image-20230817133319982](STM32卷二.assets/image-20230817133319982.png)

数据块写操作同数据块读操作基本类似，只是数据块写时，多了一个繁忙判断，新的数据块必须在SD卡非繁忙时发送。

这里的繁忙信号由SD卡拉低SDIO_D0表示，SDIO硬件自动控制，不需要我们软件处理。

## 相关寄存器

|   **寄存器**    |       **名称**        |                **作用**                |
| :-------------: | :-------------------: | :------------------------------------: |
|   SDIO_POWER    |  SDIO电源控制寄存器   |        定义卡时钟的当前功能状态        |
|   SDIO_CLKCR    |  SDIO时钟控制寄存器   | 用于设置SDIO_CK输出时钟(设置分频系数)  |
|    SDIO_ARG     |    SDIO参数寄存器     |    用于存储命令参数(写命令之前先写)    |
|    SDIO_CMD     |    SDIO命令寄存器     |       用于设置命令索引和命令类型       |
|  SDIO_RESPCMD   |  SDIO命令响应寄存器   | 用于存储最后收到的命令响应中的命令索引 |
| SDIO_RESPx(1~4) | SDIO命令响应1~4寄存器 |     用于存放接收到的卡响应部分信息     |
|   SDIO_DTIMER   | SDIO数据定时器寄存器  |  用于存储以卡时钟为周期的数据超时时间  |
|    SDIO_DLEN    |  SDIO数据长度寄存器   |     用于设置需要传输的数据字节长度     |
|   SDIO_DCTRL    |  SDIO数据控制寄存器   |         用于控制数据通道状态机         |
|    SDIO_STA     |    SDIO状态寄存器     |      用于查询SDIO控制器的当前状态      |
|    SDIO_FIFO    |  SDIO数据FIFO寄存器   |        用于接收或者发送FIFO数据        |

**SDIO电源控制寄存器(SDIO_POWER)** 

![image-20230817133423799](STM32卷二.assets/image-20230817133423799.png)

该寄存器复位值为 0，所以 SDIO 的电源是关闭的，我们要启用 SDIO，第一步就是要设置该寄存器最低 2 个位均为 1，让 SDIO 上电，开启卡时钟。



**SDIO 时钟控制寄存器（SDIO_CLKCR）**

SDIO 时钟控制寄存器（SDIO_CLKCR），该寄存器主要用于设置 SDIO_CK 的分配系数，开关等，并可以设置 SDIO 的数据位宽

![image-20230817133504906](STM32卷二.assets/image-20230817133504906.png)

WIDBUS 用于设置 SDIO 总线位宽，正常使用的时候，设置为 1，即 4 位宽度，初始化时为0。BYPASS 用于设置分频器是否旁路，我们一般要使用分频器，所以这里设置为 0，禁止旁路。CLKEN 则用于设置是否使能 SDIO_CK，我们设置为 1。最后，CLKDIV 则用于控制 SDIO_CK 的分频，设置为 1，即可得到 24Mhz 的 SDIO_CK 频率。



**SDIO 参数寄存器（SDIO_ARG）**

该寄存器比较简单，就是一个 32 位寄存器，用于存储命令参数，不过需要注意的是，必须在写命令之前先写这个参数寄存器！

**SDIO 命令响应寄存器（SDIO_RESPCMD）**

该寄存器为 32 位，但只有低 6 位有效，比较简单，用于存储最后收到的命令响应中的命令索引。如果传输的命令响应不包含命令索引，则该寄存器的内容不可预知。

**SDIO 响应寄存器组（SDIO_RESP1~SDIO_RESP4）**

SDIO 响应寄存器组（SDIO_RESP1~SDIO_RESP4），该寄存器组总共由 4 个 32 位寄存器组成，用于存放接收到的卡响应部分信息。如果收到短响应，则数据存放在 SDIO_RESP1 寄存器里面，其他三个寄存器没有用到。而如果收到长响应，则依次存放在SDIO_RESP1~SDIO_RESP4 里面



**SDIO 命令寄存器（SDIO_CMD）**

![image-20230817134027998](STM32卷二.assets/image-20230817134027998.png)

其中低 6 位为命令索引，也就是我们要发送的命令索引号（比如发送 CMD1，其值为 1，索引就设置为 1）。位[7:6]，用于设置等待响应位，用于指示 CPSM 是否需要等待，以及等待类型等。这里的 CPSM，即命令通道状态机，我们就不详细介绍了，请参阅《STM32F10xxx 参考手册_V10（中文版）.pdf》V10 的第 368 页，有详细介绍。命令通道状态机我们一般都是开启的，所以位 10 要设置为 1。



**SDIO 数据定时器寄存器（SDIO_DTIMER）** 在写入数据控制寄存器进行数据传输之前，必须先写入数据定时器寄存器和数据长度寄存器



**SDIO 数据长度寄存器（SDIO_DLEN）**

SDIO 数据长度寄存器（SDIO_DLEN）低 25 位有效，用于设置需要传输的数据字节长度。对于块数据传输，该寄存器的数值，必须是数据块长度（通过 SDIO_DCTRL 设置）的倍数。



**SDIO 数据控制寄存器（SDIO_DCTRL）**

![image-20230817134302739](STM32卷二.assets/image-20230817134302739.png)

用于控制数据通道状态机（DPSM），包括数据传输使能、传输方向、传输模式、DMA 使能、数据块长度等信息，都是通过该寄存器设置。我们需要根据自己的实际情况，来配置该寄存器，才可正常实现数据收发。



**SDIO状态寄存器（SDIO_STA）**

![image-20230817134358853](STM32卷二.assets/image-20230817134358853.png)



**SDIO 数据 FIFO 寄存器（SDIO_FIFO）**

接收和发送FIFO是32位宽度读或写一组寄存器，它在连续的32个地址上包含32个寄存器，CPU可以使用FIFO读写多个操作数。

我们操作SDIO_FIFO（不论读出还是写入）必须是以4字节对齐的内存进行操作，否则将导致出错。



## HAL库

SD卡HAL库驱动文件：stm32f1xx_hal_sd.c和stm32f1xx_ll_sdmmc.c

### 常用函数

|           驱动函数            |       关联寄存器        |         功能描述         |
| :---------------------------: | :---------------------: | :----------------------: |
|   __HAL_RCC_SDIO_CLK_ENABLE   |    AHBENR / APB2ENR     |     用于SDIO时钟使能     |
|          HAL_SD_Init          | SDIO_CLKCR/CMD/ARG/RESP |      用于初始化SD卡      |
|        HAL_SD_MspInit         |     初始化回调函数      | 用于初始化SDIO的底层硬件 |
| HAL_SD_ConfigWideBusOperation | SDIO_CLKCR/CMD/ARG/RESP |       配置总线宽度       |
|       HAL_SD_ReadBlocks       | SDIO_DCTRL/CMD/ARG/RESP |        读取块数据        |
|      HAL_SD_WriteBlocks       | SDIO_DCTRL/CMD/ARG/RESP |        写入块数据        |
|      HAL_SD_GetCardInfo       |                         |        获取卡信息        |
|      HAL_SD_GetCardState      |                         |        获取卡状态        |
|       HAL_SD_GetCardCID       |                         |         获取卡ID         |

```
/*  读取块数据  
*   参数1：SDIO句柄
*   参数2：接收地址缓冲区
*   参数3：块地址
*   参数4：读取几个块（一个快512字节）
*   参数5：超时时间，HAL库内部有个宏#define SD_TIMEOUT  ((uint32_t)100000000)
*/
HAL_StatusTypeDef HAL_SD_ReadBlocks(SD_HandleTypeDef *hsd, uint8_t *pData, uint32_t BlockAdd, uint32_t NumberOfBlocks, uint32_t Timeout)

HAL_StatusTypeDef HAL_SD_WriteBlocks(SD_HandleTypeDef *hsd, uint8_t *pData, uint32_t BlockAdd, uint32_t NumberOfBlocks, uint32_t Timeout)    

/*  获取卡状态
*   返回值：
*       HAL_SD_CARD_READY
*       HAL_SD_CARD_IDENTIFICATION
*       HAL_SD_CARD_STANDBY
*       HAL_SD_CARD_TRANSFER
*       HAL_SD_CARD_SENDING
*       HAL_SD_CARD_RECEIVING
*       HAL_SD_CARD_PROGRAMMING
*       HAL_SD_CARD_DISCONNECTED
*       HAL_SD_CARD_ERROR
*/
HAL_SD_CardStateTypeDef HAL_SD_GetCardState(SD_HandleTypeDef *hsd) 
```

**结构体**

SDIO外设相关结构体：

- SD_HandleTypeDef：SDIO句柄
- SDIO_CmdInitTypeDef：命令相关结构体
- SDIO_DataInitTypeDef：数据相关结构体
- HAL_SD_CardInfoTypeDef：卡相关结构体

```
typedef struct
    SD_TypeDef *Instance;   /* SD 相关寄存器基地址 */
    SD_InitTypeDef Init;    /* SDIO 初始化变量,宏定义替代SDIO_InitTypeDef */
    HAL_LockTypeDef Lock;   /* 互斥锁，用于解决外设访问冲突 */
    uint8_t *pTxBuffPtr;    /* SD 发送数据指针 */
    uint32_t TxXferSize;    /* SD 发送缓存按字节数的大小 */
    uint8_t *pRxBuffPtr;    /* SD 接收数据指针 */
    uint32_t RxXferSize;    /* SD 接收缓存按字节数的大小 */
    __IO uint32_t Context;  /* HAL 库对 SD 卡的操作阶段 */
    __IO HAL_SD_StateTypeDef State; /* SD 卡操作状态 */
    __IO uint32_t ErrorCode;    /* SD 卡错误代码 */
    DMA_HandleTypeDef *hdmatx;  /* SD DMA 数据发送指针 */
    DMA_HandleTypeDef *hdmarx;  /* SD DMA 数据接收指针 */
    HAL_SD_CardInfoTypeDef SdCard;  /* SD 卡信息的 */
    uint32_t CSD[4];    /* 保存 SD 卡 CSD 寄存器信息 */
    uint32_t CID[4];    /* 保存 SD 卡 CID 寄存器信息 */
}SD_HandleTypeDef;
typedef struct{
    uint32_t ClockEdge;            /* 数据或指令变化的时钟沿 */
    uint32_t ClockBypass;          /* 是否设置旁路分频器 */
    uint32_t ClockPowerSave;       /* 空闲状态，是否输出时钟 */
    uint32_t BusWide;              /* 设置SDIO总线位宽 */
    uint32_t HardwareFlowControl;  /* 用于使能硬件流控制 */
    uint32_t ClockDiv;             /* 设置SDIO时钟分频 */ 
}SDIO_InitTypeDef;
//注意：在搜索宏定义时 SDMMC_LL_XXX_XXX搜索不到将SDMMC替换为SDIO就能搜索到了，SDIO_LL_XXX_XXX
/* 数据或指令变化的时钟沿 */
#define SDIO_CLOCK_EDGE_RISING 
#define SDIO_CLOCK_EDGE_FALLING
/* 是否设置旁路分频器 */
#define SDIO_CLOCK_BYPASS_DISABLE   //不开启
#define SDIO_CLOCK_BYPASS_ENABLE
/* 空闲状态，是否输出时钟 */
#define SDIO_CLOCK_POWER_SAVE_DISABLE   
#define SDIO_CLOCK_POWER_SAVE_ENABLE
/* 设置SDIO总线位宽 */
#define SDIO_BUS_WIDE_1B            // 初始化使用
#define SDIO_BUS_WIDE_4B            // 想提高速度时使用
#define SDIO_BUS_WIDE_8B
/* 用于使能硬件流控制，控制FIFO溢出 */
#define SDIO_HARDWARE_FLOW_CONTROL_DISABLE
#define SDIO_HARDWARE_FLOW_CONTROL_ENABLE
/* 设置SDIO时钟分频 */ 
#define SDIO_INIT_CLK_DIV     ((uint8_t)0x76)   /* 48MHz / (SDMMC_INIT_CLK_DIV + 2) < 400KHz */
#define SDIO_TRANSFER_CLK_DIV ((uint8_t)0x4)    /* SDIO Data Transfer Frequency (25MHz max) */

/*----------------------------HAL库内部使用------------------------------------*/
typedef struct{
    uint32_t Argument;      /* 命令参数 */
    uint32_t CmdIndex;      /* 命令索引 */
    uint32_t Response;      /* 响应类型 */
    uint32_t WaitForInterrupt;  /* 等待中断请求 */
    uint32_t CPSM;          /* 是否使用命令通道状态机 */
}SDIO_CmdInitTypeDef;
// SDIO_DataInitTypeDef关联的函数SDIO_ConfigData，发送/接收数据块会使用到。
typedef struct{
    uint32_t DataTimeOut;   /* 数据超时时间 */
    uint32_t DataLength;    /* 数据长度 */
    uint32_t DataBlockSize; /* 数据块大小 */
    uint32_t TransferDir;   /* 传输方向(读/写) */
    uint32_t TransferMode;  /* 传输模式(块/流)*/
    Uint32_t DPSM;          /* 是否使用数据通道状态机 */
}SDIO_DataInitTypeDef;
// 通过HAL_SD_GetCardInfo函数获取卡信息
typedef struct{
    uint32_t CardType;      /* 卡类型 */
    uint32_t CardVersion;   /* 卡版本 */
    uint32_t Class;         /* 卡类型 */       
    uint32_t RelCardAdd;    /* 卡相对地址 */
    uint32_t BlockNbr;      /* 整个卡的块数 */
    uint32_t BlockSize;     /* 每个块的字节数 */ 
    uint32_t LogBlockNbr;   /* 整个卡的逻辑块数 */
    uint32_t LogBlockSize;  /* 逻辑块大小 */ 
}HAL_SD_CardInfoTypeDef;
typedef struct{
  __IO uint8_t  ManufacturerID;  /*!< Manufacturer ID       */
  __IO uint16_t OEM_AppliID;     /*!< OEM/Application ID    */
}HAL_SD_CardCIDTypeDef;
```

### 案例

没有使用文件管理器，将SD卡当作Flash来读，指定块读/写

```
#define SDIO_PORT GPIOC
#define SDIO_CMD_PORT GPIOD
#define SDIO_PIN_D0  GPIO_PIN_8
#define SDIO_PIN_D1  GPIO_PIN_9
#define SDIO_PIN_D2  GPIO_PIN_10
#define SDIO_PIN_D3  GPIO_PIN_11
#define SDIO_PIN_CLK  GPIO_PIN_12
#define SDIO_PIN_CMD  GPIO_PIN_2
SD_HandleTypeDef SD_HandleStructure;
void SD_Init(void){         
    SD_HandleStructure.Instance = SDIO;                                 
    SD_HandleStructure.Init.ClockEdge = SDIO_CLOCK_EDGE_RISING;         /* 上升沿 */
    /* 不使用bypass模式，直接用HCLK进行分频得到SDIO_CK */
    SD_HandleStructure.Init.ClockBypass = SDIO_CLOCK_BYPASS_DISABLE;    
    SD_HandleStructure.Init.ClockPowerSave = SDIO_CLOCK_POWER_SAVE_DISABLE; /* 空闲时不关闭时钟电源 */
    SD_HandleStructure.Init.BusWide = SDIO_BUS_WIDE_1B;                 /* 1位数据线 */
    SD_HandleStructure.Init.HardwareFlowControl =  SDIO_HARDWARE_FLOW_CONTROL_ENABLE;   /* 开启硬件流控 */
    SD_HandleStructure.Init.ClockDiv = SDIO_TRANSFER_CLK_DIV;           /* SD传输时钟频率最大25MHZ */
    HAL_SD_Init(&SD_HandleStructure);
}
void HAL_SD_MspInit(SD_HandleTypeDef *hsd){ 
    if(SDIO ==hsd->Instance){
        __HAL_RCC_SDIO_CLK_ENABLE();
        __HAL_RCC_GPIOC_CLK_ENABLE();
        __HAL_RCC_GPIOD_CLK_ENABLE();
        GPIO_InitTypeDef GPIO_InitStructure;
        GPIO_InitStructure.Mode = GPIO_MODE_AF_PP;      /* 推挽复用 */
        GPIO_InitStructure.Pin = SDIO_PIN_D0 | SDIO_PIN_D1 | SDIO_PIN_D2 |SDIO_PIN_D3 |SDIO_PIN_CLK;
        GPIO_InitStructure.Pull = GPIO_NOPULL;
        GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;
        HAL_GPIO_Init(SDIO_PORT,&GPIO_InitStructure); 
        GPIO_InitStructure.Pin = GPIO_PIN_2;
        HAL_GPIO_Init(SDIO_CMD_PORT,&GPIO_InitStructure);  
    }
}
/** 获取Card信息
*   参数：SD卡信息结构体
**/
uint8_t GetSD_Card_Info(HAL_SD_CardInfoTypeDef *cardInfo){
    return HAL_SD_GetCardInfo(&SD_HandleStructure,cardInfo);
}
/* 获取卡状态 */
uint8_t GetSD_State(void){
    if(HAL_SD_GetCardState(&SD_HandleStructure) == HAL_SD_CARD_TRANSFER ){  /* 是否正在传输 */
        return 0x00;            /* 传输完成 */  
    }else{
        return 0x01;            /* 忙 */
    }
}
/* 读取扇区 */
uint8_t SD_ReadDisk(uint8_t *pData,uint32_t Addr,uint32_t cnt){
    uint32_t timeout = 100000;
    uint8_t sta;
    sta = HAL_SD_ReadBlocks(&SD_HandleStructure,pData,Addr,cnt,HAL_TIMEOUT);
    
    while(GetSD_State() != 0x00){   /* 如果忙 则等待100000个数 */
        if(timeout-- == 0){
            sta = 0x01;
        }
    } 
    return sta;
}
/*  写扇区 */
uint8_t SD_WriteDisk(uint8_t *pData,uint32_t Addr,uint32_t cnt){
    uint32_t timeout = 100000;
    uint8_t sta;
    sta = HAL_SD_WriteBlocks(&SD_HandleStructure,pData,Addr,cnt,HAL_TIMEOUT); 
    while(GetSD_State() != 0x00){
        if(timeout-- == 0){
            sta = 0x01;
        }
    } 
    return sta;
}
uint8_t readBuf[500] = {0};
uint8_t writeBuf[500] = {0};
HAL_SD_CardInfoTypeDef SD_CardInfoStructure;    /* Card 信息结构体 */
HAL_SD_CardCIDTypeDef SD_CardCID;               /* Card ID结构体*/
int main(void){
    HAL_Init();                         /* 初始化HAL库 */
    sys_stm32_clock_init(RCC_PLL_MUL9); /* 设置时钟, 72Mhz */
    delay_init(72);                     /* 延时初始化 */
    usart_init(115200);                 /* 串口初始化为115200 */
    led_init();                         /* 初始化LED */
    lcd_init();                         /* 初始化LCD */
    key_init();                         /* 初始化key */
    SD_Init();                          /* 初始化SDIO */
    
    lcd_show_string(30, 110, 200, 16, 16, "Card ID:", RED);  
    lcd_show_string(30, 130, 200, 16, 16, "Card Size:", RED);   
    
    HAL_SD_GetCardCID(&SD_HandleStructure,&SD_CardCID);         /* 获取CardID */
    lcd_show_num(130,110,SD_CardCID.ManufacturerID,8,16,BLUE);
    
    GetSD_Card_Info(&SD_CardInfoStructure);                     /* 获取Card信息 */
    /* 计算内存大小 MB */
    uint64_t cardSize = (SD_CardInfoStructure.LogBlockNbr * SD_CardInfoStructure.LogBlockSize) >> 20; 
    lcd_show_num(130,130,cardSize,8,16,BLUE);

    uint8_t key;
    while (1) {
        key = key_scan(0);
        if(key == 1){
            SD_ReadDisk(readBuf,0,1);                       /* 从0扇区读取1*512字节的内容 */
            for(uint16_t u = 0;u < sizeof(readBuf);u++){
                printf("%d ",readBuf[u]);
            }
        }
        if(key == 2){
            for(uint16_t u = 0;u < sizeof(writeBuf);u++){
                writeBuf[u] = 1;
            }
            SD_WriteDisk(writeBuf,0,1);
        }
        LED0_TOGGLE();                  /* LED0(RED) 闪烁 */
        delay_ms(500);
    }
}
```

## FATFS

使用文件系统前，需先对存储设备进行格式化，擦除原来的数据，在存储设备上建立一个文件分配表和目录。

**FAT文件系统简介**

![image-20230817173119754](STM32卷二.assets/image-20230817173119754.png)

系统引导扇区：引导程序，以及文件系统信息(扇区字节数/每簇扇区数/保留扇区数等)

文件分配表：记录文件存储中簇与簇之间连接的信息

根目录：存在所有文件和子目录信息(文件名/文件夹名/创建时间/文件大小)

数据区：文件等数据存放地方，占用大部分的磁盘空间



FAT文件系统用“簇” 作为数据单元，一个“簇”由一组连续的扇区组成，而一个扇区的大小为512字节。

所有的簇从2开始进行编号，每个簇都有自己的地址编号，用户文件和数据都存储在簇中。

### **FATFS文件系统简介**

FATFS 是一个完全免费开源的 FAT/exFAT 文件系统模块，专门为小型的嵌入式系统而设计。它完全用标准 C 语言（ANSI C C89）编写，所以具有良好的硬件平台独立性，只需做简单的修改就可以移植到 8051、PIC、AVR、ARM、Z80、RX 等系列单片机上。它支持 FATl2、FATl6 和FAT32，支持多个存储媒介；有独立的缓冲区，可以对多个文件进行读／写，并特别对 8 位单片机和 16 位单片机做了优化。



下载链接 http://elm-chan.org/fsw/ff/archives.html

FATFS 模块的层次结构图

![image-20230817173320885](STM32卷二.assets/image-20230817173320885.png)



最顶层是应用层，使用者无需理会FATFS的内部结构和复杂的FAT协议，只需要调用FATFS模块提供给用户的一系列应用接口函数，如 f_open，f_read，f_write 和 f_close 等，就可以像在PC 上读／写文件那样简单。 中间层 FATFS 模块，实现了 FAT 文件读／写协议。FATFS 模块提供的是 ff.c 和 ff.h。除非有必要，使用者一般不用修改，使用时将头文件直接包含进去即可。 需要我们编写移植代码的是 FATFS 模块提供的底层接口，它包括存储媒介读／写接口（diskI/O）和供给文件创建修改时间的实时时钟。



**目录结构**

解压后可以得到两个文件夹：documents 和 source。documents 里面主要是对 FATFS 的介绍，而 source 里面才是我们需要的源码。source 文件夹详情

| **文件名**  |             **功能名**             |    **说明**    |
| :---------: | :--------------------------------: | :------------: |
|  ffconf.h   |         FATFS模块配置文件          | 根据需求来配置 |
|    ff.h     |   FATFS和应用模块共用的包含文件    |   不需要修改   |
|    ff.c     |     FATFS模块源码(文件系统API)     |   不需要修改   |
|  diskio.h   | FATFS和disk  I/O模块共用的包含文件 |   不需要修改   |
|  diskio.c   |   FATFS和disk  I/O模块接口层文件   | 与硬件相关代码 |
| ffunicode.c |    FATFS所支持的字体代码转换表     |   不需要修改   |
| ffsystem.c  |     FATFS的OS相关函数示例代码      |     没用到     |

FATFS 模块在移植的时候，我们一般只需要修改 2 个文件，即 ffconf.h 和 diskio.c。FATFS模块的所有配置项都是存放在 ffconf.h 里面，我们可以通过配置里面的一些选项 完整介绍 http://elm-chan.org/fsw/ff/doc/config.html 

### 配置文件

ffconf.h 

![image-20230817173943962](STM32卷二.assets/image-20230817173943962.png)



### **diskio.c文件** 

- disk_initialize 初始化磁盘驱动器 
- disk_status 获取磁盘状态
- disk_read 从磁盘驱动器读扇区
- disk_write 从磁盘驱动器写扇区
- disk_ioctl 控制设备实现指定功能，用于辅助FATFS中其他API
- get_fattime  获取当前时间

**disk_initialize函数**

![image-20230817174340353](STM32卷二.assets/image-20230817174340353.png)

需要将初始化函数添加到此函数中，如我们自己读取SD或者AT25C02等的初始化函数放到此函数

这个函数不需要自己调用，在使用f_mount挂载时自动调用



**disk_status函数**

![image-20230817174354250](STM32卷二.assets/image-20230817174354250.png)

**disk_read函数**

![image-20230817174403808](STM32卷二.assets/image-20230817174403808.png)

**disk_write函数**

![image-20230817174417835](STM32卷二.assets/image-20230817174417835.png)



**disk_ioctl函数**

![image-20230817174428314](STM32卷二.assets/image-20230817174428314.png)

**get_fattime函数**

![image-20230817174450941](STM32卷二.assets/image-20230817174450941.png)

### **FATFS开放函数**

FATFS 提供了很多 API 函数，这些函数 FATFS 的自带介绍文件里面都有详细的介绍(包括参考代码)，我们这里就不多说了。这里需要注意的是，在使用FATFS的时候，必须先通过**f_mount**函数注册一个工作区，才能开始后续 API 的使用

```c
/*_____________________________文件操作_________________________________*/
f_open		打开/创建一个文件
f_close		关闭一个打开的文件
f_read 		从文件中读取数据
f_write		往文件中写数据
f_gets		读一个字符串
f_putc 		写一个字符
f_puts		写一个字符串
f_printf	写一个格式化的字符串
f_lseek		移动文件读/写指针
f_tell		获取当前读/写指针
f_size		获取文件大小
/*_____________________________目录操作_________________________________*/
f_opendif	打开一个目录
f_closedir	关闭一个已经打开的目录
f_readdir	读取目录条目
f_mkdir		创建一个新目录
f_unlink	删除一个文件或目录
f_rename	重命名/移动一个文件或文件夹
/*_____________________________劵管理_________________________________*/  
f_mount		注册/注销一个工作区
f_mkfs		格式化，创建一个文件系统
f_getfree	获取磁盘信息以及空闲簇数量
f_setlabel	设置盘符(磁盘名字)
f_getlabel	获取盘符
```

### 移植FATFS

移植一个最简单的FATFS系统，使用SDIO中的案例

步骤：

1. 前期工作:准备好一个带有存储设备驱动的工程(SPI实验/SD卡实验)FATFS文件系统开源库
2. 复制FATFS文件到工程文件夹下 并 将移植文件添加到工程中(设置包含路径)
3. 修改ffconf.h的配置项：FF_FS_NORTC / FF_USE_STRFUNC /FF_CODE_PAGE / FF_VOLUMES等
4. 修改diskio.c文件5个函数：disk_initialize/status/read/write/ioctl
5. 编写测试代码：最简读写：f_mount、f_open、f_write、f_read、f_close

**修改ffconf.h配置项**

```h
#define FF_USE_STRFUNC	1		/* 可以使用f_gets(), f_putc(), f_puts()等函数 */
#define FF_CODE_PAGE	936		/* 简体中文 */
#define FF_VOLUMES		1		/* 一个卷积，使用的是SD卡*/
#define FF_USE_LABEL	1		/* 可以更改SD卡名称 */
#define FF_FS_NORTC		1		/* get_fattime 不使用RTC时间，使用下面宏定义的时间*/
#define FF_NORTC_MON	4
#define FF_NORTC_MDAY	6
#define FF_NORTC_YEAR	2000
```

**修改diskio.c**

```c
#include "ff.h"         /* Obtains integer types */
#include "diskio.h"     /* Declarations of disk functions */
#include "./BSP/SD/sdio.h"  /* 引入自己写的SDIO项目 */
#define DEV_SD      0       /*SD卡*/
/*disk 状态*/
DSTATUS disk_status (BYTE pdrv      ){
    return RES_OK; /* 直接返回一个OK */
}
/*f_mount 会自动调用这个函数*/
DSTATUS disk_initialize (BYTE pdrv){  
    if(pdrv == DEV_SD){
        SD_Init();          /* 自己写的SDIO初始化函数，可以添加一些判断 */
    }
    return RES_OK;
}
DRESULT disk_read (BYTE pdrv,       BYTE *buff,LBA_t sector,UINT count){
    if(pdrv == DEV_SD){
        SD_ReadDisk((uint8_t *)buff,sector,count);         /* 自己写的SDIO读函数，可以添加一些判断 */
    }
    return RES_OK;
}

#if FF_FS_READONLY == 0         /* 只有这一项宏为0时才可以进行写操作，否则就是只读*/
DRESULT disk_write (BYTE pdrv,const BYTE *buff,LBA_t sector,UINT count){
    if(pdrv == DEV_SD){
        SD_WriteDisk((uint8_t *)buff,sector,count);         /* 自己写的SDIO读函数，可以添加一些判断 */
    }
    return RES_OK;
}

#endif
DRESULT disk_ioctl (BYTE pdrv,BYTE cmd,void *buff){
    DRESULT res;
    if (pdrv == DEV_SD){
        switch (cmd) {
            case CTRL_SYNC :
                res = RES_OK;
                break;
            case GET_SECTOR_SIZE :          /* 扇区大小 */
                *(DWORD *)buff = 512;
                break;
            case GET_BLOCK_SIZE :           /* 块尺寸 */
                *(DWORD *)buff  = SD_HandleStructure.SdCard.BlockSize;      /* 自己编写的SDIO模块中定义的SD卡结构体 */
                res = RES_OK;
                break;
            case GET_SECTOR_COUNT :         /* 块数目 */
                *(DWORD *)buff  = 
                    ((long long)SD_HandleStructure.SdCard.BlockNbr * SD_HandleStructure.SdCard.BlockSize) / 512;
                res = RES_OK;
                break;
            default:
                res = RES_PARERR;
                break;
        }
    }
    return res;
}
#include "./BSP/SD/sdio.h"
#include "ff.h"         /* FATFS */

FATFS FS_Obj;       /* 每个逻辑驱动器的文件系统对象*/
FIL FIL_Obj;        /* 文件结构体 */ 
DIR dir;            /* 目录 */
FILINFO filinfo;    /* 目录 */
/* 1 挂载SD卡设备 */ 
void SD_Mount(void){
    FRESULT ret;  
    ret = f_mount(&FS_Obj, "0:", 1);        /* 挂载 */
    if (ret){
        printf("mount fail %d \r\n", ret);
    }else{
        printf("mount OK\r\n");
    }
}
void SD_OpenReadWrite(void){
    FRESULT ret;             
    uint8_t rd_buf[255] = {0};              /* 读取缓冲区 */
    uint16_t fsize = 0;
    uint16_t rd_conut, wd_conut;
    /* 2 打开文件(需要注意要关闭文件f_close) */
    ret = f_open(&FIL_Obj,"0:rain.txt",FA_READ | FA_WRITE | FA_OPEN_ALWAYS);   /* 打开0:rain.txt 读 写*/
    if (ret){
        printf("open fail %d \r\n", ret);
    }else{
        printf("open OK\r\n");
    }
    /* 3 数据写入 */ 
    f_lseek(&FIL_Obj, 0);                               /* 指针偏移到文件开头，即在文件开头进行写入 */
    f_write(&FIL_Obj, "337337", 6, (UINT *)&wd_conut);  /* 写入337337 6个数据到FIL_Obj代表的文件中，参数4：写入几个数据*/
    f_printf(&FIL_Obj, "hello_world");                  /* 写入数据的另一种方式 */
    /* 4 读取数据 */
    f_lseek(&FIL_Obj, 0);                               /* 指针偏移到文件开头，在文件开头读 */
    fsize  = f_size(&FIL_Obj);                          /* 获得文件中数据大小 */
    f_read(&FIL_Obj, rd_buf, fsize, (UINT *)&rd_conut);
    if (rd_conut != 0){
        printf("rd_conut:%d rd_buf:%s \r\n",rd_conut, rd_buf);
    }  
    f_close(&FIL_Obj);
}
/* 打开指定目录 */
void SD_ScanDir(uint8_t * path){
    FRESULT res; 
    res = f_opendir(&dir,(const TCHAR*)path);
    if (res == FR_OK) {
        printf("\r\n"); 
        while(1){
            res = f_readdir(&dir, &filinfo);   /* 读取目录下的一个文件 */
            if (res != FR_OK || filinfo.fname[0] == 0) break; /* 错误了/到末尾了,退出 */
            //if (fileinfo.fname[0] == '.') continue;           /* 忽略上级目录 */
            printf("%s/", path);                /* 打印路径 */
            printf("%s\r\n",filinfo.fname);   /* 打印文件名 */
        } 
    }
    f_closedir(&dir);
}
int main(void){
    HAL_Init();                         /* 初始化HAL库 */
    sys_stm32_clock_init(RCC_PLL_MUL9); /* 设置时钟, 72Mhz */
    delay_init(72);                     /* 延时初始化 */
    usart_init(115200);                 /* 串口初始化为115200 */
    led_init();                         /* 初始化LED */
    lcd_init();                         /* 初始化LCD */
    key_init();                         /* 初始化key */
    
    uint8_t buf[20] = {0};
    
    SD_Mount();                         /* 必须先挂载设置，才能进行下一步操作 */
    SD_OpenReadWrite();
    SD_ScanDir((uint8_t *)"0:");
    
    /* 设置劵名 */
    f_setlabel((const TCHAR *)"0:Yy");
    f_getlabel((const TCHAR *)"0:", (TCHAR *)buf , 0);
    printf("label:%s \r\n", buf);
    while (1){ 
    }
}
```













# 低功耗

## 电源控制PWR

### **电源系统**

![image-20230809130919099](STM32卷二.assets/image-20230809130919099.png)

在电源概述框图中我们划分了 3 个区域①②③，分别是独立的 A/D 转换器供电和参考电压、电压调节器、电池备份区域。

1. 独立的 A/D 转换器供电和参考电压（VDDA供电区域） VDDA 供电区域，主要是 ADC 电源以及参考电压，STM32 的 ADC 模块配备独立的供电方式，使用了 VDDA 引脚作为输入，使用 VSSA 引脚作为独立地连接，VREF 引脚为提供给 ADC 的参考电压。
2. 电压调节器（VDD /1.8V 供电区域） 电压调节器是 STM32 的电源系统中最核心部分，连接 VDD 供电区域和 1.8 供电区域。VDD供电来自于 VSS 和 VDD，给 I/O 电路以及待机电路供电，电压调节器主要为备份域以及待机电路以外的所有数字电路供电，其中包括内核、数字外设以及RAM，调节器的输出电压约为1.8V，因此由调压器供电的区域称为 1.8V 供电区域。电压调节器根据应用方式不同有三种不同的工作模式。在运行模式下，调节器以正常工作模式为内核、内存和外设提供 1.8V；在停止模式下，调节器以低功耗模式提供 1.8V 电源，以保存寄存器和 SRAM 的内容。在待机模式下，调节器停止供电，除了备用电路和备份域外，寄存器和 SRAM 的内容全部丢失。
3. 电池备份区域（后备供电区域） 电池备份区域也就是后备供电区域，使用电池或者其他电源连接到 VBAT 脚上，当 VDD 断电时，可以保存备份寄存器的内容和维持 RTC 的功能。同时 VBAT 引脚也为 RTC 和 LSE 振荡器供电，这保证了当主要电源被切断时，RTC 能够继续工作。切换到 VBAT 供电由复位模块中的掉电复位功能控制。

### 电源监控

电源监控即对某些电源电压（VDD / VDDA / VBAT）进行监控

POR/PDR监控器、PVD监控器、 BOR监控器、AVD监控器、VBAT阈值、温度阈值

- POR/PDR（power on/down reset）：上电/掉电复位
- PVD（programmable voltage detector）：监控VDD 电压
- BOR（brown out reset）：欠压复位
- AVD（analog voltage detector）：监控VDDA电压
- VBAT阈值（battery voltage thresholds）：监控VBAT电池电压
- 温度阈值（temperature thresholds）：监控结温

**上电复位(POR)/掉电复位(PDR)**

上电时，当 VDD 低于指定 VPOR 阈值时，系统无需外部复位电路便会保持复位模式。一旦VDD 电源电压高于 VPOR 阈值，系统便会退出复位状态，芯片正常工作。掉电时，当 VDD 低于指定 VPDR阈值时，系统就会保持复位模式。POR 与 PDR 的复位电压阈值是固定的，VPOR 阈值（典型值）为 1.92V，VPDR 阈值 （典型值）为 1.88V。

![image-20230809140240657](STM32卷二.assets/image-20230809140240657.png)

**可编程电压检测器(PVD)**

监视供电电压VDD。当电压下降到设定阈值以下时产生中断(PVD中断EXTI16)，通知软件做紧急处理；当电压恢复到设定阈值以上时产生中断，通知软件供电恢复(PWR_CR设置)。

供电下降的阈值和上升的阈值有固定差值，是为了防止电压在阈值上下小幅度抖动，而频繁产生中断。

![image-20230809140231361](STM32卷二.assets/image-20230809140231361.png)

PVD 阀值有 8 个等级，有上升沿和下降沿的区别，具体由 PWR_CSR 寄存器中的 PVDO 位决定。

![image-20230809140518630](STM32卷二.assets/image-20230809140518630.png)





### **电源管理**

在 STM32 的正常工作中，具有四种工作模式，运行、睡眠、停止以及待机。在上电复位后，STM32 处于运行状态时，当内核不需要继续运行，就可以选择进入后面的三种模式降低功耗。

![image-20230809132133618](STM32卷二.assets/image-20230809132133618.png)



![1691557961647](STM32卷二.assets/1691557961647.png)



**睡眠模式**

进入睡眠模式，Cortex_M3 内核停止，所有外设包括 Cortex_M3 核心的外设，如 NVIC、系统时钟（SysTick）等仍在运行，有两种进入睡眠模式的模式 WFI 和 WFE。WFI(Wait for interrupt 等待中断)和 WFE(Wait for event等待事件)是内核指令，会调用一些汇编指令，更详细的描述可以查看《CM3 权威指南》。睡眠后唤醒的方式即由等待“中断”唤醒和“事件”唤醒。

![1691558091484](STM32卷二.assets/1691558091484.png)



**停止模式** 进入停止模式，所有的时钟都关闭，所有的外设也就停止了工作。但是 VDD电源是没有关闭的，所以内核的寄存器和内存信息都保留下来，等待重新开启时钟就可以从上次停止的地方继续执行程序。 值得注意的是：当电压调节器处于低功耗模式下，当系统从停止模式退出时，将会有一段额外的启动延时。如果在停止模式期间保持内部调节器开启，则退出启动时间会缩短，但相应的功耗会增加。

停止模式退出后：默认退出后HSI RC振荡器被选为系统时钟，如果我们使用的是HSE则需要重新初始化

![1691558552307](STM32卷二.assets/1691558552307.png)

**待机模式**

待机模式可实现最低功耗。该模式是在 CM3 深睡眠模式时关闭电压调节器，整个 1.8V 供电区域被断电。PLL、HSI 和 HSE 振荡器也被断电。除备份域（RTC 寄存器、RTC 备份寄存器和备份 SRAM）和待机电路中的寄存器外，SRAM 和其他寄存器内容都将丢失。不过如果我们使能了备份区域（备份 SRAM、RTC、LSE），那么待机模式下的功耗，将达到 3.8uA 左右。

待机模式下，所有I/O引脚处于高阻态，除了复位引脚、被使能的唤醒引脚，不能下载程序，必须退出待机模式才能下载

![1691558605256](STM32卷二.assets/1691558605256.png)

## 相关寄存器

| **寄存器** | **名称**            | **作用**                                                     |
| ---------- | ------------------- | ------------------------------------------------------------ |
| SCB_SCR    | 系统控制寄存器      | 选择休眠和深度休眠模式，用于其他低功耗特性的控制             |
| PWR_CR     | 电源控制寄存器      | 可以设置低功耗相关（清除标记位、模式）/设置PVD检测的电压阈值以及使能PVD |
| PWR_CSR    | 电源控制/状态寄存器 | 用于查看系统当前状态（待机/唤醒标志 使能唤醒引脚、PVD输出）  |

**系统控制寄存器(SCB_SCR)**

![image-20230809132815425](STM32卷二.assets/image-20230809132815425.png)

进入停止模式 or 待机模式 ，SLEEPDEEP置1

**电源控制寄存器PWR_CR**

![image-20230809141217046](STM32卷二.assets/image-20230809141217046.png)

停止模式：PDDS清0，LPDS选调节器模式

待机模式：清除唤醒位 CWUF

**电源控制状态寄存器PWR_CSR**

![image-20230809141426546](STM32卷二.assets/image-20230809141426546.png)

待机模式下 ，使用WKUP引脚唤醒并需要清除WUF标记位

WKUP上升沿才能唤醒待机状态

PVDO:通过该位判断此时电压与设定的电压阈值关系

## 常用函数

### 电源管理

|          驱动函数           |   关联寄存器   |       功能描述       |
| :-------------------------: | :------------: | :------------------: |
| HAL_PWR_EnterSLEEPMode(...) |    SCB_SCR     |     进入睡眠模式     |
|  HAL_PWR_EnterSTOPMode(…)   | PWR_CR/SCB_SCR |     进入停止模式     |
| HAL_PWR_EnterSTANDBYMode(…) | PWR_CR/SCB_SCR |     进入待机模式     |
| HAL_PWR_EnableWakeUpPin(…)  |    PWR_CSR     | 使能WKUP管脚唤醒功能 |
|   __HAL_PWR_CLEAR_FLAG(…)   |     PWR_CR     |  清除PWR的相关标记   |
|   __HAL_PWR_GET_FLAG(...)   |     PWR_CR     |      获取标志位      |
| __HAL_RCC_PWR_CLK_ENABLE(…) |  RCC_APB1ENR   |     使能电源时钟     |

```c
/**
*	参数1：指定稳压器的状态。有两个选择，PWR_MAINREGULATOR_ON 表示稳压器处于正常模式，PWR_LOWPOWERREGULATOR_ON 表示稳压器处于低功耗模式。对应的是 PWR_CR 寄存器的 LPDS 位的设置（该形参在该函数中没有实质用处）。
*	参数2：指定进入睡眠模式的方式。有两个选择，PWR_SLEEPENTRY _WFI 表示使用 WFI指令，PWR_SLEEPENTRY_WFE 表示使用 WFE 指令。
**/
void HAL_PWR_EnterSLEEPMode(uint32_t Regulator, uint8_t SLEEPEntry);
/** 与上方函数参数一样 **/
void HAL_PWR_EnterSTOPMode (uint32_t Regulator, uint8_t STOPEntry);

/** 使能唤醒引脚函数
*	形参 1 取值范围：PWR_WAKEUP_PIN1。
**/
void HAL_PWR_EnableWakeUpPin (uint32_t WakeUpPinPolarity);
void HAL_PWR_EnterSTANDBYMode (void);
```

### 电源监控

|          驱动函数           | 关联寄存器  |    功能描述     |
| :-------------------------: | :---------: | :-------------: |
| __HAL_RCC_PWR_CLK_ENABLE(…) | RCC_APB1ENR |  使能电源时钟   |
|    HAL_PWR_ConfigPVD(…)     |   PWR_CR    | 配置PVD相关参数 |
|    HAL_PWR_EnablePVD(…)     |   PWR_CR    |   使能PVD功能   |
|   __HAL_PWR_GET_FLAG(...)   |   PWR_CR    |   获取标志位    |

```c
typedef struct  { 
	uint32_t			PVDLevel; 		/* PVD检测级别 */ 
	uint32_t			Mode; 			/* PVD的EXTI检测模式 */ 
} PWR_PVDTypeDef;
#define PWR_PVDLEVEL_0                  PWR_CR_PLS_2V2
#define PWR_PVDLEVEL_1                  PWR_CR_PLS_2V3
#define PWR_PVDLEVEL_2                  PWR_CR_PLS_2V4
#define PWR_PVDLEVEL_3                  PWR_CR_PLS_2V5
#define PWR_PVDLEVEL_4                  PWR_CR_PLS_2V6
#define PWR_PVDLEVEL_5                  PWR_CR_PLS_2V7
#define PWR_PVDLEVEL_6                  PWR_CR_PLS_2V8
#define PWR_PVDLEVEL_7                  PWR_CR_PLS_2V9 

#define PWR_PVD_MODE_NORMAL
#define PWR_PVD_MODE_IT_RISING
#define PWR_PVD_MODE_IT_FALLING
#define PWR_PVD_MODE_IT_RISING_FALLING
#define PWR_PVD_MODE_EVENT_RISING
#define PWR_PVD_MODE_EVENT_FALLING
#define PWR_PVD_MODE_EVENT_RISING_FALLING

/**
*	参数：PWR_FLAG_WU唤醒标志位，PWR_FLAG_SB待机标志位，PWR_FLAG_PVDO PVD输出标志位
**/
__HAL_PWR_GET_FLAG()
```

## 案例

使用按键1 2 3选择进入的模式，使用外部中断(GPIOA_PIN0)来唤醒模式,使用PVD监控电源电压

问题：无法进入睡眠模式

```c
#define PWR_WU_GPIO_PORT    GPIOA
#define PWR_WU_GPIO_PIN     GPIO_PIN_0
GPIO_InitTypeDef GPIO_InitStructure;
/**电源管理**/
void PWR_WU_Init(void){
    GPIO_InitStructure.Mode = GPIO_MODE_IT_RISING;
    GPIO_InitStructure.Pin = PWR_WU_GPIO_PIN;
    GPIO_InitStructure.Pull = GPIO_PULLDOWN;
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;
    
    __HAL_RCC_GPIOA_CLK_ENABLE();
    
    HAL_GPIO_Init(PWR_WU_GPIO_PORT,&GPIO_InitStructure);
        
    HAL_NVIC_SetPriorityGrouping(2);
    HAL_NVIC_SetPriority(EXTI0_IRQn,2,2);
    HAL_NVIC_EnableIRQ(EXTI0_IRQn);
}
void EXTI0_IRQHandler(void){
    HAL_GPIO_EXTI_IRQHandler(PWR_WU_GPIO_PIN);
}
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin){
    if(GPIO_Pin  == PWR_WU_GPIO_PIN){
        __HAL_GPIO_EXTI_CLEAR_IT(GPIO_Pin);      // 清除中断标志位，实际上不用写，HAL_GPIO_EXTI_IRQHandler中已经执行过了
    }
}
/**电源监控**/
void PWR_PVD_Init(void){
    __HAL_RCC_PWR_CLK_ENABLE();
    
    PWR_PVDTypeDef PWR_PVDStructure;
    PWR_PVDStructure.PVDLevel = PWR_PVDLEVEL_7;               /* 检测电压级别 */     
    PWR_PVDStructure.Mode = PWR_PVD_MODE_IT_RISING_FALLING;   /* 使用中断线的上升沿和下降沿双边缘触发，高于低于都触发 */
    HAL_PWR_ConfigPVD(&PWR_PVDStructure);
    HAL_PWR_EnablePVD();
    
    HAL_NVIC_SetPriority(PVD_IRQn, 2, 2);
    HAL_NVIC_EnableIRQ(PVD_IRQn); 
}
void PVD_IRQHandler(void){
    HAL_PWR_PVD_IRQHandler();
}
void HAL_PWR_PVDCallback(void){
    if(__HAL_PWR_GET_FLAG(PWR_FLAG_PVDO)){  /* 电压比 PLS 所选电压还低 */
        printf("电压过低\r\n");
    }else{
        printf("电压正常\r\n");
    }
}
int main(void){
    HAL_Init();                                 /* 初始化HAL库 */
    sys_stm32_clock_init(RCC_PLL_MUL9);         /* 设置时钟,72M */
    delay_init(72);                             /* 初始化延时函数 */
    UART_Init();
    key_init();
    PWR_WU_Init();
    PWR_PVD_Init();
    uint8_t key;
    printf("低功耗实验\r\n");
    while(1){
        key = key_scan(0);
        
        if(key){
            switch(key){
                case 1:   // 进入睡眠
                      printf("进入睡眠模式\r\n");
                      HAL_PWR_EnterSLEEPMode(PWR_MAINREGULATOR_ON,PWR_SLEEPENTRY_WFI);
                      printf("退出睡眠模式\r\n");
                break;
                case 2:  // 进入停机转态
                      printf("进入停机模式\r\n");
                      HAL_PWR_EnterSTOPMode(PWR_LOWPOWERREGULATOR_ON,PWR_SLEEPENTRY_WFI);
                      sys_stm32_clock_init(RCC_PLL_MUL9);           /* 需要重新初始化时钟 */
                      delay_init(72); 
                      printf("退出停机模式\r\n");
                break;
                case 3:  //进入待机模式
                        printf("进入待机模式\r\n");
                        __HAL_RCC_PWR_CLK_ENABLE();                 /* 使能电源时钟 */
                        HAL_PWR_EnableWakeUpPin(PWR_WAKEUP_PIN1);   /* 使能 KEY_UP 引脚的唤醒功能 */
                        __HAL_PWR_CLEAR_FLAG(PWR_FLAG_WU);          /* 需要清此标记，否则将保持唤醒状态 */
                        HAL_PWR_EnterSTANDBYMode();                 /* 进入待机模式 */
                        printf("退出待机模式\r\n");                   // 这一句永远没法触发，因为待机模式会从启MCU
                break;
                default:break;
            }
        }
    }
}
```



# 其他

## SysTick

## FPU

FPU：Float Point Unit，浮点运算单元

![image-20230816160511880](STM32卷二.assets/image-20230816160511880.png)

|      对比项       |   F1   |   F4   |  F7  |  H7  |
| :---------------: | :----: | :----: | :--: | :--: |
|      有无FPU      |   无   |   有   |  有  |  有  |
| 是否支持单精度FPU | 不支持 |  支持  | 支持 | 支持 |
| 是否支持双精度FPU | 不支持 | 不支持 | 支持 | 支持 |

## DSP

提高数字运算速度

DSP：Digital Signal Processing，数字信号处理

M3内核没有硬件FPU，也没有DSP指令集

M4和M7等内核自带FPU和DSP单元，并集成了专用的DSP指令集（如：单周期乘加指令(MAC)、优化的单指令多数据流指令(SIMD)、饱和算术运算等多种数字信号处理指令）

M4/M7执行的DSP指令可以在单周期内完成，M3则需要多个指令和多个周期才能完成







## IAP

**STM32三种烧录方式**

- ISP In System Programming（在系统编程）：执行芯片厂商的 Bootloader 程序进入 ISP 模式，进入ISP 模式后，用户可选择官方提供的烧录通信接口（如：串口），并配合ISP 编程工具（如：FlyMcu）对闪存进行烧录。
- ICP In Circuit Programing（在线编程）：使用IDE并通过JTAG/SWD接口对闪存进行烧录。常用
- IAPIn Application Programming（在应用编程）：使用用户的应用程序（也称为：Bootloader程序）对闪存进行烧录。该应用程序需要通过一种通信接口（如：IO口\USB\CAN\UART\I2C\SPI等）对闪存进行烧录（即把APP程序烧录到闪存）。IAP 通常被开发者用作远程升级的手段。

前两种生成.hex文件，IAP生成的是.bin文件

![image-20230816163750432](STM32卷二.assets/image-20230816163750432.png)





## USB

USB， 英文全称：Universal Serial Bus，即通用串行总线

USB提供适合各种应用的传输协议，而且协议标准向下兼容

| **标准**        | **推出时间** | **最大传输速度** | **电缆长度** | **速度模式** |
| --------------- | ------------ | ---------------- | ------------ | ------------ |
| USB 1.0         | 1996         | 1.5Mbps          | 3m           | 低速模式     |
| USB 1.1         | 1998         | 12Mbps           | 3m           | 全速模式     |
| USB 2.0         | 2000         | 480Mbps          | 5m           | 高速模式     |
| USB 3.2 Gen 1   | 2008         | 5Gbps            | 3m           | 超高速模式   |
| USB 3.2 Gen 2   | 2013         | 10Gbps           | 3m           | 超高速模式   |
| USB 3.2 Gen 2x2 | 2017         | 20Gbps           | 3m           | 超高速模式   |
| USB 4           | 2019         | 40Gbps+          | 0.8m         | 超高速模式   |

USB是一种主从结构的系统，数据交换只能发生在主从设备之间，且只能由主机主动发起

USB主机具有一个或者多个USB主控制器(Host controller)和根集线器(Root Hub)

USB主控制器：主要负责数据处理

根集线器：提供一个连接主控制器与设备之间的接口和通路

USB从机可以是各种USB设备也可以是USB集线器

USB集线器(USB Hub)：用于对原有的USB接口数量进行扩展，但不能扩展更多的带宽

USB OTG作为USB协议的补充版本，允许同一设备，在不同场合下，在主机和从机之间切换



**STM32的USB**

![image-20230816172749841](STM32卷二.assets/image-20230816172749841.png)



**STM32 USB框图**

![image-20230816172827460](STM32卷二.assets/image-20230816172827460.png)

48MHz的USB时钟由锁相环经过1.5分频得到



# 其他硬件



## 电源

| 名称 | 解释                 | 名称  | 解释                 |
| ---- | -------------------- | ----- | -------------------- |
| VCC  | 电路的供电正电压     | VDDD  | 芯片的工作数字正电压 |
| GND  | 电路的供电负电压     | VSSD  | 芯片的工作数字正电压 |
| VDD  | 芯片的工作正电压     | VREF+ | ADC基准参考正电压    |
| VSS  | 芯片的工作负电压     | VREF- | ADC基准参考负电压    |
| VDDA | 芯片的工作模拟正电压 | VBAT  | 电池或其他电源供电   |
| VSSA | 芯片的工作模拟负电压 | VEE   | 负电压供电           |

VEE：负电压供电；场效应管的源极（S）

VPP：编程/擦除电压。 

V*与V*A的区别是：数字与模拟的区别

数字电路供电VCC 
模拟电路供电VCCA
CC与DD的区别是：供电电压与工作电压的区别（通常VCC>VDD）
VDD是指工作电压，就是供电进芯片的 
VDDA是模拟电压或者叫模拟正电源，是从芯片向外供电的

1、对于数字电路来说，VCC是电路的供电电压,VDD是芯片的工作电压（通常Vcc>Vdd），VSS是接地点。 
2、有些IC既有VDD引脚又有VCC引脚，说明这种器件自身带有电压转换功能。 
3、在场效应管（或COMS器件）中，VDD为漏极，VSS为源极，VDD和VSS指的是元件引脚，而不表示供电电压。 

4、一般来说VCC=模拟电源,VDD=数字电源,VSS=数字地,VEE=负电源
5、从电气意义上说，GND分为电源地和信号地。PG是 Power Ground（电源地）的缩写。另一个是 Signal Ground（信号地）。实际上它们可能是连在一起的（不一定是混在一起哦！）。两个名称主要是便于对电路进行分析。 
 进一步说，还有因电路形式不同而必须区分的两种“地”：数字地，模拟地。 
数字地和模拟地都有信号地、电源地两种情况。数字地和模拟地之间，某些电路可以直接连接，有些电路要用电抗器连接，有些电路不可连接。

## 舵机

* 舵机是一种根据输入PWM信号占空比来控制输出角度的装置
* 输入PWM信号要求：周期为20ms，高电平宽度为0.5ms~2.5ms

**180度舵机**

![image-20230706115242630](STM32卷二.assets/image-20230706115242630.png)



**360度舵机**

360度舵机可以360度旋转，因此与180度舵机有相当大的区别，首先360度舵机不能够控制旋转角度，一般的舵机是给一个特定的PWM，舵机会转到相应的角度，而360度舵机是只能够控制方向和旋转转速，所以360度舵机给定一个PWM，会以特定的速度和方向转动。
所以有：
0.5ms----------------正向最大转速；
1.5ms----------------速度为0；
2.5ms----------------反向最大转速；



![image-20230706115256241](STM32卷二.assets/image-20230706115256241.png)

## 直流电机及驱动简介

* 直流电机是一种将电能转换为机械能的装置，有两个电极，当电极正接时，电机正转，当电极反接时，电机反转
* 直流电机属于大功率器件，GPIO口无法直接驱动，需要配合电机驱动电路来操作
* TB6612是一款双路H桥型的直流电机驱动芯片，可以驱动两个直流电机并且控制其转速和方向

![1688615635618](STM32卷二.assets/1688615635618.png)



只有当IN1和IN2输出不一样的电平时电机才可能转，到底转不转还取决于PWM信号





## EEPROM

### AT24C02

EEPROM是一种掉电后数据不丢失的储存器，常用来存储一些配置信息，在系统重新上电时就可以加载。注意:EEPROM比较慢，必须等到10ms后再写下一个字节

AT24C02是一个2K bit的EEPROM存储器，使用IIC通信方式。

**AT24Cxx  系列**

| AT24Cxx  | 容量（bit）        | 页数 | 页内字节数 | 数据地址(占用bit数) |
| -------- | ------------------ | ---- | ---------- | ------------------- |
| AT24C01  | 1K bit (128 B)     | 16   | 8 Byte     | 7bit                |
| AT24C02  | 2K bit (256 B)     | 32   | 8 Byte     | 8bit                |
| AT24C04  | 4K bit (512 B)     | 32   | 16 Byte    | 9bit                |
| AT24C08  | 8K bit (1024 B)    | 64   | 16 Byte    | 10bit               |
| AT24C16  | 16K bit (2048 B)   | 128  | 16 Byte    | 11bit               |
| AT24C32  | 32K bit (4096 B)   | 128  | 32 Byte    | 12bit               |
| AT24C64  | 64K bit (8192 B)   | 256  | 32 Byte    | 13bit               |
| AT24C128 | 128K bit (16384 B) | 256  | 64 Byte    | 14bit               |
| AT24C256 | 256K bit (32768 B) | 512  | 64 Byte    | 15bit               |
| AT24C512 | 512K bit (65535 B) | 512  | 128 Byte   | 16bit               |

**AT24C02通讯地址**

写操作地址：0xA0  读操作地址：0xA1

**AT24C02读写时序**

写操作

* AT24C02支持字节写模式和页写模式。
* 字节写模式就是一个地址一个数据进行写入。
* 页写模式就是连续写入数据。只需要写一个地址，连续写入数据时地址会自增，但存在页的限制，超出一页时，超出数据覆盖原先写入的数据。但读会自动翻页。

读操作

* AT24C02支持当前地址读模式，随机地址读模式和顺序读模式。
* 当前读模式是基于上一次读/写操作的最后位置继续读出数据。
* 随机地址读模式是指定地址读出数据。
* 顺序读模式是连续读出数据。

## Flash

Flash就是ROM

Flash程序存储的地方，RAM 程序运行的地方



flash ，它在嵌入式系统中的功能可以和硬盘在PC中的功能相比。它们都是用来存储程序和数据的，好比是ROM。而且可以在掉电的情况下继续保存数据使其不会丢失。Flash memory（闪速存储器）作为一种安全、快速的存储体，具有体积小，容量大，成本低，掉电数据不丢失等一系列优点，已成为嵌入式系统中数据和程序最主要的载体。根据结构的不同可以将其分为**NOR Flash**和**NAND Flash**两种。NOR Flash的特点是应用程序可以直接在闪存中运行，不需要再把代码读到系统RAM中运行。NAND Flash不行。而我们单片机基本都是NOR FLASN。而手机我们说的64 128应该是NAND FASH。

毫无疑问，stm32的flash是NOR Flash。





**Flash操作注意事项**

* 写入操作时：
  * 写入操作前，必须先进行写使能
  * 每个数据位只能由1改写为0，不能由0改写为1   （是不是不可思议，再结合下面擦除操作就能理解了，先擦除，令数据位为1，在进行修改）
  * 写入数据前必须先擦除，擦除后，所有数据位变为1
  * 擦除必须按最小擦除单元进行    （不能单独擦除某个字节，要擦除就擦除一偏区域）
  * 连续写入多字节时，最多写入一页的数据，超过页尾位置的数据，会回到页首覆盖写入
  * 写入操作结束后，芯片进入忙状态，不响应新的读写操作 (将Buff中的数据放入Flash中)

* 读取操作时：
  * 直接调用读取时序，无需使能，无需额外操作，没有页的限制，读取操作结束后不会进入忙状态，但不能在忙状态时读取

| 类型       | 特点                                                        | 应用举例         |
| ---------- | ----------------------------------------------------------- | ---------------- |
| NOR  FLASH | 基于字节读写，读取速度快，独立地址/数据线，无坏块，支持XIP  | 25Qxx、程序ROM   |
| NAND FLASH | 基于块读写，读取速度稍慢，地址数据线共用，有坏块，不支持XIP | EMMC、SSD、U盘等 |

### W25Qxx

* W25Qxx系列是一种低成本、小型化、使用简单的非易失性存储器，常应用于数据存储、字库存储、固件程序存储等场景
* 存储介质：Nor Flash（闪存）这是FLASH，不是RAM
* 时钟频率：80MHz / 160MHz (Dual SPI) / 320MHz (Quad SPI)
* 存储容量（24位地址）：
  * W25Q40：  4Mbit / 512KByte
  * W25Q80：  8Mbit / 1MByte
  * W25Q16：  16Mbit / 2MByte
  * W25Q32：  32Mbit / 4MByte
  * W25Q64：  64Mbit / 8MByte
  * W25Q128： 128Mbit / 16MByte
  * W25Q256： 256Mbit / 32MByte

![image-20230708135005415](STM32卷二.assets/image-20230708135005415.png)



被分为Block块、Sector扇区、Page页

结尾为0000的为起始，结尾为FF的为结尾，比如000000h为Block0的起始，00FFFFh为Block0的起始

**不可以跨页写入**（每页256个Byte），比如将多个数据写入到FF结尾的地址中，他只会将第一个数据写到结尾，其他数据写到`本页`的开头，而不是写到下一页的开头

扇区擦除后：值为0xff，即全部填充1





## 触摸屏

触摸屏按工作原理和传输介质可分为：红外线式、表面声波式、电阻式和电容式

![image-20230816132528714](STM32卷二.assets/image-20230816132528714.png)





**电阻式**

分类：四线，五线，七线和八线

电阻式触摸屏驱动IC：XPT2046、TSC2046、HR2046

电阻式传出的是AD信号，需要经过换算才能得到坐标值，而且需要经过校准，因为触摸板和屏幕可能并不重合



**电容式**

分类：

* 表面电容式（利用电场感应感测屏幕触摸，只能识别一次触摸）
* 投射式（利用触摸屏电极发射出静电场线）
  * 自我电容（扫描电极与地构成的电容）
  * 交互电容（玻璃表面的横向和纵向的ITO电极的交叉处形成的电容）

电容式触摸屏驱动IC：GT9147、GT917S、GT911、GT1151、FT5426

电容式传出的是坐标值，内部有寄存器

直接使用正点原子提供的库即可，电阻式使用SPI通信，电容式使用IIC通信



## 红外

红外遥控的情景中，必定会有一个红外发射端和红外接收端。要使两者通信成功，收/发红外波长与载波频率需一致，在这里波长就是 940nm，载波频率就是 38kHz。红外发射管也是属于二极管类，红外发射电路通常使用三极管控制红外发射器的导通或者截至，在导通的时候，红外发射管会发射出红外光，反之，就不会发射出红外光。虽然我们用肉眼看不到红外光，但是我们借助手机摄像头就能看到红外光。但是红外接收管的特性是当接收到红外载波信号时，OUT 引脚输出低电平；假如没有接收到红外载波信号时，OUT 引脚输出高电平。

红外载波信号其实就是由一个个红外载波周期组成。在频率为 38KHz 下，红外载波周期约等于 26.3us（1s / 38KHz ≈ 26.3us）。在一个红外载波发射周期里，发射红外光时间 8.77us 和不发射红外光 17.53us，发射红外光的占空比一般为 1/3。相对的，整个周期内不发射红外光，就是载波不发射周期。在红外遥控器内已经把载波和不载波信号处理好，我们需要做的就是识别遥控器按键发射出的信号，信号也是遵循某种协议的。

* 载波周期：1s / 38KHz ≈ 26.3us
* 载波发射周期：26.3us(一个周期) = 8.77us(发射红外光) + 17.53us(不发射红外光)
* 载波不发射周期：整个周期内，不发射红外光

### **红外编解码协议**

红外遥控的编码方式目前广泛使用的是：PWM（脉冲宽度调制）的 NEC 协议和 Philips PPM（脉冲位置调制）的 RC-5 协议的。



下面介绍PWM（脉冲宽度调制）的 NEC 协议



**红外发射器：**

* 发送协议数据‘0’= 发射载波信号560us + 不发射载波560us
* 发送协议数据‘1’= 发射载波信号560us + 不发射载波1680us

![image-20230816141419094](STM32卷二.assets/image-20230816141419094.png)

**红外接收器：**OUT引脚电平输出情况（接收到红外载波时，OUT输出低电平，否则输出高电平）

* 接收到协议数据‘0’= 560us低电平 + 560us高电平
* 接收到协议数据‘1’= 560us低电平 + 1680us高电平

![image-20230816141533655](STM32卷二.assets/image-20230816141533655.png)



**NEC遥控器指令格式**

* 同步码（引导码），低电平9ms + 高电平4.5ms（对于接收端）
* 地址码
* 地址反码
* 控制码
* 控制反码

注意：① 地址码、地址反码、控制码、控制反码均是8位数据格式

   ② 按照低位在前，高位在后的顺序发送（低位先行）

   ③ 采用反码是为了增加传输的可靠性（可用于校验）