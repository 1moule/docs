## Legged control note

## 1. MPC

### 1.1 动力学模型

1. 采用质心动力学模型（FullCentroidalDynamics）
   - 系统状态量：6维质心动量、6维机体姿态、3*4=12个关节角度
   - 系统输入：4条腿的3维足底反力（4*3=12个量）、12个关节角速度

### 1.2 参考轨迹

1. 步态：
   - 腿部状态分为（触地）、（离地）。通过组合4条腿的这两种状态可以实现不同步态。
   - 如何设计不同步态：将总的时间划分为多个时间段，规定不同时间段内有哪些腿触地、哪些腿离地，各种组合就能实现各种步态。eg：trot就是左前右后触地然后经过一段时间变为左后右前触地
   - legged control如何实现步态：
     - ocs2 legged robot中写好了GaitKeyboardPublisher和GaitReceiver这两个类。	
       - GaitKeyboardPublisher可以从终端输入读取步态，然后将这个步态对应的ModeSequenceTemplate（包括switchingTimes和modeSequence）发布到话题；
       - 1. GaitReceiver接收ModeSequenceTemplate，并且内部有preSolverRun函数会调用传入的gaitSchedulePtr_的insertModeSequenceTemplate更新步态。
         2. GaitReceiver作为SynchronizedModule添加到solver中，它的preSolverRun会被solver一直调用，实现根据指令实时更新步态
     - reference  manager提供getContactFlags接口，根据步态以及当前时间，返回期望的当前4个腿的触地状态.。根据触地状态进行摆动轨迹规划就可以实现期望的步态。
2. 摆动轨迹
   - 对需要触地的腿，保持落脚点高度为0，足端速度为0
   - 对需要离地的腿，是一条高度变化的轨迹，腿会抬起再落下
   - legged control如何实现摆动轨迹：
     - SwitchedModelReferenceManager重载的modifyReferences函数里面会实时调用swingTrajectoryPtr_的update，传入modeSchedule和地形高度（默认为0），最终得到每条腿的feetHeightTrajectories（1个4维的数组，存放4个SplineCpg类型的vector。实际每个vector只存了一个SplineCpg）。
     - SplineCpg类的构造函数要传入CubicSpline::Node liftOff, scalar_t midHeight, CubicSpline::Node touchDown
       - CubicSpline::Node结构体，包括时间、位置、速度。也就是说liftOff是开始摆动时的时间、位置、速度，touchDown同理。midHeight是中间时间点的足底高度
       - 根据上面这些信息就可以规划出一条连续轨迹。通过SplineCpg的position、velocity等函数，可以根据传入的时间得到指定时间的位置、速度等信息
     - 在preComputation中调用swingTrajectoryPtr_提供的getZpositionConstraint、getZvelocityConstraint设置normal velocity constraints的参数。（这两个函数调用SplineCpg的position、velocity，得到指定时间下的位置、速度）
3. 状态量参考轨迹
   - 收cmd vel，得到targetPose。通过targetPose得到trajectories，发布到话题
     - 提取观测量的第6到12个，也就是base pos（xyz+rpy）
     - 从rpy得到旋转矩阵，旋转矩阵右乘速度指令，也就是把它转到惯性系
     - 速度指令积分得到targe pos
     - 得到stateTrajectory（动量全为0+target pos+默认关节角度也就是站立时关节角度）、timeTrajectory为开始和结束两个时间点、inputTrajectory全为0。最终得到的stateTrajectory相当于站立在期望位置处。为了到达这个位置，触地腿会往前（后）倾（因为要让机体向前（后）运动到期望位置并且要满足足底速度为0的约束；而离地腿会往前摆因为腿要运动到相对于期望机体位置腿竖直的位置。

### 1.3 约束

1. 足底离地时，需要在最优化问题中对期望离地的腿添加足底反力的零力约束来保持系统稳定。具体实现是根据contact flag来决定zero force constrain是否active。如果contact flag是false，那么isActiva返回true，在最优化对当前这个足底添加零力约束。
2. 足底离地时，会activate normal velocity constraints。
   - normal velocity constraints内部构造了一个EndEffectorLinearConstraint来对足底速度进行约束，也就是通过这个约束来实现足底按照期望的速度运动。
   - EndEffectorLinearConstraint通过运动学得到足底的速度和位置
   - Ax*位置+Av\*速度+b=0进行约束，也就是：positionErrorGain\*当前z位置+当前z速度=positionErrorGain\*期望z位置+期望z速度。将positionErrorGain置0就等价于只约束z方向上的速度
3. 足底触地时，同理添加足底速度为0的约束。
4. ocs2的加约束：主要是要完成getValue和getLinearApproximation方法。
   - getValue为例，他返回一个矩阵，返回值就是要等于0的值。比如EndEffectorLinearConstraint返回g(xee, vee)，其中g(xee, vee) = Ax * xee + Av * vee + b，则这个等式约束就是g(xee, vee)=0，也就是Ax * xee + Av * vee = -b。


## 2. WBC

1. wbc将任务写为**目标函数**或者**等式约束+不等式约束**的形式
2. wbc求解出来的优化变量为加速度、足底反力、关节力矩
3. 有加权wbc和分层wbc两种。
   - 加权wbc将优先级低的部分任务乘上权重作为目标函数，将优先级高的任务作为硬约束，求解一个二次规划问题。优先级低的任务相当于被作为软约束，二次规划求解出来的wbc优化变量会尽量满足这些任务，同时必须满足硬约束即优先级高的任务
   - 分层wbc，通过在高优先级任务的零空间中求解低优先级的任务
4. Legged control将任务写为等式约束+不等式约束。通过加权wbc求解二次规划问题得到优化变量

## 3. Hardware

### 3.1 unitree hw

1. 宇树A1开放出来的接口只有交换机的网口，在自己电脑跑程序控制硬件得用宇树sdk，用udp获取底层状态（电机、IMU、足端传感器），用udp发送电机指令

## 4. 控制逻辑

1. 线性卡尔曼滤波对系统状态进行观测
2. ocs2求解最优化问题，得到当前时刻系统的最优状态和最优输入
3. 根据mpc得到的最优估计状态和最优输入，通过wbc求解关节力矩指令
4. 将最优状态中的关节角度、最优输入中的关节角速度作为期望，对每个关节的角度、角速度进行PD控制，并将wbc求出来的力矩指令作为力矩前馈

## 5. 关于参数

1. 通过各种loadSettings函数来加载指定配置文件中的指定关键字中的参数。这些函数的输入都是文件名+参数的关键字。某个关键字中的参数以树的形式存在，函数返回的是参数树。

## 6. 上实机

1. 关闭自启的程序

   - ```bash
     sudo pkill -f keep_program_alive.sh
     sudo pkill -f A1_sport_1
     ```

     杀死守护进程以及运控程序进程，不过这是一次性的，重启后所有的自启都会重启

2. 主控PC运行legged control

   - 设置环境变量robot_type

     ```
     export ROBOT_TYPE=a1（a1可替换为其它的type，具体可以填哪些看launch）
     ```

     可以把这一句写到bashrc中

   - 启动master

     ```
     roscore
     ```

   - 运行hw

     ```
     mon launch legged_unitree_hw legged_unitree_hw.launch
     ```

   - 运行控制器

     ```
     roslaunch legged_controller load_controller.launch
     ```

   - rqt controller manager加载控制器

   - 发布话题/cmd_vel，控制器受到参考轨迹后就会起身

   - 在加载控制器的终端切换步态

3. 
