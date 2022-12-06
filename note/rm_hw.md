- **before** **reading**：
  - 运行rm_hw节点，最终会进入一个循环：control loop(nh,-)







# 1. rm_hw.cpp

1. ```c++
   ros::AsyncSpinner spinner(2);
   spinner.start();
   /*我们在单独的线程中运行 ROS 循环作为外部调用，例如作为加载控制器的服务回调，可以阻塞（主）控制循环*/
   //这两句就是开了一个单独的线程
   ```

2. ```c++
   //为进程设置调度算法和/或参数；（这里是设置参数）
   struct sched_param params
   {
     .sched_priority = 95
   };
   if (sched_setscheduler(0, SCHED_FIFO, &params) == -1)
     ROS_ERROR("Set scheduler failed, RUN THIS NODE AS SUPER USER.\n");//设置失败会提示用sudo运行这个节点
   ```

3. ```c++
   try
   {
     // Create the hardware interface specific to your robot（根据机器人设置硬件接口）
     std::shared_ptr<rm_hw::RmRobotHW> rm_hw_hw_interface = std::make_shared<rm_hw::RmRobotHW>();
     // Initialise the hardware interface:
     // 1. retrieve configuration from rosparam（从参数服务器加载参数）
     // 2. initialize the hardware and interface it with ros_control
     rm_hw_hw_interface->init(nh, robot_hw_nh);
   
     // Start the control loop
     rm_hw::RmRobotHWLoop control_loop(nh, rm_hw_hw_interface);
   
     // Wait until shutdown signal received（没有shutdowm就会一直运行control_loop）
     ros::waitForShutdown();
   }
   ```

# 2. control_loop.cpp

1. 从参数服务器加载参数，包括循环频率、循环时间误差阈值、优先级

2. 初始化硬件接口： 
   - 从 rosparam 获取配置 
   - 初始化硬件并将其与 ros_control 连接
   
3. 创建controller manager

4. 获取一次当前时间戳，用于第一次的update

5. 设置一个线程，用于循环

   - ```
       loop_thread_ = std::thread([&]() {
         while (loop_running_)
         {
           if (loop_running_)
             update();
         }
       })
     ```

   - update()

     ```
     
     ```

     

​	

# 2. hardware_interface.cpp

实际上是继承了robotHW这个类，然后写我们自己的rmHW；也就是写我们自己的硬件层跟一些硬件接口

## 1. init()

1. 获取参数
2. 注册接口

```c++
  registerInterface(&robot_state_interface_);
  registerInterface(&gpio_state_interface_);
  registerInterface(&gpio_command_interface_);
```

3. 

# 3. can_bus.h

## 1. CanBus::CanBus(...)

1. 打开socketcan，成功则提示：successfully connect to....
2. 设置can包的header

## 2. write

```c++
void CanBus::write()
{
  bool has_write_frame0 = false, has_write_frame1 = false;
  // safety first
  std::fill(std::begin(rm_frame0_.data), std::end(rm_frame0_.data), 0);
  std::fill(std::begin(rm_frame1_.data), std::end(rm_frame1_.data), 0);
  // std::fill函数的作用是：将一个区间的元素都赋予指定的值，即在[first, last)范围内填充指定值。

  for (auto& item : *data_ptr_.id2act_data_)
  {
    if (item.second.type.find("rm") != std::string::npos)
    {
      if (item.second.halted)
        continue;
      const ActCoeff& act_coeff = data_ptr_.type2act_coeffs_->find(item.second.type)->second;
      int id = item.first - 0x201;
      double cmd =
          minAbs(act_coeff.effort2act * item.second.exe_effort, act_coeff.max_out);  // add max_range to act_data
      if (-1 < id && id < 4)
      {
        rm_frame0_.data[2 * id] = (uint8_t)(static_cast<int16_t>(cmd) >> 8u);
        rm_frame0_.data[2 * id + 1] = (uint8_t)cmd;
        has_write_frame0 = true;
      }
      else if (3 < id && id < 8)
      {
        rm_frame1_.data[2 * (id - 4)] = (uint8_t)(static_cast<int16_t>(cmd) >> 8u);
        rm_frame1_.data[2 * (id - 4) + 1] = (uint8_t)cmd;
        has_write_frame1 = true;
      }
    }
    else if (item.second.type.find("cheetah") != std::string::npos)
    {
      can_frame frame{};
      const ActCoeff& act_coeff = data_ptr_.type2act_coeffs_->find(item.second.type)->second;
      frame.can_id = item.first;
      frame.can_dlc = 8;
      uint16_t q_des = (int)(act_coeff.pos2act * (item.second.cmd_pos - act_coeff.act2pos_offset));
      uint16_t qd_des = (int)(act_coeff.vel2act * (item.second.cmd_vel - act_coeff.act2vel_offset));
      uint16_t kp = 0.;
      uint16_t kd = 0.;
      uint16_t tau = (int)(act_coeff.effort2act * (item.second.exe_effort - act_coeff.act2effort_offset));
      // TODO(qiayuan) add position vel and effort hardware interface for MIT Cheetah Motor, now we using it as an effort joint.
      frame.data[0] = q_des >> 8;
      frame.data[1] = q_des & 0xFF;
      frame.data[2] = qd_des >> 4;
      frame.data[3] = ((qd_des & 0xF) << 4) | (kp >> 8);
      frame.data[4] = kp & 0xFF;
      frame.data[5] = kd >> 4;
      frame.data[6] = ((kd & 0xF) << 4) | (tau >> 8);
      frame.data[7] = tau & 0xff;
      socket_can_.write(&frame);
    }
  }

  if (has_write_frame0)
    socket_can_.write(&rm_frame0_);
  if (has_write_frame1)
    socket_can_.write(&rm_frame1_);
}
```