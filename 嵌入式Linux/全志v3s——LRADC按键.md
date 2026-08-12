
---

LRADC全称“低分辨率模拟数字转换器”，就是分辨率较低的ADC，可以通过电阻分压来识别按键，单引脚可以外接多个按键，节省引脚资源。

按键电路如下：

![[Pasted image 20260801172030.png]]

下面我们对其驱动、测试：

1. 添加设备树与头文件

```
#include "dt-bindings/input/input.h"

&lradc {
        vref-supply = <&reg_vcc3v0>;
        status = "okay";

        button@200 {
                label = "key1"; 
                linux,code = <KEY_1>;
                channel = <0>;
                voltage = <200000>;
        };

        button@400 {
                label = "key2"; 
                linux,code = <KEY_2>;
                channel = <0>;
                voltage = <400000>;
        };

        button@600 {
                label = "key3"; 
                linux,code = <KEY_3>;
                channel = <0>;
                voltage = <600000>;
        };

        button@800 {
                label = "key4"; 
                linux,code = <KEY_4>;
                channel = <0>;
                voltage = <800000>;
        };
};
```

2. 修改Linux内核设置

使用`make menuconfig`，按图修改配置后保存退出:

![[Pasted image 20260801173439.png]]

3. 编译内核与设备树

``` sh
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j6
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=out modules_install
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs
```

将编译生成的zImage与dtb文件复制到sd卡中

4. 测试按键

执行`hexdump /dev/input/event0`后，分别按下各个按键查看终端输出：

![[Pasted image 20260801184225.png]]

# 参考文献

1. [【全志V3S开发】（十）-LRADC按键](https://blog.csdn.net/Jlinkneeder/article/details/141995998)