
---

## 🔍 核心概念

I2C（Inter-Integrated Circuit）：
• 由Philips（现NXP）发明的**同步、半双工、串行通信总线**
• 仅用2根线：**SDA（数据）、SCL（时钟）**
• 支持**多主机、多从机**，**通过地址寻址（7bit/10bit）**
• 典型速率：标准模式**100kbps** / 快速模式**400kbps** / 高速模式**3.4Mbps**
• 应用场景：EEPROM、RTC、传感器（温湿度/加速度）、ADC/DAC、OLED屏等**中低速外设**
>✅ **面试金句**：
>I2C**通过地址寻址实现多设备挂载**，硬件简单但协议严谨，适合**板级低速外设通信**，调试时**重点关注时序和上拉电阻**。

## ⚙️ 物理层关键参数

|项目|参数/要求|备注|
|---|---|---|
|**信号线**|SDA（数据）、SCL（时钟）|开漏输出，必须外接上拉电阻|
|**上拉电阻**|1.8kΩ~10kΩ（**常用4.7kΩ**）|阻值越小速度越快，但功耗越大；总线电容大时需减小阻值|
|**通信速率**|标准、快速、高速|速率越高对上拉电阻和布线要求越严|
|**总线电容**|≤400pF（400kbps时）|**电容过大会导致边沿变缓，通信失败**|
|**传输距离**|板级通信**（<1米）**|长线需用I2C缓冲器或改用CAN/RS485|

>✅ **调试经验**：
>用示波器看SCL/SDA波形，**正常应为方波**；若**上升沿缓慢→上拉电阻过大或电容负载过重**。

## 📡 协议层核心时序（必画图！）

1. 起始/停止信号
	- START：SCL高电平时，SDA **高→低** 跳变
	- STOP：SCL高电平时，SDA **低→高** 跳变

2. 数据帧格式（1字节数据=8数据位+1应答位）
	- {START} → {7位地址+读写位+应答位} → {N字节数据} → {STOP}
	- ACK：接收方拉低SDA（第9个时钟周期）
	- NACK：接收方释放SDA（用于主机读时结束传输）

3. 主机写数据到从机
	``` mermaid
	sequenceDiagram
    participant Host
    participant Slave
    Host->>Slave: START
    Host->>Slave: 发送从机地址+写命令(0)
    Slave-->>Host: ACK
    Host->>Slave: 发送寄存器地址
    Slave-->>Host: ACK
    Host->>Slave: 发送数据字节
    Slave-->>Host: ACK
    Host->>Slave: STOP
	```

4. 主机读取从机数据
	``` mermaid
	sequenceDiagram
    participant M as 主机 (Master)
    participant S as 从机 (Slave)

    M->>S: START
    M->>S: 发送从机地址 + 写位(0) [定位寄存器]
    S-->>M: ACK
    M->>S: 发送寄存器地址
    S-->>M: ACK
    M->>S: START (⚠️关键：不发STOP)
    M->>S: 发送从机地址 + 读位(1) [切换通信方向]
    S-->>M: ACK
    S-->>M: 返回数据字节1
    M-->>S: ACK (表示继续接收)
    S-->>M: 返回数据字节N (最后一字节)
    M-->>S: NACK (表示接收完毕)
    M->>S: STOP
	```