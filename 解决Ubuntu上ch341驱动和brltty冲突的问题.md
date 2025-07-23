#Ubuntu #串口 #ch341 #brltty

## 太长不看版

在我电脑上，淘宝爆款的CH340串口转USB模块可以正常识别为ttyUSB，但是有的开发板上自带的串口转USB芯片就没法识别。有的说是驱动太老了要去沁恒官网编译安装新版的驱动，其实不需要，查阅确定了是brltty软件和ch341驱动冲突了。如果没有盲文显示的需求，可以直接卸载brltty。

## 详细版

常用的查看USB串口相关命令：
```bash
# 列出所有连接的USB设备
lsusb

# 列出所有tty设备，ttyS是主板板载串口，ttyUSB是USB转串口
ls -l /dev | grep tty

# 查看ch341驱动的设备信息，包括何时插入，何时断开等事件
sudo dmesg | grep ch341

# 查看当前内核中安装的USB串口驱动
ls /lib/modules/$(uname -r)/kernel/drivers/usb/serial
```

首先插入USB转串口模块后，发现`/dev`下并没有ttyUSB文件，随后用`lsusb`查看USB设备，确实是识别到CH340了：
```bash
Bus 001 Device 019: ID 1a86:7523 QinHeng Electronics CH340 serial converter
```

接着用`dmesg`查看ch341的设备信息如下：
```bash
[ 9078.784240] usbcore: registered new interface driver ch341
[ 9078.784247] usbserial: USB Serial support registered for ch341-uart
[ 9078.784255] ch341 1-9.1:1.0: ch341-uart converter detected
[ 9078.784744] usb 1-9.1: ch341-uart converter now attached to ttyUSB0
[ 9079.416215] usb 1-9.1: usbfs: interface 0 claimed by ch341 while 'brltty' sets config #1
[ 9079.416713] ch341-uart ttyUSB0: ch341-uart converter now disconnected from ttyUSB0
[ 9079.416723] ch341 1-9.1:1.0: device disconnected
```

发现罪魁祸首是brltty，由于brltty软件与ch341驱动在设备接口资源占用上产生冲突，导致ch341串口设备最终断开连接。

> brltty是一个用于盲人和视力受损者的屏幕阅读器和布莱叶盲文显示驱动程序。以下是其详细介绍：
> - **功能特点**
>     - **屏幕阅读**：它可以将屏幕上的文本内容转换为语音或布莱叶盲文输出，帮助盲人或视力不佳的用户获取计算机屏幕上的信息，包括文字、菜单、提示等。
>     - **布莱叶盲文显示支持**：支持多种布莱叶盲文显示器，能将文本实时转换为盲文显示在连接的盲文设备上，方便用户通过触摸盲文点来读取信息。
>     - **输入辅助**：提供了一些辅助输入功能，例如可以通过盲文键盘输入文本，或者对普通键盘的输入进行特殊处理，以适应视力受损用户的操作习惯。
>     - **应用程序支持广泛**：能够与多种操作系统和应用程序协同工作，包括常见的办公软件、浏览器、邮件客户端等，使得视力受损用户可以像普通用户一样使用各种计算机应用。
> - **工作原理**：brltty通过与操作系统的交互，拦截屏幕上的文本信息，并根据用户的设置将其转换为语音或盲文输出。对于盲文显示，它会根据盲文显示器的类型和协议，将文本内容编码为相应的盲文信号发送给设备。同时，它也会监听输入设备（如盲文键盘或普通键盘）的输入事件，实现用户与计算机的交互。

\- 豆包

很明显我是不需要brltty的，用`sudo apt remove brltty`卸载之，然后重新插入USB转串口模块即可。或者可以参考这篇文章《[解决Ubuntu和Raspberry Pi上brltty与USB串行设备冲突的问题](https://makeronsite.com/brltty_confilicts.html)》，比较温和地配置brltty以避免错误占用。

另外，沁恒的CH340和CH341芯片都是用ch341驱动程序，所以上述操作都是查看ch341而不是ch340。