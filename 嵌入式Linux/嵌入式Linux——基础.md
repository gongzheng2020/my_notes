
---
## 简介

![[Pasted image 20260717135103.png]]

![[Pasted image 20260717135451.png]]

嵌入式Linux系统 = bootloader + linux内核 + 根文件系统(里面含有APP)  
booloader = 裸机集合  
Linux内核 = 驱动集合 + 进程调度 + 内存管理等  
Linux驱动程序 = 驱动框架 + 硬件操作

驱动框架的哲学思想：“一切皆文件”(也是Linux的哲学思想)  
所有驱动程序都提供一样的接口：open/read/write/close等。

---
## 驱动与应用程序

### 字符设备驱动

#### 设备分类

**字符设备**:以**字节流的形式访问，严格按顺序读写**。通过 /dev/ 目录下的设备节点（如 /dev/ttyS0），使用 open()、read()、write() 等系统调用。用于串口（UART）、键盘、鼠标、LED灯、I2C/SPI 设备、温度传感器等。核心是 file_operations 结构体（包含 open, read, write, ioctl 等函数指针）。

**块设备**:以**固定大小的块（Block，通常是 512 字节或 4KB）为单位访问，且支持随机访问（跳转到任意位置读写）**。同样有 /dev 节点（如 /dev/sda），但块设备上通常要挂载文件系统（如 ext4、FAT32）。用于SD卡、eMMC、U盘、机械硬盘（HDD）、固态硬盘（SSD）等。为了提升效率，Linux 会在块设备和应用程序之间加一层“页缓存（Page Cache）”，先把数据读进内存，再由 CPU 处理，所以读块设备通常比直接读硬件快很多。核心是 block_device_operations，更关注请求队列（Request Queue）的处理。

**网络设备**:以**数据包（Packet）为单位进行收发，没有 /dev 节点，不能通过普通的 read/write 文件方式访问**。必须通过 Linux 内核的网络协议栈（TCP/IP）和 Socket API（如 socket(), bind(), send(), recv()）来进行通信。用于以太网卡（Ethernet）、WiFi 模块、4G/5G 模块等。核心是 net_device 结构体和 net_device_ops 结构体（包含 ndo_open, ndo_start_xmit 等函数指针）。注意，网络设备的数据到达后，内核不会把它交给某个应用程序，而是交给协议栈去分发（判断是 TCP 还是 UDP 等）。

Linux内核中处处体现面向对象的设计思想，为了统一形形色色的设备，Linux系统将设备**分别抽象为`struct cdev`、`struct block_device`、`struct net_devce`三个对象**，具体的设备都可以包含着三种对象从而继承和三种对象属性和操作， 并通过各自的对象添加到相应的驱动模型中，从而进行统一的管理和操作

#### 字符设备驱动介绍

![[Pasted image 20260822141835.png]]

这张图是 Linux **虚拟文件系统（VFS）** 架构中极其经典的示意图，主要展示了进程是如何通过**文件描述符**一步步找到底层硬件，并完成读写操作的。

 1. 下半部分：进程层（用户视角）
	- `struct task_struct`：代表一个运行中的进程。
	- `struct files_struct` 和 `fd_array[]`：这是进程的**文件描述符表**。比如你在代码里写 `int fd = open(...)`，返回的 `fd=3` 实际上就是数组 `fd_array` 的**下标（索引）**。
	- `struct file`：代表一个**被打开的实例**（包含当前文件的读取偏移量、打开模式等动态信息）。不同的进程可以打开同一个文件，各指向不同的 `struct file`。

2. 上半部分：文件系统层（内核/驱动视角）
	- `inode`（索引节点）：代表**“文件本身”**，包含文件的静态信息（权限、大小、在磁盘/Flash上的位置等）。它内部有一个指针，指向 `struct file_operations`。
	- `struct file_operations`（文件操作集合）：这是**驱动层的核心**，是一堆函数指针（包括 `open`, `read`, `write`, `close` 等）。它相当于一份“说明书”，告诉内核：对于这个文件，它的“读”对应底层哪个函数，“写”对应底层哪个函数。
	- 底层寄存器配置：代表实际的硬件（如串口、I2C、SPI 控制器寄存器）。

这张图完美诠释了 Linux **一切皆文件**的思想。无论你是普通文件、串口（字符设备）、硬盘（块设备），还是网卡（网络设备），只要在上层挂载了 `struct file_operations`，进程就可以用统一的 `open/read/write` 接口，通过 `fd` 穿透到底层硬件，使得驱动开发与应用程序彻底解耦。

#### 一些基本概念

此处略过，具体查看这个网站教程的[第三小节](https://doc.embedfire.com/linux/imx6/driver/zh/latest/linux_driver/character_device.html)

#### 字符设备驱动程序框架

![[Pasted image 20260822153359.png]]

一、第一阶段：驱动初始化（重点在“申请”和“注册”）

1. **分配 dev_t（设备号）**：
    
    - `register_chrdev_region()`：**静态分配**。自己指定一个主设备号（比如 200），如果内核里已经有别的驱动用了这个号，就会报错。
        
    - `alloc_chrdev_region()`：**动态分配**。让内核自动分配一个空闲的主设备号（推荐使用这种方式，不容易冲突）。
        
    - > _通俗理解：这是去居委会（内核）给这个驱动领一个“门牌号”。_

2. **file_operations**（实现设备操作（核心业务））：
    
    - 这是一个结构体，里面全是**函数指针**。
        
    - 驱动开发者需要实现 `.open`, `.read`, `.write`, `.release` (对应close) 等函数，并填进这个结构体里。
        
    - 当应用层调用 `read()` 时，内核就会根据 VFS 找到这个结构体，最终调用你写好的 `.read` 函数。而你的 `.read` 函数内部，就是去操作底层的**硬件寄存器**（比如你之前学的串口、I2C、ADC等寄存器）。

3. **初始化 cdev**：
    
    - `cdev_init()`：在内存中申请并初始化一个 `struct cdev`（字符设备）结构体。最关键的一步是**将你自己的 `file_operations`（操作函数集合）绑定到这个结构体上**。
        
4. **注册 cdev**：
    
    - `cdev_add()`：把刚才初始化好的 `cdev` 结构体，**挂载到内核的设备驱动表中**，并与刚才分配的设备号绑定。
        
    - > _通俗理解：房子盖好了，服务人员（函数）也安排好了，现在把地址告诉居委会，正式把“门牌号”挂上去，以后内核就能找到这个驱动了。_

二、第二阶段：新建设备节点

4. **mknod + 主从设备号**：
    
    - 这一步是在 `/dev` 目录下创建一个**设备文件**（比如 `/dev/mydev`）。命令中的主设备号告诉内核该找哪个驱动，从设备号告诉内核该找该驱动下的哪个具体物理设备。
        
    - > _现代 Linux 中，这一步通常不需要手动敲命令，而是由**udev** 根据设备树（Device Tree）自动完成创建。但理解 `mknod` 是理解设备节点本质的基础。_
        
    - **用户态程序的入口**：有了这个文件，应用层的 `open("/dev/mydev", ...)` 就能打通进内核了。

三、驱动注销（卸载）

6. **cdev_del()**：
    
    - 卸载驱动时，先从内核中删除这个字符设备，让它无法再接收调用。
        
7. **unregister_chrdev_region()**：
    
    - 将“门牌号”（设备号）退回给内核，这样后面的驱动才能继续使用这个号。
        
    - > _如果忘了注销，驱动卸载后，设备号依然被占用；后续再次加载时，可能会因为“资源被占用”而失败。_

#### open函数到底做了什么？

![[Pasted image 20260822160048.png]]

1. 第一步：用户态发起系统调用（图中 ①）

	应用层执行 `fd = open("/dev/xxx", O_RDWR)`。这触发了软中断（如 ARM 下的 `svc` 指令），CPU 陷入内核态，进入系统调用入口 `sys_open`（内核源码 `fs/open.c`）。

2. 寻找空闲的文件描述符（图中 ②）

	进入 `do_sys_open` 后，会首先调用 `get_unused_fd_flags`。

	- **做了什么**：扫描当前进程的 `struct files_struct` 中的 `fd_array`（也就是文件描述符表），找到一个空闲的下标（比如 3 或 5），这个下标就是将来返回给用户态的 `fd`。
	    
	- **此时**：只是占了个坑，还没有指向任何真实的数据结构。
	    

3. 解析路径，找到 inode

	调用 `do_filp_open`。
	
	- **做了什么**：内核根据传入的路径字符串 `/dev/xxx`，沿着目录树层层向下查找，最终定位到这个设备文件对应的 **`inode`（索引节点）**。
	    
	- **关键点**：`inode` 是静态的，里面存放了文件的所有者、权限，以及一个非常重要的指针 **`i_fop`**（指向 `struct file_operations`）。对于字符设备，这个 `i_fop` 就是在驱动初始化（`cdev_add`）时绑定的 `file_operations` 结构体。
	    

4. 建立“动态”与“静态”的连接（**全图最核心的一步**）

	调用 `do_dentry_open`（图片中箭头指向 `struct file 结构体成员 f_op` 的地方）。
	
	- **做了什么**：内核在内存中分配一个动态的 `struct file` 结构体。
	    
	- **关键动作**：**将 `inode` 里的 `i_fop` 指针，直接赋值给 `struct file` 的 `f_op` 成员**。
	    
	- **为什么要这样做？** 因为 `inode` 是“静态的说明书”，而 `struct file` 是“动态的门禁卡”。这步执行后，后续用户态的 `read()`, `write()` 系统调用，就能通过这个 `f_op` 指针，直接找到驱动里写好的函数。
	    
	- **额外动作**：在 `do_dentry_open` 的末尾，内核会调用 `f_op->open()`。这就是**驱动层自己实现的 `open` 函数**。驱动在这里通常会做硬件初始化（比如打开串口电源、配置引脚、申请中断资源），并把这个操作的返回值返回给上层。
    

5. 完成绑定（图中最后一步 `fd_install`）

	调用 `fd_install`。
	
	- **做了什么**：将刚才初始化好的 `struct file *` 指针，正式塞进进程的 `fd_array[fd]` 中。
	    
	- **结果**：从此，用户态的 `fd=3` 就牢牢锁定了内核里的 `struct file`，而 `struct file` 又锁定了底层驱动的 `file_operations`。
    

6. 返回 fd

	`sys_open` 返回步骤二中分配的文件描述符（`fd`）给用户程序。用户程序拿着这个 `fd` 就可以为所欲为了。

#### 字符驱动设备实例

[[嵌入式Linux——字符驱动设备程序测试 | 可以看这篇博客]]

#### 同个驱动支持多个设备

**一个驱动程序（driver）如何同时管理多个“类似但独立”的设备？**
在 Linux 中，**主设备号（Major）** 告诉内核“用哪个驱动”，**次设备号（Minor）** 告诉内核“用这个驱动下的哪一个具体设备”（比如串口0、串口1）。
核心思路是：**利用 `file` 结构体里的 `private_data` 成员。**
[主要可以参考这篇教程](https://doc.embedfire.com/linux/imx6/driver/zh/latest/linux_driver/character_device.html#id23)

**方法一（用 switch 判断次设备号）：**
准备两个全局缓冲区 vbuf1 和 vbuf2。
- 在 `open` 函数里，使用`switch`函数根据次设备号进行判断：
	```C
	static int chr_dev_open(struct inode *inode, struct file *filp)
	{
		printk("\nopen\n ");
		switch(MINOR(inode->i_rdev)){
			case 0 : filp->private_data = vbuf1; break;
			case 1 : filp->private_data = vbuf2; break;
		}
		return 0;
	}
	```
	这里用宏 `MINOR(inode->i_rdev)` 拿到了当前打开的是哪个次设备（0还是1）。如果是0，就把私有数据指针指向 `vbuf1`；是1，就指向 vbuf2。

- 在 `read/write` 函数里：
	直接拿 `char *vbuf = filp->private_data;` 来用。这样，你往设备0写数据，就会写进 `vbuf1`；往设备1写，就会写进 `vbuf2`。两边的数据完全隔离，互不干扰。

- **缺点：如果设备有100个，你就要写100个 case，代码很啰嗦。**

**方法二（用 `container_of` 宏）：**

这是现代内核中最标准的做法。既然一个驱动要管多个设备，为什么不把 `cdev`（设备对象）和 `vbuf`（缓冲区）**打包在一起**呢？

- **定义结构体**：
    ```C
	struct chr_dev {
       struct cdev dev;   // 内核要求的字符设备结构体
       char vbuf[BUFF_SIZE]; // 这个设备专属的缓冲区
    };
    static struct chr_dev vcdev1;
    static struct chr_dev vcdev2;
    ```

- 初始化时直接同时关联两个设备，相当于把两个独立的设备都注册到内核里了：
	```C
	//关联第一个设备：vdev1
	cdev_init(&vcdev1.dev, &chr_dev_fops);
	ret = cdev_add(&vcdev1.dev, devno+0, 1);
	if(ret < 0){
		printk("fail to add vcdev1 ");
		goto add_err1;
	}
	//关联第二个设备：vdev2
	cdev_init(&vcdev2.dev, &chr_dev_fops);
	ret = cdev_add(&vcdev2.dev, devno+1, 1);
	if(ret < 0){
		printk("fail to add vcdev2 ");
		goto add_err2;
	}
	```

- `open` 函数中，使用`container_of`函数进行处理：
	```C
	static int chr_dev_open(struct inode *inode, struct file *filp)
	{
	    printk("open\n");
	    filp->private_data = container_of(inode->i_cdev, struct chr_dev, dev);
	    return 0;
	}
	```
    
    **这是最难懂的一点，我给你拆解**：
    
    - 内核在打开设备时，会往 `inode->i_cdev` 里填入**内核自己的 `cdev` 结构体的地址**（即 `&vcdev1.dev` 或者 `&vcdev2.dev`）。
        
    - 但是我们的 `cdev` 被包在了我们自己的 `struct chr_dev` 里面。
        
    - `container_of` 是一个极其高明的宏，它的作用是**通过结构体内部的一个成员的地址，反向推算出整个结构体的首地址**。
        
    - 所以，它拿着 `inode->i_cdev` 这个地址，减去 `dev` 成员在结构体里的偏移量，就自动推算出了 `vcdev1` 或 `vcdev2` 的起始地址，然后赋给 `private_data`。
        
- **后续 `read/write` 函数**：
	```C
	struct chr_dev *dev = filp->private_data;
	char *vbuf = dev->vbuf;
	```
    这就非常清晰了：先拿到整个设备对象，再访问它独有的缓冲区。

### 设备驱动基础知识

#### 设备驱动本质与作用

设备驱动本质是一个**“翻译器”**和**“桥梁”**。它运行在操作系统内核态（拥有最高特权级），向下直接操作底层硬件寄存器，向上向操作系统内核提供标准化的服务接口。

它主要有以下两个作用：
1. 将复杂逻辑操作进行封装，使得应用程序可使用统一的系统调用接口来访问各种设备；
2. 实现了资源管理与保护，包括内存映射、数据隔离与并发控制；

#### 内存管理单元MMU与地址转换函数

MMU 的存在，让 Linux 的驱动开发多了一步“申请虚拟地址”的步骤，即把**“直接操作物理寄存器”**变成了**“先 `ioremap` 映射，再对虚拟地址进行标准的 `ioread/iowrite`”**。它主要有以下两个作用：

1. **保护内存**： MMU给一些指定的内存块设置了读、写以及可执行的权限，这些权限存储在页表当中，MMU会检查CPU当前所处的是特权模式还是用户模式，只有与当前所设置的权限匹配才可以访问，如果CPU要访问一段虚拟地址，则将虚拟地址转换成物理地址，否则将产生异常，防止内存被恶意地修改。
2. **提供方便统一的内存空间抽象，实现虚拟地址到物理地址的转换**： CPU可以运行在虚拟的内存当中，虚拟内存一般要比实际的物理内存大很多，使得CPU可以运行比较大的应用程序。

实际的地址转换函数，包括ioremap()地址映射和取消地址映射iounmap()函数：

1. `ioremap()`函数
	- 函数原型为:
		```C
		void __iomem *ioremap(phys_addr_t paddr, unsigned long size)
		#define ioremap ioremap
		```
	**参数**：**paddr：** 被映射的IO起始地址（物理地址）；**size：** 需要映射的空间大小，以字节为单位；
	**返回值**：一个指向__iomem类型的指针，当映射成功后便返回一段虚拟地址空间的起始地址，我们可以通过访问这段虚拟地址来实现实际物理地址的读写操作。
	
	- 为了符合驱动的跨平台以及可移植性， 我们应该使用linux中指定的函数：
		```C
		unsigned int ioread8(void __iomem *addr)
		unsigned int ioread16(void __iomem *addr)
		unsigned int ioread32(void __iomem *addr)
		
		void iowrite8(u8 b, void __iomem *addr)
		void iowrite16(u16 b, void __iomem *addr)
		void iowrite32(u32 b, void __iomem *addr)
		```
	
	- 一个实际例子（操作寄存器）如下：
		```C
		unsigned long pa_dr = 0x20A8000 + 0x00;
		unsigned int __iomem *va_dr;
		unsigned int val;
		va_dr = ioremap(pa_dr, 4);
		val = ioread32(va_dr);
		val &= ~(0x01 << 19);
		iowrite32(val, va_dr);
		```

1. `iounmap`函数
	- 函数原型如下：
		```C
		void iounmap(void *addr)
		#define iounmap iounmap
		```
	**参数**：**paddr：** 需要取消ioremap映射之后的起始地址（虚拟地址）。；
	**返回值**：无。
	- 一个实际例子（取消映射后的虚拟地址）如下：
		```C
		iounmap(va_dr);     //释放掉ioremap映射之后的起始地址（虚拟地址）
		```

### 设备模型

设备模型通过几个数据结构来反映当前系统中总线、设备以及驱动的工作状况，提出了以下几个重要概念：

- **设备(device)** ：挂载在某个总线的物理设备；
    
- **驱动(driver)** ：与特定设备相关的软件，负责初始化该设备以及提供一些操作该设备的操作方式；
    
- **总线（bus)** ：负责管理挂载对应总线的设备以及驱动；
    
- **类(class)** ：对于具有相同功能的设备，归结到一种类别，进行分类管理；

![[Pasted image 20260825151256.png]]

上面图片中所描述的流程如下：

1. 在总线上管理着两个链表，分别管理着设备和驱动，当我们向系统注册一个驱动时，便会向驱动的管理链表插入我们的新驱动， 同样当我们向系统注册一个设备时，便会向设备的管理链表插入我们的新设备。
2. 在插入的同时总线会执行一个bus_type结构体中match的方法对新插入的设备/驱动进行匹配。 (它们之间最简单的匹配方式则是对比名字，存在名字相同的设备/驱动便成功匹配)。 
3. 在匹配成功的时候会调用驱动device_driver结构体中probe方法(通常在probe中获取设备资源，具体的功能可由驱动编写人员自定义)， 并且在移除设备或驱动时，会调用device_driver结构体中remove方法。

#### 总线

![[Pasted image 20260825153203.png]]

总线是连接**处理器**和**设备**的桥梁，规定了同类设备共同遵守的**工作时序**（如 I2C、USB、SPI 等）。

![[Pasted image 20260825153215.png]]

总线驱动则负责实现总线的各种行为，其管理着两个链表：
- 添加到该总线的**设备链表**；
- 注册到该总线的**驱动链表**。

当你向总线添加（移除）一个设备（驱动）时，便会在对应的列表上添加新的节点， 同时对挂载在该总线的驱动以及设备进行匹配。在内核中使用结构体`bus_type`来表示总线。

```C
struct bus_type {
    const char              *name;
    const struct attribute_group **bus_groups;
    const struct attribute_group **dev_groups;
    const struct attribute_group **drv_groups;

    int (*match)(struct device *dev, struct device_driver *drv);
    int (*uevent)(struct device *dev, struct kobj_uevent_env *env);
    int (*probe)(struct device *dev);
    int (*remove)(struct device *dev);

    int (*suspend)(struct device *dev, pm_message_t state);
    int (*resume)(struct device *dev);

    const struct dev_pm_ops *pm;

    struct subsys_private *p;
};
```

- **name** :指定总线的名称，当新注册一种总线类型时，会在/sys/bus目录创建一个新的目录，目录名就是该参数的值；
    
- **drv_groups、dev_groups、bus_groups** :分别表示驱动、设备以及总线的属性。这些属性可以是内部变量、字符串等等。通常会在对应的/sys目录下在以文件的形式存在，对于驱动而言，在目录`/sys/bus/<bus-name>/driver/<driver-name>`存放了设备的默认属性；设备则在目录`/sys/bus/<bus-name>/devices/<driver-name>`中。这些文件一般是可读写的，用户可以通过读写操作来获取和设置这些attribute的值。
    
- **match** :当向总线注册一个新的设备或者是新的驱动时，会调用该回调函数。该回调函数主要负责判断是否有注册了的驱动适合新的设备，或者新的驱动能否驱动总线上已注册但没有驱动匹配的设备；
    
- **uevent** :总线上的设备发生添加、移除或者其它动作时，就会调用该函数，来通知驱动做出相应的对策。
    
- **probe** :当总线将设备以及驱动相匹配之后，执行该回调函数,最终会调用驱动提供的probe函数。
    
- **remove** :当设备从总线移除时，调用该回调函数；
    
- **suspend、resume** :电源管理的相关函数，当总线进入睡眠模式时，会调用suspend回调函数；而resume回调函数则是在唤醒总线的状态下执行；
    
- **pm** :电源管理的结构体，存放了一系列跟总线电源管理有关的函数，与device_driver结构体中的pm_ops有关；
    
- **p** :该结构体用于存放特定的私有数据，其成员klist_devices和klist_drivers记录了挂载在该总线的设备和驱动；

![[Pasted image 20260825153911.png]]

当我们成功注册总线时，会在/sys/bus/目录下创建一个新目录，目录名为我们新注册的总线名。bus目录中包含了当前系统中已经注册了的所有总线，例如i2c，spi，platform等。我们看到每个总线目录都拥有两个子目录devices和drivers， 分别记录着挂载在该总线的所有设备以及驱动。

>内核中提供了bus_register函数来注册总线，以及bus_unregister函数来注销总线，但一般内核中包含了大部分总线，我们只要会怎么使用即可。

#### 设备

![[Pasted image 20260825162522.png]]

- **设备文件化**：在 Linux 中一切皆文件，设备也不例外。
- **`/sys/devices`**：**真实**存放所有设备信息的地方，是设备树的根。  
- **`/sys/dev`**：存放设备节点，但**全部是符号链接（软链接）**，最终均指向 `/sys/devices` 目录下的对应文件。

在内核使用`device`结构体来描述我们的物理设备。

```C
struct device {
	const char *init_name;
	struct device           *parent;
	struct bus_type *bus;
	struct device_driver *driver;
	void            *platform_data;
	void            *driver_data;
	struct device_node      *of_node;
	dev_t                   devt;
	struct class            *class;
	void (*release)(struct device *dev);
	const struct attribute_group **groups;  /* optional groups */
	struct device_private   *p;
};
```

- **init_name** :指定该设备的名称，总线匹配时，一般会根据比较名字，来进行配对；
    
- **parent** :表示该设备的父对象，前面提到过，旧版本的设备之间没有任何关联，引入Linux设备模型之后，设备之间呈树状结构，便于管理各种设备；
    
- **bus** :表示该设备依赖于哪个总线，当我们注册设备时，内核便会将该设备注册到对应的总线。
    
- **of_node** :存放设备树中匹配的设备节点。当内核使能设备树，总线负责将驱动的of_match_table以及设备树的compatible属性进行比较之后，将匹配的节点保存到该变量。
    
- **platform_data** :特定设备的私有数据，通常定义在板级文件中；
    
- **driver_data** :同上，驱动层可通过dev_set/get_drvdata函数来获取该成员；
    
- **class** :指向了该设备对应类，开篇我们提到的触摸，鼠标以及键盘等设备，对于计算机而言，他们都具有相同的功能，都归属于输入设备。我们可以在/sys/class目录下对应的类找到该设备，如input、leds、pwm等目录;
    
- **dev** :dev_t类型变量，字符设备章节提及过，它是用于标识设备的设备号，该变量主要用于向/sys目录中导出对应的设备。
    
- **release** :回调函数，当设备被注销时，会调用该函数。如果我们没定义该函数时，移除设备时，会提示“Device ‘xxxx’ does not have a release() function, it is broken and must be fixed”的错误。
    
- **group** :指向struct attribute_group类型的指针，指定该设备的属性；

内核也提供相关的API来注册和注销设备，如所示：
```C
// 注册设备（成功返回 0，失败返回负数）
int device_register(struct device *dev);
// 注销设备
void device_unregister(struct device *dev);
```

#### 驱动

- **驱动**是决定设备能否正常工作的核心。它告诉内核自己能驱动哪些设备，以及如何初始化这些设备。
- 驱动的**所有代码执行入口**不再是 `main` 函数，而是 `probe` 回调函数（这是驱动开发的逻辑核心）。

在内核中，使用device_driver结构体来描述我们的驱动，如下所示：

```C
struct device_driver {
	const char              *name;
	struct bus_type         *bus;

	struct module           *owner;
	const char              *mod_name;      /* used for built-in modules */

	bool suppress_bind_attrs;       /* disables bind/unbind via sysfs */

	const struct of_device_id       *of_match_table;
	const struct acpi_device_id     *acpi_match_table;

	int (*probe) (struct device *dev);
	int (*remove) (struct device *dev);

	const struct attribute_group **groups;
	struct driver_private *p;

};
```

- **name** :指定驱动名称，总线进行匹配时，利用该成员与设备名进行比较；
    
- **bus** :表示该驱动依赖于哪个总线，内核需要保证在驱动执行之前，对应的总线能够正常工作；
    
- **suppress_bind_attrs** :布尔量，用于指定是否通过sysfs导出bind与unbind文件，bind与unbind文件是驱动用于绑定/解绑关联的设备。
    
- **owner** :表示该驱动的拥有者，一般设置为THIS_MODULE；
    
- **of_match_table** :指定该驱动支持的设备类型。当内核使能设备树时，会利用该成员与设备树中的compatible属性进行比较。
    
- **remove** :当设备从操作系统中拔出或者是系统重启时，会调用该回调函数；
    
- **probe** :当驱动以及设备匹配后，会执行该回调函数，对设备进行初始化。通常的代码，都是以main函数开始执行的，但是在内核的驱动代码，都是从probe函数开始的。
    
- **group** :指向struct attribute_group类型的指针，指定该驱动的属性；

注册与注销 API：

```C
// 注册驱动（成功返回 0，失败返回负数）
int driver_register(struct device_driver *drv);
// 注销驱动
void driver_unregister(struct device_driver *drv);
```

>Sysfs 映射：成功注册后，驱动会记录在 `/sys/bus/<bus_name>/drivers` 目录下。

#### 流程

![[Pasted image 20260825165429.png]]

上图是总线关联上设备与驱动之后的数据结构关系图。

![[Pasted image 20260825165516.png]]

大致注册流程如上图：
1. 系统启动之后会调用buses_init函数创建/sys/bus文件目录，这部分系统在开机时已经帮我们准备好了；
2. 接下去就是通过总线注册函数bus_register进行总线注册，注册完总线后在总线的目录下生成devices文件夹和drivers文件夹；
3. 最后分别通过device_register以及driver_register函数注册相对应的设备和驱动。

#### attribute 属性文件

**核心概念**：Linux 通过 `/sys` 目录向用户空间导出设备、驱动、总线的属性和控制接口，这些接口在内核中由 `attribute` 结构体描述，最终表现为 `/sys` 下的文件。用户可通过 `cat`/`echo` 等命令直接读写这些文件，实现运行时控制，避免反复编译内核。

此处略过，具体查看这个网站教程的[第四小节](https://doc.embedfire.com/linux/imx6/driver/zh/latest/linux_driver/linux_device_model.html#)



## 参考文献
1. [韦东山：6000字长文告诉你如何学习嵌入式linux](https://zhuanlan.zhihu.com/p/140412992)
2. [嵌入式Linux课程列表](https://linux.100ask.net/docs/Linux-CourseList/Introduction)
3. [嵌入式Linux驱动开发实战指南——基于i.MX6ULL系列](https://doc.embedfire.com/linux/imx6/driver/zh/latest/linux_driver/module.html#)