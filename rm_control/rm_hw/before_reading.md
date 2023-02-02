- **before** **reading**：

  - 运行rm_hw节点，最终会进入一个循环：control loop(nh,-)

  - control_loop的update()最主要的三件事：

    (1)获取电机的状态

    - 调用hardware_interface_->read()
      - 调用CanBus::read()
        - can_bus.cpp中写了一个收到can帧就会调用的回调——CanBus::frameCallback()；这个回调会把收到的can帧（电机数据）写到read_buffer_中
        - CanBus::read()会对read_buffer_中的数据，也就是电机数据进行处理，然后赋给act_data/imu_data/tof_data...
        - 在parse.cpp中，会对这些act_data/imu_data/tof_data...之类的数据进行处理
      - 调用GpioManager::readGpio()
        - gpio必须为input才能读状态
        - 打开文件/sys/class/gpio/gpio" + std::to_string(iter->pin) + "/value，获取值后赋给value

    (2)调用controller_manager的update()，compute the new command

    - 调用controller_manager_->update()

    (3)给电机发指令

    - 调用hardware_interface_->write()
      - 调用CanBus::write()
        - 写好can帧后，调用SocketCAN::write()
          - 调用write(sock_fd_, frame, sizeof(can_frame))，向can总线发这帧数据
      - 调用gpio_manager_.writeGpio()
        - 打开/sys/class/gpio/gpio$(pin)/value"这个文件
        - 把0/1写到文件中
        - 关闭文件
      - 调用publishActuatorState(time)
        - 发布下一次的电机状态（电机的数据）

