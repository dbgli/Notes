#IgH #EtherCAT #实时内核

按照ICube团队的文档[https://icube-robotics.github.io/ethercat_driver_ros2/quickstart/installation.html](https://icube-robotics.github.io/ethercat_driver_ros2/quickstart/installation.html)一步步来，在执行`./configure --prefix=/usr/local/etherlab --disable-8139too --disable-eoe --enable-generic`时报错`configure: error: Failed to find Linux sources. Use --with-linux-dir!`。

原因是没有找到Linux头文件，如果是官方内核，从apt安装对应的linux-headers，如果是自己编译的内核，记得编译完要保留内核的build文件夹。

再来一遍就能找到`checking for Linux kernel sources... /usr/src/linux-6.9 (Kernel 6.9)`这样的输出。

ICube文档里用rosdep安装依赖，命令如下`rosdep install --ignore-src --from-paths . -y -r`。但是众所周知几乎没法直接更新rosdep，我不想浪费功夫在这上面，于是直接看每个功能包的package.xml文件，手动安装依赖。所幸的是要安装的并不多，就一个hardware_interface，查到这是隶属于ros2_control的，然后看文档用apt安装ros-jazzy-ros2-control和ros-jazzy-ros2-controllers这两个包就行了，反正早晚都是要装的，最后回到正轨继续用colcon编译ethercat_driver_ros2。