## 1. GPIO使用原理

1. 硬件部分

- 电路制作了USB转GPIO的板子，通过USB转7pin的线可以插到PC上
- 因为NUC上没有gpio外设，所以需要制作一块GPIO的板子，让NUC具备操作GPIO的功能

2. 软件部分

- 将板子插到PC上，linux通过自带的GPIO子系统驱动框架，可以识别到GPIO；
- 将gpio编号写入/sys/class/gpio/export目录，用于通知系统导出需要控制的gpio的编号，也就是向内核申请将某个gpio的控制权导出到用户空间，只有写入了之后，用户才可以对gpio进行操作，
  -  注：成功后生成/sys/class/gpio/gpio${编号}目录。如果没有出现相应的目录，说明此引脚不可导出，一般这种情况是驱动中pinmux功能配置不正确，或者配置了多种pinmux功能引起冲突导致

## 2. GPIO操作（最底层）

1. 配置gpio输入输出方向 

   eg：echo out >/sys/class/gpio/gpio195/direction direction可接收的参数：in，out，high，low；其中high，low设置方向为输出并将value值设置为相应的1/0。 

2. 配置gpio的高低电平(0/1) 

   （**当gpio设置为输出时，才可以调过设置value的值，设置高低电平** ）

   eg：echo 1 >/sys/class/gpio/value

## 3. 通过rm_control操作GPIO

1. rm_hw > harware_interface > gpio_manager（**实现从硬件抽象层 -> 真实机器人**）
   - readGpio()会从 /sys/class/gpio/gpio%d/value 读取GPIO的状态（0/1）
   - writeGpio()会向 /sys/class/gpio/gpio%d/value 写入0/1
2. rm_common > hardware_interface > gpio_interface（**从控制器 -> 硬件抽象层**）
   - GpioStateHandle类，这是gpio状态的句柄，通过方法getValue()可以获取gpio的状态
   - GpioCommandHandle类，这是gpio命令的句柄，通过方法setCommand()可以设置gpio的状态
3. 在gpio_controller中，通过句柄操作硬件，可以获取GPIO的状态、以及向GPIO写入状(**控制器层，详细可看gpio_controller.md**)