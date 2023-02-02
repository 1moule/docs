**before reading**

1. engineer_middleware.cpp中定义了节点的main函数，会循环调用Middleware::run()
2. 当收到一个goal时，会调用一个回调，回调中会把is_middleware_control_设为true；如果is_middleware_control为true，Middleware::run()会调用ChassisInterface::run()
3. ChassisInterface::run()
   - 获取base_link到map的偏移
   - 获得目标点跟当前位置的xy的误差
   - 将误差从map转换到base_link
   - 获取底盘yaw误差
   - 计算pid，设置cmd_vel
   - 发布cmd_vel
   - 更新error_pos，error_yaw