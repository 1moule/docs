## 1. method

1. 检测是否识别到gpio

```
cd /sys/bus/i2c/devices
```

拔插usb2gpio，对比若在该目录下出现一个新的i2c总线，则识别成功

2. 修改 rm_bringup/scripts/auto_start/auto_set_gpio.sh

   - 将以下这一行中的i2c-5修改成拔插后出现的总线名称

   ```bash
   sudo sh -c 'echo pcf8574 0x20 > /sys/bus/i2c/devices/i2c-5/new_device'
   ```

   - 运行auto_set_gpio.sh，然后进入以下目录

   ```bash
   cd /sys/class/gpio
   ```

   拔插usb2gpio，出现一个新的gpiochip...，修改脚本

   ```bash
   for i in 760 768 #760和768为目录中的gpio编号
   do
   echo ${i} > /sys/class/gpio/export
   sudo chmod 777 /sys/class/gpio/gpiochip${i}/direction
   sudo chmod 777 /sys/class/gpio/gpiochip${i}/value
   ```

3. 运行auto_set_gpio.sh

4. 修改rm_hw配置文件

5. 修改rm_controllers配置文件

6. 通过rqt向/controllers/gpio_controller/command发消息，听/controllers/gpio_controller/gpio_state，gpio state变成对应的state，使用成功

