实际上是继承了robotHW这个类，然后写我们自己的rmHW；也就是写我们自己的硬件抽象层

## 1. init()

1. 获取参数
2. 注册接口

```c++
  registerInterface(&robot_state_interface_);
  registerInterface(&gpio_state_interface_);
  registerInterface(&gpio_command_interface_);
```

3. reset电机状态的实时发布者、定义一个服务器

## 2. read()

1. 遍历每一个can bus，调用CanBus::read()，读取can帧
2. 