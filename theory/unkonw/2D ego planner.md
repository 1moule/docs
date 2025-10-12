# 2D ego planner

1. 规划从起点到终点的全局路径、用minsnap生成轨迹
2. minsnap生成当前起点到局部终点的轨迹
3. b样条轨迹优化

## 1. 局部规划（ego就是一个local planner）

1. rviz选择2d nav goal，会发布目标点到话题/move_base_simple/goal，这个话题会remap到话题/way_point

2. ego_planner_node节点实例化一个EGOReplanFSM类，类中订阅/way_point，在回调中：

   - 将当前机器人在惯性系下的位置作为初始位置，将话题得到的目标位置作为最终位置
   - 调用planner_manager_->planGlobalTraj规划路径以及根据路径生成轨迹
   
     - 首先定义一个vector存放全部路径点，先将上面得到的起点和终点丢进去
     - 如果起点和终点距离太远则设置中间路径点。中间点通过线段等比例线性插值得到，会把一个长线段分成均匀短线段
     - 假设最终得到的路径有n个点，则定义一个3行n列的矩阵用来存放路径点（因为有n个3维路径点），每一列都是一个3维的路径点。定义一个n-1维的矩阵存放时间轨迹，用相邻两个路径点的距离除以速度得到时间轨迹。
     - 完成上面的步骤，最终得到了一系列的路径点、时间轨迹，也就是完成了路径规划。最后通过minSnap对这些稀疏不平滑的路径点进行轨迹规划，得到稠密平滑的轨迹。
     - minSnapTraj轨迹规划，得到初始轨迹
     - 最终得到的轨迹会存放在GlobalTrajData类对象中的 PolynomialTraj类型成员变量，同时设置FSM状态为GEN_NEW_TRAJ
   
2. 类中还定义了一个定时器exec_timer_，定时来根据状态执行对应的动作函数。在定时器回调函数中：

   - WAIT_TARGET：没有收到目标点直接返回，反之则进入GEN_NEW_TRAJ

   - GEN_NEW_TRAJ：

     - 进行实时轨迹重规划
       - 寻找一个局部目标点（以时间步长t_step向前查找，找到第一个离当前起点的距离大于planning_horizen_的点作为局部目标点）
       - 调用planner_manager_->reboundReplan进行轨迹重规划
         - step1 进行轨迹初始化。用min snap生成初始轨迹，并插入随机扰动的中间点
         - step2 B样条优化，优化过程会考虑障碍信息，对控制点进行优化，使最终优化出来的轨迹避开障碍。
         - 重规划的结果存在LocalTrajData类对象中
   
   - 
   
      
   
4. traj_server节点

   