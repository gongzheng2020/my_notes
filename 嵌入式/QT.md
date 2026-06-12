
## 一、🔍 核心概念

Qt：C++跨平台应用框架，核心优势 = 对象模型 + 元对象系统(MOC) + 事件驱动架构
• 架构分层：Core(基础) → GUI/Widgets(传统) → Quick/QML(声明式/硬件加速)
• 嵌入式定位：ARM Linux HMI、工业屏、车载中控、调试上位机
• 关键特性：信号槽机制、跨平台抽象(QPA)、C++11/14/17兼容、CMake/qmake构建

✅ **面试金句**：
> "Qt通过MOC元对象系统实现运行时反射与信号槽，事件循环驱动GUI与异步通信；在嵌入式场景中需重点关注线程安全、资源占用与QPA平台适配。"

## 二、⚙️ 四大核心机制

|机制|核心原理|线程安全|标准用法|⚠️ 避坑指南|
|---|---|---|---|---|
|**信号与槽**|MOC预处理生成元数据，运行时动态绑定|同线程直连(Direct)；跨线程自动队列化(Queued)|`connect(sender, &Sender::sig, receiver, &Receiver::slot)`|避免槽函数阻塞；Qt5.12后必须用函数指针语法，禁止`SIGNAL()`宏|
|**事件循环**|`QApplication::exec()` 持续分发 `QEvent`|仅创建对象的主线程安全|GUI刷新/网络/定时器/信号槽驱动|耗时操作放子线程，否则界面卡死；勿在槽中调用`processEvents()`|
|**多线程**|`QThread`是线程管理器，非线程本体|对象归属创建线程，槽在归属线程执行|`worker->moveToThread(&thread)` + 信号槽控制启停|❌ 勿继承`QThread`重写`run()`（脱离事件循环，丧失跨线程能力）|
|**内存管理**|父子对象树自动析构 + 智能指针|父对象析构时自动delete子对象|`new QWidget(this)` / `QScopedPointer`|避免循环引用；跨线程对象勿手动`delete`，用`deleteLater()`|

## 三、📦 嵌入式专项要点（部署/性能/资源）

|维度|嵌入式常见方案|面试关注点|
|---|---|---|
|**显示后端**|`EGLFS`（无X11，直接OpenGL ES）、`LinuxFB`（纯Framebuffer）、`Wayland`|问清项目是否跑X11；EGLFS需正确配置`qt.conf`与GPU驱动|
|**交叉编译**|Qt5: `qmake -platform linux-aarch64-gnu-g++`  <br>Qt6: `cmake -DCMAKE_TOOLCHAIN_FILE=xxx.cmake`|能口述工具链配置路径、sysroot挂载、依赖库裁剪|
|**性能优化**|QML硬件加速、减少信号发射频率、用`QVector`/`QByteArray`替代`std::vector`|帧率低？查是否主线程阻塞/频繁重绘/未开GPU加速|
|**资源受限**|静态编译裁剪模块、关闭调试符号、字体/插件按需加载|镜像>50MB？查是否打包了无用模块（如WebEngine/Print）|
|**部署打包**|`linuxdeployqt` 自动收集依赖、`rpath` 设置、`QT_PLUGIN_PATH` 配置|运行报`libQt5Core.so not found`？查`ldconfig`或`LD_LIBRARY_PATH`|

✅ **调试口诀**：
> `卡顿看线程 → 崩溃看指针 → 黑屏看QPA → 大体积看依赖 → 乱码看字体/编码`