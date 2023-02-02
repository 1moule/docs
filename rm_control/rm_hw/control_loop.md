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

     