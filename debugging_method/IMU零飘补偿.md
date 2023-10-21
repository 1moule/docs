# method

1. 将rm_config/rm_hw/${robot_type}.yaml中的imu：gimbal_imu的angular_vel_offset全部设成0

2. plotjuggler查看gimbla_imu/angular_x/y/z
3. 右键点击图像，选中apply_filter_to_data，选中moving_average，将simple_count拉满
4. 机器人静止一段时间，等到图像小幅波动时，记录三个图像的大概的值（我一般buffer给200，然后等到最开始那段波动大的没有了之后，每个轴取最大跟最小的平均值）
5. 将4中得到的值取反，填到rm_config/rm_hw/${robot_type}.yaml中的imu：gimbal_imu的angular_vel_offset