
---

使用buildroot编译出带有qt5的根文件系统，并测试qt5自带的例子。

# 一、配置buildroot

首先在buildroot根目录下使用`make menuconfig`命令配置qt5软件包：

![[Pasted image 20260726225631.png]]

然后执行`make`进行编译。

# 二、编译例程

首先使用`qmake -v`检查qmake是否正常：

![[Pasted image 20260726233442.png]]

然后随意编译一个例程（以模拟时钟为例）：

1. 首先切换到例子的对应目录：
![[Pasted image 20260726233742.png]]

2. 再对`.pro`文件执行`qmake`，得到makefile：
![[Pasted image 20260726234016.png]]

3. 再执行`make`生成可执行文件：
![[Pasted image 20260726234228.png]]

# 三、拷贝到SD卡并上电测试

先使用`df -h`确认第二分区名称，再将sd卡第二分区中原有的文件全部删除：

![[Pasted image 20260726234746.png]]

将先前buildroot编译完成的rootfs.tar解压到sd卡第二分区中：

![[Pasted image 20260726235127.png]]

最后将之前编译好的qt5例子的可运行文件也复制到rootfs中：

![[Pasted image 20260726235515.png]]
![[Pasted image 20260726235528.png]]

将sd卡插入板子并上电，运行例子查看效果：

``` sh
cd /
ls
./analogclock -platform linuxfb
```

![[Pasted image 20260726235856.png]]
![[699b72101c9fdc1cac9b0a7577e14de6.jpg]]

# 参考文献

1. [荔枝派Zero(全志V3S)运行Qt5程序](https://blog.csdn.net/qq_41839588/article/details/130129792)