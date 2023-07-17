## robot_state_interface.h

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