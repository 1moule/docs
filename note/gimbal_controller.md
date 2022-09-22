- **before_reading**：
  - rate：
  - track：
  - direct：



# 1. gimbal_base.h

1. ```
   geometry_msgs::TransformStamped odom2gimbal_des_, odom2pitch_, odom2base_, last_odom2base_;
   ```

geometry_msgs::TransformStamped类型的变量存储两个坐标系之间的tf转换关系

2. ```
   effort_controllers::JointPositionController ctrl_yaw_, ctrl_pitch_;
   ```

   （1）effort_controllers::JointPositionController类型的变量，就是定义了一个关节的位置控制器

---



# 2. gimbal_base.cpp

## 1. update()

1. 流程：
   - 获取gimbal的command以及data_track
   - 获取tf转换关系
   - 给底盘法速度指令
   - 根据不同状态执行不同的动作函数
   - movejoint

2. ```
   try
    {
      odom2pitch_ = robot_state_handle_.lookupTransform("odom", ctrl_pitch_.joint_urdf_->child_link_name, time);
      odom2base_ = robot_state_handle_.lookupTransform("odom", ctrl_yaw_.joint_urdf_->parent_link_name, time);
    }
   ```

   （1）获取odom2pitch和odom2yaw的坐标转换关系

   （2）ctrl_pitch.joint_urdf_->child_link_name；joint_urdf _是joint_position_controller下的类成员，类型是urdf::JointConstSharedPtr；

```c++
joint_urdf= urdf.getJoint(joint_name);//在joint_position_controller.cpp
```

joint_name是从参数服务器中加载的，比如yaw：

```
yaw:
	joint:"yaw_joint"
```

## 2. rate()

1. on enter

   （1）

   ```
   odom2gimbal_des_.transform.rotation = odom2pitch_.transform.rotation
   ```

   获取odom2gimbal_des的旋转矩阵；因为gimbal的姿态跟pitch这个link是一样的，所以gimbal的旋转矩阵就是pitch的

   （2）

   ```
   robot_state_handle_.setTransform(odom2gimbal_des_, "rm_gimbal_controllers");
   ```

   将odom2gimbal_des的转换信息存放到tf、数据结构中

2. 不是第一次进入rate模式时(就是保持在rate模式时)

   （1）把on enter时那个四元数（odom2gimbal_des）转换成欧拉角，作为pitch yaw的初始位置
   
   （2）setDes()会在初始的yaw pitch基础上加上command sender发过来的pitch yaw的位置指令，设置一个目标点

## 3. track()：追踪模式

```c++
void Controller::track(const ros::Time& time)
{
  if (state_changed_)
  {  // on enter
    state_changed_ = false;
    ROS_INFO("[Gimbal] Enter TRACK");
  }
  double roll_real, pitch_real, yaw_real;
    
  //获取odom到pitch的四元数并转换为欧拉角
  quatToRPY(odom2pitch_.transform.rotation, roll_real, pitch_real, yaw_real);
  double yaw_compute = yaw_real;
  double pitch_compute = -pitch_real;
  geometry_msgs::Point target_pos = data_track_.target_pos;
  
  //定义target_pos和target_vel，将data_track（这个是视觉获取的信息）中的target_pos和target_vel赋值给他们
  geometry_msgs::Vector3 target_vel = data_track_.target_vel;
  try
  {
    /*如果data_track的信息不是空的
	  1. 获取data_track中数据所在的坐标系到odoom坐标系的转换关系
	  2. 把target_pos和target_vel转换到odom坐标系*/
    if (!data_track_.header.frame_id.empty())
    {
      geometry_msgs::TransformStamped transform =
          robot_state_handle_.lookupTransform("odom", data_track_.header.frame_id, data_track_.header.stamp);
      tf2::doTransform(target_pos, target_pos, transform);
      tf2::doTransform(target_vel, target_vel, transform);
    }
  }
  catch (tf2::TransformException& ex)
  {
    ROS_WARN("%s", ex.what());
  }
    
  //
  target_pos.x = target_pos.x - odom2pitch_.transform.translation.x;
  target_pos.y = target_pos.y - odom2pitch_.transform.translation.y;
  target_pos.z = target_pos.z - odom2pitch_.transform.translation.z;

  bool solve_success = bullet_solver_->solve(target_pos, target_vel, cmd_gimbal_.bullet_speed);

  if (publish_rate_ > 0.0 && last_publish_time_ + ros::Duration(1.0 / publish_rate_) < time)
  {
    if (error_pub_->trylock())
    {
      //如果上🔓成功
      double error =
          bullet_solver_->getGimbalError(target_pos, target_vel, yaw_compute, pitch_compute, cmd_gimbal_.bullet_speed);
      error_pub_->msg_.stamp = time;
      error_pub_->msg_.error = solve_success ? error : 1.0;
      error_pub_->unlockAndPublish();
    }
    //上🔓失败
    bullet_solver_->bulletModelPub(odom2pitch_, time);
    last_publish_time_ = time;
  }
  //如果解算成功，则setDes
  if (solve_success)
    setDes(time, bullet_solver_->getYaw(), bullet_solver_->getPitch());
  //解算失败，设置tf
  else
  {
    odom2gimbal_des_.header.stamp = time;
    robot_state_handle_.setTransform(odom2gimbal_des_, "rm_gimbal_controllers");
  }
}
```

## 4. direct()

```c++
void Controller::direct(const ros::Time& time)
{
  if (state_changed_)
  {  // on enter
    state_changed_ = false;
    ROS_INFO("[Gimbal] Enter DIRECT");
  }
    
  //从cmd_gimbal中获取目标点的位置
  geometry_msgs::Point aim_point_odom = cmd_gimbal_.target_pos.point;
  try
  {
    if (!cmd_gimbal_.target_pos.header.frame_id.empty())//就是说指令中设置了目标
      
      //将目标点位置从cmd_gimbal中的target_pos所在的坐标系转换到odom下
      tf2::doTransform(aim_point_odom, aim_point_odom,
                       robot_state_handle_.lookupTransform("odom", cmd_gimbal_.target_pos.header.frame_id,
                                                           cmd_gimbal_.target_pos.header.stamp));
  }
  catch (tf2::TransformException& ex)
  {
    ROS_WARN("%s", ex.what());
  }
  
  //获取计算反正切，得pitch跟yaw要转动的角度
  double yaw = std::atan2(aim_point_odom.y - odom2pitch_.transform.translation.y,
                          aim_point_odom.x - odom2pitch_.transform.translation.x);
  double pitch = -std::atan2(aim_point_odom.z - odom2pitch_.transform.translation.z,
                             std::sqrt(std::pow(aim_point_odom.x - odom2pitch_.transform.translation.x, 2) +
                                       std::pow(aim_point_odom.y - odom2pitch_.transform.translation.y, 2)));
  setDes(time, yaw, pitch);
}
```

## 5. updateChassisVel()

1. tf_period，是上一个获取odom2base_转换关系的时刻，到现在这次获取转换关系的时刻之间的时间间隔
2. chassis_vel；因为这个是通过odom2base_ (实际就是odom2yaw)算出来的，所以云台动了之后，才得到odom2base_，才有chassis_vel，才让底盘运动

## 6. setDesIntoLimit(...)

1. 原理：如果base2gimbal_current_des_没有超过关节限位，那么odom2gimbal的current_des就没有问题，就可以让real_des等于current_des
2. 这个函数会判断base2gimbal_current_des有没有超过关节限位，没有就令real_des = current_des再返回true；否则返回false

## 7. setDes()

1. tf2::Quaternion；这是创建Quaternion类的实例，是声明tf类型的四元数；ros中四元数一种是msg类型，一种是tf类型

2. ```c++
    tf2::fromMsg(odom2base_.transform.rotation, odom2base);
   ```

   tf2::fromMsg（const geometry_msgs::Quaternion & in，tf2::Quaternion& out）；将msg类型的四元数转换为tf类型

3. ```
   odom2gimbal_des.setRPY(0, pitch_des, yaw_des);
   ```

​		通过欧拉角（弧度）设置四元数

4. ```c++
    base2gimbal_des = odom2base.inverse() * odom2gimbal_des;
   ```

   ？(1)odom2base.inverse()；返回odom2base这个旋转矩阵的逆，也就是说，当一个在odom下的矩阵乘这个逆阵，就会得到一个base下的矩阵 

5. current_des是通过tf转换得到的应该移动到的位置，real_des是真实会移动到的位置；因为jonit是有限位的，如果current_des超过了这个限位，就会有一个合理的real_des

6. ```c++
   if (!setDesIntoLimit(yaw_real_des, yaw_des, base2gimbal_current_des_yaw, ctrl_yaw_.joint_urdf_))
   ```

   （1）如果当前的current_des超过了关节限位，那么就会设置一个新的不超过限位的base2new_des

   （2）method：定义一个tf2类型的四元数，获取关节限位后，通过一个三目运算，通过设置欧拉角来设置四元数

7. ```
   base2new_des.setRPY(0,
                       std::abs(angles::shortest_angular_distance(base2gimbal_current_des_pitch, upper_limit)) <
                               std::abs(angles::shortest_angular_distance(base2gimbal_current_des_pitch, lower_limit)) ?
                           upper_limit :
                           lower_limit,
                       base2gimbal_current_des_yaw);
   ```

   （1）这段代码运行的条件是base2gimbal_current_des_超过了关节限位；如果base2gimbal_current_des_pitch到upper_limit的最小角度（保证这个角在0到2pi的范围）小于到lower_limit的，那就说明要抬头并且抬的角度超过了限位；反之就是低头超限位
   
   ```
   quatToRPY(toMsg(odom2base * base2new_des), roll_temp, pitch_temp, yaw_real_des);
   ```
   
   （1）odom2base*base2new_des得到odom2new_des的转换关系，toMsg()把这个四元数转换为msg类型，quatToRPY()会把msg类型的四元数转换为欧拉角存放到另外那三个参数里
   
8. ```
    robot_state_handle_.setTransform(odom2gimbal_des_, "rm_gimbal_controllers");
    ```

## 8. moveJoint()

1. 先判断是否有imu，有imu
   - 获取三轴上的速度，存放到一个geometry_msgs::vector3（三维向量）类型的变量(gyro)中
   - tf2::dotransform(...)，函数参数中t_in是转换的输入，t_out是转换的输出，transform是转换关系；这里通过imu到pitch（或者imu到yaw）的转换关系，把imu坐标系下的三个轴的速度转换成pitch（或yaw）坐标系下的三轴的速度
   
2. 没有imu，就直接通过读取关节速度，精度会比imu低一些
   - jointPositionController.joint是hardware_interface::JointHandle类型的变量；是用于读取和命令单个关节的句柄

3. ```
   base_frame2des =
         robot_state_handle_.lookupTransform(ctrl_yaw_.joint_urdf_->parent_link_name, gimbal_des_frame_id_, time);
     double roll_des, pitch_des, yaw_des;  // desired position
     quatToRPY(base_frame2des.transform.rotation, roll_des, pitch_des, yaw_des);
   ```

   （1）获取从gimbal_des坐标系到yaw的父坐标系的四元数转换关系，并转换为欧拉角
   
4. 定义pitch和yaw的目标速度

   （1）如果state_是RATE，则pitch和yaw的目标速度直接从cmd_gimbal获取；

   （2）非RATE

   - 从data_track获取目标位置和速度

   - ```c++
     geometry_msgs::TransformStamped transform = robot_state_handle_.lookupTransform(
         ctrl_yaw_.joint_urdf_->parent_link_name, data_track_.header.frame_id, data_track_.header.stamp);
         
     //获取从data_track_.header.frame_id到yaw的父link的转换关系
     //data_track_.header.frame_id可以理解为：数据所在坐标系的名称，在这里也就是指目标速度和目标位置所在的坐标系
     ```

   - doTransform，把目标速度和位置转换到yaw的父坐标系下

   - tf2::fromMsg，msg类型转换为tf类型

   - ```c++
     yaw_vel_des = target_vel_tf.cross(target_pos_tf).z() / std::pow((target_pos_tf.length()), 2);
     //tf2::Vector3.cross()这个函数会返回该向量与另一个向量（cross函数的参数）之间的叉乘
     //z.()会返回这个向量的z的值
     ```

   - 

## 9. feedForward()

- Eigen库

（1）Eigen是一个用头文件搭起来的线性代数库，没有二进制文件，使用时只要引入头文件

（2）Eigen是一个模板类，前三个参数为：数据类型，行，列

（3）eg:

```c++
//声明一个 2*3 的 float 矩阵
Eigen::Matrix<float, 2, 3>; matrix_23;
```

（4）Eigen通过typedef提供了许多内置的类型，不过底层都是(3)，比如：

```c++
//声明一个 三维向量 
Eigen::Vector3d v_3d;
```

---

1. feedforward其实就是pitch受到的重力，通过向量叉乘计算得到
2. 如果enable_gravity_compensation_为true说明有重力补偿，会通过同样的计算方法得到一个值，这个值可以看作是对抗重力的力，feedforward减去这个值后的结果，就是feedForward函数返回的值，也就是实际pitch会受到的力

---



# 3. bullet_solver.cpp

## 1. solve(...)

1. 从参数的实时buffer中获取参数

2. 通过函数传入的参数，给变量target_pos（目标点位置）、bullet_speed（弹速）赋值

3. resistance_coff（发射过程中的阻力系数）的赋值

   - ```c++
     getResistanceCoefficient(bullet_speed)
     //这个函数会通过config_这个结构体中的参数给变量resistance_coff赋值并返回
     ```

   - 如果返回的值不为0，resistance_coff赋值为getResistanceCoefficient(bullet_speed)返回的值；否则赋值为0.001

4. ```c++
   bool BulletSolver::solve(geometry_msgs::Point pos, geometry_msgs::Vector3 vel, double bullet_speed)
   {
     config_ = *config_rt_buffer_.readFromRT();
     target_pos_ = pos;
     bullet_speed_ = bullet_speed;
     resistance_coff_ = getResistanceCoefficient(bullet_speed_) != 0 ? getResistanceCoefficient(bullet_speed_) : 0.001;
   
     int count{};
     double temp_z = pos.z;
     double target_rho;
     double error = 999;
     while (error >= 0.001)
     {
       output_yaw_ = std::atan2(target_pos_.y, target_pos_.x);
       //因为yaw只在xoy平面上移动，所以这里只需要通过目标点的xy来获取yaw的角度
       //std::atan2(y,x)会返回arctan(y/x)；这里通过对目标点的xy坐标计算反正切，得到云台yaw应该转动的角度
       
       output_pitch_ = std::atan2(temp_z, std::sqrt(std::pow(target_pos_.x, 2) + std::pow(target_pos_.y, 2)));
       //将目标点投影到xoy平面，与原点连线；通过求这条连线与目标点的z高度的反正切，可以得到pitch的角度0
       //temp_z是视觉获取的目标点的z
       
       target_rho = std::sqrt(std::pow(target_pos_.x, 2) + std::pow(target_pos_.y, 2));
       //原点到目标投影点的距离
       
       double fly_time =
           (-std::log(1 - target_rho * resistance_coff_ / (bullet_speed_ * std::cos(output_pitch_)))) / resistance_coff_;
       //子弹到达目标点的飞行时间
       
       double real_z = (bullet_speed_ * std::sin(output_pitch_) + (config_.g / resistance_coff_)) *
                           (1 - std::exp(-resistance_coff_ * fly_time)) / resistance_coff_ -
                       config_.g * fly_time / resistance_coff_;
       //子弹发射之后，子弹真实能到的高度
   
       target_pos_.x = pos.x + vel.x * (config_.delay + fly_time);
       target_pos_.y = pos.y + vel.y * (config_.delay + fly_time);
       target_pos_.z = pos.z + vel.z * (config_.delay + fly_time);
       //加上飞行时间和延迟时间，预测目标的位置
   
       double target_yaw = std::atan2(target_pos_.y, target_pos_.x);
       
       double error_theta = target_yaw - output_yaw_;
       double error_z = target_pos_.z - real_z;
       temp_z += error_z;
       error = std::sqrt(std::pow(error_theta * target_rho, 2) + std::pow(error_z, 2));
       count++;
       
       //如果修正次数大于20或者error是一非数值  
       if (count >= 20 || std::isnan(error))//std::isnan(...),会先把参数转换成浮点数，如果是一个非数值，返回true，否则返回false
         return false;
     }
     return true;
   }
   ```
   

<img src="/home/chen/Desktop/typora-user-image/IMG_20220725_154624.jpg" alt="image" style="zoom: 25%;" />

​		（1）解算就是不断地修正target_pos和output_yaw、output_pitch，让error不断减小

​		（2）当修正次数达到20次，或着error是一个非数值，返回false，此时解算是失败的

​		（3）如果解算次数不到20次，error就已经小于0.01了，那解算就是成功了，返回true；此时的output_pitch和output_yaw是合适的，可以用来movejoint

## 2. getGimbalError

1. 这里通过不断修正target_pos、计算fly_time，当前后两次计算出来的fly_time相差小于等于0.01，则说明修正完成
2. 如果修正次数大于20或者算出来的fly_time是非值数，说明修正失败，error直接返回999
3. 如果修正成功，则计算此时的error并返回
