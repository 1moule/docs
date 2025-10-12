# OCS2

## 1. 使用OCS2求解最优化问题的步骤

1. 构建最优化问题（至少定义系统动力学以及代价函数）
   - ocs2提供了一个结构体（Optimal Control Modules），存放指向 最优化问题要定义的内容 的指针，包括Cost、Constraints、Dynamics、pre-computation等。
   - 通常定义3种代价函数和约束：预测时域中间的（intermediate）、系统切换时的（prejump）、预测时域末尾（final）的。代价函数和约束分别都是一组的而不是单个的，因此可以很方便地添加新的代价函数和约束而不用修改其他代码
   - Costs：
     - 时域中间的可以使用 状态+输入关于时间的函数或状态关于时间的函数
     - 切换时和末尾时只能用状态关于时间的函数
     - 代价函数通过继承StateCost or StateInputCost类实现（在“ocs2_core/cost” 中）
   - Constraints：
     - 同样分为预测时域中间的（intermediate）、系统切换时的（prejump）、预测时域末尾（final）的
     - intermediate可以用time, state, input 或 time, state；prejump、final使用 time, state
     - 软约束是尽量满足但不强制要求的约束。
     - 硬约束是不可违背的约束，包括等式、不等式约束。通常说的约束就是硬约束
   - Dynamics：
     - 继承SystemDynamicsBase
       - 需要在子类中实现SystemDynamicsBase的纯虚函数linearApproximation
       - 由于SystemDynamicsBase继承ControlledSystemBase并且没有实现ControlledSystemBase中的纯虚函数，所以还需要实现ControlledSystemBase中的纯虚函数computeFlowMap
     - 继承SystemDynamicsBaseAD（SystemDynamicsBase的自动微分版本，继承SystemDynamicsBase）
       - linearApproximation和computeFlowMap都通过CppAd来实现，不需要自己实现
       - 子类只需要实现systemFlowMap
   - pre-computation（可选）：
     - 在评估 cost、constraints、system dynamics之前会调用PreComputation::request，在request中完成预计算并将计算结果存储起来。cost、constraints、system dynamics的getter方法可以获取预计算的结果
   - 
2. 设置MPC loop
   - MPC Interface
     - 负责使用最新的测量值安全地**更新**求解器
   - MRT Interface
     - 负责安全地**访问**求解器结果
     - updatePolicy()加载最新的MPC策略
     - evaluatePolicy()，获取指定时刻的 MPC预测的状态、最优输入。MPC可以预测系统在预测时域内的状态以及最优输入，通过线性插值可以得到指定时刻的预测状态以及输入。



## 2. ocs2+lmpc使用

### 2.1 构建最优化问题

1. 在interafce类的构造函数中，检查task file和libFolder，并获取MPC的设置以及SQP等求解器的参数设置
2. 在构造函数中，构建最优化问题，至少包括dynamics、cost、rollout、initialization
2. 求解器需要设置一个继承自ReferenceManagerDecorator的RosReferenceManager，用来与ros交互；ReferenceManagerDecorator的构造需要传入一个继承ReferenceManagerInterface并实现了内部接口的类，用来实例化内部的referenceManagerPtr。

### 2.2 在controller中使用mpc

1. 在控制器构造函数或初始化函数中 实例化interface，设置mpc loop

   - setupMpc：构造求解器，并设置求解器的ReferenceManager
   - setupMrt：单开线程，运行advanceMpc函数
     - advanceMpc函数会调用MPC的run函数。
     - 在run函数中调用calculateController函数。
     - calculateController函数会调用内部的solver的run函数。
     - solver的run函数在solverBase中定义，会调用preRun、runImpl、postRun，其中runImpl在子类中实现
       - preRun会调用referenceManager中定义的preSolverRun
       - postRun与SolverSynchronizedModule有关
       - runImpl则是实现 求解最优化问题 的函数

2. 在starting中获取inittial policy

3. 在主循环中

   ```c++
   // Update the current state of the system
   mpcMrtInterface_->setCurrentObservation(currentObservation_);
   // Load the latest MPC policy
   mpcMrtInterface_->updatePolicy();
   // Evaluate the current policy
   mpcMrtInterface_->evaluatePolicy(currentObservation_.time, currentObservation_.state, optimizedState, optimizedInput, plannedMode);
   ```

   就可以得到mpc估计的系统最优状态，以及当前的系统最优输入
   
   

## 3. ocs2+nmpc使用

### 3.1 创建interface包，构建最优化问题

1. dynamics:

   - 使用质心动力学模型需要通过实例化PinocchioCentroidalDynamicsAD类来完成，这个类的构造函数需要传入以下参数

     ```c++
     * @param [in] pinocchioInterface : The pinocchio interface.
     * @param [in] CentroidalModelInfo : The centroidal model information.
     * @param [in] modelName : Name of the generate model library
     * @param [in] modelFolder : Folder to save the model library files to
     * @param [in] recompileLibraries : If true, the model library will be newly compiled. If false, an existing library will be loaded if
     *                                  available.
     * @param [in] verbose : print information.
     ```

   - Model Settings:

     - 在ocs2 legged robot 例程中，创建ModelSetting结构体，并通过loadModelSettings函数从info文件中获取参数，然后返回存储了 除了pinocchioInterface和CentroidalModelInfo和modelName 的参数的ModelSetting结构体。
     - ModelSetting中还有jointNames的vector，用来后面通过createPinocchioInterface函数创建PinocchioInterface，不在jointNames中的joint会被从urdf中移除

   - pinocchioInterface通过centroidal_model::createPinocchioInterface接口构造，传入urdf以及ModelSetting中的jointNames。

   - CentroidalModelInfo通过createCentroidalModelInfo函数构造。需要传入前面构造的pinocchioInterface，DefaultJointState（从referenceFile中读取），Model Settings中的contactNames3DoF和contactNames6DoF。

     - 一个要理解的点：numJointState等于pinocchio的model的nq - 6。
       - model.nq是配置空间中的变量数量，它与关节类型以及参数化方法有关（比如一个旋转关节，需要用旋转描述姿态，旋转可以用四元数或欧拉角表示。用四元数则nq=4，用欧拉角则nq=3；再例如对于平移关节，要用3维的平移描述姿态，那么nq就是3）
       - 在浮动基中，nq包括关节的和浮动基座的，因为通常旋转关节只有一个转轴也就是一个自由度，用欧拉角只需要一个变量就可以描述，所以关节的nq=关节的数量；浮动基包括3维平移和绕3维的旋转，因此基座的nq=6；整个模型的nq=关节nq+基座nq=关节数量+6。因此关节数量=model.nq-6

   - 自己创建的动力学模型的类中需要实例化PinocchioCentroidalDynamicsAD类，需要把ModelSetting结构体、pinocchioInterface、centroidalModelInfo作为构造函数参数传给PinocchioCentroidalDynamicsAD的构造函数来构造PinocchioCentroidalDynamicsAD类

2. initializition

   - mpc求解时会初始化state和input轨迹来让求解速度加快，通过initializeStateInputTrajectories函数实现。
     - 这个函数首先会确定插值时间段，然后根据初始时间是否在时间段内选择通过 插值 或 传入的init state 进行一步state的初始化。
     - 在之后，如果在普通时间节点（也就是不在系统切换之类的节点），当前事件在可插值时间外则使用initializer求初值，反之用上一次求解的结果插值求初值。在事件节点用stateTrajectory的最后一个

3. reference manager

   - 用于定义目标轨迹以及进行模式调度。继承ReferenceManager实现的SwitchedModelReferenceManager包括GaitSchedule和SwingTrajectoryPlanner，GaitSchedule用来进行模式调度，SwingTrajectoryPlanner根据当前的模式生成轨迹
   - SwingTrajectoryPlanner
     - 
   - 

4. 



## 4. 各种求解器

### 4.1 IPM

### 4.2 SQP