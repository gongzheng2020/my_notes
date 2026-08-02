
---

全志v3s内部集成了mac与phy，因此只需要一个带有网络变压器的RJ45网口即可使用以太网。

硬件电路如下：

![[Pasted image 20260801193522.png]]

接下来是软件部分，按照如下步骤启用以太网：

1. 修改设备树文件

向设备树文件中添加相关设备描述：

```
&emac {
	phy-handle = <&int_mii_phy>;
	phy-mode = "mii";
	allwinner,leds-active-low;
	status = "okay";
};
```

2. 配置Linux内核

执行`make menuconfig`命令，按照下图配置内核：

![[Pasted image 20260801195018.png]]

3. 编译内核与设备树

``` sh
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j6
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=out modules_install
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs
```

将编译生成的zImage与dtb文件复制到sd卡中

开机后执行`ifconfig eth0 up`命令启用以太网：

![[Pasted image 20260801200028.png]]

# 参考文献

1. [【全志V3S开发】（五）-Kernel适配以太网以及nfs、tftp服务开启](https://blog.csdn.net/Jlinkneeder/article/details/141420955)