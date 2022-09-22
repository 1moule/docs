# 1. 开shoot

## 1. method 1(rqt)

1. 连接nuc

```
~$ ssh dynamicx@ip
```

2. 运行以下launch

```
mon launch rm_config rm_hw.launch //打开硬件接口
mon launch rm_config load_controller.launch  //加载控制器
```

3. 打开rqt运行控制器

- plugin -> robot_tool -> controller_manager

  - 开启robot state controller（发布、维护tf）joint state controller（可以听到joint state，包括关节的effort velocity position）

  - 开启拨盘校准控制器（开shooter前一定要校准！！！）

  - 开启shooter controller

- plugin -> Topics -> Message Publisher
  - 在上方选中话题，/controller/shooter_controller/command，点+号
  - 点开话题，mode 1为stop、2为ready、3为push；speed的值对应的弹速从rm_control/rm_msg/Shootmsg查看；hz为拨盘拨的频率
  - 注意：一定先进ready再进push，这样urdf中设置的拨盘offset才会生效

## 2. method 2(manual)

1. 连接nuc
1. 停掉自启
1. 

# 2. 注意问题

1. 摩擦轮磨损

- 这个会导致射速波动变大，因为磨损程度不同必然导致初速度波动变大

2. 摩擦轮电机

- 确保电机完好，碰到过电机轴承有问题

3. 弹丸姿态

- 观察弹丸发射前的位置，保证每次的位置都相同
- 根据发射的情况（比如是否双发、卡弹），可以通过改变trigger的offset来解决；或者说一定要调一个合适的offset来使弹丸不双发、不卡弹

4. 弹丸粗糙程度

- 弹丸粗糙程度不同，也可能导致弹丸初速度不同，因为摩擦轮转速和弹丸的速度是符合动量守恒的

5. 单发限位

- 单发限位太软，限不住弹丸，会导致发射前弹丸的位置有问题，从而导致双发
- 单发限位太硬，会导致弹丸被单发限位卡住，导致卡弹
- 单发限位左右偏而非竖直，会导致弹丸发射之后左右偏

6. 底盘移动

- 每次发射的后座力可能会让底盘后移，导致弹丸发射的起点改变，进而导致重复度降低
- 其他原因导致的底盘移动都会导致重复度降低，要观察底盘确定他是否移动
- 底盘移动可以通过把底盘架起来解决，但是一定要架稳

7. 云台移动

- 英雄一定要校准imu，其他兵种可以不用
- 可以通过把底盘架起来，然后关掉orientation controller，这样用的就是底盘的数据，follow的时候底盘不会动，然后云台会follow过去；如果开了这个控制器，那就是用imu的数据，云台yaw动了之后，是底盘会follow过去

8. 撞枪管