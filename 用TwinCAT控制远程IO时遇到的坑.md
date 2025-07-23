#EtherCAT #TwinCAT #远程IO #网卡

在用TwinCAT3测试EtherCAT的远程IO时，一直扫描不到设备，找出来的只有EtherCAT Automation Protocol设备，而不是EtherCAT设备。手动添加EtherCAT Master项再手动添加远程IO设备后，也无法正常使用。

和厂家确认后得知他们测试过I210、I211、I219-V，我的迷你主机的网卡是I225-V，于是打算更换网卡。临时解决办法是在电脑和远程IO之间串一个伺服或其它能正常通讯的EtherCAT设备，这样远程IO和电脑就不是直连，能够扫描和使用了。