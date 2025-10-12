# GDB+ROS

## 1. ros controller gdb调试（sim）

1. 查看gzserver进程PID

   ```bash
   ps aux | grep gazebo
   ```

2. 将GDB attach到gzserver进程

   ```bash
   # 假设进程PID为27432
   gdb -p 27432 
   ```

3. 继续运行控制器

   ```bash
   # 在gdb终端中输入
   continue
   ```

4. 此时可以通过rqt controller manager正常启动控制器

5. 设置断点

   ```bash
   # 设置源码路径
   dir ~/catkin_ws/src/your_controller_pkg/src
   dir /home/guanlin/arm_ws/src/qm_control/qm_controllers/src
   
   # 设置断点
   break your_file:your_line_num
   ```

6. 分屏源码、命令行

   ```bash
   layout src 
   ```

6. 单步运行

   ```bash
   # 运行源码中的单行
   next
   ```

7. 进入函数内部的单步运行

   ```bash
   step
   ```

8. 