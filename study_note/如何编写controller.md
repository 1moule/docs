1. 创建功能包

```
catkin create pkg pkg_name dependence
```

2. 创建目录（按照一个标准的功能包）

```
mkdir src include launch config 
```

3. 编写cpp文件以及.h文件

- cpp：定义头文件中的函数

- .h

```c++
namespace MyController
{
class Controller:public controller_interface::MultiInterfaceController<rm_control::RobotStateInterface,hardware_interface::EffortJointInterface>
{
    public:
    bool init(hardware_interface::RobotHW* robot_hw, ros::NodeHandle& root_nh, ros::NodeHandle& controller_nh));
    void starting(const ros::Time& time);
    void update(const ros::Time& time, const ros::Duration& period);
    void stop(const ros::Time& time, const ros::Duration& period);
}
}

//头文件一般写接口，写控制器要继承一个基类；这四个函数是控制器最基本的接口，必须要有，因为ros control中使用控制器的方法就是：加载控制器的时候调用一次init、开启控制器时调用staritinig、然后controller manager以1k hz的频率一直调用update、停止控制器调用stop
```

4. 修改CmakeList.txt

```c++
//添加以下两句
## Declare cpp executables
add_library(${PROJECT_NAME}
        src/MyController.cpp
        )

## Specify libraries to link executable targets against
target_link_libraries(${PROJECT_NAME}
        ${catkin_LIBRARIES}
        )
    
//如果想要调用其他功能包的头文件，添加以下
    find_package(catkin REQUIRED
        COMPONENTS
        other_package
        )
```

5. 将控制器注册为插件

- 只有将控制器注册为插件，这个控制器才能被congtroller manager加载
- 具体方法参照“控制器注册为插件.md”
