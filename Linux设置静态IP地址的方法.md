#Linux #IP #局域网 #命令行

图形化环境就不用说了，主要是命令行方法。

## ifconfig

ifconfig可以临时设置，立即生效，但重启失效。
```bash
# 查看网卡信息
ifconfig
# 临时设置网卡的地址和子网掩码
ifconfig eth0 192.168.16.2 netmask 255.255.255.0
# 设置网关
route add default gw 192.168.16.1
```

## 网络配置文件

对于Debian系发行版，修改后重启网络服务或者重启系统生效。
```bash
sudo vim /etc/network/interfaces

# 加上
auto eth0
iface eth0 inet static
address 192.168.16.2
netmask 255.255.255.0
gateway 192.168.16.1
```

## nmtui或nmcli

属于NetworkManager，推荐用tui的，相当于图形化界面了，很方便。不过如果用其它方法（比如网络配置文件）设置了网卡，nm就不会纳入自己的管理，只有删掉或注释掉配置并重启NetworkManager才能将这个网卡显示在nm中。
```bash
sudo systemctl restart NetworkManager
sudo nmtui
```
