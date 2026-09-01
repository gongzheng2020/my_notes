
---

## 外设

### DMA

![[Pasted image 20260715103356.png]]

#### 概念

1. DMA用来提供在**外设和存储器之间或者存储器和存储器之间**的高速数据传输。无须CPU的干预，通过DMA数据可以快速地移动。这就节省了CPU的资源来做其他操作。

2. DMA主要涉及**源地址、目标地址、传输数据量**这三个参数，除了正常模式外（单次传输），**DMA 还有循环传输模式** （当到达传输终点时会重新启动DMA传输）。

#### 流程

![[Pasted image 20260714170842.png]]

1. **配置 DMA 通道**：
    - 设定外设地址（如 UART 的数据寄存器 DR、ADC 的数据寄存器 DR）；
    - 设定存储器地址（如 RAM 中的数组）；
    - 设定传输数据量（字节数、半字数、字数）；
    - 设定传输方向、数据宽度、是否循环模式、优先级等。
2. **使能外设 DMA 请求**：外设需开启 DMA 模式（如 UART 的 DMAT 位、ADC 的 DMA 位）。
3. **启动 DMA 传输**：使能 DMA 通道，当外设产生请求（如 UART 发送缓冲区空、ADC 转换完成）时，DMA 自动开始传输。
4. **传输完成处理**：通过中断或查询标志位，确认传输完成后进行后续操作（如处理数据、关闭 DMA 等）

#### 参考文献
1. [DMA原理，步骤超详解释，一文看懂DMA](https://blog.csdn.net/as480133937/article/details/104927922)
2. [完整教程：STM32——DMA](https://www.cnblogs.com/tlnshuju/p/19236184)
3. [DMA直接存储器---基础概念](https://blog.csdn.net/qq_53878049/article/details/134229692?spm=1001.2014.3001.5502)

### SPI

![[Pasted image 20260715103050.png]]

#### 基本流程

![[Pasted image 20260714150118.png]]

如上图所示，STM32使用SPI外设通讯时，在通讯的不同阶段它会对**“状态寄存器SR”**的不同数据位写入参数，我们通过读取这些寄存器标志来了解通讯状态。

主模式收发流程及事件说明如下：

1. 控制NSS信号线， 产生起始信号(图中没有画出)；  
2.  把**要发送的数据写入到“数据寄存器DR”中**， 该数据会被存储到发送缓冲区；  
3.  通讯开始，SCK时钟开始运行。MOSI与MISO分别互发数据；  
4.  当发送完一帧数据的时候，“状态寄存器SR”中的**“TXE标志位”会被置1，表示传输完一帧，发送缓冲区已空**；类似地， **当接收完一帧数据的时候，“RXNE标志位”会被置1**，表示传输完一帧，接收缓冲区非空；  
5.  等待到“TXE标志位”为1时，若还要继续发送数据，则再次往“数据寄存器DR”写入数据即可；等待到“RXNE标志位”为1时， 通过读取“数据寄存器DR”可以获取接收缓冲区中的内容。  

轮询的方式直接调用函数即可：
``` c
//SPI2 读写一个字节
//TxData:要写入的字节
//返回值:读取到的字节
u8 SPI3_ReadWriteByte(u8 TxData)
{
    u8 Rxdata;
    HAL_SPI_TransmitReceive(&hspi1, &TxData, &Rxdata, 1, 1000);
    return Rxdata;                      //返回收到的数据
}
HAL_SPI_Receive(&hspi1, &rx_data, 1, HAL_MAX_DELAY);
HAL_SPI_Transmit(&hspi1, &data, 1, HAL_MAX_DELAY);
```

#### 中断方式收发数据

假如我们使能了TXE或RXNE中断，**TXE或RXNE置1时会产生SPI中断信号，进入同一个中断服务函数**，到SPI中断服务程序后， **可通过检查寄存器位来了解是哪一个事件，再分别进行处理**

1. HAL库中可以直接使用`HAL_SPI_TransmitReceive_IT`这个函数：

``` c
// 调用中断收发函数（触发传输）
HAL_SPI_TransmitReceive_IT(&hspi1, tx_buffer, rx_buffer, 3);
// 数据收发完成后，HAL会自动回调该函数，通知应用程序
void HAL_SPI_RxCpltCallback(SPI_HandleTypeDef *hspi){
    if(hspi->Instance == SPI1) {
        // 数据已经全部存在 rx_buffer 中了，在这里处理数据
    }
}
```

2. 手动写的话，需要在 SPI_CR2 寄存器中，操作 TXEIE和 RXNEIE，需要注意的是：
	- TXEIE只能在发送数据时打开，否则会一直触发中断，卡死CPU
	- RXNE标志位在读取数据后自动清除，若不清除则会导致总线卡死
``` c
void SPI1_IRQHandler(void)
{
    // 1. 处理接收中断（RXNE = 1，代表收到一个字节）
    if(SPI_I2S_GetITStatus(SPI1, SPI_I2S_IT_RXNE) != RESET) {
        // 从数据寄存器读走数据，并存入接收缓冲区
        rx_buffer[rx_idx++] = SPI_I2S_ReceiveData(SPI1);
        // ！！！务必注意：即使你只需要发送，不接收，这里也必须读走数据（扔到临时变量里）
        // 否则 SPI 硬件会认为接收缓冲区满了，产生溢出错误（OVR），导致总线卡死。
    }
    
    // 2. 处理发送中断（TXE = 1，代表发送寄存器空了，可以发下一个字节）
    if(SPI_I2S_GetITStatus(SPI1, SPI_I2S_IT_TXE) != RESET) {
        // 如果还有数据未发送完，继续发
        if(tx_idx < transmit_len) {
            SPI_I2S_SendData(SPI1, tx_buffer[tx_idx++]);
        } 
        // 如果数据全部发完了
        else {
            // 关闭 TXE 中断，否则 SPI 会一直触发中断空转，占用 CPU！
            SPI_I2S_DisableIT(SPI1, SPI_I2S_IT_TXE);
        }
    }
}
```

#### 使用DMA收发

- 发送时，在每次TXE被设置为’1’时发出DMA请求，DMA控制器则写数据至SPI_DR寄存器，TXE标志因此而被清除。
- 接收时，在每次RXNE被设置为’1’时发出DMA请求，DMA控制器则从SPI_DR寄存器读出数据，RXNE标志因此而被清除。

轮询（检查counter）：
``` c
 while (1)
  {
    /* USER CODE END WHILE */
	HAL_GPIO_WritePin(GPIOA,GPIO_PIN_15,GPIO_PIN_RESET);
    HAL_SPI_TransmitReceive_DMA(&hspi1,txbuff,rxbuff,8);
    while(__HAL_DMA_GET_COUNTER(&hdma_spi1_rx)!=0);
    HAL_GPIO_WritePin(GPIOA,GPIO_PIN_15,GPIO_PIN_SET);
    /* USER CODE BEGIN 3 */
  }
```

中断方式：
``` c
// 1. 调用 API 开启 DMA 收发
// 参数：SPI句柄，发送数组，接收数组，数据长度
HAL_SPI_TransmitReceive_DMA(&hspi1, tx_buffer, rx_buffer, BUFFER_SIZE);

// 2. 数据全部传输完成后，HAL 库会自动调用的回调函数
void HAL_SPI_TxRxCpltCallback(SPI_HandleTypeDef *hspi)
{
    if(hspi->Instance == SPI1) {
        // 此时 rx_buffer 里已经填满了从从机读回来的数据
        // 可以在这里处理数据，或者发送信号量给RTOS任务
    }
}
```

> 在SPI中使用DMA时，可以将NSS引脚的拉高拉低操作在中断中进行处理，以提高实时性
> NSS若由硬件进行控制，因为它每发一个字节就会自动拉高，对大于一个字节的数据就有问题

#### 参考文献

1. [STM32SPI外设简介](https://blog.csdn.net/qq_53878049/article/details/134632256?spm=1001.2014.3001.5502)
2. [STM32F7实现SPI读写，读取W25Q16型号](https://blog.csdn.net/qq_27508477/article/details/105702541)
3. [STM32F103 SPI详解及示例代码](https://www.cnblogs.com/zhaoweiwei/p/18310882/SPI)
4. [笔记之STM32F0芯片SPI_DMA的使用（HAL库）](https://blog.csdn.net/chenyuanlidejiyi/article/details/121639160)
5. [STM32的SPI口的DMA读写](https://www.cnblogs.com/helesheng/p/16757245.html)

### I2C

![[Pasted image 20260715103034.png]]

#### 基本流程

- SDA部分：数据收发的核心部分是数据寄存器和数据移位寄存器，
当需要发送数据时，把一个字节数据写到数据寄存器DR，当移位寄存器没有数据移位时，数据寄存器的值就会进一步转到移位寄存器，在移位过程中，就可以把下一个数据放到数据寄存器里等待，当前一个数据移位完成，下一个数据无缝连接，继续发送；**当数据由数据寄存器转到移位寄存器时，会置状态寄存器的TXE为1，表示发送寄存器为空**；接收时，输入的数据一位一位的从引脚移入到移位寄存器里，**当一个字节的数据收齐之后，数据就整体从移位寄存器转到数据寄存器，同时置标志位RXNE，表示接收寄存器非空**，就可以从数据寄存器读出数据

- SCL部分：时钟控制SCL线，**在时钟控制寄存器CCR写对应的位**，电路会执行对应的功能。

1. 主机发送

![[Pasted image 20260715104042.png]]

2. 主机接收

![[Pasted image 20260715104054.png]]

一些相关函数：
``` c
/* 第1个参数为I2C操作句柄
   第2个参数为从机设备地址
   第3个参数为从机寄存器地址
   第4个参数为从机寄存器地址长度
   第5个参数为发送的数据的起始地址
   第6个参数为传输数据的大小
   第7个参数为操作超时时间 　　*/
HAL_I2C_Mem_Write(&hi2c2,salve_add,0,0,PA_BUFF,sizeof(PA_BUFF),0x10);
HAL_I2C_Mem_Write_IT()；
HAL_I2C_Mem_Read();
HAL_I2C_Mem_Read_IT();
HAL_I2C_Mem_Read_DMA();
HAL_I2C_Mem_Write_DMA();

/* 不需要用到寄存器地址的主机HAL库IIC收发函数　　 */
HAL_I2C_Master_Receive(&hi2c2,salve_add,PA_BUFF,sizeof(PA_BUFF),0x10)； //STM32 主机接收，不需要用到寄存器地址
HAL_I2C_Master_Transmit();
HAL_I2C_Master_Receive_IT()；　　　//中断IIC接收 
HAL_I2C_Master_Transmit_IT();　　 //中断IIC发送
HAL_I2C_Master_Receive_DMA()；　　//DMA 方式的IIC接收   
HAL_I2C_Master_Transmit_DMA(); 　 //DMA 方式的IIC发送  

/* 不需要用到寄存器地址的从机HAL库IIC收发函数 　　*/ 
HAL_I2C_Slave_Receive();　　　　//STM32 从机机接收，不需要用到寄存器地址  HAL_I2C_Slave_Transmit();　　 //STM32 从机机发送，不需要用到寄存器地址  HAL_I2C_Slave_Receive_IT(); 
HAL_I2C_Slave_Receive_DMA(); 
HAL_I2C_Slave_Transmit_IT();
HAL_I2C_Slave_Transmit_DMA();
```

#### 参考文献
1. [I2C通信外设简介](https://blog.csdn.net/qq_53878049/article/details/134319507?spm=1001.2014.3001.5502)

### UART

![[Pasted image 20260715143942.png]]

![[Pasted image 20260715151126.png]]

#### 轮询模式

1. **底层核心寄存器**：
    
    - **`USART_SR` (状态寄存器)**：
        
        - `TXE` (Bit 7)：发送数据寄存器为空（发送完一个字节到移位寄存器）。
            
        - `TC` (Bit 6)：发送完成（移位寄存器里的数据也发送完毕了）。
            
        - `RXNE` (Bit 5)：接收数据寄存器非空（收到一个字节）。
  
    - **`USART_DR` (数据寄存器)**：存放实际发送或接收的 8/9 位数据。
        
2. **工作流程**：
    
    - **发送**：`while` 死循环检测 `TXE` 标志位（等待为空）。空后把数据写入 `DR` 寄存器。**强烈建议在发完所有数据后，额外等待 `TC` 标志位**，否则在关闭串口或进入低功耗时，最后一个字节可能会丢失。
        
    - **接收**：`while` 死循环检测 `RXNE` 标志位（等待非空）。一旦置位，立即从 `DR` 寄存器读取数据。
        
3. **对应的 HAL 库函数**：
	``` c
	// 发送
	HAL_UART_Transmit(&huart1, tx_buffer, data_len, timeout_ms);
	// 接收
	HAL_UART_Receive(&huart1, rx_buffer, data_len, timeout_ms);
	```

#### 中断方式

1. **底层核心寄存器**：
    
    - **`USART_CR1` (控制寄存器 1)**：
        
        - `TXEIE` (Bit 7)：发送寄存器空中断使能。
            
        - `TCIE` (Bit 6)：发送完成中断使能。
            
        - `RXNEIE` (Bit 5)：接收寄存器非空中断使能。
            
    - _（HAL 库会在调用 `_IT` 函数时，自动操作这些使能位）_
        
2. **工作流程与中断链路**：
    
    1. **主函数触发**：调用发送/接收的中断 API。
        
    2. **硬件触发**：每完成一个字节，硬件自动将 `TXE` 或 `RXNE` 置位，并向 CPU 发出中断信号。
        
    3. **执行 ISR**：CPU 跳转到 `USARTx_IRQHandler()`。
        
    4. **HAL 底层分发**：在 ISR 中，必须调用 **`HAL_UART_IRQHandler(&huart1)`**。HAL 底层会自动判断是发送中断还是接收中断，并帮你操作 `DR` 寄存器搬运数据、维护内部缓冲区的索引。
        
    5. **回调通知**：当传输完你指定的所有字节后，HAL 底层**会自动关闭 TXEIE/RXNEIE**（防止中断风暴），并调用**用户重写的回调函数**通知应用层处理。

3. 一个例子
``` c
int main(void)
{
	...
	
	// 【重要】第一次调用，开启中断接收 5 个字节
	HAL_UART_Receive_IT(&huart1, rx_data, RX_BUFFER_SIZE);
	while (1)
	{
		// 主循环完全解放，不用担心这里的代码被阻塞
		// 例如可以在这里翻转LED、处理其他传感器数据
	}
}
/* ========== 2. 中断处理逻辑（HAL 底层自动调用） ========== */
/* 
 * 注意：本函数由HAL库底层在 USART1_IRQHandler 中自动调用。
 * 你根本不需要去管“标志位怎么清”，HAL库会帮你处理得干干净净。
 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if (huart->Instance == USART1) // 判断是哪个串口触发的
	{
		// --- 步骤 A：处理接收到的数据 ---
		for (int i = 0; i < RX_BUFFER_SIZE; i++) {
			tx_data[i] = rx_data[i] + 1; // 接收到的数据加1，存入发送缓冲区
		}
		// --- 步骤 B：将处理好的数据通过中断发送给上位机 ---
		HAL_UART_Transmit_IT(huart, tx_data, RX_BUFFER_SIZE);
		// --- 步骤 C：⚠️ 极其重要的动作：重启接收中断！ ⚠️ ---
		/* 
		 * HAL库的 Receive_IT 是一次性的！
		 * 接收完指定的 5 个字节后，底层会自动关闭 RXNEIE 接收中断使能。
		 * 必须再次调用它，STM32 才能继续接收下一次下发的数据！
		 */
		HAL_UART_Receive_IT(huart, rx_data, RX_BUFFER_SIZE);
	}
}
/* ========== 3. 发送完成回调（可选） ========== */
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
{
	if (huart->Instance == USART1) {
		// 这 5 个字节已经安全地发送出去了
		// 这里可以置一个标志位，通知主循环“发送已完成”
	}
}
```

#### DMA方式

1. 发送非常简单，一旦调用，CPU 立刻解放。
``` c
uint8_t tx_buffer[] = "Hello DMA UART!";
// 1. 发起 DMA 发送（数据完全异步，立刻返回）
HAL_UART_Transmit_DMA(&huart1, tx_buffer, sizeof(tx_buffer));
// 2. 传输完成后的回调（由 HAL 底层在 DMA ISR 中调用）
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
{
    if(huart->Instance == USART1) {
        // 发送结束，可以在这里释放信号量，或准备下一包数据
    }
}
```

2. 如果你想一次性收满 10 个字节，代码逻辑如下：
``` c
#define RX_LEN 10
uint8_t rx_buffer[RX_LEN];
int main(void)
{
    // 1. 开启 DMA 接收（异步）
    HAL_UART_Receive_DMA(&huart1, rx_buffer, RX_LEN);
    
    while(1) {
        // CPU 可以在这里安心做其他事情
    }
}
// 2. 接收完成回调
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if(huart->Instance == USART1) {
        // 此时 rx_buffer 已填满 10 个字节
        // 处理数据...
        
        // ⚠️ 必须再次调用，才能接收下一批 10 个字节！
        HAL_UART_Receive_DMA(&huart1, rx_buffer, RX_LEN); 
    }
}
```

3. 处理不定长数据使用IDLE中断

绝大部分实际工程中，串口发来的数据包是不定长的（例如 GPS 数据、MODBUS 报文等）。此时可以使用IDLE中断

1. **配置开启**：
    
    - 把 DMA 接收配置为 **循环模式（DMA_CIRCULAR）**。
        
    - 在初始化时开启串口的**空闲中断**：`__HAL_UART_ENABLE_IT(&huart1, UART_IT_IDLE);`

2. **代码实现：接收与解析**：
``` c
#define RX_BUF_SIZE 128
uint8_t rx_buffer[RX_BUF_SIZE]; // DMA 循环写入这个数组
// 1. 主函数启动 DMA
HAL_UART_Receive_DMA(&huart1, rx_buffer, RX_BUF_SIZE);
// 2. 在串口中断服务函数中处理 IDLE 中断
void USART1_IRQHandler(void)
{
    // 必须调用 HAL 库底层，处理标准的 RXNE/TXE 等中断
    HAL_UART_IRQHandler(&huart1); 
    
    // 额外处理空闲中断
    if(__HAL_UART_GET_FLAG(&huart1, UART_FLAG_IDLE) != RESET) 
    {
        __HAL_UART_CLEAR_IDLEFLAG(&huart1); // 清除空闲标志位
        
        // 关键：计算当前接收了多少个字节！
        // 总容量 - DMA 计数器当前剩余值 = 已接收字节数
        uint32_t rx_len = RX_BUF_SIZE - __HAL_DMA_GET_COUNTER(&hdma_usart1_rx);
        
        // 把取出来的数据拷贝到用户空间，或者进行协议解析
        // 注：由于是循环模式，处理完这里后，DMA 会自动接着从后面覆盖写入
        // 实际工程中，通常在这里停止 DMA（HAL_UART_DMAStop），处理完再重启。
    }
}
```

#### 参考文献
1. [STM32的USART学习笔记](https://blog.csdn.net/weixin_42108533/article/details/128795079)
2. [USART—串口通讯](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/USART.html)

### 定时器

模式：定时中断、PWM、输入捕获、输出比较

### ADC

模式：单次、连续、扫描（ADC 会自动把 IN0 -> IN1 -> IN2 全部转完）、注入（“高优先级抢占”，它会打断常规转换，先转注入组，转完再恢复常规转换）

触发源：软件、外部、定时器

>刚上电时，必须调用一次 ADC 校准（HAL 库中为 HAL_ADCEx_Calibration_Start）。

---

## 存储

![[Pasted image 20260716135843.png]]

### Flash

Flash 是 MCU 的 “程序硬盘”，用于永久存储程序代码和常量数据。地址范围：0x0800 0000 ~ 0x0807 FFFF（示例中为 512KB 容量的 Flash）。

区域划分：

- 用户代码区：存储我们编写的程序代码，包含多个段：

- .text段：存放程序的指令代码（函数、指令序列等）。

- .rodata段：存放只读常量（如字符串常量、const 修饰的变量）。

- .rwdata段：存放需要初始化的可读写数据（运行时会被复制到 SRAM 的.data 段）。

![[Pasted image 20260716140915.png]]

### SRAM

![[Pasted image 20260716144542.png]]

SRAM 是程序运行时的 “临时内存”，其布局可以拆解为以下几个核心部分：

1. 数据段区域（静态存储区）

	- **.data 段**：存放 “初始化了特定值的可修改变量”（如`int counter_3 = 1; static int counter_4 = 2;`）。这些数据在程序启动时会从 Flash 的.rwdata 段复制到 SRAM 的.data 段。
	
	- **.bss 段**：存放 “未初始化或初始化为 0 的可修改变量”（如`int counter = 0; int counter_1; static int counter_2;`）。程序启动时，系统会自动将这块区域清零。

2. 堆（Heap）区域

	- 功能：用于动态内存分配（如 C 语言中的malloc、C++ 中的new）。
	
	- 分布：普通堆（HEAP_SIZE 0X800，即 2KB）

3. 栈（Stack）区域

	- 功能：用于存储函数调用的上下文（局部变量、函数参数、返回地址等），以及中断发生时的现场保护。

	- 生长方向：**地址减小的方向**（从栈顶向栈底生长）。

	- 系统栈（MSP）：用于中断处理、内核级操作。

>堆和栈的大小在编译链接阶段就被“规划”好了，在启动文件（startup_xxx.s）中定义了 Stack_Size（栈大小）和 Heap_Size（堆大小）的宏，具体形成过程如下：
>- **栈（Stack）**：在 `startup` 汇编里，会将 `__initial_sp`（栈顶指针）赋值给 `MSP` 寄存器。栈是 **“向下生长”** 的（从高地址往低地址长），每当函数里声明局部变量或发生函数调用压栈时，MSP 指针就会向下移动。
>- **堆（Heap）**：通常在 C 运行时库初始化（`__main`）阶段，会设置 `__heap_start` 和 `__heap_end` 两个指针。**堆是“向上生长”** 的（从低地址往高地址长），当你在代码中调用 `malloc` 申请动态内存时，堆指针就会向上移动。

### 参考文献

1. [揭秘STM32内存布局：从Flash到SRAM全解析](https://blog.csdn.net/2402_83411382/article/details/151901683)
2. [STM32堆和栈（Heap & Stack）及SRAM存储使用](https://www.cnblogs.com/icaowu/p/18178959)

---

## STM32启动流程与BootLoader

### 启动流程

![[Pasted image 20260715202733.png]]

**第一步：硬件复位与向量表提取**

当 STM32 上电或按下复位键时：

1. **读取启动配置**：硬件会根据 `BOOT0` 和 `BOOT1` 引脚的电平，决定从 **主 Flash（最常用）、系统存储器（ISP 下载模式）还是 SRAM** 中启动。
    
2. **获取栈顶指针（MSP）**：CPU 自动从启动地址的 **第 1 个字（0x00000000）** 读取主堆栈指针（MSP）的初始值，并赋给 `SP` 寄存器。
    
3. **获取复位向量（PC）**：CPU 自动从启动地址的 **第 2 个字（0x00000004）** 读取复位中断处理函数 `Reset_Handler` 的入口地址，并赋给 `PC` 寄存器。
    
4. **执行跳转**：CPU 紧接着就开始执行 `Reset_Handler` 中的汇编指令。

Reset_Handler：

``` c
; Reset handler routine
Reset_Handler    PROC
                 EXPORT  Reset_Handler                 [WEAK]
        IMPORT  __main
        IMPORT  SystemInit  
                 LDR     R0, =SystemInit
                 BLX     R0
                 LDR     R0, =__main
                 BX      R0
                 ENDP
```

**第二步：汇编启动文件（`startup_stm32xxx.s`）中的 C 环境搭建**

这是启动文件中（通常由 ST 官方提供，后缀为 `.s`）主要干的事情：

1. **初始化堆栈（Stack）**：设置 MSP 和 PSP（若使用 RTOS）的栈底位置。
    
2. **开启 FPU（浮点运算单元）**：如果是 STM32F4/F7/H7 等带硬件 FPU 的芯片，汇编会配置 `CPACR` 寄存器，开启协处理器，确保后续 C 代码中能使用 `float` 和 `double` 运算。
    
3. **搬运 `.data` 段（已初始化全局变量）**：将 Flash 中保存的带有初值的全局变量（如 `int a = 10;`），复制到 SRAM 的 `.data` 区域，让程序能访问到初始值。
    
4. **清零 `.bss` 段（未初始化全局变量）**：将未赋初值的全局变量（如 `int b;`）所在的 SRAM 区域全部清零，防止出现随机乱码。
    
5. **调用 `SystemInit`**：这是一个 C 语言编写的弱函数（ST 官方已经写好），在里面配置**系统时钟树**（如使能 HSE、配置 PLL 倍频、配置 Flash 读取等待周期等）。

**第三步：C 运行时库初始化（`__main`）**

汇编代码执行完后，不会直接跳到 `main()`，而是调用了一个 C 编译器（如 ARMCC 或 GCC）内部自带的函数 `__main`（注意前面有两个下划线）。

- `__main` 是 C 语言的 **“运行时入口”**。它负责进一步初始化 C 库（例如如果是 C++，这里负责调用全局对象的构造函数；或者标准库的 `stdio` 底层等等）。
    
- 在 `__main` 执行完毕后，**最终才通过一条 `BL main` 汇编指令，跳转到了你写的 `main()` 函数**。

**第四步：进入 `main()` 与中断正常响应**

当进入 `main()` 时：

1. 此时外设时钟尚未全部开启（除了 `SystemInit` 可能配置的 HSE/PLL）。你通常要在 `main()` 里调用 `HAL_Init()` 和 `MX_GPIO_Init()` 等去打开对应的时钟。
    
2. **中断向量表偏移（VTOR）**：如果你使用了 Bootloader（引导程序），必须手动修改 `SCB->VTOR` 寄存器，将中断向量表重定位到新程序的地址。否则发生中断时，CPU 依然会去默认的 Flash 头部查找中断入口，导致程序跑飞。

### BootLoader怎么实现的？

#### 思路

bootloader其实就是一段启动程序，它在芯片启动的时候首先被执行，它可以用来做一些硬件的初始化，当初始化完成之后跳转到对应的应用程序中去。

#### 程序跳转

在STM32中只要**将要跳转的地址直接写入PC寄存器**，就可以跳转到对应的地址中去。
当我们实现一个函数的时候，这个函数最终会占用一段内存，而**它的函数名代表的就是这段内存的起始地址**。当我们调用这个函数的时候，单片机会将这段内存的首地址（函数名对应的地址）加载到PC寄存器中，从而跳转到这段代码来执行。那么我们也可以利用这个原理，**定义一个函数指针，将这个指针指向我们想要跳转的地址**，然后调用这个函数，就可以实现程序的跳转了。

示例代码：
``` c
#define  APP_ADDR  0x08002000   //应用程序首地址定义 
typedef void (*APP_FUNC)(); //函数指针类型定义
APP_FUNC jump2app; //定义一个函数指针
jump2app = ( APP_FUNC )(APP_ADDR + 4); //给函数指针赋值
jump2app(); //调用函数指针，实现程序跳转
```

>`APP_ADDR + 4`是因为STM32启动时，首先会从内存地址位0x0800 0000(由启动模式决定)的地方加载栈顶地址（4字节），从0x0800 0004的地方加载程序复位地址（4字节），然后跳转到对应的复位地址去执行。

#### 加载栈地址

由于我们是执行BootLoader跳转到APP，因此我们需要手动加载栈顶指针（MSP）。

以Keil举例：

``` c
__asm void MSR_MSP(uint32_t addr)
{
    MSR MSP, r0
    BX r14;
}
```

- MSR MSP, r0 意思是将r0寄存器中的值加载到MSP（主栈寄存器，复位时默认使用）寄存器中，r0中保存的是参数值，即addr的值  
- BX r14 跳转到连接寄存器保存的地址中，即退出函数，跳转到函数调用地址

#### 编译设置

![[Pasted image 20260716105505.png]]

我们需要在设置界面将默认（0x8000000）改为我们的应用程序地址（0x8002000）

#### 中断向量表重映射

``` c
/**
  * @brief  Setup the microcontroller system.
  * @param  None
  * @retval None
  */
void SystemInit (void)
{
/*!< Set MSION bit */
  RCC->CR |= (uint32_t)0x00000100U;

  /*!< Reset SW[1:0], HPRE[3:0], PPRE1[2:0], PPRE2[2:0], MCOSEL[2:0] and MCOPRE[2:0] bits */
  RCC->CFGR &= (uint32_t) 0x88FF400CU;

  /*!< Reset HSION, HSIDIVEN, HSEON, CSSON and PLLON bits */
  RCC->CR &= (uint32_t)0xFEF6FFF6U;

  /*!< Reset HSI48ON  bit */
  RCC->CRRCR &= (uint32_t)0xFFFFFFFEU;

  /*!< Reset HSEBYP bit */
  RCC->CR &= (uint32_t)0xFFFBFFFFU;

  /*!< Reset PLLSRC, PLLMUL[3:0] and PLLDIV[1:0] bits */
  RCC->CFGR &= (uint32_t)0xFF02FFFFU;

  /*!< Disable all interrupts */
  RCC->CIER = 0x00000000U;

  /* Configure the Vector Table location add offset address ------------------*/
#ifdef VECT_TAB_SRAM
  SCB->VTOR = SRAM_BASE | VECT_TAB_OFFSET; /* Vector Table Relocation in Internal SRAM */
#else
  SCB->VTOR = FLASH_BASE | VECT_TAB_OFFSET; /* Vector Table Relocation in Internal FLASH */
#endif
}
```

默认SystemInit函数中会初始化中断向量表（SCB）默认为基地址（0x0800 0000），而APP地址实际在0x08002000，因此我们需要手动修改`SCB->VTOR = 0x08002000`。

#### 完整程序

``` c
#define APP_ADDR 0x08002000 //应用程序首地址定义 
typedef void (*APP_FUNC)(); //函数指针类型定义/**

__asm void MSR_MSP(uint32_t addr)
{
    MSR MSP, r0
    BX r14;
}

void run_app(uint32_t app_addr)
{
    uint32_t reset_addr = 0;
    APP_FUNC jump2app;
    
    /* 跳转之前关闭相应的中断 */
    NVIC_DisableIRQ(SysTick_IRQn);
    NVIC_DisableIRQ(LPUART_IRQ);
    
    /* 栈顶地址是否合法(这里sram大小为8k) */
    if(((*(uint32_t *)app_addr)&0x2FFFE000) == 0x20000000)
    {
        /* 设置栈指针 */
        MSR_MSP(app_addr);
        /* 获取复位地址 */
        reset_addr = *(uint32_t *)(app_addr+4);
        jump2app = ( APP_FUNC )reset_addr;
        jump2app();
    }
    else
    {
        printf("APP Not Found!\n");
    }
}

void main(){
	/* 一些外设初始化代码
	...
	*/
	run_app(APP_ADDR);
	while(1){}
}
```

### 参考文献

1. [基于STM32的简易Bootloader实现](https://www.cnblogs.com/jiuliblog-2016/p/11411887.html)
2. [STM32单片机实现Bootloader跳转的关键步骤](https://zhuanlan.zhihu.com/p/648855822)
3. [[STM32]HAL库实现自己的BootLoader-BootLoader与OTA-STM32CUBEMX](https://jishuzhan.net/article/1819327870636396546)
4. [无调试器时栈回溯解决hardfault问题](https://blog.csdn.net/qq_51385715/article/details/149296052)

---

## 一些不会的面试题

1. Q：ARM Cortex-M 内核中，如何通过栈回溯定位 HardFault 异常的根本原因？
A：可以通过调试器或直接重写HardFault 函数得到PC与LR寄存器值，并通过addr2line工具得到具体出错的行号。

2. Q：STM32系列单片机所用的架构和内核？Cortex-M3特点及与M4内核的区别？
A：STM32采用的是哈佛架构，一般是Cortex-M系列内核。Cortex-M3 是基于**ARMv7-M架构**的32位处理器内核，具有3级流水线，支持硬件除法和位带操作，支持嵌套向量中断控制器，同时可以添加丰富外设，广泛应用于微控制器、汽车电子、传感器等，具有**高性能、低功耗与低成本**的优势；M4与M3内核相比，主要就是添加了浮点单元和DSP指令集，适合需要大量浮点和数学运算的场景中。

3. Q：PWM定时器输入捕获？
A：通过检测TIMx_CHx通道上的边沿信号，在边沿信号发生跳变（比如上升沿/下降沿）的时候，将当前定时器的值（TIMx_CNT）存放到对应的捕获/比较寄存器（TIMx_CCRx）里面，完成一次捕获。

4. Q：CPU中断响应流程？
A：当发生中断后，首先就是保存现场，将寄存器压入栈中，设置LR寄存器（决定MSP还是PSP进行恢复），再从中断向量表中取入口地址装入PC中，开始执行中断函数，执行完毕后恢复现场，根据LR寄存器的值决定返回模式，寄存器出栈并恢复PC指针，回到主程序继续运行。

5. Q：中断中执行耗时操作的影响？
A：会影响后台程序的实时性（主程序代码不能及时运行、同级或低优先级中断被阻塞）；导致系统看门狗复位。

6. GPIO 中断假如有很多个，怎么判断是哪一个
A：GPIO中断也就是外部中断，使用外设EXTI边沿检测+NVIC实现。但是实际中可能多个引脚挂在同一根中断向量线上，所以在触发之后需要进入中断服务丽数中之后，通过读取EXIT的挂起寄存器(EXIT→PR)并且按位与&操作判断到底是哪一位触发了中断，处理完之后该位必须写1清零。
补充:PR相关知识
PR寄存器里面存放32数据，相当于一份待办清单。每一位对应一根外部中断线，EXIT检测到边沿变化时硬件自动将对应位置1(空闲为0)，该中断请求被挂起等待CPU处理，故处理完之后要给该位写1清零。W1C(write 1 to clear)
补充:用if-else按位与太慢怎么办?
1)可以用lowbit法直接找最地位的1。也就是x&-x(-x=x取反+1)，所以x&-x的结果就是当前的最低位的 1。然后再建立一个switch 查表即可。无需一位一位判断到底哪个是1。2)CLZ前导零计数，是一个硬件指令可以计算一个32位数前面一共有几个0,能直接锁定最后一位1。