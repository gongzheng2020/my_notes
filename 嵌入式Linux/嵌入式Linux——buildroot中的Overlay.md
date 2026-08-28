

---

在嵌入式开发中需要经常修改rootfs，这就大概率会涉及到buildroot的重配置与编译，这就会造成我们手动修改的部分rootfs中的内容丢失。
据此，我们可以使用buildroot中的Overlay机制去保存我们的一些个人文件。

Overlay 是 Buildroot 官方推荐的持久化定制方式：通过创建一个与根文件系统结构一致的 “覆盖目录”，编译时 Buildroot 会自动将该目录中的文件复制到 output/target/ 中（覆盖原有文件或新增文件），且修改不会因 make clean 丢失。

具体步骤：

1. 创建 Overlay 目录结构:
在 Buildroot 项目中创建一个 Overlay 目录（路径自定义，通常放在 `board/<厂商>/<板子名>/overlay/` 下，便于管理），并按根文件系统的结构组织子目录。例如：
	``` sh
	# 假设板子名为“licheepi”，创建Overlay目录
	mkdir -p board/licheepi/overlay/
	# 创建对应需要添加的目录与文件
	mkdir -p board/licheepi/overlay/etc
	mkdir -p board/licheepi/overlay/lib
	```
	![[Pasted image 20260731200228.png]]

2. 配置buildroot以启用Overlay
![[Pasted image 20260731202728.png]]
>注意此处需要填写相对于output的路径，而不是绝对路径。

3. 重新执行编译查看`output/target/`目录确认文件已添加 / 覆盖。

# 参考文献

1.[关于buildroot文件系统中rootfs的内容，该怎么增删（瑞芯微rv1126b）](https://www.cnblogs.com/clnchanpin/p/19252983)