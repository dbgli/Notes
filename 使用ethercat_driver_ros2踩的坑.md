TODO:
- 机器人描述用话题发布而不是用参数传递
- URDF中也必须要有ros2_control中对应的joint，即使不需要仿真
- 各种控制器同一时刻只能有一个工作，不是位置速度力矩一起来控制
- 至少汇川的EtherCAT通讯只能有最多十个对象或40个字节，否则报错，可复位
- xacro相关
- xacro标签要定义在robot标签内，外面无效，但是一旦定义，定义位置和调用位置没有先后顺序
- `$()`用于调用外部ros命令，如arg和find，`${}`用于借助python.math计算表达式和变量替换，支持一些预定义的数学常量和函数，如pi
- 似乎gazebo只能绝对路径？
- 常用urdf模板：
```yaml
link
	visual
		origin
			xyz
			rpy
		geometry
			box
			mesh filename
		material
			color
				rgba
			texture
	collision
		origin
			xyz
			rpy
		geometry
			box
			mesh filename
		material
			color
				rgba
			texture
	inertial
		mass
		inertia
```