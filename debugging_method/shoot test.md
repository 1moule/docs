# 1. 开shoot

## 1. method 1(rqt)

1. 上网关查看nuc的ip

- 浏览器输入网址：192.168.1.1，找到机器人对应的主机名的ip
- 网关帐号密码都是admin

2. 连接nuc

```
~$ ssh dynamicx@ip
```

3. 关闭自启

```
~$ sudo systemctl stop rm_start.service start_master.service
```

4. 运行roscore以及以下launch

```
mon launch rm_config rm_hw.launch //打开硬件接口
mon launch rm_config load_controller.launch  //加载控制器
```

5. 打开rqt运行控制器

- plugin -> robot_tool -> controller_manager

  - 开启robot state controller（发布、维护tf）joint state controller（可以听到joint state，包括关节的effort velocity position）

  - 开启拨盘校准控制器（开shooter前一定要校准！！！并且一定要保证校准成功，即卡到机械限位）

  - 开启shooter controller

- plugin -> Topics -> Message Publisher
  - 在上方选中话题，/controller/shooter_controller/command，点+号
  - 点开话题，mode：1为stop、2为ready、3为push；speed的值对应的弹速从rm_control/rm_msg/Shootmsg查看（要根据当前机器人的射速上限选对应的speed，并且参数文件中也要有对应射速的摩擦轮转速）；hz为拨盘拨的频率
  - 注意：一定先进ready再进push，这样urdf中设置的拨盘offset才会生效

## 2. method 2(manual)

1. 连接nuc
1. 停掉自启
1. 运行roscore以及以下launch

```
mon launch rm_bringup start.launch
```

4. 开启遥控器，右上角拨杆拨到中间(遥控器操作)，或拨到最上(使用键鼠)；左拨杆拨到中间(进ready)，再拨到最上(进push)

## 3. 使用自己的电脑测发射

1. 注释.bashrc以下一行

```
export ROS_MASTER_URI=http://192.168.1.101:11311
```

2. 在.bashrc中添加环境变量

```
export ROBOT_TYPE=hero（机器人类型）
```

3. 关闭所有终端
4. 打开第一个终端

```
roscore
```

5. 第二个终端

```
mon launch rm_config rm_hw.launch
```

- 使用manual

  - 第三个终端

  ```
  mon launch rm_config manual.launch
  ```

  - 第四个终端

  ```
  mon launch rm_dbus rm_dbus
  ```

- 使用rqt

  - 第三个终端

  ```
  mon launch rm_config load_controller.launch
  ```

  - 第四个终端

  ```
  rqt
  ```

6. 再开一个终端

```
rosrun plotjuggler plotjuggler
```

- 听/controller/shooter_controller/state/trigger/set_point、process_value，观察拨盘曲线
- 听/joint_state/left_friction_wheel/velocity、/joint_state/right_friction_wheel/velocity，观察摩擦轮掉速

## 4. 注意

1. 使用nuc时，要把自己电脑上的.bashrc里，ROS_MASTER_URI改成对应的nuc的ip，修改后要重新开终端或source .bashrc；因为rqt，plotjuggler这些工具是在自己的电脑上运行的，他们通过网络跟nuc通信，而master url指定了数据从哪里获取
2. 拨盘曲线每次基本一致，不超调、不过阻尼，基本紧贴期望曲线
3. 每次发射两摩擦轮掉速差10rad/s以内；连续发射，单个摩擦轮掉速极差大概在30rad/s左右，应该是比较正常

# 2. 注意问题（影响重复度的因素）

## 1. 非控制部分

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

- 英雄一定要标定imu，其他兵种可以不用
- imu一定要做好零飘补偿（method：IMU零飘补偿.md）
- 可以通过把底盘架起来，然后关掉orientation controller，这样用的就是底盘的数据，follow的时候底盘不会动，然后云台会follow过去；如果开了这个控制器，那就是用imu的数据，云台yaw动了之后，是底盘会follow过去

8. 撞枪管

- 对于17mm小弹丸，可以直接听声音判断
- 对于42mm大弹丸，个人认为：在测速内壁贴一圈复写纸+白纸为较好的方法

9. 温度

- 一定要**预热！！！**
- 空转温度上升幅度有限且速度较慢，可以套管发射，当射速波动大概稳定在一个值时，可以认为预热完成

10. **机械安装问题**

- **一定**要保证安装没有问题，精细到一个**垫片！！！**
- 确保**定心**没有问题

## 2. 控制部分

1. 调整拨盘offset来改变弹丸初始位置，要求调到他不卡弹、不双发
1. 调拨盘pid，使其不超调，不过阻尼
1. 摩擦轮掉速

- 个人测试：
  - 单个摩擦轮每次掉速波动大约在**30rad/s**
  - 每次发射两个摩擦轮掉速大约差**10rad/s**

## 3. 经验

- **影响上下散布**：射速波动
  - 弹丸新旧、摩擦轮温度、机械安装

- **影响左右散布**：弹丸定位
  - 单发限位是否晃动/歪、弹丸定心

<mark>机械的安装、环境（温度、湿度）、（弹丸的新旧），这是影响重复度最大的因素</mark>

