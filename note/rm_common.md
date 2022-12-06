- **include/rm_common/hardware_interface**

  - robot_state_interface.h

  1. tf2_ros::Buffer继承tf2_ros::BufferInterface 和tf2::BufferCore

  2. ```
     bool setTransform(const geometry_msgs::TransformStamped& transform, const std::string& authority,
                       bool is_static = false) const
     {
       return buffer_->setTransform(transform, authority, is_static);
     }
     ```

     （1）buffer_->setTransform(transform, authority, is_static)；setTransform()是tf2::BufferCore这个类的成员函数，用来把转换消息存放到tf的数据结构

     （2）这里将这个函数写成了一个类成员函数，这样可以方便使用这个函数，代码复用性也好

---

- **include/rm_common/decision/command_sender.h**

  - 用控制器控制时，通过rqt向topic发消息

  - 当开manual时，就是用遥控器控制时，他是通过command_sender发送消息，

  - chassis_gimbal_manaual.cpp为例，里面有一个sendCommand函数，是用来向话题发消息的；这个函数实际上是command_sender里面多个类的实例的sendCommaVel2DCommandSendernd函数的集合

1. Vel2DCommandSender

   构造函数：

   - 从参数服务器获取max_linear_x、max_linear_y、max_linear_z（每个都是一个列表），并初始化
   - 订阅话题，获取功率限制，在回调中存下来

   setLinearXVel(double scale)：

   - 通过线性插值的方法，得到当前power_lmit对应的max_linear_x，乘上一个scale就是发到话题的linear.x
   - 其他类似