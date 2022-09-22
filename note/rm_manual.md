## 1. /src/common

1. 这里是manual（手动控制）通用的部分

## 2. /src/referee

1. 这是有关ui界面的部分

## 3. /src其余

1. 不同兵种的手动控制的代码



# 1.src/common/manual_base.cpp

## 1. state_

1. IDLE：空闲
2. RC：remote control，遥控器
3. PC：键鼠

## 2.ManualBase::ManualBase(ros::NodeHandle& nh) : data_(nh), nh_(nh), controller_manager_(nh)

### 1.controller_manager.startStateControllers()

1. controller_manager这个类rm_control/rm_common/include/decision
2. 这个函数用来开启控制器

### 2. .setRising()

1. 

# 2.src/chassis_gimbal_manual.cpp 

## 1. 

```
ChassisGimbalManual::ChassisGimbalManual(ros::NodeHandle& nh) : ManualBase(nh)
```

1. chassis_command_sender：