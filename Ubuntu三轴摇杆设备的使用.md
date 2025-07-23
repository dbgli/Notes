#摇杆 #HID驱动 #uboot #jstest #joy

## 背景

有些机器人示教器会有一个三轴摇杆，或者叫6D鼠标，可以很方便地手动控制机器人，ROS里也可以用joy包来监听摇杆事件并通过话题发布出去。一般是用标准的游戏手柄，但是这些手柄都是几个两轴摇杆，虽说从轴数的角度看足够了，但是操作体验上就和三轴的不一样，而且为了照顾操作方式，功能切换代码也不能和三轴的通用，正经示教器也不会采用游戏手柄那样的结构。于是我专门买了一个三轴摇杆，但是一开始系统识别不了我的摇杆设备，花了点时间解决了，记录如下。

## 问题追踪

插上设备后`/dev/input`下没有新增js设备，但是`lsusb`有显示，说明这是模拟成PS3手柄了。
```bash
Bus 001 Device 007: ID 054c:0268 Sony Corp. Batoh Device / PlayStation 3 Controller
```

接着用`dmesg`，看样子问题出在索尼驱动上。
```bash
[  174.240908] usb 1-9: new full-speed USB device number 7 using xhci_hcd
[  174.367573] usb 1-9: New USB device found, idVendor=054c, idProduct=0268, bcdDevice= 1.00
[  174.367577] usb 1-9: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  174.367578] usb 1-9: Product: 3D Joystick Keyboard
[  174.367579] usb 1-9: Manufacturer: China Longcctv
[  174.375673] sony 0003:054C:0268.000B: failed to retrieve feature report 0xf2 with the Sixaxis MAC address
[  174.375733] sony 0003:054C:0268.000B: hiddev4,hidraw10: USB HID v81.11 Joystick [China Longcctv 3D Joystick Keyboard] on usb-0000:00:14.0-9/input0
[  174.375735] sony 0003:054C:0268.000B: failed to claim input
```

遇到这问题的也不少，基本都是自制游戏机用山寨索尼手柄的人，这个帖子[Force PS3 clone arcade gamepad to be hid generic](https://unix.stackexchange.com/questions/577549/force-ps3-clone-arcade-gamepad-to-be-hid-generic)给出了线索，发现用hid_generic通用驱动代替hid_sony专用驱动就可以。一开始我像评论里说的那样用blacklist禁用hid_sony模块，意料之中并不行，缺少驱动后不会回退到hid_generic。于是只能用hid驱动的ignore_special_drivers参数忽略专用驱动，这个参数是全局的，颗粒度不高，但也没其它可行的方法了。

需要注意的是，对于一般桌面版系统的Linux内核，hid驱动应该会编译为ko文件，也就是独立的内核模块，可以用帖子里的方法设置参数。但是在嵌入式Linux里，很可能hid等驱动直接编译进内核，也就是内建模块，这样就不能按照同样的方法设置参数了。

## 适用于hid为ko模块

创建modprobe的配置文件
```bash
sudo vim /etc/modprobe.d/hid.conf
```
写入
```
# 三轴摇杆通过模拟成PS3手柄进行通信，
# 但是没有实现完整的手柄功能，导致hid_sony不能识别，
# 因此忽略专用驱动，用hid_generic代替。
options hid ignore_special_drivers=1
```
然后更新initramfs
```bash
# 更新当前内核的initramfs
sudo update-initramfs -u
# 更新所有内核的initramfs
sudo update-initramfs -c -k all
# 更新指定内核的initramfs
sudo update-initramfs -c -k <kernel-version>
```
最后重启系统。

## 适用于hid为内建模块

这种情况多出现在嵌入式Linux中，由于hid编译进了内核，成为内核的一部分，所以不能像ko模块那样动态地加载卸载，修改配置后只能重启内核也就是重启系统生效。但读取modprobe的配置文件时，内核早就加载完成了，这时再修改参数也晚了，所以要在内核由bootloader启动时设置。uboot的启动项参数设置如下，GRUB类似。

打开uEnv.txt，不同开发板的路径名称各有不同
```bash
sudo vim /boot/firmware/ubuntuEnv.txt
```
找到bootargs，在后面加上hid的参数，格式为`<模块名>.<参数名>=<参数值>`，重启即可。
```
# bootargs: Kernel parameters for system boot configuration.
bootargs=root=UUID=03b98908-f811-4442-abd3-4b5e456b0348 rootfstype=ext4 rootwait rw console=ttyS2,1500000 console=tty1 console=ttyFIQ0 cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory swa
paccount=1 systemd.unified_cgroup_hierarchy=0 hid.ignore_special_drivers=1
```

## 测试

这时插入设备，dmesg日志表明由hid_generic接管了。因为我的这个摇杆还支持鼠标模式，所以dmesg中会有两个input，分别对应摇杆模式下的输入事件和鼠标模式下的输入事件。
```bash
[  247.746627] usb 1-9: new full-speed USB device number 7 using xhci_hcd
[  247.873311] usb 1-9: New USB device found, idVendor=054c, idProduct=0268, bcdDevice= 1.00
[  247.873313] usb 1-9: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  247.873314] usb 1-9: Product: 3D Joystick Keyboard
[  247.873315] usb 1-9: Manufacturer: China Longcctv
[  247.875296] input: China Longcctv 3D Joystick Keyboard as /devices/pci0000:00/0000:00:14.0/usb1/1-9/1-9:1.0/0003:054C:0268.000B/input/input25
[  247.875339] input: China Longcctv 3D Joystick Keyboard Mouse as /devices/pci0000:00/0000:00:14.0/usb1/1-9/1-9:1.0/0003:054C:0268.000B/input/input26
[  247.875383] hid-generic 0003:054C:0268.000B: input,hidraw10: USB HID v1.11 Joystick [China Longcctv 3D Joystick Keyboard] on usb-0000:00:14.0-9/input0
```

最后确认下`/dev/input`，确实多了一个js设备，用jstest测试，没有的话先安装
```bash
sudo apt install jstest-gtk
jstest /dev/input/js1
```

再用ROS的joy包测试，用joy_enumerate_devices列出所有摇杆设备，如果摇杆不止一个，还需要用参数指定下设备。
```bash
ros2 run joy joy_enumerate_devices
ros2 run joy joy_node --ros-args -p device_id:=1
```
新开一个终端
```bash
ros2 topic echo /joy
```
