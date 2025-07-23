#Ubuntu #串口 #dialout #udev

在Ubuntu上使用串口通信时，不像Windows那样直接连接串口就能用了，默认情况下，我们的普通用户是没有权限访问`/dev/ttyS`和`/dev/ttyUSB`文件的。既然是权限问题，总结了以下三种方法：

## 1. 用chmod修改串口设备文件

Linux上一切皆文件，串口设备也不例外。于是可以用`sudo chmod 666 /dev/ttyUSB0`来赋予该串口读写权限。优点是方便快捷，缺点是只要电脑重启甚至重新插拔设备就会删除旧文件创建新文件，导致权限失效。所以只适合临时调试使用。

## 2. 将用户加入到dialout用户组

Ubuntu默认将串口设备的所有权分配给dialout组，专门用于管理串行通信设备的权限。用`ls -l /dev | grep tty`命令可以验证：
```bash
crw-rw----   1 root  dialout   4,  64 May  3 09:22 ttyS0
crw-rw----   1 root  dialout 188,   0 May  3 17:05 ttyUSB0
```

使用`sudo usermod -aG dialout $USER`命令将当前用户追加到dialout组中，参数`a`表示追加而不是直接覆盖，避免移除用户所属的其它组，参数`G`表示目标用户组。

重启系统就可以生效了，或者不想重启的话也可以用`newgrp dialout`在当前终端中新开一个shell子进程，在这个进程中用户的初始组就更改为dialout组了。每一次使用newgrp切换用户的初始组，用户都会切换到一个新的子shell中，可以通过`exit`命令不断退回到父进程中，相当于临时修改。

不管怎样，现在当前用户已经属于dialout组成员了，可以用下面命令来确认：
```bash
# 查看当前用户属于哪些用户组
groups $USER

# 查看dialout组中有哪些成员
getent group dialout
```

## 3. 自定义udev规则

还有一种高级方法是自定义udev规则，在`/etc/udev/rules.d/`下新建`70-ttyusb.rules`规则文件。不同规则首先按照数字从小到大执行，数字相同再按照后面的字母顺序执行，后面执行的规则会覆盖前面的规则。所以可以把文件名前面的数字理解为优先级，数字越大优先级越高，可以根据需要自行决定优先级。然后在规则文件中编写需要自定义的设备读写权限。

不过这种方法并没有深究，因为前面两种方法已经足够完成任务了，只有在配置IgH EtherCAT Master时接触过，见[[编译IgH EtherCAT Master时配置报错的解决方案]]。