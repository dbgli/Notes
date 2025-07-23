#设备树 #嵌入式 #Linux

在一些开发板提供的教程中会把设备树和内核一起编译，编译内核的同时顺便编译了设备树，但其实也可以单独编译设备树文件。

一些缩写解释：

| 缩写     | 英文                         | 中文       | 用途                                   |
| ------ | -------------------------- | -------- | ------------------------------------ |
| *.dts  | Device Tree Source         | 设备树源文件   | 定义设备树的文本格式                           |
| *.dtsi | Device Tree Source Include | 设备树头文件   | 类似于C语言的头文件，共用相同的部分                   |
| *.dtb  | Device Tree Blob           | 设备树二进制文件 | 编译后的二进制格式，供内核使用                      |
| *.dtbo | Device Tree Blob Overlay   | 设备树叠加层文件 | 区别于主dtb，可以动态加载替换而不用重新编译整个设备树，也叫设备树插件 |
| dtc    | Device Tree Compiler       | 设备树编译器   | 可以编译和反编译设备树文件                        |

dts要引入dtsi可以用`/include/`指令，但是不能像C语言那样使用宏定义，为了方便编写，干脆直接引入C语言预处理器的语法，用`#include`包含dtsi头文件，甚至是h头文件，但是这里的h文件应该只能有宏定义。这样流程就变成了先用CPP（C Preprocessor）展开预处理指令，再将展开合并后的单个dts文件交给dtc编译。

先将所有需要包含的文件放在一起，然后执行cpp命令
```bash
# -nostdinc 不搜索标准包含目录
# -I. 指定当前目录为包含目录
# -undef 不预定义系统和gcc特定的宏
# -x assembler-with-cpp 指定语言为C，C++，Objective-C
cpp -nostdinc -I. -undef -x assembler-with-cpp device.dts > device.tmp.dts
```

再用dtc编译dts，编译为dtbo只需要修改输出文件名就行
```bash
dtc -I dts -O dtb -o device.dtb device.tmp.dts
dtc -I dts -O dtb -o device.dtbo device.tmp.dts
```

也可以反编译dtb或dtbo文件
```bash
dtc -I dtb -O dts -o device.dts device.dtb
```
