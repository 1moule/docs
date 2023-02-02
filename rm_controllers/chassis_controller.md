- **before_reading**：
  - raw（0）模式：
    - 将底盘的速度指令全部清除
  - follow（1） 跟随模式：
    - 如果没有设置跟随的坐标系or命令他跟随的坐标系，都将跟随的坐标系设置为yaw（follow跟随的一般都是yaw）
    - 流程：
      - 获取yaw到base_link的四元数转换关系，并转换为一组欧拉角
      - 获取yaw角到0角的最小的角度差（follow_error），这个结果通常在-pi到pi
      - 通过follow_error得到pid指令
      - 从这个pid指令得到底盘速度指令（vel_cmd_z）
  - gyro（2）小陀螺：
    - 把vel_cmd从yaw坐标系转换到base_link坐标系（开gazebo向cmd_vel发速度指令，会发现这个速度指令是yaw坐标系上的）
    - 给一个cmd_vel.z（自转角速度）就能小陀螺
  - twist（3）扭腰模式
    - follow_error的计算公式，导致follow_error拧转一样的变化，再导致vel_cmd.z呈现拧转的变化，最终重现出拧转


# 1. chassis_base.cpp

## 1. power_limit()344-349

```c++
if (joint.getName().find("wheel") != std::string::npos)  // The pivot joint of swerve drive doesn't need power limit
    {
      a += square(cmd_effort);
      b += std::abs(cmd_effort * real_vel);
      c += square(real_vel);
    }
```

npos可以表示string的结束位子，是string::type_size 类型的，也就是find（）返回的类型。find函数在找不到指定值得情况下会返回string::npos。上面代码就是当找到了一个joint叫”wheel“时，执行判断。

## 2. follow()

### 1. pid_follow.reset()

1. ```
   control_toolbox::Pid pid_follow_;
   
   这个类可以用于创建范围广泛的pid控制器
   ```

2. .reset()；Reset the state of the PID controller. 

### 2. pid_follow_.computeCommand(-follow_error, period)；

1. 根据输入的误差计算pid指令，根据dt计算微分项

## 3. tfVelToBase(&from)

1. 将from坐标系下的速度指令转换到base_link下的

## 4. twist()

1. 跟follow的唯一区别就是com_source_frame固定为yaw
2. 

# 2. reaction_wheel.cpp

1. 77

```c++
  XmlRpc::XmlRpcValue q, r;
```

利用ROS自带的XmlRpc::XmlRpcValue可以实现一维数组或者二位数组等类似json数据的读取

C++代码实现如下

```
  XmlRpc::XmlRpcValue scanner_params;
  nh.getParam("scanners", scanner_params);
  for(size_t i = 0; i<scanner_params.size(); ++i)
  {
    const String& server_ip     = scanner_params[i]["server_ip"];
    const int&    server_port   = scanner_params[i]["server_port"];
    const String& frame_id      = scanner_params[i]["frame_id"];
    const String& pub_topic     = scanner_params[i]["pub_topic"];
  }
```

Yaml文件中数据如下

```
scanners: 
- { pub_topic: "scan_head", frame_id: "laser_scanner_link_head",server_ip: "192.168.167.100",server_port: 2111}
- { pub_topic: "scan_middle",frame_id: "laser_scanner_link_middle",server_ip: "192.168.167.101",server_port: 2111}
- {pub_topic: "scan_tail",frame_id: "laser_scanner_link_tail",server_ip: "192.168.167.102",server_port: 2111}
```