#校园网 #WiFi #DNS #热点 #开发板

鲁班猫5开发板接了RTL8852BE的无线网卡，也用nmtui成功连接了热点，但是死活ping不通外网，只能在WiFi的局域网里ping通其它设备。一开始以为是DNS的问题，费了功夫设置了全局DNS，但还是不行，于是改用手机开数据开热点，这才能连上外网。原来是校园网限制设备数，开热点中转也不行，校园网误我啊。

不过也摸清楚了常用的网络管理工具：
nmtui：TUI版的nmcli，非常友好。
nmcli：`sudo nmcli dev wifi list`列出可用的WiFi设备，要等一会扫描完才显示，第一次还以为卡住了。
ifconfig：不用说了。
iwconfig：专攻无线，可以显示WiFi的标准，名称，频段，速率等信息。
systemd-resolved：Ubuntu默认的DNS管理工具，Systemd全家桶，见[[Ubuntu systemd-resolved的DNS设置]]。
curl：常用来下文件，也可以网络测试用`curl -I www.baidu.com`。