
---

全志v3s内部自带一个codec声卡，可以使用它来进行音频相关操作。

1. 首先确认设备树中是否启用了codec

```
&codec {
	allwinner,audio-routing =
	"Headphone", "HP",
	"Headphone", "HPCOM",
	"MIC1", "Mic",
	"Mic", "HBIAS";
	status = "okay";
};
```

![[Pasted image 20260801134930.png]]

内核启动时可以看到声卡已经被正常识别：

![[Pasted image 20260801134852.png]]

使用`ls /dev/snd`命令查看具体设备信息：

![[Pasted image 20260801135227.png]]

>- controlC0表示控制器，C0就是声卡0。  
>- pcmC0D0c 表示capture，是用于录音的pcm设备。  
>- pcmC0D0p 表示playback，是用于放音的pcm设备。  

2. 在Buildroot中启用alsa

![[Pasted image 20260801135716.png]]

再将alsa-utils内的所有软件全部勾选。保存后重新编译，烧录到sd卡

3. 测试

使用`arecord -l`命令查看声卡设备：

![[Pasted image 20260801151317.png]]

其余命令可以看[这篇博客](https://developer.aliyun.com/article/1268429)

>可以播放，但录音失败！！！

# 参考文献

1. [全志V3S开发】（七）-CODEC音频播放](https://blog.csdn.net/Jlinkneeder/article/details/141606736)
2. [荔枝派Zero(全志V3S)开启alsa，测试codec](https://developer.aliyun.com/article/1268429)