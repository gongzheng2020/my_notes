
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

## 应用



## 驱动



## 参考文献
1. [韦东山：6000字长文告诉你如何学习嵌入式linux](https://zhuanlan.zhihu.com/p/140412992)
2. [嵌入式Linux课程列表](https://linux.100ask.net/docs/Linux-CourseList/Introduction)
3. [嵌入式Linux驱动开发实战指南——基于i.MX6ULL系列](https://doc.embedfire.com/linux/imx6/driver/zh/latest/linux_driver/module.html#)