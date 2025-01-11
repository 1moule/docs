## 编写ros2 controller

1. 一些概念

   - ros2的控制器需要继承controller_interface::ControllerInterface

   - ControllerInterface继承自ControllerInterfaceBase类

   - ControllerInterfaceBase继承自rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface即[生命周期节点](https://design.ros2.org/articles/node_lifecycle.html)；增加init、on_init、command_interface_config、state_interface_config、update等方法

     - init被controller manager调用，内部会调用on_init函数，on_init需要用户自定义

   - 关于生命周期节点

     - 区别与普通节点（rclcpp::Node），LifecycleNode将节点运行状态进行了具体划分，并且可以对生命周期中每一个状态进行管理，类似状态机

     - LifecycleNode有**4**个**主要状态**（稳态）（图中**蓝**框）：Unconfigured、Inactive、Active、Finalized

     - LifecycleNode有**6**个**过渡状态**（瞬态）（图中**黄**框）：Configuring、CleaningUp、ShuttingDown、Activating、Deactivating、ErrorProcessing；过渡状态是主要状态切换的中间过程，会执行相应的逻辑

     - 主要状态之间的转换需要一个外部的监督进程

     - LifecycleNode有**7**种**转换**（图中**蓝线**）：create、configure、cleanup、activate、deactivate、shutdown、destroy；表示当前发生的转换，提供给监督进程

     - LifecycleNode运行流程

       ![The proposed node life cycle state machine](https://design.ros2.org/img/node_lifecycle/life_cycle_sm.png)
   
       - 左上开始，进入Unconfigured状态（初始状态）
       
       - 进入瞬态Configuring，调用onConfigure，加载配置，获取资源；成功则切换为inactive（待激活）状态；失败则回到Unconfigured初始状态
       
       - 进入inactive状态后，进入瞬态CleaningUp，执行onCleanup，放弃资源，清理状态，回到初始状态重新开始、或进入瞬态Activating，执行onActivate，激活所有资源，启动任务，成功则进入Active状态，运行主循环
       
       - 在Active状态可以调用服务进入瞬态Deactivating，调用onDeactivate，停用节点，回到Inactive状态
       
       - 这3个主状态都可以调用服务进入瞬态ShutingDown，调用onShutdown，关闭节点，成功则进入Finalized状态，节点运行完成
       
         
   
2. 编写控制器

   - cpp
     - on_init：
       - 在该函数内部完成参数获取，通过ParamListener或auto_declare
     - on_configure：
       - 实现发布者、订阅者的初始化