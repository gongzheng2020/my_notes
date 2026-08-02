
---

[[嵌入式Linux——基础#简介| 嵌入式Linux系统 = bootloader + linux内核 + 根文件系统]]

# 前置工作：交叉编译环境配置+SD卡分区

## [[创建交叉编译环境#1、包管理器下载 | 交叉编译环境配置]]：

首先下载并解压[工具链](https://developer.arm.com/-/cdn-downloads/permalink/legacy-linaro-gnu-toolchains/4.9-2017.01/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf.tar.xz)：

``` sh
wget https://developer.arm.com/-/cdn-downloads/permalink/legacy-linaro-gnu-toolchains/4.9-2017.01/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf.tar.xz
sudo tar -vxf gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf.tar.xz
```

接着设置环境变量：

``` sh
1、打开编辑~/.bashrc 文件
sudo vim ~/.bashrc
2、在最底部添加以下内容
export PATH=$PATH:/home/gz/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin
3、使环境变量立即生效
source ~/.bashrc
```

安装其他必要的库：

``` sh
sudo apt-get install lsb-core lib32stdc++6
```

使用`arm-linux-gnueabihf-gcc -v`命令查看是否安装成功：

![[Pasted image 20260725005344.png]]

>注意这里一定不能使用apt方式下载的工具链，否则buildroot会报错。

## SD卡分区

分两个区，第一分区存储启动配置与内核相关文件（zImage, dtb, boot.scr），第二分区存储根文件系统（rootfs）。

>需要注意的是，第一分区并不是从SD卡的零地址开始的，而是有一个偏移，用于存放BootLoader（U-Boot）。

使用`ls /dev/sd*`查看sd卡插拔前后的设备号变化：

![[Pasted image 20260723004834.png]]

>我这里的sd卡是/dev/sda，如果有sda1、sda2等等，则表示该sd卡已经存在分区，需要使用umount将其全部卸载：`sudo umount /dev/sda1`

使用fdisk命令进行分区操作：

``` sh
sudo fdisk /dev/sda   	# 进行分区操作
##### 操作步骤如下 #####
# 若已存分区即按 d 删除各个分区
# 通过 n 新建分区，第一分区暂且申请为32M，剩下的空间都给第二分区
	# 第一分区操作：n p 1 2048 +32M
		# p 主分区、默认 1 分区、地址偏移默认2048（1MB，存放U-Boot）、+32M
	# 第二分区操作：n 后面全部回车默认即可
		# p 主分区、默认 2 分区、默认、默认剩下的全部空间
# p 查询分区表确定是否分区成功
# w 保存写入并退出
########################
```

![[Pasted image 20260724021825.png]]

![[Pasted image 20260724021841.png]]

最后进行将两个分区进行格式化：

```
sudo mkfs.vfat /dev/sda1 # 将第一分区格式化成FAT
sudo mkfs.ext4 /dev/sda2 # 将第二分区格式化成EXT4
```

>- EXT4：只用于Linux系统的内部磁盘
>- NTFS：与Windows共用的磁盘
>- FAT：所有系统和设备共用的磁盘

# 1、BootLoader

## 编译

下载定制后的UBoot源码：
`git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-current`

![[Pasted image 20260723000922.png]]

>注意这里有个错误，路径需要纯英文，否则编译可能会出错

编译源码：
``` sh
cd u-boot
//不带屏幕
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_defconfig 
//带4.3寸屏幕
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_480x272LCD_defconfig
//配置和编译
make ARCH=arm menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
```

一些错误及解决方法：
1. Your dtc is too old, please upgrade to dtc 1.4 or newer
`sudo apt install device-tree-compiler`

2. Missing parentheses in call to 'print'.
![[Pasted image 20260723003012.png]]
这个错误是因为 binman 脚本使用了 Python 2 的 print 语法，但你的系统默认 Python 是 Python 3，导致语法错误。
	``` sh
	sudo apt install python2
	sed -i '1s/python/python2/' tools/binman/binman
	```

## 烧录

系统镜像 = boot区 + linux内核区 + 根文件区（rootfs），这里先烧录boot区。

### a. 通过 sunxi-fel 工具烧录

参考[这篇博客](https://blog.csdn.net/p1279030826/article/details/112672535)

### b. 烧录到SD卡

使用`ls /dev/sd*`确认sd卡设备号：

![[Pasted image 20260724022701.png]]

接着将编译后的bin文件（`u-boot-sunxi-with-spl.bin`在uboot根目录下）烧录至sd卡：

``` sh
# 写入 u-boot 文件 8KB 位置
sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/sda bs=1024 seek=8
```

![[Pasted image 20260724022927.png]]

## 最终效果

![[fe2be46fd1f0804951aa90077d6c129f.jpg]]

# 2、编译Linux内核

## 编译

下载源码：`git clone -b zero-4.13.y https://github.com/Lichee-Pi/linux.git`

![[Pasted image 20260724004130.png]]

编译源码（配置文件在`arch/arm/configs`文件夹内）：

``` sh
cd linux
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- licheepi_zero_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j6
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=out modules_install
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs
```

编译完成后，zImage在arch/arm/boot/下。

一些错误：
1. /bin/sh: 1: flex: not found
`sudo apt install flex bison`

## 烧录

需要准备：boot.scr、zImage、设备树文件（dtb）。后面两个文件都有（设备树文件在arch/arm/boot/dts/下），我们需要额外准备boot.scr文件。

### 准备boot.src文件

boot.src 用来让 U-Boot 自动执行一系列预设的命令，从而加载内核、设备树（dtb）并启动系统，实现一键开机启动。

具体过程可以参考[这篇博客](https://blog.csdn.net/p1279030826/article/details/114135757)。

首先在UBoot根目录新建boot.cmd文件，写入如下内容：

``` sh
# 屏幕不显示
setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10 earlyprintk rw
load mmc 0:1 0x41000000 zImage
load mmc 0:1 0x41800000 sun8i-v3s-licheepi-zero.dtb
bootz 0x41000000 - 0x41800000

# 屏幕显示
setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10 earlyprintk rw
setenv video-mode sunxi:480x272-18@60,monitor=lcd
setenv lcd-mode x:480,y:272,depth:18,pclk_khz:10000,le:42,ri:8,up:11,lo:4,hs:1,vs:1,sync:3,vmode:0
setenv stderr serial,lcd
setenv stdout serial,lcd
load mmc 0:1 0x41000000 zImage
load mmc 0:1 0x41800000 sun8i-v3s-licheepi-zero-with-480x272-lcd.dtb
bootz 0x41000000 - 0x41800000
```

然后将其转换成boot.src文件：

``` sh
./tools/mkimage -C none -A arm -T script -d boot.cmd boot.scr
```

![[Pasted image 20260724013042.png]]

转换完成的boot.scr文件在uboot根目录中。

### 进行烧录

将下面三个文件一起放到SD卡的第一分区：

- boot.scr
- 设备树sun8i-v3s-licheepi-zero-with-480x272-lcd.dtb
- 刚生成的zImage

使用`df -h`查询挂载名:

![[Pasted image 20260724212037.png]]

将前述三个文件复制到第一分区（32M卷，图中为`/media/gz/5DB2-4139`）：

``` sh
cp zImage boot.scr sun8i-v3s-licheepi-zero-with-480x272-lcd.dtb /media/gz/5DB2-4139
```

![[Pasted image 20260724212440.png]]

## 最终效果

由于没有烧录根文件系统（rootfs），因此内核会崩溃并不断重启。

putty终端打印信息：

![[Pasted image 20260724213143.png]]

屏幕信息：

![[20260724_213505(1)(1).gif]]

# 3、编译根文件系统

## 编译

获取buildroot：

``` sh
wget https://buildroot.org/downloads/buildroot-2019.08.tar.gz
tar xvf buildroot-2019.08.tar.gz&&cd buildroot-2019.08/
make menuconfig
```

修改基本配置：

![[Pasted image 20260724230432.png]]

使用`which arm-linux-gnueabihf-gcc`进行查询编译工具链位置：

![[Pasted image 20260725005423.png]]

>注意，因为Buildroot 会自动在路径后面加上 `/bin/` 和工具前缀(prefix)，因此根据上图：
>1. 需要将Toolchain path设置为`/home/gz/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf`。
>2. 再将Toolchain prefix设置为`arm-linux-gnueabihf`

使用`arm-linux-gnueabihf-gcc -v`查询编译工具链版本：

![[Pasted image 20260725010139.png]]

查询内核版本号：

``` sh
cat /home/gz/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/libc/usr/include/linux/version.h | grep LINUX_VERSION_CODE
```

![[Pasted image 20260725005650.png]]

>十进制262144转二进制得到0x40000，内核版本选择4.0.x

根据以上三个信息及自己所需配置修改编译工具链配置：

![[Pasted image 20260725010604.png]]

除此之外的其他配置可以按照自己的需求进行修改，建议参考[这篇博客](https://blog.csdn.net/qq_28877125/article/details/130652435)，在此我只简单配置主机名与开机提示。

![[Pasted image 20260724233624.png]]

配置完成后记得先save再exit，最后使用`make`命令进行编译：

![[Pasted image 20260725014230.png]]

>如果中途出错，要先`make clean`再重新`make`。

最终得到的压缩根文件系统位于：

![[Pasted image 20260725014620.png]]

一些遇到的问题：

1. LD_LIBRARY_PATH environment variable. This doesn't work.
![[Pasted image 20260724234648.png]]
临时取消环境变量：`unset LD_LIBRARY_PATH`

2. UnicodeEncodeError: 'utf-8' codec can't encode character '\udce6' in position 22: surrogates not allowed
当前路径有中文，切换到纯英文路径即可。

## 烧录

查看SD卡第二分区名称`df -h`：

![[Pasted image 20260725014404.png]]

解压根文件（rootfs.tar）至SD卡第二分区：

``` sh
sudo tar xvf output/images/rootfs.tar -C /media/gz/b94942c5-13c5-491f-abf9-bbea0ee90327
```

![[Pasted image 20260725014840.png]]

# 参考文档

1. [全志v3s学习笔记（2）——u-boot编译与烧录](https://blog.csdn.net/p1279030826/article/details/112672535)
2. [全志v3s学习笔记（5）——主线Linux编译与烧录](https://blog.csdn.net/p1279030826/article/details/113483775)
3. [全志v3s学习笔记（4）——u-boot传参(boot.scr)和参数配置(script.bin)文件](https://blog.csdn.net/p1279030826/article/details/114135757)
4. [全志v3s学习笔记（7）——Buildroot 根文件系统构建](https://blog.csdn.net/p1279030826/article/details/114500777)