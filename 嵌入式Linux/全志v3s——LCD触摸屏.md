
---

# 设置屏幕打印终端信息

根据自己的需求，按照如下几个选项修改boot.cmd文件：

1. 默认串口显示
`setenv bootargs console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw`

2. 在LCD显示
`setenv bootargs console=tty0,115200 root=/dev/mmcblk1p2 rootwait rw`

3. 在LCD和串口同时显示
`setenv bootargs console=tty0 console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw`
>第一次设置tty0 作为console控制台（LCD屏幕），第二次设置tttymxc0作为控制台（串口），两个可以同时显示终端，但此时LCD屏幕终端还不能交互。
>可以通过修改/etc/inittab文件，添加一行`tty0::askfirst:-/bin/sh`，以支持交互：
![[Pasted image 20260730202457.png]]

# ns2009触摸芯片

首先需要配置设备树以添加ns2009，编译后替换原有设备树文件dtb：

```
&i2c0 {
	status = "okay";
	ns2009: ns2009@48 {
		compatible = "nsiway,ns2009";
		reg = <0x48>;
	};
};
```

修改Linux内核配置以添加ns2009驱动，编译后替换原有zImage：

![[Pasted image 20260731172332.png]]

进入系统后可以使用`i2cdetect -y 1`检查设备是否存在：

![[Pasted image 20260731212421.png]]

使用`cat /proc/bus/input/devices`命令查看是否产生输入子系统设备：

![[Pasted image 20260731212505.png]]

修改buildroot，添加tslib库，编译后替换原有根文件系统rootfs：

![[Pasted image 20260731172804.png]]

在根文件系统/etc/profile的最后一行插入，如下代码：

``` sh
export TSLIB_TSDEVICE=/dev/input/event0
export TSLIB_CALIBFILE=/etc/pointercal
export TSLIB_CONFFILE=/etc/ts.conf
export TSLIB_PLUGINDIR=/usr/lib/ts
export TSLIB_CONSOLEDEVICE=none
export TSLIB_FBDEVICE=/dev/fb0
```

![[Pasted image 20260731214152.png]]

执行`ts_calibrate`对触摸屏进行校准：

![[0fd4f12bd7931be2999f21e5e1a20005.jpg]]

>出现了点不到屏幕的问题！！！

测试触摸屏，在开发板命令终端分别输入：`ts_print`、`ts_test`等命令，会在屏幕上或者命令终端看到不同的效果。

# 参考文献

1. [嵌入式Linux | 使Linux的启动信息显示到LCD上面&设置LCD屏幕为终端控制台](https://blog.csdn.net/qq_39400113/article/details/128424290)
2. [10、LCPI(F1C200S)驱动电阻屏触摸芯片ns2009(tsc2007)](https://blog.csdn.net/GJF712/article/details/126720236)