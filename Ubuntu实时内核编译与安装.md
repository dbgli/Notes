#RT_Linux #实时内核 #PREEMPT_RT #Ubuntu

因为要用到Ethercat通讯，需要一个实时系统，所以选择PREEMPT_RT补丁自己编译实时内核。关于其它的实时Linux方案和Ubuntu Pro实时内核暂且不谈。

首先下载Linux内核与实时补丁，注意两者版本要一致。直接下网速慢，各显神通吧。
内核：[https://mirrors.edge.kernel.org/pub/linux/kernel/](https://mirrors.edge.kernel.org/pub/linux/kernel/)
补丁：[https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/)

用`uname -a`查看当前内核版本，自己编译的内核版本最好不要相差太多，方便后面抄当前内核的编译配置。目前我的是6.8.0，编译的是6.9.0。

按照网上教程安装一些必要的软件。
```bash
sudo apt install autoconf automake libtool make libncurses-dev flex bison libelf-dev libssl-dev zstd net-tools
```

下载解压后，cd到内核源代码文件夹打补丁。patch文件应该在上一层目录中，能找到就行。
```bash
patch -p1 < ../patch-6.9-rt5.patch
```

编译不难，主要是配置。有的教程说要手动复制当前内核的配置文件，其实新版本不需要，自动识别了。
```bash
make menuconfig
```

进入到TUI配置界面后，有以下几个部分要修改，有些选项的默认值已经符合的话就不用改了。网上几乎所有教程都是用menuconfig配置后再在.config文件里手动修改配置，完全可以用menuconfig全部配置好，用`/`搜索配置项。我一开始就是按教程手动修改.config禁用调试信息，结果编译时又给我改回去了，估计配置冲突了吧，坑！
- 抢占模式
  - General setup > Preemption Model设置为Fully Preemptible Kernel (Real-Time)

- 定时器
  - General setup > Timers subsystem > Timer tick handling设置为Full dynticks system (tickless)
  - General setup > Timers subsystem中启用High Resolution Timer Support
  - Processor type and features > Timer frequency设置为1000 HZ

- 证书与签名
  - Cryptographic API > Certificates for signature checking中，删除File name or PKCS#11 URI of module signing key，Additional X.509 keys for default system keyring和X.509 certificates to be preloaded into the system blacklist keyring这三个的证书
  - Enable loadable module support > Module signature verification禁用Automatically sign all modules

- 驱动
  - Device Drivers里取消Staging drivers

- 调试信息
  - Kernel hacking > Compile-time checks and compiler options > Debug information设置为Disable debug information

配置后开始编译内核，用`nproc`命令查看电脑可用的CPU核心数（逻辑核心数），make时用`-j`参数指定编译线程数，我的电脑是20。大概10到20分钟编译完成。
```bash
make -j20
sudo make modules_install -j20
sudo make install
```

很多教程说安装后执行`update-grub`，新版本也不需要了，`make install`包含了更新grub命令，看输出结果自行判断。

检查`/boot`下是否生成了对应的内核，应该多了四个文件：config-6.9.0-rt5、initrd.img-6.9.0-rt5、System.map-6.9.0-rt5、vmlinuz-6.9.0-rt5。另外还有两个符号链接文件initrd.img和vmlinuz进行了更新，指向最新编译的内核版本，同时将旧的符号链接文件备份为对应的.old文件，可以通过`ls -l`命令查看符号链接的对象。

检查`/lib/modules`下是否生成了对应的内核模块，应该多了一个6.9.0-rt5文件夹。里面的build文件夹也是符号链接，指向我解压内核源码的目录`~/Downloads/linux-6.9`。一般源码是要放在`/usr/src`下的，~~不过后续也不需要再开发内核了，build文件夹删了也可以~~。（别删！后面编译IgH EtherCAT Master还要用到这里面的源文件，不然又要重新编译内核了！）

我第一次编译带着调试信息，生成的内核非常大，第二次没有就小了很多。build文件夹从26GB降到5GB，initrd.img从600MB降到66MB，内核模块从2GB降到130MB。

最后重启电脑，在grub里进高级选项，选择刚刚编译的实时内核启动。再用`uname -a`查看当前内核版本，出现PREEMPT_RT就行（原来的是PREEMPT_DYNAMIC）。可以下载rt-tests进行实时性测试，下面的指令表示以80的优先级运行10个线程，第一个线程每1000微秒调用一次，后面每个线程的间隔时间依次增加500微秒。
```bash
sudo cyclictest -p 80 -t 10 -i 1000
```

空载测试结果为：最小延迟1到2微秒，最大延迟10到60微秒，平均延迟2到3微秒。在这个量级的话基本满足实时性要求。在非实时内核上测试，最大延迟200微秒左右。

因为没有打包成deb并通过包管理器安装内核，所以如果要卸载的话，只能先切换到别的内核，然后手动删除之前编译生成的文件（四个内核文件，两个符号链接文件，一个内核模块文件夹），最后更新下grub。打包成deb是另外的话题了。