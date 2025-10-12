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
