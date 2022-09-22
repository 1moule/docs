# 1. dbus.cpp

## 1. read()

1. 键鼠or遥控，按下某个按键后，接收机会受到相对应的数据，然后通过串口把数据传给nuc
2. read()功能就是：让nuc读到接收机通过串口发送过来的数据

## 2. uppack()

1. 当接收完数据后，需要对数据进行解包，
2. 这个函数实际就是把buffer这个数组中每一个数据赋给d_bus_data_中的对应的变量
3. uppack函数是在read函数中被调用的

## 3. getdata()

1. 利用d_bus_data_中的值，给d_bus_data中的变量赋值（通过三目运算符+位运算，最终得到的结果是ture/false）

## 4. 流程

（以按键变量为例）

1. 遥控器连接pc，在pc上按下按键
2. 遥控器接收机通过串口发送对应数据给nuc
3. nuc上运行的dbus节点会一直运行run函数，即一直运行read、unpack、getdata
4. getdata最终结果就是相应的按键变量被赋值为true
4. 机器人根据按键进行相应的动作，这个写在manual中

