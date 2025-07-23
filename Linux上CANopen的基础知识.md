#Linux #CANopen #EtherCAT

配置虚拟CAN设备
```bash
sudo modprobe vcan  # 加载虚拟CAN驱动模块
sudo ip link add dev vcan0 type vcan  # 添加设备
sudo ip link set vcan0 txqueuelen 1000
sudo ip link set up vcan0  # 启动设备
sudo ip link set down vcan0  # 停止设备
sudo ip link del dev vcan0  # 删除设备
```

对于真实CAN设备，需要加载对应的模块，并且还需要设置比特率
```bash
sudo ip link set can0 up type can bitrate 1000000
```

查看Socket设备，在Linux中，CAN是作为网络设备统一管理的，所以和以太网命令一样
```bash
ifconfig  # 只查看up的设备
ifconfig -a  # 查看所有设备，包括down的
```

CAN每一个标准数据帧为8个字节，需要根据设备手册定义的数据结构自行解析数据，发布数据。

而CANopen在这之上规定了标准的对象字典OD，通过index和subindex来读写数据对象DO，有PDO过程数据对象和SDO服务数据对象，使用方式正好对应ROS的话题通讯和服务通讯。

SDO必须由主站的CAN客户端向从站的服务器端发起请求，效率低，可以在运行前使用，常用来设置参数。PDO是主动发送数据，效率高，常用于运行时实时通讯，分为TPDO和RPDO，分别表示发送和接收的PDO，是站在从站的角度理解发送和接收。

在CANopen之上还有CiA402标准，进一步规定了对象字典的索引布局，常用于伺服驱动器，比如0x6040控制字和0x6041状态字。

## 和EtherCAT对比

EtherCAT如果使用COE的话，也是可以支持CiA402标准，内容一样。

在主站使用上，
- CANopen
	- EDS（Electronic Data Sheet，ini格式）文件定义从站的对象字典，包括索引位置，数据类型，默认值，访问类型。
	- DCF（Device Configuration File，ini格式）文件定义数据对象的具体值配置，包括最小值，最大值，默认值。
- EtherCAT
	- ESI（EtherCAT Slave Information，xml格式）文件相当于EDS文件。
	- ENI（EtherCAT Network Information，xml格式）文件定义整个EtherCAT网络的拓扑结构，设备列表，网络参数等。

在ROS中，CANopen用yaml定义总线拓扑，再加载每个从站的EDS文件。EtherCAT用urdf定义网络结构（相当于ENI），再用yaml定义对象字典（相当于ESI）。不过如果是从站厂家自定义的CAN协议，可能还需要厂家提供的SDK进行通讯。